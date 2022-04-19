---
title: git高级使用
weight: 3
---
# 版本说明

* git基本使用
| 日期     | 版本 | 修改内容 |
| -------- | ---- | -------- |
| 2021/02/19 | V0.3 | 创建     |

# 个人使用总结

## Merge节点

Git有两种合并：一种是"直进式合并"（fast forward），不生成单独的合并节点；另一种是"非直进式合并"（none fast-forword），会生成单独节点。

前者不利于保持commit信息的清晰，也不利于以后的回滚，建议总是采用后者（即使用`--no-ff`参数）。只要发生合并，就要有一个单独的合并节点。

![img](http://www.ruanyifeng.com/blogimg/asset/201207/bg2012070505.png)![img](http://www.ruanyifeng.com/blogimg/asset/201207/bg2012070506.png)

## push策略

不带任何参数的`git push`，默认只推送当前分支，这叫做simple方式。此外，还有一种matching方式，会推送所有有对应的远程分支的本地分支。Git 2.0版本之前，默认采用matching方法，现在改为默认采用simple方式。如果要修改这个设置，可以采用`git config`命令。

```shell
$ git config --global push.default matching
# 或者
$ git config --global push.default simple
```

## 解决**merge** **和** **rebase** 合并冲突

```shell
#merge 和 rebase 对于 ours 和 theirs 的定义是完全相反的。在 merge 时，ours 指代的是当前分支，theirs 代表需要被合并的分支。而在 rebase 过程中，ours 指向了修改参考分支，theirs 却是当前分支
git checkout --ours src/MyFile.cs.
git checkout --theirs src/MyFile.cs

git checkout HEAD -- src/MyFile.cs
git checkout my_branch -- src/MyFile.cs
git checkout 6a363d8 -- src/MyFile.cs
```

> revert 、rebase  、cherry-pick后面都可以跟  --continue  --skip --abort
>

## cherry-pick使用

cherry-pick (somebody/something) to choose the best people or things from a group and leave those that are not so good

```bash
#62ecb3为分支上的commit
git cherry-pick 62ecb3

#合并多个commit
git cherry-pick A B C D E F

#会把从从版本A（不包含）到B（包含）即（A，B]的版本pull到当前分支
git cherry-pick A..B
git cherry-pick A..B C..D E..F

git cherry-pick --continue
git cherry-pick --quit
git cherry-pick --abort
```

## 打标记：

```bash
#默认给当前分支的HEAD打标记
git tag -a v1.0.23.39 -m "release:v1.0.1" 

#给指定分支的HEAD打标记
git tag -a v1.0.23.39 -m "release:v1.0.1" dev_new_feature

#给指定的commit_id打标记
git tag -a v1.0.23.39 -m "release:v1.0.1" a2701a9  

#上传标记 git push origin tag_name,例如：
git push origin  v1.0.1

#上传所有标记
git push origin --tags
```

## 删除文件

配合.gitignore文件

```bash
#删除 untracked files
 git clean -f 

#连 untracked 的目录也一起删掉
 git clean -fd 

#连 gitignore 的untrack 文件/目录也一起删掉 （慎用，一般这个是用来删掉编译出来的 .o之类的文件用的）
 git clean -xfd 

#在用上述 git clean 前，墙裂建议加上 -n 参数来先看看会删掉哪些文件，防止重要文件被误删
 git clean -nxfd
 git clean -nf
 git clean -nfd
```

# 《pro git》总结

```bash
#查看已经暂存起来的变化
git diff --cached 

#重新提交，合并到上一次提交
git commit –amend

#如果要一次推送所有本地新增的标签上去，可以使用 --tags 选项
git push origin --tags

#若要查看各个分支最后一个提交对象的信息，运行
git branch -v

#查看哪些分支已被并入当前分支
git branch –merged

#查看尚未合并的工作
git branch --no-merged 

#查看master到contrib之间的所有差异提交
git diff master...contrib

#在 git log 后加 -p 选项将展示每次提交的内容差异。

#查看简报
git shortlog --no-merges master --not v1.0.1

#如果你想查找所有从refA 或refB 包含的但是不被refC 包含的提交
git log refA refB ^refC

#提交时将差异分块添加
git add -p 或者git add –patch

#储藏（Stashing）
git stash
git stash list
git stash apply stash@{2}
git stash apply --index
git stash pop
git stash clear

#查看历史提交记录，被删除后不在分支上的commit也能在这里找到
git reflog

#垃圾回收
git fsck --full
```

# SVN迁移到Git

https://blog.axosoft.com/migrating-git-svn/

https://git-scm.com/book/en/v2/Git-and-Other-Systems-Migrating-to-Git

安装工具

```shell
sudo apt-get install git-svn
```

# Git其它用途

## 格式化代码

https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks

使用了git提供的hook     **pre-commit**

```shell
#!/bin/bash
array=`git diff-index --name-only HEAD`

for name in ${array}
do
    extension=${name##*.}
    if [[ ${extension} == "h" || ${extension} == "cpp" || ${extension} == "c" ]]
    then
#        echo "###astyle###"$name
        if [ -f ${name} ]; then
        #去掉window下的^M
        #        fromdos ${name}
        #去掉文件头中的BOM标记
                sed -i '1s/^\xEF\xBB\xBF//' ${name}
        #格式化代码
                astyle --style=ansi -s4 -S -N -L -m0 -M40 -f -U -k1 -W1  -j -xL -n ${name}
        fi
        git add ${name}
    fi
done
```

 创建提交hook软链接文件
     chmod +x pre-commit.sh
     cd .git/hooks/
     ln -s ../../pre-commit.sh pre-commit

## 通过git-web提供的hook发送邮件

原理：

1、当有push event时就触发web hook，发送事件信息到指定的服务端

2、服务端接收到事件，解析事件，构造html页

3、使用smtp将html以邮件发送



![image-20201123090440864](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201123090440864.png)

```python
root@7f438265e3c9:/home/root# cat web_hook_server.py 
#-*- coding:utf-8 -*-
import BaseHTTPServer
import SocketServer
import CGIHTTPServer 
import logging
import json
import urllib
import urlparse
from urlparse import unquote
import send_notify

class RequestHandler(BaseHTTPServer.BaseHTTPRequestHandler):
    '''处理请求并返回页面'''
    def do_POST(self):
        logging.debug('POST %s' % (self.path))
        print( "incomming http: ", self.path )

        content_length = int(self.headers['Content-Length']) # <--- Gets the size of data
        post_data = self.rfile.read(content_length) # <--- Gets the data itself
        #将URL编码方式的字符转换为普通字符串
        js_str = urllib.unquote(post_data[8:])
        send_notify.sendNotify(js_str)
        self.send_response(200)

if __name__ == '__main__':
    serverAddress = ('', 5410)
    SocketServer.TCPServer.allow_reuse_address = True
    server = SocketServer.TCPServer(serverAddress, RequestHandler)
    server.serve_forever()
```



## 解决xls文件在git比较差异

https://stackoverflow.com/questions/17083502/how-to-perform-better-document-version-control-on-excel-files-and-sql-schema-fil

https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes#Binary-Files

https://gitlab.inria.fr/sgilles/homepage/-/wikis/enable-diffing-of-excel-files-with-git

通过指定不同格式的文件读取工具，将工具的输出进行对比

```shell
# -*- coding: utf-8 -*-  
#!/usr/bin/env python

import sys
from mmap import mmap,ACCESS_READ
from xlrd import open_workbook # http://www.python-excel.org/
# 使得 sys.getdefaultencoding() 的值为 'utf-8'  
reload(sys)                      # reload 才能调用 setdefaultencoding 方法  
sys.setdefaultencoding('utf-8')  # 设置 'utf-8' 

file_name=sys.argv[1]

wb = open_workbook(file_name)

s = wb.sheets()[0]

r_count = s.nrows
c_count = s.ncols

for row in range(r_count):
    for col in range(c_count):
        print ("row[%4d],col[%4d],value[%s]" % (row,col,s.cell(row, col).value))
        continue
    sys.stdout.write('\n')
```

## 制作补丁包

将发布的二进制包用git管理之后，可以用git命令导出，两个版本之前的删除、新增和修改文件列表

git diff

```shell
--name-status
Show only names and status of changed files. See the description of the --diff-filter option on what the status letters mean.

--diff-filter=[(A|C|D|M|R|T|U|X|B)…[*]]
Select only files that are Added (A), Copied (C), Deleted (D), Modified (M), Renamed (R), have their type (i.e. regular file, symlink, submodule, …) changed (T), are Unmerged (U), are Unknown (X), or have had their pairing Broken (B). Any combination of the filter characters (including none) can be used. When * (All-or-none) is added to the combination, all paths are selected if there is any file that matches other criteria in the comparison; if there is no file that matches other criteria, nothing is selected.

Also, these upper-case letters can be downcased to exclude. E.g. --diff-filter=ad excludes added and deleted paths.

Note that not all diffs can feature all types. For instance, diffs from the index to the working tree can never have Added entries (because the set of paths included in the diff is limited by what is in the index). Similarly, copied and renamed entries cannot appear if detection for those types is disabled.
```

获取删除文件列表

```shell
git diff --diff-filter=D  --name-only 05e0ccf fd48e04  opt/
```

获取新增和修改文件列表，并拷贝到输出目录output_dir

```shell
git diff old_commit new_commit  --diff-filter=AM --name-only |xargs -t  -i{} cp --parents -d {} output_dir
```

cp选项说明

```shell
--parents     use full source file name under DIRECTORY
#copies the file a/b/c to existing_dir/a/b/c, creating any missing intermediate directories.
#即，当被复制的源文件路径包含子目录名，--parent 选项会在目标目录下自动创建不存在的子目录。目标目录本身必须已经存在。
cp --parents a/b/c existing_dir

-d     same as --no-dereference --preserve=links
#当拷贝软链接时，如果不添加-d选项，则会拷贝软链接对应的文件，加了-d选项后，则直接拷贝这个软链接
```

## mantis plugin集成（未实现）

作用：

开发人员执行commit时，当日志信息中包含Fix #xxxx等信息时，mantis中对应的缺陷会自动修改状态为已修正，已解决，并将日志信息添加到mantis中。减少了开发人员工作量，方便缺陷与代码的跟踪

https://github.com/mantisbt-plugins/source-integration/blob/master/SourceGitlab/README.md

https://noswap.com/blog/integrating-git-svn-with-mantisbt

![si-regexes.png (760×119)](https://noswap.com/blog/files/2009/01/si-regexes.png)

![si-related.png (1005×228)](https://noswap.com/blog/files/2009/01/si-related.png)

![si-browse.png (1006×269)](https://noswap.com/blog/files/2009/01/si-browse.png)

## 使用Git管理WIKI（未实现）

 gollum -- A git-based Wikihttps://github.com/gollum/gollum

gitlab集成了gollum工具

作用：记录项目迭代中的所有文档，集中管理

## CI和CD（未实现）

CI 持续集成（Continuous Integration）

CD 持续交付（Continuous Delivery）

CD 持续部署（Continuous Deployment）

https://www.redhat.com/zh/topics/devops/what-is-ci-cd

![ci-cd-flow-desktop_1](/images/git工具/ci-cd-flow-desktop_1.png)

对于iMAC项目的应用场景：

实现代码提交后，执行编译、打包、FTP上传发布

# git生成patch

```shell
3B-pi@raspberrypi:~/Public/Qin-master/patch $ git lg
* 92770dd - (HEAD -> master, origin/master) style:change handle_Default(int keyId) to handle_Default(int unicode, int keyId) => QinIMTables.h (9 hours ago) <author2>
* c8a0e5a - style:格式化代码 (10 hours ago) <author1>
* f281c86 - fix:完善五笔和仓颉的翻页功能 (10 hours ago) <author2>
* 5e8aecf - style:删除无用文件 (10 hours ago) <author2>
* b807285 - feat: 增加仓颉和五笔输入法 (10 hours ago) <author2>
* 09d81b2 - style:将TableIM(无虾米)从IMBase中分离出来 (10 hours ago) <author2>
* 05657a6 - fix:shift三种状态的转换 (10 hours ago) <author2>
* 1ddf8ed - feat:谷歌拼音输入法实现用户自定义造词逻辑 (11 hours ago) <author1>
* a94fece - feat:谷歌拼音完成选择候选词上屏 (14 hours ago) <author1>
* ff34572 - feat:谷歌拼音增加候选词翻页 (14 hours ago) <author1>
* 1fdcf02 - fix:修改UI文件中指定的字母按键键值 (32 hours ago) <author1>
* 3c86422 - fix:完善谷歌拼音输入法 (32 hours ago) <author1>
* d710041 - feat:候选增加左右按钮完成翻页功能 (7 days ago) <author2>
* cadc079 - feat:增加长按连续输入，长按连续删除。设置setAutoRepeat(true)属性 (7 days ago) <author2>
* b633021 - fix:符号|和\，=和+切换显示有问题 符号&不能显示 (7 days ago) <author2>
* 1940107 - feat:添加谷歌拼音核心代码 (8 days ago) <author1>
* 9ae1441 - chore:整理开源依赖库目录，添加谷歌拼音输入法 (8 days ago) <author1>
* d17947c - feat:增加数字符号切换按钮 (3 weeks ago) <author1>
* 646a407 - fix:解决英文输入下，按空格无效 (3 weeks ago) <author1>
* b588a3b - feat:左右shift按钮状态关联，准备添加左右翻页按钮 (3 weeks ago) <author1>
* 9644ac2 - feat:在虚拟键盘布局中增加虚拟键盘关闭按钮 (4 weeks ago) <author1>
* 3b97463 - (dev_init) init (4 weeks ago) <author1>

#生成区间范围的补丁
git format-patch 3b97463..92770dd

#结果如下
3B-pi@raspberrypi:~/Public/Qin-master/patch $ ls -lth
total 20M
-rw-r--r-- 1 pi pi 1.3K Feb 18 00:08 0021-style-change-handle_Default-int-keyId-to-handle_Defa.patch
-rw-r--r-- 1 pi pi 163K Feb 18 00:08 0020-style.patch
-rw-r--r-- 1 pi pi  32K Feb 18 00:08 0019-fix.patch
-rw-r--r-- 1 pi pi 5.1M Feb 18 00:08 0018-style.patch
-rw-r--r-- 1 pi pi  13M Feb 18 00:08 0017-feat.patch
-rw-r--r-- 1 pi pi  11K Feb 18 00:08 0016-style-TableIM-IMBase.patch
-rw-r--r-- 1 pi pi 7.6K Feb 18 00:08 0015-fix-shift.patch
-rw-r--r-- 1 pi pi 9.2K Feb 18 00:08 0014-feat.patch
-rw-r--r-- 1 pi pi 2.7K Feb 18 00:08 0013-feat.patch
-rw-r--r-- 1 pi pi  14K Feb 18 00:08 0012-feat.patch
-rw-r--r-- 1 pi pi  20K Feb 18 00:08 0011-fix-UI.patch
-rw-r--r-- 1 pi pi 7.7K Feb 18 00:08 0010-fix.patch
-rw-r--r-- 1 pi pi  14K Feb 18 00:08 0008-feat-setAutoRepeat-true.patch
-rw-r--r-- 1 pi pi 5.6K Feb 18 00:08 0009-feat.patch
-rw-r--r-- 1 pi pi 4.0K Feb 18 00:08 0007-fix.patch
-rw-r--r-- 1 pi pi  11K Feb 18 00:08 0006-feat.patch
-rw-r--r-- 1 pi pi 1.4M Feb 18 00:08 0005-chore.patch
-rw-r--r-- 1 pi pi  16K Feb 18 00:08 0004-feat.patch
-rw-r--r-- 1 pi pi 4.5K Feb 18 00:08 0003-fix.patch
-rw-r--r-- 1 pi pi 8.0K Feb 18 00:08 0002-feat-shift.patch
-rw-r--r-- 1 pi pi 7.0K Feb 18 00:08 0001-feat.patch

#应用补丁
git am ~/Public/Qin-master/patch/*.patch
```

# git批量修改author

参考 https://www.git-tower.com/learn/git/faq/change-author-name-email

- Run the following script from terminal after changing the variable values

  ```shell
  #!/bin/sh
  
  git filter-branch --env-filter '
  
  OLD_EMAIL="your-old-email@example.com"
  CORRECT_NAME="Your Correct Name"
  CORRECT_EMAIL="your-correct-email@example.com"
  
  if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
  then
      export GIT_COMMITTER_NAME="$CORRECT_NAME"
      export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
  fi
  if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
  then
      export GIT_AUTHOR_NAME="$CORRECT_NAME"
      export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
  fi
  ' --tag-name-filter cat -- --branches --tags
  ```

- Push the corrected history to GitHub:

  ```shell
  git push --force --tags origin 'refs/heads/*'
  ```

OR if you like to push selected references of the branches then use

```shell
git push --force --tags origin 'refs/heads/develop'
```

# git验证机制更新

https://www.bswen.com/2021/09/others-how-to-solve-github-issue1.html

