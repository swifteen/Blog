---
title: GDB tui模式
weight: 5
---


# 编译GDB支持TUI模式

## 方法1：按照下面两个参考链接中尝试无效

https://eastrivervillage.com/debugging-application-with-cross-gdb-yocto/

https://mikeframpo.net/notes/2021/06/20/yocto-debugging-gdb.html

```shell
$ cat meta-kaba-hacks/recipes-devtools/gdb/gdb-%.bbappend 
EXTRA_OECONF += " --enable-tui"
```

## 方法2：更换顺序tui的配置顺序，重新编译

```shell
 vi ../sources/oe-core/meta/recipes-devtools/gdb/gdb-common.inc +40
 
 40 #PACKAGECONFIG[tui] = "--enable-tui,--disable-tui"
 41 PACKAGECONFIG[tui] = "--disable-tui,--enable-tui"
```

```shell
MACHINE=am335x-evm bitbake  -c clean gdb
MACHINE=am335x-evm bitbake  -c cleansstate gdb
MACHINE=am335x-evm bitbake  -k gdb
MACHINE=am335x-evm bitbake package-index
```

## 提示出错

Cannot enable the TUI: terminal doesn't support cursor addressing [TERM=dumb]

解决办法

```shell
export TERM=linux
```


# Beej's Quick Guide to GDB

| **command**                       | Note                               |
| ------------------------------------------- | ------------------------------------------------------------ |
| **Window Commands**                         |                                                              |
| **info win**                                | Shows current window info                                    |
| **focus *winname***                       | Set focus to a particular window bby name ("SRC", "CMD", "ASM", or "REG") or by position ("next" or "prev") |
| **fs**                                      | Alias for **focus**                                          |
| **layout *type***                         | Set the window layout ("src", "asm", "split", or "reg")      |
| **tui reg *type***                        | Set the register window layout ("general", "float", "system", or "next") |
| **winheight *val***                       | Set the window height (either an absolute value, or a relative value prefaced with "+" or "-") |
| **wh**                                      | Alias for **winheight**                                      |
| **set disassembly-flavor *flavor***       | Set the look-and-feel of the disassembly. On Intel machines, valid flavors are **intel** and **att** |

