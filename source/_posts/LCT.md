---
title: 'Link Cut Tree 总结'
date: 2021-01-01 21:43:38
tags: [LCT]
categories:
- 📚算法笔记
---


维护一颗森林，支持断边/连边，维护链上信息、连通性等。

主要思想是实链剖分，有虚边和实边，一些实边构成实链并由一颗 Splay 维护，Splay 根结点指向链顶的父亲，这条是虚边：认父不认子。这样一陀构成了若干个辅助树，辅助树有且仅有一个点没父亲，改点为辅助树的根。

<!--more-->

## 写代码时注意事项

1. 与普通 Splay 写法不同，`Rotate` 中先判断是否到根：

   ```cpp
   inline void Rotate(int x) {
       int y=f[x],z=f[y],k=Get(x);
       if(!IsRoot(y)) ch[z][Get(y)]=x; // 注意这个 IsRoot(y) 放在最前面
       ch[y][k]=ch[x][k^1]; f[ch[x][k^1]]=y;
       ch[x][k^1]=y; f[y]=x; f[x]=z;
       PushUp(y); PushUp(x);
   }
   ```

2. `Splay` 前要 `Updata` 将所有标记从根到下依次下传：

   ```cpp
   void Updata(int x) {
       if(!IsRoot(x)) Updata(f[x]);
       PushDown(x);
   }
   inline void Splay(int x) {
       Updata(x);
       for(int fa;fa=f[x],!IsRoot(x);Rotate(x)) {
           if(!IsRoot(fa)) Rotate(Get(fa)==Get(x)?fa:x);
       }
       PushUp(x);
   }
   ```

3. **Link & Cut** 判断是否合法：
    Link 比较简单，只要判断是否连通即可。`MakeRoot & FindRoot` 
    Cut 要保证 $u,v$ 间有边：连通 & 实边 & $u,v$点间无其他点

    ```cpp
    inline bool Cut(int x,int y) { 
        MakeRoot(x);
        if(FindRoot(y)==x&&f[y]==x&&!ch[y][0]) {
            f[y]=ch[x][1]=0; PushUp(x);
            return true;
        }
        return false;
    }
    ```

4. 干大事前先 `MakeRoot`  ，链上操作先 `Split` 提取链，再在改链 Splay 根上打标记。
5. 访问孩子/更改孩子 注意 `PushDown,PushUp` 

## 维护信息

Link-Cut-Tree 的复杂度是 $O(n\log n)$ 的，比树剖少一只 $\log n$ ，但同时常数也变大了。 

### 维护树链

先 `Split` 提取链，然后再在这个 Splay 的根上操作 (打标记/求值)

### 维护是否连通

Link-Cut-Tree 基操，`Link-Cut-FindRoot` 

### 维护双连通分量

只处理加边修改，维护双连通分量。考虑不连通的点直接 Link，连通的点将这条链提取出来暴力缩点，用并查集把这些点合并，且以该链 Splay 的根作为代表。

```cpp
void Del(int x,int y) { // 递归缩点,merge(x,y) 并查集上合并
    if(x) merge(x,y),Del(ch[x][0],y),Del(ch[x][1],y);
}
void Merge(int x,int y) { // 连边 x,y 维护双连通分量
    x=find(x); y=find(y);
    if(x==y) return; // 本就在同一连通分量里无需操作
    MakeRoot(x); // 这里相当于 Link
    if(FindRoot(y)!=x) {
        f[x]=y; return; 
    } 
    Del(ch[x][1],x); ch[x][1]=0;  // Link 不成功（已经连通）需要暴力缩点
    PushUp(x); // 该过孩子需要 PushUp
}
```

再其它操作时 **注意要 `x=find(x)`**

### 维护边权

Link-Cut-Tree 本身不能维护边权，因为边是不固定的，不能用某点指代。于是拆边为点，另建虚点，注意这样 Link/Cut 都需要两次。

### 维护子树

注意到 Link-Cut-Tree 每个点最多两个实儿子，其他都是虚儿子，若要维护子树需要将额外记录每个点虚儿子的信息，且在改变虚实边时要更新虚儿子信息。

1. 维护的信息要有 可减性，在改变链的虚实时需要 加上/减去 贡献
2. 在统计子树信息时一定将其作为根节点。**`MakeRoot`**
3. 如果维护的信息没有可减性，如维护区间最值，可以对每个结点开一个平衡树维护结点的虚子树中的最值。

```cpp
// 以维护子树大小为例：
// siz2 表示所有虚子树结点数,若链的虚实不变则不会改变 
inline void PushUp(int x) {if(x) siz[x]=siz[ls]+siz[rs]+1+siz2[x];}
inline void Access(int x) {
    for(int p=0;x;p=x,x=f[x]) {
        Splay(x); siz2[x]+=siz[rs]-siz[p]; rs=p; PushUp(x); // !!! 改变了虚/实链 siz2 要更新!!
    }
}
inline bool Link(int x,int y) { // !!! 连的是虚边，故需要改 siz2! 
    MakeRoot(x);
    if(FindRoot(y)==x) return false;
    MakeRoot(y); // !!! 这句不加会错。。。
    f[x]=y; siz2[y]+=siz[x]; PushUp(y);
    return true; 
}
// 对于 Cut :断的是实边，PushUp会维护更改，无需改动
```
