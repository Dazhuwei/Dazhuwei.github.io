---
title: "解决inode占用过多的解决方案"
date: 2021-11-04T18:29:46+08:00
draft: true
tags: [操作系统,linux]
categories: [Linux]
featuredImage: "/images/linux-images.jpg"
featuredImagePreview: "/images/linux-images.jpg"
---


# 解决inode占用过多的解决方案

## 前情提要：

​	某台服务器inode满了，很久没有处理过类似问题了，有点忘记，记录在此。
# 一、理解inode

​	要理解inode，要从文件存储说起，Linux系统文件在物理上都是存储在硬盘上面的，硬盘存储里面，最小存储单位是"扇区(Sector)"，每个扇区存储512字节。
​	
​	操作系统在读取硬盘的时候不会一个一个读取扇区，这样效率很低，而是一次性连续读取多个扇区，多个扇区就组成了一个块(block)，而这种由多个扇区组成的块，就是文件存储的最小单位，块的大小一般为4KB，也就是说8个扇区组成了一个block。

​	文件信息存储在block中，如何找到这些block，以及怎么知道这些block存储了哪些内容，如文件的创建人、时间、大小等信息，这些信息又叫元数据，而这种存储元数据的区域就叫做inode，inode就是索引节点。

​	也就是说每个文件都需要记录这些元信息，也必然会占用inode，因此inode占用过多，多数是小文件太多导致。

# 二、查找哪里占用inode

​	查看 / 目录 inode满了，导致服务无法正常重启，并且存在大量的僵尸进程造成内存耗尽，知道了inode占用多是文件多导致之后，那么只需要找到哪些目录下小文件过多，然后删除即可

### **1.查看哪个盘占用inode**

```shell
df -ih 
df -ia
两个命令均可
top  ##查看异常使用进程
```
![20818113-8B7E-42DB-9113-529A0D98592B](/images/inode-01.png)

![F02B5AB5-C3D1-4462-9665-782798F58E62](/images/inode-02.png)

### 2.进入该挂载目录，然后通过wc -l统计哪些占用多。

### 3.可以看到是/var/spool/postfix/maildrop下很多小文件

### 4.解决办法：执行以下命令

```shell
ps -ef | egrep "sendmail|postdrop" | grep -v grep |xargs kill 
或者
killall postdrop
```

​	查到了/var/spool/postfix/maildrop目录下有大量小文件，原来是crond在执行脚本时，会将保持信息以邮件的形式发送给crond用户，而环境的postfix没有正常运行，导致邮件发送失败，都会堆积在/var/spool/postfix/maildrop/目录中，要解决该问题，一是可以启动postfix，让邮件服务正常运行.另外还可以在/etc/crontab中修改配置MAILTO=""发送为空，这样就不会堆积了。



​		删除占用inode的文件

```shell
find /var/spool/postfix/maildrop/ -type f |xargs rm -rf
```

####  ps：

​	建议正式环境非工作时间操作，避免影响服务器性能。

​	遍历查找数据盘inode占用最多的目录

```shell
find /data -xdev -printf '%h\n' | sort | uniq -c | sort -k 1 -n
## /data 目录根据需求进行替换
```

![9B653340-607B-407C-A183-B05C43CAEBA9](/images/inode-03.png)




