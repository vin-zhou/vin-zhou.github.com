title: Vim 使用手册
tags: Vim
categories: 技术
date: 2015-03-25 17:03:03
---
##引言

本文是对文章[程序猿系列：简明Vim练级攻略](http://www.acfun.tv/v/ac421874)的整理，感谢原作者的辛勤工作。

##第一级- 存活

命令 |  说明
------ |-------
i | Insert模式，按ESC回到Normal模式
x | 删当前光标所在的一个字符
dd | 删除当前行，并把删除的行存到剪贴板里
p | 粘贴剪贴板
:wq| 存盘 + 退出 (:w 存盘, :q 退出)   （陈皓注：:w 后可以跟文件名）
hjkl | 移动光标

VIM的Normal模式下，所有的键就是功能键了

##第二级- 感觉良好

###插入模式
命令 |  说明
------ |-------
i | 进入Normal模式，1	在光标后插入
shift + i | 在行首（非空非TAP）处插入
a | 在光标后插入
A | 在光标所在行末尾插入
o  | 在当前行后插入一个新行
O | 在当前行前插入一个新行

###移动光标

命令 |  说明
------ |-------
0 | 数字零，到行头
^ | 到本行第一个不是blank字符的位置（所谓blank字符就是空格，tab，换行，回车等）
$ | 到本行行尾
g_ | 到本行最后一个不是blank字符的位置
/pattern | 搜索 pattern 的字符串（陈皓注：如果搜索出多个匹配，可按n键到下一个， N到上一个）
%Number G 或 :N , enter | 光标移动到到第 N 行
gg | 到第一行。（陈皓注：相当于1G，或 :1）
G | 到最后一行
w | 到下一个单词的开头
e | 到下一个单词的结尾

### 选择

V | 选择一行


###拷贝粘贴 

（陈皓注：p/P都可以，p是表示在当前位置之后，P表示在当前位置之前）

命令 |  说明
------ |-------
P | 粘贴 
yy | 拷贝当前行， 相当于 ddP

###Undo/Redo
命令 |  说明
------ |-------
u |	undo
Ctr+r | redo

###更改字符
命令 |  说明
------ |-------
rx |	x为当前光标下要更改成的任意字符
~ | 更改当前光标下的字符的大小写

###打开/保存/退出/改变文件(Buffer)
命令 |  说明
------ |-------
:e path/to/file  |	打开一个文件
:w | 存盘
:saveas path/to/file | 另存为 path/to/file
:x， ZZ 或 :wq | 保存并退出 (:x 表示仅在需要时保存，ZZ不需要输入冒号并回车)
:q! | 退出不保存 
:qa! | 强行退出所有的正在编辑的文件，就算别的文件有更改。
:bn 和 :bp | 你可以同时打开很多文件，使用这两个命令来切换下一个或上一个文件。（陈皓注：我喜欢使用:n到下一个文件）

##第三级- 更好，更强，更快

###更好
命令 |  说明
------ |-------
. | (小数点) 可以重复上一次的命令
N *[command]* | 重复某个命令N次
```
下面是一个示例，找开一个文件你可以试试下面的命令：
2dd → 删除2行
3p → 粘贴文本3次
100idesu [ESC] → 会写下 “desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu desu “
. → 重复上一个命令—— 100 “desu “.
3. → 重复 3 次 “desu” (注意：不是 300，你看，VIM多聪明啊).
```

###更强
命令 |  说明
------ |-------
% | 匹配括号移动，包括 (, {, [. （陈皓注：你需要把光标先移到括号上）
* | 匹配光标当前所在的单词，移动光标到下一个匹配单词
# | 匹配光标当前所在的单词，移动光标到上一个匹配单词
（Ctrl + f） | 光标向下翻一页(Forward)
（Ctrl + b） | 光标向上翻一页(Forward)
（Ctrl + d） | 光标向下翻半页(Forward)
（Ctrl + u） | 光标向上翻半页(Upward)
（Ctrl + o） | Insert模式切换为Normal模式
zz | 将光标所在位置屏幕居中
zt | 将光标所在位置屏幕居顶部（Top）
zb | 将光标所在位置屏幕居底部（Bottom）

###更快
你一定要记住光标的移动，因为很多命令都可以和这些移动光标的命令连动。很多命令都可以如下来干：

（start position）（command）（end position）

例如 0y$ 命令意味着：

0 → 先到行头
y → 从这里开始拷贝
$ → 拷贝到本行最后一个字符
你可可以输入 ye，从当前位置拷贝到本单词的最后一个字符。

你也可以输入 y2/foo 来拷贝2个 “foo” 之间的字符串。

还有很多时间并不一定你就一定要按y才会拷贝，下面的命令也会被拷贝：

d (删除 )
v (可视化的选择)
gU (变大写)
gu (变小写)
等等
（陈皓注：可视化选择是一个很有意思的命令，你可以先按v，然后移动光标，你就会看到文本被选择，然后，你可能d，也可y，也可以变大写等）

####比如删除

命令 |  说明
------ |-------
dl 或 x | 删除光标下的当前字符 
dd |	删除一行
dw | 删除至光标后的一个单词的剩余部分
d3w | 删除至光标后的第三个单词的剩余部分
db | 删除至光标前单词的开始位置
d$ 或 D| 删除至行尾
d0 | 删除至行首
d^ | 删除至行首的第一个字符位置（保留空格或TAB字符）
ndd | 删除当前行及其后 n-1 行
d{，}，（，） |删除至下一个{，}，（，）
dgg |删除至文章头
dG | 删除至文章尾


##第四级- Vim 超能力

块操作: （C-v）
块操作，典型的操作： 0 （C-v） （C-d） I-- [ESC]

^ → 到行头
（C-v） → 开始块操作
（C-d） → 向下移动 (你也可以使用hjkl来移动光标，或是使用%，或是别的)
I-- [ESC] → I是插入，插入“--”，按ESC键来为每一行生效。

由于涉及比较多的图片和GIF动画，请参考原文。

##参考

*[程序猿系列：简明Vim练级攻略](http://www.acfun.tv/v/ac421874)

*[Vi/Vim 删除及其他指令](http://lxs647.iteye.com/blog/1245948)