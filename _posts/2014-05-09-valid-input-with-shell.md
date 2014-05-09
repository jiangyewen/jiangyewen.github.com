---
title: "Shell对输入参数的验证"
layout: post
description: "本文实现了对Shell输入参数的验证，包括个数和数据类型。学习如何接收shell函数的返回值以及正则表达式。
	     "
tags: [输入验证,Shell,Wicked Cool Shell Scripts]
---
##shell函数的返回值
函数的返回值分为2两种：

* 函数中的echo的标准输出
* 函数中显式的return返回值

举例：

```bash
add () {
	result=$[ $1 + $2 ]
	return $result
}

add_2 () {
	result=$[ $1 + $2 ]
	echo $result
}
```
add函数的返回值，接收方式为

```bash
	add 1 2
	result=$?
```
即return的返回值，只能在函数执行后，根据$?状态判断。
而add_2的接收方式为

```bash
	result=$(add 1 2)
```
即echo的返回值可直接从子语句中接收

##shell对输入参数的验证
本文要实现目的：如果输入参数全为数字，则显示valid;其他全部为invalid。

思路：先保存输入数据到一个变量，接着对数据进行正则表达式过滤，过滤掉非数字的部分，再与原来的数据对比，一致则说明全部是数据，否则包含了其他字母或者标点。

实现的代码如下：

```bash
#!/bin/bash
 valid_input (){
        input=$1
        compressed=$(echo $input | sed -e "s/[^[:digit:]]//g" );
        if [ "$compressed" != "$input" ];then
                return 1
        else
                return 0
        fi
}
        read -p "input your value here>" input
	valid_input $input
        if  [ $? -eq 1 ];then
                echo "invalid"
        else
                echo "valid"
        fi
        exit 0
```
其中，[:digit:]表示数字，可根据需求换成其他，如[:alnum:]表示数字和字母；[^[:digit:]] 表示否定，即非数字；
另外，函数执行是否成功，可以用下面的方法，简写：

```bash
	read -p "input your value here>" input
        if  ! valid_input "$input";then
                echo "invalid"
        else
                echo "valid"
        fi
        exit 0
```
if fun_name表示，函数成功后（返回0），if为真；否则if为假。
if ! fun_name表示，函数不成功（返回不为0），if为真。
这和上面的有一点冲突，只需要记住 if认为后面的函数返回为0时，条件才为真。

##执行结果

```bash
sh valid_input.sh 
input>frank
invalid

sh valid_input.sh 
input>11111
valid

sh valid_input.sh 
input>1111frank
invalid

```
At 11:00 PM

##参考资料
* Wicked.Cool.Shell.Scripts
* http://blog.csdn.net/mdx20072419/article/details/9381339
