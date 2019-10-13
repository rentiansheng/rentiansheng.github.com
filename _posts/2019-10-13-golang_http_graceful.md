---
layout: post
title:  mysql http 优雅重启
category: [golang, graceful]
tags: [golang]
---
### 目标
1.  停止服务（正在处理HTTP 请求不受影响，需要等待其完成）
2.  启动服务
3. 服务不中断

### 遇到的问题

#### 1. 如何知道HTTP 请求是否已经处理结束

这里主要使用Server.Shutdown 方法。 
使用Shutdown方法要注意一下问：
- golang 版本必须是1.8及其以上的版本。
- 参数context 使用WithTimeout 生成的时候，如果调用Shutdown后HTTP 请求处理时间， 将不会等待
- 调用shutdown后， 不会接受新的请求， http.Server会收到 http.ErrServerClosed 
- Shutdown 只是关闭socket接受新的请求。 如果有新的请求发来，不会直接拒绝。 


#### 2. 如何解决在等待HTTP服务结束，无法启动服务的问题？

使用以下的方法解决问题：

-  exec.Command开启新的服务
- scoket file descriptor 共享监听


### 具体实现

#### 优点
- 对自我的更新
- 服务不中断
#### 缺点
- 无法使用supervisor管理
- 通过graceful更新的进程父进程会变成init进程
- 在执行shutdown 会有两个进程
- ps 查看进程，参数会发生变化

[查看完整的代码](https://github.com/rentiansheng/golang-graceful-restart-demo/blob/master/graceful/graceful.go "完整的代码")

主要代码如下：

``` golang 

var (
	// 等待更新命令
	graceful = make(chan bool, 1)
)

func main() {
	var graceful bool
	// 是否通过程序自身完成优雅的重启
	flag.BoolVar(&graceful, "graceful", false, "graceful start")
	flag.Parse()

	server := &http.Server{Addr: ":9999"}
	var listenFile *os.File
	if graceful {
		// 使用共享的listen fd， 在执行shutdown的过程中， 
		// 启动不会收到listen tcp :8080: bind: address already in use
		// why fd = 3. 
		// ExtraFiles specifies additional open files to be inherited by the
        // new process. It does not include standard input, standard output, or
        // standard error. If non-nil, entry i becomes file descriptor 3+i.
		listenFile = os.NewFile(3, "")
	} else {
		// 首次启动， 需要建立socket 链接
		listen, err := net.Listen("tcp", ":9999")
		if err != nil {
			log.Fatal(err)
		}
		l := listen.(*net.TCPListener)
		listenFile, err = l.File()
		if err != nil {
			log.Fatal(err)
		}
	}
	go runWorker(listenFile)
	httServer(listenFile, server)
}

// runWorker 启动新的worker 来接受HTTP 请求
func runWorker(listenFile *os.File) {
	// 等待更新命令
	<-graceful
	fmt.Println("execute start")

	// 使用新的二进制更新本身
	execFile := "./graceful"
	execCmd := exec.Command(execFile, "-graceful=true" )
	// 共享 stdout, stderr
	execCmd.Stdout = os.Stdout
	execCmd.Stderr = os.Stderr
	// 共享socket 链接
	execCmd.ExtraFiles = []*os.File{listenFile}
	// 父进程退出，子进程可以接续执行
	execCmd.SysProcAttr = &syscall.SysProcAttr{
		Setpgid: true,
	}
	// 这里是阻塞，需要注意
	err := execCmd.Run()
	// 这里如果想知道进程是否准备好，可以接受新的任务。 
	// 可以在xecCmd.ExtraFiles 多传入一个file，来处理。相对比较麻烦
	//  或者通过信号量
	fmt.Println("execute end", err)
}

func httServer(file *os.File, server *http.Server) {
	l, err := net.FileListener(file)
	if err != nil {
		log.Fatal(err)
		return
	}
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello, pid:%d, path:%s, time:%s",os.Getpid(), html.EscapeString(r.URL.Path), time.Now().String())
	})
	// 执行优雅的重启 HTTP api
	http.HandleFunc("/graceful", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello, graceful  pid:%s, time:%s", os.Getpid(),time.Now().String())
		close(graceful)
	})
	http.HandleFunc("/sleep", func(w http.ResponseWriter, r *http.Request) {
		time.Sleep(50 * time.Second)
		fmt.Fprintf(w, "Hello, graceful pid:%d,  time:%s", os.Getpid(),time.Now().String())
	})

	go func() {
		err := server.Serve(l)
		// http.ErrServerClosed 是有http shutdown 引起
		if err != nil && err != http.ErrServerClosed {
			log.Fatalf("listen %s error. err:%s\n", server.Addr, err)
		}
		return
	}()
	select {
			case <-shutdown(server):
	}
}

func shutdown(server *http.Server) chan bool {
	// 执行优雅的的命令
	<-graceful
	// 这里最好有sleep， 保证接受新服务已经启动,
	log.Println("Server is shutting down...")

	server.SetKeepAlivesEnabled(false)
	err := server.Shutdown(context.Background())
	if err != nil {
		log.Fatalf("Could not gracefully shutdown the server: %v \n", err)
	}
	quitChan := make(chan bool, 1)
	quitChan <- true
	return quitChan
}
```





