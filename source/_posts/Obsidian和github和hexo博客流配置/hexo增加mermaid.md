---
title: hexo增加mermaid
date: 2025-12-21
tags:
  - hexo
---
```flowchart TD
    A[开始配置Mermaid] --> B[安装插件]
    B --> C[修改Hexo配置文件]
    C --> D{主题是否内置<br>Mermaid支持?}
    D -- 是 --> E[启用主题配置]
    D -- 否 --> F[手动添加JS引用]
    E --> G[清理并重新生成]
    F --> G
    G --> H[在博文中使用Mermaid]
```