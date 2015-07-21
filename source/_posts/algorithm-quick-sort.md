title: 快速排序
date: 2013-09-05 14:40:01
categories:
- algorithm
tags:
- algorithm
- sort
---

![](http://res.astraylinux.com/algorithm/qsort.gif)
设要排序的数组是A[0]……A[N-1]，首先任意选取一个数据（通常选用数组的第一个数）作为关键数据，然后将所有比它小的数都放到它前面，所有比它大的数都放到它后面，这个过程称为一趟快速排序。值得注意的是，快速排序不是一种稳定的排序算法，也就是说，多个相同的值的相对位置也许会在算法结束时产生变动。
一趟快速排序的算法是：
1. 设置两个变量i、j，排序开始的时候：i=0，j=N-1；
2. 以第一个数组元素作为关键数据，赋值给key，即key=A[0]；
3. 从j开始向前搜索，即由后开始向前搜索(j--)，找到第一个小于key的值A[j]，将A[j]和A[i]互换；
4. 从i开始向后搜索，即由前开始向后搜索(i++)，找到第一个大于key的A[i]，将A[i]和A[j]互换；
5. 重复第3、4步，直到i=j； (3,4步中，没找到符合条件的值，即3中A[j]不小于key,4中A[i]不大于key的时候改变j、i的值，使得j=j-1，i=i+1，直至找到为止。找到符合条件的值，进行交换的时候i， j指针位置不变。另外，i==j这一过程一定正好是i+或j-完成的时候，此时令循环结束）。

<!--more-->

```bash
#include<stdio.h>
#include<iostream>
using namespace std;

int qsort(int *array,int start,int end){
     if(start >= end)
          return 0;
     int i = start;
     int j = end;
     int k = array[start];
     while(i < j){
          while(i < j && array[j] >= k)j--;
          array[i] = array[j];
          while(i < j && array[i] <= k)i++;
          array[j] = array[i];
     }
     array[i] = k;
     qsort(array,start,i-1);
     qsort(array,j+1,end);
}

int print_array(int* array,int start,int end){
     for(int i=start;i<=end;i++){
          cout<<array[i]<<",";
     }
     cout<<endl;
}
int main(int argc,char** argv){
     int array[] = {16,26,35,68,94,31,64,36,87,17,21,53,9,125,136,14,18,39,77,88,99,65,24,30};
     qsort(array,0,sizeof(array)/sizeof(int)-1);
     print_array(array,0,sizeof(array)/sizeof(int)-1);
}
```
