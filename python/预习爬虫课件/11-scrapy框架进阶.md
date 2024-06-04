# Scrapy框架进阶

**学习目标：**

1. 掌握 数据去重
2. 应用 scrapy中使用间件使用随机UA的方法
3. 了解 scrapy中使用代理ip的的方法
4. 掌握 中间件的使用方法
5. 了解 scrapy运行过程
6. 了解 scrapy的crawler类
7. 了解 scrapy权重问题





## 一、中间件的基本认识

### 1 scrapy中间件的分类和作用

##### 1.1 scrapy中间件的分类

根据scrapy运行流程中所在位置不同分为：

1. 下载中间件
2. 爬虫中间件

##### 1.2 scrapy中间的作用

1. 主要功能是在爬虫运行过程中进行一些处理，如对非200响应的重试（重新构造Request对象yield给引擎）
2. 也可以对header以及cookie进行更换和处理
3. 其他根据业务需求实现响应的功能

但在scrapy默认的情况下 两种中间件都在middlewares.py一个文件中

爬虫中间件使用方法和下载中间件相同，常用下载中间件

### 2 下载中间件的使用方法：

> 接下来我们对豆瓣爬虫进行修改完善，通过下载中间件来学习如何使用中间件 编写一个Downloader Middlewares和我们编写一个pipeline一样，定义一个类，然后在setting中开启

Downloader Middlewares默认的方法：

- process_request(self, request, spider)：

  - 当每个request通过下载中间件时，该方法被调用。
  - 返回None值：继续请求
  - 返回Response对象：不再请求，把response返回给引擎
  - 返回Request对象：把request对象交给调度器进行后续的请求

- process_response(self, request, response, spider)：

  - 当下载器完成http请求，传递响应给引擎的时候调用
  - 返回Resposne：交给process_response来处理
  - 返回Request对象：交给调取器继续请求

- from_crawler(cls, crawler):

  - 类似于init初始化方法,只不过这里使用的classmethod类方法

  - 可以直接crawler.settings获得参数，也可以搭配信号使用

    ​

    ​

### 3. 定义实现随机User-Agent的下载中间件

##### 3.1 在middlewares.py中完善代码

```python
import random
from myspider.settings import USER_AGENTS_LIST # 注意导入路径,请忽视pycharm的错误提示

class UserAgentMiddleware(object):
    def process_request(self, request, spider):
        user_agent = random.choice(USER_AGENTS_LIST)
        request.headers['User-Agent'] = user_agent
```

##### 3.2 在爬虫文件douban.py的每个解析函数中添加  

```
class DoubanSpider(scrapy.Spider):
    name = 'douban'

    def start_requests(self):
        for i in range(0, 2):
            url = 'https://movie.douban.com/top250?start={}&filter='.format(i * 25)
            yield scrapy.Request(url, meta={'data': i*25})

    def parse(self, response):
        print(response.request.headers)
```

##### 3.3 在settings中设置开启自定义的下载中间件，设置方法同管道

```
DOWNLOADER_MIDDLEWARES = {
   'Tencent.middlewares.UserAgentMiddleware': 543,
}
```

##### 3.4 在settings中添加UA的列表

```
USER_AGENTS_LIST = [ 
"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)", \
"Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)", \
"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)", \
"Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/523.15 (KHTML, like Gecko, Safari/419.3) Arora/0.3 (Change: 287 c9dfb30)", \
"Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6", \
"Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.2pre) Gecko/20070215 K-Ninja/2.1.1", \
"Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9) Gecko/20080705 Firefox/3.0 Kapiko/3.0", \
"Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5" ]
```



### 4 代理ip的使用

##### 4.1 思路分析

1. 代理添加的位置：request.meta中增加`proxy`字段
2. 获取一个代理ip，赋值给`request.meta['proxy']`代理池中随机选择代理ip代理ip的webapi发送请求获取一个代理ip

##### 4.2 具体实现

```
class ProxyMiddleware(object):
    def process_request(self,request,spider):
        proxy = random.choice(proxies) # proxies可以在settings.py中，也可以来源于代理ip的webapi
        # proxy = 'http://192.168.1.1:8118'
        request.meta['proxy'] = proxy
        request.meta['download_timeout'] = 1
        return None # 可以不写return
```

##### 4.3 检测代理ip是否可用

在使用了代理ip的情况下可以在下载中间件的process_response()方法中处理代理ip的使用情况，如果该代理ip不能使用可以替换其他代理ip

```
class ProxyMiddleware(object):
    def process_response(self, request, response, spider):
        if response.status != '200' and response.status != '302':
            #此时对代理ip进行操作，比如删除
            return request
```



### 5 中间件权重问题

- 请求时数字越小权重越高,越先执行    
- 响应时数字越大越先执行   

```python
class UserAgentMiddleware(object):
    def process_request(self, request, spider):
        print('ua中间件')
        # user_agent = random.choice(USER_AGENTS_LIST)
        # request.headers['User-Agent'] = user_agent

    def process_response(self, request, response, spider):
        print('响应ua中间件')
        # return None

class ProxyMiddleware(object):
    def process_request(self,request,spider):
        print('ip中间件')

    def process_response(self, request, response, spider):
        print('响应ip中间件')
        return response
```

```Python
DOWNLOADER_MIDDLEWARES = {
   'myspider.middlewares.UserAgentMiddleware': 543,
   'myspider.middlewares.ProxyMiddleware': 544,
}
```



### 6 网址去重

` dont_filter`实现了框架去重的功能

```python
    def start_requests(self):
        url_list = [
            'https://movie.douban.com/top250?start=25&filter=',
            'https://movie.douban.com/top250?start=25&filter='
        ]
        for url in url_list:
            # 框架默认对地址进行了去重
            yield scrapy.Request(url=url, dont_filter=False)
```





## 二、中间件的使用

### 1 下载器中间件配置自动化

- 采集网址:https://careers.tencent.com/search.html

##### 1.1selenium中间件

```python
import scrapy
from scrapy import signals
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By


class SeleniumMiddleware():
    def __init__(self, ):
        self.browser = webdriver.Chrome()
        self.browser.maximize_window()

    @classmethod
    def from_crawler(cls, crawler):
        # This method is used by Scrapy to create your spiders.
        s = cls()
        # 绑定事件
        crawler.signals.connect(s.spider_closed, signal=signals.spider_closed)
        return s

    def process_request(self, request, spider):
        self.browser.get(request.url)
        # 延时等待
        wait = WebDriverWait(self.browser, 10)
        wait.until(EC.presence_of_element_located((By.CLASS_NAME, 'recruit-list-link')))

        body = self.browser.page_source
        # 返回响应的html页面  不经过下载器
        return scrapy.http.HtmlResponse(url=request.url, body=body, request=request, encoding='utf-8')

    def spider_closed(self):
        self.browser.quit()
```

##### 1.3 配置文件

```python
DOWNLOADER_MIDDLEWARES = {
   'myspider.middlewares.SeleniumMiddleware': 544,
}
ITEM_PIPELINES = {
   'myspider.pipelines.TXMongoDBPipeline': 400,
}
```

##### 1.2 Item文件

```python
import scrapy
class TxspiderItem(scrapy.Item):
    title = scrapy.Field()
    department = scrapy.Field()
    addr = scrapy.Field()
    post = scrapy.Field()
    date = scrapy.Field()
    recruit_data = scrapy.Field()
```

##### 1.4 管道文件

```python
import pymongo
class TXMongoDBPipeline:
    def open_spider(self, spider):  # 在爬虫开启的时候仅执行一次
        if spider.name == 'tx':
            con = pymongo.MongoClient(host='127.0.0.1', port=27017)  # 实例化mongoclient
            self.collection = con.spiders9.tenxun  # 创建数据库名为spiders9,集合名为douban的集合操作对象

    def process_item(self, item, spider):
        print(item)
        if spider.name == 'tx':
            print('mongo保存成功')
            self.collection.insert_one(dict(item))  # 此时item对象需要先转换为字典,再插入
        # 不return的情况下，另一个权重较低的pipeline将不会获得item
        return item
```



##### 1.5 spider文件

```python
import scrapy
from scrapy import cmdline
from myspider.items import TxspiderItem


class TxSpider(scrapy.Spider):
    name = "tx"

    def start_requests(self):
        url = 'https://careers.tencent.com/search.html?index={}&keyword=Python'
        for i in range(1, 4):
            yield scrapy.Request(url=url.format(i))

    def parse(self, response, **kwargs):
        print(response.url)

        div_list = response.xpath('//div[@class="correlation-degree"]/div/div')
        for div in div_list:
            item = TxspiderItem()
            item['title'] = div.xpath('./a/h4/text()').extract_first()
            item['department'] = div.xpath('./a/p[1]/span[1]/text()').extract_first()
            item['addr'] = div.xpath('./a/p[1]/span[2]/text()').extract_first()
            item['post'] = div.xpath('./a/p[1]/span[3]/text()').extract_first()
            item['date'] = div.xpath('./a/p[1]/span[last()]/text()').extract_first()
            item['recruit_data'] = div.xpath('./a/p[2]/text()').extract_first().replace('\u2022', '')
            yield item


if __name__ == '__main__':
    cmdline.execute('scrapy crawl tx'.split())

```



## 三、 Scrapy的执行流程

### 1 scrapy 执行过程

**运行启动命令** :

1. 进入到cmdline.execute命令中进行加载
2. 检测并加载settings.py文件    
3. 加载spiders文件夹并声明当前文件夹为scrapy框架的爬虫模块
4. 加载spiders下面的爬虫文件
5. 检索爬虫文件中的爬虫名称,看是否为运行文件
6. 检测到正确的启动文件后,进入到crawler.py底层文件并执行加载对应的方法
7. 所执行的是CrawlerRunner类里面的crawl方法

```python
class Crawler:
    def __init__(self, spidercls, settings=None, init_reactor: bool = False):
        # 判断当前运行的爬虫任务是否为Spider的实例
        if isinstance(spidercls, Spider):
            raise ValueError("The spidercls argument must be a class, not an object")
		# 判断settings文件内容格式是否为字典,或者是否为空
        if isinstance(settings, dict) or settings is None:
            settings = Settings(settings)
		
        self.spidercls = spidercls
        # 拷贝默认的settings作为这个类的实例属性
        self.settings = settings.copy()
        # 将自己创建的爬虫任务的配置文件进行更新 [自己创建的加载到原有settings]
        self.spidercls.update_settings(self.settings)
        # 将信号量实例对象设置成实例属性
        self.signals = SignalManager(self)
        # 内存控制器
        self.stats = load_object(self.settings["STATS_CLASS"])(self)
        # 控制日志文件
        handler = LogCounterHandler(self, level=self.settings.get("LOG_LEVEL"))
        logging.root.addHandler(handler)
		
        d = dict(overridden_settings(self.settings))
        logger.info(
            "Overridden settings:\n%(settings)s", {"settings": pprint.pformat(d)}
        )

        if get_scrapy_root_handler() is not None:
            # scrapy root handler already installed: update it with new settings
            install_scrapy_root_handler(self.settings)
        # lambda is assigned to Crawler attribute because this way it is not
        # garbage collected after leaving __init__ scope
        # 当引擎关闭时回收垃圾
        self.__remove_handler = lambda: logging.root.removeHandler(handler)
        self.signals.connect(self.__remove_handler, signals.engine_stopped)
        # 格式化日志信息
        lf_cls = load_object(self.settings["LOG_FORMATTER"])
        self.logformatter = lf_cls.from_crawler(self)
        # 设置指纹初始化
        self.request_fingerprinter: RequestFingerprinter = create_instance(
            load_object(self.settings["REQUEST_FINGERPRINTER_CLASS"]),
            settings=self.settings,
            crawler=self,
        )
        # 加载异步库
        reactor_class = self.settings["TWISTED_REACTOR"]
        event_loop = self.settings["ASYNCIO_EVENT_LOOP"]
        if init_reactor:
            # this needs to be done after the spider settings are merged,
            # but before something imports twisted.internet.reactor
            if reactor_class:
                install_reactor(reactor_class, event_loop)
            else:
                from twisted.internet import reactor  # noqa: F401
            log_reactor_info()
        if reactor_class:
            verify_installed_reactor(reactor_class)
            if is_asyncio_reactor_installed() and event_loop:
                verify_installed_asyncio_event_loop(event_loop)
                
        # 加载组件信息并创建为实例属性
        self.extensions = ExtensionManager.from_crawler(self)

        self.settings.freeze()
        self.crawling = False
        self.spider = None
        self.engine: Optional[ExecutionEngine] = None
```

##### 1.1 Scrapy中的Crawler对象体系

- Crawler类是一个爬虫类，主要用来管理整个执行引擎ExecutionEngine类和蜘蛛类实例化



##### 1.2 Scrapy内置信号

- scrapy已经定义了常用的信号，开发人员可以在扩展类/spider/pipeline中对这些信号做关联。
  - signals.py   定义信号量
  - signalmanager.py 管理
  - utils/signal.py      真正干活的

**常见的内置信号**

```python
from scrapy.signals import *
```

```python
engine_started   # 引擎启动
engine_stopped  # 引擎停止
spider_opened   # spider开始
spider_idle  # spider进入空闲(idle)状态
spider_closed  # spider被关闭
spider_error  # spider的回调函数产生错误
request_scheduled    # 引擎调度一个 Request
request_dropped   # # 引擎丢弃一个 Request
response_received   # 引擎从downloader获取到一个新的 Response
response_downloaded  # 当一个 HTTPResponse 被下载
item_scraped   # item通过所有 Item Pipeline 后，没有被丢弃dropped
item_dropped  #   DropItem丢弃item
```





## 四、爬虫事件监控

### 1 ScrapeOps 扩展

ScrapeOps是专门用于网页抓取的监控和警报工具。通过简单的 30 秒安装,ScrapeOps 为您提供了开箱即用的 Web 抓取所需的所有监控、警报、调度和数据验证功能。

- 网址：https://scrapeops.io/app/jobs
- 需要注册自己的账号 
- 可视化页面

##### 1.1 配置ScrapeOps 

- 工具库安装

```Python
pip install scrapeops-scrapy
```

- 在你的`settings.py`文件中添加 3 行:

```python
## settings.py

## 你的平台秘钥
SCRAPEOPS_API_KEY = 'YOUR_API_KEY'

## 添加ScrapeOps扩展
EXTENSIONS = {
 'scrapeops_scrapy.extension.ScrapeOpsMonitor': 500, 
}

## 更新下载器中间件
DOWNLOADER_MIDDLEWARES = { 
'scrapeops_scrapy.middleware.retry.RetryMiddleware': 550, 
'scrapy.downloadermiddlewares.retry.RetryMiddleware': None, 
}

```

**总结:**

​	ScrapeOps是一款功能强大的网页抓取监控工具,它为您提供开箱即用的网页抓取所需的所有监控、警报、调度和数据验证功能。



### 2 scrapy发送邮箱提醒

##### 1.1 配置邮箱内容

邮箱需要配置smtp服务(QQ邮箱),用你的手机发送“配置邮件客户端”会收到一个密码，请保留这个密码。配置**smtppass**需要用到。邮件这边差不多就这么多配置。

##### 1.2 在settings文件添加配置并添加扩展

```python
EXTENSIONS = {
     'scrapy.extensions.statsmailer.StatsMailer': 500,
}
STATSMAILER_RCPTS = ['你的邮箱']
MAIL_FROM = '你的邮箱'
MAIL_HOST = 'smtp.qq.com'
MAIL_PORT = 465
MAIL_USER = '你的邮箱'
#配置好smtp服务给的密码
MAIL_PASS = ''
MAIL_SSL=True
```



##### 1.3 在重写start_requests(self)，将mail实例化

```python
class TxSpider(scrapy.Spider):
    name = "tx"
	def start_requests(self):
        self.emailer = MailSender.from_settings(self.settings)
        url = 'https://careers.tencent.com/search.html?index={}&keyword=Python'
        for i in range(1, 2):
            yield scrapy.Request(url=url.format(i))

    def parse(self, response, **kwargs):
        pass

    # 接着要实现一个close功能，当爬虫关闭的时候会调用。
    def close(self, spider):
        # 邮件标题
        subject = 'scrapy test'
        return self.emailer.send(to=["你的邮箱"], subject=subject, body='爬虫结束')
```

**注意:**

​	执行完之后会把执行的结果发送到邮箱



## 五、scrapy爬虫案例

- 目标地址:巨潮资讯
- 目标网址:http://www.cninfo.com.cn/new/commonUrl?url=disclosure/list/notice#szse
- 需求:通过scrapy请求20页数据信息

### 1 scrapy.FormRequest发送post请求

> 我们知道可以通过scrapy.Request()指定method、body参数来发送post请求；那么也可以使用scrapy.FormRequest()来发送post请求

##### 1.1 scrapy.FormRequest()的使用

> 通过scrapy.FormRequest能够发送post请求，同时需要添加fromdata参数作为请求体，以及callback

```python
url = 'http://www.cninfo.com.cn/new/disclosure'
for i in range(1, 2):
    data = {
        'column': 'szse_latest',
        'pageNum': str(i),
        'pageSize': '30',
        'sortName': '',
        'sortType': '',
        'clusterFlag': 'true',
    }
    yield scrapy.FormRequest(url=url, formdata=data, callback=self.parse)
```

### 2 项目实战

##### 2.1 spider文件

```python
import scrapy
from scrapy import cmdline
from scrapy.mail import MailSender

from scrapy_project.items import ScrapyProjectItem


class JcSpider(scrapy.Spider):
    name = 'jc'

    # allowed_domains = ['http://www.cninfo.com.cn/']
    # start_urls = ['http://http://www.cninfo.com.cn//']

    def start_requests(self):
        self.emailer = MailSender.from_settings(self.settings)
        url = 'http://www.cninfo.com.cn/new/disclosure'
        for i in range(1, 16):
            data = {
                'column': 'szse_latest',
                'pageNum': str(i),
                'pageSize': '30',
                'sortName': '',
                'sortType': '',
                'clusterFlag': 'true',
            }
            yield scrapy.FormRequest(url=url, formdata=data, callback=self.parse)

    def parse(self, response, **kwargs):

        for info_list in response.json()['classifiedAnnouncements']:
            for info in info_list:
                item = ScrapyProjectItem()
                item['announcementTitle'] = info['announcementTitle']
                item['announcementTypeName'] = info['announcementTypeName']
                item['batchNum'] = info['batchNum']
                item['secName'] = info['secName']
                item['adjunctType'] = info['adjunctType']
                print(item)
                yield item

    def close(self, spider):
        # 邮件标题
        subject = 'scrapy test'
        return self.emailer.send(to=["1641324821@qq.com"], subject=subject, body='爬虫结束')


if __name__ == '__main__':
    cmdline.execute('scrapy crawl jc --nolog'.split())

```

##### 2.2 items文件

```python
import scrapy

class ScrapyProjectItem(scrapy.Item):
    announcementTitle = scrapy.Field()
    announcementTypeName = scrapy.Field()
    batchNum = scrapy.Field()
    secName = scrapy.Field()
    adjunctType = scrapy.Field()
```

##### 2.3 pipelines文件

```python
import pymysql
import pymongo


class ScrapyProjectMySQLPipeline:
    def open_spider(self, spider):
        if spider.name == 'jc':
            self.db = pymysql.connect(host="localhost", user="root", password="root", db="spiders9")
            self.cursor = self.db.cursor()
            sql = '''
                CREATE TABLE IF NOT EXISTS juchao(
                    id int primary key auto_increment not null,
                    announcementTitle VARCHAR(255) NOT NULL, 
                    announcementTypeName VARCHAR(255) NOT NULL, 
                    batchNum VARCHAR(255) NOT NULL, 
                    secName VARCHAR(255) NOT NULL, 
                    adjunctType VARCHAR(255) NOT NULL)
                                '''
            try:
                self.cursor.execute(sql)
                print("CREATE TABLE SUCCESS.")
            except Exception as ex:
                print(f"CREATE TABLE FAILED,CASE:{ex}")

    def process_item(self, item, spider):
        sql = 'INSERT INTO juchao(id, announcementTitle, announcementTypeName, batchNum, secName, adjunctType) values(%s, %s, %s, %s, %s, %s)'
        try:
            self.cursor.execute(sql, (
            0, item['announcementTitle'], item['announcementTypeName'], item['batchNum'], item['secName'],
            item['adjunctType']))
            # 提交到数据库执行
            self.db.commit()
            print('mysql数据插入成功...')
        except Exception as e:
            print(f'数据插入失败: {e}')
            # 如果发生错误就回滚
            self.db.rollback()
        # 不return的情况下，另一个权重较低的pipeline将不会获得item
        return item

    def close_spider(self, spider):
        if spider.name == 'jc':
            self.db.close()


class ScrapyProjectMongoDBPipeline:
    def open_spider(self, spider):
        if spider.name == 'jc':
            con = pymongo.MongoClient(host='127.0.0.1', port=27017)
            self.collection = con.spiders9.juchao

    def process_item(self, item, spider):
        if spider.name == 'jc':
            self.collection.insert_one(dict(item))
        return item

```

##### 5.4 settings文件

```python

BOT_NAME = 'scrapy_project'

SPIDER_MODULES = ['scrapy_project.spiders']
NEWSPIDER_MODULE = 'scrapy_project.spiders'

# 邮箱配置
STATSMAILER_RCPTS = ['1641324821@qq.com']
MAIL_FROM = '1641324821@qq.com'
MAIL_HOST = 'smtp.qq.com'
MAIL_PORT = 465
MAIL_USER = '1641324821@qq.com'
# 配置好smtp服务给的密码
MAIL_PASS = 'vewqyycamwpkeehh'
MAIL_SSL = True


SCRAPEOPS_API_KEY = 'd6758257-3ce4-44fc-bd9d-fd9ea5083254'

EXTENSIONS = {
    # 邮件拓展
    'scrapy.extensions.statsmailer.StatsMailer': 501,
    # ScrapeOps 拓展
    'scrapeops_scrapy.extension.ScrapeOpsMonitor': 500,
}

ROBOTSTXT_OBEY = False

# ua池
USER_AGENTS_LIST = [
    "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)", \
    "Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)", \
    "Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)", \
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/523.15 (KHTML, like Gecko, Safari/419.3) Arora/0.3 (Change: 287 c9dfb30)", \
    "Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6", \
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.2pre) Gecko/20070215 K-Ninja/2.1.1", \
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9) Gecko/20080705 Firefox/3.0 Kapiko/3.0", \
    "Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5"
]

DOWNLOADER_MIDDLEWARES = {
   'scrapy_project.middlewares.UserAgentMiddleware': 544,
}
ITEM_PIPELINES = {
   'scrapy_project.pipelines.ScrapyProjectMySQLPipeline': 300,
   'scrapy_project.pipelines.ScrapyProjectMongoDBPipeline': 301,
}

```



