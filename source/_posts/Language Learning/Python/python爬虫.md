---
title: 基本的Python爬虫
date: 2022-12-23 12:23:23
tags:
  - Python
  - Language Learning
categories:
  - Python
  - Language Learning
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# 基本的Python爬虫

## 第一步：引入包并准备临时存储数据的列表

```python
import urllib.request
import urllib.parse
import requests
from bs4 import BeautifulSoup
import re
import random, time # 设置每个网页之间的爬取间隔，防止被ban
# import xlwt, xlwings, sqlite3
# import selenium

# 1.设定爬取网页对象
url = ""
# 2 准备临时保存数据的列表
A = []
B = []
...
# 3 文件保存路径
saveAddress = "D:\\project\\code\\python\\"
# 4 设立序号num
num = 0
```

## 第二步：获取网页内容（源代码）

#### 采用**urllib**

1. 使用手册：

    [Python urllib | 菜鸟教程 (runoob.com)](https://www.runoob.com/python3/python-urllib.html)

    [urllib.request — Extensible library for opening URLs — Python 3.9.6 documentation](https://docs.python.org/3/library/urllib.request.html#module-urllib.request)

2. 简明教程：

   > 1. **urllib**包内文件设计：
   >
   >    - **urllib.request** - 打开和读取 URL。它定义了一些打开 URL 的函数和类，包含**授权验证**、**重定向**、**浏览器 cookies**等。
   >    - **urllib.error** - 包含 urllib.request 抛出的异常。
   >    - **urllib.parse** - 解析 URL。
   >    - **urllib.robotparser** - 解析 robots.txt 文件。
   >
   > 2. 打开一个 URL：
   >
   >    urllib.request.urlopen(url, data=None, [timeout, ], cafile=None, capath=None, cadefault=False, context=None)
   >
   >     注意返回值：This function always returns an object which can work as a [context manager](https://docs.python.org/3/glossary.html#term-context-manager) **and** has the properties _url_, _headers_, and _status_.
   >
   >     注意 url 对象：Open the URL _url_, which can be either a string or a [`Request`](https://docs.python.org/3/library/urllib.request.html#urllib.request.Request) object.
   >
   > 3. 解析 urlopen()的返回值：（注意还要 decode()）
   >
   >     read( [length = number] )
   >
   >     readline( )
   >
   >     readlines( [ ] )：返回列表
   >
   > 4. 模拟头部信息（身份伪装）：
   >
   >     urllib.request.Request(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None)
   >
   >     ~~如果用到了 Request 对象，自然是直接将 data 放在这里面而不是在 urlopen 里~~

```python
def ask_html(i):
	# 对基础url做处理，拿到新的、需要爬取的实际目标网页url，并在接下来对这个网址做处理
	tempUrl = url + str(i * 25)
	# 伪装user-agent
	headers = {
		"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) ......"
	}
		# # 用户表单信息
		# data = bytes(urllib.parse.urlencode({'user_name': 'bunjie'}), encoding='utf-8')，是urlopen中的data参数，用来发送post请求，否则是get请求
	# 对爬虫身份做封装
	response = urllib.request.Request(url=url_now, headers=headers, method="GET") # 要发送表单信息的话，请使用POST
	# 进行网页爬取并解码
	req = urllib.request.urlopen(response, timeout=30)
	html = req.read().decode('utf-8')
	# 返回目标网页对应的html代码内容
	return html
```

#### 采用 requests

> 1. 官方示例：

```python
>>> r = requests.get('https://api.github.com/user', auth=('user', 'pass'))
>>> r.status_code
200
>>> r.headers['content-type']
'application/json; charset=utf8'
>>> r.encoding
'utf-8'
>>> r.text
'{"type":"User"...'
>>> r.json()
{'private_gists': 419, 'total_private_repos': 77, ...}
```

> 2. 文档：
>
>     [Requests: HTTP for Humans™ — Requests 2.26.0 documentation (python-requests.org)](https://docs.python-requests.org/en/master/)
>
>     [Requests: 让 HTTP 服务人类 — Requests 2.18.1 文档 (python-requests.org)](https://docs.python-requests.org/zh_CN/latest/)
>
> 3. 简明教程：
>
>    1. API Reference:
>
>       > **`requests`.`get`**(_url_, _params=None_, _\*\*kwargs_)
>       >
>       > | Parameters: | **url** – URL for the new [`Request`](https://docs.python-requests.org/en/latest/api/#requests.Request) object.<br>**params** – (optional) Dictionary, list of tuples or bytes to send in the query string for the [`Request`](https://docs.python-requests.org/en/latest/api/#requests.Request).<br/>**\*\*kwargs** – Optional arguments that `request` takes. |
>       > | :---------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
>       > | Returns:    | [`Response`](https://docs.python-requests.org/en/latest/api/#requests.Response) object                                                                                                                                                                                                                                                                          |
>
>       > **`requests`.`post`**(_url_, _data=None_, _json=None_, _\*\*kwargs_)
>       >
>       > Sends a POST request.
>       >
>       > | Parameters: | **url** – URL for the new [`Request`](https://docs.python-requests.org/en/latest/api/#requests.Request) object.<br>**data** – (optional) Dictionary, list of tuples, bytes, or file-like object to send in the body of the [`Request`](https://docs.python-requests.org/en/latest/api/#requests.Request).<br/>**json** – (optional) json data to send in the body of the [`Request`](https://docs.python-requests.org/en/latest/api/#requests.Request).<br/>**\*\*kwargs** – Optional arguments that `request` takes. |
>       > | :---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
>       > | Returns:    | [`Response`](https://docs.python-requests.org/en/latest/api/#requests.Response) object                                                                                                                                                                                                                                                                                                                                                                                                                                |
>
>       > **\*\*kwargs 参数列表:**
>       >
>       > - **params** – (optional) Dictionary, list of tuples or bytes to send in the query string for the [`Request`](https://docs.python-requests.org/en/latest/api/#requests.Request).
>       > - **data** – (optional) Dictionary, list of tuples, bytes, or file-like object to send in the body of the [`Request`](https://docs.python-requests.org/en/latest/api/#requests.Request).
>       > - **json** – (optional) A JSON serializable Python object to send in the body of the [`Request`](https://docs.python-requests.org/en/latest/api/#requests.Request).
>       > - **headers** – (optional) Dictionary of HTTP Headers to send with the [`Request`](https://docs.python-requests.org/en/latest/api/#requests.Request).
>       > - **cookies** – (optional) Dict or CookieJar object to send with the [`Request`](https://docs.python-requests.org/en/latest/api/#requests.Request).
>       > - **files** – (optional) Dictionary of `'name': file-like-objects` (or `{'name': file-tuple}`) for multipart encoding upload. `file-tuple` can be a 2-tuple `('filename', fileobj)`, 3-tuple `('filename', fileobj, 'content_type')` or a 4-tuple `('filename', fileobj, 'content_type', custom_headers)`, where `'content-type'` is a string defining the content type of the given file and `custom_headers` a dict-like object containing additional headers to add for the file.
>       > - **auth** – (optional) Auth tuple to enable Basic/Digest/Custom HTTP Auth.
>       > - **timeout** ([_float_](https://docs.python.org/3/library/functions.html#float) _or_ [_tuple_](https://docs.python.org/3/library/stdtypes.html#tuple)) – (optional) How many seconds to wait for the server to send data before giving up, as a float, or a [(connect timeout, read timeout)](https://docs.python-requests.org/en/latest/user/advanced/#timeouts) tuple.
>       > - **allow_redirects** ([_bool_](https://docs.python.org/3/library/functions.html#bool)) – (optional) Boolean. Enable/disable GET/OPTIONS/POST/PUT/PATCH/DELETE/HEAD redirection. Defaults to `True`.
>       > - **proxies** – (optional) Dictionary mapping protocol to the URL of the proxy.
>       > - **verify** – (optional) Either a boolean, in which case it controls whether we verify the server’s TLS certificate, or a string, in which case it must be a path to a CA bundle to use. Defaults to `True`.
>       > - **stream** – (optional) if `False`, the response content will be immediately downloaded.
>       > - **cert** – (optional) if String, path to ssl client cert file (.pem). If Tuple, (‘cert’, ‘key’) pair.
>
> ````powershell
>   > *class* **`requests`.`Response`**
>   >
>   > The [`Response`](https://docs.python-requests.org/en/latest/api/#requests.Response) object, which contains a server’s response to an HTTP request.
>   >
>   > 1. **content**
>   >
>   > ​		Content of the response, in bytes.
>   >
>   > 2. **text**
>   >
>   >    Content of the response, in unicode.
>   >
>   >    If Response.encoding is None, encoding will be guessed using `charset_normalizer` or `chardet`.
>   >
>   >    The encoding of the response content is determined based solely on HTTP headers, following RFC 2616 to the letter. If you can take advantage of non-HTTP knowledge to make a better guess at the encoding, you should set `r.encoding` appropriately before accessing this property.
>   >
>   > 3. **`url` *= None***
>   >
>   >    Final URL location of Response.
>   >
>   > 4. **`status_code` *= None***
>   >
>   >    Integer Code of responded HTTP Status, e.g. 404 or 200.
>   >
>   > 5. **`request` *= None***
>   >
>   >    The [`PreparedRequest`](https://docs.python-requests.org/en/latest/api/#requests.PreparedRequest) object to which this is a response.
>   >
>   >    You can **check the response** header through this method!
>   >
>   > > When you make a request, Requests makes educated guesses about the encoding of the response based on the HTTP headers. The text encoding guessed by Requests is used when you access **`r.text`**. You can find out what encoding Requests is using, and change it, using the **`r.encoding`** property:
>   > >
>   > > ```python
>   > > >>> r.encoding
>   > > 'utf-8'
>   > > >>> r.encoding = 'ISO-8859-1'
>   > > ```
> ````
>
> 2.  ## Make a Request
>
> > ```python
> > >>> import requests # first import the module
> >
> > >>> r = requests.get('https://api.github.com/events') # get a webpage. In GET requests mode
> > ```
> >
> > Now, we have a Response object called r. We can get all the information we need from this object.
> >
> > meanwhile, requests’ simple API means that all forms of HTTP request are as obvious. For example, this is how you make an HTTP POST request:
> >
> > ```python
> > >>> r = requests.post('https://httpbin.org/post', data = {'key':'value'})
> > # 我们当热可以在之前就将要发送的数据包data封装好，在post中就可以直接调用了
> > ```
> >
> > Nice, rights? What about the other HTTP request types: PUT, DELETE, HEAD and OPTIONS? These are all just as simple:
> >
> > ```python
> > >>> r = requests.put('https://httpbin.org/put', data = {'key':'value'})
> > >>> r = requests.delete('https://httpbin.org/delete')
> > >>> r = requests.head('https://httpbin.org/get')
> > >>> r = requests.options('https://httpbin.org/get')
> > ```
>
> 2.  ## Passing Parameters In URLs
>
> > You often want to send some sort of data in the URL’s query string. If you were constructing the URL by hand, this data would be given as key/value pairs in the URL after a question mark, e.g. `httpbin.org/get?key=val`. Requests allows you to provide these arguments as a dictionary of strings, using the `params` keyword argument. As an example, if you wanted to pass `key1=value1` and `key2=value2` to `httpbin.org/get`, you would use the following code:
> >
> > ```python
> > >>> payload = {'key1': 'value1', 'key2': 'value2'}
> > >>> r = requests.get('https://httpbin.org/get', params=payload)
> > ```
> >
> > You can see that the URL has been correctly encoded by printing the URL:
> >
> > ```python
> > >>> print(r.url)
> > https://httpbin.org/get?key2=value2&key1=value1
> > ```
> >
> > Note that any dictionary key whose value is `None` will not be added to the URL’s query string.

## 第三步：利用 bs 和 re 来对爬取内容做处理

#### bs4：一级处理（可跳过）

1. 使用手册：

    [Beautiful Soup 4.4.0 文档 — Beautiful Soup 4.2.0 中文 文档](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/)

2. 简明教程：

   > 1. 创建 BeautifulSoup 对象：
   >
   >     将一段文档传入 BeautifulSoup 的构造方法,就能得到一个文档的对象, 可以传入一段**字符串**或一个**文件**句柄.
   >
   >    ```python
   >    soup = BeautifulSoup(open("index.html")) # 方法一：读取文件
   >
   >    soup = BeautifulSoup("<html>data</html>") # 方法二：读取现成、字符串形式的html代码片段
   >    ```
   >
   >     首先,文档被转换成 Unicode,并且 HTML 的实例都被转换成 Unicode 编码
   >
   >    ```python
   >    BeautifulSoup("Sacr&eacute; bleu!")
   >    <html><head></head><body>Sacré bleu!</body></html>
   >    ```
   >
   >     然后,Beautiful Soup 选择最合适的解析器来解析这段文档,如果手动指定解析器那么 Beautiful Soup 会选择指定的解析器来解析文档.(参考 [解析成 XML](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#xml) ).
   >
   > 2. 对文档进行处理：
   >
   >     Beautiful Soup 将复杂 HTML 文档转换成一个复杂的树形结构,每个节点都是 Python 对象,所有对象可以归纳为 4 种: `Tag` , `NavigableString` , `BeautifulSoup` , `Comment` .
   >
   >     **`Tag`**：即原文档中的标签，可以通过**name**和**attrs**来直接调用对应的名称和属性，请*务必注意属性的返回值！*我们还可以通过**string**来调用内容，但是内容不能被直接修改
   >
   >     ~~如果只想得到 tag 中包含的文本内容,那么可以嗲用 `get_text()` 方法,这个方法获取到 tag 中包含的所有文版内容包括子孙 tag 中的内容,并将结果作为 Unicode 字符串返回~~
   >
   >     **`NavigableString`**：字符串常被包含在 tag 内.Beautiful Soup 用 `NavigableString` 类来包装 tag 中的字符串，tag 中包含的字符串不能编辑,但是可以被替换成其它的字符串,用 [replace_with()](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#replace-with) 方法:
   >
   >    1. 进行文档遍历：低效
   >
   >    2. 文档搜索（√）：
   >
   >       1. select()css 选择器，支持 id(#)，class(.)，tag，父子选择等 css 匹配规则
   >
   >           example: bs.select(a[class = 'hello!' ])
   >
   >       2. find_all()几乎同 select，但是支持了正则表达式
   >
   >           example: bs.find*all('div', class*='item') / bs.find_all(re.compile("a"))

#### re：二级处理（从根本解决问题）

1. 正则表达式使用手册：

    [正则表达式](onenote:https://d.docs.live.net/a85fd0932d62f536/文档/JBR_Bunjie/开发笔记/前端开发笔记/H5.one#正则表达式&section-id={95089512-DFC7-49F9-A7C1-466EEB6B429A}&page-id={262783D3-2B94-4AF5-877C-BDB6CBCD4806}&end) ([Web 视图](<https://onedrive.live.com/view.aspx?resid=A85FD0932D62F536!617&id=documents&wd=target(开发笔记%2F前端开发笔记%2FH5.one|95089512-DFC7-49F9-A7C1-466EEB6B429A%2F正则表达式|262783D3-2B94-4AF5-877C-BDB6CBCD4806%2F)>))

2. re 模块使用手册：

    [re 模块](onenote:https://d.docs.live.net/a85fd0932d62f536/文档/JBR_Bunjie/开发笔记/Linux_Windows/python.one#re模块&section-id={973AA90C-36D3-413E-AB01-5823E7C25713}&page-id={6914C45C-28DA-48B3-A552-0A19C3C9064D}&end) ([Web 视图](<https://onedrive.live.com/view.aspx?resid=A85FD0932D62F536!617&id=documents&wd=target(开发笔记%2FLinux_Windows%2Fpython.one|973AA90C-36D3-413E-AB01-5823E7C25713%2Fre模块|6914C45C-28DA-48B3-A552-0A19C3C9064D%2F)>))

```python
# 全部html代码已经在前面的url处理中完全取得
def get_information(html):
	global num
	bs = BeautifulSoup(html, 'lxml') # 推荐使用lxml作为解析器,因为效率更高.

	find_movie_address = re.compile(r'<a href="(.*?)">', re.S)
	find_title = re.compile(r'<span class="title">(.*?)</span>', re.S)
	find_more_title = re.compile(r'<span class="other">(.*?)</span>', re.S)

	for item in bs.find_all('div', class_='item'):
		print('-'*30)
		print(num + 1)
		item = str(item)
		movie_address_data.append(re.findall(find_movie_address, item)[0])

		if len(re.findall(find_title, item)) == 2:   # 不是所有影片都有“两个title”的class
			movie_title_data.append(re.findall(find_title, item)[0])
			movie_traditional_title_data.append(re.findall(find_title, item)[1])
		else:
			movie_title_data.append(re.findall(find_title, item)[0])
			movie_traditional_title_data.append(' ')  # 用空格' '来代表该电影是中文电影，没有外文“原名”

		movie_more_title_data.append(re.findall(find_more_title, item)[0])

		temp = movie_title_data[num] + movie_traditional_title_data[num] + movie_more_title_data[num]
		movie_title_collection.append(temp)

		print(movie_address_data[num])
		print(movie_title_collection[num])
		num = num + 1
	# 模拟正常浏览网页的停留时间
	time.sleep(random.random() * 100)
```

## 第四步：保存数据

1. excel 表：[Python Resources for working with Excel - Working with Excel Files in Python (python-excel.org)](http://www.python-excel.org/)

    xlwings：[Automate Excel with Python (Open Source and Free) (xlwings.org)](https://www.xlwings.org/)

   ```python
   # xlwings示例代码：
   
   import xlwings as xw
   
   #连接到excel
   workbook = xw.Book(r'path/myexcel.xlsx')#连接excel文件
   #连接到指定单元格
   data_range = workbook.sheets('Sheet1').range('A1')
   #写入数据
   data_range.value = [1,2,3]
   #保存
   workbook.save()
   ```

以上步骤综合即是一个完整的 python 爬虫程序
