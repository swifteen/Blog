---
title: GDB常用命令
weight: 2
---

# GDB调试常用命令

## 设置gdbinit初始环境

GDB reads commands from $HOME/.gdbinit, then from .gdbinit in the current directory, and then from files specified on the command line with the -x parameter.

https://wizardforcel.gitbooks.io/100-gdb-tips/content/config-gdbinit.html

创建~/.gdbinit文件，添加如下内容，避免每次重复设置

```shell
set sysroot /home/ubuntu/test_ti_sdk_debug/linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/
#设置源码搜索目录
directory /home/ubuntu/targetNFS/test
set environment LD_LIBRARY_PATH /usr/local/lib/

define bsave
    save breakpoints ~/.breakpoints
end

define brestore
   source ~/.breakpoints
end

# 保存历史命令
set history filename ~/.gdb_history
set history save on

# 退出时不显示提示信息
set confirm off

# 按照派生类型打印对象
set print object on

# 打印数组的索引下标
set print array-indexes on

# 每行打印一个结构体成员
set print pretty on
```

运行时指定gdbinit文件,推荐使用

```shell
[linux-devkit]:~> arm-none-linux-gnueabihf-gdb test_main -x ../.gdbinit 
```

## 设置断点

```shell
#filename:linenum
(gdb) break /Full/path/to/service.cpp:45
#function
(gdb) b NamespaceA::ClassA::foo
#filename:function
(gdb) b object5.cpp:test
```

## 保存断点

https://stackoverflow.com/questions/501486/getting-gdb-to-save-a-list-of-breakpoints

As of GDB 7.2 (2011-08-23) you can now use the *[save breakpoints](https://sourceware.org/gdb/onlinedocs/gdb/Save-Breakpoints.html)* command.

```vhdl
(gdb) save breakpoints <filename>
  Save all current breakpoint definitions to a file suitable for use
  in a later debugging session.  To read the saved breakpoint
  definitions, use the `source' command.
```

Use source command  to restore the saved breakpoints from the file.

```shell
(gdb) source  <filename>
```

测试

```shell(gdb) source br_file
(gdb) save breakpoints br_file
Saved to file 'br_file'.
(gdb) source br_file
Breakpoint 1 at 0x43ccea: file /home/ubuntu/targetNFS/test/TestMain.cpp, line 219.
Breakpoint 2 at 0x43ccf6: file /home/ubuntu/targetNFS/test/TestMain.cpp, line 220.
```

## 删除断点

```shell
clear
clear function
clear filename:function
clear linenum
clear filename:linenum
delete [breakpoints] [range...]
d    #清空所有
```
## 查看断点

设置记录日志

```delphi
(gdb) b main
Breakpoint 1 at 0x8049329
(gdb) info break
Num     Type           Disp Enb Address    What
1       breakpoint     keep y   0x08049329 <main+16>
(gdb) set logging file breaks.txt
(gdb) set logging on
Copying output to breaks.txt.
(gdb) info break
Num     Type           Disp Enb Address    What
1       breakpoint     keep y   0x08049329 <main+16>
(gdb) q
```

```shell
(gdb) i b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x000000000043ccea in main() at /home/ubuntu/targetNFS/test/TestMain.cpp:219
2       breakpoint     keep y   0x000000000043ccf6 in main() at /home/ubuntu/targetNFS/test/TestMain.cpp:220
```

## 查看变量值

1、每次停在断点时，自动打印变量值

Use the `display` command:

```
(gdb> display decoder.m_msg
```

2、打印所有变量

This will cause `decoder.m_msg` to be printed every time that the prompt is shown (not only after a breakpoint).

Type [`info variables`](http://sourceware.org/gdb/current/onlinedocs/gdb/Symbols.html#index-info-variables-918) to list "All global and static variable names".

Type [`info locals`](http://sourceware.org/gdb/current/onlinedocs/gdb/Frame-Info.html#index-info-locals-435) to list "Local variables of current stack frame" (names and values), including static variables in that function.

Type [`info args`](https://sourceware.org/gdb/current/onlinedocs/gdb/Frame-Info.html#index-info-args) to list "Arguments of the current stack frame" (names and values).

## 传入运行参数

```shell
gdb --args executablename arg1 arg2 arg3
```

```shell
(gdb) set args a b c
(gdb) show args
Argument list to give program being debugged when it is started is "a b c".
```

```shell
(gdb) r a b
Starting program: /home/xmj/tmp/a.out a b
```

## Beej's Quick Guide to GDB

| **command**                       | Note                               |
| ------------------------------------------- | ------------------------------------------------------------ |
| **help *command***                        | Get help on a certain command                                |
| **apropos *keyword***                     | Search help for a particular keyword                         |
| **Starting and Quitting**                   |                                                              |
| **gdb *[-tui] [-c core] [exename]***      | **(Unix Command)** Start **gdb** on an executable or standalone; specify "-tui" to start the TUI GUI; specify "-c" with a corefile name to see where a crash occurred |
| **run *[arg1] [arg2] [...]***             | Run the currently loaded program with the given command line arguments |
| **quit**                                    | Exit the debugger                                            |
| **file *exename***                        | Load an executable file by name                              |
| **Breakpoints and Watchpoints**             |                                                              |
| **break *location***                      | Set a breakpoint at a location, line number, or file (e.g. "main", "5", or "hello.c:23") |
| **watch *expression***                    | Break when a variable is written to                          |
| **rwatch *expression***                   | Break when a variable is read from                           |
| **awatch *expression***                   | Break when a variable is written to or read from             |
| **info break**                              | Display breakpoint and watchpoint information and numbers    |
| **info watch**                              | Same as **info break**                                       |
| **clear *location***                      | Clear a breakpoint from a location                           |
| **delete *num***                          | Delete a breakpoint or watchpoint by number                  |
| **Stepping and Running**                    |                                                              |
| **next**                                    | Run to the next line of this function                        |
| **step**                                    | Step into the function on this line, if possible             |
| **stepi**                                   | Step a single assembly instruction                           |
| **continue**                                | Keep running from here                                       |
| **CTRL-C**                                  | Stop running, wherever you are                               |
| **finish**                                  | Run until the end of the current function                    |
| **advance *location***                    | Advance to a location, line number, or file (e.g. "somefunction", "5", or "hello.c:23") |
| **jump *location***                       | Just like **continue**, except jump to a particular location first. |
| **Examining and Modifying Variables**       |                                                              |
| **display *expression***                  | Display the value of a variable or expression every step of the program—the expression must make sense in the current scope |
| **info display**                            | Show a list of expressions currently being displayed and their numbers |
| **undisplay *num***                       | Stop showing an expression identified by its number (see **info display**) |
| **print *expression***                    | Print the value of a variable or expression                  |
| **printf *formatstr* *expressionlist*** | Do some formatted output with `printf()` e.g. `printf "i = %d, p = %s\n", i, p` |
| **set variable *expression***             | Set a variable to value, e.g. `set variable x=20`            |
| **set (*expression*)**                    | Works like **set variable**                                  |
| **Window Commands**                         |                                                              |
| **info win**                                | Shows current window info                                    |
| **focus *winname***                       | Set focus to a particular window bby name ("SRC", "CMD", "ASM", or "REG") or by position ("next" or "prev") |
| **fs**                                      | Alias for **focus**                                          |
| **layout *type***                         | Set the window layout ("src", "asm", "split", or "reg")      |
| **tui reg *type***                        | Set the register window layout ("general", "float", "system", or "next") |
| **winheight *val***                       | Set the window height (either an absolute value, or a relative value prefaced with "+" or "-") |
| **wh**                                      | Alias for **winheight**                                      |
| **set disassembly-flavor *flavor***       | Set the look-and-feel of the disassembly. On Intel machines, valid flavors are **intel** and **att** |
| **Misc Commands**                           |                                                              |
| **RETURN**                                  | Hit RETURN to repeat the last command                        |
| **backtrace**                               | Show the current stack                                       |
| **bt**                                      | Alias for **backtrace**                                      |
| **attach *pid***                          | Attach to an already-running process by its PID              |
| **info registers**                          | Dump integer registers to screen                             |
| **info all-registers**                      | Dump all registers to screen                                 |
# 参考链接

https://visualgdb.com/gdbreference/commands/

https://wizardforcel.gitbooks.io/100-gdb-tips

https://beej.us/guide/bggdb/

