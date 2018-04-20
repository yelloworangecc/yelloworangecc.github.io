---
layout: post
title:  "QT4: 布局管理"
summary: Qt的布局管理系统提供了一个简单而强大的在窗口中自动管理窗口部件的方式,能够保证程序有效利用窗口空间.
featured-img: sleek
# categories: jekyll update
---
## 什么是布局管理 ##

Qt的布局管理系统提供了一个简单而强大的在窗口中自动管理窗口部件的方式,能够保证程序有效利用窗口空间.

Qt包含了一系列布局管理类,用来描述窗口部件如何在用户界面中布局排列.这些布局类能在可用空间改变时自动调整窗口部件的位置和大小,保证他们依次排列并且用户界面完整可用.

所有QWidget的子类都可以通过布局来管理他们的子部件.`QWidget::setLayout()`函数将一个布局应用到一个窗口部件.以这种方式指定的布局会负责如下任务:
  
  * 子部件的位置
  * 感知窗口的默认大小
  * 感知窗口的最小大小
  * 处理大小变化
  * 内容改变时自动更新,变化包括
    - 字体大小,文本或子部件的其他内容
	- 影藏或者显示一个子部件
	- 移除子部件
	
## 布局类 ##

Qt的布局类针对手写C++代码设计,允许以像素为单位进行度量,因此易于理解和使用.由Qt Designer生成的外观代码也适用布局类.Qt Designer对于实验性的外观设计非常有用,因为无需编译,链接和运行.

  * **QBoxLayout** 子部件水平或者垂直排列
  * **QButtonGroup** 组织多个按钮的容器
  * **QFormLayout** 管理输入部件和标签组成的表格
  * **QGraphicsAnchor** 在QGraphicsAnchorLayout代表两个项目中间的一个锚点
  * **QGraphicsAnchorLayout** 在图表视图下将部件固定在一起
  * **QGridLayout** 在格子中分布子部件
  * **QGroupBox** 带名字的分组框
  * **QHBoxLayout** 水平排列部件
  * **QLayout** 基类
  * **QLayoutItem** 分布所操作的抽象对象
  * **QSizePolicy** 描述水平和垂直尺寸变化的策略
  * **QSpacerItem** 空白空间
  * **QStackedLayout** 只有一个部件可见的部件栈
  * **QStackedWidget** 只有一个部件可见的部件栈
  * **QVBoxLayout** 水平排列部件
  * **QWidgetItem** 代表一个部件的布局对象
  
## 水平垂直格子和表格布局 ##

用好布局最简单的方式是使用内置的布局管理器:QHBoxLayout, QVBoxLayout, QGridLayout和QFormLayout.这几个类继承自QLayout,QLayout继承自QObject而不是QWidget.它负责管理部件的集合.为了创建更复杂的布局,你可以相互嵌套使用多个布局.

  * **QHBoxLayout** 从左到右水平排列布局

![HBoxLayout](/pics/HBoxLayout.jpeg)

```
QWidget *window = new QWidget;
QPushButton *button1 = new QPushButton("One");
QPushButton *button2 = new QPushButton("Two");
QPushButton *button3 = new QPushButton("Three");
QPushButton *button4 = new QPushButton("Four");
QPushButton *button5 = new QPushButton("Five");
QHBoxLayout *layout = new QHBoxLayout;
layout->addWidget(button1);
layout->addWidget(button2);
layout->addWidget(button3);
layout->addWidget(button4);
layout->addWidget(button5);
window->setLayout(layout);
window->show();
```
  
  * **QVBoxLayout** 从上到下垂直排列布局,与水平排列类似.

![VBoxLayout](/pics/VBoxLayout.jpeg)

  * **QGridLayout** 将部件分布到二维格子里,一个部件可以占用多个格子.

![GridLayout](/pics/GridLayout.jpeg)

```
QWidget *window = new QWidget;
QPushButton *button1 = new QPushButton("One");
QPushButton *button2 = new QPushButton("Two");
QPushButton *button3 = new QPushButton("Three");
QPushButton *button4 = new QPushButton("Four");
QPushButton *button5 = new QPushButton("Five");
QGridLayout *layout = new QGridLayout;
layout->addWidget(button1, 0, 0);
layout->addWidget(button2, 0, 1);
layout->addWidget(button3, 1, 0, 1, 2);
layout->addWidget(button4, 2, 0);
layout->addWidget(button5, 2, 1);
window->setLayout(layout);
window->show();
```

  * **QFormLayout** 两列布局,带有描述标签区域

![FormLayout](/pics/FormLayout.jpeg)

```
QWidget *window = new QWidget;
QPushButton *button1 = new QPushButton("One");
QLineEdit *lineEdit1 = new QLineEdit();
QPushButton *button2 = new QPushButton("Two");
QLineEdit *lineEdit2 = new QLineEdit();
QPushButton *button3 = new QPushButton("Three");
QLineEdit *lineEdit3 = new QLineEdit();
QFormLayout *layout = new QFormLayout;
layout->addRow(button1, lineEdit1);
layout->addRow(button2, lineEdit2);
layout->addRow(button3, lineEdit3);
window->setLayout(layout);
window->show();
```

**使用布局的小贴士**

当在使用布局时,你不需要在构建子部件时传入父对象.布局会自动更新部件的父对象(使用函数`QWidget::setParent()`),使得它们成为布局安装对象的子部件.

在布局中的部件是布局安装部件的子对象,而不是布局本身的子对象.部件只能使用其他部件作为父对象,而不能使用布局作为父对象.

## 添加部件到布局 ##

当添加部件到布局时,布局的处理过程如下:

  1. 所有部件将依据它们的`QWidget::sizePolicy()`和`QWidget::sizeHint()`分配一定初始空间.
  2. 如果任何一个部件设置了拉伸因子(大于0的值),那么根据拉伸因子分配空间.
  3. 如果任何一个部件的拉伸因子设为0,那么这个部件将获得剩余的空间,其中空间将优先分配给具有扩展大小策略的部件.
  4. 任何被分配到的空间小于最小尺寸的部件将分配得到所需的最小尺寸空间.
  5. 任何被分配到的空间大学最大尺寸的部件将分配得到所需的最大尺寸空间.

**拉伸因子**

部件创建时通常没有设置拉伸因子,当它们处于布局中时,它们依据尺寸策略和最小尺寸共享空间.拉伸因子用于改变一个部件相比其他部件得到的空间大小.

如果我们有使用QHBoxLayout布局的三个部件,它们的拉伸因子没有设置,那么它们将平分整个空间.如果为每一个部件设置了拉伸因子,那么它们会按照比例占用整个空间,但是拉伸后的大小不会小于最小尺寸.示意图如下.

![strech-factor](/pics/strech-factor.jpeg)

## 布局问题 ##

使用富文本的标签部件会向父部件的布局引入一些问题,当标签自动换行时,Qt布局处理富文本的方式导致问题发生.

在某些情况下,父对象布局处于`QLayout::FreeResize`模式,意味着不会调整布局的内容来适应小尺寸的窗口,甚至会阻止用户将窗口调小以保证可用性.这种情况可以通过将有问题的部件设置为子类,并实现合适的`sizeHint()`和`minimumSizeHint()`函数.

在另一些情况下,与布局被添加到部件时有关.当你为QDockWidget和QScrollArea调用setWidget时,布局必须已经设置好,否则部件将不可见.

## 手动布局 ##

如果你正在制作一个特殊的布局,你可以用上面描述的方式自定义一个部件.重新实现`QWidget::resizeEvent()`来计算所需的大小并在每个子对象上调用`setGeometry()`

当布局需要重新计算时,窗口部件将接受一个`QEvent::LayoutRequest`类型的事件.重新实现`QWidget::event()`来处理这个事件.

## 如何写一个自定义的布局管理器 ##

另一种手动布局的方法是通过继承QLayout来写你自己的布局管理器,[Border Layout](http://doc.qt.io/archives/qt-4.8/qt-layouts-borderlayout-example.html)和[Flow Layout](http://doc.qt.io/archives/qt-4.8/qt-layouts-flowlayout-example.html)这两个例子展示了如何来做.

这里我们详细展示一个例子,CardLayout类受到Java布局管理器的启发,它将对象依次叠加,每一个对象偏移由`QLayout::spacing()`指定的一段距离.

要写你自己的布局类,你必须做如下定义:

  * 一个数据结构来存储布局要处理的对象,每个对象是QLayoutItem类型,这个例子使用了QList.
  * `addItem()`,如何将对象加入到布局.
  * `setGeometry()`,如何执行布局.
  * `sizeHint()`,布局首选的尺寸.
  * `itemAt()`,在布局中迭代.
  * `takeAt()`,从布局中移除对象.
  
大多数情况下,你也需要实现`minimumSize()`.

### 头文件(card.h) ###

```
#ifndef CARD_H
#define CARD_H
#include <QtGui>
#include <QList>
class CardLayout : public QLayout
{
public:
    CardLayout(QWidget *parent, int dist): QLayout(parent, 0, dist) {}
    CardLayout(QLayout *parent, int dist): QLayout(parent, dist) {}
    CardLayout(int dist): QLayout(dist) {}
    ~CardLayout();
    void addItem(QLayoutItem *item);
    QSize sizeHint() const;
    QSize minimumSize() const;
    QLayoutItem *count() const;
    QLayoutItem *itemAt(int) const;
    QLayoutItem *takeAt(int);
    void setGeometry(const QRect &rect);
private:
    QList<QLayoutItem*> list;
};
#endif
```

### 实现文件(card.cpp) ###

```
#include "card.h"
//返回对象数目
QLayoutItem *CardLayout::count() const
{
    // QList::size() returns the number of QLayoutItems in the list
    return list.size();
}
//返回指定索引位置的对象
QLayoutItem *CardLayout::itemAt(int idx) const
{
    // QList::value() performs index checking, and returns 0 if we are
    // outside the valid range
    return list.value(idx);
}
//移除指定索引位置的对象
QLayoutItem *CardLayout::takeAt(int idx)
{
    // QList::take does not do index checking
    return idx >= 0 && idx < list.size() ? list.takeAt(idx) : 0;
}
//实现了对象的默认安置策略
void CardLayout::addItem(QLayoutItem *item)
{
    list.append(item);
}
//手动释放对象
CardLayout::~CardLayout()
{
     QLayoutItem *item;
     while ((item = takeAt(0)))
         delete item;
}
//实际执行布局,作为参数的矩形不包含外边距,可以使用内边距作为对象之间的间距
void CardLayout::setGeometry(const QRect &r)
{
    QLayout::setGeometry(r);
    if (list.size() == 0)
        return;
    int w = r.width() - (list.count() - 1) * spacing();
    int h = r.height() - (list.count() - 1) * spacing();
    int i = 0;
    while (i < list.size()) {
        QLayoutItem *o = list.at(i);
        QRect geom(r.x() + i * spacing(), r.y() + i * spacing(), w, h);
        o->setGeometry(geom);
        ++i;
    }
}
//下面两个函数返回的大小包含内边距不包含外边距
QSize CardLayout::sizeHint() const
{
    QSize s(0,0);
    int n = list.count();
    if (n > 0)
        s = QSize(100,70); //start with a nice default size
    int i = 0;
    while (i < n) {
        QLayoutItem *o = list.at(i);
        s = s.expandedTo(o->sizeHint());
        ++i;
    }
    return s + n*QSize(spacing(), spacing());
}
QSize CardLayout::minimumSize() const
{
    QSize s(0,0);
    int n = list.count();
    int i = 0;
    while (i < n) {
        QLayoutItem *o = list.at(i);
        s = s.expandedTo(o->minimumSize());
        ++i;
    }
    return s + n*QSize(spacing(), spacing());
}
```

### 说明 ###

  * 这个自定义的布局不能处理宽度变化时的高度变化
  * 忽略了`QLayoutItem::isEmpty()`,意味着布局将隐藏部件视为可见.
  * 对于负责的布局,可以通过缓存计算值来显著提高处理速度,这种情况下需要实现`QLayoutItem::invalidate()`来标记缓存的数据是失效的.
  * 调用`QLayoutItem::sizeHint()`等函数可以代价很大,所以因当在本地存储这些值以便重复使用.
  * 不应当在同一个对象上重复调用`QLayoutItem::setGeometry()`,这个调用在对象具有多个部件时代价非常大,因为布局管理器必须每次完成布局.替代的方法是计算值并设置.
