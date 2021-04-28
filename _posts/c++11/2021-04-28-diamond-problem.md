---
layout: post
title:  "使用虚继承解决砖石问题"
summary: C++多重继承非常强大,但是很难用,使用不慎容易出问题.本文讲述如何使用虚继承来解决多重继承带来的常见问题.
featured-img: sleek
---

[原文链接](https://www.cprogramming.com/tutorial/virtual_inheritance.html "原文链接")

## 砖石问题 ##

钻石问题是多重继承带来的一类问题.下面是一段描述钻石问题的代码.

```
class storable //类transmitter和receiver会从这个基类继承
{
        public:
        storable(const char*);
        virtual void read();
        virtual void write(); 
        virtual ~storable();
        private:
        ....
}

class transmitter: public storable 
{
        public:
        void write();
        ...
} 
 
class receiver: public storable
{
        public:
        void read();
        ...
}
 
class radio: public transmitter, public receiver
{
        public:
        void read();
        ....
}
```

由于transmitter类和receiver类都有基类中的write方法,当radio对象调用write会产生歧义,编译器不清楚要使用write的哪一个版本(transmitter的还是receiver的).

为了弄清楚这个问题,先来看下对象在内存中是如何表示的.多种继承机制会简单的将两个对象在内存中依次排列,上面这个例子中,radio同时是transmitter和receiver的子类,所以storable类会在radio对象中存在两份.在编译时,g++编译器会报错: 'request for member "write" is ambiguous',因为编译器无法区分调用哪一个write方法.

幸运的是,C++提供了虚拟继承来解决这个问题.为了阻止编译器给出这样的错误,从基类storable继承的两个类都使用虚继承关键字virtual.

```
class transmitter: public virtual storable 
{
        public:
        void read();
        ...
} 
 
class receiver: public virtual storable
{
        public:
        void read();
        ...
} 
```

当我们使用虚继承时,公共的基类只有一个实例.也就是说,radio类将只包含一个storable类的实例,这个实例会被transmitter类和receiver类共享.这样就消除了歧义,代码就可以正常编译了.

### 虚继承的内存分布 ###

为了能追踪storable类的实例,编译器必须为transmitter类和receiver类提供一个虚函数表.当radio对象被创建时,首先创建一个storable实例,接着transmitter实例和receiver实例.transmitter类和receiver类在它们的虚函数表有一个虚指针,存储了storable实例在内存中的偏移量.当这两个类要访问storable类中的成员时,就可以通过这个虚指针来找到storable实例,进而找到storable的成员.

## 构造函数与虚继承 ##

由于虚基类只有一个实例,这个实例被多个继承自它的类共享,因此虚基类的构造函数不是被继承自它的类调用(普通的继承是这样调用的),否则也就意味着构造函数会被多次调用.事实上,虚基类的构造函数由具体类来调用.在上面的例子中radio类直接调用storable类的构造函数.如果先要向storable构造函数传递参数,应该使用初始化列表.

```
radio::radio ()
    : storable( 10 ) //storable类可能需要的一个值 
    , transmitter()
    , receiver()
{}
```

另外,在构造radio对象时,如果transmitter或者receiver尝试在初始化列表触发storable的构造函数时,相关的语句会被完全跳过.当心!这里容易产生不易察觉的错误.

还有,虚基类的构造函数总是在非虚基类之前调用.这保证了继承自虚基类的类能够感知虚基类的存在,并正常调用自己的构造函数.这种含有虚基类的继承层次结构下析构函数的调用顺序遵循C++一贯的原则:析构函数的调用顺序与构造函数相反.也就是说,虚基类的实例会在最后被销毁.

## 委派 ##

虚继承的应用产生了一个非常强大的技术:使用一个公共抽象类,从一个类中类中委派方法.也叫交叉委派.我们对砖石问题中的场景稍加改动:为了让radio对象正常工作假设transmitter类中的write方法需要访问receiver类中的read方法.

```
class storable 
{
        public:
        storable(const char*);
        virtual void read()=0; //纯虚函数,使storable成为抽象接口
        virtual void write();
        virtual ~storable();
        private:
        ....
}
 
class transmitter: public virtual storable 
{
        public:
        void write()
        {
                read();
                ....
        }
} 
 
class receiver: public virtual storable
{
        public:
        void read();
}
 
class radio: public transmitter, public receiver
{
        public:
        ...
}
 
int main()
{
        radio *rad = new radio();
        receiver *r1 = rad;
        transmitter *r2 =rad;
 
        rad->write();
        r1->write();
        r2->write();
        return 1;
}
```

虚继承场景下,当transmitter类的write方法被调用时,receiver方法的read函数随之被调用,实际上transmitter类没有read方法.在上面的层次结构中,我们只需要初始化radio类因为transmitter和receiver都是抽象的.

## C++多重继承的其他问题 ##

当你在使用想radio这样的基于虚继承的类时,应当尽量避免C风格的类型转换,而应使用C++的dynamic_cast来替代.这种转换会进行运行时检查,确保要转换的类存在继承关系.否则,转换的结果会是一个空指针,或者在使用引用时抛出bad_cast异常.

## 内存分布与构造析构顺序实验 ##

实验代码如下.

```
#include<iostream>
using namespace std;

class storable //类transmitter和receiver会从这个基类继承
{
public: 
  storable():p_s1(0),p_s2(0){ cout << "storable constructed\n";}
  virtual void read(){ cout << "storable read\n"; }
  virtual void write(){ cout << "storable write\n"; }
  virtual ~storable(){ cout << "storable destructed\n"; }
private:
  unsigned long long p_s1;
  unsigned long long p_s2;
};

class transmitter: virtual public storable 
{
public:
  transmitter():p_t(1){ cout << "transmitter constructed\n";}
  void write(){ cout << "transmitter write\n"; }
  ~transmitter(){ cout << "transmitter destructed\n"; }
private:
  unsigned long long p_t;
};
 
class receiver: virtual public storable
{
public:
  receiver():p_r(2){ cout << "receiver constructed\n";}
  void read(){ cout << "receiver read\n"; }
  ~receiver(){ cout << "receiver destructed\n"; }
private:
  unsigned long long p_r;
};
 
class radio: public transmitter, public receiver
{
public:
  radio():p_add(3){ cout << "radio constructed\n";}
  void read(){ cout << "radio read\n"; }
  ~radio(){ cout << "radio destructed\n"; }
private:
  unsigned long long p_add;
};

int main(void)
{
  radio r;
  cout << "radio size " << sizeof(r) << " bytes" << endl;
  unsigned long long * ptr = (unsigned long long*)&r;
  for (int i = 0; i < sizeof(r) / 8; ++i)
  {
      cout << *ptr << endl;
      ++ptr;
  }
  return 0;
}

```

非虚继承的运行结果如下.

```
$g++ -o run virtual_inheritance.cpp  && ./run
storable constructed
transmitter constructed
storable constructed
receiver constructed
radio constructed
radio size 72 bytes

140696106780432
0
0
1
140696106780480
0
0
2
3

radio destructed
receiver destructed
storable destructed
transmitter destructed
storable destructed

```

推测的radio对象内存结构如下.

```

|--------------------------|
| vptr of transmitter 8    |
| data of storable    8x2  |
| data of transmitter 8    |
|--------------------------|
| vptr of receiver    8    |
| data of storable    8x2  |
| data of receiver    8    |
|--------------------------|
| data of radio       8    |
|--------------------------|
```

虚继承的运行结果如下.

```
$g++ -o run virtual_inheritance.cpp  && ./run
storable constructed
transmitter constructed
receiver constructed
radio constructed
radio size 64 bytes

140702825531464
1
140702825531520
2
3
140702825531584
0
0

transmitter write
radio destructed
receiver destructed
transmitter destructed
storable destructed

```

推测的radio对象内存结构如下.

```
|--------------------------|
| vptr of transmitter 8    |
| data of transmitter 8    |
|--------------------------|
| vptr of receiver    8    |
| data of receiver    8    |
|--------------------------|
| data of radio       8    |
|--------------------------|
| vptr of storable    8    |
| data of storable    8x2  |
|--------------------------|

```
