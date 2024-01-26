---
layout: post
title: ChatGLM2-6B install
category: [other]
tags: [other,ai]
---


### 准备环境

- python3 (miniconda)
### 下载代码和模型

```
#  workspace = pwd
git clone https://github.com/THUDM/ChatGLM2-6B

cd ChatGLM2-6B

mkdir THUDM
cd THUDM

# git lfs not found,  
# brew update
# brew install git-lfs
git lfs install

GIT_LFS_SKIP_SMUDGE=1


git clone https://huggingface.co/THUDM/chatglm2-6b

# if you want to clone without large files – just their pointers
# prepend your git clone with the following env var:
```

### 安装依赖

```

# 回到 ${workspace}目录， 也有ChatGLM2-6B 目录
cd ..

# 如果网络不好可以切换镜像  
pip3.9 install -r requirements.txt


``` 

### demo 配置

* 注意 THUDM/chatglm2-6b  表示在${workspace}目录 下路径。 如果嫌烦直接用却对路径即可。 



配置不同算力


```
# GPU
model = AutoModel.from_pretrained("THUDM/chatglm2-6b", trust_remote_code=True).cuda()

# CPU 
model = AutoModel.from_pretrained("THUDM/chatglm2-6b", trust_remote_code=True).float()

# mac 
# 对于搭载了 Apple Silicon 或者 AMD GPU 的Mac，可以使用 MPS 后端来在 GPU 上运行 ChatGLM-6B。需要参考 Apple 的 官方说明 安装 PyTorch-Nightly（正确的版本号应该是2.1.0.dev2023xxxx，而不是2.0.0）。
#目前在 MacOS 上只支持从本地加载模型。将代码中的模型加载改为从本地加载，并使用 mps 后端：
model = AutoModel.from_pretrained("THUDM/chatglm2-6b", trust_remote_code=True).half().to('mps')

```





#### web ui

```
# 以web_demo.py 为例子

#  关注 以下两行

tokenizer = AutoTokenizer.from_pretrained("THUDM/chatglm2-6b", trust_remote_code=True)
model = AutoModel.from_pretrained("THUDM/chatglm2-6b", trust_remote_code=True).cuda()
```

####  term
参考链接： https://huggingface.co/THUDM/chatglm2-6b

```
python3.9
from transformers import AutoTokenizer, AutoModel
tokenizer = AutoTokenizer.from_pretrained("THUDM/chatglm2-6b", trust_remote_code=True)
model = AutoModel.from_pretrained("THUDM/chatglm2-6b", trust_remote_code=True).float()
model = model.eval()
response, history = model.chat(tokenizer, "你好", history=[])
print(response)
```


### 遇到的问题

1.  AttributeError: 'Textbox' object has no attribute 'style'


```
user_input = gr.Textbox(show_label=False, placeholder="Input...", lines=10).style(
AttributeError: 'Textbox' object has no attribute 'style'
``` 

解决方法：
重新安全 gradio 为 3.50.0

https://github.com/THUDM/ChatGLM-6B/issues/1417

```
pip3.9 uninstall gradio
pip3.9 install gradio==3.50.0
```

 