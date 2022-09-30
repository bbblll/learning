# 今天开始第二次的爬虫
本来想在GitHub上找个挑战性更强一点的新项目，但是奈何大部分项目都年代久远，很多库连pip都pip不下来（难受）,于是乎突然发现有爬虫框架这种东西，所以今天想要用睡前一小时来入门一下比较流行的爬虫框架`scrapy`。
## 以下操作皆参照[scrapy官网](https://www.osgeo.cn/scrapy/intro/tutorial.html)
>### 第一步： 下载库

```cmd
pip install scrapy<br>
```

如果没彪红,那么恭喜你和我一样，安装成功

>### 第二步：搭建框架
```cmd
scrapy startproject tutorial
```
我这得到了一个名为`tutorial`的文件夹,文件夹结构如下
```python
tutorial/
    scrapy.cfg
    tutorial/ 
        __init__.py
        items.py
        middlewares.py
        pipelines.py
        settings.py
        spiders/
            __init__.py
```
>### 第三步：编写第一只爬虫

在`tutorial/spiders`目录下创建`quotes_spider.py`文件，里面写上如下内容（官网代码，加上我的注释）

```python
import scrapy

# 继承于scrapy.Spider类
class QuotesSpider(scrapy.Spider):
  # 标识蜘蛛，必须唯一
    name = "quotes"

    def start_requests(self):
      # 网址列表
        urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]
        # 遍历并且请求列表内的网址
        for url in urls:
          # 该函数会返回一个数组
          # 请求结果会通过parse函数进行解析
          # callback是回调函数的意思
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
      # 通过回调接收到请求结果response
      # 这句话通过response的url参数拿到页码
        page = response.url.split("/")[-2]
        # 设置文件名
        filename = f'quotes-{page}.html'
        # 创建文件
        with open(filename, 'wb') as f:
          # 写入文件
            f.write(response.body)
            # 应该是输出日志
        self.log(f'Saved file {filename}')
```
可以看到以上代码，将两个网页的数据爬下来了，但是并未解析，`parse`函数只是保存文件。

>### 第四步：运行框架

如果你已经在`tutorial`目录下则不需要运行第一行命令。
此命令意思为运行名为的`quotes`我们刚刚添加的爬虫。
```cmd
cd tutorial
scrapy crawl quotes
```

此时我的`tutorial`目录下多了两个`html`文件，说明成功运行框架

## 关于代码简化
我们将`quotes_spider.py`的文件内容做如下修改
```python
import scrapy

# 继承于scrapy.Spider类
class QuotesSpider(scrapy.Spider):
  # 标识蜘蛛，必须唯一
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
      # 通过回调接收到请求结果response
      # 这句话通过response的url参数拿到页码
        page = response.url.split("/")[-2]
        # 设置文件名
        filename = f'quotes-{page}.html'
        # 创建文件
        with open(filename, 'wb') as f:
          # 写入文件
            f.write(response.body)
            # 应该是输出日志
        self.log(f'Saved file {filename}')
```

可以看到函数中没有`start_requests`方法了，取而代之的是`start_urls`数组，可以知道，框架被执行1时会默认在这里取网址。此时，我们再执行上面的第四步（记得先删除原来的html文件），本人这也在此得到了之前的html文件。

## 关于在终端进行网页解析

> 先链接到网址
```cmd
scrapy shell "http://quotes.toscrape.com/page/1/"  
```

> 自行输入以下代码进行尝试

```cmd
response.css('title')
response.css('title::text').getall()
response.css('title').getall()
response.css('title::text').get()
response.css('title::text')[0].get()
response.css("noelement").get()
response.css('title::text').re(r'Quotes.*')
response.css('title::text').re(r'Q\w+')
```

`response.css('title')`会返回`title`标签出现到最后的所有内容，

`response.css('title::text')`会返回`title`标签后出现的第一段文本，

`response.css('title').getall()`会返回数组，

`response.css('title::text').get()`会返回数组第一个元素，

`response.css('title::text').re(r'Q\w+')`后面接了正则搜索，表示找到Q开头的单词。

>### 也能用xpath进行解析

```cmd
response.xpath('//title')
response.xpath('//title/text()').get()
```
试了一下和上面结果差不多

## 接下来是一些案例

> 案例1

```cmd
response.css("div.quote")
quote = response.css("div.quote")[0]
text = quote.css("span.text::text").get()
author = quote.css("small.author::text").get()
text
author
```
>输出

[1]“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”

[2]Albert Einstein

> 案例2

```cmd
tags = quote.css("div.tags a.tag::text").getall()
tags
```
>输出

[1]['change', 'deep-thoughts', 'thinking', 'world']

> 案例3

```cmd
for quote in response.css("div.quote"):
    text = quote.css("span.text::text").get()
    author = quote.css("small.author::text").get()
    tags = quote.css("div.tags a.tag::text").getall()
    print(dict(text=text, author=author, tags=tags))
```
>输出

[1]一大堆东西。。。（字典构成的数组）

## 为框架添加解析部分
&emsp; 绕了一大圈，我们终于可以为框架加上解析部分的代码了。

> tutorial\spiders\quotes_spider.py
```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('small.author::text').get(),
                'tags': quote.css('div.tags a.tag::text').getall(),
            }
```
运行该代码我们可以在终端找到输出日志(列出部分)
```cmd
2022-10-01 01:18:24 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>{'text': '“The world as we have created it is a process of our thinking. It cannot be changed without 
changing our thinking.”', 'author': 'Albert Einstein', 'tags': ['change', 'deep-thoughts', 'thinking', 'world']}
2022-10-01 01:18:24 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>{'text': '“It is our choices, Harry, that show what we truly are, far more than our abilities.”', 'author': 'J.K. Rowling', 'tags': ['abilities', 'choices']}
```
>### 存储数据
```cmd
scrapy crawl quotes -O quotes.json
```
会得到json文件如下
> tutorial\quotes.json
```json
[
{"text": "\u201cThe world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.\u201d", "author": "Albert Einstein", "tags": ["change", "deep-thoughts", "thinking", "world"]},
{"text": "\u201cIt is our choices, Harry, that show what we truly are, far more than our abilities.\u201d", "author": "J.K. Rowling", "tags": ["abilities", "choices"]},
{"text": "\u201cThere are only two ways to live your life. One is as though nothing is a miracle. The other is as though everything is a miracle.\u201d", "author": "Albert Einstein", "tags": ["inspirational", "life", "live", "miracle", "miracles"]},
{"text": "\u201cThe person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.\u201d", "author": "Jane Austen", "tags": ["aliteracy", "books", "classic", "humor"]},
{"text": "\u201cImperfection is beauty, madness is genius and it's better to be absolutely ridiculous than absolutely boring.\u201d", "author": "Marilyn Monroe", "tags": ["be-yourself", "inspirational"]},
{"text": "\u201cTry not to become a man of success. Rather become a man of value.\u201d", "author": "Albert Einstein", "tags": ["adulthood", "success", "value"]},
{"text": "\u201cIt is better to be hated for what you are than to be loved for what you are not.\u201d", "author": "Andr\u00e9 Gide", "tags": ["life", "love"]},
{"text": "\u201cI have not failed. I've just found 10,000 ways that won't work.\u201d", "author": "Thomas A. Edison", "tags": ["edison", "failure", "inspirational", "paraphrased"]},
{"text": "\u201cA woman is like a tea bag; you never know how strong it is until it's in hot water.\u201d", "author": "Eleanor Roosevelt", "tags": ["misattributed-eleanor-roosevelt"]},
{"text": "\u201cA day without sunshine is like, you know, night.\u201d", "author": "Steve Martin", "tags": ["humor", "obvious", "simile"]},
{"text": "\u201cThis life is what you make it. No matter what, you're going to mess up sometimes, it's a universal truth. But the good part is you get to decide how you're going to mess it up. Girls will be your friends - they'll act like it anyway. But just remember, some come, some go. The ones that stay with you through everything - they're your true best friends. Don't let go of them. Also remember, sisters make the best friends in the world. As for lovers, well, they'll come and go too. And baby, I hate to say it, most of them - actually pretty much all of them are going to break your heart, but you can't give up because if you give up, you'll never find your soulmate. You'll never find that half who makes you whole and that goes for everything. Just because you fail once, doesn't mean you're gonna fail at everything. Keep trying, hold on, and always, always, always believe in yourself, because if you don't, then who will, sweetie? So keep your head high, keep your chin up, and most importantly, keep smiling, because life's a beautiful thing and there's so much to smile about.\u201d", "author": "Marilyn Monroe", "tags": ["friends", "heartbreak", "inspirational", "life", "love", "sisters"]},
{"text": "\u201cIt takes a great deal of bravery to stand up to our enemies, but just as much to stand up to our friends.\u201d", "author": "J.K. Rowling", "tags": ["courage", "friends"]},
{"text": "\u201cIf you can't explain it to a six year old, you don't understand it yourself.\u201d", "author": "Albert Einstein", "tags": ["simplicity", "understand"]},
{"text": "\u201cYou may not be her first, her last, or her only. She loved before she may love again. But if she loves you now, what else matters? She's not perfect\u2014you aren't either, and the two of you may never be perfect together but if she can make you laugh, cause you to think twice, and admit to being human and making mistakes, hold onto her and give her the most you can. She may not be thinking about you every second of the day, but she will give you a part of her that she knows you can break\u2014her heart. So don't hurt her, don't change her, don't analyze and don't expect more than she can give. Smile when she makes you happy, let her know when she makes you mad, and miss her when she's not there.\u201d", "author": "Bob Marley", "tags": ["love"]},
{"text": "\u201cI like nonsense, it wakes up the brain cells. Fantasy is a necessary ingredient in living.\u201d", "author": "Dr. Seuss", "tags": ["fantasy"]},
{"text": "\u201cI may not have gone where I intended to go, but I think I have ended up where I needed to be.\u201d", "author": "Douglas Adams", "tags": ["life", "navigation"]},
{"text": "\u201cThe opposite of love is not hate, it's indifference. The opposite of art is not ugliness, it's indifference. The opposite of faith is not heresy, it's indifference. And the opposite of life is not death, it's indifference.\u201d", "author": "Elie Wiesel", "tags": ["activism", "apathy", "hate", "indifference", "inspirational", "love", "opposite", "philosophy"]},
{"text": "\u201cIt is not a lack of love, but a lack of friendship that makes unhappy marriages.\u201d", "author": "Friedrich Nietzsche", "tags": ["friendship", "lack-of-friendship", "lack-of-love", "love", "marriage", "unhappy-marriage"]},
{"text": "\u201cGood friends, good books, and a sleepy conscience: this is the ideal life.\u201d", "author": "Mark Twain", "tags": ["books", "contentment", "friends", "friendship", "life"]},
{"text": "\u201cLife is what happens to us while we are making other plans.\u201d", "author": "Allen Saunders", "tags": ["fate", "life", "misattributed-john-lennon", "planning", "plans"]}
]
```
##### 上一条命令`O`会覆盖同名文件，而`o`则是在同名文件内继续写入(建议使用JSON Line文件，JSON文件再次写入会破坏原有格式)

```cmd
scrapy crawl quotes -o quotes.jl
```

输出如下文件
> tutorial\quotes.jl

```json
{"text": "\u201cThe world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.\u201d", "author": "Albert Einstein", "tags": ["change", "deep-thoughts", "thinking", "world"]}
{"text": "\u201cIt is our choices, Harry, that show what we truly are, far more than our abilities.\u201d", "author": "J.K. Rowling", "tags": ["abilities", "choices"]}
{"text": "\u201cThere are only two ways to live your life. One is as though nothing is a miracle. The other is as though everything is a miracle.\u201d", "author": "Albert Einstein", "tags": ["inspirational", "life", "live", "miracle", "miracles"]}
{"text": "\u201cThe person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.\u201d", "author": "Jane Austen", "tags": ["aliteracy", "books", "classic", "humor"]}
{"text": "\u201cImperfection is beauty, madness is genius and it's better to be absolutely ridiculous than absolutely boring.\u201d", "author": "Marilyn Monroe", "tags": ["be-yourself", "inspirational"]}
{"text": "\u201cTry not to become a man of success. Rather become a man of value.\u201d", "author": "Albert Einstein", "tags": ["adulthood", "success", "value"]}
{"text": "\u201cIt is better to be hated for what you are than to be loved for what you are not.\u201d", "author": "Andr\u00e9 Gide", "tags": ["life", "love"]}
{"text": "\u201cI have not failed. I've just found 10,000 ways that won't work.\u201d", "author": "Thomas A. Edison", "tags": ["edison", "failure", "inspirational", "paraphrased"]}
{"text": "\u201cA woman is like a tea bag; you never know how strong it is until it's in hot water.\u201d", "author": "Eleanor Roosevelt", "tags": ["misattributed-eleanor-roosevelt"]}
{"text": "\u201cA day without sunshine is like, you know, night.\u201d", "author": "Steve Martin", "tags": ["humor", "obvious", "simile"]}
{"text": "\u201cThis life is what you make it. No matter what, you're going to mess up sometimes, it's a universal truth. But the good part is you get to decide how you're going to mess it up. Girls will be your friends - they'll act like it anyway. But just remember, some come, some go. The ones that stay with you through everything - they're your true best friends. Don't let go of them. Also remember, sisters make the best friends in the world. As for lovers, well, they'll come and go too. And baby, I hate to say it, most of them - actually pretty much all of them are going to break your heart, but you can't give up because if you give up, you'll never find your soulmate. You'll never find that half who makes you whole and that goes for everything. Just because you fail once, doesn't mean you're gonna fail at everything. Keep trying, hold on, and always, always, always believe in yourself, because if you don't, then who will, sweetie? So keep your head high, keep your chin up, and most importantly, keep smiling, because life's a beautiful thing and there's so much to smile about.\u201d", "author": "Marilyn Monroe", "tags": ["friends", "heartbreak", "inspirational", "life", "love", "sisters"]}
{"text": "\u201cIt takes a great deal of bravery to stand up to our enemies, but just as much to stand up to our friends.\u201d", "author": "J.K. Rowling", "tags": ["courage", "friends"]}
{"text": "\u201cIf you can't explain it to a six year old, you don't understand it yourself.\u201d", "author": "Albert Einstein", "tags": ["simplicity", "understand"]}
{"text": "\u201cYou may not be her first, her last, or her only. She loved before she may love again. But if she loves you now, what else matters? She's not perfect\u2014you aren't either, and the two of you may never be perfect together but if she can make you laugh, cause you to think twice, and admit to being human and making mistakes, hold onto her and give her the most you can. She may not be thinking about you every second of the day, but she will give you a part of her that she knows you can break\u2014her heart. So don't hurt her, don't change her, don't analyze and don't expect more than she can give. Smile when she makes you happy, let her know when she makes you mad, and miss her when she's not there.\u201d", "author": "Bob Marley", "tags": ["love"]}
{"text": "\u201cI like nonsense, it wakes up the brain cells. Fantasy is a necessary ingredient in living.\u201d", "author": "Dr. Seuss", "tags": ["fantasy"]}
{"text": "\u201cI may not have gone where I intended to go, but I think I have ended up where I needed to be.\u201d", "author": "Douglas Adams", "tags": ["life", "navigation"]}
{"text": "\u201cThe opposite of love is not hate, it's indifference. The opposite of art is not ugliness, it's indifference. The opposite of faith is not heresy, it's indifference. And the opposite of life is not death, it's indifference.\u201d", "author": "Elie Wiesel", "tags": ["activism", "apathy", "hate", "indifference", "inspirational", "love", "opposite", "philosophy"]}
{"text": "\u201cIt is not a lack of love, but a lack of friendship that makes unhappy marriages.\u201d", "author": "Friedrich Nietzsche", "tags": ["friendship", "lack-of-friendship", "lack-of-love", "love", "marriage", "unhappy-marriage"]}
{"text": "\u201cGood friends, good books, and a sleepy conscience: this is the ideal life.\u201d", "author": "Mark Twain", "tags": ["books", "contentment", "friends", "friendship", "life"]}
{"text": "\u201cLife is what happens to us while we are making other plans.\u201d", "author": "Allen Saunders", "tags": ["fate", "life", "misattributed-john-lennon", "planning", "plans"]}
```

> 可以看到新文件中没有最外层大括号

## 嵌套链接的访问

+ 如果我们的目标不是第一次的链接，则需要获取到目标链接后继续进行后续的访问

以下面的html代码为例，我们的目标是a标签的链接
```html
<ul class="pager">
    <li class="next">
        <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
    </li>
</ul>
```
以下代码，我们能获取到该链接（任一都可）
```python
response.css('li.next a::attr(href)').get()
response.css('li.next a').attrib['href']
```
结果为
```
'/page/2/'
```
### 应用到框架
> tutorial\spiders\quotes_spider.py
```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('small.author::text').get(),
                'tags': quote.css('div.tags a.tag::text').getall(),
            }

        next_page = response.css('li.next a::attr(href)').get()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)
```
快捷写法

```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('span small::text').get(),
                'tags': quote.css('div.tags a.tag::text').getall(),
            }

        next_page = response.css('li.next a::attr(href)').get()
        if next_page is not None:
            yield response.follow(next_page, callback=self.parse)
```
框架会一直找下一段链接，直到尽头
> 我们也能爬当前页面的所有链接(直到尽头)
```python 
for href in response.css('ul.pager a::attr(href)'):
    yield response.follow(href, callback=self.parse)
```
简写如下
```python 
for a in response.css('ul.pager a'):
    yield response.follow(a, callback=self.parse)
```
这是a标签特有的快捷方式

再次简写如下
```python 
anchors = response.css('ul.pager a')
yield from response.follow_all(anchors, callback=self.parse)
```
再再次简写如下
```python 
yield from response.follow_all(css='ul.pager a', callback=self.parse)
```
## 最后一部分，给框架传参
```python 
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        url = 'http://quotes.toscrape.com/'
        # 获取参数
        tag = getattr(self, 'tag', None)
        if tag is not None:
            # 拼接网址
            url = url + 'tag/' + tag
        yield scrapy.Request(url, self.parse)

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('small.author::text').get(),
            }
        # 自动翻页
        next_page = response.css('li.next a::attr(href)').get()
        if next_page is not None:
            yield response.follow(next_page, self.parse)

```
> 运行命令

```cmd
scrapy crawl quotes -O quotes-humor.json -a tag=humor
```

> 运行结果
```json
[
{"text": "\u201cThe person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.\u201d", "author": "Jane Austen"},
{"text": "\u201cA day without sunshine is like, you know, night.\u201d", "author": "Steve Martin"},
{"text": "\u201cAnyone who thinks sitting in church can make you a Christian must also think that sitting in a garage can make you a car.\u201d", "author": "Garrison Keillor"},
{"text": "\u201cBeauty is in the eye of the beholder and it may be necessary from time to time to give a stupid or misinformed beholder a black eye.\u201d", "author": "Jim Henson"},
{"text": "\u201cAll you need is love. But a little chocolate now and then doesn't hurt.\u201d", "author": "Charles M. Schulz"},
{"text": "\u201cRemember, we're madly in love, so it's all right to kiss me anytime you feel like it.\u201d", "author": "Suzanne Collins"},
{"text": "\u201cSome people never go crazy. What truly horrible lives they must lead.\u201d", "author": "Charles Bukowski"},
{"text": "\u201cThe trouble with having an open mind, of course, is that people will insist on coming along and trying to put things in it.\u201d", "author": "Terry Pratchett"},
{"text": "\u201cThink left and think right and think low and think high. Oh, the thinks you can think up if only you try!\u201d", "author": "Dr. Seuss"},
{"text": "\u201cThe reason I talk to myself is because I\u2019m the only one whose answers I accept.\u201d", "author": "George Carlin"},
{"text": "\u201cI am free of all prejudice. I hate everyone equally. \u201d", "author": "W.C. Fields"},
{"text": "\u201cA lady's imagination is very rapid; it jumps from admiration to love, from love to matrimony in a moment.\u201d", "author": "Jane Austen"}
]
```
由此可见参数可以通过`-a`来传递，`getattr`方法获取

## 完结撒花