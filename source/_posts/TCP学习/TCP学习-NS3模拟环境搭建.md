
> 参考https://github.com/ituring/tcp-book/?tab=readme-ov-file

# 前置工具下载
## virtualBox（最新版本即可）
[https://www.virtualbox.org/](https://www.virtualbox.org/)
```shell
VirtualBox 图形用户界面
版本 7.0.22 r165102 (Qt5.15.2)
Copyright © 2024 Oracle and/or its affiliates
```
## Vagrant（最新版本即可）
[https://developer.hashicorp.com/vagrant](https://developer.hashicorp.com/vagrant)
```shell
PS C:\Users\dushenda> vagrant.exe -v
Vagrant 2.4.9
```
## Powershell
[https://learn.microsoft.com/en-us/powershell/scripting/install/install-powershell-on-windows?view=powershell-7.5](https://learn.microsoft.com/en-us/powershell/scripting/install/install-powershell-on-windows?view=powershell-7.5)
```shell
PS C:\Users\dushenda> pwsh.exe -v
PowerShell 7.1.4
```
## X server
### 下载并安装 VcXsrv：
1. 访问：https://sourceforge.net/projects/vcxsrv/
2. 下载并安装 VcXsrv Windows X Server
### 配置 VcXsrv：
1. 启动 "XLaunch"
2. 选择 "Multiple windows"
3. Display number 设为 "0" 或 "-1"（自动）
4. 选择 "Start no client"
5. 在 "Extra settings" 中勾选 "Disable access control"（重要！）
6. 完成设置，VcXsrv 会在系统托盘运行
### 设置 PowerShell 环境变量
```shell
# 在启动 Vagrant 前设置 DISPLAY
$env:DISPLAY = "localhost:0.0"

# 或者尝试
$env:DISPLAY = "127.0.0.1:0.0"
```
### 修改 Vagrantfile 配置
```shell
Vagrant.configure("2") do |config|
  config.vm.define "guest1" do |guest1|
    guest1.vm.box = "ubuntu/xenial64"
    
    # 关键配置：启用 X11 转发
    guest1.ssh.forward_x11 = true
    guest1.ssh.forward_agent = true
    guest1.ssh.forward_x11_trusted = true
    guest1.ssh.keep_alive = true
    
    # 网络配置
    guest1.vm.network "private_network", type: "dhcp"
  end
end
```

# 安装文件
## 配置环境
```shell
$ git clone https://github.com/ituring/tcp-book.git
$ cd tcp-book/wireshark/vagrant
$ vagrant up
```

## 登录虚拟机
使用以下的命令，ssh连接到Guest操作系统上。在登录消息显示之后，命令行提示会变成`vagrant@guest1:~$`。

注意Windows操作系统，在`MobaXterm`中运行`vagrant ssh guest1`之前，请先运行`cmd.exe`，启动命令行，然后再运行`vagrant ssh guest1`，否则会报错。

```shell
$ vagrant ssh guest1

> Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-139-generic x86_64)
>
> * Documentation:  https://help.ubuntu.com
> * Management:     https://landscape.canonical.com
> * Support:        https://ubuntu.com/advantage
>
> Get cloud support with Ubuntu Advantage Cloud Guest:
> http://www.ubuntu.com/business/services/cloud
>
> 0 packages can be updated.
> 0 updates are security updates.
>
> New release '18.04.1 LTS' available.
> Run 'do-release-upgrade' to upgrade to it.

vagrant@guest1:~$
```

启动Wireshark。
```shell
vagrant@guest1:~$ wireshark
```
![](环境搭建wireshark.png)