---
title: 从1打印到最大的N位数
layout: post
description: 从1打印到最大的N位数, 当N=3时，即打印1,2,3,4...999。...
tags: [C,算法,tech]
date: 2014-03-14 23:10
---


###Problem: 从1打印到最大的N位数, 当N=3时，即打印1,2,3,4...999

可以用一个循环，循环999次，依次打印；但是，如果变量定义为int类型时，32位的机子上，最大的unsigned数是2^32 -1, 最大的signed数是 2^31 -1 （更小，有一半为负数），这时，当最大的数超过2^32 -1时，就会溢出。
 
解决的办法是，申请一个长度为N+1的字符串，最后一位是结束符'\0'。初始化字符串的每一位都为0；然后循环打印，每次将字符串所代表的数字加1；直到最高位有进位时(999+1, 代表到达最大的N位数)，停止；
 
具体程序如下：

```c
void Print1ToMaxOfNDigits ( int n);
bool increament (char * num, int n);
void PrintNum (char * num);
 
int main(){
      Print1ToMaxOfNDigits(3);
    system("pause");
    return 0;
}
 
void Print1ToMaxOfNDigits ( int n){
    if (n<0)
        return;
    char num[n+1];
    int index = 0;
    for (;index < n;index++)
        num[index]='0';
    num[n] = '\0';
     
    /* 如果最高位，没有进位，继续打印并加1 */
    while (!increament(num,n)){  
        PrintNum (num);
    }
}
 
/*将字符串所代表的数加1, 如最高位有进位，则返回true，否则false*/
bool increament (char * num, int n){
    int takeOver = 0;  /* 进位标志 */
    bool overFlow = false;
    int index;
    for (index=n-1;index>=0;index--){
        int sum = num[index] - '0' + takeOver;
        if (index==n-1)
            sum++;
        if (sum>=10){
            sum -=10;
            takeOver =1;
            num[index]= sum + '0';
            if (index == 0)
                overFlow = true;
        }else{
            num[index]= sum + '0';
            break;    /* 没有进位时，终止本次函数的调用 */
        }
 
    }
    return overFlow;
}
 
/* 打印时，去掉最前面的0，如009变成9 */
void PrintNum (char * num){
    while ( *num!='\0' && *num =='0' )
        num++;
    printf ("%s,",num);
}
```


At 23:10
