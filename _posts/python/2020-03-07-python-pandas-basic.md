---
layout: post
title:  "Pandas表格处理实例"
summary: "利用python的pandas对集团公司下属各公司新冠疫情上报数据进行统计,提高数据统计效率,数据表格来源于腾讯文档."
featured-img: python
# comments: true
---

由于疫情影响,集团公司要求员工每日上报身体和接触情况,员工在微信通过腾讯文档填写上报信息,最终在腾讯文档后台汇总成一张表格.对于表格的处理和统计,利用python的pandas对集团公司下属各公司新冠疫情上报数据进行统计,提高数据统计效率.

## pandas安装 ##

正常可以通过pip安装,本人使用windows平台的msys2环境,通过pacman包管理工具安装.
`pacman -S mingw-w64-x86_64-python-pandas`

## 数据表格样例 ##

![表格样例](excel_example.png "表格样例")

## 代码分析 ##

```
import numpy as np
import pandas as pd
import io
import sys
import os

#sys.stdout = io.TextIOWrapper(sys.stdout.buffer,encoding='utf8')
companies=['博创','财务公司','国华','国贸实业','国绵','国盛','海外','汉帛','华博','华诚','华盛','华泰','华通公司','慧贸通','集团总部','景云房产','力天','其他','实业','物业','亿达','亿盛','紫金','华荣化工','超威','上海进出口']
df = pd.read_excel('data.xlsx', 'Sheet1')
df = df.drop_duplicates(subset=['提交人','姓名','手机号码'],keep = 'first')
print(len(df))
for company in companies:
  os.system('rm ' + company +'*')
  index = company
  if company != '其他':
    companydf = df[df['公司'] == index]
  else :
    companydf = df[df['公司'].isnull()]
  companydf.to_excel(company + str(len(companydf)) + '.xlsx', sheet_name = 'Sheet1')
```

```
#sys.stdout = io.TextIOWrapper(sys.stdout.buffer,encoding='utf8')
```
由于是windows平台,python默认输出gbk编码,但msys2控制台默认输出utf-8,如果需要打印中文内容,需要将python输出编码改为utf-8.

```
companies=['博创','财务公司','国华','国贸实业','国绵','国盛','海外','汉帛','华博','华诚','华盛','华泰','华通公司','慧贸通','集团总部','景云房产','力天','其他','实业','物业','亿达','亿盛','紫金','华荣化工','超威','上海进出口']
```
预先定义好要统计的公司名称.

```
df = pd.read_excel('data.xlsx', 'Sheet1')
```
读取表格.

```
df = df.drop_duplicates(subset=['提交人','姓名','手机号码'],keep = 'first')
```
去除重复数据.

```
print(len(df))
```
输出数据总数.

```
for company in companies:
```
开始循环统计各公司的数据.

```
  os.system('rm ' + company +'*')
```
删除已存在的各公司的表格.

```
  if company != '其他':
    companydf = df[df['公司'] == company]
  else :
    companydf = df[df['公司'].isnull()]
```
筛选出各公司的数据,其中公司字段未填的数据归为"其他".

```
  companydf.to_excel(company + str(len(companydf)) + '.xlsx', sheet_name = 'Sheet1')
```
将各公司的数据写入新表格,并在文件名显示数据条数.

## 运行结果 ##

![运行结果1](excel_resualt1.png "运行结果1")
![运行结果2](excel_resualt2.png "运行结果2")
