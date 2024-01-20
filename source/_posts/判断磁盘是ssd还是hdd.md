---
title: 判断磁盘是ssd还是hdd
date: 2024-01-21
tags:
- 磁盘
- 存储
---
## Linux
### 通过文件系统
`rotational`为1代表可以旋转，为hdd，为0代表不能旋转，为ssd
位置在`/sys/block/sd*/queue/rotational`
```console
[root@dushenda home]# grep ^ /sys/block/sd*/queue/rotational  
/sys/block/sda/queue/rotational:1  
/sys/block/sdb/queue/rotational:1  
/sys/block/sdc/queue/rotational:1
```

### lsblk
```console
[root@dushenda home]# lsblk -o name,rota,VENDOR  
NAME ROTA VENDOR  
sda 1 Msft  
sdb 1 Msft  
sdc 1 Msft
```
`lsblk`可选行信息如下等，通过`lsblk --help`查看
![](判断磁盘是ssd还是hdd/判断磁盘是ssd还是hdd_20240121.png)

### smartctl
该工具需要自行安装Ubuntu和CentOS安装包名称均为`smartmontools`。
```console
[root@dushenda home]# smartctl -a /dev/sdc  
smartctl 7.1 2019-12-30 r5022 [x86_64-linux-5.15.133.1-microsoft-standard-WSL2] (local build)  
Copyright (C) 2002-19, Bruce Allen, Christian Franke, www.smartmontools.org  
  
=== START OF INFORMATION SECTION ===  
Vendor: Msft  
Product: Virtual Disk  
Revision: 1.0  
Compliance: SPC-3  
User Capacity: 1,099,511,627,776 bytes [1.09 TB]  
Logical block size: 512 bytes  
Physical block size: 4096 bytes  
LU is thin provisioned, LBPRZ=0
```

## Windows

### powershell
```console
(base) PS C:\Users\dushenda> Get-PhysicalDisk

Number FriendlyName                  SerialNumber    MediaType CanPool OperationalStatus HealthStatus Usage            Size
------ ------------                  ------------    --------- ------- ----------------- ------------ -----            ----
1      Samsung SSD 860 EVO M.2 500GB S414NB0K722943N SSD       False   OK                Healthy      Auto-Select 465.76 GB
0      WDC WD10SPCX-24HWST1          WD-WXB1AC41L2P1 HDD       False   OK                Healthy      Auto-Select 931.51 GB
```

### GUI
在任务管理器下查看
![](判断磁盘是ssd还是hdd/判断磁盘是ssd还是hdd_20240121%201.png)