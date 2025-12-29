---
title: virsh cheatsheet
date: 2025-12-29
tags:
  - virsh
  - cheatsheet
---
# 常用命令

## 启动与关闭

|操作|命令|说明|
|---|---|---|
|**启动**|`virsh start <虚拟机名>`|启动一个处于关闭状态的虚拟机|
|**正常关机**|`virsh shutdown <虚拟机名>`|模拟按下电源键，优雅关机（推荐）|
|**强制关闭**|`virsh destroy <虚拟机名>`|相当于直接拔掉电源，仅在卡死时使用|
|**重启**|`virsh reboot <虚拟机名>`|重启虚拟机|
|**强制重置**|`virsh reset <虚拟机名>`|强制硬重启|

##  挂起与恢复

| 操作     | 命令                     | 说明              |
| ------ | ---------------------- | --------------- |
| **挂起** | `virsh suspend <虚拟机名>` | 暂停虚拟机，内存数据保留在主机 |
| **恢复** | `virsh resume <虚拟机名>`  | 从挂起状态恢复运行       |

##  自启动配置

- 设置开机自启： `virsh autostart <虚拟机名>`
- 取消开机自启： `virsh autostart --disable <虚拟机名>`
- 查看自启列表： `virsh autostart --list`

# 虚拟机信息查询

##  基本状态查询

```shell
# 列出所有虚拟机（运行中+已关闭）
virsh list --all

# 查看虚拟机详细信息（UUID、OS类型、最大内存等）
virsh dominfo <虚拟机名>

# 仅查看虚拟机当前状态（running/shut off/paused）
virsh domstate <虚拟机名>
```
## 配置与连接信息

```shell
# 导出完整的XML配置（非常重要！）
virsh dumpxml <虚拟机名>

# 搜索特定配置（例如查看磁盘路径）
virsh dumpxml <虚拟机名> | grep -A 5 "<source file"

# 查看VNC远程连接端口
virsh vncdisplay <虚拟机名>
```
## 资源监控

```shell
# 查看CPU使用统计
virsh cpu-stats <虚拟机名>

# 查看内存使用详情
virsh dommemstat <虚拟机名>

# 查看挂载的硬盘列表
virsh domblklist <虚拟机名>

# 查看虚拟网卡列表及MAC地址
virsh domiflist <虚拟机名>
```

# 虚拟机配置管理

## XML配置编辑（核心）

KVM的所有配置都存储在XML文件中：

**编辑配置（推荐）**：

```shell
virsh edit <虚拟机名>
```

> 使用此命令会自动检查XML语法，保存生效。不要直接去改 /etc/libvirt/qemu/ 下的文件，容易出错。

**备份配置**：

```shell
virsh dumpxml <虚拟机名> > vm-config.xml
```

**从XML恢复/定义**：

```shell
virsh define vm-config.xml   # 注册虚拟机（不启动）
```

## 删除与重命名

**取消定义（删除配置）**：
```shell
virsh undefine <虚拟机名>
```
> 此命令只删除配置文件，不会删除虚拟磁盘文件！
**彻底删除（配置+磁盘）**：
```shell
virsh undefine --remove-all-storage <虚拟机名>
```
> 高危操作： 请务必确认磁盘数据已不再需要，执行后不可恢复！

**重命名虚拟机**：  
```shell
virsh domrename <旧名称> <新名称>
```