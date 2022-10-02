# selenium
<center>

咱又又又来快速入门了！

（***学无止尽，永不放弃***）

理由吗，大概就是我太菜了！

本来想出个爬网页的，谁知道就连一两年前的爬虫代码也没法用了

（***反爬太牛了！***）

神特么翻页的a标签没有href就算了，连js都没有，也能正常翻页？

（***小爷在这给大佬跪了***）

于是我突发奇想，可能我们还需要一个库来帮我们模拟用户操作，

于是乎就有了今天的笔记【selenium入门】

</center>

参考[官网](https://www.selenium.dev/zh-cn/documentation/webdriver/getting_started/install_library/)

参考[网友](https://www.cnblogs.com/gaidy/p/12095708.html)

>### 1. 安装
```
$ pip install selenium
```
查看详情
```
$ pip show selenium

Name: selenium
Version: 4.5.0
Summary:
Home-page: https://www.selenium.dev
Author:
Author-email:
License: Apache 2.0
Location: c:\users\your_master\anaconda3\lib\site-packages
Requires: trio-websocket, trio, certifi, urllib3
Required-by:
```
>### 2. 查看chrome浏览器版本

访问[网址](chrome://version)(chrome://version)

> ### 3. [下载驱动](https://chromedriver.chromium.org/downloads)

> ### 4. 查找python位置
```cmd 
$ where python

C:\Users\your_master\anaconda3\python.exe
C:\Users\your_master\AppData\Local\Microsoft\WindowsApps\python.exe
```
> ### 5. 将解压的驱动放在python目录下（与python.exe同级目录下）

以下命令查看是否添加成功
```
$ chromedriver

Starting ChromeDriver 105.0.5195.52 (412c95e518836d8a7d97250d62b29c2ae6a26a85-refs/branch-heads/5195@{#853}) on port 9515
Only local connections are allowed.
Please see https://chromedriver.chromium.org/security-considerations for suggestions on keeping ChromeDriver safe.
ChromeDriver was started successfully.
```

> ### 6. 利用驱动打开我们的第一个页面

后两句代码建议一起执行，否则会出现页面一闪即逝的情况（本人用cmd和notebook执行时出现该情况）
```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get("http://www.baidu.com/")
```
> ### 7. 8个脚本基本部分

|序号|名称|代码|
|-|-|-|
|1|创建实例|driver = webdriver.Chrome()|
|2|执行操作|driver.get(url)|
|3|浏览器信息|title = driver.title|
|4|等待策略|driver.implicitly_wait(0.5)|
|5|查找元素|text_box = driver.find_element(by=By.NAME, value="my-text")<br>submit_button = driver.find_element(by=By.CSS_SELECTOR, value="button")|
|6|操作元素|text_box.send_keys("Selenium")<br>submit_button.click()|
|7|获取元素信息|value = message.text|
|8|结束会话|driver.quit()|

#### 创建实例 [官网参考代码]
```python
driver = webdriver.Chrome(service=ChromeService(executable_path=ChromeDriverManager().install()))
```
### 组合操作

该代码完成了一次提交操作
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service as ChromeService
from webdriver_manager.chrome import ChromeDriverManager


def test_eight_components():
  #创建实例
    driver = webdriver.Chrome(service=ChromeService(executable_path=ChromeDriverManager().install()))
    #访问网页
    driver.get("https://www.selenium.dev/selenium/web/web-form.html")
    #获取网页标题
    title = driver.title
    #断言：如果为否则结束
    assert title == "Web form"
    #等待0.5s
    driver.implicitly_wait(0.5)
    #找到名字为 "my-text"的元素
    text_box = driver.find_element(by=By.NAME, value="my-text")
    #利用css选择器，找到button标签
    submit_button = driver.find_element(by=By.CSS_SELECTOR, value="button")
    #输入"Selenium"
    text_box.send_keys("Selenium")
    # 点击按钮
    submit_button.click()
    #用ID找到元素
    message = driver.find_element(by=By.ID, value="message")
    #获取元素的值
    value = message.text
    # 断言：如果为否就结束
    assert value == "Received!"
    # 结束
    driver.quit()
```
>### 三种等待

|名字|解释|
|-|-|
|显式等待|冻结程序，直到条件满足（需要指定条件）|
|隐式等待|在一定时间内，轮询DOM（不需要指定条件）|
|流畅等待|可以设置频率，最大时间，以及要忽略的报错（同显式）|

> 示例代码

显式等待

```python
from selenium.webdriver.support.wait import WebDriverWait

driver.navigate("file:///race_condition.html")
el = WebDriverWait(driver, timeout=3).until(lambda d: d.find_element(By.TAG_NAME,"p"))
assert el.text == "Hello from JavaScript!"
```
隐式等待

```python
driver = Firefox()
driver.implicitly_wait(10)
driver.get("http://somedomain/url_that_delays_loading")
my_dynamic_element = driver.find_element(By.ID, "myDynamicElement")
```

流畅等待

```python
driver = Firefox()
driver.get("http://somedomain/url_that_delays_loading")
wait = WebDriverWait(driver, timeout=10, poll_frequency=1, ignored_exceptions=[ElementNotVisibleException, ElementNotSelectableException])
element = wait.until(EC.element_to_be_clickable((By.XPATH, "//div")))
```

> 源码解析（终于爬到东西！）
```python
import unittest
from selenium import webdriver
from bs4 import BeautifulSoup as bs

class douyu(unittest.TestCase):
    # 初始化方法，必须是setUp()
    def setUp(self):
        self.driver = webdriver.Firefox()
        self.num = 0
        self.count = 0

    # 测试方法必须有test字样开头
    def testDouyu(self):
        self.driver.get("https://www.douyu.com/directory/all")

        while True:
            soup = bs(self.driver.page_source, "lxml")
            # 房间名, 返回列表
            names = soup.find_all("h3", {"class" : "ellipsis"})
            # 观众人数, 返回列表
            numbers = soup.find_all("span", {"class" :"dy-num fr"})

            # zip(names, numbers) 将name和number这两个列表合并为一个元组 : [(1, 2), (3, 4)...]
            for name, number in zip(names, numbers):
                print u"观众人数: -" + number.get_text().strip() + u"-\t房间名: " + name.get_text().strip()
                self.num += 1
                #self.count += int(number.get_text().strip())

            # 如果在页面源码里找到"下一页"为隐藏的标签，就退出循环
            if self.driver.page_source.find("shark-pager-disable-next") != -1:
                    break

            # 一直点击下一页
            self.driver.find_element_by_class_name("shark-pager-next").click()

    # 测试结束执行的方法
    def tearDown(self):
        # 退出Firefox()浏览器
        print "当前网站直播人数" + str(self.num)
        print "当前网站观众人数" + str(self.count)
        self.driver.quit()

if __name__ == "__main__":
    # 启动测试模块
    unittest.main()
```

这里用到了unittest框架，执行顺序

unittest.main()->setUp(self)->testDouyu(self)->tearDown(self)

这里开始拆

> 第一步：导库

```python
from lib2to3.pgen2 import driver
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.by import By
from selenium.webdriver.common.action_chains import ActionChains

```
> 第一步：初始化驱动

```python
driver = webdriver.Chrome()
# 直播间数
num = 0
# 页面数
count = 0

```

> 第三步：隐式等待

为什么设置这么长？因为会弹广告，我选选择手动关闭
```python
driver.implicitly_wait(20)

```

> 第四步：访问，创建动作链

```python
driver.get("https://www.douyu.com/directory/all")
actions = ActionChains(driver)
```
> 最后一段主体，不好拆：

基本意思看注释
```python
while True:
  print("---------------第一页--------------")
  count+=1
  soup = bs(driver.page_source,"lxml")
  # 房间名,已经改名，需要修改，我这算是改好的
  names = soup.find_all("h3",{"class":"DyListCover-intro"})
  # 观众人数
  numbers = soup.find_all("span",{"class":"DyListCover-hot"})
  # 打印观看人数
  for name,number in zip(names,numbers):
    print(u"--观看人数--：" ,number.text ,u"--直播间名字--：" + name.text)
    num+=1
  # 与源码不同，我打算找到最后一个标签的值作为结束点
  last_page = driver.find_element(by=By.CSS_SELECTOR,value="ul.ListPagination li:nth-last-child(2) a")
  if str(count) == last_page.text:
    break
  # 找到下一页标签
  next_button = driver.find_element(by=By.CSS_SELECTOR,value=".dy-Pagination-next")
  actions.click(next_button)
  actions.perform()
print(u"--当前直播间数:" + str(num))
print(u"--页面总数:" + str(count))
```

结果
```
---------------第一页--------------
--观看人数--： 1.5万 --直播间名字--：舟阿i：爆炸的艺术~
--观看人数--： 9.7万 --直播间名字--：给我一次证明盖伦的机会
--观看人数--： 47.5万 --直播间名字--：旭旭宝宝斗地主专场-9月16日更新
--观看人数--： 105.7万 --直播间名字--：【重播】PMRC区域对抗赛第四日
--观看人数--： 10.5万 --直播间名字--：日搬30角色-目标200亿
--观看人数--： 11.4万 --直播间名字--：4837387正能量纪录片
--观看人数--： 15.1万 --直播间名字--：打排位秒上号0.0
--观看人数--： 11.5万 --直播间名字--：本厅由“奈何”独冠
--观看人数--： 11.5万 --直播间名字--：夜夜声色 卿本迷人

.....


```

##### 不容易，学了这么多终于爬到第一个网页了，已有接下来的计划，
1. 熟悉web驱动的函数
2. 熟悉css选择
3. 掌握框架和web驱动的混合写法

## 辛苦辛苦，完结撒花（汗）


