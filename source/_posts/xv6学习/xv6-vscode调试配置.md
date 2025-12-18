---
title: xv6-vscode调试
date: 2025-12-10
tags:
  - xv6
  - 操作系统
  - 环境搭建
---
## .gdbinit
```shell
set $lastcs = -1

define hook-stop
  # There doesn't seem to be a good way to detect if we're in 16- or
  # 32-bit mode, but in 32-bit mode we always run with CS == 8 in the
  # kernel and CS == 35 in user space
  if $cs == 8 || $cs == 35
    if $lastcs != 8 && $lastcs != 35
      set architecture i386
    end
    x/i $pc
  else
    if $lastcs == -1 || $lastcs == 8 || $lastcs == 35
      set architecture i8086
    end
    # Translate the segment:offset into a physical address
    printf "[%4x:%4x] ", $cs, $eip
    x/i $cs*16+$eip
  end
  set $lastcs = $cs
end

# 需要注释掉
# echo + target remote localhost:25000\n
# target remote localhost:25000

echo + symbol-file kernel\n
symbol-file kernel

```

## launch.json
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug xv6 x86",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceFolder}/kernel",
      "miDebuggerServerAddress": "localhost:25000",
      "miDebuggerPath": "/usr/bin/gdb",
      "stopAtEntry": true,
      "cwd": "${workspaceFolder}",
      "externalConsole": false,
      "setupCommands": [
        {
          "text": "set architecture i386:x86-64"
        },
        {
          "text": "set disassemble-next-line auto"
        },
        {
          "description": "Enable pretty-print for gdb",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        }
      ],
      "preLaunchTask": "xv6build",
      "logging": {
        "trace": false,
        "traceResponse": false,
        "engineLogging": false
      }
    }
  ]
}
```
## tasks.json
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "xv6build",
      "type": "shell",
      "isBackground": true, 
      "command": "make qemu-nox-gdb", 
      "problemMatcher": {
        "pattern":{
          "regexp": ".",
          "file": 1,
          "location": 2,
          "message": 3
        },
        "background": {
          "beginsPattern": ".* Now run 'gdb'.",
          "endsPattern": "."
        }
      }
    }
  ]
}
```
