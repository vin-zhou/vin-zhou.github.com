title: Git使用小记
date: 2014-10-29 16:15:44
tags: Git
---

## 设置user.name和user.email

```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```
### 查看设置结果
```
$ git config --global user.name 
$ git config --global user.email
```
### 显示当前路径
```
    $ pwd
```
## 提交修改
```
$ git add .
$ git add -u
$ git commit -m " XX comments "
```
## 查看各版本提交记录
```
$ git log   // 查看详细版本日志
$ git log --pretty=oneline  // 查看简略版本日志
```
  **查看操作记录(包含版本commit提交记录）**
```
$ git reflog
```
## 版本恢复
```
$ git reset --hard // 查看当前的版本号
$ git reset --hard XX // 恢复至版本 XX 
```
##  工作区和暂存区

* 工作区（Working Directory）：设定的git_work目录, 存放了各物理文件；
* 版本库（Repository）：.git 隐藏目录。 版本库中包含了暂存区（stage或index），一个自动创建的分支master，以及一个指向master的指针head。
```
$ git add  // 是将文件修改添加到暂存区中；
$ git commit  // 是将暂存区中内容提交到当前分支中。
$ git diff HEAD -- XX_file_name  // 可以查看工作区和版本库中文件最新版本（HEAD来表示）的区别
```

## 撤销文件的修改
```
$ git checkout -- XX_file_name  // 注意 -- 不能丢，否则为新建一个分支的命令
```
此时文件xx_file_name在工作区中所做的修改将会被撤销，文件内容根据以下2中情况进行回退：
    1. 如果当前暂存区中有该文件，则回退到暂存区的版本；
    2. 如果没有，则回退到版本库中的最新版本。

## 撤销文件的add操作
```
$ git reset HEAD XX_file_name  // 将文件XX_file_name在暂存区的最新版本重新放回工作区，撤销 add
```
* 场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。
* 场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD file，就回到了场景1，第二步按场景1操作。
* 场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。

## 删除文件
```
$ rm XX_file_name
$ git rm XX_file_name  // 版本库中删除
$ git checkout -- XX_file_name // 版本库中恢复
```

## 创建分支
```
$ git branch XX_branch_name // 创建分支
$ git checkout XX_branch_name // 切换到分支 XX_branch_name
或者
$ git checkout -b  XX_branch_name // 创建并切换分支
```
### 查看当前各分支及状态
```
$ git branch
$ git checkout XX_branch_name // 切换到分支 XX_branch_name
```
## 合并分支

```
$ git merge XX_branch_name // 合并分支 XX_branch_name 到当前分支
```

## 删除分支
```
$ git branch -d XX_branch_name // 删除分支
```
## 分支的管理策略

在实际开发中，我们应该按照几个基本原则进行分支管理：

1. 有一个master分支保持非常稳定，也就是仅用来发布新版本，平时不能在上面干活；
2. 有一个主dev分支用于开发，是不稳定的，修改都在dev上。只是到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；
3. 你和你的小伙伴们每个人都有自己的dev分支，时不时地往主dev分支上合并就可以了。

如下图所示：



   **修改Bug和添加Feature时，都应当新建一个对应的Dev分支，待改动结束后，merge到主dev分支上。**

## 多人协作

查看远程库信息：
```
$ git remote -v // 查看
$ git remote add origin_blog git@github.com:vin-zhou/vin-zhou.github.com.git // 添加远程分支
$ git remote remove origin_blog // 删除远程分支
``` 
 
 多人协作的工作模式通常是这样：

   （1）首先，可以试图用git push origin branch-name推送自己的修改；
   （2）如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；
   （3）如果合并有冲突，则解决冲突，并在本地提交；
   （4）没有冲突或者解决掉冲突后，再用git push origin branch-name推送就能成功！

   如果git pull提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream branch-name origin/branch-name。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。


## 参考
* 笔记参考自[Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)，在此对原作者表示感谢！
* [如何高效利用Github](http://www.yangzhiping.com/tech/github.html)
* [学习Git](http://yangjian.me/learning/learning-git/)
## 补充：

 1. 强制添加文件 ：git add -f   ， 将会无视.gitignore文件，将目录下所有文件都添加进来。
 2. git 忽略文件  讲忽略文件的[好文章](http://cwind.iteye.com/blog/1666646)
 3. 使用  git rm --cached file_name  ，  会将该file_name从tracked中删除，不予跟踪。
 4. 讲版本控制的[好文章](http://sjtuecho.blog.163.com/blog/static/2050120742013011944140)
 