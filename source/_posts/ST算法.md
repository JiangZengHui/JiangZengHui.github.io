---
title: ST算法
top: false
cover: false
toc: true
mathjax: true
date: 2019-08-03 16:27:49
password:
summary: ST算法模板
tags: 模板
categories: 算法
---

#### ST算法初始化

 - dp[i][j]代表以i为起点，2^j 宽的最值 ，即以i+2^j-1结束。
 - arr[i]代表原数组。

```
void ST_prework(int n){
	fir(i,1,n) dp[i][0]=arr[i];	
	int k=log(n*1.0)/log(2.0);
	fir(j,1,k){
		fir(i,1,n-(1<<j)+1){
			dp[i][j]=min(dp[i][j-1],dp[i+(1<<(j-1))][j-1]);
		}
	}
} 
```

#### ST取值

 - 取出L到R的最值。

```
int ST_query(int l,int r){
	int k=log((r-l+1)*1.0)/log(2.0);
	return min(dp[l][k],dp[r-(1<<k)+1][k]); 
}
```
