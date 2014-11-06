title: Hexo搭建个人博客
date: 2014-10-23 15:24:49
tags: [Hexo, iOS]
categories: 技术
---
Hexo折腾了蛮久，总结了一点安装心得。步骤记录如下：

## 安装Hexo
 ```c
$ sudo npm install -g hexo 
 ```
   **注意，要加sudo，否则可能提示无权限访问。(npm: node package manager)**
> 卸载Hexo方法：
  1. terminal至/usr/local/lib
  2. `` $ sudo npm uninstall hexo ``

## 启用Hexo

1. 新建目标文件夹，如git_blog
2.  terminal至git_blog
```c
$ hexo init //  Hexo会在git_blog中建立网站所需要的文件
$ npm install // ** 注意，一定要加这个命令！！！！否则生成的public\index.html文件可能各种空白、错误。**
$ hexo g // 生成网页
$ hexo s // 开启server
```
 打开网址：http://localhost:4000/ ，可以看到HelloWorld的网页后，说明安装成功

## 设置Hexo


### 设置_config.yml文件
```
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
```
### 更换主题  
   这里使用了jacman主题
```c
$ git clone https://github.com/wuchong/jacman.git themes/jacman
```
   相应已修改了_config.yml中的theme值。  
#### 更新主题
```c
$ cd themes/jacman
$ git pull
```
#### 修改代码高亮(code highlight)
   jacman提供的代码高亮风格不是很喜欢，想修改一下。[Pacman](http://yangjian.me/workspace/introducing-pacman-theme/)说由于使用Stylus重写了主题，没有了原来的highligt.js文件，需在主题的/source/css/_base/code.styl文件中直接修改，参考了[Tomorrow-Theme](https//github.com/chriskempson/tomorrow-theme)的tomorrow风格的代码颜色设置，最终修改后的颜色设置为：
```
highlight-background = #ffffff
highlight-current-line = #efefef
highlight-line-numbers = #666
highlight-selection = #d6d6d6
highlight-foreground = #4d4d4c
highlight-comment = #00A86B
highlight-red = #c82829
highlight-orange = #f5871f
highlight-yellow = #eab700
highlight-green = #718c00
highlight-aqua = #3e999f
highlight-blue = #4271ae
highlight-purple = #8959a8
```
Objective-C的高亮效果如下：

```objc
- (void) myMethod
{
    NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
    NSDictionary* myDictionary = [[NSDictionary alloc] initWithObjectsAndKeys:@"asdfasds", nil];
    @try
    {
        [self processDictionary:myDictionary];
    }
    @catch (NSException *exception)
    {
        @throw; // 可再次抛出该异常
    }
    @finally
    {
        [pool release]; // cleanup
    }
}
```

## Hexo常用命令

```c
$ hexo new "new post name" // 创建一个新文章
$ hexo g   // 生成本地网页
$ hexo d   // 部署到github上
   ```
  对于每个post，其head部分的内容示例如下：

```
title: xcode Undefined symbols for architecture i386
date: 2014-10-18 16:33:04
tags: [iOS, XCode]
categories: 技术
```
##  关联Github
可以根据附录的参考文章链接，生成SSH密钥，这里补充一点，最后一步测试  

```c
$ ssh -T git@github.com
```
若出现  `` ssh: connect to host github.com port 22: Connection ``

则可以在 ./.ssh/config文件中（没有，则新建）
```
host github.com
    hostname ssh.github.com
    port 443
```
更改使用新的443访问端口。

## 其他说明

* 在搭建时，出现过需用localhost:4000/public访问资源的问题、以及生成页面空白、css访问不到等问题，出现问题一般尝试用 hexo clean ； hexo g; hexo s;重新生成。
* 如问题仍不能解决，最终采用的是卸载hexo，重新安装配置一次。
* 注意卸载前要先把_config.xml、source、theme等文件备份。

## Homebrew
 安装方法：打开Terminal输入
 
 ```objc
  ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
 ```
 以后即可使用 ``brew``安装管理其所支持的软件(如git\nodejs\等）了。
## 参考

* [如何搭建一个独立博客](http://cnfeat.com/2014/05/10/2014-05-11-how-to-build-a-blog/)

