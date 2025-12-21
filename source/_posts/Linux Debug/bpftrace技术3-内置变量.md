---
title: bpftrace技术3-内置变量
date: 2025-12-10
tags:
  - linux
  - bpftrace
---
# 内置变量
bpftrace 的内置变量是其强大功能的基石，它们能让你在脚本中轻松获取事件触发时的上下文信息。

| 变量类别          | 变量名                        | 说明与典型应用场景                                                            |
| ------------- | -------------------------- | -------------------------------------------------------------------- |
| **进程/线程信息**​  | `pid`                      | 当前**进程ID**。用于过滤特定进程的事件。                                              |
|               | `tid`                      | 当前**线程ID**。用于进行更精细的线程级分析。                                            |
|               | `comm`                     | 当前**进程名**（最多16个字符）。常用于按进程名进行统计和过滤。                                   |
|               | `uid`                      | 当前**用户ID**。用于分析特定用户的操作。                                              |
| **时间信息**​     | `nsecs`                    | 自系统启动以来的**纳秒级时间戳**。常用于计算函数耗时、事件间隔等。                                  |
| **CPU/系统信息**​ | `cpu`                      | 当前事件发生时所在的**CPU处理器ID**。                                              |
|               | `curtask`                  | 当前进程的内核`task_struct`结构体地址（以64位无符号整数表示），用于高级内核调试。                     |
| **探针上下文信息**​  | `arg0`, `arg1`, ... `argN` | **Kprobe/Uprobe 的参数**。用于获取被探测函数的参数值（注意：Tracepoint 不可用）。              |
|               | `args`                     | **Tracepoint 的参数结构体**。用于通过 `args->field_name`的方式访问 Tracepoint 的特定字段。 |
|               | `retval`                   | **Kretprobe/Uretprobe 的返回值**。用于获取被探测函数的返回值。                          |
|               | `func`                     | 当前触发的 **Kprobe/Uprobe 所探测的函数名**。                                     |
|               | `kstack`/ `ustack`         | 当前时刻的**内核栈**或**用户栈**描述。用于分析代码执行路径，定位性能瓶颈                             |
# 例子
## **计算函数执行时间**
```shell
# 测量 vfs_read 函数的执行时间（微秒）
~ » bpftrace -e 'kprobe:vfs_read { @start[tid] = nsecs; }
                                kretprobe:vfs_read /@start[tid]/ {
                                    $duration_us = (nsecs - @start[tid]) / 1000;
                                    @us = hist($duration_us);
                                    delete(@start[tid]);
}'
Attaching 2 probes...
^C

@start[551]: 84374532046638
@start[323672]: 84374600291711
@us:
[0]                   45 |@@@@@                                               |
[1]                  452 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
[2, 4)               264 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                      |
[4, 8)               338 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@              |
[8, 16)               43 |@@@@                                                |
[16, 32)              25 |@@                                                  |
[32, 64)               4 |                                                    |
[64, 128)              0 |                                                    |
[128, 256)             0 |                                                    |
[256, 512)             0 |                                                    |
[512, 1K)              6 |                                                    |
[1K, 2K)               0 |                                                    |
[2K, 4K)               3 |                                                    |
[4K, 8K)               1 |                                                    |
```

## **追踪特定进程的系统调用**
```shell
~ » bpftrace -e 'tracepoint:syscalls:sys_enter_openat /comm == "top"/ {
    printf("PID %d is opening: %s\n", pid, str(args->filename));
}'
Attaching 1 probe...
PID 323556 is opening: /proc/uptime
PID 323556 is opening: /proc
PID 323556 is opening: /proc/uptime
PID 323556 is opening: /proc/1/stat
PID 323556 is opening: /proc/1/statm
PID 323556 is opening: /proc/2/stat
PID 323556 is opening: /proc/2/statm
PID 323556 is opening: /proc/7/stat
```

## **分析内核调用路径**
```shell
// 统计 ip_output 函数的调用栈，这里要注意，@[kstack]的key记录的是两条不同的调用链
~ » bpftrace -e 'kprobe:ip_output { @[kstack] = count(); }'
Attaching 1 probe...
^C

@[
    ip_output+5
    __ip_queue_xmit+400
    ip_queue_xmit+25
    __tcp_transmit_skb+2666
    __tcp_send_ack.part.0+198
    tcp_send_ack+32
    __tcp_ack_snd_check+66
    tcp_rcv_established+660
    tcp_v4_do_rcv+362
    tcp_v4_rcv+3671
    ip_protocol_deliver_rcu+55
    ip_local_deliver_finish+138
    ip_local_deliver+115
    ip_sublist_rcv_finish+137
    ip_sublist_rcv+410
    ip_list_rcv+314
    __netif_receive_skb_list_core+639
    netif_receive_skb_list_internal+463
    napi_complete_done+126
    netvsc_poll+1451
    __napi_poll+49
    net_rx_action+680
    handle_softirqs+244
    __irq_exit_rcu+120
    irq_exit_rcu+18
    sysvec_hyperv_callback+180
    asm_sysvec_hyperv_callback+31
    pv_native_safe_halt+15
    arch_cpu_idle+13
    default_idle_call+46
    do_idle+517
    cpu_startup_entry+49
    rest_init+218
    arch_call_rest_init+18
    start_kernel+1241
    x86_64_start_reservations+37
    __pfx_reserve_bios_regions+0
    secondary_startup_64_no_verify+381
]: 3
@[
    ip_output+5
    __ip_queue_xmit+400
    ip_queue_xmit+25
    __tcp_transmit_skb+2666
    __tcp_send_ack.part.0+198
    tcp_send_ack+32
    __tcp_ack_snd_check+66
    tcp_rcv_established+660
    tcp_v4_do_rcv+362
    tcp_v4_rcv+3671
    ip_protocol_deliver_rcu+55
    ip_local_deliver_finish+138
    ip_local_deliver+115
    ip_sublist_rcv_finish+137
    ip_sublist_rcv+410
    ip_list_rcv+314
    __netif_receive_skb_list_core+639
    netif_receive_skb_list_internal+463
    napi_complete_done+126
    netvsc_poll+1451
    __napi_poll+49
    net_rx_action+680
    handle_softirqs+244
    __irq_exit_rcu+120
    irq_exit_rcu+18
    sysvec_hyperv_callback+180
    asm_sysvec_hyperv_callback+31
    pv_native_safe_halt+15
    arch_cpu_idle+13
    default_idle_call+46
    do_idle+517
    cpu_startup_entry+49
    start_secondary+281
    secondary_startup_64_no_verify+381
]: 8
```

![](image1.png)

# 其他说明

- argX与 args的区别：arg0, arg1...用于 kprobe/uprobe，它们是简单的整数参数。而 args是一个结构体，专用于 tracepoint，需要通过 args->字段名来访问其成员。
- 查看 Tracepoint 参数：你可以使用 bpftrace -lv tracepoint:name命令来查看某个 tracepoint 有哪些参数可用。例如，要查看 write系统调用的参数，可以执行：
```shell
~ » bpftrace -lv tracepoint:syscalls:sys_enter_write
tracepoint:syscalls:sys_enter_write
    int __syscall_nr
    unsigned int fd
    const char * buf
    size_t count
```
