# Day1
## 首先介绍本人学习爬虫时候的背景

1. 大四期间通过`bilibili`学习过`python`基础，但是已经过去两年左右<br>
2. 目前有`前端三件套`基础 (与爬虫没太大联系)<br>
3. 有`vue`基础，并且跟着`网易云`有阿里工作经验的老师完成了一次商城后台管理系统的开发(目前在不看视频的情况下二刷)<br><br>

#### **总的来说，在学习爬虫上本人也只是对网络请求有点了解而已，目前也只是抱着试着学的心态，希望能通过源码学习的形式一步一步掌握这门技术。至于学习动机，好像也没有什么    目的，可能就是为了以后求职能在简历上多写上一行吧（流下了没有技术的泪水~~~~）。**

## 观察源码结构
第一步：找到源码 [github源码](https://github.com/wistbean/learn_python3_spider/blob/master/dangdang_top_500.py)
```python
import requests
import re
import json


def request_dandan(url):
    try:
        response = requests.get(url)
        if response.status_code == 200:
            return response.text
    except requests.RequestException as e:
        print(e)
        return None


def parse_result(html):
    pattern = re.compile(
        '<li.*?list_num.*?(\d+)\.</div>.*?<img src="(.*?)".*?class="name".*?title="(.*?)">.*?class="star">.*?class="tuijian">(.*?)</span>.*?class="publisher_info">.*?target="_blank">(.*?)</a>.*?class="biaosheng">.*?<span>(.*?)</span></div>.*?<p><span class="price_n">(.*?)</span>.*?</li>', re.S)
    items = re.findall(pattern, html)

    for item in items:
        yield {
            'range': item[0],
            'image': item[1],
            'title': item[2],
            'recommend': item[3],
            'author': item[4],
            'times': item[5],
            'price': item[6]
        }


def write_item_to_file(item):
    print('开始写入数据 ====> ' + str(item))
    with open('book.txt', 'a', encoding='UTF-8') as f:
        f.write(json.dumps(item, ensure_ascii=False) + '\n')


def main(page):
    url = 'http://bang.dangdang.com/books/fivestars/01.00.00.00.00.00-recent30-0-0-1-' + str(page)
    html = request_dandan(url)
    items = parse_result(html)  # 解析过滤我们想要的信息
    for item in items:
        write_item_to_file(item)


if __name__ == "__main__":
    for i in range(1, 26):
        main(i)
```
>由于是第一天学习，我就好好和大家一起分析一下这个python脚本的结构
#### 第一个重要的结构，脚本函数入口
```python
if __name__ == "__main__":
    for i in range(1, 26):
        main(i)
```
#### 这段代码什么时候执行呢？自然是脚本被运行的时候执行，而运行脚本的一个常用方法就是在`cmd`里面输入如下代码（相信大家都知道）
```cmd
python 函数名.py
```
>接下来看看其他函数（根据main里面的执行顺序依次如下）
1. main(page)
2. request_dandan(url)
3. parse_result(html)
4. write_item_to_file(item)


>**主函数main(page)**<br>

|输入参数|类型|解释|
| --- | --- | --- |
|page|数字|爬取第几页的内容|

>**request_dandan(url)**<br>

功能：请求并返回页面内容
|输入参数|类型|解释|
|-|-|-|
|url|字符串|爬取对象的链接|

>**parse_result(html)**<br>

功能：根据页面内容解析出想要的数据
|输入参数|类型|解释|
|-|-|-|
|html|字符串|页面内容|

>**write_item_to_file(item)**<br>

功能：将解析出来的内容整理成文件
|输入参数|类型|解释|
|-|-|-|
|item|字典|以键值的形式存储的数据（详情见源码中parse_result的返回值）|

#### **总结来说，基础的爬虫就是三个步骤，`请求数据`，`解析`，`整理成文件`，如今本人只想了解两个步骤进行研究，最后一步吗，想写成文件怎么写，只要想总有办法的啦。**

## 逐步分析源码
>可以跟着我的步骤来，一步一步拆分源码，了解源码的原理

+ 导入所需库 (没什么好说的，`import`就完事)
```python
import requests
import re
import json
```
+ 设置网页链接 （这里用到了字符串拼接）
```python
url = 'http://bang.dangdang.com/books/fivestars/01.00.00.00.00.00-recent30-0-0-1-' + str(1)
url
```
output:'http://bang.dangdang.com/books/fivestars/01.00.00.00.00.00-recent30-0-0-1-1'
#### 设置网页链接 （这里用到了字符串拼接）

+ 请求网页数据
```python
try:
    response = requests.get(url)
    if response.status_code == 200:
        html = response.text
except requests.RequestException as e:
    print(e)
type(html)
```
output:str<br>

#### 可以看到，这里用get来请求网页，如果标志为`200`则表示请求成功，而`except`会在请求出错时抛出异常，存储为`e`，并且打印出来。
到这一步页面内容就算请求到了，是不是很快~~~

## 本节学习最后一步，解析页面内容
>虽然说我们已经得到了页面数据，我们目标也就在`html`里面，但是手动去一大串数据里面手动去找目标并不是程序员的作风，所以我们需要将它提取（解析）出来。

```python
pattern = re.compile('<li.*?list_num.*?(\d+)\.</div>.*?<img src="(.*?)".*?class="name".*?title="(.*?)">.*?class="star">.*?class="tuijian">(.*?)</span>.*?class="publisher_info">.*?target="_blank">(.*?)</a>.*?class="biaosheng">.*?<span>(.*?)</span></div>.*?<p><span class="price_n">(.*?)</span>.*?</li>',
                     re.S)
items = re.findall(pattern,html)
items[0]
```
output:('1',<br>
 'http://img3m1.ddimg.cn/86/2/25546541-1_l_13.jpg',<br>
 '空间简史(与《时间简史》《人类简史》《未来简史》并称“四大简史”)',<br>
 '100%推荐',<br>
 '托马斯・马卡卡罗',<br>
 '135650次',<br>
 '&yen;18.50' )<br>
 
> 这里我们就拿到了我们想要的内容，为了美观一点，我们不妨把它存为`字典`

```python
result = {
    'range': items[0][0],
    'image': items[0][1],
    'title': items[0][2],
    'recommend': items[0][3],
    'author': items[0][4],
    'times': items[0][5],
    'price': items[0][6]
}
result
```
output]:<br>
{'range': '1',<br>
 'image': 'http://img3m1.ddimg.cn/86/2/25546541-1_l_13.jpg',<br>
 'title': '空间简史(与《时间简史》《人类简史》《未来简史》并称“四大简史”)',<br>
 'recommend': '100%推荐',<br>
 'author': '托马斯・马卡卡罗',<br>
 'times': '135650次',<br>
 'price': '&yen;18.50'}<br>

给大伙整理一下
|参数| 内容|
|-|-|
|范围 <a href="#">range</a>| 1 |
|图片链接<a href="#">image</a>| http://img3m1.ddimg.cn/86/2/25546541-1_l_13.jpg|
|标题 <a href="#">title</a>|空间简史(与《时间简史》《人类简史》《未来简史》并称“四大简史”)|
|评价 <a href="#">recommend</a>| 100%推荐 |
|作者 <a href="#">author</a>| 托马斯・马卡卡罗 |
|次数 <a href="#">times</a>| 135650次 |
|价格 <a href="#">price</a>| ¥18.50 |

# 完结撒花

