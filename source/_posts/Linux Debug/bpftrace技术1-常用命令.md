---
title: bpftrace技术1-常用命令
date: 2025-12-10
tags:
  - linux
  - bpftrace
---
# bpftrace是什么

一种追踪内核态和用户态的新技术

## 看可以跟踪哪些内核函数

```c
bpftrace -l | grep kprobe
```

## 跟踪内核函数例子

跟踪 `net/ipv4/netfilter/ip_tables.c looks promising` 的两个函数: `compat_do_ipt_get_ctl` 和 `do_ipt_get_ctl`。

```shell
~# bpftrace -e 'kprobe:do_ipt_get_ctl { printf("function was called!\n"); }'
Attaching 1 probe...
function was called!
function was called!
```

`compat_do_ipt_get_ctl` 函数签名如下

```c
static int compat_do_ipt_get_ctl(struct sock *sk, int cmd, void __user *user, int *len)
```

打印调用命令，pid，入参的bpf程序如下

```c
#include <net/sock.h>

kprobe:do_ipt_get_ctl
{
    printf("called by %s (pid: %d). and: %d\n", comm, pid, ((sock *)arg0)->__sk_common.skc_family);
}
```

执行结果如下

```c
~# bpftrace test.bpf
/bpftrace/include/stdarg.h:52:1: warning: null character ignored [-Wnull-character]       
/lib/modules/4.19.0-8-amd64/source/arch/x86/include/asm/bitops.h:209:2: error: 'asm goto' constructs are not supported yet
/lib/modules/4.19.0-8-amd64/source/arch/x86/include/asm/bitops.h:256:2: error: 'asm goto' constructs are not supported yet
/lib/modules/4.19.0-8-amd64/source/arch/x86/include/asm/bitops.h:310:2: error: 'asm goto' constructs are not supported yet
/lib/modules/4.19.0-8-amd64/source/arch/x86/include/asm/jump_label.h:23:2: error: 'asm goto' constructs are not supported yet
/lib/modules/4.19.0-8-amd64/source/arch/x86/include/asm/signal.h:24:2: note: array 'sig' declared here
Attaching 1 probe...
called by iptables-legacy (pid: 2981). and: 2
called by iptables-legacy (pid: 2981). and: 2
```

 `2` 的含义解释如下

```c
/usr/src/linux-headers-4.19.0-8-common/include/linux/socket.h
160 /* Supported address families. */
161 #define AF_UNSPEC   0
162 #define AF_UNIX     1   /* Unix domain sockets      */
163 #define AF_LOCAL    1   /* POSIX name for AF_UNIX   */
164 #define AF_INET     2   /* Internet IP Protocol     */
165 #define AF_AX25     3   /* Amateur Radio AX.25      */
166 #define AF_IPX      4   /* Novell IPX           */
```

### 其他例子
```c
bpftrace -e 'kprobe:vfs_open { printf("open path: %s\n", str(((path *)arg0)->dentry->d_name.name)); }'
```
