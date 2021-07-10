---
title: '五子棋'
date: 2019-08-21 14:53:01
tags: [📍小游戏]
categories:
- 杂
---
c++实现，控制台游戏。

<!--more-->
![PP](1582700600562.jpg)
调用了 `windows.h` 库，其中 `system(char)` 函数用于发出一个DOS命令，相当于在CMD中直接写命令。`MessageBox()` 是调用对话框。通过`_getch()`函数可以直接获取输入字符而不会显示。

## 代码

```cpp
#include <windows.h>
#include <bits/stdc++.h>
#include<conio.h>
const int n=19;
int map[30][30];
int x,y,sx=0,sy=2;
const int fig[9][2]={{0,0},{-1,0},{1,0},{0,-1},{0,1},{-1,-1},{1,1},{-1,1},{1,-1}};
void SetPos(int x,int y)
{
    COORD pos;
    HANDLE handle;
    pos.X=x;
    pos.Y=y;
    handle=GetStdHandle(STD_OUTPUT_HANDLE);
    SetConsoleCursorPosition(handle,pos);
}
void setColor(unsigned short ForeColor=7,unsigned short BackGroundColor=0){
    HANDLE handle=GetStdHandle(STD_OUTPUT_HANDLE);//获取当前窗口句柄
    SetConsoleTextAttribute(handle,ForeColor+BackGroundColor*0x10);//设置颜色
}
void end(bool player)
{
    if(player==0)
    MessageBox(NULL,"BLACK player wins!","WIN!",MB_OK);
    if(player==1)
    MessageBox(NULL,"WHITE player wins!","WIN!",MB_OK);
}
bool checkst(int x,int y)
{
    if(x<1||y<1||x>n||y>n) return 0;
    return 1;
}
int main()
{
    start:
    x=y=10;
    system("cls");
    system("color 68");
    system("title BLACK play");
    system("mode con cols=42 lines=23");
    setColor(0,6);
    std::cout<<"             ┌            ┐"<<std::endl;
    std::cout<<"             ├   五子棋   ┤"<<std::endl;
    std::cout<<"             └            ┘"<<std::endl;
    memset(map,0,sizeof(map));
    setColor(8,6);
    for(int i=1;i<=n;i++)
    {
        std::cout<<"  ";
        for(int i=1;i<=n;i++)
        std::cout<<"○";
        std::cout<<std::endl;
    }
    SetPos(sx+x*2,sy+y);
    int st=0;
    while(true)
    {
        char ch=_getch();
//        printf("%d",ch);
        if(ch=='a') if(!checkst(--x,y))x++;
        if(ch=='d') if(!checkst(++x,y))x--;
        if(ch=='w') if(!checkst(x,--y))y++;
        if(ch=='s') if(!checkst(x,++y))y--;
        SetPos(sx+x*2,sy+y);
        if(ch=='e')
        {
            if(map[x][y]==0)// st&1 blue else red
            {
                int t=st%2;
                if(st&1)
                {
                    system("title BLACK play");
                    setColor(15,6);
                    map[x][y]=2;
                }
                else
                {
                    system("title WHITE play");
                    setColor(0,6);
                    map[x][y]=1;
                }
                st++;
                std::cout<<"●";
                SetPos(sx+x*2,sy+y);
                int a[9]={0};
                for(int i=1;i<=8;i++)
                {
                    int tx=x,ty=y;
                    while(checkst(tx,ty)&&map[tx][ty]==t+1)
                    {
                        tx+=fig[i][0];
                        ty+=fig[i][1];
                        a[i]++;
                    }
                }
                for(int i=1;i<=4;i++)
                {
                    if(a[i*2]+a[i*2-1]-1>=5) {
                        end(t);
                        goto start;
                    }
                }
            }
            SetPos(sx+x*2,sy+y);
        }
    }   
}
```
