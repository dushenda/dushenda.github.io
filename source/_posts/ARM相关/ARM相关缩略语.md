
> 转载：https://zhuanlan.zhihu.com/p/1985799288740136627

ARM 相关缩略语大全（按大类系统整理）  
本文系统性整理 **ARM 架构**中常见缩略语，覆盖 **CPU / Core / AMBA / CoreSight / GIC / 内存系统 / 异构与加速 / 安全 / 虚拟化 / SoC 互连** 等多个层面，适合 **内核、驱动、SoC、性能分析、芯片架构** 方向长期查阅。

  
一、ARM 架构与指令集（Architecture & ISA）

|   |   |   |
|---|---|---|
|缩略语|全称|说明|
|ARM|Advanced RISC Machines|ARM 公司 / 架构统称|
|ISA|Instruction Set Architecture|指令集架构|
|AArch32|ARM Architecture 32-bit|ARMv7 及兼容 32 位执行态|
|AArch64|ARM Architecture 64-bit|ARMv8+ 的 64 位执行态|
|ARMv7-A|ARM Architecture v7 Application|经典 32 位应用处理器架构|
|ARMv8-A|ARM Architecture v8 Application|引入 AArch64|
|ARMv9-A|ARM Architecture v9 Application|引入 SVE2 / CCA|
|Thumb|Thumb Instruction Set|16-bit 压缩指令集|
|Thumb-2|Thumb-2 Instruction Set|16/32 混合指令|
|NEON|Advanced SIMD|ARM 向量 SIMD 扩展|
|SVE|Scalable Vector Extension|可变向量长度 SIMD|
|SVE2|SVE v2|面向通用计算与 DSP|
|SME|Scalable Matrix Extension|ARMv9 矩阵扩展|

  
二、ARM CPU Core 家族（Processor Cores）  
Cortex 系列

|   |   |   |
|---|---|---|
|缩略语|全称|说明|
|Cortex-A|Cortex Application|应用处理器（Linux / Android）|
|Cortex-R|Cortex Real-time|实时处理器|
|Cortex-M|Cortex Microcontroller|微控制器|

  
Cortex-A 常见核心

|   |   |   |
|---|---|---|
|Core|特点|应用|
|A53|小核，低功耗|手机 / 嵌入式|
|A55|A53 后继|big.LITTLE|
|A57|高性能|服务器 / 手机|
|A72|高 IPC|SoC 主核|
|A73|优化能效|移动平台|
|A75|ARMv8.2|DynamIQ|
|A76|高性能单核|旗舰 SoC|
|A78|高能效|新一代移动|
|X1 / X2 / X3|Cortex-X|极致性能|

Neoverse（服务器）

|   |   |
|---|---|
|Core|定位|
|N1|通用服务器|
|N2|ARMv9 数据中心|
|V1|SVE 向量优化|
|E1|边缘高吞吐|

  
三、异常级别与特权模型（Exception Levels）

|   |   |   |
|---|---|---|
|缩略语|全称|说明|
|EL0|Exception Level 0|用户态|
|EL1|Exception Level 1|内核态|
|EL2|Exception Level 2|Hypervisor|
|EL3|Exception Level 3|Secure Monitor|
|PL|Privilege Level|特权等级|
|SPSR|Saved Program Status Register|异常返回寄存器|
|ESR|Exception Syndrome Register|异常原因|

  
四、内存系统（Memory System & MMU）

|      |                                       |           |
| ---- | ------------------------------------- | --------- |
| 缩略语  | 全称                                    | 说明        |
| MMU  | Memory Management Unit                | 内存管理单元    |
| SMMU | System Memory Management Unit         | 系统内存管理单元  |
| MPU  | Memory Protection Unit                | 无 MMU 系统  |
| TLB  | Translation Lookaside Buffer          | 地址翻译缓存    |
| ASID | Address Space ID                      | 进程地址空间标识  |
| VA   | Virtual Address                       | 虚拟地址      |
| PA   | Physical Address                      | 物理地址      |
| IPA  | Intermediate Physical Address         | 虚拟化中间地址   |
| TTBR | Translation Table Base Register       | 页表基址      |
| MAIR | Memory Attribute Indirection Register | 内存属性      |
| ATS  | Address Translation Service           | PCIe 地址翻译 |
|      |                                       |           |

  
五、AMBA 总线与互连（Bus & Interconnect）  
AMBA 总线协议

|          |                                           |                   |
| -------- | ----------------------------------------- | ----------------- |
| 缩略语      | 全称                                        | 说明                |
| AMBA     | Advanced Microcontroller Bus Architecture | ARM 总线体系          |
| AXI      | Advanced eXtensible Interface             | 高性能总线             |
| AXI4     | AXI version 4                             | 主流 SoC 总线         |
| AXI-Lite | AXI Lite                                  | 寄存器访问             |
| AHB      | Advanced High-performance Bus             | 中速总线              |
| APB      | Advanced Peripheral Bus                   | 低速外设              |
| CHI      | Coherent Hub Interface                    | Cache Coherent 总线 |
| ACE      | AXI Coherency Extensions                  | 一致性扩展             |

互连与一致性

|       |                            |          |
| ----- | -------------------------- | -------- |
| 缩略语   | 说明                         |          |
| CCN   | Cache Coherent Network     | 一致性网络    |
| CMN   | Coherent Mesh Network      | Mesh 架构  |
| DVM   | Distributed Virtual Memory | TLB 一致性  |
| Snoop | Cache Snoop                | Cache 探测 |

  
六、GIC 中断控制器（Generic Interrupt Controller）

|   |   |   |
|---|---|---|
|缩略语|全称|说明|
|GIC|Generic Interrupt Controller|ARM 中断控制器|
|GICv2|GIC version 2|ARMv7 常用|
|GICv3|GIC version 3|ARMv8 主流|
|GICv4|GIC version 4|虚拟化直通|
|SGI|Software Generated Interrupt|软件中断|
|PPI|Private Peripheral Interrupt|CPU 私有中断|
|SPI|Shared Peripheral Interrupt|共享中断|
|ITS|Interrupt Translation Service|MSI 映射|
|LPI|Locality-specific Peripheral Interrupt|大规模中断|

  
七、CoreSight 调试与追踪（Debug & Trace）

|   |   |   |
|---|---|---|
|缩略语|全称|说明|
|CoreSight|ARM Debug & Trace 架构|调试体系|
|JTAG|Joint Test Action Group|硬件调试接口|
|SWD|Serial Wire Debug|精简调试|
|ETM|Embedded Trace Macrocell|指令追踪|
|PTM|Program Trace Macrocell|程序追踪|
|STM|System Trace Macrocell|系统事件追踪|
|ITM|Instrumentation Trace Macrocell|软件埋点|
|DWT|Data Watchpoint and Trace|数据监控|
|CTI|Cross Trigger Interface|模块联动|
|TPIU|Trace Port Interface Unit|Trace 输出|

  
八、安全架构（Security）

|   |   |   |
|---|---|---|
|缩略语|全称|说明|
|TrustZone|ARM TrustZone|安全隔离|
|Secure World|安全世界|TEE|
|Normal World|普通世界|REE|
|TEE|Trusted Execution Environment|安全 OS|
|OP-TEE|Open Portable TEE|开源 TEE|
|CCA|Confidential Compute Architecture|ARMv9|
|RME|Realm Management Extension|Realm 隔离|

  
九、虚拟化（Virtualization）

|   |   |
|---|---|
|缩略语|说明|
|HYP|Hypervisor 模式|
|VHE|Virtualization Host Extensions|
|Stage-2|二级地址翻译|
|VGIC|Virtual GIC|
|VCPU|Virtual CPU|
|VMID|Virtual Machine ID|

  
十、SoC / 系统级常见缩略语

|   |   |
|---|---|
|缩略语|说明|
|SoC|System on Chip|
|PMU|Performance Monitor Unit|
|DVFS|Dynamic Voltage and Frequency Scaling|
|QoS|Quality of Service|
|NoC|Network on Chip|
|SRAM|Static RAM|
|DRAM|Dynamic RAM|
|LPDDR|Low Power DDR|

  
十一、软件与生态

|          |                                            |
| -------- | ------------------------------------------ |
| 缩略语      | 说明                                         |
| PSCI     | Power State Coordination Interface         |
| SMC      | Secure Monitor Call                        |
| HVC      | Hypervisor Call                            |
| UEFI     | Unified Extensible Firmware Interface      |
| ACPI     | Advanced Configuration and Power Interface |
| DT / DTS | Device Tree / Source                       |