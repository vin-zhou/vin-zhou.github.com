title: Mac 端口占用
date: 2014-10-29 17:07:53
tags: Mac
categories: 技术
---
## 查看端口
终端输入：
```
lsof -i tcp:port
```
将port换成被占用的端口(如：8086、9998),或 只使用tcp，则显示所有的tcp端口占用情况

## 杀死进程
```
kill -9 进程号    //强制结束
```
