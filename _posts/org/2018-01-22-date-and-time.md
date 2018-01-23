---
layout: post
title:  "Emacs Org模式系列学习笔记:日期和时间"
date:   2018-01-22 19:50:07 +0800
categories: jekyll update
---

# 日期和时间 #

{% include org-introduction.md %}

## 时间戳 ##

emacs的org模式可以为TODO项目打上时间和日期标签,这个标签也叫"时间戳".

  * 时间戳是一个特定格式的日期,时间戳可以出现在标题或者文本主体的任意位置,下面是利用时间戳管理日程表的一个例子.
![org时间戳的例子](/pics/org-date-time-example.jpeg)
  * 时间戳类型1:普通时间戳,时间,会议.
  
```
* Meet Peter at the movies
  <2006-11-01 Wed 19:15>
* Discussion on climate change
  <2006-11-02 Thu 20:00-22:00>
```

  * 时间戳类型2:周期重复

```
* Pick up Sam at school
  <2007-05-16 Wed 12:30 +1w>
```

  * 时间戳类型3:sexp diary格式

```
* 22:00-23:00 The nerd meeting on every 2nd Thursday of the month
  <%%(diary-float t 4 2)>
```

  * 时间和日期范围

```
* Meeting in Amsterdam
  <2004-008-23 Mon>--<2004-08-26 Thu>
```
  
  * 非活跃时间戳,与普通时间戳相同但不会触发org模式的agenda

```
* Gillian comes late for the fifth time
  [2006-11-01 Wed]
```

## 创建时间戳 ##

  * C-c .                 插入日期,连续使用两次时插入日期范围
  * C-c !                 插入非活跃时间戳
  * C-u C-c . / C-u C-c ! 插入包含时间和日期的时间戳,时间最小单位为5分钟
  * C-c C-c               规范化时间戳
  * C-c <                 插入与日历一致的时间戳
  * C-c >                 通过时间戳访问日历
  * C-c C-o               访问给出日期或日期范围的agenda
  * S-left / S-right      以天为单位改变当前时间戳
  * S-up / S-down         改变时间戳的光标所在部分,包括年,月,日,时,分,当遇到时间范围时,改变第一个时间也会改变第二个时间,保持长度不变,改变第二个时间则不会改变第一个时间.该快捷键存在冲突情况.
  * C-c C-y               在时间范围之后计算和插入时间长度

### 日期和时间输入框 ###

在插入时间戳时,光标会移动到Emacs底部的mini buffer,并且弹出日历(效果参见前图).此时mini buffer可以输入各种格式的日期和时间信息,org模式可以根据这些的信息推断出时间戳.org优先使用离当前时间最近的将来时间.如果今天是2006-06-13,一些输入的例子和推断结果如下.

```
3-2-5        => 2003-02-05
2/5/3        => 2003-02-05
14           => 2006-06-14
12           => 2006-07-12
2/5          => 2007-02-05
Fri          => nearest Friday after the default date
sep 15       => 2006-09-15
feb 15       => 2007-02-15
sep 12 9     => 2009-09-12
12:45        => 2006-06-13
22 sept 0:34 => 2006-09-22 00:34
w4           => ISO week four of the current year 2006
2012 w4 fri  => Friday of ISO week 4 in 2012
2012-w04-5   => same as above
```

除此以外还可以使用相对时间对上面输入的时间进行补充.

```
+0    => today
.     => today
+4d   => four days from today
+4    => same as above
+2w   => tow weeks from today
++5   => five days from default date
+2tue => second Tuesday from now
-wed  => last Wednesday
```

下面的例子可以指定时间范围.

```
11am-1:15pm  => 11:00-13:15
11am--1:15pm => same as above
11am+2:15    => same as above
```

与输入框一同显示的还有日历,日历中所选日期将结合输入内容推断出时间戳,在日历buffer中可以进行如下操作.

```
RET               选择光标当前的日期
mouse-l           鼠标单击选择日期
S-right/left      以天为单位移动
S-down/up         以周为单位移动
M-S-right/left    以月为单位移动
> / <             滚动日历一个月
M-v / C-v         滚动日历三个月
M-S-down/up       滚动日历一年
```

### 配置时间格式

默认使用ISO标准格式,可使用`org-display-custom-times`和`org-time-stamp-custom-formats`进行配置,使用`C-c C-x C-t`触发变更.

## 截止日期和调度日期

时间戳可以结合关键字来方便做计划.这是关键字和时间戳应该紧跟在任务后面.
  * DEADLINE 任务计划完成时间,任务会在agenda中列出,当接近或者超过截止日期时agenda会发出警告,直到任务标记为DONE,下面的-5d是设置提醒时间.

```
*** TODO write atcicle about the earth for the guide
    DEADLINE:<2004-02-29 Sun -5d>
    the editor in charge is [[bbdb:ford prefect]]
```

  * SCHEDULED 开始执行任务的时间,org模式会不断提醒你任务已经开始的天数直到把它标记为DONE,下面的-2d是设置推迟时间.对于周期性任务,使用--2d只对第一次有效.

```
*** TODO call trillian for a date on new years eve.
    SCHEDULED: <2004-12-25 Sat -2d>
```

### 插入截止日期和调度日期

  * C-c C-d 插入DEADLINE和时间戳.
  * C-c C-s 插入SCHEDULED和时间戳.
  * C-c / d 创建一个稀疏的树,显示过期和即将过期的截止时间.
  * C-c / b 创建一个稀疏的树,显示特定日期前的截止时间的调度时间项.
  * C-c / a 创建一个稀疏的树,显示特定日期后的截止时间和调度时间项.

### 重复任务

可以在DEADLINE,SCHEDULED和普通时间戳中定义重复,使用y/w/m/d指定按年月日小时重复任务,也可以与告警时间一同使用.

```
** TODO pay the rent
   DEADLINE:<2005-10-01 Sat +1m -3d>
```

当重复任务设置为DONE时,基准时间会更新并且重新将任务设置为TODO.
对于未完成的任务不需要累计处理时,使用++代替+.

```
** TODO Call Father
   DEADLINE: <2008-02-10 Sun ++1w>
```

此时你如果连续超期3次的话,只要完成一次即可继续下一轮任务了.

## 记录工作时间

org模式允许记录你花在项目特定任务上的时间.并且支持记录任务树总体花费时间和历史任务花费时间.如果要开启开记录历史任务花费时间,在emacs中保存如下配置.

```
(setq org-clock-persist 'history)
(org-clock-persistence-insinuate)
```

### 时钟命令

  * C-c C-x C-i 在当前项目开启时钟.会插入CLOCK关键字和一个时间戳.
  * C-c C-x C-o 关闭时钟.会在时钟开始的地方插入另一个时间戳,并且计算时间范围.

```
*** DONE 完善文档
    CLOCK: [2018-01-23 Tue 08:12]--[2018-01-23 Tue 09:01] =>  0:49
```

  * C-c C-x C-x 重新开启已经关闭的时钟.
  * C-c C-x C-e 更新当前时钟任务的工作量.
  * C-c C-c / C-c C-y 在改变时间戳后重新计算时间间隔.
  * C-S-up/down 增减两个成对时间戳,保持时长不变.
  * S-M-up/down 同时增减相邻单不成对的两个时间戳相同长度.
  * C-c C-t 改变TODO状态为DONE,并停止计时.
  * C-c C-x C-q 取消当前时钟.
  * C-c C-x C-j 跳转到正在计时的标题项.
  * C-c C-x C-d 显示每个子树的时间摘要.

### 时钟表格

基于计时机制,org模式能产生相当复杂的报告.这份报告就叫做时钟表格,它采用了org的表格来呈现.
  * C-c C-x C-r 在当前文件插入包含时钟表格的动态块,如果光标在时钟表格内,那更新这个表格.插入的例子如下.

```
#+BEGIN: clocktable :maxlevel 2 :scope subtree
#+CAPTION: Clock summary at [2018-01-19 Fri 16:03]
| Headline     | Time   |
|--------------+--------|
| *Total time* | *0:00* |
#+END:
```

  * C-c C-c / C-c C-x C-u 更新动态块,光标需要出现在动态块的`#+BEGIN`行中.
  * C-u C-c C-x C-u 更新文件中所有动态块.
  * S-left / S-right 时钟表格的`BEGIN`行可以指定一些用来定义范围,结构和格式的选项.这些选项的默认值可以在`org-clocktable-defaults`配置.选项的介绍如下.
  * :maxlevel 表格中所列时间的最大深度.深层次的时间记录会累加到高层次.
  * :scope 范围可以是:
    + nil 当前buffer或选择区域.
	+ file 整个文件buffer.
	+ subtree 时钟表所在的所有子树.
	+ treeN 周边层级为N的树.
	+ tree 周边层级为1的树.
	+ agenda 所有agenda文件.
	+ ("file"..) 扫描这些文件.
	+ file-with-archives 当前文件和它的关联文件.
	+ agenda-with-archives 所有agenda文件和关联文件.
  * :block 时间块,有相对时间和绝对时间以及各种格式相关设置.
  * :tstart 指定纳入摘要考虑范围的开始时间.
  * :tend 指定纳入摘要考虑范围的结束时间.
  * :wstart 一周的开始日期,默认1代表星期一.
  * :mstart 一个月的开始时间,默认1代表第一天.
  * :step 周或天来分割表格.
  * :stepskip0 不要显示值为0的步长时间.
  * :fileskip0 不要显示没有贡献的项目.
  * :tags 所选条目对应的标签.
  * :emphasize 为t时,加粗显示级别1和级别2的项目.
  * :lang 使用的语言.
  * :link 链接标题项到表格的区域.
  * :narrow 限制标题在表格中的长度.
  * :indent 根据级别缩进缩进标题项.
  * :tcolumns 表示时间是占用的列数.
  * :level 指定是否包含表示级别的数字.
  * :sort 指定列以某种方式排序.
  * :compact `:level nil :indent t :narrow 40! :tcolumns 1`的缩写形式
  * :timestamp 条目的时间戳
  * :properties 要显示在表格中的属性列表.
  * :inherit-props 设置为t时,`properties`的值由继承得到.
  * :formula 添加`#+TBLFM`行
  * :formater 格式化时间数据并插入到buffer的函数.

要显示当天,级别为1的时间摘要可以这样写.

```
#+BEGIN: clocktable :maxlevel 2 :block today :scope tree1 :link t
#+END: clocktable
```

要显示某时间阶段的摘要可以这样写.

```
#+BEGIN: clocktable :tstart "<2006-08-10 Thu 10:00>"
                    :tend "<2006-08-10 Thu 12:00>"
#+END: clocktable
```

或者这样写.

```
#+BEGIN: clocktable :tstart "<-1w>"                                                     
                    :tend "<now>"
#+END: clocktable
```

要显示当前子树的摘要,时间表示为%形式,可以这样写.

```
#+BEGIN: clocktable :scope subtree :link t :formula %
#+END: clocktable
```

要水平紧凑地显示上周所有时间记录可以这样写.

```
#+BEGIN: clocktable :scope agenda :block lastweek :compat t
#+END: clocktable
```

### 空闲时间 ###

当你开启计时器后需要离开,你需要从计时器中剥离这段时间.当你设置了`org-clock-idle-time`为一个整数时,当你离开电脑超过这个值指定的时间,emacs会在你回到电脑前时提醒你,并询问你要如何处理空闲时间,有如下选项.
  * k 保留部分和全部时间到计时器.
  * K 保留全部时间到计时器并停止计时.
  * s 丢弃所有时间并回到你离开时的状态.
  * S 丢弃所有时间并停止计时.
  * C 取消整个计时器.

### 继续计时 ###

你有时需要从上一个停掉的计时器开始计时,这时你需要把`org-clock-continuously`的值设置为`t`.当你上机时会接着上次下机的时间继续计时.当你知识偶尔需要这个功能时可以在`org-clock-in`命令前加3个前缀命令,或者在`org-clock-in-last`前加2个前缀命令.
  
## 工作量估算 ##

如果你想更精细地安排你的工作,你需要提供关于工作量的数据,你可能需要给任务项目预估工作量.如果你在工作时记录了任务时间,你很可能会把实际完成时间和估算时间进行比较,这是一种改善计划估算的有效方法.工作量估算存储在一个叫EFFORT的属性中.相关命令如下.
  * C-c C-x e 为当前项目设置工作量估算值.
  * C-c C-x C-e 更改当前项目的工作量估算值.

```
*** TASK A
    :PROPERTIES:
	:Effort:   5
	:END:
```

## 记笔记用定时器

org模式提供了两种类型的计时器,其中一种累加的相对计时器在会议和观看视频时对记笔记非常有用.该计时器也可以设置为递减的计时器.相关命令如下.
  * C-c C-x 0 开启或者重置相对计时器器,初始值为0.
  * C-c C-x ; 开启递减的计时器.
  * C-c C-x . 插入当前相对或递减计时器的值到当前buffer.
  * C-c C-x - 插入当前相对或递减计时器的描述列表项到当前buffer,下面是一个例子.

```
  - 1:39:21 :: 描述内容写在这里
```

  * M-RET 当上面命令的计时器列表启动时,该命令可以插入新的计时器项目.
  * C-c C-x , 暂停或者继续计时器.
  * C-c C-x _ 终止计时器.
