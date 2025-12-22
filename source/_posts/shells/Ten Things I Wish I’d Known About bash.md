---
title: Ten Things I Wish I’d Known About bash
date: 2025-12-22
tags:
  - Linux
  - bash
---
> https://zwischenzugs.com/2018/01/06/ten-things-i-wish-id-known-about-bash/

## 1)  ` `` ` vs `$()`

这两完成了一样的事情：
```shell
$ echo `ls`
$ echo $(ls)
```
` `` ` 是Unix时代引入的，`$()`更现代和易读易写。
比如：
```shell
$ echo `echo \`echo \\\`echo inside\\\`\``
$ echo $(echo $(echo $(echo inside)))
```

## 2) globbing vs regexps

glob和regexps是不一样的东西，glob是通配符，regexp是正则表达式。
看以下的表达式：
```shell
$ rename -n 's/(.*)/new$1/' *
```
这里有两个`*`号
- 第一个在引号内部，作为正则表达式的一部分
- 第二个在外部，作为通配符的部分
```shell
$ ls *
$ ls .*
```
所以第二个看着像是正则表达式，但是实际上并不是

## 3) Exit Codes

所有的命令都会返回值给shell
```shell
$ grep not_there /dev/null
$ echo $?
```
- 0就是命令执行正确，非0就是执行失败
- `$?`是一个特殊的符号，用来从shell里获取上一条命令的返回值

## 4) `if` statements, `[` and `[[`

- `[`（单方括号）实际上是`test`命令的别名，是一个内置命令。它用于基本的条件测试，需要遵循严格的语法格式（如括号内必须有空格）。
- `[[`（双方括号）​ 是Bash的关键字，提供了比`[`更强大的功能。它不是普通命令，而是Bash的语法结构，因此具有更灵活和直观的行为。
```shell
# 使用[（需要引号防止空变量错误）
if [ "$name" = "John" ]; then
    echo "Hello John"
fi

# 使用[[（更安全，无需引号）
if [[ $name == "John" ]]; then
    echo "Hello John"
fi
```
举个例子
```shell
# 使用[]
if [ $(grep not_there /dev/null) = '' ]
then
    echo -n hi
else
    echo -n lo
fi

# 使用[[]]
if [[ $(grep not_there /dev/null) = '' ]]
then
    echo -n hi
else
    echo -n lo
fi
```
- 第一个会报错，因为`$(grep not_there /dev/null)`返回值是空，那么最后就会变成`[ = '' ]`直接报错，这就是为什么经常能看到这些语句
```shell
if [ x$(grep not_there /dev/null) = 'x' ]
```
- 推荐使用第二种用法，因为它更安全、功能更丰富

## 5) `set`

当脚本中**任何一条命令**的退出状态码（`$?`）**非零**（即执行失败）时，脚本**立即终止**运行，后续命令不再执行
```shell
set -e
```
建议使用trap来做一些清理工作，例如
```shell
#!/bin/bash
set -e

cleanup() {
    echo "脚本在第 $1 行出错，正在执行清理..."
    # 清理临时文件等操作
}
# 设置陷阱，在收到 ERR 信号时（即命令失败）调用 cleanup 函数
trap 'cleanup $LINENO' ERR
```
`set -e`**管道命令的盲点**：默认只检查管道中**最后一个命令**的退出状态。如果管道中前面的命令失败，但最后一个命令成功，脚本不会退出 。解决方案是结合 `set -o pipefail`。推荐设置如下
```shell
#!/bin/bash
set -euo pipefail  # -e: 出错退出; -u: 使用未定义变量时报错; -o pipefail: 处理管道错误
```
调试命令，打印执行的命令
```shell
set -x
```
因此这个是设置一些选项
```shell
#!/bin/bash
set -e
set -x
grep not_there /dev/null  <-- 错误了之后直接退出
echo $?
```

## 6) ​​`<()`

`<()`输出的是文件，比如
```shell
# diff比较grep完file1和file2的结果
$ grep somestring file1 > /tmp/a
$ grep somestring file2 > /tmp/b
$ diff /tmp/a /tmp/b

# 直接等价于下面的命令，更优雅
diff <(grep somestring file1) <(grep somestring file2)
```

## 7) Quoting

引用在 bash 中是一个棘手的主题，就像在许多软件环境中一样。
```shell
A='123'  
echo "$A"   # <-- 123
echo '$A'   # <-- $A
```
非常简单——双引号取消引用变量，而单引号则直接引用
下面的表格清晰地总结了两者的主要差异：

| 特性        | 单引号 (`'`)                      | 双引号 (`"`)                          |
| --------- | ------------------------------ | ---------------------------------- |
| **变量替换**​ | 不进行替换，原样输出                     | 进行替换，输出变量值                         |
| **命令替换**​ | 不执行，原样输出                       | 执行，输出命令结果                          |
| **转义字符**​ | 不解析（``也视为普通字符）                 | 解析（如 `\n`, `\t`等，需配合 `echo -e`）    |
| **特殊字符**​ | 所有字符（`$`, `` ` ``, ``等）均失去特殊含义 | 仅部分字符（`$`, `` ` ``, `"`, ``）保留特殊含义 |
简单来说，单引号提供严格的字面意义保护，而双引号在保护字符串整体的同时，允许变量和命令等动态内容的插入。
```shell
mkdir -p tmp
cd tmp
touch a
echo "*"   # 输出"*"
echo '*'   # 输出"*"，*不保留特殊含义
```

- **使用单引号 (`'`)**：当你的字符串是**纯粹的文本常量**，不需要任何变量或命令替换，也不需要解析转义字符时。例如，固定的提示信息、符号文字等。
- **使用双引号 (`"`)**：当你的字符串中**需要包含变量、命令执行结果，或者需要处理包含空格的参数**时。这是更常见的情况，也是更安全的做法，因为它可以防止字符串被意外分割。
- **避免无引号**：除非是极简单的连续字符（如数字、路径），否则为变量赋值或传递参数时，**强烈建议总是使用引号**，这能有效预防许多难以排查的错误。

## 8) Top three shortcuts



## 9) startup order


|功能|语法|作用|经典使用场景|
|---|---|---|---|
|**获取最后参数**​|`!$`|代表**上一条命令的最后一个参数**。|当你执行 `ls /a/very/long/path`后，想进入该目录，只需 `cd !$`，等效于 `cd /a/very/long/path`。|
|**获取参数范围**​|`!:1-$`|代表**上一条命令中从第1个到最后一个的所有参数**。|误将 `tar -czf archive.tar.gz file1 file2`打成了 `zip`，可快速改为 `tar -czf archive.tar.gz !:1-$`。|
|**提取目录路径**​|`:h`  <br>（修饰符）|移除路径中的**最后一级**（文件名或目录名），返回纯目录路径。|操作文件失败时（如 `cat /etc/nginx/sites-available/my_site`），用 `cd !$:h`直接切换到文件所在目录 `/etc/nginx/sites-available`查看。|

`!$`：特殊变量，上一条命令的最后一个参数。如果你在处理文件，懒得一遍又一遍地重新输入命令，这可以省下很多工作：
```shell
grep somestring /long/path/to/some/file/or/other.txt
vi !$
```

`!:1-$`:这个组合使这更进一步。 它获取上一个命令的所有参数并将它们放入。所以：
```shell
grep isthere /long/path/to/some/file/or/other.txt
egrep !:1-$
fgrep !:1-$
```
> :是分隔符，1-$指的是第一个到最后一个参数，！指的是上一个命令

我也经常使用这个。 如果将其放在文件名后面，它将更改该文件名以删除该文件夹中的所有内容。 像这样：
```shell
grep isthere /long/path/to/some/file/or/other.txt
cd !$:h # 进入了/long/path/to/some/file/or/ 目录
```

## 9) startup order

bash的启动顺序如下图
![](shell-startup-actual.png)
它显示 bash 决定根据有关 bash 运行上下文的决策（决定要遵循的颜色）从顶部运行哪些脚本。

因此，如果处于本地（非远程）、非登录、交互式 shell 中（例如，当您从命令行运行 bash 本身时），您就位于“绿色”行，这些是读取文件的顺序：
```shell
/etc/bash.bashrc
~/.bashrc
# [bash runs, then terminates]
~/.bash_logout
```

## 10) getopts (cheapci)

Bash 的 `getopts`是一个内置命令，用于在 Shell 脚本中规范地解析命令行选项和参数。

`getopts`的基本命令格式为：
```
getopts optstring name [args]
```

- **`optstring`**：定义脚本识别的选项字符。
    - 如果一个字符后跟冒号（`:`），表示该选项需要参数，如 `a:`表示 `-a`需要跟一个参数。
    - 如果 `optstring`以冒号（`:`）开头，则开启**静默错误模式**，抑制默认错误信息，便于自定义错误处理。
- **`name`**：每次调用 `getopts`时，会将解析到的选项字符（不带 `-`）存入该变量。
- **`[args]`**：可选参数。若提供，`getopts`会解析这些参数而非脚本的位置参数（`$1, $2, ...`）。
`getopts`依赖两个重要的环境变量：

- **`OPTARG`**：当选项需要参数时，该参数值存放在 `OPTARG`变量中。
- **`OPTIND`**：保存下一个要被处理的参数索引。初始值为 1，处理完选项后，可用 `shift $(($OPTIND - 1))`跳过已处理的选项，使 `$1`指向第一个非选项参数。

通常将 `getopts`放入 `while`循环，结合 `case`语句处理不同选项。
```shell
#!/bin/bash

verbose=false
input_file=""
output_dir=""

while getopts ":vi:o:" opt; do
    case $opt in
        v) 
            verbose=true
            echo "详细模式已开启" 
            ;;
        i) 
            input_file="$OPTARG"
            echo "输入文件设置为: $input_file" 
            ;;
        o) 
            output_dir="$OPTARG"
            echo "输出目录设置为: $output_dir" 
            ;;
        \?) 
            echo "错误：不支持的选项 -$OPTARG" >&2
            exit 1 
            ;;
        :) 
            echo "错误：选项 -$OPTARG 需要一个参数" >&2
            exit 1 
            ;;
    esac
done

# 处理剩余的非选项参数
shift $(($OPTIND - 1))

if [ $# -gt 0 ]; then
    echo "剩余的非选项参数: $@"
fi
```
**脚本用法示例**：
```
$ ./myscript.sh -i data.txt -o /tmp/output -v file1 file2
输入文件设置为: data.txt
输出目录设置为: /tmp/output
详细模式已开启
剩余的非选项参数: file1 file2
```


1. **选项处理顺序与组合**
    `getopts`按顺序解析选项，遇到非选项参数（不以 `-`开头）或 `--`时停止。选项可以组合，如 `-vi file`等效于 `-v -i file`。
2. **错误处理模式**
    通过在 `optstring`前加冒号（`:`）开启静默模式。此模式下：

    - 遇到无效选项，`name`被设置为 `?`，`OPTARG`为无效选项字符。
    - 遇到缺少参数的选项，`name`被设置为 `:`，`OPTARG`为对应选项字符。
3. 处理完选项后，使用 `shift $(($OPTIND - 1))`来移除已处理的选项和其参数，便于后续处理剩余的非选项参数。
4. 重置 OPTIND
    如果在同一脚本中需要多次调用 `getopts`解析不同参数集，必须在每次解析新参数集前手动重置 `OPTIND=1`，因为 Shell 不会自动重置它。

%% 
- `getopts`主要用于解析**短选项**（如 `-a`, `-l`）。虽然可以通过一些技巧模拟处理长选项（如 `--help`），但过程较为复杂，通常需要借助 `getopt`命令（注意，不是内置的 `getopts`）。
- `getopts`是 Shell 内置命令，执行效率高。而 `getopt`是外部命令，功能更强大（如直接支持长选项），但使用也更复杂，且不同系统上的实现可能有差异。 
%%

If you go deep with bash, you might end up writing chunky utilities in it. If you do, then getting to grips with `getopts` can pay large dividends.

For fun, I once wrote a [script](https://github.com/ianmiell/cheapci/blob/master/cheapci) called `cheapci` which I used to work like a Jenkins job.

The code [here](https://github.com/ianmiell/cheapci/blob/master/cheapci#L70-L96) implements the reading of the [two required, and 14 non-required arguments](https://github.com/ianmiell/cheapci/blob/master/cheapci#L33-L51). Better to learn this than to build up a bunch of bespoke code that can get very messy pretty quickly as your utility grows.