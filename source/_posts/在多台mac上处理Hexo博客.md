title: 在多台mac上处理Hexo博客
date: 2014-10-29 15:00:55
tags: Hexo
---
之前利用了 Hexo+Github搭建了自己的博客网站，但问题来了：如何在多台mac上对此博客进行维护？  

这里参考了文章[让你的Octopress博客在多台Mac上同时使用](http://wangzz.github.io/blog/2014/04/02/ru-he-pei-zhi-rang-ni-de-octopressbo-ke-zai-duo-tai-macshang-tong-shi-shi-yong/), 思路与之一致，即对于一个已创建的博客目录，如之前的git_blog，其已通过Hexo建立了public目录下的内容与远程仓库vin-zhou@github.com的master分支的连接关系，通过hexo的hexo deploy命令即可保持更新。  

因此只需要在此仓库vin-zhou@github.com上再创建一个分支如source，将其与git_blog下其他文件如node_modules、themes、scaffolds等（即除了public目录及.gitignore包含的文件外）进行同步绑定。在不同的mac下设置好Hexo环境，通过hexo命令维护master分支，通过git命令维护source分支即可。

下面给出具体步骤：
## 建立关系
（假定创建博客的mac为A，其他另一个mac为B)

* 在A中的git_blog目录下，建立source分支：


```c
$ git branch source // 创建source分支
$ git checkout source // 切换到source分支
$ git remote -v //查看远程分支名字
 // 我的内容为如下，说明已绑定
 // origin    git@github.com:vin-zhou/vin-zhou.github.com.git (fetch)
 // origin	git@github.com:vin-zhou/vin-zhou.github.com.git (push)
$ git push origin source // 将当前git_blog下的内容push到Github上的远程仓库的source分支（会自动创建）上
```

* 在B中建立git_blog目录，安装npm，安装Hexo，添加SSH，然后使用

```c
$ git clone -b source git@github.com:vin-zhou/vin-zhou.github.com.git
```
在B中建立本地的source分支，并与远程的source分支进行了绑定。
git_blog下的vin-zhou.github.com文件夹里保存了和远程source分支相同的内容。

## 使用方法

1>.在任意一台mac(如A)，使用Hexo的new、g、d方法添加、生成、部署新的博客，内容都会被同步自动放到Github的master分支上；  

2>.在1>.之后，保证git当前在git_blog下的source分支下，  
先使用：
```c
$ git pull origin source
```
获得Github的source分支上的最新版本，再使用：

```c
$ git add -u
$ git commit -m ""
$ git push origin source
```
将新内容提交至Github的source分支上。

## 其他说明
* 可以使用 

```c
$ git add -f "" // 强制添加文件/文件夹为tracked状态
```


* 使用

```c
$ git rm --cache "" // 解除文件为tracked状态
$ git rm -r --cache "" // 解除文件夹为tracked状态
```

## 参考

* [让你的Octopress博客在多台Mac上同时使用](http://wangzz.github.io/blog/2014/04/02/ru-he-pei-zhi-rang-ni-de-octopressbo-ke-zai-duo-tai-macshang-tong-shi-shi-yong/)