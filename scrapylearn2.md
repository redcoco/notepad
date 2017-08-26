# 第4章 scrapy爬取知名技术文章网站 #
----------
### scrapy安装以及目录结构介绍 ###
> mkvitualenv --python=python路径 airticle_spider
> 
> pip install -i https://pypi.douban.com/simple/ scrapy
> 
> scrapy startproject ArticleSpider
> 
> cd ArticleSpider
> 
> scrapy genspider jobbole jobbole.com
> 
> scrapy.cfg 类似django的配置
> 
> setting.py SPIDER_MODULES NEWSPIDER_MODULE spider存放路径
> 
> pipeline.py 数据存储相关
> 
> middleware.py 定制中间件
> 
> item.py 数据定义
> 
> spider 文件夹爬虫路径

----------

### pycharm 调试scrapy 执行流程 ###

scrapy crawl jobbole

----------

### xpath的用法 ###
#### xpath简介 ####

1. xpath使用路径表达式在xml和html中进行导航
1. xpath包含标准函数库
1. xpath是一个w3c的标准

#### xpath节点关系 ####

1. 父节点
1. 子节点
1. 同胞节点
1. 先辈节点
1. 后代节点

----------
#### xpath语法 ####

> article	选取所有article元素的所有子节点
> 
> /article 选取根元素article
> 
> article/a 选取所有属于article的子元素的a元素
> 
> //div 选取所有div子元素（不论出现在文档任何地方）
> 
> article// 选取所有属于article元素的后代的div元素，不管它出现在article之下的任何位置
> 
> //@class 选取所有名为class的属性
> 
> /article/div[1] 选取属于article子元素的第一个div元素
> 
> /article/div[last（)] 选取属于article子元素的最后一个div元素
> /article/div[last（)-1] 选取属于article子元素的倒数第二个div元素
> 
> //div[@lang] 选取所有拥有属性为lang的div元素
> 
> //div[@lang='eng'] 选取所有拥有属性为lang属性为eng的div元素
> 
> /div/* 选取属性div元素的所有子节点
> 
> //* 选取所有元素
> 
> //div[@*] 选取所有带属性的div元素
> 
> //div/a|//div/p 选取所有div的a和p元素
> 
> //span|//ul 选取文档中的span和ul元素
> 
> article/div/p|//span 选取属于article元素div元素p元素以及所有的span元素
> 
> //span[contains(@class,"vote_up")] 选取span的class属性包含vote_up值的span元素
> 

进入调试模式：
scrapy shell http://blog.jobbole.com/110287/

----------
### css选择器实现字段解析 ###

----------

#### css选择器 ####
> * 选择所有节点
> 
> ·#container 选择id为container的节点、
> 
> .container 选择所有class包含container的节点
> 
> li a  选取所有li下的所有a节点
> 
> ul+p 选择ul后面的第一个p元素
> 
> div#container>ul 选取id为container的div的第一个ul子元素
> 
> ul~p 选取与ul相邻的所有p元素
> 
> a[title] 选取所有有title属性的a元素
> 
> a[href="http://jobbole.com"] 选取href属性为jobbole.com值的a元素
> 
> a[href*="jobbole"] 选取所有href属性含有jobbole的a元素
> 
> a[href^="http"] 选取所有href属性以http开头值的a元素
> 
> a[href$=".jpg"] 选取所有href属性以。jpg结尾值的a元素
> 
> input[type=radio]:checked 选择选中的radio元素
> 
> div:not(#container) 选取所有id非container的div属性
> 
> li:nth-child(3) 选取第三个li元素
> 
> li:nth-child(2n) 选取第偶数个li元素
> 

    extract_first()不需要做异常处理 相对extract()[0]较安全

----------

### 编写spider爬取jobbole的所有文章 ###

----------

    a class="next page-numbers" href="http://blog.jobbole.com/all-posts/page/2/">下一页 »</a>
		css提取只需去掉空格即 .next.page-numbers 否则代表不属于同一个class



----------





















	| 表达式        | 说明    |  数量  |
    | --------   | -----:   | :----: |
    | 香蕉        | $1      |   5    |
    | 苹果        | $1      |   6    |
    | 草莓        | $1      |   7    |