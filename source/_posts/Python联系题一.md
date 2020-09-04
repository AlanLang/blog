---
title: Pthon练习题(一)
tags: python
categories: Pthon练习题
abbrlink: '50947504'
date: 2018-01-02 11:00:23
---

2018年1月3日

Python 版本 ：3.6.3

[题目地址](https://github.com/AlanLang/show-me-the-code) | [参考地址](https://www.jianshu.com/p/05e3973a77ed) | [源码地址](https://github.com/AlanLang/show-me-the-code/tree/master/0000)
<!-- more -->
----
### 使用 Python图像处理库：Pillow
```
pip install pillow
```
### 新建文件demo.py
输入：
``` python
# 引入Pillow
from PIL import Image, ImageDraw, ImageFont, ImageColor
def add_num(img):
    # 创建一个Draw对象
    draw = ImageDraw.Draw(img)
    # 创建一个Fount
    myfont = ImageFont.truetype('C:/windows/fonts/Arial.ttf', size=40)
    fillcolor = ImageColor.colormap.get('red')
    width, height = img.size
    draw.text((width-30, 0), '4', font=myfont, fill=fillcolor)
    img.save('result.jpg', 'jpeg')
    return 0
if __name__ == '__main__':
    image = Image.open('test.jpg')
    add_num(image)
```
### 效果
![](http://oqdzx28cd.bkt.clouddn.com/18-1-3/6525123.jpg)

