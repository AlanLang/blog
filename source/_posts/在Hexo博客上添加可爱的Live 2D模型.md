---
title: 在Hexo博客上添加可爱的Live 2D模型
tags: 博客美化
categories: 知识点
abbrlink: 59303
date: 2018-10-10 14:24:34
---
## 第一步 安装Live2D
安装 [hexo-helper-live2d](https://github.com/EYHN/hexo-helper-live2d)
```
npm install --save hexo-helper-live2d
```
<!-- more -->

## 第二步 修改配置文件
在hexo的配置文件`_config.yml`中添加如下配置，详细配置可以参考项目文档：
```
live2d:
  enable: true
  scriptFrom: local
  pluginRootPath: live2dw/
  pluginJsPath: lib/
  pluginModelPath: assets/
  tagMode: false
  debug: false
  model:
    use: live2d-widget-model-hibiki
  display:
    position: left
    width: 100
    height: 210
  mobile:
    show: false
```
## 第三步 下载模型
下载模型，模型名称如下，一些模型的预览可以访问[这里](https://huaji8.top/post/live2d-plugin-2.0/)。
```
npm install live2d-widget-model-hibiki
```
所有模型列表如下：
* live2d-widget-model-chitose
* live2d-widget-model-epsilon2_1
* live2d-widget-model-gf
* live2d-widget-model-haru/01 (use npm install --save live2d-widget-model-haru)
* live2d-widget-model-haru/02 (use npm install --save live2d-widget-model-haru)
* live2d-widget-model-haruto
* live2d-widget-model-hibiki
* live2d-widget-model-hijiki
* live2d-widget-model-izumi
* live2d-widget-model-koharu
* live2d-widget-model-miku
* live2d-widget-model-ni-j
* live2d-widget-model-nico
* live2d-widget-model-nietzsche
* live2d-widget-model-nipsilon
* live2d-widget-model-nito
* live2d-widget-model-shizuku
* live2d-widget-model-tororo
* live2d-widget-model-tsumiki
* live2d-widget-model-unitychan
* live2d-widget-model-wanko
* live2d-widget-model-z16

## 第四步 配置模型
下载完之后，在Hexo根目录中新建文件夹live2d_models，然后在node_modules文件夹中找到刚刚下载的live2d模型，将其复制到live2d_models中，然后编辑配置文件中的model.use项，将其修改为live2d_models文件夹中的模型文件夹名称。
然后重新发布博客即可。