---
title: git基本使用
weight: 2
---
# 版本说明

* git基本使用
| 日期     | 版本 | 修改内容 |
| -------- | ---- | -------- |
| 2021/02/19 | V0.3 | 创建     |
# Git对比SVN

![Git-vs-SVN](/images/git工具/Git-vs-SVN-02-800x270.png)

## 1.小步提交，互不干扰 

并行开发过程中各开发人员可以随时多次commit代码且互不影响，最后在merage到主分支，并且能记录所有成员的所有commint记录。SVN只能大量的一次性提交到中心库。

## 2.打断开发：在开发新功能过程中，突然需要你去修复一个Bug

使用Git，你可以直接stash/commit当前改动，然后switch到主分支去修复Bug，之后在pop/switch回你原来的分支继续开发。

## 3.Git分支切换-指针移动，SVN分支切换-Copy项目

Git支持本地无限Branches，当我们个体在本地创建多个branches用于不同目的的时候（修改，新增，探索），分支轻量化，秒创分支，创建分支满足客户定制化需求

## 4.Git Tag-指针标示，SVN Tag-Copy项目

Git管理的项目要比SVN小得多。Git初次拉取代码的速度也远小于SVN。

## 5.两级提交

本地创建分支开发，本地提交，需要合并时再提交到远程

## 6.日志查看

Git本地包含了完整的日志，闪电的速度查看（并且无需网络)。SVN需要从服务拉取。

## 7.安全

Git是分布式版本控制系统，每个用户都相当于一份备份， 管理员无需为数据备份而担心。SVN作为集中式版本控制系统，存在单点故障的风险。备份版本库的任务非常繁重。



linus在google的演讲感悟
链接：https://www.zhihu.com/question/19601997/answer/95363587

1. 自洽的、最少依赖的个人工作得到支持。1000多人的Linux开发团队是分布在世界各地的，使用git也就不必依赖中心服务器、不必需要很少的网络。就在自己的电脑上就有完整的仓库，可以做任何版本管理，除了分享代码。SVN显然是不合适的，因为单点故障大家甚至无法提交，更加无法开分支，这是无法忍受的。
2. 剔除害群之马很简单。如果Linus经过观察，发现有些程序员特别容易出漏子，那么封杀的办法就是不必拉取即可。实际上Linus就是这样干过。如果是SVN，就变成了撤销惹麻烦的开发者的账号或者限定他的访问范围，并且从仓库中移除麻烦的代码提交。就是说，封杀的方法在git而言，是不做某事即可，SVN是做一系列事情才可以。一正一反，大家可以体会一下。Linus喜欢前者，并且得心应手。这样的工作流程就避开了很多“政治”问题，让他的集成代码过程变得主动。
3. 可以使用信任网络。Linux太大了，不可能完全看完补丁代码的方式来识别信任，这个Linus曾经干过，最后的结果当然是放弃。如果发现有些程序员特别优秀，他只要选择拉取他们的实现。这些程序员也只是拉取他们信任的程序员的实现。这样的信任网络是可以层次化的，因此对应于1000多人的开发者来说，这样做确实可以通过分层的信任网络达成大规模的团队协作。如果是SVN，我不知道如何做可以更好
4. 轻量的分支开销鼓励大量被使用。对于这样的团队，为了敏捷的迭代，如果有想法就分支（这样的开发隔离想法是很有价值的），那么在svn上分支是海量的并且全局的大家互相影响，因此是要命的。而对于Git总数当然是海量，但是每个人的分支都在自己的仓库内，不会影响到他人。且分支无需连接服务器，因此是飞速的。



# Git工作流

http://www.ruanyifeng.com/blog/2015/12/git-workflow.html

https://www.jianshu.com/p/5e847c12709c

http://www.ruanyifeng.com/blog/2015/12/git-workflow.html


Git 作为一个源码管理系统，不可避免涉及到多人协作。

协作必须有一个规范的工作流程，让大家有效地合作，使得项目井井有条地发展下去。"工作流程"在英语里，叫做"workflow"或者"flow"，原意是水流，比喻项目像水流那样，顺畅、自然地向前流动，不会发生冲击、对撞、甚至漩涡。

都采用["功能驱动式开发"](https://en.wikipedia.org/wiki/Feature-driven_development)（Feature-driven development，简称FDD）。

它指的是，需求是开发的起点，先有需求再有功能分支（feature branch）或者补丁分支（hotfix branch）。完成开发后，该分支就合并到主分支，然后被删除。

| 工作流      | 策略                                                         | 缺点                     |
| ----------- | ------------------------------------------------------------ | ------------------------ |
| Git flow    | 2个长期分支(master\| develop)、3种短期分支(feature \|hotfix \|release) | 经常要切换分支，非常烦人 |
| Github flow | 1个长期分支 master,发起一个[pull request](https://help.github.com/articles/using-pull-requests/)（简称PR） |                          |
| Gitlab flow | "上游优先"（upsteam first）                                  |                          |

对于"版本发布"的项目，建议的做法是每一个稳定版本，都要从`master`分支拉出一个分支，比如`2-3-stable`、`2-4-stable`等等。

以后，只有修补bug，才允许将代码合并到这些分支，并且此时要更新小版本号。



## git flow工作流程：
![2178607-f9cad12439bb7d34](/images/git工具/git-flow.png)
- devloper：新功能开发分支一般命名为（feature_name或feature/name）；开发人员在进行功能开发的时候，先从master 分支checkout新的feature分支（例如feature/open_activity）,分支拉取后，开发人员在当前分支开发自己的功能，开发完成后，merge feature/open_activity到 qa分支进行功能测试，经过QA测试该功能没有问题，merge feature/open_activity 到 release 分支（release 分支不能直接push代码，需要request merge,code review之后，就可以在灰度环境测试，然后上线了），到此为止新的功能已经上线完成，delete feature/open_activity分支、 merge release 分支到master 分支，并且打下tag。整个一个开发流程结束，新的需求一个轮回又开始了；
- bugfix:bugfix分支一般命名为（bugfix_name或bugfix/name）； bugfix 是对重大bug的修改，修改完成后，需要合并到qa分支进行测试，整个流程与 #devloper# 的流程一致；
- hotfix: hotfix分支一般命名为（hotfix_name或hotfix/name）； hotfix为线上代码的热更新，比如配置文件的修改或者极小的功能修改。开发人员从master 拉取hotfix/mysql_config分支，然后进行bug fix ,完成后，merger 到release分支code review之后上线。到此bug热修复完成，delete hotfix/mysql_config分支、merge release 分支到master 分支，hotfix一般不需要打tag；

# Git命令
## Git命令流程图

![preview](https://pic1.zhimg.com/0df9740f9b41522ab12c14dbbf6bbad4_r.jpg?source=1940ef5c)

![img](https://img-blog.csdnimg.cn/20190628180240306.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1YW5zaHVqaW43Nw==,size_16,color_FFFFFF,t_70)

![img](https://img2018.cnblogs.com/blog/1479220/201809/1479220-20180925212913309-536352719.png)

http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html

## 一、新建代码库

> ```bash
> # 在当前目录新建一个Git代码库
> $ git init
> 
> # 新建一个目录，将其初始化为Git代码库
> $ git init [project-name]
> 
> # 下载一个项目和它的整个代码历史
> $ git clone [url]
> ```

## 二、配置

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

## 三、增加/删除文件

> ```bash
> # 添加指定文件到暂存区
> $ git add [file1] [file2] ...
> 
> # 添加指定目录到暂存区，包括子目录
> $ git add [dir]
> 
> # 添加当前目录的所有文件到暂存区
> $ git add .
> 
> # 添加每个变化前，都会要求确认
> # 对于同一个文件的多处变化，可以实现分次提交
> $ git add -p
> 
> # 删除工作区文件，并且将这次删除放入暂存区
> $ git rm [file1] [file2] ...
> 
> # 停止追踪指定文件，但该文件会保留在工作区
> $ git rm --cached [file]
> 
> # 改名文件，并且将这个改名放入暂存区
> $ git mv [file-original] [file-renamed]
> ```

## 四、代码提交

> ```bash
> # 提交暂存区到仓库区
> $ git commit -m [message]
> 
> # 提交暂存区的指定文件到仓库区
> $ git commit [file1] [file2] ... -m [message]
> 
> # 提交工作区自上次commit之后的变化，直接到仓库区
> $ git commit -a
> 
> # 提交时显示所有diff信息
> $ git commit -v
> 
> # 使用一次新的commit，替代上一次提交
> # 如果代码没有任何新变化，则用来改写上一次commit的提交信息
> $ git commit --amend -m [message]
> 
> # 重做上一次commit，并包括指定文件的新变化
> $ git commit --amend [file1] [file2] ...
> ```

## 五、分支

> ```bash
> # 列出所有本地分支
> $ git branch
> 
> # 列出所有远程分支
> $ git branch -r
> 
> # 列出所有本地分支和远程分支
> $ git branch -a
> 
> # 新建一个分支，但依然停留在当前分支
> $ git branch [branch-name]
> 
> # 新建一个分支，并切换到该分支
> $ git checkout -b [branch]
> 
> # 新建一个分支，指向指定commit
> $ git branch [branch] [commit]
> 
> # 新建一个分支，与指定的远程分支建立追踪关系
> $ git branch --track [branch] [remote-branch]
> 
> # 切换到指定分支，并更新工作区
> $ git checkout [branch-name]
> 
> # 切换到上一个分支
> $ git checkout -
> 
> # 建立追踪关系，在现有分支与指定的远程分支之间
> $ git branch --set-upstream [branch] [remote-branch]
> 
> # 合并指定分支到当前分支
> $ git merge [branch]
> 
> # 选择一个commit，合并进当前分支
> $ git cherry-pick [commit]
> 
> # 删除分支
> $ git branch -d [branch-name]
> 
> # 删除远程分支
> $ git push origin --delete [branch-name]
> $ git branch -dr [remote/branch]
> ```

## 六、标签

> ```bash
> # 列出所有tag
> $ git tag
> 
> # 新建一个tag在当前commit
> $ git tag [tag]
> 
> # 新建一个tag在指定commit
> $ git tag [tag] [commit]
> 
> # 删除本地tag
> $ git tag -d [tag]
> 
> # 删除远程tag
> $ git push origin :refs/tags/[tagName]
> 
> # 查看tag信息
> $ git show [tag]
> 
> # 提交指定tag
> $ git push [remote] [tag]
> 
> # 提交所有tag
> $ git push [remote] --tags
> 
> # 新建一个分支，指向某个tag
> $ git checkout -b [branch] [tag]
> ```

## 七、查看信息

> ```bash
> # 显示有变更的文件
> $ git status
> 
> # 显示当前分支的版本历史
> $ git log
> 
> # 显示commit历史，以及每次commit发生变更的文件
> $ git log --stat
> 
> # 搜索提交历史，根据关键词
> $ git log -S [keyword]
> 
> # 显示某个commit之后的所有变动，每个commit占据一行
> $ git log [tag] HEAD --pretty=format:%s
> 
> # 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
> $ git log [tag] HEAD --grep feature
> 
> # 显示某个文件的版本历史，包括文件改名
> $ git log --follow [file]
> $ git whatchanged [file]
> 
> # 显示指定文件相关的每一次diff
> $ git log -p [file]
> 
> # 显示过去5次提交
> $ git log -5 --pretty --oneline
> 
> # 显示所有提交过的用户，按提交次数排序
> $ git shortlog -sn
> 
> # 显示指定文件是什么人在什么时间修改过
> $ git blame [file]
> 
> # 显示暂存区和工作区的差异
> $ git diff
> 
> # 显示暂存区和上一个commit的差异
> $ git diff --cached [file]
> 
> # 显示工作区与当前分支最新commit之间的差异
> $ git diff HEAD
> 
> # 显示两次提交之间的差异
> $ git diff [first-branch]...[second-branch]
> 
> # 显示今天你写了多少行代码
> $ git diff --shortstat "@{0 day ago}"
> 
> # 显示某次提交的元数据和内容变化
> $ git show [commit]
> 
> # 显示某次提交发生变化的文件
> $ git show --name-only [commit]
> 
> # 显示某次提交时，某个文件的内容
> $ git show [commit]:[filename]
> 
> # 显示当前分支的最近几次提交
> $ git reflog
> ```

## 八、远程同步

> ```bash
> # 下载远程仓库的所有变动
> $ git fetch [remote]
> 
> # 显示所有远程仓库
> $ git remote -v
> 
> # 显示某个远程仓库的信息
> $ git remote show [remote]
> 
> # 增加一个新的远程仓库，并命名
> $ git remote add [shortname] [url]
> 
> # 取回远程仓库的变化，并与本地分支合并
> $ git pull [remote] [branch]
> 
> # 上传本地指定分支到远程仓库
> $ git push [remote] [branch]
> 
> # 强行推送当前分支到远程仓库，即使有冲突
> $ git push [remote] --force
> 
> # 推送所有分支到远程仓库
> $ git push [remote] --all
> ```

## 九、撤销

> ```bash
> # 恢复暂存区的指定文件到工作区
> $ git checkout [file]
> 
> # 恢复某个commit的指定文件到暂存区和工作区
> $ git checkout [commit] [file]
> 
> # 恢复暂存区的所有文件到工作区
> $ git checkout .
> 
> # 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
> $ git reset [file]
> 
> # 重置暂存区与工作区，与上一次commit保持一致
> $ git reset --hard
> 
> # 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
> $ git reset [commit]
> 
> # 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
> $ git reset --hard [commit]
> 
> # 重置当前HEAD为指定commit，但保持暂存区和工作区不变
> $ git reset --keep [commit]
> 
> # 新建一个commit，用来撤销指定commit
> # 后者的所有变化都将被前者抵消，并且应用到当前分支
> $ git revert [commit]
> 
> # 暂时将未提交的变化移除，稍后再移入
> $ git stash
> $ git stash pop
> ```

## 十、其他

> ```bash
> # 生成一个可供发布的压缩包
> $ git archive
> ```

# Git 使用规范流程

http://www.ruanyifeng.com/blog/2015/08/git-use-process.html

## 第一步：新建分支

首先，每次开发新功能，都应该新建一个单独的分支

> ```bash
> # 获取主干最新代码
> $ git checkout master
> $ git pull
> 
> # 新建一个开发分支myfeature
> $ git checkout -b myfeature
> ```

## 第二步：提交分支commit

分支修改后，就可以提交commit了。

> ```bash
> $ git add --all
> $ git status
> $ git commit --verbose
> ```

git add 命令的all参数，表示保存所有变化（包括新建、修改和删除）。从Git 2.0开始，all是 git add 的默认参数，所以也可以用 git add . 代替。

git status 命令，用来查看发生变动的文件。

git commit 命令的verbose参数，会列出 [diff](http://www.ruanyifeng.com/blog/2012/08/how_to_read_diff.html) 的结果。

## 第三步：撰写提交信息

提交commit时，必须给出完整扼要的提交信息，下面是一个范本。

> ```bash
> Present-tense summary under 50 characters
> 
> * More information about commit (under 72 characters).
> * More information about commit (under 72 characters).
> 
> http://project.management-system.com/ticket/123
> ```

第一行是不超过50个字的提要，然后空一行，罗列出改动原因、主要变动、以及需要注意的问题。最后，提供对应的网址（比如Bug ticket）。

## 第四步：与主干同步

分支的开发过程中，要经常与主干保持同步。

> ```bash
> $ git fetch origin
> $ git rebase origin/master
> ```

## 第五步：合并commit

分支开发完成后，很可能有一堆commit，但是合并到主干的时候，往往希望只有一个（或最多两三个）commit，这样不仅清晰，也容易管理。

那么，怎样才能将多个commit合并呢？这就要用到 git rebase 命令。

> ```bash
> $ git rebase -i origin/master
> ```

git rebase命令的i参数表示互动（interactive），这时git会打开一个互动界面，进行下一步操作。

下面采用[Tute Costa](https://robots.thoughtbot.com/git-interactive-rebase-squash-amend-rewriting-history)的例子，来解释怎么合并commit。

> ```bash
> pick 07c5abd Introduce OpenPGP and teach basic usage
> pick de9b1eb Fix PostChecker::Post#urls
> pick 3e7ee36 Hey kids, stop all the highlighting
> pick fa20af3 git interactive rebase, squash, amend
> 
> # Rebase 8db7e8b..fa20af3 onto 8db7e8b
> #
> # Commands:
> #  p, pick = use commit
> #  r, reword = use commit, but edit the commit message
> #  e, edit = use commit, but stop for amending
> #  s, squash = use commit, but meld into previous commit
> #  f, fixup = like "squash", but discard this commit's log message
> #  x, exec = run command (the rest of the line) using shell
> #
> # These lines can be re-ordered; they are executed from top to bottom.
> #
> # If you remove a line here THAT COMMIT WILL BE LOST.
> #
> # However, if you remove everything, the rebase will be aborted.
> #
> # Note that empty commits are commented out
> ```

上面的互动界面，先列出当前分支最新的4个commit（越下面越新）。每个commit前面有一个操作命令，默认是pick，表示该行commit被选中，要进行rebase操作。

4个commit的下面是一大堆注释，列出可以使用的命令。

> - pick：正常选中
> - reword：选中，并且修改提交信息；
> - edit：选中，rebase时会暂停，允许你修改这个commit（参考[这里](https://schacon.github.io/gitbook/4_interactive_rebasing.html)）
> - squash：选中，会将当前commit与上一个commit合并
> - fixup：与squash相同，但不会保存当前commit的提交信息
> - exec：执行其他shell命令

上面这6个命令当中，squash和fixup可以用来合并commit。先把需要合并的commit前面的动词，改成squash（或者s）。

> ```bash
> pick 07c5abd Introduce OpenPGP and teach basic usage
> s de9b1eb Fix PostChecker::Post#urls
> s 3e7ee36 Hey kids, stop all the highlighting
> pick fa20af3 git interactive rebase, squash, amend
> ```

这样一改，执行后，当前分支只会剩下两个commit。第二行和第三行的commit，都会合并到第一行的commit。提交信息会同时包含，这三个commit的提交信息。

> ```bash
> # This is a combination of 3 commits.
> # The first commit's message is:
> Introduce OpenPGP and teach basic usage
> 
> # This is the 2nd commit message:
> Fix PostChecker::Post#urls
> 
> # This is the 3rd commit message:
> Hey kids, stop all the highlighting
> ```

如果将第三行的squash命令改成fixup命令。

> ```bash
> pick 07c5abd Introduce OpenPGP and teach basic usage
> s de9b1eb Fix PostChecker::Post#urls
> f 3e7ee36 Hey kids, stop all the highlighting
> pick fa20af3 git interactive rebase, squash, amend
> ```

运行结果相同，还是会生成两个commit，第二行和第三行的commit，都合并到第一行的commit。但是，新的提交信息里面，第三行commit的提交信息，会被注释掉。

> ```bash
> # This is a combination of 3 commits.
> # The first commit's message is:
> Introduce OpenPGP and teach basic usage
> 
> # This is the 2nd commit message:
> Fix PostChecker::Post#urls
> 
> # This is the 3rd commit message:
> # Hey kids, stop all the highlighting
> ```

[Pony Foo](http://ponyfoo.com/articles/git-github-hacks)提出另外一种合并commit的简便方法，就是先撤销过去5个commit，然后再建一个新的。

> ```bash
> $ git reset HEAD~5
> $ git add .
> $ git commit -am "Here's the bug fix that closes #28"
> $ git push --force
> ```

squash和fixup命令，还可以当作命令行参数使用，自动合并commit。

> ```bash
> $ git commit --fixup  
> $ git rebase -i --autosquash 
> ```

这个用法请参考[这篇文章](http://fle.github.io/git-tip-keep-your-branch-clean-with-fixup-and-autosquash.html)，这里就不解释了。

## 第六步：推送到远程仓库

合并commit后，就可以推送当前分支到远程仓库了。

> ```bash
> $ git push --force origin myfeature
> ```

git push命令要加上force参数，因为rebase以后，分支历史改变了，跟远程分支不一定兼容，有可能要强行推送（参见[这里](http://willi.am/blog/2014/08/12/the-dark-side-of-the-force-push/)）。

## 第七步：发出Pull Request

提交到远程仓库以后，就可以发出 Pull Request 到master分支，然后请求别人进行代码review，确认可以合并到master。

# 个人总结

1、修改的代码每次提交commit后，这个commit就像组装好的零件，在之后可以进行任意拼装或拆分

2、大胆创建、修改分支，随意commit，不用关心冲突问题，所有的修改全部上传自己独有的临时分支，commit原则：一个功能、一个模块、一个子缺陷上传，不用在乎commit数量的多少，主要是为了之后回滚方便

3、还有很多强大的高级功能需要慢慢探索，最好是遇到需求再去搜索能否用git实现

