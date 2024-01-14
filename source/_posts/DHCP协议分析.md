---
title: DHCP协议分析
date: 2024-01-14
tags:
	- 计算机网络
	- 网络协议
---
## 协议流程
## 仿真
### 组网
![](Pastedimage20231215004144.png)

### 路由设置
```console
<Huawei>sys
[Huawei]sysname AR1
[AR1]int g0/0/0
[AR1-GigabitEthernet0/0/0]ip adderss 1.1.1.200 24
[AR1]dhcp enable
[AR1]ip pool ip_pool1
[AR1-ip-pool-ip_pool1]network 1.1.1.0 24
[AR1-ip-pool-ip_pool1]gateway-list 1.1.1.200
[AR1-ip-pool-ip_pool1]lease day 3
[AR1]int g0/0/0
[AR1-GigabitEthernet0/0/0]dhcp select global
```
![](Pastedimage20231215004416.png)


## 抓包
![](Pastedimage20231215005008.png)

### dhcp discover
![](Pastedimage20231215010535.png)

![](Pastedimage20231215010615.png)
### dhcp offer
