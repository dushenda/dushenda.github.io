---
title: 常用shell命令
date: 2025-12-21
tags:
  - shell
  - linux
---
# Bash快捷键
![](moving_cli.png)
## Moving

|command|description|
|---|---|
|ctrl + a|Goto BEGINNING of command line|
|ctrl + e|Goto END of command line|
|ctrl + b|move back one character|
|ctrl + f|move forward one character|
|alt + f|move cursor FORWARD one word|
|alt + b|move cursor BACK one word|
|ctrl + xx|Toggle between the start of line and current cursor position|
|ctrl + ] + x|Where x is any character, moves the cursor forward to the next occurance of x|
|alt + ctrl + ] + x|Where x is any character, moves the cursor backwards to the previous occurance of x|

## Edit / Other

|command|description|
|---|---|
|ctrl + d|Delete the character under the cursor|
|ctrl + h|Delete the previous character before cursor|
|ctrl + u|Clear all / cut BEFORE cursor|
|ctrl + k|Clear all / cut AFTER cursor|
|ctrl + w|delete the word BEFORE the cursor|
|alt + d|delete the word FROM the cursor|
|ctrl + y|paste (if you used a previous command to delete)|
|ctrl + i|command completion like Tab|
|ctrl + l|Clear the screen (same as clear command)|
|ctrl + c|kill whatever is running|
|ctrl + d|Exit shell (same as exit command when cursor line is empty)|
|ctrl + z|Place current process in background|
|ctrl + _|Undo|
|ctrl + x ctrl + u|Undo the last changes. ctrl+ _ does the same|
|ctrl + t|Swap the last two characters before the cursor|
|esc + t|Swap last two words before the cursor|
|alt + t|swap current word with previous|
|esc + .||
|esc + _||
|alt + [Backspace]|delete PREVIOUS word|
|alt + <|Move to the first line in the history|
|alt + >|Move to the end of the input history, i.e., the line currently being entered|
|alt + ?|display the file/folder names in the current path as help|
|alt + *|print all the file/folder names in the current path as parameter|
|alt + .|print the LAST ARGUMENT (ie "vim file1.txt file2.txt" will yield "file2.txt")|
|alt + c|capitalize the first character to end of word starting at cursor (whole word if cursor is at the beginning of word)|
|alt + u|make uppercase from cursor to end of word|
|alt + l|make lowercase from cursor to end of word|
|alt + n||
|alt + p|Non-incremental reverse search of history.|
|alt + r|Undo all changes to the line|
|alt + ctl + e|Expand command line.|
|~[TAB][TAB]|List all users|
|$[TAB][TAB]|List all system variables|
|@[TAB][TAB]|List all entries in your /etc/hosts file|
|[TAB]|Auto complete|
|cd -|change to PREVIOUS working directory|

## History

|command|description|
|---|---|
|ctrl + r|Search backward starting at the current line and moving 'up' through the history as necessary|
|crtl + s|Search forward starting at the current line and moving 'down' through the history as necessary|
|ctrl + p|Fetch the previous command from the history list, moving back in the list (same as up arrow)|
|ctrl + n|Fetch the next command from the history list, moving forward in the list (same as down arrow)|
|ctrl + o|Execute the command found via Ctrl+r or Ctrl+s|
|ctrl + g|Escape from history searching mode|
|!!|Run PREVIOUS command (ie `sudo !!`)|
|!vi|Run PREVIOUS command that BEGINS with vi|
|!vi:p|Print previously run command that BEGINS with vi|
|!n|Execute nth command in history|
|!$|Last argument of last command|
|!^|First argument of last command|
|^abc^xyz|Replace first occurance of abc with xyz in last command and execute it|

# 常用命令

> https://github.com/jlevy/the-art-of-command-line/tree/master

## bash命令行

- 如果你输入命令的时候中途改了主意，按下 **alt-#** 在行首添加 `#` 把它当做注释再按下回车执行（或者依次按下 **ctrl-a**， **#**， **enter**）。这样做的话，之后借助命令行历史记录，你可以很方便恢复你刚才输入到一半的命令。
```shell
➜  ~ # yum install gcc   <--前面加#
➜  ~ # yum install gcc   <-- 按上键
```
- 使用 `netstat -lntp` 或 `ss -plat` 检查哪些进程在监听端口（默认是检查 TCP 端口; 添加参数 `-u` 则检查 UDP 端口）或者 `lsof -iTCP -sTCP:LISTEN -P -n` (这也可以在 OS X 上运行)。
- `pstree -p` 以一种优雅的方式展示进程树。
- 使用 `nohup` 或 `disown` 使一个后台进程持续运行。
- 使用 `pgrep` 和 `pkill` 根据名字查找进程或发送信号（`-f` 参数通常有用）。
- 将 shell 切换为其他用户，使用 `su username` 或者 `su - username`。加入 `-` 会使得切换后的环境与使用该用户登录后的环境相同。省略用户名则默认为 root。切换到哪个用户，就需要输入_哪个用户的_密码。
- 可以把别名、shell 选项和常用函数保存在 `~/.bashrc`，具体看下这篇[文章](http://superuser.com/a/183980/7106)。这样做的话你就可以在所有 shell 会话中使用你的设定。
- 把环境变量的设定以及登陆时要执行的命令保存在 `~/.bash_profile`。而对于从图形界面启动的 shell 和 `cron` 启动的 shell，则需要单独配置文件。
- shell PS1显示
```shell
# 1. 编辑配置文件（以当前用户的 ~/.bashrc 为例）
vim ~/.bashrc

# 2. 在文件末尾添加一行，例如：
# 显示效果类似于：[Sun Dec 21 14:05:30] user@server:~$
export PS1="[\d \t] \u@\h:\W\$ "

# 3. 让配置立即生效
source ~/.bashrc
```

## 文件及数据处理

- `wc` 去计算新行数（`-l`），字符数（`-m`），单词数（`-w`）以及字节数（`-c`）。
- 使用 `tee` 将标准输入复制到文件甚至标准输出，例如 `ls -al | tee file.txt`。
- - 替换一个或多个文件中出现的字符串：
```shell
	perl -pi.bak -e 's/old-string/new-string/g' my-files-*.txt
```
- `rsync` 是一个快速且非常灵活的文件复制工具。它闻名于设备之间的文件同步，但其实它在本地情况下也同样有用。在安全设置允许下，用 `rsync` 代替 `scp` 可以实现文件续传，而不用重新从头开始。它同时也是删除大量文件的[最快方法](https://web.archive.org/web/20130929001850/http://linuxnote.net/jianingy/en/linux/a-fast-way-to-remove-huge-number-of-files.html)之一：

```shell
mkdir empty && rsync -r --delete empty/ some-dir && rmdir some-dir
```
- 标准的源代码对比及合并工具是 `diff` 和 `patch`。使用 `diffstat` 查看变更总览数据。注意到 `diff -r` 对整个文件夹有效。使用 `diff -r tree1 tree2 | diffstat` 查看变更的统计数据。`vimdiff` 用于比对并编辑文件。
![](diff.png)
- 对于二进制文件，使用 `hd`，`hexdump` 或者 `xxd` 使其以十六进制显示，使用 `bvi`，`hexedit` 或者 `biew` 来进行二进制编辑。
- 同样对于二进制文件，`strings`（包括 `grep` 等工具）可以帮助在二进制文件中查找特定比特。
- 拆分文件可以使用 `split`（按大小拆分）和 `csplit`（按模式拆分）。
- 操作日期和时间表达式，可以用 [`dateutils`](http://www.fresse.org/dateutils/) 中的 `dateadd`、`datediff`、`strptime` 等工具。
- 使用 `zless`、`zmore`、`zcat` 和 `zgrep` 对压缩过的文件进行操作。
- 文件属性可以通过 `chattr` 进行设置，它比文件权限更加底层。例如，为了保护文件不被意外删除，可以使用不可修改标记：`sudo chattr +i /critical/directory/or/file`
- 使用 `getfacl` 和 `setfacl` 以保存和恢复文件权限。例如：
```shell
   getfacl -R /some/path > permissions.txt
   setfacl --restore=permissions.txt
```
- 为了高效地创建空文件，请使用 `truncate`（创建[稀疏文件](https://zh.wikipedia.org/wiki/%E7%A8%80%E7%96%8F%E6%96%87%E4%BB%B6)），`fallocate`（用于 ext4，xfs，btrf 和 ocfs2 文件系统），`xfs_mkfile`（适用于几乎所有的文件系统，包含在 xfsprogs 包中），`mkfile`（用于类 Unix 操作系统，比如 Solaris 和 Mac OS）。

## 系统调试


- `curl` 和 `curl -I` 可以被轻松地应用于 web 调试中，它们的好兄弟 `wget` 也是如此，或者也可以试试更潮的 [`httpie`](https://github.com/jkbrzt/httpie)。
- 获取 CPU 和硬盘的使用状态，通常使用使用 `top`（`htop` 更佳），`iostat` 和 `iotop`。而 `iostat -mxz 15` 可以让你获悉 CPU 和每个硬盘分区的基本信息和性能表现。
- 使用 `netstat` 和 `ss` 查看网络连接的细节。
- `dstat` 在你想要对系统的现状有一个粗略的认识时是非常有用的。然而若要对系统有一个深度的总体认识，使用 [`glances`](https://github.com/nicolargo/glances)，它会在一个终端窗口中向你提供一些系统级的数据。
- 若要了解内存状态，运行并理解 `free` 和 `vmstat` 的输出。值得留意的是“cached”的值，它指的是 Linux 内核用来作为文件缓存的内存大小，而与空闲内存无关。
- Java 系统调试则是一件截然不同的事，一个可以用于 Oracle 的 JVM 或其他 JVM 上的调试的技巧是你可以运行 `kill -3 <pid>` 同时一个完整的栈轨迹和堆概述（包括 GC 的细节）会被保存到标准错误或是日志文件。JDK 中的 `jps`，`jstat`，`jstack`，`jmap` 很有用。[SJK tools](https://github.com/aragozin/jvm-tools) 更高级。
- 使用 [`mtr`](http://www.bitwizard.nl/mtr/) 去跟踪路由，用于确定网络问题。
- 用 [`ncdu`](https://dev.yorhel.nl/ncdu) 来查看磁盘使用情况，它比寻常的命令，如 `du -sh *`，更节省时间。
- 查找正在使用带宽的套接字连接或进程，使用 [`iftop`](http://www.ex-parrot.com/~pdw/iftop/) 或 [`nethogs`](https://github.com/raboof/nethogs)。
- `ab` 工具（Apache 中自带）可以简单粗暴地检查 web 服务器的性能。对于更复杂的负载测试，使用 `siege`。
- [`wireshark`](https://wireshark.org/)，[`tshark`](https://www.wireshark.org/docs/wsug_html_chunked/AppToolstshark.html) 和 [`ngrep`](http://ngrep.sourceforge.net/) 可用于复杂的网络调试。
- 了解 `strace` 和 `ltrace`。这俩工具在你的程序运行失败、挂起甚至崩溃，而你却不知道为什么或你想对性能有个总体的认识的时候是非常有用的。注意 profile 参数（`-c`）和附加到一个运行的进程参数 （`-p`）。
- 了解使用 `ldd` 来检查共享库。但是[永远不要在不信任的文件上运行](http://www.catonmat.net/blog/ldd-arbitrary-code-execution/)。
- 了解如何运用 `gdb` 连接到一个运行着的进程并获取它的堆栈轨迹。
- 学会使用 `/proc`。它在调试正在出现的问题的时候有时会效果惊人。比如：`/proc/cpuinfo`，`/proc/meminfo`，`/proc/cmdline`，`/proc/xxx/cwd`，`/proc/xxx/exe`，`/proc/xxx/fd/`，`/proc/xxx/smaps`（这里的 `xxx` 表示进程的 id 或 pid）。
- 当调试一些之前出现的问题的时候，[`sar`](http://sebastien.godard.pagesperso-orange.fr/) 非常有用。它展示了 cpu、内存以及网络等的历史数据。
- 关于更深层次的系统分析以及性能分析，看看 `stap`（[SystemTap](https://sourceware.org/systemtap/wiki)），[`perf`](https://en.wikipedia.org/wiki/Perf_\(Linux\))，以及[`sysdig`](https://github.com/draios/sysdig)。
- 查看你当前使用的系统，使用 `uname`，`uname -a`（Unix／kernel 信息）或者 `lsb_release -a`（Linux 发行版信息）。
- 无论什么东西工作得很欢乐（可能是硬件或驱动问题）时可以试试 `dmesg`。
- 如果你删除了一个文件，但通过 `du` 发现没有释放预期的磁盘空间，请检查文件是否被进程占用： `lsof | grep deleted | grep "filename-of-my-big-file"`

## 单行脚本

- 当你需要对文本文件做集合交、并、差运算时，`sort` 和 `uniq` 会是你的好帮手。具体例子请参照代码后面的，此处假设 `a` 与 `b` 是两内容不同的文件。这种方式效率很高，并且在小文件和上 G 的文件上都能运用（注意尽管在 `/tmp` 在一个小的根分区上时你可能需要 `-T` 参数，但是实际上 `sort` 并不被内存大小约束），参阅前文中关于 `LC_ALL` 和 `sort` 的 `-u` 参数的部分。

```shell
	sort a b | uniq > c   # c 是 a 并 b
	sort a b | uniq -d > c   # c 是 a 交 b
	sort a b b | uniq -u > c   # c 是 a - b
```
- 使用 `grep . *`（每行都会附上文件名）或者 `head -100 *`（每个文件有一个标题）来阅读检查目录下所有文件的内容。这在检查一个充满配置文件的目录（如 `/sys`、`/proc`、`/etc`）时特别好用。
- 计算文本文件第三列中所有数的和（可能比同等作用的 Python 代码快三倍且代码量少三倍）：
```shell
      awk '{ x += $3 } END { print x }' myfile
```
- 如果你想在文件树上查看大小/日期，这可能看起来像递归版的 `ls -l` 但比 `ls -lR` 更易于理解：
```shell
	find . -type f -ls
```
- 假设你有一个类似于 web 服务器日志文件的文本文件，并且一个确定的值只会出现在某些行上，假设一个 `acct_id` 参数在 URI 中。如果你想计算出每个 `acct_id` 值有多少次请求，使用如下代码：
```shell
	egrep -o 'acct_id=[0-9]+' access.log | cut -d= -f2 | sort | uniq -c | sort -rn
```
- 要持续监测文件改动，可以使用 `watch`，例如检查某个文件夹中文件的改变，可以用 `watch -d -n 2 'ls -rtlh | tail'`；或者在排查 WiFi 设置故障时要监测网络设置的更改，可以用 `watch -d -n 2 ifconfig`。
- 运行这个函数从这篇文档中随机获取一条技巧（解析 Markdown 文件并抽取项目）：
```shell
	function taocl() {
		curl -s https://raw.githubusercontent.com/jlevy/the-art-of-command-line/master/README-zh.md|
		pandoc -f markdown -t html |
        iconv -f 'utf-8' -t 'unicode' |
        xmlstarlet fo --html --dropdtd |
        xmlstarlet sel -t -v "(html/body/ul/li[count(p)>0])[$RANDOM mod last()+1]" |
        xmlstarlet unesc | fmt -80
    }
```
## 冷门但有用

- `expr`：计算表达式或正则匹配
- `m4`：简单的宏处理器
- `yes`：多次打印字符串
- `cal`：漂亮的日历
- `env`：执行一个命令（脚本文件中很有用）
- `printenv`：打印环境变量（调试时或在写脚本文件时很有用）
- `look`：查找以特定字符串开头的单词或行
- `cut`，`paste` 和 `join`：数据修改
- `fmt`：格式化文本段落
- `pr`：将文本格式化成页／列形式
- `fold`：包裹文本中的几行
- `column`：将文本格式化成多个对齐、定宽的列或表格
- `expand` 和 `unexpand`：制表符与空格之间转换
- `nl`：添加行号
- `seq`：打印数字
- `bc`：计算器
- `factor`：分解因数
- [`gpg`](https://gnupg.org/)：加密并签名文件
- `toe`：terminfo 入口列表
- `nc`：网络调试及数据传输
- `socat`：套接字代理，与 `netcat` 类似
- [`slurm`](https://github.com/mattthias/slurm)：网络流量可视化
- `dd`：文件或设备间传输数据
- `file`：确定文件类型
- `tree`：以树的形式显示路径和文件，类似于递归的 `ls`
- `stat`：文件信息
- `time`：执行命令，并计算执行时间
- `timeout`：在指定时长范围内执行命令，并在规定时间结束后停止进程
- `lockfile`：使文件只能通过 `rm -f` 移除
- `logrotate`： 切换、压缩以及发送日志文件
- `watch`：重复运行同一个命令，展示结果并／或高亮有更改的部分
- [`when-changed`](https://github.com/joh/when-changed)：当检测到文件更改时执行指定命令。参阅 `inotifywait` 和 `entr`。
- `tac`：反向输出文件
- `shuf`：文件中随机选取几行
- `comm`：一行一行的比较排序过的文件
- `strings`：从二进制文件中抽取文本
- `tr`：转换字母
- `iconv` 或 `uconv`：文本编码转换
- `split` 和 `csplit`：分割文件
- `sponge`：在写入前读取所有输入，在读取文件后再向同一文件写入时比较有用，例如 `grep -v something some-file | sponge some-file`
- `units`：将一种计量单位转换为另一种等效的计量单位（参阅 `/usr/share/units/definitions.units`）
- `apg`：随机生成密码
- `xz`：高比例的文件压缩
- `ldd`：动态库信息
- `nm`：提取 obj 文件中的符号
- `ab` 或 [`wrk`](https://github.com/wg/wrk)：web 服务器性能分析
- `strace`：调试系统调用
- [`mtr`](http://www.bitwizard.nl/mtr/)：更好的网络调试跟踪工具
- `cssh`：可视化的并发 shell
- `rsync`：通过 ssh 或本地文件系统同步文件和文件夹
- [`wireshark`](https://wireshark.org/) 和 [`tshark`](https://www.wireshark.org/docs/wsug_html_chunked/AppToolstshark.html)：抓包和网络调试工具
- [`ngrep`](http://ngrep.sourceforge.net/)：网络层的 grep
- `host` 和 `dig`：DNS 查找
- `lsof`：列出当前系统打开文件的工具以及查看端口信息
- `dstat`：系统状态查看
- [`glances`](https://github.com/nicolargo/glances)：高层次的多子系统总览
- `iostat`：硬盘使用状态
- `mpstat`： CPU 使用状态
- `vmstat`： 内存使用状态
- `htop`：top 的加强版
- `last`：登入记录
- `w`：查看处于登录状态的用户
- `id`：用户/组 ID 信息
- [`sar`](http://sebastien.godard.pagesperso-orange.fr/)：系统历史数据
- [`iftop`](http://www.ex-parrot.com/~pdw/iftop/) 或 [`nethogs`](https://github.com/raboof/nethogs)：套接字及进程的网络利用情况
- `ss`：套接字数据
- `dmesg`：引导及系统错误信息
- `sysctl`： 在内核运行时动态地查看和修改内核的运行参数
- `hdparm`：SATA/ATA 磁盘更改及性能分析
- `lsblk`：列出块设备信息：以树形展示你的磁盘以及磁盘分区信息
- `lshw`，`lscpu`，`lspci`，`lsusb` 和 `dmidecode`：查看硬件信息，包括 CPU、BIOS、RAID、显卡、USB设备等
- `lsmod` 和 `modinfo`：列出内核模块，并显示其细节
- `fortune`，`ddate` 和 `sl`：额，这主要取决于你是否认为蒸汽火车和莫名其妙的名人名言是否“有用”