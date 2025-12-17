---
title: 性能分析-Windows（二）
date: 2025-12-17
tags:
  - windows
  - 性能
---
# Overview

介绍几款Windows下的性能分析工具。

# 工具介绍

- Perfmon（性能监视器）：是Windows自带的性能监控工具，可以通过它来查看CPU、内存、硬盘、网络等系统性能指标的实时数据，也可以将数据保存到文件中进行后续分析。
- Sysinternals Suite：是一个由微软官方提供的一组系统工具集合，其中包括了很多用于性能监控的工具，例如Process Monitor、DiskMon、TCPView等。
- Windows Performance Toolkit：是一组高级性能监控工具，可用于性能分析和故障排查，包括xperf、WPR、WPA等。
- perfview：Perfview是一个开源的CPU和内存性能分析工具，也包括一些针对.NET的分析功能，例如GC分析，JIT分析，甚至ASP.NET中的请求统计等等。
- System Informer：系统资源监控工具，支持windows10、windows11。

# 工具概览

## Perfmon

Windows自带的，详细使用方法，参考官方[链接](https://techcommunity.microsoft.com/t5/ask-the-performance-team/windows-performance-monitor-overview/ba-p/375481)
![](perfmon.png)

## Sysinternals

下载后即可使用的工具集，官方下链接[下载](https://learn.microsoft.com/zh-cn/sysinternals/)，如常用的process monitor也包括在内
![](sysinternal.png)

## Windows Performance Toolkit

这个是Windows下的性能工具合集，见[官网](https://learn.microsoft.com/zh-cn/windows-hardware/test/wpt/windows-performance-toolkit-technical-reference)，Windows 性能工具包包含两个独立的工具：Windows Performance Recorder (WPR) 和 Windows Performance Analyzer (WPA) 此外，还保留了对以前的命令行工具 Xperf 的支持。 但是，不再支持 Xperfview。 所有记录都必须使用 WPA 来打开和分析。

需要使用WPR收集数据，再使用WPA分析数据。
![](wpa.png)

## perfview

perfview[下载地址](https://github.com/microsoft/perfview)。Perfview是一个开源的CPU和内存性能分析工具，也包括一些针对.NET的分析功能，例如GC分析，JIT分析，甚至ASP.NET中的请求统计等等。Perfview是一个Windows应用程序，但也能对在Linux系统上采集的数据进行分析（[参考](https://blogs.msdn.microsoft.com/vancem/2016/02/20/analyzing-cpu-traces-from-linux-with-perfview/)）。Perfview免安装，而且只是一个14M的.exe文件，非常容易部署到需要进行性能分析的机器上，例如生产环境的服务器。而且在性能数据收集的过程中不需要重启应用程序或者服务器，而且收集的性能数据日志（.etl文件）可以被拷贝到其他Windows机器上，再进行分析工作，对业务的影响非常少。
![](perfview.png)  

## System Informer
系统资源监控工具，支持windows10、windows11，官网[地址](https://systeminformer.sourceforge.io/)。
![](sysinformer.png)

