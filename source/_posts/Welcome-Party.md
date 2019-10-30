---
title: Welcome Party
top: false
cover: false
toc: true
mathjax: true
date: 2019-08-16 22:22:30
password:
summary: The 44th World Finals of the International Collegiate Programming Contest (ICPC 2020) will be held in Moscow, Russia.
tags: 
	- 并查集
	- 优先队列
	- BFS
categories: 算法
---

#### Problem Description
The 44th World Finals of the International Collegiate Programming Contest (ICPC 2020) will be held in Moscow, Russia. To celebrate this annual event for the best competitive programmers around the world, it is decided to host a welcome party for all n participants of the World Finals, numbered from 1 to n for convenience.

The party will be held in a large hall. For security reasons, all participants must present their badge to the staff and pass a security check in order to be admitted into the hall. Due to the lack of equipment to perform the security check, it is decided to open only one entrance to the hall, and therefore only one person can enter the hall at a time.

Some participants are friends with each other. There are m pairs of mutual friendship relations. Needless to say, parties are more fun with friends. When a participant enters the hall, if he or she finds that none of his or her friends is in the hall, then that participant will be unhappy, even if his or her friends will be in the hall later. So, one big problem for the organizer is the order according to which participants enter the hall, as this will determine the number of unhappy participants. You are asked to find an order that minimizes the number of unhappy participants. Because participants with smaller numbers are more important (for example the ICPC director may get the number 1), if there are multiple such orders, you need to find the lexicographically smallest one, so that important participants enter the hall first.

Please note that if participant a and b are friends, and if participant b and c are friends, it's NOT necessary that participant a and c are friends.

#### Input
There are multiple test cases. The first line of the input contains a positive integer T, indicating the number of cases. For each test case:

The first line contains two integers n and m (1<=n,m<=10^6), the number of participants and the number of friendship relations.

The following m lines each contains two integers a and b (1<=a,b<=n,a≠b), indicating that the a-th and the b-th participant are friends. Each friendship pair is only described once in the input.

It is guaranteed that neither the sum of n nor the sum of m of all cases will exceed 10^6.

#### Output
For each case, print a single integer on the first line, indicating the minimum number of unhappy participants. On the second line, print a permutation of  to  separated by a space, indicating the lexicographically smallest ordering of participants entering the hall that achieves this minimum number.

Consider two orderings P=p1,p2,...,pn and Q=q1,q2,..,qn, we say P is lexicographically smaller than Q, if there exists an integer  k(1<=k<=n), such that  holds for all pi=qi,1<=i<k and pk<qk.

Please, DO NOT output extra spaces at the end of each line, or your solution may be considered incorrect!

#### Sample Input

    2
    4 3
    1 2
    1 3
    1 4
    4 2
    1 2
    3 4

#### Sample Output

    1
    1 2 3 4
    2
    1 2 3 4
#### 题意：
有n个人要参加聚会，1~n分别表示他们的序列，m行表示他们之间的关系，这n个人一次进场，若进场后发现没有自己的朋友，这个人就会不开心。现在要求排他们的入场顺序，使得不开心的人数最少，同时，进场人的字典序最小。
#### 分析：

 - 我们可以将每个人看成一个点，那么每队关系都是一条边，那么我们可以知道，肯定会有多个连通图，对于每个连通图，只要有一个人进入了，那么这个连通图的每个人进去的时候我们都有办法让他们开心，只要沿着当前进入的这个人后面他的朋友进去，然后她朋友的朋友再进，这就是一个广搜的过程。
 - 但是有多个联通图，所以我们可以将他们每个连通图的序号最小的人先加入联通图，然后再进行广搜即可。但是怎么保证字典序最小呢？
 - 我们可以使用优先队列，初始化的时候是每个连通图最小的点。然后模拟广搜过程，每当优先队列第一个人出列，就将他们的朋友加进来，记得标记进了队列。
 - 代码有详细注释。
#### 代码
 

```
#include<iostream>
#include<algorithm>
#include<queue>
#include<cstdio>
#include<cmath>
#include<vector>
using namespace std;

const int N=1000005;
int vis[N];
int root[N];
int arr[N];
vector<int> p[N];

int find(int x){
    if(x!=root[x]) root[x]=find(root[x]);
    return root[x];
}
void Union(int x,int y){
    int fx=find(x);
    int fy=find(y);
    if(fx<fy) root[fy]=fx;
    else root[fx]=fy;
}

int main(){
    int t;
    scanf("%d",&t);
    while(t--){
        priority_queue<int, vector<int>, greater<int> >q;
//      while(!q.empty()) q.pop();
        int n,m;
        scanf("%d%d",&n,&m);
        for(int i=0;i<=n;i++){  //初始化工作 
            root[i]=i;
            vis[i]=0;
            p[i].clear();
        } 
        int x,y;
        for(int i=1;i<=m;i++){
            scanf("%d%d",&x,&y);
            Union(x,y);         //并查集只是为了找出每个集合最小的点作为根节点 
            p[x].push_back(y);  //此处要注意编号大的人可以拥有编号小的作为朋友 
            p[y].push_back(x);  //编号小的也可以拥有编号大的作为朋友 
        }
        int cnt=0;
        for(int i=1;i<=n;i++){
            if(root[i]==i){
                q.push(i);      //将每个集合编号最小的根放入队列 
                cnt++;          //这些人包括那些没有任何朋友的人 
            } 
        }   
        int point=0;
        while(!q.empty()){
            int x=q.top();
            q.pop();
            if(vis[x]==0){//防止重复加点 
                vis[x]=1;
                arr[++point]=x;
                for(int i=0;i<p[x].size();i++){//遍历下标为x的每一个元素          
                    if(vis[p[x][i]]==0) q.push(p[x][i]);   
                    //如果这个朋友没有在之前就被排入队列则将它放入优先队列 
                } 
            }   
        }
        printf("%d\n",cnt);
        for(int i=1;i<=point;i++){
            if(i!=1) printf(" ");
            printf("%d",arr[i]);
        }
        printf("\n");
    }
    return 0;
}
```
