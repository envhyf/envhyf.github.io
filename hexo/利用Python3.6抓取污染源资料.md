title: 简易的网络爬虫获取大气污染源资料
date: 2018-06-06 19:00:00
tags: 
- Python
- Air quality

---

此文以[四川省污染源监测信息公开平台](http://182.148.109.183)中数据资料的获取为例，介绍Python3.6版本的网络爬虫基本应用。

利用此方法，可获得该省份不同城市废气、废水等重点监控企业的基本资料(位置，主要污染物等)，用于研究分析

<!--more-->

## <font color="blue"> request_html库安装 </font> 

大多数网页中的内容是HTML静态生成的，从HTML源码中就能直接读取相信息(如chrome浏览器下按住“⌘-Option-U”即可检查源代码)。但部分网页的内容由JavaScript动态生成，在HTML源码中并不存在。例如，[目标网页1](http://182.148.109.184/gisnavigation!citysuriverPage.action?regioncode=510300) (如图1) 中左侧表格内容，并不直接存放在HTML源码中。

[![enter image description here][1]][1]


[1]: https://i.stack.imgur.com/T8Vyg.png

<center>图1. 污染源平台HTML源码示意</center> 

Python2下的BeautifulSoup工具并不能读取此类内容。搜索网络资源得知，JS动态生成的内容可利用[Selenium]对网页进行模拟访问获取，但我完全不会。Python的request_html库可实现这一功能，但目前只支持3.6版本。按照其官网的教程，开始安装。

```bash
#构建PYTHON3环境，系统默认为PYTHON 2.7，可能会导致之后安装失败
pipenv --three
# 安装request_html
pipenv install requests-html
```

## <font color="blue">获取企业网址列表</font>

我的思路是首先获取该平台中全部监控企业的网页列表，继而以相同方法进行目标内容的抓取。因而，此处的关键是获取图1表格中各企业的链接内容

```python
# coding：utf-8
import requests
import json

```

<时间有限，未完待续>