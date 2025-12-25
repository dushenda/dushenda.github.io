---
title: Mellanox cheatsheet
date: 2025-12-25
tags:
  - Mellanox
  - cheatsheet
---
# 状态检查与配置

```shell
# IB设备的状态
ibstat

# IB设备的细节
ibv_devinfo

# 列出所有的IB设备
ibdevices

# 检查IB设备端口状态
iblinkinfo

# 查看IB地址配置
ibaddr

# 显示IB端口状态细节信息
ibportstate

# 显示IB端口统计信息
ibportinfo
```

# 配置和管理

```shell
# 配置IB设备，显示IB和以太设备的映射关系
ibdev2netdev
ibdev2netdev -v

# 设置IB设备到一个特定的驱动
ibdriver

# 更新IB设备固件
mstflint -d <device> qflash <firmware_file.bin>

# 列出IB设备所有可获取的路径
ibpathln

# 节点间所有的IB连接
ibnetdiscover

# 查询固件参数
mlxconfig -d ${PCI_ID} q

# 设置端1为以太模式，端口2为IB模式
mlxconfig -d ${PCI_ID} set LINK_TYPE_P1=2 LINK_TYPE_P2=1

# 配置设置的SRIOV开启和个数
mlxconfig -d ${PCI_ID} set SRIOV_EN=1 NUM_OF_VFS=8

# 配置链路聚合模式，队列亲和性
mlxconfig -d ${PCI_ID} s LAG_RESOURCE_ALLOCATION=0

# 允许压缩CQE（小包调优，其他可能性能劣化）
mlxconfig -d ${PCI_ID} s CQE_COMPRESSION=1
```

# 固件更新

```shell
# 更新NIC卡固件（非OEM卡）
mlxfwmanager

# 常用的固件烧录命令（卡和包PSID要一致）
mstflint -d "${PCI_ID}" -i "${NEW_FIRMWARE_BIN}" burn
flint -d "${PCI_ID}" -i "${NEW_FIRMWARE_BIN}" burn

# 查询固件的全量信息
mstflint -d ${PCI_ID} query full

# 忽略PSID强制烧入固件
mstflint -d "${PCI_ID}" -i "${NEW_FIRMWARE_BIN}" -allow_psid_change burn


```

# 性能测试和监控

```shell
# 测试两设备见IB性能（时延和带宽）
ib_send_bw <device> <destination>

# 测试IB RDMA性能
ib_send_lat <device> <destination>

# 监控IB设备流量
ibstat -i <device> -s

# 监控IB设备错误
iberrdump

# mlnx调优
mlnx_tune -h
mlnx_tune -v
mlnx_tune -q
mlnx_tune -p HIGH_THROUGHPUT
```

# 配置RDMA

```shell
# 检查RDMA配置
rdma link

# 设置IB设备RDMA用途
rdma_resolve_route

# 测试IB的RDMA性能
ib_send_lat -D <device> <destination>

# 检查RDMA安全和权限
rdma_sec

# 配置RDMA加密和安全
rdma_auth
```

# 定位问题

```shell
# 检查多节点的IB配置
ibnetdiscover

# dmesg显示ib 相关打印
dmesg | grep -i ib

# 查看IB日志
tail -f /var/log/ib_log

# 检查IB端口状态
ibportstate
```

# 配置和管理IB子网

```shell
# 显示IB网络信息
ibnetdiscover -v

# 设置IB网络管理
sm_profile

# 检查IB网口状态
ibstat -s
```

# Mellanox管理命令

```shell
# 网卡状态查询
mst status -v

# 开启mst服务
mst start

# 配置RDMA的Mellanox设备
rdma_configure_device

# 安装配置Mellanox管理软件
mstflint -d <device> qflash <firmware_file.bin>

# 获取网卡qos配置
mlnx_qos -i eth2

# 设置网卡端口的0~7 PFC优先级
mlnx_qos -i eth2 -f 0,0,0,1,0,0,0,0
```

# 配置IB QoS

```shell
# Set quality of service policy for InfiniBand
# 设置IB qos
ib_qos_policy

# 配置IB流优先级
ib_flowcontrol

# 检查QoS配置和流优先级
ib_qos_show
```

## 参考
- [https://gist.github.com/githubfoam/da75951b97e9aec21dcebadf68a6a360](https://gist.github.com/githubfoam/da75951b97e9aec21dcebadf68a6a360)
- 


