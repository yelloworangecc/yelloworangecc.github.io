---
layout: post
title:  "QT4: UI编译器"
summary: UI编译器(uic)适用于Qt Widgets模块,它读取一个XML格式的用户界面定义文件(.ui)并创建一个对应的C++头文件.
featured-img: qt
# categories: jekyll update
---
## 用法 ##

UI编译器(uic)适用于Qt Widgets模块,它读取一个XML格式的用户界面定义文件(.ui)并创建一个对应的C++头文件.`.ui`文件由Qt Designer创建.Qt Widgets模块提供了用于创建经典桌面风格用户界面的UI元素集合.Qt还提供了Qt Quick和Qt webEngine用于不同场景下的UI开发.
uic的用法:`uic [options] <uifile>`
uic的命令行选项如下:

  * `-o <file>` 输出到文件而不是标准输出
  * `-tr <func>` 使用<func>代替`tr()`来转换字符串
  * `-p` 不产生防止重复引用头文件的#ifndef指令
  * `-h` 显示帮助信息
  * `-v` 电视uic的版本号
  * `-d` 显示UI的依赖
  * `-n` 不产生任何#include指令
  * `--postfix <postfix>` 为所有的类名添加后缀
  * `--include <file>` 添加#include指令到输出
  
## 例子 ##

如果使用qmake,uic会自动执行.如果只使用GNU的make,下面是一些有用的例子.

```
ui_%.h: %.ui
        uic $< -o $@
```

也可以在makefile中使用独立的规则:

```
ui_foo.h: foo.ui
        uic $< -o $@
```

记得必须将ui_foo.h加入到头文件变量中(HEADERS).
