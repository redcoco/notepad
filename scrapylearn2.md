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
spider类入口函数def start_requests(self) 默认返回处理函数def parse(self, response):


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
# 第5章 scrapy爬去知名问答网站 #

----------
###  session和cookie自动登录机制 ###

----------


- 无状态请求：请求之间无关系，各部干涉
- 有状态请求：登录可见内容，有关系的请求
- 不能跨域访问，存在域名下的cookie，客户端自动发送cookie，并与服务器互相验证，发回相应用户内容
- 服务器自动分配的id，session-id可以认为是用户名，直接验证登录

### requests模拟登陆知乎 ###

----------
#### 常见httpcode ####
- 200 请求被成功处理
- 301/302 永久性重定向/临时性重定向
- 403 没有权限的访问
- 404 表示没有对应资源
- 500 服务器错误
- 503 服务器停机或正在维护

pip install -i https://pypi.douban.com/simple requests


----------

### scrapy模拟知乎登录 ###
----------
    只匹配一行需要加参数re.DOTALL
    match_obj = re.match('.*name="_xsrf" value="(.*?)"', response.text,re.DOTALL)

    scrapy模拟登陆和request模拟登陆类似
    def start_requests 函数实现第一登陆提取xsrf，回调login
    def login(self,response) 实现表单提交，登陆功能，回调check_login是否登陆成功
    def check_login(self,response):实现验证登陆是否成功，如果成功则回调解析函数parse，默认回调parse
    注：scrapy自动保存cookie信息，check_login处自动保存

----------
### (补充小节)知乎验证码登录 ###

----------
    
    验证码图片地址：当前时间*1000转换int
    t = str(int(time.time() * 1000))
    captcha_url = "https://www.zhihu.com/captcha.gif?r={0}&type=login".format(t)
    t = session.get(captcha_url, headers=header)
    session不能改为request，否则失败，原因是不带cookie

集成到scrapy中需要注意session与cookie的传递，保证同一session下的登陆

    import time
    t = str(int(time.time() * 1000))
    captcha_url = "https://www.zhihu.com/captcha.gif?r={0}&type=login".format(t)
    yield scrapy.Request(captcha_url, headers=self.headers, meta={"post_data": post_data},
     callback=self.login_after_captcha)

----------


### 知乎分析以及数据表设计 ###

----------
> scrapy shell -s USER-AGENT="Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.90 Safari/537.36"  "https://www.zhihu.com/question/57834705"
> 需要user-agent的访问添加参数方式



----------
### item loder方式提取question ###

----------
scrapy 默认深度优先算法



----------
### 知乎spider爬虫逻辑的实现以及answer的提取 ###

----------
    item_load.add_css("title", '.zh-question-title h2 a::text')
    可能在span或者a标签里，但是css选择器无法实现或的选择改为xpath
    item_load.add_xpath("title",'//*[@id="zh-question-title"]/h2/a/text()|//*[@id="zh-question-title"]/h2/span/text()')


----------
### 保存数据到mysql中 ###

----------
    item.__class__.__name__ == "JobboleItem"并不实用
    
    如果在pipeline中写sql语句：
    多个spider不可公用
    def do_insert(self, cursor, item):
    insert_sql = """
    insert into jobbole () VALUES (%s,%s,%s,%s)
    """
    self.cursor.execute(insert_sql, (
    item["title"], item["create_date"], item["fav_nums"], item["content"], item["tags"], item["comment_nums"],
    item["praise_num"], item["front_image_url"]))
    
    如何解决多个spider公用pipeline并实现不同sql调用：
    
    def do_insert(self, cursor, item):
    insert_sql,params = item.get_insert_sql()
    self.cursor.execute(insert_sql,params)

----------
# 第6章 通过CrawlSpider对招聘网站进行整站爬取 #

----------
### 数据表结构设计 ###


----------

### CrawlSpider源码分析 ###

----------
    scrapy genspider --list 列出可用模板
    scrapy genspider -t crawl lagou www.lagou.com 使用指定模板生成spider
    
    import os
    import sys
    sys.path.insert(0,'e:\linux') #将该路径加入到pythonpath中
    
    继承CrawlSpider不能覆盖parse函数，否则rule无法使用，可以重载parse_start_url实现替代parse函数
    
    def set_crawler(self, crawler):
    super(CrawlSpider, self).set_crawler(crawler)
    self._follow_links = crawler.settings.getbool('CRAWLSPIDER_FOLLOW_LINKS', True) #CRAWLSPIDER_FOLLOW_LINKS在settings设置为false时，rules = (
    Rule(LinkExtractor(allow=r'Items/'), callback='parse_item', follow=True),
    )   则失效
    



----------
### Rule和LinkExtractor使用 ###

----------


----------
### item loader方式解析职位 ###

----------

----------
### 职位数据入库 ###


----------

----------

