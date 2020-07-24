# Selenium 基本操作

随便写了一些，有不对的欢迎指正。

## Hello World

环境搭配好之后，写一个简单的程序测试一下

```python
from selenium import webdriver

browser = webdriver.Chrome()         # 打开Chrome驱动
browser.get('http://www.baidu.com')     # 打开百度网页
browser.find_element_by_id('kw').send_keys('华为')        # 通过id定位百度的输入框元素kw，并输入查找的内容
browser.find_element_by_id('su').click()                  # 通过id定位百度的查找按钮，并模拟点击

# browser.quit()            # 此处为了保留网页，不退出浏览器，其他时候，记得用完退出，避免进程过多
```



## Action

- 打开网页

打开页面之后，由于SPES的免密登陆或页面加载，建议采用等待元素的方法，避免网页没加载出来就去定位元素造成报错。等待方法在下面会说。

```python
from selenium import webdriver

browser = webdriver.Chrome()
url = 'http://www.baidu.com'			# 访问的网址
browser.get(url)					   # 打开页面
```



- 最大化页面

浏览器默认打开不是最大化，而有些web系统在小窗口的情况下，元素的位置和加载情况有变化，所以建议直接默认最大化打开

```python
from selenium import webdriver

browser = webdriver.Chrome()
browser.maximize_window()
url = 'http://www.baidu.com'			# 访问的网址
browser.get(url)					   # 打开页面
```



- 刷新页面

```python
from selenium import webdriver

browser = webdriver.Chrome()
browser.refresh()
```



- Frame切换

不知道为啥，华为的有些页面很多frame，经常回导致，明明xpath对的，运行的时候却定位异常。这时候，一般都考虑为没有切换到正确的frame上，导致定位出错，切换方法如下：

```python

```



- 定位元素

核心操作，只要元素定位好了，一起操作都好说，需要灵活运用浏览器的F12开发这么模式。

```python
# #######单个元素定位方法，举一些常见例子，推荐使用xpath定位，xpath因为是正则表达式，最灵活#######
# 通过标签的id属性进行定位，需要注意有些id是动态变化的，该该方法只能定位静态的ID
browser.find_element_by_id("table id")

# 通过正则表达式进行定位，可通过浏览器F12开发者模式对正则表达式进行测试
browser.find_element_by_xpath("//div[@ng-bind='chartModel.title']")


# #######多个元素定位#######
# 加了s，返回的是符合条件的对象集合，通过for循环进行遍历
elems = browser.find_elements_by_id("table ID")
for i in elems:
    i.click()		# 所有该类元素进行点击操作的举例
```



- 等待元素

网页加载是需要时间的，有些元素如果在没加载出来之前就进行定位，系统会报`NoSuchElementException`等异常

等待分为三类：

1. 强制等待：强制等待多少时间
2. 隐式等待：只需要设置一次，表示查找元素未出现时等待30s，再查没有则抛出异常
3. 显式等待：等待某个元素的加载完成、可点击、内容出现等情况，再去获取元素

```python
from time import sleep
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as ec
from selenium.webdriver.common.by import By

# 强制等待
sleep(5)		# 程序等待5s

# 隐式等待
browser.implicitly_wait(30)		# 隐式等待30s

# 显示等待
timeout = 10		# 超时时间
checktime = 1		# 每1s去去确认一次
xpath = '//div[@id="your id"]'			# xpath定位路径
# 点击等待
WebDriverWait(browser, timeout, checktime).until(ec.element_to_be_clickable((By.XPATH, xpath)))
# 加载等待
WebDriverWait(browser, timeout, checktime).until(ec.presence_of_element_located((By.XPATH, xpath)))
# 多个元素加载等待
WebDriverWait(browser, timeout, checktime).until(ec.presence_of_all_elements_located((By.XPATH, xpath)))
```



- 点击元素

```python
# 两种方式

# 直接点击
browser.find_element_by_xpath("//div[@ng-bind='chartModel.title']").click

# 获取元素，在需要的时候点击（如果页面刷新了等情况，点击的元素可能就不存在了，两个操作最好不要离太远）
elem = browser.find_element_by_xpath("//div[@ng-bind='chartModel.title']")
elem.click()
```



- 输入框输入内容

```python
browser.find_element_by_xpath("//div[@ng-bind='chartModel.title']").send_keys('your content')
```



- 元素截图

```python
browser.find_element_by_xpath("//div[@class='max-chart-area']").screenshot('图片保存路径.png')
```



- 获取元素文本

```python
browser.find_element_by_xpath("//div[@ng-bind='chartModel.title']").text()
```



- js操作

js操作适用于有些情况下无法点击或者输入框需要读取value值的情况

```python
elem = browser.find_element_by_xpath("//div[@ng-bind='chartModel.title']")

# 点击
browser.execute_script("arguments[0].click();", elem)
# 输入
val = 'your input'
browser.execute_script("arguments[0].value='"+val+"';", elem)
```



- 无界面登录

该方法并非对一切页面适用，有些页面不显示浏览器无法加载元素，请谨慎尝试

```python
from selenium.webdriver.chrome.options import Options

chrome_options = Options()
chrome_options.add_argument('--headless')
browser = webdriver.Chrome(options=chrome_options)
```



- 退出网页

操作完王爷后，记得退出浏览器，否则进程不会主动退出，长久会导致内存占用大量`chromdriver`程序

```python
browser.quit()
```



## References

[Selenium元素定位总结](https://www.cnblogs.com/malus-fish/p/12441849.html)

[ElementClickInterceptedException报错的解决](https://blog.csdn.net/WanYu_Lss/article/details/84137519)

[Selenium EC 与 Wait](https://blog.csdn.net/weixin_41733260/article/details/88982268)

[Selenium父子、兄弟、相邻节点的定位方式](https://blog.csdn.net/huilan_same/article/details/52541680)

[Selenium处理反爬虫点击的方式](https://blog.csdn.net/lly1122334/article/details/103504169)

