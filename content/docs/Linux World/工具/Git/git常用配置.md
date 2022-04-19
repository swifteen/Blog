---
title: Git常用配置
weight: 1
---

# Github国内加速克隆及下载

fastgit.org
https://doc.fastgit.org/

gitclone.com
https://gitclone.com/

gitee
https://gitee.com/mirrors

cnpmjs.org
https://github.com.cnpmjs.org/

Github documentation contains [a script that replaces the committer info for all commits in a branch](https://help.github.com/articles/changing-author-info/) (now irretrievable, this is the [last snapshot](https://web.archive.org/web/20200823163529/https://docs.github.com/en/github/using-git/changing-author-info#changing-the-git-history-of-your-repository-using-a-script)).


# git代理

```shell
git config --global https.proxy 'socks5://192.168.31.181:10808'
git config --global http.proxy 'socks5://192.168.31.181:10808'
```

# 基本配置

Git的设置文件为`.gitconfig`，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

> ```bash
> # 显示当前的Git配置
> $ git config --list
> 
> # 编辑Git配置文件
> $ git config -e [--global]
> 
> # 设置提交代码时的用户信息
> $ git config [--global] user.name "[name]"
> $ git config [--global] user.email "[email address]"
> ```

# 自定义配置：

```bash
git config --global core.editor "vim"
git config --global alias.unstage "reset HEAD" 
#chmod产生的变化应该忽略
git config --global core.filemode false
git config --global core.autocrlf true
git config --global gui.encoding utf-8
git config --global core.quotepath false
git config --global color.ui true
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.br branch
git config --global alias.cp cherry-pick
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%ad) %C(bold blue)<%an>%Creset' --abbrev-commit --date=format:'%Y-%m-%d %H:%M:%S'"
```

增加全局git配置文件/etc/gitconfig 

```shell
root@3520f78b5030:/home/imac# cat /etc/gitconfig 
[color]
        ui = true
[alias]
        co = checkout
        ci = commit
        br = branch
        lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cd) %C(white blue bold)<%an>%Creset' --abbrev-commit --date=iso
        st = status
        unstage = reset HEAD
        cp = cherry-pick
        lgg = log --color --graph --pretty=format:'%C(yellow)%d%Creset %s ' --abbrev-commit
        alias = ! git config --get-regexp ^alias\\. | sed -e s/^alias\\.// -e s/\\ /\\ =\\ /
        lggg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%ad) %C(white blue bold)<%an>%Creset' --abbrev-commit
[gui]
        encoding = utf-8
[push]
        default = current
[credential]
        helper = store
[core]
        quotepath = false
        fileMode = false
        sharedRepository = true
```
