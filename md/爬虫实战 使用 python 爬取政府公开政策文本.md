> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/jycmbetter/p/17267293.html)

目标：爬取北京市公开发布的所有人才引进相关的政策文本

准备：1、环境 Python 3.7，2、使用 selenium 库中的 webdriver，3、安装对应版本的 chromedriver

url：在北京市人民政府网站上，人才引进相关政策的 url 地址是：[https://www.beijing.gov.cn/so/s?siteCode=1100000088&tab=zcfg&qt=%E4%BA%BA%E6%89%8D%E5%BC%95%E8%BF%9B](https://www.beijing.gov.cn/so/s?siteCode=1100000088&tab=zcfg&qt=%E4%BA%BA%E6%89%8D%E5%BC%95%E8%BF%9B)

一些参考：[(1 条消息) Python 爬虫（1）一次性搞定 Selenium(新版)8 种 find_element 元素定位方式_find_element(by.class_name_轻烟飘荡的博客 - CSDN 博客](https://blog.csdn.net/qq_16519957/article/details/128740502)

　　　　　[(1 条消息) Selenium --- 标签 class 及其他属性值更改_selenium 修改属性值_Ethel L 的博客 - CSDN 博客](https://blog.csdn.net/weixin_41708323/article/details/118965181)

目标网站 html 内容
============

　　在浏览器输入 url，可以看见网站的基本内容见下图，我们需要爬取的是图中画线部分：

![](https://img2023.cnblogs.com/blog/3150827/202303/3150827-20230329112026720-818415105.png)

　　接下来看一下画线部分的 html 文本，可以看见每一个政策条目都都在 class 为'search-result'的 div 标签里，查看这个 div 标签，我们可以看见政策的类型，标题，链接等内容。

![](https://img2023.cnblogs.com/blog/3150827/202303/3150827-20230329111942030-125912377.png)

 ![](https://img2023.cnblogs.com/blog/3150827/202303/3150827-20230329111959953-829904366.png)

　　那么只需要找到所有的 class 为'search-result'的 div 标签，然后循环搜索里面的信息（注意 a 标签里的链接是关键信息，后续需要爬取链接里的具体政策内容），并保存到 csv 文件中。这里使用的是 find_element_XX_XX 函数，如果 selenium 版本比较新，可能这几个函数无法使用了，可以改为使用 find_element(By.xx,'xx')，这里需要引入 selenium.webdriver.common.by。

解决 js 渲染的翻页内容
=============

　　在查看网页的过程中，发现跳转下一页的内容，url 并未发生变化，是根据 JavaScript 动态渲染出的内容，这里就使用 find_element 找到下一页的按钮然后 click 它。代码位于 29 行和 44 行。

解决搜索位置的切换
=========

　　还有一个需求是按标题检索含有 “人才引进” 的政策文本，查看 html 发现，这两个在交互中能够点击的标签在 html 中是 div 标签，一般来说，div 标签是无法点击的，并且初步来看这里没有 js 代码，但是切换搜索位置是全文还是标签，url 并没有改变。这也是我比较困惑的地方，如果有了解的大佬可以在评论中解决一下我的疑惑。虽然不太知道这里的原因，但是我发现点击标题，则标题的 class 变为 position-con item-choose item-choose-on，全文的 class 变为 position-con item-choose，点击全文则反过来，那么我们可以执行 js 代码修改这两个标签的属性，实现搜索位置的切换，代码位于 17-21 行。

![](https://img2023.cnblogs.com/blog/3150827/202303/3150827-20230329111914980-1159244987.png)

![](https://img2023.cnblogs.com/blog/3150827/202303/3150827-20230329111854801-1205983816.png)

具体代码如下：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
1 # -*- coding: utf-8 -*-
 2 """
 3 Created on Tue Mar 28 16:46:21 2023
 5 @author:
 6 """
 8 from selenium import webdriver
 9 import time
10 import pandas as pd
11 from selenium.webdriver.common.by import By
13 browser=webdriver.Chrome()
14 browser.get('https://www.beijing.gov.cn/so/s?siteCode=1100000088&tab=zcfg&qt=%E4%BA%BA%E6%89%8D%E5%BC%95%E8%BF%9B')
15 time.sleep(5)
17 lable=browser.find_elements(By.CSS_SELECTOR,'.position-con.item-choose')#查找全文和标题的检索项
18 print(lable[0].text,lable[1].text)
19 js= 'arguments[0].setAttribute(arguments[1],arguments[2])'
20 browser.execute_script(js,lable[0],'class','position-con item-choose')#修改全文标签的class
21 browser.execute_script(js,lable[1],'class','position-con item-choose item-choose-on')#修改标题标签的class
24 time.sleep(2)
25 data=pd.DataFrame([],columns=['类型','链接','标题','文号','发文机构','主题分类','发布日期'])
26 page=1
27 while page:
28     if page !=1:
29         page.click()
30         time.sleep(1)
31     poli=browser.find_elements_by_class_name('search-result')
32     for elements in poli:#一个elements是一个记录
33         p_type=elements.find_element_by_class_name("result-header-lable").text#政策类型
34         link=elements.find_element_by_tag_name("a").get_attribute('href')#链接
35         title=elements.find_element_by_tag_name("a").text#标题
36         table=elements.find_elements_by_class_name("row-content")#文号、发文机构、主题分类、发布日期
37         content=[p_type,link,title]
38         for item in table:
39             content.append(item.text)
40         while len(content)<7:
41             content.append(0)
42         content=pd.DataFrame([content],columns=['类型','链接','标题','文号','发文机构','主题分类','发布日期'])
43         data=pd.concat([data,content])
44     page=browser.find_element_by_class_name('next')
45 data.to_csv(r'人才政策爬取\北京市人才政策.csv',encoding='utf_8_sig',index=False)
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

 提取政策的具体内容
==========

　　在上面的代码中，我们实现了爬取所有标题含 “人才引进” 的政策的大致信息，结果类似下图的表格内容。

![](https://img2023.cnblogs.com/blog/3150827/202303/3150827-20230329111809169-1313791754.png)

接下来需要做的是逐个读取每个政策的链接，将其中的文本内容提取到 txt 文件中，从 html 文本中可以看到文本内容均位于 id='mainText'的 div 标签中，每个 p 标签为一个需要换行的段落。

![](https://img2023.cnblogs.com/blog/3150827/202303/3150827-20230329111740816-798299845.png)

 下面的代码实现了这个过程：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
1 # -*- coding: utf-8 -*-
 2 """
 3 Created on Tue Mar 28 21:41:39 2023
 5 @author: 
 6 """
 8 import pandas as pd
 9 from selenium import webdriver
10 import time
11 import numpy as np
12 from selenium.webdriver.common.by import By
14 files=pd.read_csv(r'人才政策爬取\北京市人才政策.csv',\
15                   encoding='utf-8',header=0)
16 files_link=files[['链接','标题']]
17 files_link=np.array(files_link)
18 files_link=files_link.tolist()
19 print(files_link)
20 browser=webdriver.Chrome()
21 time.sleep(3)
22 for link,title in files_link:
23     with open(r"人才政策爬取\北京市政策文本\\"+title+'.txt',"w") as f:
24         f.write(title+'\n') 
25         browser.get(link)
26         time.sleep(1)
27         out_time=browser.find_element_by_xpath("//*[contains(text(),'[发布日期]')]").text
28         f.write(out_time+'\n')
29         content=browser.find_element(By.ID,'mainText')
30         lines=content.find_elements_by_tag_name('p')
31         for line in lines:
32             f.write(line.text+'\n')
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")