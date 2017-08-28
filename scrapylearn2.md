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

    extract_first()不需要做异常处理 相对extract()[0]较安全,如出现下面的报错
    praise_num = response.css('div.post-adds span h10::text').extract()[0]IndexError: list index out of range


----------

### 编写spider爬取jobbole的所有文章 ###

----------

    <a class="next page-numbers" href="http://blog.jobbole.com/all-posts/page/2/">下一页 »</a>
	css提取只需去掉空格即 .next.page-numbers 否则代表不属于同一个class



----------
### items设计 ###

----------

item实现统一各处的字段名称，可以路由到pipeline中处理
    
    yield Request(url=parse.urljoin(response.url, post_url), callback=self.parse_detail,
      meta={"front_image_url": image_url}) #通过meta信息传递自定义上层字段，传递到下层的response

----------

    ITEM_PIPELINES = {
       'ArticleSpider.pipelines.ArticlespiderPipeline': 300,
    # 'scrapy.pipelines.images.ImagesPipeline':1, #系统内置的图片下载类，下面相关的IMAGES_URLS_FIELD和IMAGES_STORE参数都是该类中的设置参数
    'ArticleSpider.pipelines.JobboleImagesPipeline':1, #自定义的图片下载pipeline
    }
    IMAGES_URLS_FIELD="front_image_url"
    project_dir = os.path.abspath(os.path.dirname(__file__))
    IMAGES_STORE=os.path.join(project_dir,"images")


----------

    front_image_url = response.meta.get("front_image_url", "")
    jobbole_item["front_image_url"] = [front_image_url] # 后台解析IMAGES_URLS_FIELD 是一个数组，所以需要转数组

----------
需要下载图片处理库：pip install -i https://pypi.douban.com/simple pillow

----------

### 数据表设计和保存item到json文件 ###

----------
    自己写pipeline:JsonFilePipeline
    系统自带scrapy.exporters.JsonItemExporter
    



----------
### 通过pipeline保存数据到mysql ###

> pip install -i https://pypi.douban.com/simple mysqlclient

同步插入MySQL：MysqlPipeline

异步大并发插入mysql：


      dbparms = dict(
            host =settings["MYSQL_HOST"],
            db = settings["MYSQL_DBNAME"],
            user = settings["MYSQL_USER"],
            password = settings["MYSQL_PASSWORD"],
            charset = 'utf-8',
            cursorclass = MySQLdb.cursors.DictCursor,
            use_unicode  = True,
        )
        dbpool = adbapi.ConnectionPool("MySQLdb",**dbparms)

    等同于：dbpool = adbapi.ConnectionPool("MySQLdb",host =settings["MYSQL_HOST"], db = settings["MYSQL_DBNAME"], user = settings["MYSQL_USER"])



----------
### scrapy item loader机制 ###

----------
    item_loader = ItemLoader(item=JobboleItem(),response=response)
    item_loader.add_css("title","div.entry-header h1::text")
    # item_loader.add_xpath()
    item_loader.add_value("url",response.url)
	缺点无法自定义对字段的处理，如正则取评论条数字，可以转移到item中进行处理

	
	
    front_image_url =scrapy.Field(input_processor=MapCompose()) 其中MapCompose可以接收任意多的函数

----------

    图片下载相对路径转换：if "http" not in front_image_url:
    front_image_url = urljoin(get_base_url(response),front_image_url)

----------
    无法json化日期字段：
    def date_cov(value):
    try:
    create_date = datetime.datetime.strptime(value, "%Y/%m/%d").date()
    except Exception as e:
    create_date = datetime.datetime.now().date()
    return str(create_date)

----------

