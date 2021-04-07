---
layout: post
title:  "随机数与蓄水池抽样算法"
summary: 蓄水池抽样(Reservoir Sampling)是一个很有趣的问题，它能够在O(n)时间内对n个数据进行等概率随机抽取,另外算法用到了随机数生成函数,本文将对相关内容进行学习和巩固.
featured-img: sleek
---

## 随机数生成 ##
**随机数生成函数**

```
int rand (void);
```

返回一个整型伪随机数,范围在0-RAND_MAX之间.RAND_MAX的值依赖于库的实现,至少是32767.

通过特定算法生成一个数值各不相关的序列,每次调用rand就从这个序列中返回一个值.这个序列的产生需要一个种子来初始化,初始化函数是srand.

要生成更精确的随机数需要使用取模运算将随机数限定在一定范围,并加上一个初始值.

```
v1 = rand() % 100;         // 0 - 99
v2 = rand() % 100 + 1;     // 1 - 100
v3 = rand() % 30 + 1985;   // 1985-2014 
```

**初始化随机数生成器**
```
void srand (unsigned int seed);
```
使用参数seed来初始化伪随机数生成器.

传入srand的每一个不同的seed都能产生不同伪随机数序列,这个序列通过不断调用rand函数返回.

使用相同的seed多次初始化将得到相同的序列.

为了产生更加随机化的值,可以使用一些各不相同的运行时的值来初始化srand函数,比如time函数返回的当前时间秒数可以满足大多数简单的随机化场景.

```
srand (time(NULL));
```

## 蓄水池抽样算法 ##
**398. 随机数索引**
给定一个可能含有重复元素的整数数组,要求随机输出给定的数字的索引.您可以假设给定的数字一定存在于数组中.注意: 数组大小可能非常大.不能使用太多额外空间.具体例子如下.

```
int[] nums = new int[] {1,2,3,3,3};
Solution solution = new Solution(nums);

// pick(3) 应该返回索引 2,3 或者 4。每个索引的返回概率应该相等。
solution.pick(3);

// pick(1) 应该返回 0。因为只有nums[0]等于1。
solution.pick(1);
```

**解题思路:**我们总是选择第一个对象，以1/2的概率选择第二个，以1/3的概率选择第三个，以此类推，以1/i的概率选择第i个对象。当该过程结束时,如果共有m个对象，每一个对象具有相同的选中概率，即1/m.

```
p=(1/i)*(i/(i+1))*((i+1)/(i+2))*...*((m-1)/m)=1/m
```

**代码实现:**

```
#include<iostream>
#include<vector>
#include<ctime>
using namespace std;

int pick(vector<int> &nums, int target) {
  int count = 1;
  int result = 0;
  int n = nums.size();
  for (int i = 0; i < n; ++i)
    {
      int num = nums[i];
      if (num == target)
        {
          if ((rand() % count + 1) == 1) //在1-count间生成随机数,每次有1/count的概率更新为新值
            {
              result = i;
            }
          ++count;
          cout<<result<<' '; //打印当前值
        }
    }
  cout<<endl;
  return result;
}

int main(void){
  srand (time(NULL));
  vector<int> nums = {1,8,6,9,1,3,8,1,6,7,1,2,1,4,8};
  pick(nums,1);
  pick(nums,1);
  pick(nums,1);
  pick(nums,1);
  pick(nums,1);
  pick(nums,1);
  pick(nums,1);
  pick(nums,1);
  pick(nums,1);
  return 0;
}
```

**编译和运行:**

```
$g++ -o run reservoir_sampling.cpp && ./run
0 4 7 7  7
0 0 0 0  0
0 0 7 10 10
0 0 0 0  0
0 0 0 0  12
0 4 7 10 12
0 4 4 4  4
0 4 4 4  12
0 4 4 4  4
```

