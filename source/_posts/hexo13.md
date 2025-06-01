---
title: P3435 [POI 2006] OKR-Periods of Words 题解
date: 2025-3-23 20:45:44
top_img: /img/2025-03-23-20-48-53.png
cover: /img/2025-03-23-20-48-53.png
type: "post"
categories:
  - 题解
tags:
  - 题解
---


# 题目描述

A string is a finite sequence of lower-case (non-capital) letters of the English alphabet. Particularly, it may be an empty sequence, i.e. a sequence of 0 letters. By A=BC we denotes that A is a string obtained by concatenation (joining by writing one immediately after another, i.e. without any space, etc.) of the strings B and C (in this order). A string P is a prefix of the string !, if there is a string B, that A=PB. In other words, prefixes of A are the initial fragments of A. In addition, if P!=A and P is not an empty string, we say, that P is a proper prefix of A.


A string Q is a period of Q, if Q is a proper prefix of A and A is a prefix (not necessarily a proper one) of the string QQ. For example, the strings abab and ababab are both periods of the string abababa. The maximum period of a string A is the longest of its periods or the empty string, if A doesn't have any period. For example, the maximum period of ababab is abab. The maximum period of abc is the empty string.

Task Write a programme that:

reads from the standard input the string's length and the string itself,calculates the sum of lengths of maximum periods of all its prefixes,writes the result to the standard output.

# 输入格式

In the first line of the standard input there is one integer $k$ ($1\le k\le 1\ 000\ 000$) - the length of the string. In the following line a sequence of exactly $k$ lower-case letters of the English alphabet is written - the string.

# 输出格式

In the first and only line of the standard output your programme should write an integer - the sum of lengths of maximum periods of all prefixes of the string given in the input.

# 输入输出样例 #1

## 输入 #1

```
8
babababa
```

## 输出 #1

```
24
```

# 题意翻译

对于一个仅含小写字母的字符串 $a$，$p$ 为 $a$ 的前缀且 $p\ne a$，那么我们称 $p$ 为 $a$ 的 $proper$ 前缀。

规定字符串 Q 表示 a 的周期，当且仅当 Q 是 a 的 proper 前缀且 a 是 Q+Q 的前缀。若这样的字符串不存在，则 a 的周期为空串。

例如 `ab` 是 `abab` 的一个周期，因为 `ab` 是 `abab` 的 proper 前缀，且 `abab` 是 `ab`+`ab` 的前缀。

求给定字符串所有前缀的最大周期长度之和。

# 解析

首先介绍一个叫$next$数组的东西
因为当匹配失败时，就会利用$next$数组往前跳，这样就能节省匹配次数
$next$简称$ne$.
对于一个字符串$s$ ，对应的 $ne_i$  定义为 $s$ 长度为 $i$ 的前缀的最长公共前后缀的长度。
例如，给定一个 $s =$ `abacabad`.

|  $i$   |  1  |  2  |  3  |  4  |  5  |  6  |  7  |  8  |
| :----: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| $s_i$  | `a` | `b` | `a` | `c` | `a` | `b` | `a` | `d` |
| $ne_i$ |  0  |  0  |  1  |  0  |  1  |  2  |  3  |  0  |

这段代码可以求出$ne$数组：
```cpp
ne[1]=0;
for(int i=2,j=0;i<=n;i++){
	while(j&&s[i]!=s[j+1])j=ne[j];
	if(s[i]==s[j+1])j++;
	ne[i]=j;
}
```

整个过程就是，如果能匹配，就一直往前进（`j++`）;否则，不断地利用之前的信息往回跳（`j=ne[j]`），直到可以匹配为止。

设 $Q_{max}$ 为字符串的最大周期。那么：

| $i$              | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   |
| ---------------- | --- | --- | --- | --- | --- | --- | --- | --- |
| $Q_{max}$        | `a` | `b` | `a` | `b` |     |     |     |     |
| $s$              | `a` | `b` | `a` | `b` | `a` | `b` |     |     |
| $Q_{max}Q_{max}$ | `a` | `b` | `a` | `b` | `a` | `b` | `a` | `b` |
所以，对于一个字符串 $s$ , 它的 $|Q_{max}|$ 就是$|s|-$它最短公共前后缀的长度。
最短公共前后缀的长度不好求，但是我们发现可以通过不断`j=ne[j]`求得。

于是就有了这样一段代码，一趟循环求得本题答案：
```cpp
for(int i=1;i<=n;i++){
	now=i;
	while(ne[now])now=ne[now];
	if(ne[i]!=0)ne[i]=now;
	ans+=i-now;
}
```

# 完整代码

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1000086;
int n,ne[N],now;
char s[N];
long long ans;
int main(){
    cin>>n>>s+1;
    ne[1]=0;
    for(int i=2,j=0;i<=n;i++){
        while(j&&s[i]!=s[j+1])j=ne[j];
        if(s[i]==s[j+1])j++;
        ne[i]=j;
    }
    for(int i=1;i<=n;i++){
        now=i;
        while(ne[now])now=ne[now];
        if(ne[i]!=0)ne[i]=now;
        ans+=i-now;
    }
    cout<<ans;
    return 0;
}
```