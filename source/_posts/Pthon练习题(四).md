---
title: Pthon练习题(四)
tags: python
categories: Pthon练习题
abbrlink: 7b152142
date: 2018-02-01 11:00:23
---
## 题目
任一个英文的纯文本文件，统计其中的单词出现的个数。
<!-- more -->
## 编码
```Python
# -*- coding: utf-8 -*-
def count_words_byInput():
    temp = open("page.text")
    myStr = temp.read()
    print ("请输入你想要查找的单词：")
    while True:
        try:
            key_word = raw_input(">>")
            break
        except Exception:
            print ("输入有误，请重新输入：")
    num = myStr.upper().count(key_word.upper())
    print ("搜索结果：\n>>有%d个【%s】" % (num, key_word))
    temp.close()

if __name__ == '__main__':
    count_words_byInput()
```