---
layout: post
title:  POST 和 GET 在中间件的处理差异（502错误分析）
category: ["测试","HTTP"]
tags: [""]
---

### 背景

在做压测的时候，出现502错误. 但是将POST 改为GET 就没有问题。 排除项目代码问题。怀疑是中间件的问题。 
当前项目是Python的Django，使用gunicorn作为WSGI服务器，nginx作为反向代理服务器.

> 环境：Django + gunicorn gthread + nginx  
> 现象：高并发下 POST 请求偶发 502，GET 请求无此问题，两者业务逻辑完全相同

### 构造测试环境



在 200 并发用户、持续压测场景下：

| 路径 | 请求数 | 失败数 | 失败率 | 失败类型 |
|---|---|---|---|---|
| `[direct] GET`（绕过 nginx） | 2610 | 11 | 0.42% | `status=0`（TCP 断连） |
| `[direct] POST`（绕过 nginx） | 2575 | 13 | 0.50% | `status=0`（TCP 断连） |
| `[nginx] GET`（经过 nginx） | 2619 | **0** | **0.00%** | — |
| `[nginx] POST`（经过 nginx） | 2524 | **14** | **0.55%** | **502 Bad Gateway** |

**结论**：GET 和 POST 在 gunicorn 层面失败率相同（~0.5%），但经过 nginx 后，GET 失败全部消失，POST 失败变成 502 暴露给客户端。问题在 nginx 层，不在业务代码。

 
###  结论

#### 原因

1. gunicorn --max-requests=500 — worker 每处理 500 个请求后自动退出，关闭 TCP 连接
2. nginx upstream 收到 TCP RST（recv() failed: Connection reset by peer）
3. GET：nginx 认为 GET 是幂等的（RFC 7230），自动 retry 到下一个 worker → 客户端看到 200，无感知
4. POST：nginx 认为 POST 不可重试（非幂等，可能有副作用）→ 直接返回 502 Bad Gateway 
 
#### 解决方案


**`--max-requests-jitter=100`**：让多个 worker 的退出时间随机错开，避免同时退出导致大批连接同时失效。有效，但单次退出的窗口期依然存在。

**`--graceful-timeout=120`**：控制 worker 等待 in-flight 线程完成的时间上限。对本场景无效——请求只有 50–100ms，30 秒默认值已足够，问题不在这里。

##### 效果

| 方案 | `[nginx] POST` 502 数 | 变化 | 说明 |
|---|---|---|---|
| 基线（原始配置） | **14**（0.55%） | — | 基准 |
| `--max-requests-jitter=100` + `--graceful-timeout=120` + `--keep-alive=5` | **9**（0.35%） | ↓ 36% | jitter 错开退出时间，有效但不彻底 |
| 上述 + nginx `keepalive_timeout=4s`（< gunicorn keepalive=5s） | **14**（0.54%） | 无改善 | RST 来自进程退出，不是空闲超时，此方案无效 |
| 上述 + nginx `proxy_next_upstream non_idempotent` | **10**（0.40%） | ↓ 29% | 部分 retry 成功，但单 upstream 时 retry 也可能踩 RST |
| `proxy_request_buffering on` | **2**（0.16%） | ↓ 86% | nginx 先缓冲完整 body 再连接 upstream，大幅缩小时间窗口 |


### 过程


#### 触发源：gunicorn `--max-requests=500`

gunicorn 配置了 `--max-requests=500`，每个 worker 处理满 500 个请求后自动退出并由 arbiter 拉起新 worker。

**源码路径**（`gunicorn/workers/gthread.py`）：

```python
def handle_request(self, req, conn):
    self.nr += 1
    if self.nr >= self.max_requests:
        if self.alive:
            self.log.info("Autorestarting worker after current request.")
            self.alive = False      # 标记退出
        resp.force_close()          # 当前响应加 Connection: close
    # 继续正常处理当前请求，返回 200
```

**日志证据**：
```
[INFO] Autorestarting worker after current request.
[INFO] Worker exiting (pid: 10)
[INFO] Booting worker with pid: 522
```

第 500 个请求本身**正常完成并返回 200**，worker 随后退出。

#### RST 的产生：OS 强制清理 socket

worker 进程退出时，OS 对该进程持有的**所有未关闭 TCP socket** 执行强制清理。

gunicorn 的 `util.close()` 实现：

```python
def close(sock):
    try:
        sock.close()   # 直接 close，没有先 shutdown(SHUT_WR)
    except socket.error:
        pass
```

当 socket 接收缓冲区还有未读数据时（nginx 已发来新请求），直接 `close()` 导致 OS 发送 **TCP RST**（而非优雅的 FIN）。

**nginx 错误日志**：
```
recv() failed (104: Connection reset by peer) while reading response header from upstream,
  request: "POST /api/items/ HTTP/1.1"
```

错误码 104 = `ECONNRESET` = TCP RST。

#### 出问题的是哪些请求

**不是**第 500 个请求（它正常完成）。  
**是**第 501、502、503... 个请求——这些请求通过 nginx keepalive 连接池里的**旧连接**路由到了已退出的 worker，被 OS 的进程退出清理发出的 RST 拒绝。

时序：

```
t=0    worker-A 处理完第 500 个请求，alive=False，进程退出
       OS 对 worker-A 持有的所有 TCP 连接发 RST

t=0+   nginx 连接池里有若干条 keep-alive 连接指向 worker-A（已死）
       这些连接变成"僵尸连接"

t=1    新请求到来，nginx 从连接池取出一条僵尸连接，发送请求
       OS 回 RST（进程已死）
       nginx 收到 104: Connection reset by peer
```

#### nginx 的 GET/POST 处理差异

GET 和 POST 的不对称处理：nginx retry 策略

nginx 对 upstream 连接失败的 retry 策略遵循 RFC 7230：

| 方法 | 是否幂等 | nginx 是否 retry |
|---|---|---|
| GET、HEAD | 是 | **是**（自动重试到下一个可用连接） |
| POST、PUT、PATCH | 否 | **否**（不重试，直接返回 502） |

**nginx 日志铁证**：

```
# GET：upstream 502 → nginx 自动重试 → 客户端看到 200
GET  status=200  upstream_status=502, 200  upstream_addr=...8000, ...8000

# POST：upstream 502 → nginx 不重试 → 客户端看到 502
POST status=502  upstream_status=502       upstream_addr=...8000
```

同样的 RST 事件，GET 被 nginx 静默重试消化，POST 直接暴露为 502。

 

#### 为什么多 worker 也无法消除

gunicorn 启动了 2 个 worker（`-w 2`），但问题依然存在，原因如下：

**TCP 连接是点对点绑定的**。一条连接通过 `accept()` 被某个 worker 接受后，就固定属于该 worker。nginx 的 keepalive 连接池按连接存储，不按 worker 负载均衡。

```
nginx 有 18 个 worker 进程（worker_processes auto，18 核）
每个 nginx worker 维护独立的 keepalive 连接池（keepalive=32）
总连接池 = 18 × 32 = 576 条连接，分布在 gunicorn worker-A 和 worker-B 上

worker-A 退出时：
  ~288 条连接（指向 worker-A）全部变成僵尸
  nginx 不知道 worker-A 已死，继续从池里取这些连接发请求
  → RST → POST 502
```

增加 worker 数量只能**降低概率**（每个 worker 分摊的连接比例减小），无法消除：

| worker 数 | 单次退出影响连接比例 |
|---|---|
| 2 | ~50% |
| 4 | ~25% |
| 8 | ~12.5% |

 


```
gunicorn --max-requests=500
    ↓ worker 处理满 500 个请求后退出
    ↓ 进程退出时 OS 对所有 socket 执行强制 close()
    ↓ socket 接收缓冲区有未读数据 → OS 发 TCP RST（非优雅 FIN）
    ↓ nginx keepalive 连接池里的旧连接变成僵尸
    ↓ 新请求复用僵尸连接 → RST
        ├── GET：nginx 自动 retry（RFC 7230 幂等方法）→ 200，客户端无感知
        └── POST：nginx 不 retry（非幂等）→ 502 Bad Gateway
```

这是 gunicorn gthread worker 的已知行为（`util.close()` 没有先 `shutdown(SHUT_WR)`），结合 nginx 的 RFC 合规 retry 策略，导致 POST 独有的 502。

 
