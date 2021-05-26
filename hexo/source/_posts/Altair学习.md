title: '可视化工具Altair学习'
date: 2020-05-10 10:00:00

categories:
- Tech

tags:
- Python
- Visulaization
- Web
---
提要：Altair是强大的可视化库，其基于Vega-lite,可快速生成简洁、美观、可互动的统计图形。本文介绍个人相关学习经历，具体包括：(1)数据载入与基本图形绘制; (2)基于web端呈现
<!--more-->

### <font color="blue"> 背景知识介绍</font>   
常用的数据可视化工具包，例如R语言下的ggplot2和Python语言下的matplotlib，均难以制作交互式图表。不同于传统的静态图表，交互式图表允许研究者及读者可以探索甚至操纵原始数据，便于我们理解数据关系，发现其隐藏规律。
Python语言下可交互的可视化工具包括[Altair](https://altair-viz.github.io/), [Bokeh](https://docs.bokeh.org/en/latest/index.html)及[plotly](https://plotly.com/dash/)等多种。

MatPlotlib的优势：
1. Designed like MatLab: switching was easy
2. Many rendering backends
3. well-tested, standard tool for 15 years
4. can reproduce just about any plot with a bit of effort

MatPlotlib的劣势
1. API is imperative & often overly verbose
2. Poor/no support for interactive/web graphs
存在问题: we are mixing the __what__ with the __how__

[![enter image description here][1]][1]

  [1]: https://i.stack.imgur.com/8ajtS.png

在应用MatPlotLib的过程中，不仅需要考虑画什么，还需要考虑如何实现。坐标轴等亦需要自行设置，每次都在等微调中耗费大量时间。
实际上，应当将更多的时间用于数据的分析及写作中，出图效率应提高。
因此，我决定全面转向altair作为未来个人的主要可视化工具，在日常工作中频繁使用，不断积累。

```bash
## 安装
## https://raw.githubusercontent.com/vega/vega/master/docs/examples/bar-chart.vg.json
pip install altair vega_datasets
```



### <font color="blue"> Web端数据展示</font>   
先存成json文件，再利用vega embed 实现。其基本格式总结如下：
```html
{% raw %}
<html>
<head>
  <script src="https://cdn.jsdelivr.net/npm/vega@5"></script>
  <script src="https://cdn.jsdelivr.net/npm/vega-lite@4"></script>
  <script src="https://cdn.jsdelivr.net/npm/vega-embed@6"></script>
</head>
<body>
  <div id="vis"></div>
  <script type="text/javascript">
  # spec中载入altar对象生成的json文件
  var spec = "/uploads/vconcat.json";
  var opt = {"renderer": "canvas", "actions": false};
  vegaEmbed("#vis", spec, opt);
  </script>
</body>
</html>
{% endraw %}  
```

此处展示基本例子
{% raw %}
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdn.jsdelivr.net/npm/vega@5"></script>
  <script src="https://cdn.jsdelivr.net/npm/vega-lite@4"></script>
  <script src="https://cdn.jsdelivr.net/npm/vega-embed@6"></script>
</head>
<body>
  <div id="vis1"></div>
  <script type="text/javascript">
  var spec1 = "/uploads/vconcat.json";
  var opt1 = {"renderer": "canvas", "actions": false};
  vegaEmbed("#vis1", spec1, opt1);
  </script>
</body>
</html>
{% endraw %}  

在同一个静态网站中需要展示多个interactive plots, 基本方法如下：
修改上述文件中3个变量, vis, spec, opt
{% raw %}
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdn.jsdelivr.net/npm/vega@5"></script>
  <script src="https://cdn.jsdelivr.net/npm/vega-lite@4"></script>
  <script src="https://cdn.jsdelivr.net/npm/vega-embed@6"></script>
</head>
<body>
  <div id="vis2"></div>
  <script type="text/javascript">
  var spec2 = "/uploads/cc.json";
  var opt2 = {"renderer": "canvas", "actions": false};
  vegaEmbed("#vis2", spec2, opt2);
  </script>
</body>
</html>
{% endraw %}  


{% raw %}
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdn.jsdelivr.net/npm/vega@5"></script>
  <script src="https://cdn.jsdelivr.net/npm/vega-lite@4"></script>
  <script src="https://cdn.jsdelivr.net/npm/vega-embed@6"></script>
</head>
<body>
  <div id="vis3"></div>
  <script type="text/javascript">
  var spec3 = "/uploads/car-clickable-legend.json";
  var opt3 = {"renderer": "canvas", "actions": false};
  vegaEmbed("#vis3", spec3, opt3);
  </script>
</body>
</html>
{% endraw %}  


### <font color="blue"> 常见问题</font>   
1. 数据集长度过大 (The number of rows in your dataset is greater than the maximum allowed (5000).)
官方回答摘录如下
> This is not because Altair cannot handle larger datasets, but it is because it is important for the user to think carefully about how large datasets are handled. As noted above in Why does Altair lead to such extremely large notebooks?, it is quite easy to end up with very large notebooks if you make many visualizations of a large dataset, and this error is a way of preventing that.
2.上标、下标应当如何展示
altair不支持latex符号的显示，因此需要寻找各数学符号的unicode字符。常用字符整理如下：
* 微克/米三次方：'\u03BC'+'g/m'+'\u00B3'



### 参考文献
1. [PYCON 2018](https://youtu.be/ms29ZPUKxbU)
2. [D3 in hexo](https://zhuzilin.github.io/D3-in-Hexo/)
3. [嵌入bokeh绘图至hexo博客中](https://alanlee.fun/2018/03/15/embed-bokeh-plot/)
4. [The reason I am using Altair for most of my visualization in Python](http://fernandoi.cl/blog/posts/altair/)
5. [如何插入多个图片](https://matthewkudija.com/blog/2018/06/22/altair-interactive/)
6. [vega-embed](https://github.com/vega/vega-embed)
7. [vega-lite](https://vega.github.io/vega-lite/)
8. [How to find the unicode of the subscript alphabet?](https://stackoverflow.com/questions/17908593/how-to-find-the-unicode-of-the-subscript-alphabet)
