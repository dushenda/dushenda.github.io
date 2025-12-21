---
title: hexo增加mermaid
date: 2025-12-21
tags:
  - hexo
---
```mermaid
flowchart TD
    A[开始配置Mermaid] --> B[安装插件]
    B --> C[修改Hexo配置文件]
    C --> D{主题是否内置<br>Mermaid支持?}
    D -- 是 --> E[启用主题配置]
    D -- 否 --> F[手动添加JS引用]
    E --> G[清理并重新生成]
    F --> G
    G --> H[在博文中使用Mermaid]
```

	
sequenceDiagram
Alice->>John: Hello John, how are you?
John-->>Alice: Great!
Alice-)John: See you later!
flowchart TD
Start --> Stop
id1(Some text)
id1[(Database)]
id1((Some text))



flowchart TD
Start --> Stop


````
```mehrmaid
graph LR
T1 --> T2 & T3 --> T4 & T5 --> T6

T1("![|100x100](https://upload.wikimedia.org/wikipedia/commons/thumb/1/10/2023_Obsidian_logo.svg/1024px-2023_Obsidian_logo.svg.png)")
T2("$\nabla_\theta \mathbb{E}_{\tau\sim p_\theta}[R(\tau)]x$")
T3("| First Name | Last Name |
| ---------- | --------- |
| Doug       | Engelbart |")
T4("#plugins/mehrmaid")
T5("[[mehrmaid]]")
T6("![|80](https://upload.wikimedia.org/wikipedia/commons/thumb/6/60/Obsidian_software_logo.svg/1297px-Obsidian_software_logo.svg.png)")
```
````


