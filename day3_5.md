## 补充一下，scrapy的结构说明

### 1. 项目目录
>### scrapy.cfg

项目的配置文件（如配置默认项目，其他项目）
```python
[settings]
# 设置项目名称
default = tutorial.settings
project1 = myproject1.settings
project2 = myproject2.settings


[deploy]
#url = http://localhost:6800/
project = tutorial
```

>### tutorial/

该项目的python模块。在此放入代码（核心）


>### tutorial/items.py

这是创建容器的地方，爬取的信息分别放到不同容器里
```python
import scrapy
class MyspiderItem(scrapy.Item):
    title = scrapy.Field()
    artist = scrapy.Field()
```

>### tutorial/pipelines.py

项目中的pipelines文件,处理爬到的内容
```python
from itemadapter import ItemAdapter
 
#管道,负责item后期的处理或保存
class MyspiderPipeline:
 
    #定义初始化参数(可以省略)
    def __init__(self):
        self.file=open("music.txt","a")
 
    #管道每次接受到的item后执行的方法(必须实现)
    def process_item(self, item, spider):
        content=str(item)+"\n"
        self.file.write(content) #写入数据到本地
        return item
 
    #当爬虫结束时执行的方法(可以省略)
    def close_spider(self,spider):
        self.spider.close()
```
>### tutorial/settings.py


项目的设置文件.（目前未知）

>### tutorial/spiders/
放爬虫的地方

>### 项目整体框架 [参考网络](https://cloud.tencent.com/developer/article/1977027#:~:text=%E4%B8%89%E3%80%81Scrapy%E6%A1%86%E6%9E%B6%E7%9A%84%E6%95%B4%E4%BD%93%E6%9E%B6%E6%9E%84%E5%92%8C%E7%BB%84%E6%88%90%201%20Spiders%EF%BC%9A%E7%88%AC%E8%99%AB%EF%BC%8C%E5%AE%9A%E4%B9%89%E4%BA%86%E7%88%AC%E5%8F%96%E7%9A%84%E9%80%BB%E8%BE%91%E5%92%8C%E7%BD%91%E9%A1%B5%E5%86%85%E5%AE%B9%E7%9A%84%E8%A7%A3%E6%9E%90%E8%A7%84%E5%88%99%EF%BC%8C%E4%B8%BB%E8%A6%81%E8%B4%9F%E8%B4%A3%E8%A7%A3%E6%9E%90%E5%93%8D%E5%BA%94%E5%B9%B6%E7%94%9F%E6%88%90%E7%BB%93%E6%9E%9C%E5%92%8C%E6%96%B0%E7%9A%84%E8%AF%B7%E6%B1%82%202%20Engine%EF%BC%9A%E5%BC%95%E6%93%8E%EF%BC%8C%E5%A4%84%E7%90%86%E6%95%B4%E4%B8%AA%E7%B3%BB%E7%BB%9F%E7%9A%84%E6%95%B0%E6%8D%AE%E6%B5%81%E5%A4%84%E7%90%86%EF%BC%8C%E5%87%BA%E5%8F%91%E4%BA%8B%E7%89%A9%EF%BC%8C%E6%A1%86%E6%9E%B6%E7%9A%84%E6%A0%B8%E5%BF%83%E3%80%82,3%20Scheduler%EF%BC%9A%E8%B0%83%E5%BA%A6%E5%99%A8%EF%BC%8C%E6%8E%A5%E5%8F%97%E5%BC%95%E6%93%8E%E5%8F%91%E8%BF%87%E6%9D%A5%E7%9A%84%E8%AF%B7%E6%B1%82%EF%BC%8C%E5%B9%B6%E5%B0%86%E5%85%B6%E5%8A%A0%E5%85%A5%E9%98%9F%E5%88%97%E4%B8%AD%EF%BC%8C%E5%9C%A8%E5%BC%95%E6%93%8E%E5%86%8D%E6%AC%A1%E8%AF%B7%E6%B1%82%E6%97%B6%E5%B0%86%E8%AF%B7%E6%B1%82%E6%8F%90%E4%BE%9B%E7%BB%99%E5%BC%95%E6%93%8E%204%20Downloader%EF%BC%9A%E4%B8%8B%E8%BD%BD%E5%99%A8%EF%BC%8C%E4%B8%8B%E8%BD%BD%E7%BD%91%E9%A1%B5%E5%86%85%E5%AE%B9%EF%BC%8C%E5%B9%B6%E5%B0%86%E4%B8%8B%E8%BD%BD%E5%86%85%E5%AE%B9%E8%BF%94%E5%9B%9E%E7%BB%99spider%20%E3%81%9D%E3%81%AE%E4%BB%96%E3%81%AE%E3%82%A2%E3%82%A4%E3%83%86%E3%83%A0)
|英文|定义|职能|
|-|-|-|
|Spiders|爬虫，定义了爬取的逻辑和网页内容的解析规则|***解析响应，生成结果和新的请求***|
|Engine|引擎，框架的核心|***解整个系统的数据流处理***|
|Spider Middlewares|spider中间件，middlewares.py里实现|***处理spider的输入响应，输出结果，新请求***|
|ItemPipeline|项目管道|***清洗，验证和向数据库中存储数据***|
|Downloader Middlewares|下载器|***下载网页内容，并将下载内容返回给spider***|
|Downloader Middlewares|下载中间件|***Scrapy的Request和Requesponse之间的处理模块***|
|Scheduler|调度器|***接受引擎的请求，再次请求时将请求提供给引擎***|

#### 1. spider的yeild将request发送给engine
#### 2. engine对request不做任何处理发送给scheduler
#### 3. scheduler，生成request交给engine
#### 4. engine拿到request，通过middleware发送给downloader
#### 5. downloader在获取到response之后，又经过middleware发送给engine
#### 6. engine获取到response之后，返回给spider，spider的parse()方法对获取到的response进行处理，解析出items或者requests
#### 7. 将解析出来的items或者requests发送给engine
#### 8. engine获取到items或者requests，将items发送给ItemPipeline，将requests发送给scheduler（ps，只有调度器中不存在request时，程序才停止，及时请求失败scrapy也会重新进行请求）




