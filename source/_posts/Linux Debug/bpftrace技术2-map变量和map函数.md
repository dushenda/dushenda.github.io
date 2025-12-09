---
title: bpftrace技术2-map变量和map函数
date: 2025-12-10
tags:
  - linux
  - bpftrace
---
# Map 的核心概念与用途 
在 bpftrace 中，符号 `@`用于定义和操作 **Map 变量**（或称映射变量）。这是 bpftrace 实现高效数据聚合的核心机制，你可以把它理解为一个在内核中运行的、功能强大的**迷你数据库**

Map 的主要作用是在不同的事件探针（probe）之间**存储、共享和聚合数据**。当某个事件触发时，你可以将数据记录到 Map 中；当另一个相关事件触发时，再从中读取或更新数据。这使得实现复杂的追踪逻辑成为可能。

| 特性         | 说明                                                                                              |
| ---------- | ----------------------------------------------------------------------------------------------- |
| **数据持久化**​ | Map 中的数据在多个探针事件之间保持存在，不像普通变量那样每次事件触发后就被重置。（全局变量）                                                |
| **键值对结构**​ | Map 使用键（key）来索引值（value），格式为 `@map_name[key] = value`。键可以是单一值，也可以是多个值的组合（如 `@a[pid, comm]`）。kv结构 |
| **自动输出**​  | 默认情况下，当 bpftrace 程序退出时（例如你按下 `Ctrl-C`），所有非空的 Map 内容会自动打印到屏幕上。                                   |

#  Map 的常见操作与示例
| 操作/函数                               | 说明                             | 示例                                                          |
| ----------------------------------- | ------------------------------ | ----------------------------------------------------------- |
| **赋值**​                             | 直接给 Map 赋值。                    | `@start_time[tid] = nsecs`(记录线程的开始时间)                       |
| **计数 (`count()`)**​                 | 统计事件发生的次数。                     | `@syscall_count[comm] = count()`(统计每个进程名的系统调用次数)            |
| **求和 (`sum()`)**​                   | 对数值进行累加。                       | `@total_bytes[pid] = sum(args->ret)`(累计每个进程读取的总字节数)         |
| **统计 (`avg()`, `min()`, `max()`)**​ | 计算平均值、最小值、最大值。                 | `@response_time = avg($latency)`                            |
| **直方图 (`hist()`)**​                 | **非常实用**，生成2的幂次方的直方图，直观展示数据分布。 | `@latency_ns = hist(nsecs - @start[tid])`(可视化读操作的延迟分布)      |
| **线性直方图 (`lhist()`)**​              | 生成自定义区间的线性直方图。                 | `@read_sizes = lhist(args->ret, 0, 10000, 1000)`(统计读取大小的分布) |
| **数据清理 (`delete()`)**​              | 从 Map 中删除特定的键值对，防止内存无限增长。      | `delete(@start_time[tid])`(在处理完一个事件后清理对应的开始时间)              |

#  与其他变量的区别
| 变量类型        | 前缀  | 作用域与用途                                                          |
| ----------- | --- | --------------------------------------------------------------- |
| **Map 变量**​ | `@` | **全局**。用于在探针之间持久化存储和聚合数据。                                       |
| **内置变量**​   | 无   | **只读**。提供事件上下文信息，如 `pid`(进程ID)、`comm`(命令名)、`retval`(函数返回值)等。    |
| **暂存变量**​   | `$` | **局部临时**。用于单次探针触发过程中的中间计算，例如 `$duration = nsecs - @start[tid]`。 |

# map函数参考表
| 函数原型                                            | 核心作用与参数说明                                              | 典型应用场景                                          |
| ----------------------------------------------- | ------------------------------------------------------ | ----------------------------------------------- |
| **`count()`**​                                  | **计数**。统计事件被触发的次数。无参数。                                 | 统计系统调用次数、函数调用次数等。                               |
| **`sum(int n)`**​                               | **求和**。对参数 `n`的值进行累加。                                  | 计算总的字节读写量、总耗时等。                                 |
| **`avg(int n)`**​                               | **求平均值**。计算参数 `n`的平均值。                                 | 计算平均延迟、平均数据包大小等。                                |
| **`min(int n)`**​                               | **求最小值**。记录参数 `n`的最小值。                                 | 追踪最小延迟、最小数据块大小。                                 |
| **`max(int n)`**​                               | **求最大值**。记录参数 `n`的最大值。                                 | 追踪最大延迟、最大数据块大小。                                 |
| **`stats(int n)`**​                             | **统计摘要**。返回参数 `n`的计数、平均值、总和、最小值、最大值。                   | 获取一个指标的全面统计信息。                                  |
| **`hist(int n)`**​                              | **对数直方图**。按2的幂次方区间（如 `[4-8)`, `[8-16)`）展示参数 `n`的分布。    | 直观展示延迟、数据大小的分布情况，易于发现模式。                        |
| **`lhist(int n, int min, int max, int step)`**​ | **线性直方图**。在指定的线性区间（`min`到`max`，步长为`step`）内展示参数 `n`的分布。 | 当需要自定义固定区间进行分析时使用。                              |
| **`delete(@m[key])`**​                          | **删除键值对**。从 Map `@m`中删除指定的 `key`及其对应的值。                | 清理临时数据，防止 Map 无限增长，常用于配对探针（如 kprobe/kretprobe）。 |
| **`clear(@m)`**​                                | **清空 Map**。清除 Map `@m`中的所有键值对。                         | 在定时器（如 `interval`）中定期重置统计。                      |
| **`zero(@m)`**​                                 | **归零 Map**。将 Map `@m`中所有键的值重置为 0。                      | 重置计数或求和等数据，但保留键的结构。                             |

# 例子


## count()- 统计系统调用次数
统计每个进程调用的系统调用次数

`bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @[comm] = count(); }'`

输出
```shell
@count[bash]: 15
@count[sshd]: 28
@count[snmpd]: 102
```

- @\[comm\]：以进程名 comm为键（key）。
- count()：每次事件触发，对应键的值加 1

## sum(int n)- 计算读取的总字节数
累计所有进程通过 read 系统调用成功读取的字节数

`bpftrace -e 'tracepoint:syscalls:sys_exit_read /args->ret > 0/ { @bytes = sum(args->ret); }'`

输出
```shell
@bytes: 1048576
```

- /args->ret > 0/：过滤器，只处理成功读取（返回值大于0）的情况。
- sum(args->ret)：对返回值（读取的字节数）进行累加

## avg(int n)- 计算平均读取大小
计算每次 `read`系统调用成功读取的平均字节数。

`bpftrace -e 'tracepoint:syscalls:sys_exit_read /args->ret > 0/ { @avg_size = avg(args->ret); }'`

输出
```shell
@avg_size: 512
```

- /args->ret > 0/：过滤器，只处理成功读取（返回值大于0）的情况。
- avg(args->ret)：对返回值（读取的字节数）求平均

## stats(int n)- 获取完整的统计摘要
对 `read`系统调用的返回值进行全面的统计。

`bpftrace -e 'tracepoint:syscalls:sys_exit_read /args->ret > 0/ { @s = stats(args->ret); }'`

输出
```shell
@s: count 100, average 4096, total 409600, min 1, max 8192
```

- /args->ret > 0/：过滤器，只处理成功读取（返回值大于0）的情况。
- stats(args->ret)：求调用次数，对返回值（读取的字节数）求平均，求总数，求最大最小值

## hist(int n)- 分析读取字节数的对数分布
显示 `read`系统调用返回值的分布，区间按2的幂次方划分。

`bpftrace -e 'tracepoint:syscalls:sys_exit_read { @bytes = hist(args->ret); }'`

输出
```shell
@bytes:
[0, 1]                12 |@@@@@@@@@@@@@@@@@@@@                                |
[2, 4)                18 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                     |
[4, 8)                 0 |                                                    |
...
[128, 256)             1 |@
```

- hist(args->ret)：分为2^x ~2^y 大小的桶，统计每个桶的里面的数目个数

## lhist(int n, int min, int max, int step)- 分析读取字节数的线性分布
使用线性直方图统计 `read`返回值，范围从0到2000，步长为200。

`bpftrace -e 'tracepoint:syscalls:sys_exit_read { @bytes = lhist(args->ret, 0, 2000, 200); }'`

输出
```shell
@bytes:
(..., 0)                0 |                                                    |
[0, 200)              66 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
[200, 400)             2 |@                                                   |
...
[2000, ...)            39 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                      |
```

- hist(args->ret)：分为0-200区间桶，统计每个桶的里面的数目个数

## delete(@m\[key\])- 清理临时数据
在计算函数耗时后，删除用于存储开始时间的临时键，避免 Map 无限增长。
```c
bpftrace -e 'kprobe:vfs_read 
			{ @start[tid] = nsecs; } 
			   kretprobe:vfs_read /@start[tid]/ 
			   { $duration_ns = nsecs - @start[tid]; 
				 @us = hist($duration_ns); 
			     delete(@start[tid]); }'
```

- 此函数通常用于配对操作的探针（如 `kprobe`和 `kretprobe`），在操作完成后及时清理资源

## clear(@m)和 zero(@m)- 重置 Map
每5秒打印并清空一次系统调用计数
```c
bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @[comm] = count(); } interval:s:5 { print(@); clear(@); }'
```
输出
```shell
@[Relay(89295)]: 3
@[mini_init]: 4
@[gmain]: 5
@[Relay(909)]: 6
@[Relay(893)]: 6
@[systemd-udevd]: 37
@[chronyd]: 40
@[bpftrace]: 118
```

- `clear(@m)`会删除 Map 中的所有键值对。而 `zero(@m)`则将所有键的值重置为0，但保留键的结构
```c
bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @[comm] = count(); } interval:s:5 { print(@); zero(@); }'
```
输出
```shell
@[gmain]: 5
@[Relay(893)]: 6
@[Relay(909)]: 6
@[Relay(89295)]: 9
@[mini_init]: 12
@[chronyd]: 44
@[init]: 103
@[bpftrace]: 114
@[GnsPortTracker]: 242
@[top]: 350
@[grep]: 664
@[sh]: 923
@[Xwayland]: 1978
@[ps]: 2778
@[libuv-worker]: 4028
@[ls]: 4256
@[node]: 4722
@[mini_init]: 0   <---------主要看这个，明显没有调用，但是结构被保留下来了
@[Relay(109)]: 3
@[Relay(909)]: 6
@[Relay(89295)]: 6
@[Relay(893)]: 6
```
# map特点

- **自动打印**：默认情况下，当 bpftrace 程序终止时（例如用户按下 `Ctrl-C`），所有非空的 Map 变量会自动打印出来。
- **过滤条件**：使用 `/<filter>/`可以设置条件，只有满足条件时才会执行后面的动作，这能有效提升脚本效率和输出内容的针对性
- **结合变量**：Map 函数常与内置变量（如 `comm`, `pid`, `nsecs`）或临时变量（以 `$`开头）结合使用，以实现复杂的追踪逻辑