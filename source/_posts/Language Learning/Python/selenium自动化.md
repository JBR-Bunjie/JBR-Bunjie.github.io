---
title: selenium自动化爬虫
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

# 目标：使用 selenium 来解决网页中的大量同质化人工操作内容

## 零：想好该怎么做：

### 关于指定网站：

网站是网盘网站，其文件有密码保护，即每个文件都有不同且无规律的受保护地址；同时，这些网页的操作完全重复。

我们可以使用 selenium 来完成自动化代替重复劳动。

### 关于 selenium

- 需要使用浏览器的自动化 driver
- 需要使用键盘输入模拟模块
- 需要使用鼠标输入模拟模块
- 需要事先了解需要操作的网页元素对象的路径

## 一、准备工作：将所有需要爬取的地址整合到一个列表里：

```python
import os

path = r"D:\Project\Code Trainning\Learning\PythonDemo"
filename = r"addresslist.txt" # 存储所有网址的文件
code = "849227" # 网站文件的密码

os.chdir(path)
os.getcwd()

f = open(file=filename, mode='r', encoding='utf-8')
FileContent = f.readlines()
f.close()

AddressList = []
t = 1
for i in FileContent:
    if t % 2 == 0 and t % 4 != 0:
        AddressList.append(i)
    t = t + 1

# for i in AddressList:
#     print(i)
```

## 二、用 selenium 来打开这些网页并模拟操作：

### 打开网页：

```python
from selenium import webdriver # selenium.webdriver模块提供了所有WebDriver的实现
from selenium.webdriver.common.keys import Keys # Keys类提供了键盘的代码，用来输入特殊的键盘符（如：回车,ALT,F1等等）
	# 比如，在上条语句输入之后，可以使用Keys.ENTER来模拟输入回车符
import time

wd = webdriver.Edge(r"D:\language\EdgeDriver\msedgedriver.exe") # 创建一个Edge浏览器的实例

try:
    for i in AddressList:
        wd.get(i)

        …… # 接第二步

        time.sleep(60)
except Exception as e:
    print(e)

    # driver.get方法会导向给定的URL的页面，WebDriver会等待页面完全加载完(就是onload函数被触发了)，才把程序的控制权交给你的测试或者脚本。
    # 但是！如果 你的页面用了太多的AJAX，那么这个机制将会失效，因为原本完整的页面只占用很小一部分时间，而ajax是“页面完成之后的操作”，selenium根本不知道页面到底是什么时候加载完。就像是requests面对众多ajax存在的网页一样
```

### 对网页进行操作

```python
time.sleep(5) # 最差的实现方式，最好使用隐式等待


input_box1 = wd.find_element_by_id('passcode') # 找到输入box
input_box1.send_keys(code) # 输入验证码，send_keys函数能模拟大部分的键盘输入，其他的需要Keys类来实现
confirm_button1 = wd.find_element_by_xpath(r"/html/body/main/div/div[1]/div/div/div/div[2]/div[2]/button")
confirm_button1.click() # 模拟鼠标电机

time.sleep(5) # 如上

confirm_button2 = wd.find_element_by_xpath(r"/html/body/main/div/div/div[4]/div[1]/div[2]/button")
confirm_button2.click()

time.sleep(180)
wd.find_element_by_tag_name('body').send_keys(Keys.Control + 't') # 模拟组合键

"""
显式Waits
	+ 用WebDriverWait结合ExpectedCondition来实现：
		element = selenium.webdriver.support.ui.WebDriverWait(wd, 10).unitl(
			selenium.webdriver.support.expected_conditions.presence_of_located(By.ID, "anyIDisOK")
		)
		# 这段代码将会等待10秒，并在等待过程中，每0.5s就调用一下ExpectedCondition，如果成功则立即返回，否则持续重试直到超时报错，报出错误TimeoutException
	+ 显示Wait中有许多预期条件，这样子就无需自己编写expected_condition（见文档

隐式Waits
please forget that，but when I understand，I will c
"""
```

### 结束

```python
wd.close() # 或者 wb.quit()
```
