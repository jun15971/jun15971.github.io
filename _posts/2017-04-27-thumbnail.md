---
layout: post
title: 自动化生成文章缩略图
description:
date: 2017-04-27 20:41:32 +0800
image: assets/images/thumbnail/PIL.png
---


在自己建立的网站上写了那么多文章，现在最大的问题就是架设在`Git Pages`上，国内访问不稳定，有时候需要加载很久。
之前写了`PILLOW`库的笔记，正好可以现学现用优化一下文章的缩略图，变相加快访问网站速度。

主要的优化思路如下：
1. 网站之前上传的文章缩略图都是原图，在下载的时候会下载原图然后再转成宽度为`200px`的图像来显示。
现在直接跳过这个步骤，上传图片的时候直接使用宽为`200px`的缩略图，优化逻辑。
2. 图片自动化批量修改分辨率使用`PILLOW`库的`thumbnail(size)`方法，其中需要输入参数`size`。
3. `size`参数是一个二元元组：（缩略图的宽，缩略图的高）。其中单位为像素`px`。
缩略图的宽度统一为`200`，需要根据原图的尺寸求出所需的高度。
4. 假设所求高度为`x`,则根据公式:<img src="http://latex.codecogs.com/gif.latex?\bg_white&space;\frac{200}{x}&space;=&space;\frac{thumbnail\_width}{thumbnail\_hight}" title="\frac{200}{x} = \frac{thumbnail\_width}{thumbnail\_hight}" /> 可求得所需的值。

如下：
```python
#!usr/bin/env python3
# Filename: auto_thumbnail.py


import os
import sys
from PIL import Image

"""
该程序直接将同一文件夹下的图像读取并制作相应的缩略图。
程序逻辑：
遍历当前文件夹下所有文件（不打开文件夹）；
将图像文件按原图比例保存为指定宽度的PNG格式文件。
"""
# -----------------------------------
#---------全局变量--------------------
thumbnail_width = 200  # 指定宽度
#------------------------------------

cwd = os.getcwd()
file_list = os.listdir(cwd)  # 列出当前文件夹下所有的目录与文件
for i in range(len(file_list)):
    current_file = file_list[i]
    filename_without_extension0 = current_file.split('.')
    filename_without_extension = '.'.join(filename_without_extension0[:-1])
    try:
        im = Image.open(current_file)
        print('图像的格式为：' + im.format)
        original_size = (im.size[0], im.size[1])
        thumbnail_hight = im.size[1]*thumbnail_width/im.size[0]
        thumbnail_size = (thumbnail_width, thumbnail_hight)
        im.thumbnail(thumbnail_size)
        im.save(filename_without_extension + '.png', "PNG")
    except IOError:
        print("cannot create thumbnail for", current_file)
```
