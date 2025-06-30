---
title: P13020 [GESP202506 八级] 遍历计数 题解
date: 2025-6-30 19:18:06
top_img: /img/2025-03-23-20-48-53.png
cover: /img/2025-03-23-20-48-53.png
type: post
categories:
  - 题解
tags:
  - 题解
---
# 原题
## 题目描述

给定一棵有 $n$ 个结点的树 $T$，结点依次以 $1,2,\dots,n$ 标号。树 $T$ 的深度优先遍历序可由以下过程得到：

1. 选定深度优先遍历的起点 $s$（$1 \leq s \leq n$），当前位置结点即是起点。
2. 若当前结点存在未被遍历的相邻结点 $u$ 则遍历 $u$，也即令当前位置结点为 $u$ 并重复这一步；否则回溯。
3. 按照遍历结点的顺序依次写下结点编号，即可得到一组深度优先遍历序。

第一步中起点的选择是任意的，并且第二步中遍历相邻结点的顺序也是任意的，因此对于同一棵树 $T$ 可能有多组不同的深度优先遍历序。请你求出树 $T$ 有多少组不同的深度优先遍历序。由于答案可能很大，你只需要求出答案对 $10^9$ 取模之后的结果。

## 输入格式

第一行，一个整数 $n$，表示树 $T$ 的结点数。

接下来 $n-1$ 行，每行两个正整数 $u_i, v_i$，表示树 $T$ 中的一条连接结点 $u_i, v_i$ 的边。

## 输出格式

输出一行，一个整数，表示树 $T$ 的不同的深度优先遍历序数量对 $10^9$ 取模的结果。

## 输入输出样例 #1

### 输入 #1

```
4
1 2
2 3
3 4
```

### 输出 #1

```
6
```

## 输入输出样例 #2

### 输入 #2

```
8
1 2
1 3
1 4
2 5
2 6
3 7
3 8
```

### 输出 #2

```
112
```

## 说明/提示

对于 40% 的测试点，保证 $1 \leq n \leq 8$。

对于另外 20% 的测试点，保证给定的树是一条链。

对于所有测试点，保证 $1 \leq n \leq 10^5$。
# 解

题目表面上说是多少种深搜的序列，实际上深搜不是重点，重点是求排列数。

**40分代码**

对于每一个节点，假设它有 $n$ 个子节点，那么可以选择的，抛开子节点的子节点不看，遍历方法就有 $A_n^n$ 即 $n!$ 种。

因此，总的方案数就是所有的节点的子节点数阶乘的总乘积。

时间复杂度 $O(n^2)$ 。

```cpp
#include<bits/stdc++.h>
using namespace std;
int n,u,v,ans,sum;
const int N=1e5+5;
vector<int>g[N];//邻接表方便求连边数
int f[N];
const int mod=1e9;
bool vis[N];
void dfs(int x,int fa){
    //如果是起始节点，子节点数就是连边数，否则要减去和父节点相连的一条边
    ans=(ans*f[(fa==-1?g[x].size():g[x].size()-1)])%mod;
    for(auto i:g[x]){
        if(!vis[i]){
            vis[i]=1;
            dfs(i,x);
        }
    }
}
int main(){
    cin>>n;
    f[0]=1;
    //预处理阶乘数
    for(int i=1;i<=n;i++){
        f[i]=(f[i-1]*i)%mod;
    }
    for(int i=1;i<n;i++){
        cin>>u>>v;
        g[u].push_back(v);
        g[v].push_back(u);
    }
    //每一个节点都可能是起始节点
    for(int i=1;i<=n;i++){
        memset(vis,0,sizeof(vis));
        vis[i]=1;
        ans=1;
        dfs(i,-1);
        sum=(sum+ans)%mod;
    }
    cout<<sum;
    return 0;
}
```
**60分代码**

观察每次的乘积可知，每次所有的子节点都乘以相同的因数（连边数-1），所以可以一次性算出来，由于起始节点要乘 $n!$ 而不是 $(n-1)!$ ，所以再乘 $n$ 即可。
时间复杂度 $O(n)$ 。

```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
int n,u,v,mul=1,sum;
const int N=1e5+5;
vector<int>g[N];
int f[N];
const int mod=1e9;
bool vis[N];
void dfs(int x){
    mul=(mul*f[g[x].size()-1])%mod;
    for(auto i:g[x]){
        if(!vis[i]){
            vis[i]=1;
            dfs(i);
        }
    }
}
signed main(){
    cin>>n;
    f[0]=1;
    for(int i=1;i<=n;i++){
        f[i]=(f[i-1]*i)%mod;
    }
    for(int i=1;i<n;i++){
        cin>>u>>v;
        g[u].push_back(v);
        g[v].push_back(u);
    }
    vis[1]=1;
    dfs(1);
    for(int i=1;i<=n;i++){
        int res=(mul*g[i].size())%mod;
        sum=(sum+res)%mod;
    }
    cout<<sum;
    return 0;
}
```
**AC代码**

考虑特殊情况，当 $n=1$ 时，由于这个节点没有子节点所以连边数减 $1$ 会得到 $0$ ，造成 `mul` 得到 $0$。
所以再加一个特判得全分。
```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
int n,u,v,mul=1,sum;
const int N=1e5+5;
vector<int>g[N];
int f[N];
const int mod=1e9;
bool vis[N];
void dfs(int x){
    mul=(mul*f[g[x].size()-1])%mod;
    for(auto i:g[x]){
        if(!vis[i]){
            vis[i]=1;
            dfs(i);
        }
    }
}
signed main(){
    cin>>n;
    f[0]=1;
    for(int i=1;i<=n;i++){
        f[i]=(f[i-1]*i)%mod;
    }
    for(int i=1;i<n;i++){
        cin>>u>>v;
        g[u].push_back(v);
        g[v].push_back(u);
    }
    vis[1]=1;
    dfs(1);
    for(int i=1;i<=n;i++){
        int res=(mul*g[i].size())%mod;
        sum=(sum+res)%mod;
    }
    if(n==1)cout<<1;
    else{
        cout<<sum;
    }
    return 0;
}
```

