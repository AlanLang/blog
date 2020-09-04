---
title: Pthon练习题(二)
tags: python
categories: Pthon练习题
abbrlink: ffa5ee66
date: 2018-01-22 11:00:23
---

### 题目
做为 Apple Store App 独立开发者，你要搞限时促销，为你的应用生成激活码（或者优惠券），使用 Python 如何生成 200 个激活码（或者优惠券）
<!-- more -->
### UUID简介
UUID是128位的全局唯一标识符，通常由32字节的字符串表示。它可以保证时间和空间的唯一性。
*   uuid1:基于时间戳
*   uuid2:基于分布式计算环境DCE（python中没有此函数）
*   uuid3:基于名字的MD5散列值
*   uuid4:基于随机数
*   uuid5:基于名字的SHA-1散列值
### 代码
```Python
import uuid

def generateCode(num):
    list = []
    for i in range(num):
        list.append(uuid.uuid1())
    return list

if __name__ == "__main__":
    codes = generateCode(200)
    code_file = open('gencodes.txt', 'w')
    for code in codes:
        code_file.write(str(code) + "\n")
    code_file.close()
```
### 效果
![](http://oqdzx28cd.bkt.clouddn.com/18-1-8/697127.jpg)