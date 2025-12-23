---
title: ftrace技术-静态跟踪（一）
date: 2025-12-23
tags:
  - ftrace
  - linux
---
# 介绍

Ftrace 是一款内部跟踪器，旨在帮助系统开发人员和设计人员了解内核内部的运行情况。它可用于调试或分析用户空间之外发生的延迟和性能问题。

## Debugfs 文件结构

Ftrace 使用 debugfs 文件系统来保存控制文件以及用于显示输出的文件。

当 debugfs 配置到内核中时（选择任何 ftrace 选项都会执行此操作），将创建目录 /sys/kernel/debug。要挂载此目录，可以将其添加到 /etc/fstab 文件中：
```shell
# /etc/fstab
debugfs /sys/kernel/debug debugfs default 0 0
```
或者，也可以在运行时使用以下命令挂载它：
```shell
mount -t debugfs nodev /sys/kernel/debug
```
为了更快地访问该目录，可以创建一个指向它的软链接：
```shell
ln -s /sys/kernel/debug /debug
```
## function跟踪

下表汇总了 `ftrace`最常用的操作命令，适用于直接操作 `/sys/kernel/debug/tracing`目录下的文件。

| 操作目的         | 命令示例                                             | 简要说明                                                  |
| ------------ | ------------------------------------------------ | ----------------------------------------------------- |
| **查看可用跟踪器**​ | `cat available_tracers`                          | 显示系统支持的所有跟踪器 。                                        |
| **设置当前跟踪器**​ | `echo function > current_tracer`                 | 设置跟踪器，如 `function`，`function_graph`，`sched_switch`等 。 |
| **开始/停止跟踪**​ | `echo 1 > tracing_on`  <br>`echo 0 > tracing_on` | 控制跟踪的启动和停止 。                                          |
| **查看跟踪结果**​  | `cat trace`                                      | 显示跟踪缓冲区的内容 。                                          |
| **过滤跟踪函数**​  | `echo sys_* > set_ftrace_filter`                 | 只跟踪函数名匹配 `sys_*`的函数 。                                 |
| **过滤跟踪进程**​  | `echo 1234 > set_ftrace_pid`                     | 只跟踪 PID 为 1234 的进程 。                                  |
| **跟踪特定模块**​  | `echo ':mod:ext4' > set_ftrace_filter`           | 只跟踪 `ext4`模块内的函数 。                                    |
| **启用特定事件**​  | `echo 1 > events/sched/sched_switch/enable`      | 启用 `sched_switch`调度事件 。                               |
| **清空跟踪缓冲区**​ | `echo > trace`                                   | 清空当前跟踪缓冲区以便重新记录 。                                     |

可通过调试文件系统启用函数跟踪器。确保已设置 ftrace_enabled；否则，此跟踪器将不执行任何操作。
```shell
 # sysctl kernel.ftrace_enabled=1
 # echo function > current_tracer
 # echo 1 > tracing_enabled
 # usleep 1
 # echo 0 > tracing_enabled
 # cat trace
# tracer: function
#
#           TASK-PID   CPU#    TIMESTAMP  FUNCTION
#              | |      |          |         |
            bash-4003  [00]   123.638713: finish_task_switch <-schedule
            bash-4003  [00]   123.638714: _spin_unlock_irq <-finish_task_switch
            bash-4003  [00]   123.638714: sub_preempt_count <-_spin_unlock_irq
            bash-4003  [00]   123.638715: hrtick_set <-schedule
            bash-4003  [00]   123.638715: _spin_lock_irqsave <-hrtick_set
            bash-4003  [00]   123.638716: add_preempt_count <-_spin_lock_irqsave
            bash-4003  [00]   123.638716: _spin_unlock_irqrestore <-hrtick_set
            bash-4003  [00]   123.638717: sub_preempt_count <-_spin_unlock_irqrestore
            bash-4003  [00]   123.638717: hrtick_clear <-hrtick_set
            bash-4003  [00]   123.638718: sub_preempt_count <-schedule
            bash-4003  [00]   123.638718: sub_preempt_count <-preempt_schedule
            bash-4003  [00]   123.638719: wait_for_completion <-__stop_machine_run
            bash-4003  [00]   123.638719: wait_for_common <-wait_for_completion
            bash-4003  [00]   123.638720: _spin_lock_irq <-wait_for_common
            bash-4003  [00]   123.638720: add_preempt_count <-_spin_lock_irq
[...]
```

## irqoff跟踪

当中断被关闭时，CPU 无法响应外部事件（如硬件中断），这会导致系统响应延迟。`irqsoff`跟踪器的工作原理是 ：
- **挂钩关键函数**：它在 `local_irq_disable()`和 `local_irq_enable()`等关键函数中插入钩子。
- **记录时间戳**：当中断被关闭时开始计时，并在中断重新开启时停止计时。
- **记录最大延迟**：它会记录下中断关闭的最大时间，并将导致该延迟的调用路径信息保存下来，帮助开发者定位问题根源。

当中断被禁用时，CPU 将无法响应任何其他外部事件（NMI 和 SMI 除外）。这会阻止定时器中断触发，或鼠标中断无法将新的鼠标事件告知内核。其结果是响应时间出现延迟。

`irqsoff` 跟踪器会跟踪中断被禁用的时间。当达到新的最大延迟时，跟踪器会保存达到该延迟点之前的跟踪记录，以便每次达到新的最大值时，都会丢弃旧的跟踪记录并保存新的跟踪记录。
	
要重置最大值，请将 0 回显到 tracing_max_latency 中。以下是一个示例：
```shell
 # echo irqsoff > current_tracer
 # echo 0 > tracing_max_latency
 # echo 1 > tracing_enabled
 # ls -ltr
 [...]
 # echo 0 > tracing_enabled
 # cat latency_trace
# tracer: irqsoff
#
	# irqsoff latency trace v1.1.5 on 4.0.0
# --------------------------------------------------------------------
# latency: 259 us, #4/4, CPU#0 | (M:preempt VP:0, KP:0, SP:0 HP:0 #P:4)
#    -----------------
#    | task: ps-6143 (uid:0 nice:0 policy:0 rt_prio:0)
#    -----------------
#  => started at: __lock_task_sighand
#  => ended at:   _raw_spin_unlock_irqrestore
#
#
#                  _------=> CPU#
#                 / _-----=> irqs-off
#                | / _----=> need-resched
#                || / _---=> hardirq/softirq
#                ||| / _--=> preempt-depth
#                |||| /     delay
#  cmd     pid   ||||| time  |   caller
#     \   /      |||||  \    |   /
    ps-6143    0d.h1    0us+: _raw_spin_lock_irqsave <-__lock_task_sighand
    ps-6143    0d.h.  259us : _raw_spin_unlock_irqrestore <-__lock_task_sighand
```
- **`latency: 259 us`**：这是最重要的信息，表示本次跟踪中发现的最大中断关闭延迟为 259 微秒 。
- **`task: ps-6143`**：发生该延迟时正在执行的进程是 `ps`（PID 为 6143）。
- **`started at`和 `ended at`**：指出了中断关闭开始和结束的函数位置 。
- **`irqs-off`**：`d`表示中断被关闭 。
- **`delay`符号**：输出中可能使用特殊符号（如 `!`表示大于100ms）来直观表示延迟大小。
- **函数调用栈**：在跟踪条目下方，通常会显示导致此次延迟的完整函数调用栈，这对于定位问题代码至关重要 。

请注意，上面的示例中未设置 ftrace_enabled。如果我们设置了 ftrace_enabled，则会得到更大的输出：
```shell
# tracer: irqsoff
#
irqsoff latency trace v1.1.5 on 2.6.26-rc8
--------------------------------------------------------------------
 latency: 50 us, #101/101, CPU#0 | (M:preempt VP:0, KP:0, SP:0 HP:0 #P:2)
    -----------------
    | task: ls-4339 (uid:0 nice:0 policy:0 rt_prio:0)
    -----------------
 => started at: __alloc_pages_internal
 => ended at:   __alloc_pages_internal

#                _------=> CPU#
#               / _-----=> irqs-off
#              | / _----=> need-resched
#              || / _---=> hardirq/softirq
#              ||| / _--=> preempt-depth
#              |||| /
#              |||||     delay
#  cmd     pid ||||| time  |   caller
#     \   /    |||||   \   |   /
      ls-4339  0...1    0us+: get_page_from_freelist (__alloc_pages_internal)
      ls-4339  0d..1    3us : rmqueue_bulk (get_page_from_freelist)
      ls-4339  0d..1    3us : _spin_lock (rmqueue_bulk)
      ls-4339  0d..1    4us : add_preempt_count (_spin_lock)
      ls-4339  0d..2    4us : __rmqueue (rmqueue_bulk)
      ls-4339  0d..2    5us : __rmqueue_smallest (__rmqueue)
      ls-4339  0d..2    5us : __mod_zone_page_state (__rmqueue_smallest)
      ls-4339  0d..2    6us : __rmqueue (rmqueue_bulk)
      ls-4339  0d..2    6us : __rmqueue_smallest (__rmqueue)
      ls-4339  0d..2    7us : __mod_zone_page_state (__rmqueue_smallest)
      ls-4339  0d..2    7us : __rmqueue (rmqueue_bulk)
      ls-4339  0d..2    8us : __rmqueue_smallest (__rmqueue)
[...]
      ls-4339  0d..2   46us : __rmqueue_smallest (__rmqueue)
      ls-4339  0d..2   47us : __mod_zone_page_state (__rmqueue_smallest)
      ls-4339  0d..2   47us : __rmqueue (rmqueue_bulk)
      ls-4339  0d..2   48us : __rmqueue_smallest (__rmqueue)
      ls-4339  0d..2   48us : __mod_zone_page_state (__rmqueue_smallest)
      ls-4339  0d..2   49us : _spin_unlock (rmqueue_bulk)
      ls-4339  0d..2   49us : sub_preempt_count (_spin_unlock)
      ls-4339  0d..1   50us : get_page_from_freelist (__alloc_pages_internal)
      ls-4339  0d..2   51us : trace_hardirqs_on (__alloc_pages_internal)
```
这里我们追踪到了 50 微秒的延迟。但我们也看到了这段时间内调用的所有函数。请注意，启用函数追踪会增加额外的开销。这种开销可能会延长延迟时间。尽管如此，这次追踪仍然提供了一些非常有用的调试信息。

## scdule wakeup跟踪

在实时环境中，了解最高优先级任务从被唤醒到执行所需的唤醒时间至关重要。这也被称为“调度延迟”。

实时环境关注的是最大延迟，也就是事件发生所需的最长延迟，而不是平均延迟。我们可以拥有一个速度非常快的调度器，它可能只是偶尔出现较大的延迟，但这并不适用于实时任务。唤醒跟踪器旨在记录实时任务的最大唤醒延迟。非实时任务不会被记录，因为跟踪器只记录一个最大延迟，而跟踪不可预测的非实时任务会覆盖实时任务的最大延迟。

由于此追踪器仅处理实时任务，因此我们将采用与之前追踪器略有不同的运行方式。我们不会执行“ls”命令，而是在“chrt”命令下运行“sleep 1”命令，这将改变任务的优先级。

```shell
 # echo wakeup > current_tracer
 # echo 0 > tracing_max_latency
 # echo 1 > tracing_enabled
 # chrt -f 5 sleep 1
	 # echo 0 > tracing_enabled
 # cat latency_trace
# tracer: wakeup
#
wakeup latency trace v1.1.5 on 2.6.26-rc8
--------------------------------------------------------------------
 latency: 4 us, #2/2, CPU#1 | (M:preempt VP:0, KP:0, SP:0 HP:0 #P:2)
    -----------------
    | task: sleep-4901 (uid:0 nice:0 policy:1 rt_prio:5)
    -----------------

#                _------=> CPU#
#               / _-----=> irqs-off
#              | / _----=> need-resched
#              || / _---=> hardirq/softirq
#              ||| / _--=> preempt-depth
#              |||| /
#              |||||     delay
#  cmd     pid ||||| time  |   caller
#     \   /    |||||   \   |   /
  <idle>-0     1d.h4    0us+: try_to_wake_up (wake_up_process)
  <idle>-0     1d..4    4us : schedule (cpu_idle)
```
在空闲系统上运行此程序，我们发现任务切换仅耗时 4 微秒。请注意，由于调度中的跟踪标记位于实际“切换”之前，因此当记录的任务即将调度时，我们会停止跟踪。如果我们在调度程序末尾添加新的标记，则此情况可能会发生变化。

请注意，记录的任务是“sleep”，进程 ID 为 4901，其 rt_prio 为 5。此优先级为用户空间优先级，而非内核内部优先级。策略为：SCHED_FIFO 为 1，SCHED_RR 为 2。

使用 chrt -r 5 和 ftrace_enabled set 执行相同的操作。
```shell
# tracer: wakeup
#
wakeup latency trace v1.1.5 on 2.6.26-rc8
--------------------------------------------------------------------
 latency: 50 us, #60/60, CPU#1 | (M:preempt VP:0, KP:0, SP:0 HP:0 #P:2)
    -----------------
    | task: sleep-4068 (uid:0 nice:0 policy:2 rt_prio:5)
    -----------------

#                _------=> CPU#
#               / _-----=> irqs-off
#              | / _----=> need-resched
#              || / _---=> hardirq/softirq
#              ||| / _--=> preempt-depth
#              |||| /
#              |||||     delay
#  cmd     pid ||||| time  |   caller
#     \   /    |||||   \   |   /
ksoftirq-7     1d.H3    0us : try_to_wake_up (wake_up_process)
ksoftirq-7     1d.H4    1us : sub_preempt_count (marker_probe_cb)
ksoftirq-7     1d.H3    2us : check_preempt_wakeup (try_to_wake_up)
ksoftirq-7     1d.H3    3us : update_curr (check_preempt_wakeup)
ksoftirq-7     1d.H3    4us : calc_delta_mine (update_curr)
ksoftirq-7     1d.H3    5us : __resched_task (check_preempt_wakeup)
ksoftirq-7     1d.H3    6us : task_wake_up_rt (try_to_wake_up)
ksoftirq-7     1d.H3    7us : _spin_unlock_irqrestore (try_to_wake_up)
[...]
ksoftirq-7     1d.H2   17us : irq_exit (smp_apic_timer_interrupt)
ksoftirq-7     1d.H2   18us : sub_preempt_count (irq_exit)
ksoftirq-7     1d.s3   19us : sub_preempt_count (irq_exit)
ksoftirq-7     1..s2   20us : rcu_process_callbacks (__do_softirq)
[...]
ksoftirq-7     1..s2   26us : __rcu_process_callbacks (rcu_process_callbacks)
ksoftirq-7     1d.s2   27us : _local_bh_enable (__do_softirq)
ksoftirq-7     1d.s2   28us : sub_preempt_count (_local_bh_enable)
ksoftirq-7     1.N.3   29us : sub_preempt_count (ksoftirqd)
ksoftirq-7     1.N.2   30us : _cond_resched (ksoftirqd)
ksoftirq-7     1.N.2   31us : __cond_resched (_cond_resched)
ksoftirq-7     1.N.2   32us : add_preempt_count (__cond_resched)
ksoftirq-7     1.N.2   33us : schedule (__cond_resched)
ksoftirq-7     1.N.2   33us : add_preempt_count (schedule)
ksoftirq-7     1.N.3   34us : hrtick_clear (schedule)
ksoftirq-7     1dN.3   35us : _spin_lock (schedule)
ksoftirq-7     1dN.3   36us : add_preempt_count (_spin_lock)
ksoftirq-7     1d..4   37us : put_prev_task_fair (schedule)
ksoftirq-7     1d..4   38us : update_curr (put_prev_task_fair)
[...]
ksoftirq-7     1d..5   47us : _spin_trylock (tracing_record_cmdline)
ksoftirq-7     1d..5   48us : add_preempt_count (_spin_trylock)
ksoftirq-7     1d..6   49us : _spin_unlock (tracing_record_cmdline)
ksoftirq-7     1d..6   49us : sub_preempt_count (_spin_unlock)
ksoftirq-7     1d..4   50us : schedule (__cond_resched)
```
中断发生在 ksoftirqd 运行期间。该任务运行在 SCHED_OTHER 调度级别。为什么我们没有提前看到“N”被置位？这可能是 x86_32 架构和 4K 堆栈的一个无害 bug。在配置了 4K 堆栈的 x86_32 架构上，中断和软中断使用它们自己的堆栈运行。一些信息保存在任务堆栈的顶部（need_resched 和 preempt_count 都存储在那里）。NEED_RESCHED 位的设置直接作用于任务堆栈，但读取 NEED_RESCHED 位是通过查看当前堆栈完成的，在本例中，当前堆栈是硬中断的堆栈。这隐藏了 NEED_RESCHED 已被设置的事实。直到我们切换回任务分配的堆栈，才能看到“N”。
