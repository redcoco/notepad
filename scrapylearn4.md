# 第10章 elasticsearch搜索引擎的使用 #

----------
### elasticsearch介绍 ###
我们建设一个网站或者程序，希望添加搜索功能，发现搜索工作很难

1. 我们希望搜索解决方案要高效
1. 我们希望零配置和完全免费的搜索方案
1. 我们希望能够简单的通过json和http与搜索引擎交互
1. 我们希望我们的搜索服务器稳定
1. 我们希望能够简单的将一台服务器扩展到上百台

#### 关系数据搜索缺点 ####

1. 无法打分
1. 无分布式
1. 无法解析搜索请求
1. 效率低
1. 分词

### elasticsearch安装 ###

#### 安装 ####

 安装elasticsearch-rtf [https://github.com/medcl/elasticsearch-rtf](https://github.com/medcl/elasticsearch-rtf)如果内存不足，调整elasticsearch-rtf\config\jvm.options 中的-Xms1g 和 -Xmx1g


----------
### elasticsearch-head插件以及kibana的安装 ###

[https://github.com/mobz/elasticsearch-head](https://github.com/mobz/elasticsearch-head)

cmder安装nodejs安装npm
[https://nodejs.org/en/download/](https://nodejs.org/en/download/)

或者安装cnpm，使用国内镜像
[http://npm.taobao.org/](http://npm.taobao.org/)

- git clone git://github.com/mobz/elasticsearch-head.git
- cd elasticsearch-head
- cnpm install
- cnpm run start
- open http://localhost:9100/

不允许第三方库访问9200端口
修改elasticsearch-rtf\config\elasticsearch.yml

    # add for elasticsearch-head connect
    
    http.cors.enabled: true
    http.cors.allow-origin: "*"
    http.cors.allow-methods: OPTIONS,HEAD,GET,POST,PUT,DELETE
    http.cors.allow-headers: "X-Requested-With,Content-Type,Content-Length,X-User"


[https://www.elastic.co/downloads/kibana](https://www.elastic.co/downloads/kibana)
相应版本的kebana 5.1.1
[https://www.elastic.co/downloads/past-releases/kibana-5-1-1](https://www.elastic.co/downloads/past-releases/kibana-5-1-1)


----------
### elasticsearch的基本概念 ###
- 集群：一个或者多个节点组织在一起
- 节点：一个节点是集群的一个服务器，由一个名字标识，默认是随机的漫画角色名字
- 分片：将索引划分为多份的能力，允许水平分割和扩展容量，多个分片响应请求，提供性能和吞吐量
- 副本：创建分片的一份或者多份的能力，在一个节点失败其余节点可以顶上


> index索引 相当于数据库
> 
> type 类型 相当于表
> 
> document 文档 相当于行
> 
> fields 相当于列
> 

#### http方法 ####

> http 1.0 get post head
> 
> http 1.1 options put delete trace connect
> 
> 主要get post put delete 



----------
### 倒排索引 ###
属性值确定记录的位置，与正常的相反，因此称为倒排索引。

> 分词的各个词在各个文章的映射
> 
> python 文章1，文章3 key-value

#### 倒排索引待解决的问题 ####
1. 大小写转换问题
1. 词干抽取，looking与look应该理解为一个词
1. 分词 ，屏蔽系统 or 屏蔽 - 系统
1. 倒排索引文件过大 -压缩编码


----------
### elasticsearch 基本的索引和文档CRUD操作 ###
    
    等同建数据库
    # 索引chushih初始化caoz初始化操作
    # 指定分片和副本的数量分片和副本的数量
    # shards 指定无法修改 
    PUT lagou
    {
      "settings": {
    "index": {
    "number_of_replicas": 1
    , "number_of_shards": 5
      }
      }
    }
    
    #获取settings
    GET lagou/_settings
    GET _all/_settings
    GET lagou,jobbole/_settings
    
    # 修改信息
    PUT lagou/_settings
    {
      "number_of_replicas": 2
    }
    
    保存数据到表
    # 保存文档
    PUT lagou/job/1  不指明id自生成uuid PUT lagou/job
    {
      "title":"pythonfenbushi分布式 kaifa开发 ",
      "salary_min":15000,
      "city":"beijing背景板 ",
      "company":{
    "name":"baidu百度 ",
    "company_addr":"beijins北京市 ruanjian软件yuan软件园 "
      },
      "publish_date":"2017-08-20",
      "comments":13
    }
    
    
    GET lagou/job/1
    GET lagou/job/1?_source=title,city
    GET lagou/job/1?_source
    
    # 修改覆盖式
    PUT lagou/job/1
    {
      "title":"pythonfenbushi分布式 kaifa开发 ",
      "salary_min":15000,
      "city":"beijing背景板 ",
      "company":{
    "name":"baidu百度 ",
    "company_addr":"beijins北京市 ruanjian软件yuan软件园 "
      },
      "publish_date":"2017-08-20",
      "comments":13
    }
    
    # 修改增量式
    POST lagou/job/1/_update
    {
    "doc"{
    "document":20
    }
    }
    
    # 删除
    
    DELETE lagou/job/1
    DELETE lagou
    

----------
### elasticsearch的mget和bulk批量操作 ###
    GET _mget 
    {
      "docs":[
      {"_index":"testdb",
      "_type":"job1",
      "_id":1
      }, {"_index":"testdb",
      "_type":"job1",
      "_id":2
      }
      ]
    }
    
    GET testdb/job1/_mget
    {
      "docs":[
    { "_id":1},
    { "_id":2}
    ]
    }
    
    GET testdb/job1/_mget
    {
      "ids":[1,2]
    }
    
    bulk批量生产操作：
    一般两行，delete一行
    不能美化格式，只能一行
    POST _bulk
    {"index":{"_index":"lagou","_type":"job","_id":1}}
    {"title":"pythonfenbushi分布式 kaifa开发 ","salary_min":13000,"city":"beijing背景板 ","company":{"name":"baidu百度 ","company_addr":"beijins北京市 ruanjian软件yuan软件园 "},"publish_date":"2017-08-20","comments":131}
    {"index":{"_index":"lagou","_type":"job1","_id":2}}
    {"title":"pythonfenbushi分布式 kaifa开发 ","salary_min":15000,"city":"beijing背景板 ","company":{"name":"baidu百度 ","company_addr":"beijins北京市 ruanjian软件yuan软件园 "},"publish_date":"2017-08-20","comments":31}
    
    
    POST _bulk
    {"delete":{"_index":"lagou","_type":"job","_id":1}}


----------
### elasticsearch的mapping映射管理 ###
    映射：创建索引的时候，可以预先定义字段的类型以及相关属性
    
    mapping就是我们自己定义的字段的数据类型，同时告诉elasticsearch如何索引数据以及是否可以被搜索
    作用：会让索引建立的更加细致完善
    类型：静态映射
#### 内置类型 ####
相当于字段类型设置

- string类型：text，keyword
- 数字类型：long，integer，short，byte，double，float
- 日期类型：date
- bool类型：boolean
- binary类型：binary
- 复杂类型：object，nested
- geo类型：geo-point，geo-shape
- 专业类型：ip，competion

更多属性：https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-boost.html

    #创建索引(设置字段类型)
    PUT jobbole #创建索引设置索引名称
    {
      "mappings": { #设置mappings映射字段类型
    "job": {#表名称
      "properties": {   #设置字段类型
    "title":{   #title字段
      "type": "text"#text类型，text类型可以分词，建立倒排索引
    },
    "salary_min":{  #salary_min字段
      "type": "integer" #integer数字类型
    },
    "city":{#city字段
      "type": "keyword" #keyword普通字符串类型
    },
    "company":{ #company字段，是嵌套字段
      "properties":{#设置嵌套字段类型
    "name":{#name字段
      "type":"text" #text类型
    },
    "company_addr":{#company_addr字段
      "type":"text" #text类型
    },
    "employee_count":{  #employee_count字段
      "type":"integer"  #integer数字类型
    }
      }
    },
    "publish_date":{#publish_date字段
      "type": "date",   #date时间类型
      "format":"yyyy-MM-dd" #yyyy-MM-dd格式化时间样式
    },
    "comments":{#comments字段
      "type": "integer" #integer数字类型
    }
      }
    }
      }
    }
    
    
    #保存文档(相当于数据库的写入数据)
    PUT jobbole/job/1   #索引名称/表/id
    {
      "title":"python分布式爬虫开发",   #字段名称：字段值
      "salary_min":15000,   #字段名称：字段值
      "city":"北京",#字段名称：字段值
      "company":{   #嵌套字段
    "name":"百度",  #字段名称：字段值
    "company_addr":"北京市软件园",  #字段名称：字段值
    "employee_count":50 #字段名称：字段值
      },
      "publish_date":"2017-4-16",   #字段名称：字段值
      "comments":15 #字段名称：字段值
    }
    
    #获取一个索引下的所有表的mappings映射字段类型
    GET jobbole/_mapping
    #获取一个索引下的指定表的mappings映射字段类型
    GET jobbole/_mapping/job

----------

### elasticsearch的简单查询 ###

    elasticsearch是功能非常强大的搜索引擎，使用它的目的就是为了快速的查询到需要的数据
    
    查询分类：
    　　基本查询：使用elasticsearch内置的查询条件进行查询
    　　组合查询：把多个查询条件组合在一起进行复合查询
    　　过滤：查询同时，通过filter条件在不影响打分的情况下筛选数据

    首先我们先创建索引、表、以及字段属性、字段类型、添加好数据
    
    注意：一般我们中文使用ik_max_word中文分词解析器，所有在需要分词建立倒牌索引的字段都要指定，ik_max_word中文分词解析器
    系统默认不是ik_max_word中文分词解析器
    
    ik_max_word中文分词解析器是elasticsearch(搜索引擎)的一个插件，在elasticsearch安装目录的plugins/analysis-ik文件夹里，版本为5.1.1
    
    更多说明：https://github.com/medcl/elasticsearch-analysis-ik

    #创建索引(设置字段类型)
    #注意：一般我们中文使用ik_max_word中文分词解析器，所有在需要分词建立倒牌索引的字段都要指定，ik_max_word中文分词解析器
    #系统默认不是ik_max_word中文分词解析器
    PUT jobbole #创建索引设置索引名称
    {
      "mappings": { #设置mappings映射字段类型
    "job": {#表名称
      "properties": {   #设置字段类型
    "title":{   #表名称
      "store": true,#字段属性true表示保存数据
      "type": "text",   #text类型，text类型可以分词，建立倒排索引
      "analyzer": "ik_max_word" #设置分词解析器，ik_max_word是一个中文分词解析器插件
    },
    "company_name":{#字段名称
      "store": true,#字段属性true表示保存数据
      "type": "keyword" #keyword普通字符串类型，不分词
    },
    "desc":{#字段名称
      "type": "text"#text类型，text类型可以分词，但是没有设置分词解析器，使用系统默认
    },
    "comments":{#字段名称
      "type": "integer" #integer数字类型
    },
    "add_time":{#字段名称
      "type": "date",   #date时间类型
      "format":"yyyy-MM-dd" #yyyy-MM-dd时间格式化
    }
      }
    }
      }
    }
    
    
    
    #保存文档(相当于数据库的写入数据)
    POST jobbole/job
    {
      "title":"python django 开发工程师", #字段名称：值
      "company_name":"美团科技有限公司",   #字段名称：值
      "desc":"对django的概念熟悉， 熟悉python基础知识", #字段名称：值
      "comments":20,#字段名称：值
      "add_time":"2017-4-1" #字段名称：值
    }
    
    POST jobbole/job
    {
      "title":"python scrapy redis 分布式爬虫基础",
      "company_name":"玉秀科技有限公司",
      "desc":"对scrapy的概念熟悉， 熟悉redis基础知识",
      "comments":5,
      "add_time":"2017-4-2"
    }
    
    POST jobbole/job
    {
      "title":"elasticsearch打造搜索引擎",
      "company_name":"通讯科技有限公司",
      "desc":"对elasticsearch的概念熟悉",
      "comments":10,
      "add_time":"2017-4-3"
    }
    
    POST jobbole/job
    {
      "title":"pyhhon打造推荐引擎系统",
      "company_name":"智能科技有限公司",
      "desc":"熟悉推荐引擎系统算法",
      "comments":60,
      "add_time":"2017-4-4"
    }


    match查询【用的最多】
    会将我们的搜索词在当前字段设置的分词器进行分词，到当前字段查找，匹配度越高排名靠前，如果搜索词是大写字母会自动转换成小写

    #match查询
    #会将我们的搜索词进行分词，到指定字段查找，匹配度越高排名靠前
    GET jobbole/job/_search
    {
      "query": {
    "match": {
      "title": "搜索引擎"
    }
      }
    }


    term查询
    不会将我们的搜索词进行分词，将搜索词完全匹配的查询
    
    #term查询
    #不会将我们的搜索词进行分词，将搜索词完全匹配的查询
    GET jobbole/job/_search
    {
      "query": {
    "term": {
      "title":"搜索引擎"
    }
      }
    }
    
    terms查询
    传递一个数组，将数组里的词分别匹配
    
    #terms查询
    #传递一个数组，将数组里的词分别匹配
    GET jobbole/job/_search
    {
      "query": {
    "terms": {
      "title":["工程师","django","系统"]
    }
      }
    }
    
    控制查询的返回数量
    　　from从第几条数据开始
    　　size获取几条数据
    

    #控制查询的返回数量
    #from从第几条数据开始
    #size获取几条数据 
    GET jobbole/job/_search
    {
      "query": {
    "match": {
      "title": "搜索引擎"
    }
      },
      "from": 0,
      "size": 3
    }
    
    match_all查询,查询所有数据
    

    #match_all查询,查询所有数据
    GET jobbole/job/_search
    {
      "query": {
    "match_all": {}
      }
    }
    
    match_phrase查询
    短语查询
    短语查询，会将搜索词分词，放进一个列表如[python,开发]
    然后搜索的字段必须满足列表里的所有元素，才符合
    slop是设置分词后[python,开发]python 与 开发，之间隔着多少个字符算匹配
    间隔字符数小于slop设置算匹配到，间隔字符数大于slop设置不匹配

    #match_phrase查询
    #短语查询
    #短语查询，会将搜索词分词，放进一个列表如[python,开发]
    #然后搜索的字段必须满足列表里的所有元素，才符合
    #slop是设置分词后[python,开发]python 与 开发，之间隔着多少个字符算匹配
    #间隔字符数小于slop设置算匹配到，间隔字符数大于slop设置不匹配
    GET jobbole/job/_search
    {
      "query": {
    "match_phrase": {
      "title": {
    "query": "elasticsearch引擎",
    "slop":3
      }
    }
      }
    }
    
    multi_match查询
    比如可以指定多个字段
    比如查询title字段和desc字段里面包含python的关键词数据
    query设置搜索词
    fields要搜索的字段
    title^3表示权重，表示title里符合的关键词权重，是其他字段里符合的关键词权重的3倍
    
  
    #multi_match查询
    #比如可以指定多个字段
    #比如查询title字段和desc字段里面包含python的关键词数据
    #query设置搜索词
    #fields要搜索的字段
    #title^3表示权重，表示title里符合的关键词权重，是其他字段里符合的关键词权重的3倍
    GET jobbole/job/_search
    {
      "query": {
    "multi_match": {
      "query": "搜索引擎",
      "fields": ["title^3","desc"]
    }
      }
    }
    
    stored_fields设置搜索结果只显示哪些字段
    
    注意：使用stored_fields要显示的字段store属性必须为true，如果要显示的字段没有设置store属性那么默认为false，如果为false将不会显示该字段
    
   
    #stored_fields设置搜索结果只显示哪些字段
    GET jobbole/job/_search
    {
      "stored_fields": ["title","company_name"], 
      "query": {
    "multi_match": {
      "query": "搜索引擎",
      "fields": ["title^3","desc"]
    }
      }
    
    通过sort搜索结果排序
    注意：排序的字段必须是数字或者日期
    desc升序
    asc降序
    
    
    #通过sort搜索结果排序
    #注意：排序的字段必须是数字或者日期
    #desc升序
    #asc降序
    GET jobbole/job/_search
    {
      "query": {
    "match_all": {}
      },
      "sort": [{
      "comments": {
    "order": "asc"
      }
    }]
    }
    
    
    range字段值范围查询
    查询一个字段的值范围
    注意：字段值必须是数字或者时间
    gte大于等于
    ge大于
    lte小于等于
    lt小于
    boost是权重，可以给指定字段设置一个权重
    
    
    #range字段值范围查询
    #查询一个字段的值范围
    #注意：字段值必须是数字或者时间
    #gte大于等于
    #ge大于
    #lte小于等于
    #lt小于
    #boost是权重，可以给指定字段设置一个权重
    GET jobbole/job/_search
    {
      "query": {
    "range": {
      "comments": {
    "gte": 10,
    "lte": 20,
    "boost": 2.0
      }
    }
      }
    }
    
    range字段值为时间范围查询
    
    
    #range字段值为时间范围查询
    #查询一个字段的时间值范围
    #注意：字段值必须是时间
    #gte大于等于
    #ge大于
    #lte小于等于
    #lt小于
    #now为当前时间
    GET jobbole/job/_search
    {
      "query": {
    "range": {
      "add_time": {
    "gte": "2017-4-1",
    "lte": "now"
      }
    }
      }
    }
    
    wildcard查询，通配符查询
    *代表一个或者多个任意字符
    
    
    #wildcard查询，通配符查询
    #*代表一个或者多个任意字符
    GET jobbole/job/_search
    {
      "query": {
    "wildcard": {
      "title": {
    "value": "py*n",
    "boost": 2
      }
    }
      }
    }
    
    fuzzy模糊查询
    

    #fuzzy模糊搜索
    #搜索包含词的内容
    GET lagou/biao/_search
    {
      "query": {
    "fuzzy": {"title": "广告"}
      },
      "_source": ["title"]
    }
    
    
    #fuzziness设置编辑距离,编辑距离就是把要查找的字段值，编辑成查找的关键词需要编辑多少个步骤（插入、删除、替换）
    #prefix_length为关键词前面不参与变换的长度
    GET lagou/biao/_search
    {
      "query": {
    "fuzzy": {
      "title": {
    "value": "广告录音",
    "fuzziness": 2,
    "prefix_length": 2
      }
    }
      },
      "_source": ["title"]
    }



----------
### elasticsearch的bool组合查询 ###

    bool查询说明
    
    filter:[],字段的过滤，不参与打分
    must:[],如果有多个查询，都必须满足【并且】
    should:[],如果有多个查询，满足一个或者多个都匹配【或者】
    must_not:[],相反查询词一个都不满足的就匹配【取反，非】
    
    # bool查询
    # 老版本的filtered已经被bool替换
    #用 bool 包括 must should must_not filter 来完成
    #格式如下：
    
    #bool:{
    # "filter":[],  字段的过滤，不参与打分
    # "must":[],如果有多个查询，都必须满足【并且】
    # "should":[],  如果有多个查询，满足一个或者多个都匹配【或者】
    # "must_not":[],相反查询词一个都不满足的就匹配【取反，非】
    #}
    
    bool组合查询——最简单的filter过滤查询之term查询，相当于等于
    
    过滤查询到salary字段等于20的数据
    
    可以看出执行两个两个步骤，先查到所有数据，然后在查到的所有数据过滤查询到salary字段等于20的数据
    
    # bool查询
    # 老版本的filtered已经被bool替换
    #用 bool 包括 must should must_not filter 来完成
    #格式如下：
    
    #bool:{
    # "filter":[],  字段的过滤，不参与打分
    # "must":[],如果有多个查询，都必须满足
    # "should":[],  如果有多个查询，满足一个或者多个都匹配
    # "must_not":[],相反查询词一个都不满足的就匹配
    #}
    
    
    
    #简单过滤查询
    #最简单的filter过滤查询
    #如果我们要查salary字段等于20的数据
    GET jobbole/job/_search
    {
      "query": {
    "bool": {   #bool组合查询
      "must":{  #如果有多个查询词，都必须满足
    "match_all":{}  #查询所有字段
      },
      "filter": {   #filter过滤
    "term": {   #term查询，不会将我们的搜索词进行分词，将搜索词完全匹配的查询
      "salary": 20  #查询salary字段值为20
    }
      }
    }
      }
    }
    
    
    
    #简单过滤查询
    #最简单的filter过滤查询
    #如果我们要查salary字段等于20的数据
    GET jobbole/job/_search
    {
      "query": {
    "bool": {
      "must":{
    "match_all":{}
      },
      "filter": {
    "term": {
      "salary": 20
    }
      }
    }
      }
    }
    
    bool组合查询——最简单的filter过滤查询之terms查询，相当于或
    
    过滤查询到salary字段等于10或20的数据
    
    # bool查询
    # 老版本的filtered已经被bool替换
    #用 bool 包括 must should must_not filter 来完成
    #格式如下：
    
    #bool:{
    # "filter":[],  字段的过滤，不参与打分
    # "must":[],如果有多个查询，都必须满足
    # "should":[],  如果有多个查询，满足一个或者多个都匹配
    # "must_not":[],相反查询词一个都不满足的就匹配
    #}
    
    
    
    
    #简单过滤查询
    #最简单的filter过滤查询
    #如果我们要查salary字段等于20的数据
    #过滤salary字段值为10或者20的数据
    GET jobbole/job/_search
    {
      "query": {
    "bool": {
      "must":{
    "match_all":{}
      },
      "filter": {
    "terms": {
      "salary":[10,20]
    }
      }
    }
      }
    }
    注意：filter过滤里也可以用其他基本查询的
    
    _analyze测试查看分词器解析的结果
    analyzer设置分词器类型ik_max_word精细化分词，ik_smart非精细化分词
    text设置词
    #_analyze测试查看分词器解析的结果
    #analyzer设置分词器类型ik_max_word精细化分词，ik_smart非精细化分词
    #text设置词
    GET _analyze
    {
      "analyzer": "ik_max_word",
      "text": "Python网络开发工程师"
    }
    
    GET _analyze
    {
      "analyzer": "ik_smart",
      "text": "Python网络开发工程师"
    }
    bool组合查询——组合复杂查询1
    查询salary字段等于20或者title字段等于python、salary字段不等于30、并且salary字段不等于10的数据
    
    # bool查询
    # 老版本的filtered已经被bool替换
    #用 bool 包括 must should must_not filter 来完成
    #格式如下：
    
    #bool:{
    # "filter":[],  字段的过滤，不参与打分
    # "must":[],如果有多个查询，都必须满足【并且】
    # "should":[],  如果有多个查询，满足一个或者多个都匹配【或者】
    # "must_not":[],相反查询词一个都不满足的就匹配【取反，非】
    #}
    
    # 查询salary字段等于20或者title字段等于python、salary字段不等于30、并且salary字段不等于10的数据
    GET jobbole/job/_search
    {
      "query": {
    "bool": {
      "should": [
    {"term":{"salary":20}},
    {"term":{"title":"python"}}
      ],
      "must_not": [
    {"term": {"salary":30}},
    {"term": {"salary":10}}]
    }
      }
    }
    
    bool组合查询——组合复杂查询2
    查询salary字段等于20或者title字段等于python、salary字段不等于30、并且salary字段不等于10的数据
    
    
    # bool查询
    # 老版本的filtered已经被bool替换
    #用 bool 包括 must should must_not filter 来完成
    #格式如下：
    
    #bool:{
    # "filter":[],  字段的过滤，不参与打分
    # "must":[],如果有多个查询，都必须满足【并且】
    # "should":[],  如果有多个查询，满足一个或者多个都匹配【或者】
    # "must_not":[],相反查询词一个都不满足的就匹配【取反，非】
    #}
    
    # 查询title字段等于python、或者、(title字段等于elasticsearch并且salary等于30)的数据
    GET jobbole/job/_search
    {
      "query": {
    "bool": {
      "should":[
    {"term":{"title":"python"}},
    {"bool": {
      "must": [
    {"term": {"title":"elasticsearch"}},
    {"term":{"salary":30}}
      ]
    }}
      ]
    }
      }
    }
    
    
    bool组合查询——过滤空和非空
    处理null空值的方法
    
    获取tags字段，值不为空并且值不为null的数据
    
    # bool查询
    # 老版本的filtered已经被bool替换
    #用 bool 包括 must should must_not filter 来完成
    #格式如下：
    
    #bool:{
    # "filter":[],  字段的过滤，不参与打分
    # "must":[],如果有多个查询，都必须满足【并且】
    # "should":[],  如果有多个查询，满足一个或者多个都匹配【或者】
    # "must_not":[],相反查询词一个都不满足的就匹配【取反，非】
    #}
    
    
    #处理null空值的方法
    #获取tags字段，值不为空并且值不为null的数据
    GET bbole/jo/_search
    {
      "query": {
    "bool": {
      "filter": {
    "exists": {
      "field": "tags"
    }
      }
    }
      }
    }
    
    获取tags字段值为空或者为null的数据，如果数据没有tags字段也会获取
    
    # bool查询
    # 老版本的filtered已经被bool替换
    #用 bool 包括 must should must_not filter 来完成
    #格式如下：
    
    #bool:{
    # "filter":[],  字段的过滤，不参与打分
    # "must":[],如果有多个查询，都必须满足【并且】
    # "should":[],  如果有多个查询，满足一个或者多个都匹配【或者】
    # "must_not":[],相反查询词一个都不满足的就匹配【取反，非】
    #}
    
    
    #获取tags字段值为空或者为null的数据，如果数据没有tags字段也会获取
    GET bbole/jo/_search
    {
      "query": {
    "bool": {
      "must_not": {
    "exists": {
      "field": "tags"
    }
      }
    }
      }
    }
    
    
----------
### scrapy写入数据到elasticsearch中 ###

https://github.com/elastic/elasticsearch-dsl-py
pip install -i https://pypi.douban.com/simple elasticsearch-dsl


----------

# 第11章 django搭建搜索网站 #

### es完成搜索建议-搜索建议字段保存 ###

注意：因为elasticsearch-dsl源码问题，设置字段为Completion类型指定分词器时会报错，所以我们需要重写CustomAnalyzer类

只有Completion类型才是，其他类型不用，其他类型直接指定分词器即可


----------

### django实现elasticsearch的搜索建议 ###
    pip install -i https://pypi.douban.com/simple django
    pip install -i https://pypi.douban.com/simple elasticsearch-dsl
    
    1.将搜索框绑定一个事件，每输入一个字触发这个事件，获取到输入框里的内容，用ajax将输入的词请求到Django的逻辑处理函数。
    
    2.在逻辑处理函数里，将请求词用elasticsearch(搜索引擎)的fuzzy模糊查询，查询suggest字段里存在请求词的数据，将查询到的数据添加到自动补全
    
    
    #搜索自动补全fuzzy查询
    POST lagou/biao/_search?pretty
    {
      "suggest":{　　　　　　　　　　#字段名称
    "my_suggest":{　　　　　　　#自定义变量
      "text":"广告",　　　　　　#搜索词
      "completion":{
    "field":"suggest",　　#搜索字段
    "fuzzy":{
      "fuzziness":1　　　　#编辑距离
    }
      }
    }
      },
      "_source":"title"
    }

----------

### django实现elasticsearch的搜索功能 ###


----------

### django实现搜索结果分页 ###
js中：
current_page :{{ page|add:'-1' }}, //当前页码


----------
### 搜索记录、热门搜索功能实现 ###

----------

# 第12章 scrapyd部署scrapy爬虫 #

----------
    pip install -i https://pypi.douban.com/simple scrapyd
    
    scrapyd 启动服务器
    
    pip install -i https://pypi.douban.com/simple scrapyd-client
    scrapyd-client打包爬虫
    
    打开scrapy.cfg的注释：
    [deploy：bobby]
    url = http://localhost:6800/
    project = ArticleSpider
    
    windows下scrapyd-deploy命令无法使用：
    需要新建scrapyd-deploy.bat内容：
    @echo off
    "E:\downloads\envs\article_spider\Scripts\python.exe" "E:\downloads\envs\article_spider\Scripts\scrapyd-deploy" %1 %2 %3 %4 %5 %6 %7 %8 %9
    
    部署：
    获取作业状态：
    curl http://localhost:6800/daemonstatus.json
    
    {"pending": 0, "node_name": "wenjuan-PC", "running": 0, "finished": 0, "status": "ok"}
    
    调度作业：
    curl http://localhost:6800/schedule.json -d project=ArticleSpider -d spider=jobbole
    {"node_name": "wenjuan-PC", "jobid": "5847fee4924711e7a2c2ccaf78f171d4", "status": "ok"}
    
    2017-09-05 22:34:49 [twisted] CRITICAL: 
    Traceback (most recent call last):
      File "e:\downloads\envs\article_spider\lib\site-packages\twisted\internet\defer.py", line 1386, in _inlineCallbacks
    result = g.send(result)
      File "e:\downloads\envs\article_spider\lib\site-packages\scrapy\crawler.py", line 76, in crawl
    self.spider = self._create_spider(*args, **kwargs)
      File "e:\downloads\envs\article_spider\lib\site-packages\scrapy\crawler.py", line 99, in _create_spider
    return self.spidercls.from_crawler(self, *args, **kwargs)
      File "e:\downloads\envs\article_spider\lib\site-packages\scrapy\spiders\__init__.py", line 51, in from_crawler
    spider = cls(*args, **kwargs)
    TypeError: __init__() got an unexpected keyword argument '_job'
    
    解决：def __init__(self):
    改为 def __init__(self,**kwargs):
    
    删除工程：
    curl http://localhost:6800/delproject.json -d project=ArticleSpider
    {"node_name": "wenjuan-PC", "status": "ok"}
    
    重新放打包作业：
    scrapyd-deploy bobby -p ArticleSpider
    
    重新调度作业：
    curl http://localhost:6800/schedule.json -d project=ArticleSpider -d spider=jobbole
    取消作业：
    curl http://localhost:6800/cancel.json -d project=ArticleSpider -d job=4a771614924811e787c6ccaf78f171d4
    
    
----------
# 第13章 课程总结 #

----------
重点章节：
第八章 scrapy进阶开发



----------
