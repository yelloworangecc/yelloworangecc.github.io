---
layout: post
title:  "Python爬取BDI指数"
summary: "利用Python从http://value500.com/BDI.asp页面爬取一周BDI指数"
featured-img: python
# comments: true
---

BDI是波罗的海干散货指数（Baltic Dry Index）的简称，它是由几条主要航线的即期运费（Spot Rate）加权计算而成，为即期市场的行情的反映，因此，运费价格的高低会影响到指数的涨跌。波罗的海综合指数是散装船航运运价指标，而散装船运以运输钢材、纸浆、谷物、煤、矿砂、磷矿石、铝矾土等民生物资及工业原料为主。由于散装航运业营运状况与全球经济景气荣枯、原物料行情高低息息相关。因此波罗的海指数被认为是国际间贸易情况的领先指数及经济晴雨表。

## beautifulsoup安装 ##

```
pip install beautifulsoup4
```

## 目标页面 ##
![目标页面](/pics/target_page "目标页面")

## 代码分析 ##
全部代码如下.
```
import io
import sys
import urllib
from bs4 import BeautifulSoup

sys.stdout = io.TextIOWrapper(sys.stdout.buffer,encoding='utf8')
#file_obj=open("bdi.html","r",encoding="UTF-8")
content=urllib.request.urlopen("http://value500.com/BDI.asp")
soup = BeautifulSoup(content,features="html5lib")
unicode_result = str(soup.body.contents[1].contents[7].tbody.tr.contents[3].table.tbody.tr.td.contents[7].table.tbody)
print(unicode_result)
```
由于是windows平台,python默认输出gbk编码,但本人使用的msys2控制台默认输出utf-8,如果需要打印中文内容到控制台进行调试,需要将python输出编码改为utf-8.
```
sys.stdout = io.TextIOWrapper(sys.stdout.buffer,encoding='utf8')
```
使用urllib获取html页面全部内容.
```
content=urllib.request.urlopen("http://value500.com/BDI.asp")
```
使用html5lib解析器生成beautiful soup对象.
```
soup = BeautifulSoup(content,features="html5lib")
```
定位到目标标签内容.并将beautiful soup的navigable string 转换成普通unicode字符串.
* beautiful soup可以通过".+标签的名称"的方式访问标签的内容,例如soup.body可以访问`<body>...</body>`标签内的所有内容.
* 存在多个同名子标签时,需要使用标签的.contents属性,这个属性可以将标签的子节点以列表的方式输出.
```
unicode_result = str(soup.body.contents[1].contents[7].tbody.tr.contents[3].table.tbody.tr.td.contents[7].table.tbody)
```
输出结果.
```
print(unicode_result)
```
控制台打印如下内容
```
      <tbody><tr align="left" height="20">
                <td style="padding-left:10px;" width="33%"><b>日　期</b></td>
                <td style="padding-left:10px;" width="33%"><b>BDI指数</b></td>
                <td style="padding-left:10px;" width="33%"><b>涨跌幅</b></td>
              </tr>
              <tr align="left" height="20">
                <td style="padding-left:10px;">2021年1月18日</td>
                <td style="padding-left:10px;">1740.00</td>
                <td style="padding-left:10px;">-0.80%</td>
              </tr>
              <tr align="left" height="20">
                <td style="padding-left:10px;">2021年1月15日</td>
                <td style="padding-left:10px;">1754.00</td>
                <td style="padding-left:10px;">-2.12%</td>
              </tr>
              <tr align="left" height="20">
                <td style="padding-left:10px;">2021年1月14日</td>
                <td style="padding-left:10px;">1792.00</td>
                <td style="padding-left:10px;">-3.45%</td>
              </tr>
              <tr align="left" height="20">
                <td style="padding-left:10px;">2021年1月13日</td>
                <td style="padding-left:10px;">1856.00</td>
                <td style="padding-left:10px;">0.38%</td>
              </tr>
              <tr align="left" height="20">
                <td style="padding-left:10px;">2021年1月12日</td>
                <td style="padding-left:10px;">1849.00</td>
                <td style="padding-left:10px;">5.00%</td>
              </tr>
            </tbody>
```

## 嵌入爬取内容 ##
![运行结果](/pics/beautifulsoup_result.jpg "运行结果")
