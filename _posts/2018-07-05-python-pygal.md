---
layout: post
title: 使用pygal画图
categories: Python
description: pygal画图
keywords: python, pygal, 画图
---

今天需要用python画一些数据比较的图，上网对比完python的图像库后决定选用pygal（考虑到 上手的难易度，和我对图形暂时没有复杂的需求）  
[pygal文档](http://www.pygal.org/en/latest/documentation/index.html)

python安装 
```shell
pip install pygal    
```
基本使用方式  

```python
    bar = pygal.Bar()
    bar.title = "bar测试"
    bar.x_labels = ["1", "2"]
    bar.add("webp", [20, 30])
    bar.add("jpg", [20, 30])
    bar.render_to_file(path)
```

生成的图片是 svg格式的，可以通过bar.render_in_browser() 在浏览器预览.  
如果需要png图片 可以使用 bar.render_to_png(path)  

输出为png图片 需要安装 cairosvg 库 依然通过pip安装   

```
pip install cairosvg
```

运行时可能出错：  
<pre>
OSError: dlopen() failed to load a library: cairo / cairo-2 / cairo-gobject-2

</pre>
是因为没有安装cairo， mac 下可以通过brew install cairo安装， windows下请自行百度cairo安装  

在使用的时候，遇到一个问题，中文无法显示在png图片上，此时需要特殊指明字体  

```python
import pygal
from pygal.style import Style


if __name__ == '__main__':
	 style = Style(font_family="Microsoft YaHei")
         bar = pygal.Bar(style=style)
	 bar.title = "测试中文"
	 bar.add("测试1", [1, 2])
	 bar.add("测试2", [3, 4])
	 bar.render_in_browser()
```  

通过制定 pygal.Radar(fill=True) 可以控制图形填充  
pygal.Radar(print_values=True)显示具体数值  

其它控制属性见文章开头的官方文档
