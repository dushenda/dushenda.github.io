---
title: audit审计工具使用
date: 2024-04-11
tags: 
	- RHEL系列
	- 审计
	- 监控
	- audit
---

## audit用途
监控文件、命令、网络等，生成监控报告。

## 安装启动audit
安装audit工具
```console
yum install audit
```
配置了 `auditd` 后，启动服务以收集 审计信息，并将它存储在日志文件中。以 root 用户身份运行以下命令来启动 `auditd` ：

```console
service auditd start
```

将 `auditd` 配置为在引导时启动：

```console
systemctl enable auditd
```

可以使用 `# auditctl -e 0` 命令临时禁用 `auditd`，并使用 `# auditctl -e 1` 重新启用它。

可以使用 `service auditd _<action>_` 命令对 `auditd` 执行其他操作，其中 _<action>_ 可以是以下之一：

`stop` ：停止 `auditd`。

`restart`：重新启动 `auditd`。

`reload` 或 `force-reload`：重新加载 `/etc/audit/auditd.conf` 文件中 `auditd` 的配置。

`rotate`：轮转 `/var/log/audit/` 目录中的日志文件。

`resume`：在其之前被暂停后重新恢复审计事件记录，例如，当保存审计日志文件的磁盘分区中没有足够的可用空间时。

`condrestart` 或 `try-restart`：只有当 `auditd` 运行时才重新启动它。

`status`：显示 `auditd` 的运行状态。

## 配置规则

举例说明



## 参考链接
[1] https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/8/html/security_hardening/auditing-the-system_security-hardening#linux-audit_auditing-the-system

[2] https://deepinout.com/linux-cmd/linux-audit-system-related-cmd/linux-cmd-auditctl.html