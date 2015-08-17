title: 启用Mac自带的Apache
date: 2014-10-27 16:48:40
tags: iOS
categories: 技术
---

## 启动Apache

1.打开 terminal, 输入

```javascript
  sudo apachectl -v // 查看内置的Apache版本
```
2.接着启动Apache

```javascript
  sudo apachectl start // 启动
```
3.查看Apache是否成功启用

```javascript
  sudo apachectl -t  // 查看情况
```
 
其他命令：

```javascript
  sudo apachectl restart // 重启动

  sudo apachectl stop // 关闭
```

4.在浏览器中输入 “http://localhost”, 可以看到内容为“It works!”的页面。其位于“/Library/WebServer/Documents/”下，**这就是Apache的默认根目录。**   

5.**Apache的安装目录在：/etc/apache2/**，etc默认是隐藏的。有三种方式查看：

* dock下右键Finder，选择"前往文件夹"，输入"/etc"
* 在finder下－－－－》前往－－－》前往文件夹，然后输入/etc
* 可以在terminal 输入 "open /etc"

## 设置虚拟主机

1.在终端运行“sudo vi /etc/apache2/httpd.conf”，打开Apche的配置文件

2.在httpd.conf中找到“#Include /private/etc/apache2/extra/httpd-vhosts.conf”，去掉前面的“＃”，保存并退出。

3.运行“sudo apachectl restart”，重启Apache后就开启了虚拟主机配置功能。  

4.运行“sudo vi /etc/apache2/extra/httpd-vhosts.conf”，就打开了配置虚拟主机文件httpd-vhost.conf，配置虚拟主机了。需要注意的是该文件默认开启了两个作为例子的虚拟主机：

```
<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host.example.com
    DocumentRoot "/usr/docs/dummy-host.example.com"
    ServerName dummy-host.example.com
    ErrorLog "/private/var/log/apache2/dummy-host.example.com-error_log"
    CustomLog "/private/var/log/apache2/dummy-host.example.com-access_log" common
</VirtualHost>

<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host2.example.com
    DocumentRoot "/usr/docs/dummy-host2.example.com"
    ServerName dummy-host2.example.com
    ErrorLog "/private/var/log/apache2/dummy-host2.example.com-error_log"
    CustomLog "/private/var/log/apache2/dummy-host2.example.com-access_log" common
</VirtualHost> 
```

而实际上，这两个虚拟主机是不存在的，在没有配置任何其他虚拟主机时，可能会导致访问localhost时出现如下提示:  

Forbidden
You don't have permission to access /index.php on this server  

最简单的办法就是在它们每行前面加上#，注释掉就好了，这样既能参考又不导致其他问题。

5.增加如下配置
```
<VirtualHost *:80>
    DocumentRoot "/Library/WebServer/Documents"
    ServerName localhost
    ErrorLog "/private/var/log/apache2/localhost-error_log"
    CustomLog "/private/var/log/apache2/localhost-access_log" common
</VirtualHost> 

<VirtualHost *:80>
    DocumentRoot "/Users/XXX_my_username/Sites/MicroStrategy/"
    ServerName mysites
    ErrorLog "/private/var/log/apache2/sites-error_log"
    CustomLog "/private/var/log/apache2/sites-access_log" common
    <Directory "/Users/XXX_my_username/Sites/MicroStrategy/">
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Require all granted
    </Directory>
</VirtualHost>
```

保存退出，并重启Apache。

**注意： 如果不加 ·<\Directory/>·进行权限设置，则可能会报 " Forbidden 403, You don’t have permission to access /mysites/ on this server "错误。**

6.运行“sudo vi /etc/hosts”，打开hosts配置文件，加入"127.0.0.1 mysites"，这样就可以配置完成sites虚拟主机了，可以访问“http://mysites” 了

7.在10.8之前Mac OS X版本其内容和“http://localhost/~[用户名]” 完全一致。


##参考文章：
1. [Mac OS X中配置Apache](http://www.cnblogs.com/snandy/archive/2012/11/13/2765381.html), 感谢原作者的辛勤工作。
