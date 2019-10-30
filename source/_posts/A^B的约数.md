---
title: 约数之和
top: false
cover: false
toc: true
mathjax: true
date: 2019-8-29 15:28:37
password:
summary: 求A^B所有约数之和对9901取模的值。
tags: 递归
categories: 算法
---
#### 题目

`A^B所有约数之和对9901取模的值。`

#### 输入 
```
2 3
```
#### 输出

```
15 
```

#### 解析 

```
A= p1^k1 *p2^k2 *p3^k *...* pn^kn
A^B= p1^k1B *p2^k2B *p3^k3B *... *pn^knB
```
 - 因为p1取( p1^0...k1)  都是A^B的约数。
而( pi^c *pj^d )也是A^B的约数。(所有最简约数的乘积也是他的约数)
因此A^B的约数之和等于

```
 (p1^0 +p1^1 +... +pn^k1) * (p2^0 +p2^1 +.... +p2^k2) *... 
 *(pn^0 +pn^1 +... +pn^kn);
```

 - 计算 
 - 当k为奇数时 

```
sum(p,k)= p^0 +p^1 +... +p^k
        =( p^0 +p^1 +...+p^(k/2)) +(p^(k/2+1) +...p^k);
        =( p^0 +p^1 +...+p^(k/2)) +p^(k/2+1) (p^0 +... p^(k/2));
        =( 1+ p^(k/2+1)) *( p^0 +p^1 +... +p^(k/2))
        =(1 +p^(k/2+1)) *sum(p, k/2);
```

  - 当k为偶数时 
 

```
sum(p,k)=sum(p,k-1)*p+1    
```

 -  (相当于把指数都减1   然后再乘以p相当于指数加一，再加上p^0=1); 



#### 代码

```
#include <iostream>
using namespace std;
const int mod=9901;
int A,B; 

int kuaisumi(int p,int k){
	p%=mod;
	int ans=1;
	while(k){
		if(k&1) ans=ans*p%mod;
		p=p*p%mod;
		k>>=1;
	}
	return ans;
}

int sum(int p,int k){
	if(k==0) return 1;
	if(k%2==0) return (p%mod*sum(p,k-1)+1)%mod;
	return (1+kuaisumi(p,k/2+1))*sum(p,k/2)%mod;
}

int main()
{
	cin>>A>>B;
	int ans=1;
	for(int i=2;i<=A;i++){
		int s=0;
		while(A%i==0){
			s++;
			A/=i;
		}
//		cout<<i<<" "<<s<<" "<<B<<endl;
		if(s) ans=ans*sum(i,s*B)%mod;
	} 
	if(A) cout<<ans<<endl;
	else cout<<0<<endl;
	return 0;
}
```