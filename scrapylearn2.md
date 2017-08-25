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
