---
title: "Perl实现可变字节码算法
	"
date: 2014-05-04
layout: post
description: "最近在阅读《大规模Web服务开发技术》一书，作为练习和鞭策，决定系统地写一写博客，加深理解。本文用Perl实现了可变字节码算法，对递增的整数序列的文本进行了压缩，压缩率能达到59%。
	     "
tags: [可变字节码,Variable Byte Code,Perl,压缩,算法]
---
##可变字节码的定义

```
数值	                  32位二进制			    可变字节码
5		00000000 00000000 00000000 00000101 	     10000101
130		00000000 00000000 00000000 10000010	  00000001 10000010
```
可变字节码，将整数序列中的冗余的高位0都去掉，其中130的可变字节码为00000001 10000010，前面8位中的1表示1*128，后面8位中最高位的1表示，这是最终的一个字节，代表数字的结束。
所以130 = 1*128 + 2 （最后字节中的10），而 5 = 101（二进制）
可以看出，数值越小，可变字节码位数越少。因此对于递增的整数序列[1,2,12,20,39],可以只存储相邻值的差，从而减少了数值的大小。处理后变为[1,1,10,8,19]

##Perl实现的原理

###测试数据的生成
data_gen.pl > data.txt

```perl
#!/usr/bin/perl

while (1) {
  print "1,2,3,4,5,128,130,258,300,512,568,1024\n";
}
```

###压缩 
1. 对输入的数值n，与128比较。
2. 小于128，则直接用pack函数输出二进制到文件中；
3. 大于128，则循环，将n%128的结果插入到数组的开头（unshift）；
4. 最后，在数组的最后一位加上128，表示这是最终的一个字节。
并将数组pack成字符（1个4Byte的int，被压缩成了2Byte的二进制），输出到二进制文件中

encode.pl程序如下：
./encode.pl data.txt > data.bin

```perl
#!/usr/bin/perl
use strict;
use warnings;
use integer;

while(<>){
        chomp;
        my $pre = 0;
        my $output;
        foreach my $num ( split ',',$_ ){
                $output .= encode($num-$pre);
                $pre = $num;
        }
        print $output."\n";
}
 
sub encode {
	my $num = shift;
	my @bytes;

	while (1) {
        unshift @bytes, $num % 128;
        last if ($num < 128);
        $num = $num/128;
	}
	$bytes[-1] += 128; #add 1 sign to the last byte(8 bits)
	return pack('C*',@bytes);
}
```

###解压
1. 对输入的二进制流进行unpack
2. 数值 = 高8位*128 + 低8位-128 （减去结束字节的标志）
3. 整数序列中[a,b,c,d],还原后变成[a,a+b,(a+b)+c,(a+b+c)+d]

./decode.pl data.bin > data.restore

```perl
#!/usr/bin/perl
use strict;
use warnings;

while(<>){
        chomp;
        my $pre =0;
        my @result;
        foreach my $c (decode($_)) {
                push @result, $pre+$c;
                $pre += $c;
        }
        print join ("," ,@result) ,"\n";
}

sub decode {
	my $stream = shift;
	my $n = 0;
	my @arr;
	foreach my $c ( unpack('C*',$stream) ){
		if ($c < 128){
			$n = 128 * $n + $c;		
		}else{
			push @arr,128*$n + ($c-128);
			$n = 0;
		}
	}
	return wantarray? @arr : $arr[0]; #根据调用的环境，决定返回数组还是标量
}
```
###压缩的效果图

![setting](/media/file/20140504/process.jpg)

![setting](/media/file/20140504/result.jpg)

文件已经从399M压缩到了163M，压缩率为(399-163)/399 = 0.59


At 1:03 AM

##参考资料
* 《大规模Web服务开发技术》
