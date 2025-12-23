---
title: bpftrace cheatsheet
date: 2025-12-23
tags:
  - cheatsheet
  - linux
---
# 使用方法

```shell
probe[,probe,...] /filter/ { action }
```
探针指定要检测哪些事件，过滤器是可选的，可以根据布尔表达式筛选事件，而操作是要运行的小程序。

例子hello world:
```shell
bpftrace -e 'BEGIN { printf("Hello eBPF!\n"); }'
```
探针是BEGIN，这是一个特殊的探针，会在程序开始时运行（类似于 awk）。没有过滤器。操作是printf()语句。

举个实际例子：
```shell
bpftrace -e 'kretprobe:vfs_read /pid == 181/ { @bytes = hist(retval); }'
```
这段代码使用 kretprobe 来检测 sys_read() 内核函数的返回值。如果进程 ID 为 181，则会将一个特殊的映射变量@bytes填充为一个以 sys_read() 返回值retval为参数的 log2 直方图函数。这样就能生成进程 ID 为 181 的读取大小的直方图。你的应用是否执行了大量 1 字节的读取操作？或许可以对此进行优化。

# 探针类型

这些是相关的探针库。目前支持的类型有：

|Alias|Type|Description|
|---|---|---|
|t|tracepoint|Kernel static instrumentation points|
|U|usdt|User-level statically defined tracing|
|k|kprobe|Kernel dynamic function instrumentation (standard)|
|kr|kretprobe|Kernel dynamic function return instrumentation (standard)|
|f|kfunc|Kernel dynamic function instrumentation (BPF based)|
|fr|kretfunc|Kernel dynamic function return instrumentation (BPF based)|
|u|uprobe|User-level dynamic function instrumentation|
|ur|uretprobe|User-level dynamic function return instrumentation|
|s|software|Kernel software-based events|
|h|hardware|Hardware counter-based instrumentation|
|w|watchpoint|Memory watchpoint events|
|p|profile|Timed sampling across all CPUs|
|i|interval|Timed reporting (from one CPU)|
||iter|Iterator tracing over kernel objects|
||BEGIN|Start of bpftrace|
||END|End of bpftrace|

动态插桩允许在不重启程序的情况下跟踪正在运行的二进制文件中的任何软件函数。但是，它所暴露的函数并非稳定的 API，因为它们可能会随软件版本的变化而改变，从而导致您开发的 bpftrace 工具失效。请尽可能使用静态探针类型，因为它们通常能提供一定的稳定性。

# 变量类型

|Variable|Description|
|---|---|
|@name|global|
|@name[key]|hash|
|@name[tid]|thread-local|
|$name|scratch|

以“@”为前缀的变量使用 BPF 映射，其行为类似于关联数组。它们可以通过以下两种方式之一进行填充：

- 变量赋值：@name = x;
- 函数赋值：@name = hist(x);
内置了多种地图填充函数，可以快速汇总数据。

# 内置变量

|Variable|Description|
|---|---|
|pid|Process ID|
|tid|Thread ID|
|uid|User ID|
|username|Username|
|comm|Process or command name|
|curtask|Current task_struct as a u64|
|nsecs|Current time in nanoseconds|
|elapsed|Time in nanoseconds since bpftrace start|
|kstack|Kernel stack trace|
|ustack|User-level stack trace|
|arg0...argN|Function arguments|
|args|Tracepoint arguments|
|retval|Function return value|
|func|Function name|
|probe|Full probe name|
|$1...$N|Positional parameters|
|cgroup|Default cgroup v2 ID|

# 内置函数

| Function                                | Description                                         |
| --------------------------------------- | --------------------------------------------------- |
| printf("...")                           | Print formatted string                              |
| time("...")                             | Print formatted time                                |
| join(char *arr[])                       | Join array of strings with a space                  |
| str(char *s [, int length])             | Return string from s pointer                        |
| buf(void *p [, int length])             | Return a hexadecimal string from p pointer          |
| strncmp(char *s1, char *s2, int length) | Compares two strings up to length                   |
| sizeof(expression)                      | Returns the size of the expression                  |
| kstack([limit])                         | Kernel stack trace up to limit frames               |
| ustack([limit])                         | User-level stack trace up to limit frames           |
| ksym(void *p)                           | Resolve kernel address to symbol                    |
| usym(void *p)                           | Resolve user-space address to symbol                |
| kaddr(char *name)                       | Resolve kernel symbol name to address               |
| uaddr(char *name)                       | Resolve user-space symbol name to address           |
| ntop([int af,]int\|char[4:16] addr)     | Convert IP address data to text                     |
| reg(char *name)                         | Return register value                               |
| cgroupid(char *path)                    | Return cgroupid for /sys/fs/cgroup/... path         |
| time("...")                             | Print formatted time                                |
| system("...")                           | Run shell command                                   |
| cat(char *filename)                     | Print file content                                  |
| signal(char[] sig \| int sig)           | Send a signal to the current task                   |
| override(u64 rc)                        | Override a kernel function return value             |
| exit()                                  | Exits bpftrace                                      |
| @ = count()                             | Count events                                        |
| @ = sum(x)                              | Sum the value                                       |
| @ = hist(x)                             | Power-of-2 histogram for x                          |
| @ = lhist(x, min, max, step)            | Linear histogram for x                              |
| @ = min(x)                              | Record the minimum value seen                       |
| @ = max(x)                              | Record the maximum value seen                       |
| @ = stats(x)                            | Return the count, average, and total for this value |
| delete(@x[key])                         | Delete the map element                              |
| clear(@x)                               | Delete all keys from the map                        |