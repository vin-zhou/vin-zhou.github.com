title: Hexo搭建个人博客
date: 2014-10-23 15:24:49
tags: Hexo
---
Hexo折腾了好久，总结了一点安装心得。步骤记录如下：

###1> 安装Hexo
* 
 ```c
$ sudo npm install -g hexo 
 ```

   **注意，要加sudo，否则提示无权限访问。(npm: node package manager)**
> 卸载Hexo方法：
  * terminal至/usr/local/lib
  * 
    ```c
$ sudo npm uninstall hexo
    ```

###2> 启用Hexo

1. 新建目标文件夹，如git_blog
* terminal至git_blog
* 
  ```c
$ hexo init
  ```
  Hexo会在git_blog中建立网站所需要的文件
* 
   ```c
$ npm install
   ```
  **注意，一定要加这个命令！！！！否则生成的public\index.html文件可能各种空白、错误。**

*  ```c
$ hexo g 
$ hexo s
   ```
  生成网页; 开启server
* 打开网址：http://localhost:4000/ ，可以看到HelloWorld的网页后，说明安装成功

###3> 设置Hexo


  * 设置_config.yml文件
 
  

    # Hexo Configuration
    ## Docs: http://hexo.io/docs/configuration.html
    ## Source: https://github.com/hexojs/hexo/

    # Site
    title: 没有什么能够阻挡
    subtitle:
    description: tech
    author: Vin Zhou
    email:  
    language: zh-CN

    # URL
    ## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
    url: http://vin-zhou.github.com
    root: /
    permalink: :year/:month/:day/:title/
    tag_dir: tags
    archive_dir: archives
    category_dir: categories
    code_dir: downloads/code
    permalink_defaults: 

    # Directory
    source_dir: source
    public_dir: public 

    # Writing
    new_post_name: :title.md # File name of new posts
    default_layout: post
    titlecase: false # Transform title into titlecase
    external_link: true # Open external links in new tab
    filename_case: 0
    render_drafts: false
    post_asset_folder: false
    relative_link: false
    highlight:
      enable: true
      line_number: true
      tab_replace:

    # Category & Tag
    default_category: uncategorized
    category_map:
    tag_map:

    # Archives
    ## 2: Enable pagination
    ## 1: Disable pagination
    ## 0: Fully Disable
    archive: 1
    category: 1
    tag: 1

    # Server
    ## Hexo uses Connect as a server
    ## You can customize the logger format as defined in
    ## http://www.senchalabs.org/connect/logger.html
    port: 4000
    server_ip: localhost
    logger: false
    logger_format: dev

    # Date / Time format
    ## Hexo uses Moment.js to parse and display date
    ## You can customize the date format as defined in
    ## http://momentjs.com/docs/#/displaying/format/
    date_format: MMM D YYYY
    time_format: H:mm:ss

    # Pagination
    ## Set per_page to 0 to disable pagination
    per_page: 10
    pagination_dir: page

    # Disqus
    disqus_shortname:

    # Extensions
    ## Plugins: https://github.com/hexojs/hexo/wiki/Plugins
    ## Themes: https://github.com/hexojs/hexo/wiki/Themes
    theme: jacman
    exclude_generator:
    Plugins:
    - hexo-generator-feed
    - hexo-generator-sitemap

    # Markdown
    ## https://github.com/chjj/marked
    markdown:
    gfm: true
    pedantic : false
    sanitize: false
    tables: true
    breaks: true
    smartLists: true
    smartpants: true    

    # Stylus
    stylus:
    compress: true    
    

    # Deployment
    ## Docs: http://hexo.io/docs/deployment.html
    deploy:
      type: github
      repository: git@github.com:vin-zhou/vin-zhou.github.com.git
      branch: master

*  更新主题  
   这里使用了jacman主题
```c
$ git clone https://github.com/wuchong/jacman.git themes/jacman
```
   相应已修改了_config.yml中的theme值。  
   更新主题:
```c
$ cd themes/jacman
$ git pull
```

 
###4> Hexo常用命令

*  ```c
$ hexo new "new post name" // 创建一个新文章
   ```
*  ```c
$ hexo g   // 生成本地网页
   ```
*  ```c
$ hexo d   // 部署到github上
   ```
###5> 其他说明

* 在搭建时，出现过需用localhost:4000/public访问资源的问题、以及生成页面空白、css访问不到等问题，出现问题一般尝试用 hexo clean ； hexo g; hexo s;重新生成。如问题仍不能解决，最终采用的是卸载hexo，重新安装配置一次。
  注意要先把_config.xml、source、theme等文件备份。


## 参考

* [如何搭建一个独立博客](http://cnfeat.com/2014/05/10/2014-05-11-how-to-build-a-blog/)

