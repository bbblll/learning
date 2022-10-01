# 学习爬虫第三天——命令行工具

今天当然是摆烂继续学官网，谁让官网是最好的学习资料捏

> scrapy.cfg

```python
[settings]
# 设置项目名称
default = tutorial.settings

[deploy]
#url = http://localhost:6800/
project = tutorial
```
该文件位于根目录下，和项目文件同级。`tutorial`就是我们昨天创建的项目，我们也能指定多项目（同根目录情况）
```python
[settings]
# 设置项目名称
default = tutorial.settings
project1 = myproject1.settings
project2 = myproject2.settings
```
这里就指定了三个项目，接下来我们可以输入如下代码，查看项目


+ 当然我们应该先创建该项目
```cmd
scrapy startproject myproject1
scrapy startproject myproject2
```
现在我们的目录结构如下
```
SCRAPY 01/
  scrapy.cfg
  tutorial/
  myproject1/
  myproject2/
```
我们继续，

试试获取项目名称
```
$ scrapy settings --get BOT_NAME
tutorial
```
修改项目(官网写的export（mac），实际该用set（win）)
```
$ set SCRAPY_PROJECT=project2
$ scrapy settings --get BOT_NAME
myproject2
```
>### 获取命令
```
$ scrapy
```

可以获取可用操作信息
```
Usage:
  scrapy <command> [options] [args]

Available commands:
  bench         Run quick benchmark test
  check         Check spider contracts
  commands
  crawl         Run a spider
  edit          Edit spider
  fetch         Fetch a URL using the Scrapy downloader
  genspider     Generate new spider using pre-defined templates
  list          List available spiders
  parse         Parse URL (using its spider) and print the results
  runspider     Run a self-contained spider (without creating a project)
  settings      Get settings values
  shell         Interactive scraping console
  startproject  Create new project
  version       Print Scrapy version
  view          Open URL in browser, as seen by Scrapy

Use "scrapy <command> -h" to see more info about a command
```

#### 比如用命令创建一只新的蜘蛛
```
$ scrapy genspider mydomain mydomain.com
Created spider 'mydomain' using template 'basic' in module:
  myproject2.spiders.mydomain
```
效果如下
> myproject2\spiders\mydomain.py
```python
import scrapy

class MydomainSpider(scrapy.Spider):
    name = 'mydomain'
    allowed_domains = ['mydomain.com']
    start_urls = ['http://mydomain.com/']

    def parse(self, response):
        pass
```
>## 基因蜘蛛

顾名思义，就是根据模板创建新的蜘蛛（第一眼真的吓尿了小爷，这特么啥??）

1. 先查看有哪些模板
```
$ scrapy genspider -l
Available templates:
  basic
  crawl
  csvfeed
  xmlfeed
```

可以看到有我们创建上一个蜘蛛的模板`basic`,那我们来创建另一个模板`crawl`

```
$ scrapy genspider -t crawl spider2 scrapy.org

Created spider 'spider2' using template 'crawl' in module:
  myproject2.spiders.spider2
```
得到新蜘蛛
> myproject2\spiders\spider2.py
```python
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule


class Spider2Spider(CrawlSpider):
    name = 'spider2'
    allowed_domains = ['scrapy.org']
    start_urls = ['http://scrapy.org/']

    rules = (
        Rule(LinkExtractor(allow=r'Items/'), callback='parse_item', follow=True),
    )

    def parse_item(self, response):
        item = {}
        #item['domain_id'] = response.xpath('//input[@id="sid"]/@value').get()
        #item['name'] = response.xpath('//div[@id="name"]').get()
        #item['description'] = response.xpath('//div[@id="description"]').get()
        return item
```
>### 使用蜘蛛
```
$ scrapy crawl myspider

[ ... myspider starts crawling ... ]
```

昨天应该用过了。。。
>### 蜘蛛列表
```
$ scrapy list

mydomain
spider2
```
没啥毛病，是我们刚刚创建的蜘蛛。

>## 编辑蜘蛛
```
$ scrapy edit spider1

'%s' 不是内部或外部命令，也不是可运行的程序
或批处理文件。
```
本人使用情况，报错

>### 取来
```
$ scrapy fetch --nolog http://www.example.com/some/page.html
[ ... html content here ... ]
$ scrapy fetch --nolog --headers http://www.example.com/
[请求头内容（来回）]
```
和上节课的拿respose方法类似，但这里直接拿到了请求头
>### 看法
```
$ scrapy view http://www.example.com/some/page.html

[会直接跳到页面]
```
用来检查爬虫看到的界面和用户看到的界面是否一致

>### 获取设置值
```
$ scrapy settings --get BOT_NAME
tutorial

$ scrapy settings --get DOWNLOAD_DELAY
0
```
之前也用过，就是获取设置对应目标的值

>### 运行蜘蛛
```
$ scrapy runspider tutorial\spiders\quotes_spider.py

[爬取日志，和昨天一样。。。]
```
整体用下来没啥感触，感觉学了个寂寞

## 完结撒花