---
layout: post
title:  "QT系列学习笔记:源对象编译器"
date:   2018-01-28 22:25:16 +0800
categories: jekyll update
---

# 元对象编译器 #

{% include copyright.md %}
{% include qt-introduction.md %}

## 什么是元对象编译器 ##

元对象编译器(moc)是用来处理QT的C++扩展的程序.moc工具读取C++头文件,如果它发现有一个以上的类声明中包含`Q_OBJECT`,那么它产生一个包含元对象代码的源文件.这些元对象代码是用来满足信号和槽机制,运行时类型信息和动态属性系统的要求.moc生成的C++源文件必须与类的实现一起编译和链接.如果你使用qmake来创建makefile,编译规则将包含moc的调用,因此不需要开发者直接调用moc.

## 用法 ##

moc典型的用法是以包含以下类声明的文件为输入文件:

```
class MyClass : public QObject
{
    Q_OBJECT

public:
    MyClass(QObject *parent = 0);
    ~MyClass();

signals:
    void mySignal();

public slots:
    void mySlot();
};
```

除了信号和槽,moc也实现了下面这个例子中的对象属性.其中` Q_PROPERTY()`宏声明了一个对象属性,而`Q_ENUMS()`在类的内部声明了一个枚举列表,它可以用在"属性系统"中.在这个例子中声明了一个枚举`Property`类型的属性`property`,它有一个get方法叫`priority()`和一个set方法叫`setPriority()`.

```
class MyClass : public QObject
{
    Q_OBJECT
    Q_PROPERTY(Priority priority READ priority WRITE setPriority)
    Q_ENUMS(Priority)

public:
    enum Priority { High, Low, VeryHigh, VeryLow };

    MyClass(QObject *parent = 0);
    ~MyClass();

    void setPriority(Priority priority) { m_priority = priority; }
    Priority priority() const { return m_priority; }

private:
    Priority m_priority;
};
```

`Q_FLAGS()`宏声明的枚举会当做标记来使用,`Q_CLASSINFO()`宏允许你添加名/值对到类的元对象中:

```
class MyClass : public QObject
{
    Q_OBJECT
    Q_CLASSINFO("Author", "Oscar Peterson")
    Q_CLASSINFO("Status", "Active")

public:
    MyClass(QObject *parent = 0);
    ~MyClass();
};
```

moc的生成文件必须像普通C++代码一样经过编译和链接,否则构建会在链接阶段失败.qmake可以自动生成相关make规则来完成这些工作.如果类声明在myclass.h中,moc会生成`moc_myclass.cpp`,这个文件需要经过编译生成对象文件,例如windows下的`moc_myclass.obj`,然后和其他对象文件一起链接到程序中.

## 为moc编写make规则 ##

除了最简单的程序以外,我们推荐自动运行moc,只需要向负责构建程序的makefile中加入一些规则,make可以在必要的时候运行moc并处理moc的输出.我们也推荐使用qmake来生成makefile.qmake生成的makefile包含了moc所有内容.如果你想自己写makefile,下面是一些推荐做法.
对于在头文件中声明了`Q_OBJECT`的类,makefile可以这样写:

```
moc_%.cpp: %.h
        moc $(DEFINES) $(INCPATH) $< -o $@
```

也可以像这样,使用独立的规则:

```
moc_foo.cpp: foo.h
        moc $(DEFINES) $(INCPATH) $< -o $@
```

此外你还必须将`moc_foo.cpp`加入到代表所有源文件的变量中去,将`moc_foo.o`或者moc_foo.obj加入到代表所有目标文件的变量中去.上面的例子中`$(DEFINES)`和`$(INCPATH)`会展开为传递给C++编译器的宏定义和头文件路径,moc在处理源文件时会用到.我们推荐使用后缀.cpp,当然你依然可以选择使用.C, .cc, ,.CC ,.cxx, .c++.

对于在源文件中声明了`Q_OBJECT`的类,我们推荐这样写makefile:

```
foo.o: foo.moc

foo.moc: foo.cpp
        moc $(DEFINES) $(INCPATH) -i $< -o $@
```

这保证了正在编译foo.cpp前,make会先运行moc.然后你需要将`#include "foo.moc"`加入到foo.cpp中.

## 命令行选项 ##
  * `-o<file>` 输出到文件<file>而不死标准输出.
  * `-f[<file>]` 强制在输出中产生一个`#include`语句.对于.h或.H的头文件,这个选项是默认有效的.当头文件不符合命名惯例时,这个选项才有用.
  * `-i` 不再输出中产生`#include`语句.这个在C++源文件中包含类声明时有效.
  * `-nw` 不要产生任何告警.
  * `-p<path>` 使moc在产生`#include`语句时在文件名前加`<path>/`.
  * `-I<dir>` 将目录添加到头文件搜索路径.
  * `-E` 只处理,不生成代码.
  * `-D<macro> [=<def>]` 定义定义宏.
  * `-U<macro>` 取消宏定义.
  * `@<file>` 从文件中读取额外的命令行选项.
  * `-h` 显示帮助信息.
  * `-v` 显示版本号.
  * `-Fdir` 适用于苹果电脑,将framework目录添加到头文件搜索列表开始处.
  
还可以使用预处理符号`Q_MOC_RUN`来告诉moc不要处理头文件中的某部分代码块.

```
#ifndef Q_MOC_RUN
    ...
#endif
```

## 诊断 ##

如果你在构建程序的最后阶段遇到链接错误,比如说`YourClass::className()`未定义或者`YourClass`缺少vtable等一些问题.最有可能的事你忘记编译或者`#include`moc生成的C++代码,或者没有在链接命令中包含对应的对象文件.如果你使用qmake,重新运行qmake以更新makefile就可以解决了.


## 限制 ##

moc不能处理所有的C++代码,主要问题在于模板类不能包含信号或者槽,下面是一个例子:

```
class SomeTemplate<int> : public QFrame
{
    Q_OBJECT
    ...

signals:
    void mySignal(int);
};
```

另一个限制在于moc不会展开宏,所以不能使用宏来声明信号和槽,也不能使用宏来定义QObject基类.本文后续描述的其他一些限制并不特别重要,可以采用一些替代方式来规避.

### 多重继承中QObject在前 ###

如果你使用多重继承,moc认为第一个继承的类应当是QObject的子类.确保只有第一个继承的类是QObject.

```
// correct
class SomeClass : public QObject, public OtherClass
{
    ...
};
```

此外moc不支持QObject的虚继承.(虚继承:为了解决从不同途径继承来的同名的数据成员在内存中有不同的拷贝造成数据不一致问题，将共同基类设置为虚基类。这时从不同的路径继承过来的同名数据成员在内存中就只有一个拷贝，同一个函数名也只有一个映射。这样不仅就解决了二义性问题，也节省了内存，避免了数据不一致的问题.)

## 函数对象不能作为信号或槽参数 ##

多数情况下你可能认为使用函数指针作为信号或槽参数,但是我们推荐使用继承来代替,下面是一个不合法的例子:

```
class SomeClass : public QObject
{
    Q_OBJECT

public slots:
    void apply(void (*apply)(List *, void *), char *); // WRONG
};
```

你可以这样来规避限制:

```
typedef void (*ApplyFunction)(List *, void *);

class SomeClass : public QObject
{
    Q_OBJECT

public slots:
    void apply(ApplyFunction, char *);
};
```

有时使用继承和虚函数代替函数指针是更好的选择.

### 枚举和类型定义必须完整 ###

在检查函数的签名时,`QObject::connect()`仅在字面上比较数据的类型.因此,`Alignment`和`QT::Alignment`会当做两种不同的类型来对待.因此,需要在声明信号和槽以及建立联系时使用完整的数据类型.

```
class MyClass : public QObject
{
    Q_OBJECT

    enum Error {
        ConnectionRefused,
        RemoteHostClosed,
        UnknownError
    };

signals:
    void stateChanged(MyClass::Error error);
};
```

### 类型宏不能用作信号和槽参数 ###

由于moc不会展开宏,可以接收一个参数的类型宏用在信号和槽中是无效的,下面是一个例子:

```
#ifdef ultrix
#define SIGNEDNESS(a) unsigned a
#else
#define SIGNEDNESS(a) a
#endif

class Whatever : public QObject
{
    Q_OBJECT

signals:
    void someSignal(SIGNEDNESS(int));
};
```

没有参数的宏是可以使用的.

### 嵌套的类不能有信号和槽 ###

错误的例子如下:

```
class A
{
public:
    class B
    {
        Q_OBJECT

    public slots:   // WRONG
        void b();
    };
};
```

### 信号和槽的返回类型不能被引用 ###

信号和槽可以有返回类型,但是返回引用将被视作返回void.

### 只有信号和槽可以出现在类的对应区域 ###

moc会在你尝试将其他内容放在类的信号和槽区域内时报错.
