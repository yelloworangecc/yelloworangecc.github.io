---
layout: post
title:  "QT4: qmake工程文件简介"
summary: 工程文件包含了给qmake使用的,用于构建应用程序,库和插件的所有信息.
featured-img: sleek
# categories: jekyll update
---
## 工程文件元素 ##

工程文件包含了给qmake使用的,用于构建应用程序,库和插件的所有信息.工程用到的资源一般以一系列的声明来指定,对简单编程结构的支持允许描述在不同平台和环境下的不同构建过程.

qmake 使用的工程文件可以用来支持简单和相对复杂的构建系统.简单工程文件使用直接了当的声明风格,定义标准变量来指明工程中用到的源文件和头文件.复杂的工程可以使用控制流结构来调整构建过程.

### 变量 ###

在工程文件中,变量用来存储一系列的字符串.最简单的情况下,这些变量告知qmake关于配置选项的信息,或者提供编译过程中用到的文件名和路径.

qmake在每一个工程文件中查找特定变量,并使用变量的内容来决定写到makefile中的内容.举例来说,`HEADER`和`SOURCES`变量中的一系列值用来告诉qmake,与工程文件同一目录下的头文件和源文件的所在位置.

变量也可以在内部存储临时的值,或者对已经存在的值进行覆盖或扩展的值.下面的行显示了为分变量分配值序列.

```
HEADERS = mainwindow.h paintwidget.h
SOURCES = main.cpp mainwindow.cpp \
          paintwidget.cpp
CONFIG += qt
```

需要注意的是,第一个变量的赋值内容只包含本行内容,第二个则使用了 '\\' 字符来支持跨行,第三个则对变量的原始内容进行了扩展.

`CONFIG`是qmake用到的另一个特殊变量,下面给出了qmake能识别的变量列表.

  * `CONFIG` 通用工程配置选项
  * `DESTDIR` 二进制或可执行文件的位置
  * `FORMS` UI文件列表,由uic对这些文件进行处理
  * `HEADERS` 头文件列表
  * `QT` Qt相关的配置选项
  * `RESOURCES` 资源文件列表
  * `TEMPLATE` 决定构建的输出文件是应用,二进制还是插件
  
位于变量名前面`$$`可以读取变量的内容,这样可以将变量的内容赋值到另一个变量:`TEMP_SOURCES = $$SOURCES`.`$$`操作对所有操作字符串和值序列的内置函数的适用.

### 空格 ###

变量一般可以用来包含带有空格分割的值序列.但是有时变量本身包含空格,这种情况下应该使用双引号:`DEST = "Program Files"`.双引号引用的文本会被看作独立的值.处理路径中的空格时也应该使用双引号.

```
win32:INCLUDEPATH += "C:/mylibs/extra headers"
unix:INCLUDEPATH += "/home/user/extra headers"
```

### 注释 ###

可以在工程文件中通 '#' 来添加注释,注释直到行尾结束,下面是一个例子.

```
# Comments usually start at the beginning of a line, but they
# can also follow other content on the same line.
```

如果要在变量中使用 '#' ,需要使用内置的`LITERAL_HASH`变量.

### 内置函数和控制流 ###

qmake提供了一些列的内置函数来处理变量的内容.最简单的一个例子是`include`函数,它接收文件名作为参数,将指定文件的内容包含到工程文件的`include`函数所在位置.`include`函数在包含其他工程文件时最常用到.

```
include(other.pro)
```

可以通过大括号区域来支持条件结构,有点像编程语言中的if语句,例子如下:

```
win32 {
    SOURCES += paintwidget_win.cpp
}
```

这个例子中的win32必须被设置过,在windows平台上这个变量是自动设置的,在其他平台上也可以通过传给qmake一个`-win32`选项来设置这个变量.注意, '{' 必须与条件在同一行.

`for`函数可以遍历值列表,下面的一个例子将列表中存在的目录加入到变量`SUBDIRS`.

```
EXTRAS = handlers tests docs
for(dir, EXTRAS) {
    exists($$dir) {
        SUBDIRS += $$dir
    }
}
```

对变量更复杂的操作可能会用到其他包含循环功能的内置函数,比如`find`,`unique`和`count`.内置函数可以操作字符串和路径,支持用户输入和调用外部工具.

## 工程模板 ##

`TEMPLATE`变量用来定义工程的类型.如果该变量没有声明,qmake默认使用应用程序模板,并以此生成相应的makefile.工程类型有如下几种.

  * **app** 创建构建应用程序的makefile
  * **lib** 创建构建库的makefile
  * **subdirs** 创建由`SUBDIRS`变量指定的子目录中的makefie
  * **vcapp** 创建构建应用程序的Visual Studio工程文件 
  * **vclib** 创建构建库的Visual Studio工程文件
  * **vcsubdirs** 创建由`SUBDIRS`变量指定的子目录中的Visual Studio工程文件

当使用subdirs模板时,qmake生成一个makeflie来检查每一个子目录,处理子目录中的工程文件,并调用make进行构建.`SUBDIRS`变量用来包含要处理的子目录.

## 通用配置 ##

`CONFIG`变量指定了编译器会用到的选项和特性以及连接用到的库.任何内容都可以加入到`CONFIG`变量,qmake内部能识别的选项如下:

  * **release** 构建发布版
  * **debug** 构建调试版
  * **debug_and_release** 支持构建发布和调试版
  * **debug_and_release_target** 支持构建工程和TARGET的发布和调试版
  * **build_all** 默认发布和调试版全部构建
  * **autogen_precompile_source** 根据预编译头文件生成一个.cpp的源文件
  * **ordered** 如果是subdirs模板,依次处理各子目录
  * **warn_on** 编译器尽可能多的输出告警
  * **warn_off** 编译器尽可能少的输出告警
  * **copy_dir_files** 允许安装规则拷贝目录

**debug_and_release**允许构建发布版和调试版,这种情况下,qmake生成构建这两种版本的对应规则,运行`make all`触发构建全部两种版本.**build_all**添加到`CONFIG`后,规则默认构建全部两种版本.

需要注意的是,`CONFIG`中的所有选项都可以用于条件判断.可以使用内置的`CONFIG`函数来测试某个配置选项是否出现在变量中.下面一个例子展示了对`opengl`的测试.

```
CONFIG(opengl) {
    message(Building with OpenGL support.)
} else {
    message(OpenGL support is not available.)
}
```

这样就能对release和debug的构建做不同的配置.

下面的两个选项定义了工程的不同类型,一些选项只有在相关平台上才能生效,在其他平台上就不起作用了.

  * **qt** 指明这是一个qt应用程序,应当链接Qt库.通过`QT`变量可以控制应用程序要用到的额外Qt模块
  * **thread** 多线程应用程序
  * **x11** X11应用程序或库
  
当使用应用程序和库模板时,还有许多其他的配置选项可以用来调整构建过程.下是一个例子,当你的应用使用了Qt库,并且是一个多线程应用,在debug模式下构建,那么可以这样配置:`CONFIG += qt thread debug`.需要注意的是,这里必须要用'+='而不是'='.

## 声明Qt库 ##

如果`CONFIG`变量包含值qt,qmake支持构建Qt应用程序,这样可以调整你的应用程序用到哪些Qt模块.通过`QT`变量可以指定需要用到的扩展模块,例如,你可以这样向你的工程加入XML和网络模块.

```
CONFIG += qt
QT += network xml
```

`QT`默认包含了core和gui模块,要使用'+=',如果工程不适用gui模块,可以使用'-='来排除它:`QT -= gui`.下面内容列出了`QT`变量可用的选项.

  * **core** QtCore
  * **gui** QtGui
  * **network** QtNetwork
  * **opengl** QtOpenGL
  * **sql** QtSql
  * **svg** QtSvg
  * **xml** QtXml
  * **xmlpatterns** QtXmlPatterns
  * **qt3support** Qt3Support
  
## 特性配置##

qmake可以添加在.prf后缀的文件中定义的额外特性.这些额外特性通常提供对定制工具的支持.为了给构建过程添加特性,将特性名称加入`CONFIG`变量即可.例如qmake可以配置构建过程利用pkg-config提供的库,例如D-Bus和ogg库.

```
CONFIG += link_pkgconfig
PKGCONFIG += ogg dbus-1
```

## 声明其他库 ##

如果你在工程时钟使用其他库,你需要在工程文件中指明.qmake搜索的库路径和要链接的库可以加到`LIBS`变量中.支持添加库路径以及Unix风格的库引用.另外库相关的头文件搜索路径可以使用`INCLUDEPATH`进行添加.

```
INCLUDEPATH = c:/msdev/include d:/stn/include
LIBS += -L/usr/local/lib -lmath
```

*参考文献*

1. [官方文档链接] [http://doc.qt.io/archives/qt-4.8/qmake-project-files.html](http://doc.qt.io/archives/qt-4.8/qmake-project-files.html)
