# 第1章 课程介绍
***
### 基于数据爬取 数据分析服务 医疗病例分析 互联网金融 自然语言处理 数据建模 信息聚类
***
### 知识体系 学习流程
1. 环境配置和基础知识铺垫
2. 爬取真实数据
3. scrapy 突破反爬虫技术
4. scrapy 进阶
5. scrapy redis 分布式爬虫
6. elasticsearch  django 实现搜索引擎
***
### 爬虫基础知识
1. 正则表达式
2. 深度优先和广度优先遍历算法
3. url 去重的常见策略
***
### scrapy spider item item loader pipeline feed export crawlspider
***
### scrapy 进阶开发
scrapy的原理 
基于scrapy 的中间件开发
动态网站的抓取处理
将selenium和phantomjs 集成到scrapy中
scrapy log配置
email 发送
scrapy 信号
***
### 课程体验
* 开发爬虫所需要用到的技术以及网站分析技巧
* 理解scrapy的原理和所有组件的使用以及分布式爬虫scrapy-redis的使用和原理
* 理解分布式开源搜索引擎elasticsearch 的使用以及搜索引擎的原理
* 体验Django 如何快速搭建网站


# 第2章 windows下搭建开发环境
***
### IDE  pycharm的安装使用
### 数据库  mysql和navicat的安装和使用
### windows 和linux下安装python2 和python3
### virtualenv 和 virtual恶女wrapper虚拟环境的安装和配置
* pip install -i https://pypi.douan.com/simple/ virtualenv
* pip uninstall virtualenv
* pip install -i https://pypi.douan.com/simple/ virtualenvwrapper
* mkvirtualenv  --python = 路径 ppp     
* workon ppp 
* deactivate
***

# 第3章 爬虫基础知识回顾
***
### 技术选型 爬虫能做什么
* `scrapy vs  requests ＋beautifulsoup`
1. `requests 和 beautifulsoup 都是库，scrapy 是框架`
2. scrapy 框架中可以加入requests 和 beautifulsoup
3. scrapy 基于twisted ，性能是最大优势
4. scrapy 方便扩展，提供了很多内置的功能
5. scrapy 内置的css和xpath selector 非常方便，beautifulsoup 最大的缺点是慢
* 网页分类
1. 静态网页
2. 动态网页
3. webservice（restapi）
* 爬虫能做什么
1. 搜索引擎 -- 百度。google、垂直领域搜索引擎
2. 推荐引擎 -- 今日头条
3. 机器学习的数据样本
4. 数据分析（金融数据分析）、舆情分析等
***
### 正则表达式
***
### 深度优先和广度优先原理
***
### url去重方法
***
### 彻底搞清楚unicode和utf8编码
***

