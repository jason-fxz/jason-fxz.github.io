---
title: '贪吃蛇'
date: 2019-08-30 15:03:41
tags: [📍小游戏]
categories:
- 杂
---
c++实现，控制台小游戏

<!--more-->

![PP](1582701676862.jpg)

## 代码

```cpp
#include <windows.h>
#include <conio.h>
#include <cstdio>
#include <ctime>
#include <cstdlib>
const int fig[4][2]={{-1,0},{1,0},{0,-1},{0,1}};
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
void HideCursor(int n) /*隐藏光标*/
{ 
    CONSOLE_CURSOR_INFO cursor_info={1,n};
    SetConsoleCursorInfo(GetStdHandle(STD_OUTPUT_HANDLE),&cursor_info);
} //n default 0
int n=29;
int mape[35][35];
int sx=2,sy=9,hx,hy,tx,ty,way,spe;
void drawstart()
{
    spe=150;
    memset(mape,0,sizeof(mape));
    srand(time(NULL));
    system("cls");//31*29
    system("color 0F");
    printf("           _________________________                       \n");
    printf("          / ________________________)     _               \n");
    printf("         / /                             | |  __        ●◆◆ \n");
    printf("         \\ \\____    _  ____     _____    | | / /   _____    ◆ \n");
    printf("          \\____ \\  | |/ __ \\   / ___ \\   | |/ /   / ___ \\   ◆ \n");
    printf("               \\ \\ | |_/  \\ \\ / /   \\ \\  | |\\ \\  / /___\\_\\  ◆ \n");
    printf("     __________/ / | |    | | \\ \\___/  \\ | | \\ \\ \\ \\______  ◆◆◆◆ \n");
    printf("    (___________/  |_|    |_|  \\_____/_/ |_|  \\_\\ \\______/   \n");
    printf("\n  ");
    for(int i=1;i<=33;i++)
    printf("■");
    for(int i=1;i<30;i++)
    printf("\n  ■                                                              ■");
    printf("\n  ");
    for(int i=1;i<=33;i++)
    printf("■");
}
void end()
{
    MessageBox(NULL,"GameOver","end",MB_OK);
}
int main()
{
    HideCursor(0);
    start:
    system("mode con cols=70 lines=40");
    drawstart();
    hx=1,hy=2,tx=1,ty=1,way=3;
    mape[1][1]=4;
    int st=GetTickCount();
    int last=GetTickCount();
    bool tt=1;
    bool cak=1;
    bool canch=1;
    int count=0;
    SetPos(63,4); printf("%2d",count); 
    while(true)
    {
        
        if(cak)
        {
            int x=rand()%25+2,y=rand()%24+2;
            while(mape[x][y]!=0)
            {
                x=rand()%29+1,y=y%29+1;
            }
            SetPos(sx+2*x,sy+y);
            mape[x][y]=-1;
            setColor(12,0);
            printf("●");
            cak=0;
        }
        if(_kbhit()&&canch) {
            char ch=_getch();
            if(ch=='w'&&way!=3) way=2;
            if(ch=='s'&&way!=2) way=3;
            if(ch=='a'&&way!=1) way=0;
            if(ch=='d'&&way!=0) way=1;
            if(ch=='e'&&tt) 
            {
                tt=0,count++,SetPos(63,4);
                printf("%2d",count);
            }
            if(ch==' ') {
                getchar();
            }
            canch=0;
        }
        if(st-last>spe)
        {
            last=GetTickCount();
            canch=1;
            setColor(15,0);
            if(mape[hx][hy]==-1) 
            {
                tt=0,cak=1;
                count++;
                SetPos(63,4); printf("%2d",count); 
                if(count%10==1) spe=spe*0.95;
            }
            if(mape[hx][hy]>0||hx<1||hx>31||hy<1||hy>29) {end();goto start;} 
            mape[hx][hy]=way+1;
            SetPos(sx+hx*2,sy+hy);printf("◆");
            hx+=fig[way][0],hy+=fig[way][1];
            SetPos(sx+hx*2,sy+hy);printf("●");
            int wway=mape[tx][ty]-1;
            st=0;
            if(tt)
            {
                SetPos(sx+tx*2,sy+ty);printf("  ");mape[tx][ty]=0;
                tx+=fig[wway][0],ty+=fig[wway][1];
            }
            
            tt=1;
        }
        st=GetTickCount();
    }
}
```
