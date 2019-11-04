---
title: 天才ACM
top: false
cover: false
toc: true
mathjax: true
date: 2019-08-02 10:29:37
password:
summary: 现在给定一个长度为 N 的数列 A 以及一个整数 T。
         我们要把 A 分成若干段，使得每一段的“校验值”都不超过 T。
         求最少需要分成几段。
tags: 
	- 归并
	- 快排
	- 倍增
categories: 算法
---

#### 题目
给定一个整数 M，对于任意一个整数集合 S，定义“校验值”如下:

从集合 S 中取出 M 对数(即 2∗M 个数，不能重复使用集合中的数，如果 S 中的整数不够 M 对，则取到不能取为止)，使得“每对数的差的平方”之和最大，这个最大值就称为集合 S 的“校验值”。

现在给定一个长度为 N 的数列 A 以及一个整数 T。

我们要把 A 分成若干段，使得每一段的“校验值”都不超过 T。

求最少需要分成几段。
#### 输入格式
第一行输入整数 K，代表有 K 组测试数据。

对于每组测试数据，第一行包含三个整数 N,M,T 。

第二行包含 N 个整数，表示数列 A1,A2…AN 。
#### 输出格式
对于每组测试数据，输出其答案，每个答案占一行。
#### 数据范围
1≤K≤12 
1≤N,M≤500000
0≤T≤1018
0≤Ai≤220
#### 输入样例

    2
    5 1 49
    8 2 1 7 9
    5 1 64
    8 2 1 7 9
#### 输出样例

    2
    1
#### 分析：
 

 - 初看这个题，校检值可以排序后枚举最前最后来计算，但是怎么知道应该计算的范围呢？即每段区间。
 - 我们可以用二分来解决，二分枚举区间范围，然后计算。但是二分每次计算的都是大范围的一半，然后每次都要重复排序，所以时间会超限。
 - 我们就想到，可以倒着来，用倍增，每次枚举一，排好序计算如果满足条件就递增距离就乘以二，然后区间长度加上他，直到k+1是枚举的递增的长度为len时，刚好不满足校检值小于t，那么就从k开始枚举递增len的一半直到不能枚举。
 - 这样的话，还有一个好处，就是每次枚举递增的一段，我们排好序以后，再枚举递增的下一段的时候，我们可以用归并排序的思想，只要把后面的刚加入的一段用快排排好，然后两端合并即可，这样会大大减少时间复杂度。


#### 代码

```
#include <iostream>
#include <cmath>
#include <algorithm>
using namespace std;
typedef long long ll;
#define fiir(i,a,b) for(int i=a;i<=b;i++)
const int N=500005;
ll arr[N],temp[N],b[N],n,m,t,ans;

void merger(int l,int mid,int r){
	int x=l,y=mid;
	fiir(i,l,r) {
		if(y>r||(x<mid&&temp[x]<=temp[y])){
			b[i]=temp[x++];
		}else{
			b[i]=temp[y++];
		}
	}
}

int check(int l,int mid,int r){
	fiir(i,mid,r) temp[i]=arr[i];
	sort(temp+mid,temp+r+1);
	merger(l,mid,r);
	ll kk=0;
	for(int i=0;i<(r-l+1)/2&&i<m;i++) kk+=(ll)pow(b[r-i]-b[l+i],2);
	if(kk>t) return 0;
	else {
		fiir(i,l,r) temp[i]=b[i];
		return 1;
	}
}

void work(int l,int r){
	temp[l]=arr[l];
	int len=1;
	while(r<=n){
		if(!len){
			len=1;
			ans++;
			l=++r;
			temp[l]=arr[l];
		}else if(r+len<=n&&check(l,r+1,r+len)){
			r+=len;
			len<<=1;
			if(r==n) break;
		}else{
			len>>=1;
		}
	}
	if(r==n) ans++;
//	fiir(i,1,n) cout<<temp[i]<<" ";cout<<endl;
}

int main()
{
	ios::sync_with_stdio(false);
	int k;
	cin>>k;
	while(k--){
		ans=0;
		cin>>n>>m>>t;
		fiir(i,1,n) cin>>arr[i];
		work(1,1);
		cout<<ans<<endl;
	}
	return 0;
}
```
