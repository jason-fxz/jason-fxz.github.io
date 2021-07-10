---
title: 'XJ - Remote Control'
date: 2020-11-21 20:00:18
tags: [XJ,模拟]
categories: 题解
hidden: true
---

<!--more-->
 
[XJ-Remote Control](https://dev.xjoi.net/contest/1597/problem/2)
 
------

## 题意
 
小 Y 有一辆遥控小车，它可以在平面上行驶。我们定义 $(x,y)$ 的上下左右分别为 $(x,y+1),(x,y−1),(x−1,y),(x+1,y)$，遥控小车接收到了四个方向之一的信号后，会朝着那个方向前进一步。在平面的 $(0,0)$ 处有个障碍物，如果遥控小车的下一步会到达障碍物，它会忽略这个操作。
 
小 Y 已经计划好了将会连续发送的信号 $s_1,s_2,\dots,s_n$。接着她向你提出 $Q$ 次询问：如果遥控小车从 $(x_i,y_i)$ 出发（保证不是 $(0,0)$） ，连续接收这些信号之后，它最终会停留在哪个位置。 （ $s_i$ 为一个字符 为 `U`,`D`,`L`或`R`，表上下左右 ）
 
$$1\le n,Q \le 10^5,-10^5 \le x_i,y_i\le 10^5$$
 
 
## 题解
 
考虑操作序列（信号序列），小车在碰到障碍物前已经走了一段，这是操作序列的一个前缀。在碰到障碍后，若明确剩下得操作序列，那么终点也是确定的。故我们考虑预处理出 ：*哪些点*走了一个*长度为多少*得操作序列前缀后到达点 $(0,0)$ ；在操作序列第 $i$ 处碰到障碍最后到*哪个点*
 
考虑如何预处理：
1. 对于第一个比较方便，我们从 $(0,0)$ 把操作序列反着（左右互换，上下互换）着走完，再从后往前撤销。
2. 对于第二个，同样从后往前考虑，考虑第 $i$ 步碰到 $(0,0)$ 第 $i$ 步之前的位置可以算出，再考虑后面到哪里，（要注意的是，碰到 $(0,0)$ 后可能还会再碰到，故要记录 $i$ 步以后那些位置会碰到 $(0,0)$ 具体见代码注释）

## CODE

预处理的细节挺多的，调了半天....

```cpp
#include <bits/stdc++.h>
using namespace std;
template<typename T>
inline void red(T &x) {
    x=0;bool fg=0;char ch=getchar();
    while(ch<'0'||ch>'9') {if(ch=='-') fg^=1;ch=getchar();}
    while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(T)(ch^48),ch=getchar();
    if(fg) x=-x;
}
const int N = 100010;
const int fig[4][2]={{0,1},{0,-1},{-1,0},{1,0}};
//                     U     D      L      R
int n,q;
int dx[N],dy[N],ct[4];
char s[N];
struct pt {
    int x,y;
    pt(int _x,int _y):x(_x),y(_y){}
    pt(){}
    bool operator<(const pt a)const {
        return (x==a.x)?(y<a.y):(x<a.x);
    }
};
// cr(pt) 表点 (x,y) 会在操作序列得第i步上撞墙
map<pt,int> cr,pr;
// go(i) 表点在第i步撞墙后何去何从
pt go[N];
 
int id(char op) {
    switch (op){
    case 'U': return 0;break;
    case 'D': return 1;break;
    case 'L': return 2;break;
    case 'R': return 3;break;
    default : return 100;break;
    }
}
 
void init() {
    for(int i=1;i<=n;i++) {
        dx[i]=dx[i-1]+fig[id(s[i])][0];
        dy[i]=dy[i-1]+fig[id(s[i])][1];
        ct[id(s[i])]++;
    }
}
void prework() {
    int tx=-dx[n],ty=-dy[n];
    for(int i=n;i>=1;i--) {
        cr[pt(tx,ty)]=i;
        tx+=fig[id(s[i])][0];
        ty+=fig[id(s[i])][1];
    }
    int ex=0,ey=0;
    // go[i] 表示第i步撞墙，之后到哪里
    for(int i=n;i>=1;i--) {
        int px=fig[id(s[i])^1][0],py=fig[id(s[i])^1][1];
        // 现在 pr 中得点为前i步走完，要走 i+1 及以后步数，哪些点会撞墙，且撞墙在第几步
        // pr 中得点为 实际 (x,y) 在 pr 中为 (ex+x,ey+y)
         
        // 现在走完i-1步在 px,py，第i步撞墙，第i步也在 (px,py)
 
        // case1 (ex+px,ey+py) 在 pr 中 那么 go[i]= pr[(ex+px,ey+py)]
        if(pr.find(pt(ex+px,ey+py))!=pr.end()) {
            go[i]=go[pr[pt(ex+px,ey+py)]];
        }else {
            // case2 (ex+px,ey+py) 不在 pr 中，那么 (px,py) 直接走 i+1 到 n 步即可
            go[i]=pt(px+dx[n]-dx[i],py+dy[n]-dy[i]);      
        }
        // 现在要更新 pr ，将 (px,py) 插入，从走完 i 步倒退到走完 i-1 步，即回退第 i 步
        // pr中的点 集体移动 id(s[i])^1，等价于其他点 移动 id(s[i]) ，对 ex，ey 更新
        ex+=fig[id(s[i])][0],ey+=fig[id(s[i])][1];
        // 再将 (px+ex,py+ey) 插入
        px+=ex,py+=ey;
        pr[pt(px,py)]=i;
    }
}
int main() {
    red(n);scanf("%s",s+1);init();
    prework();
    red(q);
    for(int i=1;i<=q;i++) {
        int x,y;red(x);red(y);
        if(cr.find(pt(x,y))!=cr.end()) {
            // 如果会碰到则按照预处理的来
            pt ed=go[cr[pt(x,y)]];
            printf("%d %d\n",ed.x,ed.y);
        }else {
            // 如果不会碰到直接走
            printf("%d %d\n",x+dx[n],y+dy[n]);
        }
    }
    return 0;
}
```