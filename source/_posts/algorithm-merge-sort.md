#归并排序
title: 归并排序
date: 2014-09-06 14:40:01
categories:
- algorithm
tags:
- algorithm
- sort
---

![](http://res.astraylinux.com/algorithm/msort.gif)
归并操作（merge），也叫归并算法，指的是将两个已经排序的序列合并成一个序列的操作。归并排序算法依赖归并操作。

归并操作的过程如下：
1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置
3. 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
4. 重复步骤3直到某一指针到达序列尾
5. 将另一序列剩下的所有元素直接复制到合并序列尾

<!--more-->

```bash
#include<stdio.h>
#include<iostream>
using namespace std;

//归并排序
int marge(int* src,  int* target,  int start, int mid,  int end){
	int i, j, k;
	for(i = mid+1, j = start,  k=start ;i <= end && k<= mid ;j++){
		if(src[k] <= src[i])
			target[j] = src[k++];
		else
			target[j] = src[i++];
	}
	if(k <= mid)
		for(int l=0; l <= mid - k; l++)
			target[l+j] = src[l+k];

	if(i <= end)
		for(int l=0; l <= end - i; l++)
			target[l+j] = src[l+i];
	return 0;
}

int msort(int *src, int *target, int start,  int end){
	if( start == end )
		target[start] = src[start];
	else{
		int mid = (start + end)/2;
		int *tmp = new int[end - start + 1];
		msort(src, tmp, start, mid);
		msort(src, tmp, mid+1, end);
		marge(tmp, target, start, mid, end);
		//delete [] tmp;
	}
	return 0;
}

int print_array(int* array, int start, int end){
	for(int i=start;i<=end;i++){
		cout<<array[i]<<", ";
	}
	cout<<endl;
	return 0;
}

int main(int argc,  char** argv){
	int array[] = {16, 26, 35, 68, 94, 31, 64, 36, 87, 17, 21, 53, 9, 125, \
		136, 14, 18, 39, 77, 88, 99, 65, 24, 30};
	int size = sizeof(array)/sizeof(int);
	int *target = new int[size];
	msort(array, target, 0, size-1);
	print_array(target, 0, size-1);
	return 0;
}

```
