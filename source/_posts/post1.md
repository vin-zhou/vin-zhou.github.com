title: Markdown学习笔记
date: 2014-10-24 09:25:50
tags: Hexo
categories: 技术
---
##引言
由于使用了Hexo搭建了[个人主站](http://vin-zhou.github.io/)，因此用Markdown进行内容编辑，就成了自然而然的事。这里有一篇文章总结了Markdown的各种牛逼之处：[Markdown写作浅谈](http://www.jianshu.com/p/PpDNMG)。简言之，Markdown作为一种非常简单、易学的文本标记格式语法，可以： 
- 轻松布局，满足文艺青年的写作需求
- 支持Latex、R语言，满足科学青年的paper需求
- 满足技术青年的技术博客需求
今天试了下 Hexo+Github+Markdown， 怎一个爽字了得!

##Markdown语法总结
若想看到Markdown语法的对应显示效果，可以查看
- [Stackedit.io在线编辑器](https://stackedit.io/editor)
- [献给写作者的Markdown新手指南](http://www.jianshu.com/p/q81RER)
- [怎样使用Markdown](http://www.ituring.com.cn/article/23)
这里参考了[Markdown简明语法](http://lutaf.com/markdown-simple-usage.htm)，对语法进行总结，便于日后的查阅
###标题
- 用1~6个#分别代表一级标题~六级标题
- > *斜体*
- > **加粗**
###换行
- 


####来段代码试试
C#:

    //这里显示一些代码，在正文显示中会自动识别语言，进行代码染色，这是一段C#代码
    public class Blog
    {
         public int Id { get; set; }
         public string Subject { get; set; }
    }

Python:

    keywords = ["dsaa","Asd","sadc","Gdfd","gdfdd","gaf","gabdddddd","eg"]
    print dict([(i[0],list(i[1])) for i in groupby(sorted(keywords),lambda    x:x[0].lower())])

Javascript:

    /**
     * nth element in the fibonacci series.
     * @param n >= 0
     * @return the nth element, >= 0.
     */
    function fib(n) {
        var a = 1, b = 1;
        var tmp;
        while (--n >= 0) {
            tmp = a;
            a += b;
           b = tmp;
        }
        return a;
    }

    document.write(fib(10));
##本文主要参考自 [Markdown指南](http://zipperary.com/2013/05/22/introduction-to-markdown/), 感谢原作者的辛勤工作！