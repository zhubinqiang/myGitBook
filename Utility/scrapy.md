# scrapy

[TOC]

官方文档: https://docs.scrapy.org/en/latest/intro/tutorial.html

## 安装

```sh
pip3 install scrapy
```

## 创建项目
```sh
scrapy startproject tutorial

cd tutorial
scrapy genspider ruanyifeng www.ruanyifeng.com
```

items.py
```py
import scrapy


class RuanyifengItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    title = scrapy.Field()
    link = scrapy.Field()
    desc = scrapy.Field()
```

spiders/ruanyifeng_spider.py
```py
import scrapy
from tutorial.items import RuanyifengItem

class DmozSpider(scrapy.Spider):
    name = "ruanyifeng"
    allowed_domains = ['www.ruanyifeng.com']
    start_urls = [
        'http://www.ruanyifeng.com/blog/opinions/',
        'http://www.ruanyifeng.com/blog/sci-tech/'
    ]

    def parse(self, response):
        """  """
        filename = response.url.split('/')[-2]
        with open(filename, 'wb') as f:
            f.write(response.body)
        

        sel = scrapy.selector.Selector(response)
        ## sites = sel.xpath('//div[@id="alpha"]')
        sites = sel.xpath('//div[@class="module-content"]/ul/li/a')
        print(sites)
        
        items = []
        for site in sites:
            title = site.xpath('text()').extract()
            link = site.xpath('@href').extract()
            
            item = RuanyifengItem()
            item['title'] = title
            item['link'] = link
            
            items.append(item)
            # desc = 
            print('******************')
            print(title, link)
        """ """
        
        return items
```


## 开始爬取
```sh
scrapy crawl ruanyifeng
```

```sh
scrapy shell http://www.ruanyifeng.com/blog/opinions/

response.xpath('//title/text()').extract()
```

## 导出
```sh
scrapy crawl ruanyifeng -o items.json
```

使用utf-8解决中文字符
```sh
scrapy crawl ruanyifeng -o items.json -t json -s FEED_EXPORT_ENCODING=utf-8 -s FEED_EXPORT_INDENT=1
```

## 代理
### 利用中间件
在 settings.py 打开`SPIDER_MIDDLEWARES` 和 `DOWNLOADER_MIDDLEWARES`
```py
SPIDER_MIDDLEWARES = {
    'spider.middlewares.SpiderSpiderMiddleware': 543,
}

# Enable or disable downloader middlewares
# See https://docs.scrapy.org/en/latest/topics/downloader-middleware.html
DOWNLOADER_MIDDLEWARES = {
    'spider.middlewares.SpiderDownloaderMiddleware': 543,
}
```

在 middlewares.py 加入代理
```py
class SpiderDownloaderMiddleware:

    def process_request(self, request, spider):
        proxy = "http://proxy-example.com:918"
        request.meta["proxy"] = proxy
        print(f"TestProxyMiddleware --> {proxy}")
```

### 在爬虫程序中设置proxy字段
但是此方法不稳定, 感觉运行比较久
```py
    def start_requests(self):
        proxy = "http://proxy-example.com:918"
        for url in self.start_urls:
            yield scrapy.Request(url=url, callback=self.parse, meta={'proxy': proxy})

    def parse(self, response):
        pass
```

## 设置 User Agent
在谷歌浏览器地址栏中输入: `chrome://version/` 找到 **User Agent**字段

可以在 `chrome://chrome-urls/` 看到更多浏览器的配置

添加到 settings.py
```py
DEFAULT_REQUEST_HEADERS = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36'
}
```

火狐浏览器没找到在哪看，但是可以通过这个网址查询: https://useragent.buyaocha.com/
"Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0"


## issue
在ipython阶段出现 "[parso.python.diff] DEBUG: diff parser start"
```py
import logging

logging.getLogger().setLevel(logging.WARNING);
```

