---
title: Python数据操作
tags:
  - dataframe
  - python
id: '299'
categories:
  - - Python
date: 2019-03-10 21:04:00
---

# 1\. 时间

## 1.1 日期转化

import time, datetime
import pandas as pd

# 今天
today = datetime.datetime.today().date()

# 时间格式化
date\_point\_formatted = date\_point.strftime('%Y%m%d')

# 字符串转化日期
date = '20190101'
date = datetime.datetime.strptime(str(date), '%Y%m%d').date()

# 各种结构转化为日期：integer, float, string, datetime, list, tuple, 1-d array, Series
date = pd.to\_datetime(str(date), format='%Y%m%d')

## 1.2 日期加减、比较

import datetime

today = datetime.datetime.today().date()

# 五年前
delta\_year = 5
start\_date = datetime.date(today.year - delta\_year, today.month, today.day)

# 七天前
day\_delta = datetime.timedelta(days=-7)

# 大小比较
today > start\_date

# 2\. 基本操作

## 2.1字符串

\# 转为字符串
price = 12
price = str(12)

# 转为int
price = int(price)

# 分割
text = "600690.SH"
tmp = text.split(".")
print(tmp\[1\]+tmp\[0\])

# 大小写
text = text.lower()
text = text.upper()

##  2.2 列表

 1. 遍历 元素
for item in list:
 
2. 遍历 索引, 元素
for index, item in enumerate(list):

## 2.3 HTTP请求设置代理

r = requests.post(url, params=para, headers=header, proxies={
    'http': 'http://127.0.0.1:1086',
    'https': 'https://127.0.0.1:1086'
})

# 3\. DataFrame

## 3.1 设置index

date\_index = pd.to\_datetime(df\['trade\_date'\], format='%Y%m%d')
df.set\_index(date\_index, inplace=True)
df = df.sort\_index(ascending=True)

##  3.2 去重

\# 去重
df = df.drop\_duplicates(subset='trade\_date', keep='first')

##  3.3 拼接

\# 拼接
df = df.append(old\_df, ignore\_index=True)

# merge on
df = pd.merge(income\_df, balancesheet\_df, on='end\_date')

# merge
merged\_df = pd.merge(stock\_price\_df, finance\_df, left\_index=True, right\_index=True, how='left')

##  3.4 遍历、删除

\# 按index值进行遍历，并删除大于起始时间的
for index in df.index:
  print(index)
  if index.date() > start\_date:
    df.drop(index=index)

# 按下标进行遍历
for index in range(0, len(pe\_df), 1):
  line = pe\_df.iloc\[index\]

# 直接遍历line
for index, line in df.iterrows():

##  3.5 获取column中最大、最小值

max\_line = data.loc\[data\['profitRate'\].idxmax()\]
min\_line = data.loc\[data\['profitRate'\].idxmin()\]

##  3.6 cell操作

\# 字符串化
df\['trade\_date'\] = df\['trade\_date'\].apply(str)

# 日志操作
df\['end\_date'\] = df\['end\_date'\].apply(lambda x: x\[2:6\])

# 单元格操作
df\['trade\_date'\] = df\['trade\_date'\].apply(lambda x: ajust\_date\_to\_quarter(x))

# 读取\[row, column\]的内容
df.at\[row, column\]

##  3.7 loc/iloc区别

\# 下标方式获取df段
data = pe\_df\[index: index + 480\]

# loc根据index获取，index\_value: 真实值，比如20190101、20190102
line = pe\_df.loc\[index\_value\]

# iloc根据下标获取，index: 0、1、2、3
line = pe\_df.iloc\[index\]

[![DataFrame Selection](https://www.zmannotes.com/wp-content/uploads/2019/03/Screen-Shot-2019-10-06-at-8.54.08-PM-1024x337.png)](https://www.zmannotes.com/wp-content/uploads/2019/03/Screen-Shot-2019-10-06-at-8.54.08-PM.png) [https://pandas.pydata.org/pandas-docs/stable/getting\_started/dsintro.html#indexing-selection](https://pandas.pydata.org/pandas-docs/stable/getting_started/dsintro.html#indexing-selection)

##  3.8 保存、加载

\# 写文件
df.to\_csv(file\_path, index=False)

# 读文件
df = pd.read\_csv(dir\_path+index\_code)

# 第一列作为index
df = pd.read\_csv(file\_path, index\_col=0)

## 3.9 日期操作

from pandas.tseries.offsets import \*
today = '20190101'
today = pd.Timestamp(today)
# 本季度末
this\_quarter = (today - QuarterEnd(n=0))
print(this\_quarter.strftime("%Y%m%d"))
# 上一个季度末
last\_quarter = (today - QuarterEnd(n=1))
print(last\_quarter.strftime("%Y%m%d"))

## 3.9 其它

df = df\[\['total\_revenue', 'total\_profit', 'n\_income'\]\]
df = df.fillna(value=0)
df = df.round(2)
# 重命名
df = df.rename(columns={'n\_income': '净利润', 'total\_profit': '总利润'})

# 插入
finance\_df.insert(0, '类目', finance\_df.index)

# 所有column名称
finance\_df.columns.values.tolist()


# group
df\_max = df.groupby('trade\_date').max().rename(columns={'close': 'max'})

# 转置
finance\_df = finance\_df.transpose()

# 是否存在某个row
result = row\_name in analysed\_stocks.index

# 4\. 异常

## 4.1 异常栈

import traceback

try:
  
except Exception as e:
  traceback.print\_exc()