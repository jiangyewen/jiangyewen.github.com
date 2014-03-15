---
title: ubuntu在vmware下共享windows 8文件夹
layout: post
tags:
  - ubuntu
  - vmware
  - tech
---

以前一直在使用windows+ubuntu双系统，但是总是为了某个软件来回切换重启。所以还是回到了入门linux时用的vmware。
那时2个系统共享文件是用的SftpDrive，将Ubuntu的文件系统挂载到windows下。今天要分享的如何将Windows下文件夹
共享到Ubuntu，使其在Ubuntu虚拟机内也可以直接使用文件。

环境：

<pre>
* host:  windows 8.1 64bit
* guest: Ubuntu 13.10 32bit
* tools: vmware 10 Download [here](http://pan.baidu.com/s/1eQGanOA)
</pre>

安装好vmware

安装ubuntu 虚拟机

安装好后，在vmware左边面板选择 My Computer->ubuntu->Settings->options->shared folders->enable->add。

点击browser,选择windows下的文件夹，设置如下图所示	
![setting](/media/file/20140315/1.jpg)

到/mnt/hgfs下查看刚才选择的共享文件夹
![hgfs](/media/file/20140315/2.jpg)

为了方便，创建Symbolic link（软链接）到Ubuntu桌面

<pre>
ln -s /home/frank/Desktop/winDesktop /mnt/hgfs/Desktop
ln -s /home/frank/Desktop/winDownloads /mnt/hgfs/Downloads
</pre>

最后效果图：
![result](/media/file/20140315/3.jpg)


At 20:37
