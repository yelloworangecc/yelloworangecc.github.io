---
layout: post
title:  "C++代理模式"
summary: 本文用一个简单例子介绍什么是代理模式.
featured-img: sleek
---

代理类是为另一个类提供修改接口的类.

比如,我们有一个数组类,它只用于存放二进制数据(0或1).那么我们可以这样定义这个类:

```cpp
struct array1 {
    int mArray[10];
    int & operator[](int i) {
      ...
    }
}; 
```

我们如果想让`operator[]`在赋值时对于不是0或1的值(比如 `a[1] = 2`)抛出异常该怎么实现呢?这是无法实现的,因为重载的操作符只能获得数组的索引,无法获得对应的值.

要解决这个问题我们可以使用代理模式:

```cpp
#include <iostream>
using namespace std;

struct aproxy {
    aproxy(int& r) : mPtr(&r) {}
    int operator = (int n) {
        if (n > 1 || n < 0) {
            throw "不是二进制数";
        }
        *mPtr = n;
    }
    int * mPtr;
};

struct array {
    int mArray[10];
    aproxy operator[](int i) {
        return aproxy(mArray[i]);
    }
};

int main() {
    try {
        array a;
        a[0] = 1;   // 正常
        a[0] = 42;  // 异常
    }
    catch (const char * e) {
        cout << e << endl;
    }
}
```

这样就有代理类来实现对二进制数据的检查,这时数组类的`operator[]`返回的是代理类的实例,代理类可以限制对数组类内部数据的访问.
