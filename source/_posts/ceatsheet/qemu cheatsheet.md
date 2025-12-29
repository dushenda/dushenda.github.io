---
title: qemu cheatsheet
date: 2025-12-29
tags:
  - qemu
  - datasheet
---

# qemu cheatsheet

## 一、基本虚拟机启动

### 1. 基本启动虚拟机

启动x86_64虚拟机加载镜像（基本用法）。

```
qemu-system-x86_64 -drive file=disk.img,if=virtio
```

### 2. 无图形界面启动

无头模式启动VM（-nographic）。

```
qemu-system-x86_64 -nographic -drive file=disk.img
```

### 3. 指定CPU核心数

指定4核CPU启动（-smp 4）。

```
qemu-system-x86_64 -smp 4 -drive file=disk.img
```

### 4. 指定内存大小

分配2GB内存启动（-m 2G）。

```
qemu-system-x86_64 -m 2G -drive file=disk.img
```

### 5. 启用KVM加速

启用KVM硬件加速（-enable-kvm）。

```
qemu-system-x86_64 -enable-kvm -drive file=disk.img
```

### 6. 指定机器类型

指定q35机器类型启动（-machine q35）。

```
qemu-system-x86_64 -machine q35 -drive file=disk.img
```

### 7. 引导顺序指定

指定从硬盘引导（-boot order=c）。

```
qemu-system-x86_64 -boot order=c -drive file=disk.img
```

### 8. CD-ROM镜像启动

附加CD-ROM镜像启动（-cdrom）。

```
qemu-system-x86_64 -cdrom iso.img -drive file=disk.img
```

### 9. USB设备模拟

模拟USB设备启动（-usb）。

```
qemu-system-x86_64 -usb -drive file=disk.img
```

### 10. 守护进程模式启动

后台守护进程启动VM（-daemonize）。

```
qemu-system-x86_64 -daemonize -drive file=disk.img
```

## 二、设备配置与直通

### 11. VirtIO磁盘配置

使用VirtIO SCSI磁盘（-drive if=virtio）。

```
qemu-system-x86_64 -drive file=disk.img,if=virtio,format=qcow2
```

### 12. PCI设备直通

直通主机PCI设备到VM（-device vfio-pci）。

```
qemu-system-x86_64 -device vfio-pci,host=0000:01:00.0 -drive file=disk.img
```

### 13. USB主机设备直通

直通USB设备（-usbdevice host）。

```
qemu-system-x86_64 -usb -device usb-host,vendorid=0x1234,productid=0x5678 -drive file=disk.img
```

### 14. 网络设备配置

配置用户模式网络（-netdev user）。

```
qemu-system-x86_64 -netdev user,id=mynet -device virtio-net,netdev=mynet -drive file=disk.img
```

### 15. 桥接网络配置

桥接主机网络接口（-netdev bridge）。

```
qemu-system-x86_64 -netdev bridge,id=mynet,br=br0 -device virtio-net,netdev=mynet -drive file=disk.img
```

### 16. TAP网络配置

使用TAP接口网络（-netdev tap）。

```
qemu-system-x86_64 -netdev tap,id=mynet,ifname=tap0 -device virtio-net,netdev=mynet -drive file=disk.img
```

### 17. VFIO GPU直通

直通GPU设备并配置VGA（-vfio-pci -vga none）。

```
qemu-system-x86_64 -device vfio-pci,host=0000:01:00.0 -vga none -drive file=disk.img
```

### 18. 多磁盘附加

附加多个磁盘设备（-drive 多行）。

```
qemu-system-x86_64 -drive file=disk1.img,if=virtio -drive file=disk2.img,if=virtio -drive file=disk.img
```

### 19. 串口设备配置

配置串口重定向到stdio（-serial stdio）。

```
qemu-system-x86_64 -serial stdio -drive file=disk.img
```

### 20. 块设备缓存模式

指定磁盘缓存模式为none（cache=none）。

```
qemu-system-x86_64 -drive file=disk.img,cache=none,if=virtio
```

## 三、网络与迁移

### 21. 多网卡配置

配置多个网卡接口（-netdev 多行）。

```
qemu-system-x86_64 -netdev user,id=net1 -device virtio-net,netdev=net1 -netdev user,id=net2 -device virtio-net,netdev=net2 -drive file=disk.img
```

### 22. VLAN网络隔离

使用VLAN隔离网卡（-net vlan=1）。

```
qemu-system-x86_64 -net user,vlan=1 -drive file=disk.img
```

### 23. 实时迁移VM

迁移VM到远程主机（-incoming tcp）。

```
# 源: qemu-system-x86_64 -drive file=disk.img -incoming tcp:0:4444 (目标先启动)# 目标: migrate -d tcp:remote-host:4444
```

### 24. 迁移压缩优化

启用迁移压缩加速（-comp xbzrle）。

```
qemu-system-x86_64 -drive file=disk.img -enable-kvm -comp xbzrle
```

### 25. 多线程迁移

启用多线程迁移（-thread multi）。

```
qemu-system-x86_64 -drive file=disk.img -thread multi
```

### 26. 网络端口转发

转发主机端口到VM（-net user,hostfwd）。

```
qemu-system-x86_64 -net user,hostfwd=tcp::8080-:80 -drive file=disk.img
```

### 27. SMB网络共享

配置SMB网络共享到VM（-net user,smb）。

```
qemu-system-x86_64 -net user,smb=/path/to/share -drive file=disk.img
```

### 28. VDE网络配置

使用VDE交换机网络（-net vde）。

```
qemu-system-x86_64 -net vde,sock=/tmp/vde.sock -drive file=disk.img
```

### 29. 网络限速配置

限制网络带宽（-net user,throttle）。

```
qemu-system-x86_64 -net user,throttle=100 -drive file=disk.img
```

### 30. 迁移快照链

迁移时处理快照链（-drive snapshot=on）。

```
qemu-system-x86_64 -drive file=disk.img,snapshot=on
```

## 四、图形与输入输出

### 31. VNC图形界面

启用VNC远程桌面（-vnc）。

```
qemu-system-x86_64 -vnc :0 -drive file=disk.img
```

### 32. SPICE图形加速

使用SPICE协议图形（-spice）。

```
qemu-system-x86_64 -spice port=5900,disable-ticketing=on -drive file=disk.img
```

### 33. SDL图形界面

使用SDL库图形输出（-sdl）。

```
qemu-system-x86_64 -sdl -drive file=disk.img
```

### 34. 键盘布局指定

指定键盘布局为en-us（-k en-us）。

```
qemu-system-x86_64 -k en-us -drive file=disk.img
```

### 35. 鼠标输入配置

配置绝对鼠标输入（-usbdevice tablet）。

```
qemu-system-x86_64 -usb -device usb-tablet -drive file=disk.img
```

### 36. 图形分辨率设置

设置VGA分辨率（-vga std -full-screen）。

```
qemu-system-x86_64 -vga std -full-screen -drive file=disk.img
```

### 37. 多监视器配置

配置多个图形输出（-device virtio-gpu）。

```
qemu-system-x86_64 -device virtio-gpu -drive file=disk.img
```

### 38. 音频设备模拟

模拟音频设备输出（-soundhw ac97）。

```
qemu-system-x86_64 -soundhw ac97 -drive file=disk.img
```

### 39. 剪贴板共享

启用SPICE剪贴板共享（-spice clipboard=on）。

```
qemu-system-x86_64 -spice port=5900,clipboard=on -drive file=disk.img
```

### 40. 图形加速Virgl

启用Virgl 3D加速（-device virtio-gpu-virgl）。

```
qemu-system-x86_64 -device virtio-gpu-virgl -drive file=disk.img
```

## 五、调试与监视

### 41. GDB调试连接

启用GDB调试服务器（-s -S）。

```
qemu-system-x86_64 -s -S -drive file=disk.img
```

### 42. 监视器控制台

启用QMP监视器（-monitor stdio）。

```
qemu-system-x86_64 -monitor stdio -drive file=disk.img
```

### 43. QMP套接字监视

使用UNIX套接字QMP监视（-qmp unix）。

```
qemu-system-x86_64 -qmp unix:/tmp/qmp.sock,server,nowait -drive file=disk.img
```

### 44. 性能跟踪

启用性能跟踪日志（-d cpu）。

```
qemu-system-x86_64 -d cpu -drive file=disk.img
```

### 45. 日志文件输出

输出日志到文件（-D logfile）。

```
qemu-system-x86_64 -D qemu.log -drive file=disk.img
```

### 46. 内核调试模式

调试Linux内核（-kernel -initrd -append）。

```
qemu-system-x86_64 -kernel bzImage -initrd initrd.img -append "console=ttyS0" -drive file=disk.img
```

### 47. 内存转储

转储VM内存到文件（-mem-path）。

```
qemu-system-x86_64 -mem-path /tmp/mem.dump -drive file=disk.img
```

### 48. 事件监视脚本

使用脚本监视事件（-trace events=events.txt）。

```
qemu-system-x86_64 -trace events=events.txt -drive file=disk.img
```

### 49. CPU热插拔监视

启用CPU热插拔并监视（-smp 1,maxcpus=4）。

```
qemu-system-x86_64 -smp 1,maxcpus=4 -drive file=disk.img
```

### 50. 调试符号加载

加载调试符号启动（-gdb tcp::1234）。

```
qemu-system-x86_64 -gdb tcp::1234 -drive file=disk.img
```

> [常见QEMU语句](https://mp.weixin.qq.com/s?__biz=Mzg3MDQ4MjA2Mg==&mid=2247487183&idx=1&sn=984b50f454c454d3233bb40eaf613691&chksm=cf5f692e87f39bdcd3f8f4569f6d5dcb891c920298d3ecf95b14595cf419a5114ecdbbaeea6a&mpshare=1&scene=24&srcid=1218C6PzHMIsmMxnl4EutIig&sharer_shareinfo=2ff91c68bc0949e2282bcdf00a7bacd8&sharer_shareinfo_first=2ff91c68bc0949e2282bcdf00a7bacd8#rd)