---
layout: post
title:  "分治与动态规划"
summary: 3月份的计划是复习各类算法,分治和动态规划是比较常用的算法设计策略,之前我并没有搞懂,并且有些混淆两者,现在重新整理成博客.
featured-img: sleek
---

## 分治 ##

分治是一种算法设计策略.一个典型的分治算法使用下面三个步骤解决问题:

* 划分:将所给问题划分为类型相同的子问题.
* 解决:递归地解决这些子问题.
* 合并:正确地合并子问题的答案.

快速排序是一个典型的使用了分治策略的算法.下面是一段实现了快速排序的C++代码

```
#include<iostream>
#include<vector>
using namespace std;
//一次划分
int partition(vector<int> &array, int low, int high)
{
  int base = array[high]; //划分的基准数
  int i = low - 1;//用于将小于base的数交换至数组前部
  int j;
  int temp;
  for (j = low; j < high; j++)
  {
    //交换小于base的数至数组前部
    if (array[j] <= base)
    {
      i++;
      swap(array[i],array[j]);
    }
  }
  //交换base至最终所在位置
  swap(array[i+1],array[high]);

  return i+1;
}
//递归处理子问题
void quicksort(vector<int> &array, int low, int high)
{
  int mid;
  if (low >= high) return;
  mid = partition(array,low,high);
  quicksort(array,low,mid-1);
  quicksort(array,mid+1,high);
}

int main()
{
  vector<int> array {54,26,98,75,36,36,84,56,72,16,43};
  quicksort(array,0,10);
  for(auto item : array) cout<<item<<" ";
  return 0;
}
```

这段代码先定义了一个划分函数partition,这个函数内先选取数组末尾的元素作为基准值,通过比较和交换操作,将整个数组调整为小于基准值的部分(数组前部),基准值,大于基准值的部分(数组后部),最后返回基准值所在位置.

通过一次划分,可以将整个数组的排序问题可以分解为分别对小于基准值的子数组和大于基准值的子数组进行排序.这样quicksort很容易通过递归函数调用来实现.需要注意的是递归调用的终止条件(low>=high):1)最小的子问题的情况下,数组只有一个元素,已经是一个有序的数组,此时子问题数组的上下界索引(low = high)是重合,应当停止递归调用.2)前一次划分基准值本身是最小值或最大值,那么划分的两个子问题中有一个为空(low = high+1)的情况,应当停止递归调用.

最后由于算法的基本操作是交换数组元素位置,算法执行结束后只需返回完整的数组即可,不需要额外的合并操作.

## 动态规划 ##

动态规划是一种对普通递归求解的优化.对于任何输入相似的可以通过普通递归求解的问题,都可以使用动态规划来进行优化.主要的思想是将子问题的答案存储下来,后续需要这些答案的时候不必再重新计算它们.这样可以将算法的时间复杂度从指数级减小到多项式级.一个简单的例子,如果使用递归的方法来求解斐波那契数列值,那么时间复杂度是指数级的,但是如果我们将子问题的解存储下来,那么优化后时间复杂度是线性的.

```
int fib(int n)
{
  if(n<=1) return n;
  return fib(n-1) + fib(n-2);
}

int fib_optimize(int f[], int n)
{
  f[0] = 1;
  f[1] = 1;
  
  for(int i = 2; i <= n; i++)
  {
    f[i] = f[i-1] + f[i-2];
  }
  return f[n];
}
```

0-1背包问题也是一个比较典型的可以使用动态规划思想来求解的问题.0-1背包问题的可以描述为:有n件物品和一个容量为c的背包,放入第i件物品耗费的空间是wi，得到的价值是vi,求解将哪些物品装入背包可使价值总和最大。
一种很容易想到的解法是:枚举所有排列组合的情况,计算他们的价值,选出符合条件的价值最大的作为解即可.但是这种方法时间复杂度太高.下面是使用动态规划求解0-1背包问题的C++代码.

```
#include <iostream>
#include <iomanip>
#include <vector>
using namespace std;
#define max(N1,N2) (N1)>(N2)?(N1):(N2)

//回溯寻找选中的商品
void find_answer(vector<vector<int>> & m, vector<int>& w, int i, int j)
{
  if(j <= 0) return;

  if (m[i][j] == m[i - 1][j])
  {
    find_answer(m, w, i - 1, j);
  }
  else
  {
    cout <<"选中物品" << i+1 << " ";
    find_answer(m, w, i - 1, j - w[i]);
  }
}

int main()
{
  int n = 9;//物品数量
  int c = 20;//背包容量
  vector<int> w = {16,5,8,7,11,2,1,10,9};//物品花费容量
  vector<int> v = {3,5,6,10,8,4,1,5,7};//物品价值
  vector<vector<int>> m;//存储子问题的解的状态矩阵

  //初始化矩阵
  for (int i = 0; i < n; i++)
  {
      vector<int> buff(c+1, 0);
      m.push_back(buff);
  }
  
  for (int i = 0; i < n; i++)//循环处理每一个物品
  {
    for (int j = 1; j <= c; j++)//容量从1~c
    {
      if (i == 0)//第一个物品,若容量足够,选取该物品,记录当前价值,否则不取该物品
      {
        m[i][j] = (w[i] <= j ? v[i] : 0);
      }
      else//后续物品
      {
        if (j >= w[i])
        {
          m[i][j] = max(v[i] + m[i - 1][j - w[i]], m[i - 1][j]);
        }
        else
        {
          m[i][j] = m[i-1][j];
        }
      }
    }
  }

  //输出状态矩阵
  for( int i = 0; i < n; i++)
  {
    for ( int j = 1; j <= c; j++)
    {
      cout<<setw(2)<<left<<m[i][j]<<" ";
    }
    cout << endl;
  }

  //回溯寻找选中的商品
  find_answer(m, w, n-1, c);
  
  return 0;
}
```

状态矩阵的输出结果如下,可以看到背包可以装载的最大容量为22.

```
0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  3  3  3  3  3
0  0  0  0  5  5  5  5  5  5  5  5  5  5  5  5  5  5  5  5
0  0  0  0  5  5  5  6  6  6  6  6  11 11 11 11 11 11 11 11
0  0  0  0  5  5  10 10 10 10 10 15 15 15 16 16 16 16 16 21
0  0  0  0  5  5  10 10 10 10 10 15 15 15 16 16 16 18 18 21
0  4  4  4  5  5  10 10 14 14 14 15 15 19 19 19 20 20 20 22
1  4  5  5  5  6  10 11 14 15 15 15 16 19 20 20 20 21 21 22
1  4  5  5  5  6  10 11 14 15 15 15 16 19 20 20 20 21 21 22
1  4  5  5  5  6  10 11 14 15 15 15 16 19 20 20 20 21 22 22
选中物品6 选中物品5 选中物品4

```

上述代码的核心是for循环中的内容,外层循环处理所有9个物品,内层容量由1至20循环.如果是第1个物品(容量花费16,价值3),那么只要容量满足就将物品的价值记录到状态矩阵,得到状态矩阵的第一行输出.

```
0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  3  3  3  3  3
```

对于后续物品,如果整个容量不足以容纳,则不取当前物品,那么当前价值保持不变`m[i-1][j]`.如果整个容量足够容纳这个物品,那么比较取这个物品时的价值与不取这个物品时的价值,取较大者.其中如果取当前物品,那么价值求解可以转换为上一个物品在扣除当前物品容量时的价值`m[i-1][j-w[i]]`加上当前物品的价值`v[i]`.

```
0  0  0  0  5  5  5  6  6  6  6  6  11 11 11 11 11 11 11 11
```

以处理第3个物品为例,当容量为1-7时,不足以容纳物品3,取处理物品2时得到的价值;当容量为8-20时,可以容纳物品3,比较取物品3的价值与不取物品3的价值,均是取物品3的价值大.具体来说当容量为13时,取物品3的价值`6+m[2-1][13-8]->11`大于不取物品3的价值`m[2-1][13]->5`.

最后根据得到的状态矩阵回溯查找选中的物品.从矩阵的最右下角的状态开始`m[i-1][j]`回溯,如果当前物品与上一个物品在容量j下价值相同那么说明当前物品未被选中,继续对上件物品在容量j的状态下进行回溯.如果价值有变化,说明当前物品被选中,那么继续对上件物品在容量`j-w[i]`处进行回溯.

## 总结 ##

分治策略和动态规划策略都使用了划分子问题的方法来求解问题,所不同的是,分治策略划分的子问题是相互独立的,而动态规划策略划分的子问题间存在依赖性,大的问题的解依赖于前面小问题的解,将小问题的求解都记录下来供后面再使用,避免了重复计算,大大提高了算法的效率.