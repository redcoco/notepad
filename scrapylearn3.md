# 第7章 Scrapy突破反爬虫的限制 #

----------
### 爬虫和反爬的对抗过程以及策略 ###

----------
#### 基本概念 ####

----------
> **爬虫** -自动获取网站数据的程序，关键是批量的获取
> 
> **反爬虫**-使用技术手段防止爬虫程序的方法
> 
> **误伤**-反爬虫技术将普通用户识别为爬虫，如果误伤过高，效果再好也不能用，例如禁止ip
> 
> **成本**-反爬虫需要人力和机器成本
> 
> **拦截**-成功拦截爬虫，一般拦截率越高，误伤率越高

----------

#### 爬虫的目的 ####

> **初级爬虫**-简单粗暴，不管服务器压力，容易弄挂网站
> 
> **数据保护**-
> 
> **失控的爬虫**-由于某些情况下，忘记或者无法关闭的爬虫
> 
> **商业竞争对手**
 
----------
#### 爬虫与反爬虫斗争 ####

发现IP相同，user-agent都是python->user-agent设置，ip代理->ip变化，登录才可访问->注册登录每次带cookie或者token->健全账号体系，只能访问好友信息->注册多账号，多账号联合爬取，加好友->模仿人请求->验证码验证->识别验证码->增加动态网站，数据通过js加载，发现大量请求只请求html不请求image，css资源->通过selenium和phantomjs完全模拟浏览器操作->成本过高，直接放弃

----------
### scrapy架构源码分析 ###
####  scrapy架构图 ####
![scrapy架构图](https://docs.scrapy.org/en/latest/_images/scrapy_architecture_02.png)

----------

### Requests和Response介绍 ###

[request和response](http://scrapy-chs.readthedocs.io/zh_CN/0.24/topics/request-response.html )


----------
### 通过downloadmiddleware随机更换user-agent ###

    通过middleware可以实现全局的user-agent，所有的request和response都经过。
    
    from scrapy.downloadermiddlewares.useragent import UserAgentMiddleware 
    
    class UserAgentMiddleware(object):
    """This middleware allows spiders to override the user_agent"""
    
    def __init__(self, user_agent='Scrapy'):
    self.user_agent = user_agent
    
    @classmethod
    def from_crawler(cls, crawler):
    o = cls(crawler.settings['USER_AGENT'])
    crawler.signals.connect(o.spider_opened, signal=signals.spider_opened)
    return o
    
    def spider_opened(self, spider):
    self.user_agent = getattr(spider, 'user_agent', self.user_agent)
    
    def process_request(self, request, spider):
    if self.user_agent:
    request.headers.setdefault(b'User-Agent', self.user_agent)
    
    在settings文件中配置该参数USER_AGENT
    需要将默认的useragentmiddleware设置为none
    
    
    pip install fake-useragent
    from fake_useragent import UserAgent
    ua = UserAgent()
    
    ua.ie

----------
### scrapy实现ip代理池 ###


----------
### 云打码实现验证码识别 ###
#### 编码实现（tesseract-ocr） ####
#### 在线打吗 ####
#### 人工打码 ####

----------
### cookie禁用、自动限速、自定义spider的settings ###
    # Disable cookies (enabled by default)
    COOKIES_ENABLED = False 禁用
	# See also autothrottle settings and docs
	DOWNLOAD_DELAY = 0.25 250ms

	# Enable and configure the AutoThrottle extension (disabled by default)
	# See http://doc.scrapy.org/en/latest/topics/autothrottle.html
	AUTOTHROTTLE_ENABLED = True
	# The initial download delay
	AUTOTHROTTLE_START_DELAY = 5
	# The maximum download delay to be set in case of high latencies
	AUTOTHROTTLE_MAX_DELAY = 60
	# The average number of requests Scrapy should be sending in parallel to
	# each remote server
	AUTOTHROTTLE_TARGET_CONCURRENCY = 1.0
	# Enable showing throttling stats for every response received:
	AUTOTHROTTLE_DEBUG = False


----------
#  第8章 scrapy进阶开发  #
### selenium动态网页请求与模拟登录知乎 ###
http://selenium-python.readthedocs.io/
pip install -i https://pypi.douban.com/simple selenium



----------
### selenium模拟登录微博， 模拟鼠标下拉 ###

browser.execute_script（“javascript代码”） #执行javascript代码


----------
### chromedriver不加载图片、phantomjs获取动态网页 ###

phantomjs, 无界面的浏览器， 多进程情况下phantomjs性能会下降很严重

----------
### selenium集成到scrapy中 ###

    通过中间件实现selnium集成
    
    通过spider参数可以监听信号机制，在spider中初始化browser
    传递到middleware中使用
    # 使用seleinum实现chrome请求
    def __init__(self):
    self.browser =  webdriver.Chrome(executable_path="C:/Users/wenjuan/PycharmProjects/chromedriver.exe")
    super(JobboleSpider,self).__init__()
    dispatcher.connect(self.spider_closed,signals.spider_closed)
    
    def spider_closed(self,spider):
    # 档爬虫关闭时，关闭浏览器
    print("spider closed :jobbole")
    self.browser.quit()



----------
###  其余动态网页获取技术介绍-chrome无界面运行,scrapy-splash,selenium-grid,splinter ###

pip install -i https://pypi.douban.com/simple pyvirtualdisplay


    # 无界面chrome
    from pyvirtualdisplay import Display
    display = Display(visible=0,size=(800,600)) # 0 不可见
    display.start()
    browser = webdriver.Chrome()
    browser.get()

scrapy-splash

----------
### scrapy的暂停与重启 ###

    scrapy crawl lagou -s JOBDIR=job_info/001
    或者在customer_settings ={
    "JOBDIR":"job_info/001",
    }
    
    命令行Ctrl+c 命令中断，即可暂停
    
    重启直接运行原命令：
    scrapy crawl lagou -s JOBDIR=job_info/001


----------
### scrapy url去重原理 ###

使用urllib.sha1()对url生成摘要信息，放到set中比对

----------
### scrapy telnet服务 ###

    日志信息
    2017-09-02 21:46:26 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
    
    
    cmd 中输入 telnet 127.0.0.1 6023
    》est（）
    》spider
    》spider.name



----------
### spider middleware 详解 ###
    downloadermiddleware :例如之前的RandomUserAgentMiddleware，RandomHttpProxyMiddleware
    
    spidermiddleware需要在settings配置：
    SPIDER_MIDDLEWARES = {   'ArticleSpider.middlewares.ArticlespiderSpiderMiddleware': 543,
    }
    
    在middleware.py 中函数 ArticlespiderSpiderMiddleware
    


----------
### scrapy的数据收集 ###

数据收集器：

----------
###  scrapy信号详解 ###


----------
### scrapy扩展开发 ###

[扩展(Extensions)](http://scrapy-chs.readthedocs.io/zh_CN/0.24/topics/extensions.html)

----------
# scrapy-redis分布式爬虫 #
### 分布式爬虫要点 ###

----------
- 使用redis作为状态管理器，管理url的set，去重。
#### 分布式爬虫优点 ####

充分利用多台机器的带宽加速爬取

充分利用多机的ip加速爬取

#### 分布式需要解决的问题 ####

request队列集中管理

去重集中管理

----------
### redis基础知识 ###
[redis下载地址windows版本](https://github.com/ServiceStack/redis-windows/tree/master/downloads)

启动server：./redis-server.exe

启动客户端访问：./redis-cli.exe

#### redis数据类型 ####
1. 字符串
1. 散列/哈希
1. 列表
1. 集合
1. 可排序集合

#### 字符串的命令 ####

    设置值
    127.0.0.1:6379> set course_name "scrapy-redis"
    OK
    获取值
    127.0.0.1:6379> get course_name
    "scrapy-redis"
    获取子串
    127.0.0.1:6379> getrange course_name 2 5
    "rapy"
    获取长度
    127.0.0.1:6379> strlen course_name
    (integer) 12
    增加值
    127.0.0.1:6379> set course_count 3
    OK
    127.0.0.1:6379> incr course_count
    (integer) 4
    127.0.0.1:6379> get course_count
    "4"
    减少值
    127.0.0.1:6379> decr course_count
    (integer) 3
    127.0.0.1:6379>
    
    join值
    
    127.0.0.1:6379> append course_name 0
    (integer) 13
    127.0.0.1:6379> get course_name
    "scrapy-redis0"
#### 哈希命令 ####
    hash set 
    127.0.0.1:6379> hset course_dict bobby "python scrapy"
    (integer) 1
    获取所有key value
    127.0.0.1:6379> hgetall course_dict
    1) "bobby"
    2) "python scrapy"
    获取key的 value
    127.0.0.1:6379> hget course_dict bobby
    "python scrapy"
    是否存在
    127.0.0.1:6379> hexists course_dict bobby
    (integer) 1
    127.0.0.1:6379> hexists course_dict bobby2
    (integer) 0
    删除
    127.0.0.1:6379> hdel course_dict bobby
    (integer) 1
    127.0.0.1:6379> hexists course_dict bobby
    (integer) 0
    127.0.0.1:6379> hset course_dict bobby "python scrapy"
    (integer) 1
    获取所有key
    127.0.0.1:6379> hkeys course_dict
    1) "bobby"
    获取所有value
    127.0.0.1:6379> hvals course_dict
    1) "python scrapy"

#### 列表命令 ####
    后进先出
    127.0.0.1:6379> lpush imooc_courses "scrapy"
    (integer) 1
    127.0.0.1:6379> lpush imooc_courses "django"
    (integer) 2
    列出元素
    127.0.0.1:6379> lrange imooc_courses 0 10
    1) "django"
    2) "scrapy"
    127.0.0.1:6379> rpush imooc_courses "scrapy-redis"
    (integer) 3
    127.0.0.1:6379> lrange imooc_courses 0 10
    1) "django"
    2) "scrapy"
    3) "scrapy-redis"
    阻塞删除 blpop imooc_courses timeout 等待三秒删除，如果存在立马删除
    127.0.0.1:6379> blpop imooc_courses 3
    1) "imooc_courses"
    2) "django"
    127.0.0.1:6379> rpush imooc_courses "scrapy-redis"
    (integer) 3
    删除立即
    127.0.0.1:6379> lpop imooc_courses
    "scrapy"
    127.0.0.1:6379> lpush imooc_courses "scrapy"
    (integer) 3
    127.0.0.1:6379> lrange imooc_courses 0 10
    1) "scrapy"
    2) "scrapy-redis"
    3) "scrapy-redis"
    127.0.0.1:6379> rpop imooc_courses
    "scrapy-redis"
    127.0.0.1:6379> lrange imooc_courses 0 10
    1) "scrapy"
    2) "scrapy-redis"
    127.0.0.1:6379>
    长度
    127.0.0.1:6379> llen imooc_courses
    (integer) 2
    取出下标从0开始
    127.0.0.1:6379> lindex imooc_courses 1
    "scrapy-redis"
    127.0.0.1:6379> lrange imooc_courses 0 10
    1) "scrapy"
    2) "scrapy-redis"
    127.0.0.1:6379>
#### 集合命令 ####
    添加
    127.0.0.1:6379> sadd  course_set "django"
    (integer) 1
    不可重复
    127.0.0.1:6379> sadd  course_set "django"
    (integer) 0
    127.0.0.1:6379> sadd  course_set "scrapy"
    (integer) 1
    长度
    127.0.0.1:6379> scard course_set
    (integer) 2
    127.0.0.1:6379> sdiff course_set course_set
    (empty list or set)
    127.0.0.1:6379> sadd  course_set2 "scrapy"
    (integer) 1
    差集course_set - course_set2
    127.0.0.1:6379> sdiff course_set course_set2
    1) "django"
    差集course_set2 - course_set
    127.0.0.1:6379> sdiff course_set2 course_set
    (empty list or set)
    交集
    127.0.0.1:6379> sinter course_set2 course_set
    1) "scrapy"
    随机删除
    127.0.0.1:6379> spop course_set
    "scrapy"
    随机获取
    127.0.0.1:6379> srandmember course_set
    "django"
#### 可排序集合命令 ####
    设置值 分数 - 科目格式
    127.0.0.1:6379> zadd zcourse_set 5 "scrapy" 0 "django" 10 "scrapy-redis"
    (integer) 3
    获取分段之间数据
    127.0.0.1:6379> zrangebyscore zcourse_set 0 4
    1) "django"
    127.0.0.1:6379> zrangebyscore zcourse_set  4 8
    1) "scrapy"
    127.0.0.1:6379> zrangebyscore zcourse_set  4 20
    1) "scrapy"
    2) "scrapy-redis"
    127.0.0.1:6379>
    分段间数据个数
    127.0.0.1:6379> zcount zcourse_set 4 20
    (integer) 2
    127.0.0.1:6379> zcount zcourse_set 0 4
    (integer) 1

    keys * 所有数据集
    type jobbole:requests 查看数据类型

----------
### scrapy-redis编写分布式爬虫代码 ###
    pip intsall redis
    下载：https://github.com/rmax/scrapy-redis
    
    如果报错 ：
    ImportError: No module named 'win32api'
    需要安装：
    pip install -i https://pypi.douban.com/simple pypiwin32
    
    Traceback (most recent call last):
      File "E:\downloads\envs\scrapy-redis\lib\site-packages\scrapy\crawler.py", line 74, in crawl
    yield self.engine.open_spider(self.spider, start_requests)
    ImportError: No module named 'ScrapyRedisTest.utils'

> debug maim.py
> 
> redis-cli中：lpush jobbole:start_urls http://blog.jobbole.com/all-posts/

----------


### scrapy-redis源码解析-connection.py、defaults.py ###

![scrapy-redis架构图](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1504525018960&di=997f73b87a88c3634f635bae9aa3a443&imgtype=0&src=http%3A%2F%2Fpic.92to.com%2F201702%2F16%2F201727103939264.jpg)


----------
### scrapy-redis源码剖析-dupefilter.py ###

----------
### scrapy-redis源码剖析- pipelines.py、 queue.py ###

----------
### scrapy-redis源码分析- scheduler.py、spider.py ###

----------
### 集成bloomfilter到scrapy-redis中 ###

[http://www.cnblogs.com/adc8868/p/7442306.html](http://www.cnblogs.com/adc8868/p/7442306.html)

> 基本概念
> 
> 如果想判断一个元素是不是在一个集合里，一般想到的是将所有元素保存起来，然后通过比较确定。链表，树等等数据结构都是这种思路. 但是随着集合中元素的增加，我们需要的存储空间越来越大，检索速度也越来越慢。不过世界上还有一种叫作散列表（又叫哈希表，Hash table）的数据结构。它可以通过一个Hash函数将一个元素映射成一个位阵列（Bit Array）中的一个点。这样一来，我们只要看看这个点是不是 1 就知道可以集合中有没有它了。这就是布隆过滤器的基本思想。
> 
> Hash面临的问题就是冲突。假设 Hash 函数是良好的，如果我们的位阵列长度为 m 个点，那么如果我们想将冲突率降低到例如 1%, 这个散列表就只能容纳 m/100 个元素。显然这就不叫空间有效了（Space-efficient）。解决方法也简单，就是使用多个 Hash，如果它们有一个说元素不在集合中，那肯定就不在。如果它们都说在，虽然也有一定可能性它们在说谎，不过直觉上判断这种事情的概率是比较低的。
> 
>  
> 
> 优点
> 
> 相比于其它的数据结构，布隆过滤器在空间和时间方面都有巨大的优势。布隆过滤器存储空间和插入/查询时间都是常数。另外, Hash 函数相互之间没有关系，方便由硬件并行实现。布隆过滤器不需要存储元素本身，在某些对保密要求非常严格的场合有优势。
> 
> 布隆过滤器可以表示全集，其它任何数据结构都不能；
> 
> k 和 m 相同，使用同一组 Hash 函数的两个布隆过滤器的交并差运算可以使用位操作进行。
> 
>  
> 
> 缺点
> 
> 但是布隆过滤器的缺点和优点一样明显。误算率（False Positive）是其中之一。随着存入的元素数量增加，误算率随之增加。但是如果元素数量太少，则使用散列表足矣。
> 
> 另外，一般情况下不能从布隆过滤器中删除元素. 我们很容易想到把位列阵变成整数数组，每插入一个元素相应的计数器加1, 这样删除元素时将计数器减掉就可以了。然而要保证安全的删除元素并非如此简单。首先我们必须保证删除的元素的确在布隆过滤器里面. 这一点单凭这个过滤器是无法保证的。另外计数器回绕也会造成问题。

安装依赖包：

　　pip install mmh3

    py_bloomfilter.py(布隆过滤器)源码
    import mmh3
    import redis
    import math
    import time
    
    
    class PyBloomFilter():
    #内置100个随机种子
    SEEDS = [543, 460, 171, 876, 796, 607, 650, 81, 837, 545, 591, 946, 846, 521, 913, 636, 878, 735, 414, 372,
     344, 324, 223, 180, 327, 891, 798, 933, 493, 293, 836, 10, 6, 544, 924, 849, 438, 41, 862, 648, 338,
     465, 562, 693, 979, 52, 763, 103, 387, 374, 349, 94, 384, 680, 574, 480, 307, 580, 71, 535, 300, 53,
     481, 519, 644, 219, 686, 236, 424, 326, 244, 212, 909, 202, 951, 56, 812, 901, 926, 250, 507, 739, 371,
     63, 584, 154, 7, 284, 617, 332, 472, 140, 605, 262, 355, 526, 647, 923, 199, 518]
    
    #capacity是预先估计要去重的数量
    #error_rate表示错误率
    #conn表示redis的连接客户端
    #key表示在redis中的键的名字前缀
    def __init__(self, capacity=1000000000, error_rate=0.00000001, conn=None, key=‘BloomFilter‘):
    self.m = math.ceil(capacity*math.log2(math.e)*math.log2(1/error_rate))  #需要的总bit位数
    self.k = math.ceil(math.log1p(2)*self.m/capacity)   #需要最少的hash次数
    self.mem = math.ceil(self.m/8/1024/1024)#需要的多少M内存
    self.blocknum = math.ceil(self.mem/512) #需要多少个512M的内存块,value的第一个字符必须是ascii码，所有最多有256个内存块
    self.seeds = self.SEEDS[0:self.k]
    self.key = key
    self.N = 2**31-1
    self.redis = conn
    # print(self.mem)
    # print(self.k)
    
    def add(self, value):
    name = self.key + "_" + str(ord(value[0])%self.blocknum)
    hashs = self.get_hashs(value)
    for hash in hashs:
    self.redis.setbit(name, hash, 1)
    def is_exist(self, value):
    name = self.key + "_" + str(ord(value[0])%self.blocknum)
    hashs = self.get_hashs(value)
    exist = True
    for hash in hashs:
    exist = exist & self.redis.getbit(name, hash)
    return exist
    
    def get_hashs(self, value):
    hashs = list()
    for seed in self.seeds:
    hash = mmh3.hash(value, seed)
    if hash >= 0:
    hashs.append(hash)
    else:
    hashs.append(self.N - hash)
    return hashs
    
    
    pool = redis.ConnectionPool(host=‘127.0.0.1‘, port=6379, db=0)
    conn = redis.StrictRedis(connection_pool=pool)
    
    # 使用方法
    # if __name__ == "__main__":
    # bf = PyBloomFilter(conn=conn)   # 利用连接池连接Redis
    # bf.add(‘www.jobbole.com‘)   # 向Redis默认的通道添加一个域名
    # bf.add(‘www.luyin.org‘) # 向Redis默认的通道添加一个域名
    # print(bf.is_exist(‘www.zhihu.com‘)) # 打印此域名在通道里是否存在，存在返回1，不存在返回0
    # print(bf.is_exist(‘www.luyin.org‘)) # 打印此域名在通道里是否存在，存在返回1，不存在返回0

将py_bloomfilter.py(布隆过滤器)集成到scrapy-redis中的dupefilter.py去重器中，使其抓取过的URL不添加到下载器，没抓取过的URL添加到下载器


    scrapy-redis中的dupefilter.py去重器修改
    from bloomfilter.py_bloomfilter import conn,PyBloomFilter   #导入布隆过滤器
    
    logger = logging.getLogger(__name__)
    
    
    # TODO: Rename class to RedisDupeFilter.
    
    class RFPDupeFilter(BaseDupeFilter):
    """Redis-based request duplicates filter.
    
    This class can also be used with default Scrapy‘s scheduler.
    
    """
    
    logger = logger
    
    def __init__(self, server, key, debug=False):
    """Initialize the duplicates filter.
    
    Parameters
    ----------
    server : redis.StrictRedis
    The redis server instance.
    key : str
    Redis key Where to store fingerprints.
    debug : bool, optional
    Whether to log filtered requests.
    """
    self.server = server
    self.key = key
    self.debug = debug
    self.logdupes = True
    
    # 集成布隆过滤器
    self.bf = PyBloomFilter(conn=conn, key=key) # 利用连接池连接Redis
    
    @classmethod
    def from_settings(cls, settings):
    """Returns an instance from given settings.
    
    This uses by default the key ``dupefilter:<timestamp>``. When using the
    ``scrapy_redis.scheduler.Scheduler`` class, this method is not used as
    it needs to pass the spider name in the key.
    
    Parameters
    ----------
    settings : scrapy.settings.Settings
    
    Returns
    -------
    RFPDupeFilter
    A RFPDupeFilter instance.
    
    
    """
    server = get_redis_from_settings(settings)
    # XXX: This creates one-time key. needed to support to use this
    # class as standalone dupefilter with scrapy‘s default scheduler
    # if scrapy passes spider on open() method this wouldn‘t be needed
    # TODO: Use SCRAPY_JOB env as default and fallback to timestamp.
    key = defaults.DUPEFILTER_KEY % {‘timestamp‘: int(time.time())}
    debug = settings.getbool(‘DUPEFILTER_DEBUG‘)
    return cls(server, key=key, debug=debug)
    
    @classmethod
    def from_crawler(cls, crawler):
    """Returns instance from crawler.
    Parameters
    ----------
    crawler : scrapy.crawler.Crawler
    
    Returns
    -------
    RFPDupeFilter
    Instance of RFPDupeFilter.
    
    """
    return cls.from_settings(crawler.settings)
    
    def request_seen(self, request):
    """Returns True if request was already seen.
    
    Parameters
    ----------
    request : scrapy.http.Request
    
    Returns
    -------
    bool
    
    """
    fp = self.request_fingerprint(request)
    
    # 集成布隆过滤器
    if self.bf.is_exist(fp):# 判断如果域名在Redis里存在
    return True
    else:
    self.bf.add(fp) # 如果不存在，将域名添加到Redis
    return False
    
    # This returns the number of values added, zero if already exists.
    # added = self.server.sadd(self.key, fp)
    # return added == 0
    
    def request_fingerprint(self, request):
    """Returns a fingerprint for a given request.
    
    Parameters
    ----------
    request : scrapy.http.Request
    
    Returns
       -------
    str
    
    """
    return request_fingerprint(request)
    
    def close(self, reason=‘‘):
    """Delete data on close. Called by Scrapy‘s scheduler.
    
    Parameters
    ----------
    reason : str, optional
    
    """
    self.clear()
    
    def clear(self):
    """Clears fingerprints data."""
    self.server.delete(self.key)
    
    def log(self, request, spider):
    """Logs given request.
    
    Parameters
    ----------
    request : scrapy.http.Request
    spider : scrapy.spiders.Spider
    
    """
    if self.debug:
    msg = "Filtered duplicate request: %(request)s"
    self.logger.debug(msg, {‘request‘: request}, extra={‘spider‘: spider})
    elif self.logdupes:
    msg = ("Filtered duplicate request %(request)s"
       " - no more duplicates will be shown"
       " (see DUPEFILTER_DEBUG to show all duplicates)")
    self.logger.debug(msg, {‘request‘: request}, extra={‘spider‘: spider})
    self.logdupes = False
 

爬虫文件

    
    #!/usr/bin/env python
    # -*- coding:utf8 -*-
    
    from scrapy_redis.spiders import RedisCrawlSpider# 导入scrapy_redis里的RedisCrawlSpider类
    import scrapy
    from scrapy.linkextractors import LinkExtractor
    from scrapy.spiders import Rule
    
    
    class jobboleSpider(RedisCrawlSpider):   # 自定义爬虫类,继承RedisSpider类
    name = ‘jobbole‘ # 设置爬虫名称
    allowed_domains = [‘www.luyin.org‘]  # 爬取域名
    redis_key = ‘jobbole:start_urls‘ # 向redis设置一个名称储存url
    
    rules = (
    # 配置抓取列表页规则
    # Rule(LinkExtractor(allow=(‘ggwa/.*‘)), follow=True),
    
    # 配置抓取内容页规则 
    Rule(LinkExtractor(allow=(‘.*‘)), callback=‘parse_job‘, follow=True),
    )
    
    
    def parse_job(self, response):  # 回调函数，注意：因为CrawlS模板的源码创建了parse回调函数，所以切记我们不能创建parse名称的函数
    # 利用ItemLoader类，加载items容器类填充数据
    neir = response.css(‘title::text‘).extract()
    print(neir)
    
    启动爬虫 scrapy crawl jobbole
    
    cd 到redis安装目录执行命令：redis-cli -h 127.0.0.1 -p 6379  连接redis客户端
    
    连接redis客户端后执行命令：lpush jobbole:start_urls http://www.luyin.org  向redis添加一个爬虫起始url
    
    开始爬取
  
  
----------
