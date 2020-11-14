

# 6 代理地址
    'http://cn-proxy.com',
    'http://www.xicidaili.com',
    'http://www.kuaidaili.com/free',
    'http://www.proxylists.net/?HTTP',
    # www.youdaili.net的地址随着日期不断更新
    'http://www.youdaili.net/Daili/http/4565.html',
    'http://www.youdaili.net/Daili/http/4562.html',
    'http://www.kuaidaili.com',
    'http://proxy.mimvp.com'
其中 http://cn-proxy.com 已经挂掉不能访问了；
 其中 http://www.proxylists.net/?HTTP 和 www.youdaili.net 的代理是文本格式，能够直接按照给出的正则表达式匹配。
 剩余的 www.xicidaili.com 、http://www.kuaidaili.com/free 、http://www.kuaidaili.com 给出代理的格式都是表格形式， IP 和端口并不在一起，使用提供的正则匹配不到。一种解决方案是用 bs 匹配 IP 后再获取兄弟节点的端口号：
```
bsObj = BeautifulSoup(html, 'html.parser')
ipList = bsObj.findAll('td', text=re.compile('[0-9]+(?:\.[0-9]+){3}'))
for each in ipList:
    ip = each.get_text()
    port = each.next_sibling.next_sibling.get_text() #因为换行的原因要取两次兄弟节点
    proxy = ip + ':' +port
```
最后一个 http://proxy.mimvp.com 就更扯了…端口一列直接是给出的图片…

# 7 JS代码String.fromCharCode（）
String.fromCharCode()方法用于把一个或多个 Unicode 值转换为（大写）字符串，并返回该字符串。
```
<script type="text/javascript">
    String.fromCharCode(65,66,67)   #  下例返回字符串 "ABC"
</script>
```
```
car1 = {"type":'Mazda', "model":5, "color":'white'}
attr = execjs.compile("""
    function car_type(x) {var temp = x; return temp.type;} """)
print(attr.call("car_type", car1))   # 返回:Mazda
```
```
from selenium import webdriver
url = "http://www.taobao.com/"
browser = webdriver.PhantomJS()
browser.get(url)
input = browser.find_element_by_xpath("//input[@id='q']")
bnt = browser.find_element_by_xpath("//button[@class='btn-search']")
input.send_keys("watch")
bnt.submit()
```
```
my_js_function = 'function my_function(a){var b=a.split("$");var c="";for(i=0;i<=b.length-1;i++){if(!(i%2)){c+=b[i]}}var b=c.split("");c="";for(i=0;i<=b.length-1;i++){if(i%2){c+=b[i]}}return c}'
compiled_function = ExecJS.compile(my_js_function)
compiled_function.eval('my_function("heyhey")')   #  => ehy
```

# 8 获取代理IP， 快代理， PyExecJs vs Selenium  补充：还有一个库 [Js2Py](https://github.com/PiotrDabkowski/Js2Py)
``` 20170329 
file = open('tmp.html', 'r').read()
ret1 = re.findall(r'gv\((.*?)\)', file)  # ret1的结果: ['193', 'RM']
ret2 = 'var  ' + re.findall(r'var (.*?) </script>', file, re.DOTALL)[0].replace(ret1[1], ret1[0])[:25] + ' return po;'  # ret2的结果：拼接js代码，上图中的8行到25行，加上最后的return语句返回结果
ret3 = execjs.exec_(ret2)  # "document.cookie='_ydclearance=1ed2225a7c109a04e94ebde3-c605-43f4-a5ff-8293304327ce-1490766671; expires=Wed, 29-Mar-17 05:51:11 GMT; domain=.kuaidaili.com; path=/'; window.document.location=document.URL"

ret3.split(';')[0].split('=')  # ['document.cookie', "'_ydclearance", '1ed2225a7c109a04e94ebde3-c605-43f4-a5ff-8293304327ce-1490766671']
```
![image](https://github.com/Django-27/workspace/blob/master/pic/selenium2.png)
#### 补充，js代码的函数名，参数名，给定的参数都是变得，上面指定gv还是有问题的
```
ret1 = re.findall(r'function (.*?)\(', file)[0]  # 取得函数名
Out[148]: 'jq'
ret1 = re.findall(r'%s\((.*?)\)'%ret1, file)  # 取得参数名，参数值；正则中包含变量，使用格式化字符串相关
Out[150]: ['35', 'RD']

ret2 = 'var  ' + re.findall(r'var (.*?) </script>', file, re.DOTALL)[0][:-25].replace(ret1[1], ret1[0]) + ' return po;'
                
ret3 = execjs.exec_(ret2)
k = ret3.split(';')[0].split("'")[1].split('=')[0]
v = ret3.split(';')[0].split("'")[1].split('=')[1]
self.session.cookies.update({k: v})  # {'_ydclearance': '0ac1fa903ebbfd9ef7b1bd75-7e1f-4461-921c-c4b2afa95a10-1490785013'}
```
#### 最后的目地就是更新了一下cookies，之后的请求就没有问题了。（注意在js中补充的return是一个要点）代码在github[地址](https://github.com/Django-27/my_spider/blob/master/proxy_kuaidaili.py)
### 相关js代码：
```
<html>

<body>
    <script language="javascript">
        window.onload = setTimeout("kx(222)", 200);

        function kx(SE) {
            var qo, mo = "",
                no = "",
                oo = [0x65, 0xc6, 0x1e, 0xa6, 0x6d, 0x4d, 0x17, 0xdf, 0x06, 0xf0, 0xcb, 0xe9, 0x72, 0x7c, 0x82, 0x22, 0xab, 0xf4, 0xb9, 0x79, 0x61, 0x62, 0x6e, 0xb5, 0x9e, 0x38, 0xc0, 0xa6, 0x6f, 0xb7, 0xce, 0x75, 0xbd, 0xe7, 0x70, 0x3b, 0x04, 0xc3, 0xa2, 0xc1, 0x97, 0xdf, 0x9e, 0xc7, 0x50, 0x56, 0x7f, 0x9e, 0x9d, 0x86, 0x78, 0xd6, 0xd5, 0x78, 0x37, 0xe8, 0xcd, 0x70, 0xcf, 0xb8, 0x00, 0x40, 0x7f, 0x28, 0xc6, 0x38, 0x57, 0x40, 0xfd, 0xe6, 0x61, 0xa4, 0xc3, 0x4c, 0x0a, 0x1f, 0xfd, 0x00, 0xc5, 0x88, 0x98, 0x56, 0xb5, 0x18, 0x56, 0x7d, 0xff, 0xbd, 0xc0, 0x7f, 0xe3, 0xa2, 0x6b, 0xc9, 0x29, 0x45, 0x2e, 0x90, 0xef, 0x4f, 0x89, 0x72, 0x51, 0x13, 0x51, 0x0e, 0xf0, 0x30, 0x8e, 0xed, 0xa9, 0x0c, 0xaa, 0xe9, 0x49, 0x00, 0x60, 0xdf, 0x40, 0x05, 0x7d, 0xe8, 0x4f, 0x98, 0x3f, 0x2f, 0xf7, 0x7e, 0x3e, 0x49, 0x22, 0xea, 0xd3, 0xb1, 0x12, 0x2a, 0xcc, 0x0c, 0xc9, 0x97, 0x8f, 0xd7, 0x7e, 0x40, 0x7e, 0x4d, 0x50, 0xb0, 0xee, 0xad, 0x50, 0xef, 0xae, 0xcd, 0x6d, 0x60, 0xfe, 0x5e, 0xbe, 0xcb, 0xbf, 0x89, 0x78, 0xf7, 0x58, 0x0c, 0xf4, 0xfa, 0xc4, 0x09, 0xe1, 0x2b, 0x51, 0x11, 0x33, 0xb8, 0x3e, 0x05, 0x4d, 0x96, 0x92, 0x7b, 0xc3, 0x09, 0xf2, 0x76, 0xbf, 0xdd, 0x66, 0x70, 0x05, 0xce, 0x4e, 0xae, 0x15, 0x52, 0x9a, 0x81, 0xea, 0xaa, 0xf3, 0xf1, 0xf2, 0x72, 0xd2, 0x15, 0x20, 0x65, 0x8f, 0x78, 0x39, 0x43, 0x4a, 0x6c, 0x51, 0xc6, 0xd0, 0x55, 0x20, 0xe5, 0xd3, 0x9c, 0xc2, 0xa9, 0xcb, 0xe0, 0xc6, 0xd0, 0x55, 0x9d, 0x51, 0x3c, 0x81, 0x8b, 0xb1, 0xec, 0xac, 0x91, 0x9b, 0x24, 0x84, 0x4b, 0x11, 0xd9, 0x04, 0xbe, 0xa5, 0xc7, 0x92, 0x3d, 0x38, 0xcd, 0x3b];
            qo = "qo=251; do{oo[qo]=(-oo[qo])&0xff; oo[qo]=(((oo[qo]>>2)|((oo[qo]<<6)&0xff))-58)&0xff;} while(--qo>=2);";
            eval(qo);
            qo = 250;
            do {
                oo[qo] = (oo[qo] - oo[qo - 1]) & 0xff;
            } while (--qo >= 3);
            qo = 1;
            for (;;) {
                if (qo > 250) break;
                oo[qo] = ((((((oo[qo] + 118) & 0xff) + 153) & 0xff) << 5) & 0xff) | (((((oo[qo] + 118) & 0xff) + 153) & 0xff) >> 3);
                qo++;
            }
            po = "";
            for (qo = 1; qo < oo.length - 1; qo++)
                if (qo % 5) po += String.fromCharCode(oo[qo] ^ SE);
            eval("qo=eval;qo(po);");
        }
    </script>
</body>

</html>
```
### 采用Selenium解决
- 由于这个是html中包含js代码，也不能一起执行html，不能直接实现script中间的，都报错
- 最后，手动指定 参数名和参数取值，并且去掉最后的eval语句，才可以拿到结果
- 这里Selenium没能够跳转页面，也是卡在了这里，多次请求还是这个页面；这种情况下requests比Selenium更优吧（消耗更小）。
![image](https://github.com/Django-27/workspace/blob/master/pic/selenium.png)

# 9  获取代理IP， 快代理， Selenium with PhantomJS

# 10 requests离线pdf或其他大文件
#### 重点是设置：stream=True
```python
import requests

url = 'http://disclosure.szse.cn/finalpage/2016-04-25/1202231488.PDF'
r = requests.get(url, stream=True)
chunk_size = 2000  # bytes
if r.status_code == 200:
    with open('/1202231488.pdf', 'wb') as f:
        for chunk in r.iter_content(chunk_size):
            f.write(chunk)
    print('finished')
```

# 11 Firfox's about:config  in chrom
about:about or chrome://about or about:flags
Chromium是Chrom的开源版，也可以理解Chromium是Chrome的测试版（收集信息（国家、安装次数）），稳定之后会进行Chome升级，哈哈。
Chromium提供命令行的控制，[详见](http://peter.sh/experiments/chromium-command-line-switches/)。
#### FireFox的这个东西，可能也是很多人选择的原因吧；例如：结合Selenium的profile对浏览器控制。
C:\WINDOWS\system32\drivers\etc\hosts 
```
from selenium import webdriver

firefox_profile = webdriver.FirefoxProfile()                         # 初始化化一个firefox_profile实例
firefox_profile.set_preference('permissions.default.image', 2)       # 不下载和加载图片
firefox_profile.set_preference('permissions.default.stylesheet', 2)  # 禁用样式表文件
firefox_profile.set_preference('javascript.enabled', False)          # 禁止Javascript的执行
firefox_profile.update_preferences()

firfox = webdriver.Firefox(firefox_profile)
firfox.get('http://www.baidu.com')
page = firfox.page_source  # 获取网页渲染后的源代码
firfox.quit()  # 检查发现，退出后本机about：config文件中对应参数没有被改变。
```
#### WebDriverException: Message: 'geckodriver' executable needs to be in PATH.
- selenium 3.x开始，webdriver/搜索firefox/webdriver.py的__init__中，executable_path="geckodriver"；而2.x是executable_path="wires"
- firefox 47以上版本，需要下载第三方driver，即geckodriver；在[link](https://github.com/mozilla/geckodriver)下载到任意电脑任意目录，解压后将该路径加入到PC的path（针对windows）即可。 
```
from selenium import webdriver

browser = webdriver.PhantomJS(executable_path='C:/Users/qiyuan/Desktop/phantomjs.exe')  # 无界面浏览器
browser.get('http://item.jd.com/1312640.html')
page = browser.page_source  # open('tmp_jd.html', 'w', encoding='utf-8').write(browser.page_source)

# browser.find_element_by_tag_name("div").text
# selenium在webdriver的DOM中使用选择器来查找元素，名字直接了当，
# by对象可使用的选择策略有：id,class_name,css_selector,link_text,name,tag_name,tag_name,xpath等等

```

# 12 Selenium with Python 控制浏览器行为
```python 
from selenium import webdriver          # 成功打开本机FireFox浏览器，并进行搜索操作。
from selenium.webdriver.common.keys import Keys

driver = webdriver.Firefox()
driver.get("http://www.python.org")

elem = driver.find_element_by_name("q")  # 找到输入框
elem.send_keys(" pycon")
elem.send_keys(Keys.RETURN)

print(driver.page_source)
driver.quit()
```
#### 通过 Selenium 与页面交互
```
<input type="text" name="passwd" id="passwd-id" />

element = driver.find_element_by_id("passwd-id")
element = driver.find_element_by_name("passwd")
element = driver.find_elements_by_tag_name("input")
element = driver.find_element_by_xpath("//input[@id='passwd-id']") # 多个匹配，只返回第一个；如果没有抛出 NoSuchElementException 异常

element.send_keys("some text")                  # 向文本输入内容
element.send_keys("and some", Keys.ARROW_DOWN)  # 可以对任何获取到到元素使用 send_keys 方法，模拟摸个按键 
element.clear()  # 清除文本

Python

element = driver.find_element_by_xpath("//select[@name='name']")
all_options = element.find_elements_by_tag_name("option")
for option in all_options:
    print("Value is: %s" % option.get_attribute("value"))
    option.click()
```
### 完成表单填充
```
element = driver.find_element_by_xpath("//select[@name='name']")  # 方法一
all_options = element.find_elements_by_tag_name("option")
for option in all_options:
    print("Value is: %s" % option.get_attribute("value"))
    option.click()
    
from selenium.webdriver.support.ui import Select
select = Select(driver.find_element_by_name('name'))  # 方法二
select.select_by_index(index)
select.select_by_visible_text("text")
select.select_by_value(value)


select = Select(driver.find_element_by_id('id'))  # 全部取消
select.deselect_all()

select = Select(driver.find_element_by_xpath("xpath"))  # 获取全部已选项
all_selected_options = select.all_selected_options

options = select.options  # 获取全部可选项

driver.find_element_by_id("submit").click()  # 提交表单
element.submit()  # 单独提交某个元素，元素并没有被表单所包围，那么程序会抛出 NoSuchElementException 的异常
```
### 元素拖拽
```
element = driver.find_element_by_name("source") # 首先指定被拖动的元素和拖动的目标元素
target = driver.find_element_by_name("target")
 
from selenium.webdriver import ActionChains # 利用ActonChains类实现，实现元素source到target的操作
action_chains = ActionChains(driver)
action_chains.drag_and_drop(element, target).perform()
```
### 页面切换
```
driver.switch_to_window("windowName")

for handle in driver.window_handles:
    driver.switch_to_window(handle)

driver.switch_to_frame("frameName.0.child")  # 这样焦点会切换到一个 name 为 child 的 frame 上
```
### 页面等待，隐式等待和显示等待
```
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
 
driver = webdriver.Chrome()                  # 显式等待指定某个条件，设置最长等待时间,如果在这个时间还没有找到元素，那么便会抛出异常 ElementNotVisibleException
driver.get("http://somedomain/url_that_delays_loading")
try:
    element = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.ID, "myDynamicElement"))
    )
finally:
    driver.quit()
    
wait = WebDriverWait(driver, 10)              # 使用内置的等待条件
element = wait.until(EC.element_to_be_clickable((By.ID,'someid')))

driver = webdriver.Chrome()                   # 隐式等待比较简单，就是简单地设置一个等待时间，单位为秒
driver.implicitly_wait(10) # seconds
driver.get("http://somedomain/url_that_delays_loading")
myDynamicElement = driver.find_element_by_id("myDynamicElement")
```
### 其他
```
alert = driver.switch_to_alert()  # 弹出处理，获取弹窗对象
driver.forward()   driver.back()  # 页面的前进和后退
cookie = {‘name’ : ‘foo’, ‘value’ : ‘bar’}
driver.add_cookie(cookie)         # cookies处理
driver.get_cookies()              # 获得页面的cookies
driver.find_element(By.XPATH, '//button[text()="Some text"]')    # 通过By类确定方式，有ID、XPATH、LINK_TEXT、NAME等 
driver.find_elements(By.XPATH, '//button')
```
#### 执行脚本 driver.execute_script("document.getElementsByClassName('comments dno')[0].click()")

# 13 Selenium 与 requests.Session 对比(http://news.sohu.com/scroll/) 
![image](http://note.youdao.com/yws/public/resource/9de657f6beaffa68f2619e3f1fa38098/xmlnote/9424BFE29D284A92833F90CE40D57940/2627)
#### 总结：
- 有网友说是Selenium取到了js变量的值，最后得到的是一个 字典；但是如果用常规的requests方式理解，为进行了两次get，最后得到的是一个str，并且尝试了以上的办法也没有实现str到dict的转换。
- 这个例子中就显示了Selenium的优势，得到字典对后去的键值操作非常方便，但是一个str则用处不大；转换过程遇到属性不是双引号包围，尝试category属性两边加上双引号，但是后面的1,2，3等数字又没办法全部加上双引号；对于单引号需要转双引号的问题这里没有。

# 14 http.cookiejar有关cookie的使用
```python
import requests
from http import cookiejar

session = requests.Session()
session.cookies = cookiejar.LWPCookieJar(filename='cookies.txt')
try:
    session.cookies.load(ignore_discard=True)
except LoadError:
    print("load cookies failed")
···
session.cookies.save()
```
# 14 反爬虫[ctrip](https://v.qq.com/x/page/j0308hykvot.html)
- 更改链接地址(price -> prica)
- 更改key， 更改动态key，十分复杂的key(js生成，调试复杂)
- 浏览器检测（检测ie的bug，问ie1+1=3；检测ff的严格性；检测chrome的强大特性
- 抓到了怎么办：给20%假数据；技术压制：前期不管后期直接击杀；放水
```
- 轻松切换目录 cd - , 内部使用$OLDPWD存储前一个cd操作的目录 
- tmux is more powerful shell
```

# 16 Scrapy 学习-1

- scrapy startproject tutorial                 # 管理员+虚拟环境
- scrapy genspider quotes quotes.toscrape.com  # 
- scrapy crawl quotes                          # 运行
```
import scrapy
class QuotesSpider(scrapy.Spider):
    name = 'quotes'

    def start_requests(self):  # 方法一    
        urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)
    
    start_urls = [  # 方法二
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]

    def parse(self, response):
        page = response.url.split('/')[-2]
        filename = 'quotes-%s.html' % page
        with open(filename, 'wb') as f:
            f.write(response.body)
        self.log('Saved file %s' % filename)
```
#### 一种方式是定义start_requests方法，由URLs产生scrapy.Request对象；也可以直接通过start_urls类属性，一个关于的urls的列表，默认的实现了start_requests()的功能初始化爬虫的requests。
#### parse()方法处理urls的每次请求，不必明确告知，因为parse()是Scrapy的默认回调函数
- scrapy shell "http://quotes.toscrape.com/page/1/"  # 通过shell提取数据，但当urls包含&符号则不可以， windows中必须使用双引号， 其它时候使用单引号包围
```
response.css('title').extract()  >>> ['<title>Quotes to Scrape</title>']
response.css('title::text').extract()  >>> ['Quotes to Scrape'], ::text表示<title>元素内部的文本元素， .extract()表示提取的是一个SelectorList实例，当然也有.extract_first(), [0].extract(), 使用.extract_first()的好处是当没有找到时返回None，而避免 了IndexError的异常
response.css("div.quote") or quote = response.css("div.quote")[0]

response.css('title').re(r'Quotes.*')  >>> ['Quotes to Scrape'], 也是以使用正则
response.css('title').re(r'Q\w+')  >>> ['Quotes']
response.css('title').re(r'(\w+) to (\w+)') >>> ['Quotes', 'Scrape']

view(response)  # 将打开一个浏览器，可以通过开发者选项提取

response.xpath('//title')  >>> [<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>], 通过xpath指定， css选择器也是转换为了xpath
response.xpath('//title/text()').extract_first()  >>> 'Quotes to Scrape'

tags = quote.css("div.tags a.tag::text").extract()
tags >>>  ['change', 'deep-thoughts', 'thinking', 'world']
response.css('li.next a::attr(href)').extract_first()  >>> '/page/2/'
```
#### 返回数据，提取逻辑，yield返回包含数据项的字典
```
    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }
            
        next_page = response.css('li.next a::attr(href)').extract_first()  # 加入下一页
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)
```
#### 指定存储文件: scrapy crawl quotes -o quotes.json
```
scrapy crawl quotes -o quotes-humor.json -a tag=humor # 多了-a参数tag的传递， 将会传递到爬虫的__init__方法中，并且变为爬虫的一个默认属性， 通过self.tag取得
    def start_requests(self):
        url = 'http://quotes.toscrape.com/'
        tag = getattr(self, 'tag', None)
        if tag is not None:
            url = url + 'tag/' + tag  # 将只访问http://quotes.toscrape.com/tag/humor
        yield scrapy.Request(url, self.parse)
        
```

159000&sell_run=0&searchapi_version=eb_split'
#### 下面的例子，由主页开始，找到author页，并之后每一个又调用parse_author回调，分页链接使用的还是原来的parse；默认scrapy通过setting中的DUPEFILTER_CLASS对URLs实现去重
#### 不用extrract()，那么Select对象，还能继续css或xpath；否则是列表，而extract_first()之后就是字符串
```
class AuthorSpider(scrapy.Spider):
    name = 'author'

    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        # follow links to author pages
        for href in response.css('.author + a::attr(href)').extract():
            yield scrapy.Request(response.urljoin(href),
                                 callback=self.parse_author)

        # follow pagination links
        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)

    def parse_author(self, response):
        def extract_with_css(query):
            return response.css(query).extract_first().strip()

        yield {
            'name': extract_with_css('h3.author-title::text'),
            'birthdate': extract_with_css('.author-born-date::text'),
            'bio': extract_with_css('.author-description::text'),
        }
```

# 17 Scrapy 学习-2
```
- 全局命令：startproject, genspider, settings, runspider, shell, fetch, view, version
- project-only命令: crawl, check, list, edit, parse, bench
- scrapy startproject <project_name> [project_dir]  #  如果没有指定project_idr，那么将与myproject相同
- scrapy genspider [-t template] <spider_name> <domain>  # <domain>用于产生爬虫的allowed_domains和start_urls属性,scrapy genspider -l用于查看也用的模板，有basic,crawl,csvfeed,xmlfeed;也可以直接建立爬虫源文件也是一样的
- scrapy crawl <spider_name>        # 启动开始一个爬虫
- scrapy check [-l] <spider>        # 进行宏观检查
- scrapy list                       # 列出当前工程下的所有爬虫
- scrapy edit <spider>              # 提供了设置EDITOR得方式，更好地可以在IDE中完成
- scrapy fetch <url>                # 使用scrapy下载和将内容写入输出，可以观察scrapy对指定url的动作，例如查看us是否更改；支持增加其他的参数:--spider=spider_name指定爬虫, --headers打印，--no-redirect，--nolog
- scrapy view <url>                 # 在浏览器中打开指定的url检查，有可能爬虫看到的和用户不同，可以指定--spider=spider_name, --no-redirect
- scrapy shell [url]                # 在控制台中打开指定url，可以指定--spider=spider_name, -c code , --no-redirect, --nolog
- scrapy parse <url> [options]      # 获取并解析，使用--callback或-c指定response的解析回调函数，--spider=spider_name, --a NAME=VALUE, --pipelines, --rules或-r, --noitems, --nolinks, --nocolour, --depth或-d指定深度,--verbose或-v
- scrapy settings [options]         # 如scrapy settings --get BOT_NAME 或 scrapy settings --get DOWNLOAD_DELAY
- scrapy runspider <spider_file.py>  # 只是去运行一个py爬虫文件，并且之前没有创建工程
- scrapy version [-v]
- scrapy benchscrapy bench          # new in 0.17, 快速地benchmark测试
``` 
# 18 Scrapy 学习-3
#### Spiders类

- 首先执行start_requests()方法中的start_urls产生Request对象，并调用回调方法parse();　
- 回调函数中完成response的解析，返回提取的数据，可以是字典、Item对象、Request对象或可迭代对象，他们也将包含回调函数（可以与上面的相同），之后又scrapy自动下载并由回调函数完成相应的处理；
- 在回调函数中，完成内容的解析，通常使用Selectors，当然也可使用擅长的任意解析方式，生成items；
- 最后，返回的items将持久化到数据库，在[Item Pipleine](https://doc.scrapy.org/en/latest/topics/item-pipeline.html#topics-item-pipeline)或[Feed exports](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-exports)中指定。

#### scrapy.Spider
最简单的爬虫基类，其他的爬虫都必须进行继承，只提供默认的start_requests()方法，通过start_urls属性发送requests，之后parse()处理请求的responses
```
- name             # 爬虫的名字，必须是唯一的；py2中还必须是ASCII的
- allowed_domains  # 一个列表，规定了爬虫的范围；与开启OffsiteMiddleware意思一样
- start_urls       # 列表
- custom_settings  #  字典，将覆盖工程级的设定，必须定义为类的数学，因为它在实例前执行，[详细](https://doc.scrapy.org/en/latest/topics/settings.html#topics-settings-ref)
- logger           # python logger created with the spider's name, [详细](https://doc.scrapy.org/en/latest/topics/logging.html#topics-logging-from-spiders)
- crawler          # 通过from_crawler()类方法设置，非常强大，[详细](https://doc.scrapy.org/en/latest/topics/api.html#topics-api-crawler)
- settings         # configuration for running this spider,[详细](https://doc.scrapy.org/en/latest/topics/spiders.html)
- 各种法法：from_crawler(crawler, *args, **kwargs), start_requests(), make_requests_from_url(url), parse(response), log(message[,level,component]), closed(reason), [详细](https://doc.scrapy.org/en/latest/topics/spiders.html)

twisted.internet.error.DNSLookupError: DNS lookup failed: address "'http" not found: [Errno 11004] getaddrinfo failed.
>> 将单引号改正为双引号
```
#### 发送POST请求
```
    from scrapy.http import FormRequest
    
    # Start on the welcome page
    def start_requests(self):
        return [
            Request(
                "http://web:9312/dynamic/nonce", callback=self.parse_welcome)
        ]
    # Post welcome page's first form with the given user/pass
        def parse_welcome(self, response):
            return FormRequest.from_response(
                response,
                formdata={"user": "user", "pass": "pass"}
            )
```
#### meta是一个dict，主要是用解析函数之间传递值；多个解析函数直接的传递需要一直带着
一种常见的情况：在parse中给item某些字段提取了值，但是另外一些值需要在parse_item中提取，这时候需要将parse中的item传到parse_item方法中处理，显然无法直接给parse_item设置而外参数。 Request对象接受一个meta参数，一个字典对象，同时Response对象有一个meta属性可以取到相应request传过来的meta。
```
        yield scrapy.Request(url=response.urljoin(category_small_url),
                             callback=self.third_parse,
                             meta={"category_small_name": category_small_name,
                                   "category_big_name": response.meta['category_big_name']})
                                   
        # response.meta.get('key', 'if_not_so_set_the_giving_value')
```


# 19 MongoDB Scrapy Pycharm
```
windows 端口管理windows 端口管理

netstat -a -n  # 以数字形式显示TCP和UDP连接的端口号和状态
netstat -a -n | findstr 27017
cmd >>>  gpedit.msc
```
#### 涉及到考虑端口，是由于存在windows下处理了27017端口占用的情况，采用另指定端口的方式
### 重要
```
mongod --dbpath data\db --port 27018  # 首先：启动服务并指定端口
mongo --port 27018                    # 让后：控制台进入mongo是指定端口，端口不一致看不到show dbs;
# scrapy crawl dangdang  # 启动scrapy爬虫，采用mongo存储;virtualenv用的scrapy_3env
```
scrapy中采用mongodb存储数据，setting.py <-> pipline.py
查看数据采用的是pycharm的mongo插件，进行可视化查看
```
MONGODB_HOST = '127.0.0.1'                  # settings.py
MONGODB_PORT = 27018
MONGODB_DBNAME = "dangdang"
MONGODB_DOCNAME  = "saveinto_2"

import pymongo                                     # piplines.py
from scrapy.conf import settings
from .items import DangdangItem

class DangdangPipeline(object):

    def __init__(self):

        client = pymongo.MongoClient(host=settings['MONGODB_HOST'],
                                     port=settings['MONGODB_PORT'])
        tdb = client[settings['MONGODB_DBNAME']]
        
        self.post = tdb[settings['MONGODB_DOCNAME']]

    def process_item(self, item, spider):
        if isinstance(item, DangdangItem):
            try:
                self.post.insert(dict(item))
            except Exception as e:
                pass
        return item
```
![image](https://github.com/Django-27/workspace/blob/master/pic/mongo-scrapy.png)
        ipython
        ``` 
        import pymongo
        conn = pymongo.MongoClient('localhost', 27018)
        db = conn.dangdang
        coll = db.saveinto_2
        coll.count()      # 暂时是87396条数据
        ```
        cmd
        ```
        mongo --port 27018
        > show dbs;           # admin、dangdang、local; use deomdb 将自动创建一个
        > use dangdang;       
        > show collections;        # 显示doc， svaeinto_1、 saveinto_2
        > db.getCollectionNames()  # 显示doc
        > db.saveinto_2.find()     # 回显记录
        > db.saveinto_1.findOne()  # 回显一条记录
        > db.saveinto_2.count()    # 记录总数
        > db.saveinto_2.count({'book_author': 'Alon'})
        > db.saveinto_2.find({'book_author': 'Alon'}).count()
        
        > db.saveinto_2.insert({'book_author': 'Jack'})
        # 查询条件 $lt $lte $gt $gte 分别对应 < <= > >=   $ne 不等于, 还有 $in $not $or  
        # db.saveinto_2.find({age: {$gt: 33, $lt: 43}})
        
        # null可以匹配自身, 而且可以匹配"不存在的"
        > db.Student.insert({name:null,sex:1,age:18}) # 插入一条测试数据
        > db.Student.insert({sex:1,age:24})
        > db.Student.find({name:null})        --上面两条都能查到
        > db.Student.find({name:{$in:[null],$exists:true}})  ---只能查到第一条 
        
        > db.Student.find().sort({age:1,sex:1})  # 指定排序， 1升序 -1降序
        > db.Student.find().sort({age:1}).limit(3).skip(3)  # 限制
        > db.Student.remove({name:null})             # 删除数据
        > db.Student.update({name:"jack"},{age:55})  # 更新数据
        > db.Student.update({name:"lily"},{$inc:{height:175}})   # 增加指定量
        ```

### 重点补充：
- scrapy 中如果含有多级的解析，如 parse -> parse_2 -> parse_3 , 其中如果传递参数，建议使用item方式
- 如果不是一开始就初始一个 DangdangItem() , 而每次都带着自己构造的字典进行参数传递， 可能在多个parse之后出现 KeyError 错误，提示找不到自己字典的键值
- 学习中发现，有时中断scrapy的过程后，在此开始但是一条结果都找不到，返回finished；通过在每一次scrapy.Respect() 中增加参数dont_filter=True, 表示不筛选，否则筛选url
- 
```
    def parse(self, response):
        item = DangdangItem()
        item['category_big_name'] = goods.xpath('a/@title')[0] 
        return yield scrapy.Request(url= , cookises=, headers=, meta={'item': item}, callback=self.parse_2, dont_filter=True)

    def parse_2(self, response):
        item = response.meta['item']
        item['category_small_name'] = goods.xpath('a/@title')[0]
        return yield scrapy.Request(url=response.urljoin('**'), callback=self.third_parse, meta={'item': item}, dont_filter=True)

    def parse_3(self, response):
        """同理"""
        return item
        """此处还可以继续翻页"""
```
# 22 如何调试 debug scrapy
需要新建文件，命名任意，但是要写入下面的代码，之后右键 debug 此文件即可
```
from scrapy import cmdline
cmdline.execute("scrapy crawl your_spider_name".split())
```
![image](https://github.com/Django-27/workspace/blob/master/pic/debug-scrapy.pn)

# 20 定时任务设置crontab 和 python-crontab
#### Linux 自带定时任务功能 crontab　
crontab [-u user] -l 查看当前用户已有定时任务
crontab [-u user] -e 编辑当前用户定时任务
crontab [-u user] -r 删除当前用户所有定时任务
crontab [-u user] -v 查看当前用户所有定时任务状态

- 首先编辑一个crontab文件，使用规定的格式，[在线生成](https://crontab.guru/every-1-minute)时间字段
- 之后提交crontab文件 crontab cronfile 这样就将crontab文件提交给cron进程，并保存副本到/var/spool/cron/crontabs/用户名 
### 实战：定时每一分钟执行一次scrapy下的项目
```
* * * * * cd /home/ubuntu/qiyuan && /usr/local/bin/scrapy crawl quotes
```
- 文件:   /etc/cron.deny  该文件中所列的用户不允许使用crontab命令， 对应的是.allow
- 文件:   /var/spool/cron/  所有用户crontab文件存放的目录,以用户名命名;每一行都代表一项任务,格式共分为六个字段，前五段是时间设定段，第六段是要执行的命令段

#### pip install python-crontab  
提供python脚本进行自动化执行管理，[link](https://pypi.python.org/pypi/python-crontab)
##### chmod 控制文件或目录的访问权限
- 表示增加权限、- 表示取消权限、= 表示唯一设定权限, 如chmod +x
-  Linux/Unix 的档案存取权限分为三级 : 档案拥有者、群组、其他
-  u 表示该档案的拥有者，g 表示与该档案的拥有者属于同一个群体(group)者，o 表示其他以外的人，a 表示这三者皆是
- r 表示可读取，w 表示可写入，x 表示可执行，X表示只有当该档案是个子目录或者该档案已经被设定过为可执行
- 此外chmod也可以用数字来表示权限如 chmod 777 file 

