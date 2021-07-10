---
title: test post
date: 2020-10-31 21:36:57
tags: 
- Hexo
categories:
- test
---


文章测试

<!--more-->

# H1
## H2
### H3

分割线


---

**粗体**  
*斜体*  
~~删除线~~  
> 引用

图片
![](https://static.oschina.net/uploads/img/201409/26074001_bzCh.gif)

链接 [link]("https://www.luogu.com.cn/user/89797")

有序列表

1. hello
2. world
3. nice to meet you

无序列表

- 第一项
- 第二项
    - 第二.一项
    - 第二.二项
- 第三项


代码块

```cpp
#include <bits/stdc++.h>
std::vector< std::pair<int,int> > a;
int rnd(int l,int r) {
    return rand()%(r-l+1)+l;
}
int main() {
    srand(clock()*1000);
    int n=rnd(5,10);
    printf("%d\n",n);
    for(int i=1;i<=n;i++) {
        printf("%d ",rnd(1,n));
    }printf("\n");
    for(int i=2;i<=n;i++) {
        // a.push_back(std::make_pair(i,rnd(1,i-1)));
        printf("%d %d\n",i,rnd(1,i-1));
    }
    return 0;
}
```

---


行内公式 $\LaTeX$ 

独立公式


$$J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \Gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha} \text {，独立公式示例}$$

