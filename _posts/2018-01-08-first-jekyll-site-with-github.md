---
layout: post
title:  "搭建基于jekyll和github pages的个人站点"
date:   2018-01-08 14:07:00 +0800
categories: jekyll update
---

# 搭建基于jekyll和github pages的个人站点 #
## 新建站点 ##
  * 在github pages上新建和clone你的新站点,例如我的站点为TurnipBun.github.io
  * 初始化jekyll站点:`jekyll new --force TurnipBun.github.io`
  * 修改.gitignore文件,排除jekyll和系统生成的临时文件和目录,例如:
```
_drafts/
_site/
.DS_Store
*.swp
.sass-cache
.jekyll-metadata
```
   * 在`_config.yml`中配置站点基本信息,例如:
```
# Site settings
# These are used to personalize your new site. If you look in the HTML files,
# you will see them accessed via {{ site.title }}, {{ site.email }}, and so on.
# You can create any custom variable you would like, and they will be accessible
# in the templates via {{ site.myvariable }}.
title: 黄大橙yellow的博客
email: turnipbun@icloud.com
description: >- # this means to ignore newlines until "baseurl:"
  分享程序员黄大橙yellow本人的成长历程.内容包括技术博客,生活感悟,还包括热点事件,软件,游戏,书,影等评论.
baseurl: "" # the subpath of your site, e.g. /blog
url: "" # the base hostname & protocol for your site, e.g. http://example.com
github_username:  turnipbun
```
  * 使用git命令提交代码
  * 在浏览中输入站点名称,例如TurnipBun.github.io,可以看到如下效果:
![新站点效果图](/pics/new-jekyll-site-example.jpeg)
## 第一篇博客 ##
### YAML头 ###
jekyll空站点中有一篇博客文章的例子,位于_posts目录下,名为2018-01-05-welcome-to-jekyll.markdown,仿照该范例书写自己的博客文章即可.博客文章可以用markdown格式写,但是必须包含YAML头信息,这样才能被jekyll当做一个特殊的文件来处理。头信息必须在文件的开始部分，并且需要按照YAML的格式写在两行三虚线之间.下面是示例博客文章的YAML头信息.
```
---
layout: post
title:  "Welcome to Jekyll!"
date:   2018-01-05 15:36:03 +0800
categories: jekyll update
---
```
### 引用图片 ###
  * 在站点根目录新建pics目录
  * 拷贝博客中需要引用的图片到pics
  * 在博客中插入引用图片的markdown代码,例如`![网站显示效果]({{ site.url }}/pics/new-jekyll-site-example.jpeg)`
### 重命名博客 ###
在_post目录放置的博文必须按照`日期-名称.后缀`的格式来命令,例如`2018-01-08-first-jekyll-site-with-github.md`
### 预览博客 ###
  * 运行`jekyll serve`,这个命令会自动生成站点并启动web server
  * 使用浏览器访问127.0.0.1:4000
  * 点击博客链接进入查看
