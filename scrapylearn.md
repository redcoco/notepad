# 第1章 课程介绍 #
----------
### 基于数据爬取 数据分析服务 医疗病例分析 互联网金融 自然语言处理 数据建模 信息聚类  ###
----------

### 知识体系 学习流程 ###
1. 环境配置和基础知识铺垫
2. 爬取真实数据
3. scrapy 突破反爬虫技术
4. scrapy 进阶
5. scrapy redis 分布式爬虫
6. elasticsearch  django 实现搜索引擎

[*可以互相对照cnblogs.com参考日志，与本文类似*](http://www.cnblogs.com/adc8868/category/829451.html)


----------
###  爬虫基础知识 ###
1. 正则表达式
2. 深度优先和广度优先遍历算法
3. url 去重的常见策略

----------
### scrapy spider item item loader pipeline feed export crawlspider ###

----------
### scrapy 进阶开发 ###

    scrapy的原理 
    基于scrapy 的中间件开发
    动态网站的抓取处理
    将selenium和phantomjs 集成到scrapy中
    scrapy log配置
    email 发送
    scrapy 信号
    
----------
###  课程体验 ###

-  开发爬虫所需要用到的技术以及网站分析技巧
-  理解scrapy的原理和所有组件的使用以及分布式爬虫scrapy-redis的使用和原理
-  理解分布式开源搜索引擎elasticsearch 的使用以及搜索引擎的原理
-  体验Django 如何快速搭建网站


#  第2章 windows下搭建开发环境 #

----------
### IDE  pycharm的安装使用
### 数据库  mysql和navicat的安装和使用
### windows 和linux下安装python2 和python3
### virtualenv 和 virtualenvwrapper虚拟环境的安装和配置
* pip install -i https://pypi.douan.com/simple/ virtualenv
* pip uninstall virtualenv
* pip install -i https://pypi.douan.com/simple/ virtualenvwrapper
* mkvirtualenv  --python = 路径 ppp     
* workon ppp 
* deactivate

----------

# 第3章 爬虫基础知识回顾

----------
### 技术选型 爬虫能做什么

#### scrapy vs  requests ＋beautifulsoup ####

1. requests 和 beautifulsoup 都是库，scrapy 是框架
2. scrapy 框架中可以加入requests 和 beautifulsoup
3. scrapy 基于twisted ，性能是最大优势
4. scrapy 方便扩展，提供了很多内置的功能
5. scrapy 内置的css和xpath selector 非常方便，beautifulsoup 最大的缺点是慢

#### 网页分类 ####

1. 静态网页
2. 动态网页
3. webservice（restapi）
####  爬虫能做什么 ####
1. 搜索引擎 -- 百度。google、垂直领域搜索引擎
2. 推荐引擎 -- 今日头条
3. 机器学习的数据样本
4. 数据分析（金融数据分析）、舆情分析等

----------
### 正则表达式
####  为什么必须会正则表达式 ####
例如提取`<span> 1 天前</span>` 的 1 就需要使用正则表达式

----------

1. 特殊字符
    > 1) ^ $ * ? + {2} {2,} {2,5} |
    > 
    > 2) [] [^] [a-z] .
    > 
    > 3) \s \S \w \W
    > 
    > 4) [\u4E00-\u9FA5] ()  \d
    
2. 正则表达式的简单应用及python示例
----------
>  ^ 以 什么开头 
>  
>  $以什么结尾 
>  
>  *重复任意多次（0次以上） 
>  
>  .任意字符 
>  
>  ^by.*123$ 以by开头 中间任意字符重复任意多次 并以123结尾 
>  
>  （）提取子字符串  
>  
>  ？非贪婪匹配从右到左
> 
> booobb .*(b.*b).* 提取出bb .*?(b.*?b).* 提取出boob .*?(b.*b).*提取出boobb
> 
> import re
>
> str = "boobby123"
> 
> regex_str = ".*?(b.*?b).*"
> 
> match_obj = re.match(regex_str,str)
> 
> if match_obj:
> 
> print(match_obj.group(1))
> 
> + 至少出现一次 
> 
> {2} 两次 {2,} 两次以上 {2,5} 两到五次 
> 
> 上例 可以改为 .*(b.+b).*  | 或 "bobby|123"  匹配bobby 或者123 
> 
> [abd]满足括号里的任何一个 电话号码匹配 "1[34578][0-9]{9}" [^0-9] 不为数字 [^a] 不为a的任意字符 
> 
> \s 空格 \S 不为空格 \w 等同[A-Za-z0-9_] \W与之相反的含义 [\u4E00-\u9FA5] 汉字    /d 数字



----------
> 
> import re
> 
> line = "xxx出生于2001年6月1日"
> 
> line = "xxx出生于2001/6/1"
> 
> line = "xxx出生于2001-6-1"
> 
> line = "xxx出生于2001-06-01"
> 
> line = "xxx出生于2001-06"
> 
> 
> regex_str = ".*出生于(\d{4}[年/-]\d{1,2}([月/-]\d{1,2}|[日/-]$|$))"
> 
> match_obj = re.match(regex_str,line)
> 
> if match_obj:
> 
> print(match_obj.group(1))


----------
#  深度优先和广度优先原理 #

> 深度优先实现
> 
> def depth_tree(tree_node):
> 
>     if tree_node is not None:
>     
>         print(tree_node._data)
>         
>         if tree_node._left is not None:
>         
>             return depth_tree(tree_node._left)
>             
>         if tree_node._right is not None:
>         
>             return depth_tree(tree_node._right)
> 
> 
> 广度优先实现实现
> 
> def level_queue(root):
> 
>     if root is None:
>     
>         return
>         
>     my_queue =[]
>     
>     node = root
>     
>     my_queue.append(node)
>     
>     while my_queue:
>     
>         node = my_queue.pop(0)
>         
>         print(node.elem)
>         
>         if node.lchild is not None:
>         
>             my_queue.append(node.lchild)
>             
>         if node.rchild is not None:
>         
>             my_queue.append(node.rchild)


----------
### url去重方法

1. 将url保存到数据库中，与数据库比对
1. 将url保存到set中，取决于内存大小限制，url很长的话占用很大内存空间
1. url经过md5等方法哈希后保存到set中
1. 用bitmap方法，将访问过的url通过hash函数映射到某一位，冲突高
1. bloomfilter方法对bitmap进行改进，多重hash函数降低冲突
 
----------
### 彻底搞清楚unicode和utf8编码
> unicode将所有语言统一到一套编码里， ASCII 编码是美国标准编码，unicode比ASCII占用一倍空间
> 
> utf-8 变长编码 英文变长一个字节
> 
> unicode利于数据处理，utf-8利于传输和保存，读文件转为unicode 保存文件转为utf-8
> 
> python内部编码unicode，代码`u"我用python"`直接已经转为unicode
> 
> python3直接把所有的都表示为unicode，不需要前面加u
----------

