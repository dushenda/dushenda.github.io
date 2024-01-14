---
title: bcc初体验
date: 2024-01-14
tags: 
	- bpf
	- OS
	- trace
---
## 安装
### Ubuntu
BCC已经打包到Ubuntu的multiverse仓库，名字`bpfcc-tools`，使用如下命令安装
```console
sudo apt-get install build-essential bpfcc-tools linux-header-$(uname -r) bpftrace
```

## 使用
funccount

stackcount