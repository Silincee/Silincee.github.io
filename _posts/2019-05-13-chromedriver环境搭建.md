---
layout: post
title:  "chromedriver环境搭建"
date:   2019-05-13 12:20:06 +0800--
categories: [爬虫]
tags: [spider, ]  

---

# 方法一：

设置chromedriver为环境变量：

1.下载chromedriver_mac64.zip

[selenium之 chromedriver与chrome版本映射表(更新至v2.29)](https://links.jianshu.com/go?to=http%3A%2F%2Fblog.csdn.net%2Fhuilan_same%2Farticle%2Fdetails%2F51896672)

[chromedriver下载](https://links.jianshu.com/go?to=http%3A%2F%2Fjingyan.baidu.com%2Farticle%2Feb9f7b6d87e2ae869264e847.html)

2.解压得到chromedriver

3.把解压后的文件放到/usr/local/bin/下，结果像这样：/usr/local/bin/chromedriver

(1)    cd /usr/local/bin/

(2)   open .

(3)  把chromedriver  拖进来

4.添加环境变量

export PATH=$PATH:/usr/local/bin/chromedriver

运行代码：



```swift
from selenium import webdriver
driver = webdriver.Chrome()
base_url ='https://www.baidu.com'
driver.get(base_url)
```

⭐️方法二：

指定chromedriver的位置，然后实例化时指定驱动的位置：

```csharp
from seleniumimport webdriver
driver=webdriver.Chrome('../../venv/chromedriver')
driver.get("https://www.baidu.com")
```

