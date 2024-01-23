---
title: qemu调试linux（一）
date: 2024-01-24
tags:
- linux
- qemu
---
## 编译内核

### 下载
kernel：https://mirrors.edge.kernel.org/pub/linux/kernel/

镜像：https://mirrors.ustc.edu.cn/kernel.org/linux/kernel/

### 编译

```console
root@dushenda:/home/dsd/Code/linux-5.18.10# make menuconfig
root@dushenda:/home/dsd/Code/linux-5.18.10# make -j`nproc`
```
![](qemu调试linux（一）_20240124_1.png)
编译完成得到
![](qemu调试linux（一）_20240124_2.png)

## 制作initramfs
### 使用busybox

