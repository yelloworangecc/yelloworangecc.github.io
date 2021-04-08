---
layout: post
title:  "前K个高频元素"
summary: 这个算法题难度不大,但是涉及了两种常用的数据结构--散列和堆,值得贴出来复习一下!
featured-img: sleek
---

## 问题描述 ##

**[347. 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements)**:给定一个非空的整数数组,返回其中出现频率前 k 高的元素,示例如下.

```
示例 1:
输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]

示例 2:
输入: nums = [1], k = 1
输出: [1]
```

**解题思路**:首先需要对数组内的元素计数,散列能很好地完成这个任务.接着要取出计数值前K大的元素,我们当然可以使用排序算法,但是其他计数值的大小我们不必关心,因此使用堆效率更高.我们是否需要对所有计数值建堆呢?也不用.使用小根堆,始终把当前最大的k个计数值保留在小根堆内,即可.具体来说:1)如果小根堆未满k个元素,那么直接加入当前元素.2)如果小根堆满了k个元素,那么比较堆顶元素(最小值)和当前元素,如果当前元素大,那么移除堆顶元素,加入当前元素.

## 散列表 ##

在c++中,实现散列表数据结构的容器叫做`unordered_map`,与`unordered_set`的区别是它内部的元素是key-value对,而与`map`的区别是它的内部元素是无序的(map使用红黑树实现,维护了内部元素的顺序,插入和删除元素需要调整红黑树,效率低一些).

**模板类原型**:

```
template < class Key,                                    // unordered_map::key_type
           class T,                                      // unordered_map::mapped_type
           class Hash = hash<Key>,                       // unordered_map::hasher
           class Pred = equal_to<Key>,                   // unordered_map::key_equal
           class Alloc = allocator< pair<const Key,T> >  // unordered_map::allocator_type
           > class unordered_map;
```

**常用成员函数**:

* `empty` 判断散列表是否为空.
* `size` 返回key-value对的个数.
* `operator[]` 返回key对应的value,如果key不存在,则插入key,对应的value使用默认构造函数初始化.
* `at` 返回key对应的value,如果key不存在,抛出`out_of_range`异常.
* `find` 查找key返回一个迭代器,如果未找到key,那么返回`unordered_map::end`迭代器.
* `count` 查找key返回一个整数,如果找到key,返回1,否则返回0.
* `insert` 插入key-value对,key必须不能重复.
* `erase` 删除key-value对.
* `clear` 清空所有key-value对.

## 堆 ##

在c++中,实现堆数据结构的容器叫做优先队列(priority_queue),与`queue`的区别是它出队时总是将优先级最高的元素移除.更确切的说,c++的优先队列是一个容器适配器,它依赖于一个底层容器(需要支持随机访问),默认是vector.也可以直接在支持随机访问的容器上使用`make_heap`,`push_heap`,`pop_heap`等算法函数进行堆相关的操作.

**模板类原型**:

```
template <class T, class Container = vector<T>,
  class Compare = less<typename Container::value_type> > class priority_queue;
```

**常用成员函数**

* `empty` 判断优先队列是否为空
* `size` 返回队列中元素个数
* `top` 访问队首元素
* `push` 新元素入队
* `pop` 队首元素出队

## 代码分析 ##

```
#include<iostream>
#include<vector>
#include<queue>
#include<unordered_map>
using namespace std;

bool cmp(pair<int, int>& m, pair<int, int>& n)
{
    return m.second > n.second; //小根堆
}

vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int, int> hashmap;
    for (auto& key : nums) {
        hashmap[key]++; //[]会插入新key同时使用默认构造函数初始化value,其中int初始化为0
    }
    //加入小根堆
    priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(&cmp)> q(cmp);//decltype更据表达式推导类型
    for (auto& [key, value] : hashmap) { //c++17 structured bindings
        //前k个直接加入小根堆
        if (q.size() < k){
            q.emplace(key, value);
            continue;
        }
        //小根堆维持k个节点,如果有大于堆顶元素的,则更新小根堆
        if (q.top().second < value) {
            q.pop();
            q.emplace(key, value);
        }
    }
    //小根堆出队
    vector<int> ret;
    while (!q.empty()) {
        ret.emplace_back(q.top().first);
        q.pop();
    }
    return ret;
}

int main(void){
    vector<int> nums = {1,2,2,3,3,3,4,4,4,4,5,5,5,5,5,6,6,6,6,6,7,7,7,7,7};
    vector<int> result = topKFrequent(nums,2);
    for( auto item: result){
        cout << item << ' ';
    }
    cout << endl;
    return 0;
}

```

**注意:**
1. 建堆时的比较函数需要自定义,比较的对象是key-value对,实际比较的是value的值(计数值).
2. decltype更据表达式的结果自动推导类型
3. `for (auto& [key, value] : hashmap)`用到了c++17的结构化绑定,将结构体内的值直接绑定到变量.

**运行结果**

```
g++ -o run -std=gnu++17 topkfrequent.cpp && ./run
7 6
```
