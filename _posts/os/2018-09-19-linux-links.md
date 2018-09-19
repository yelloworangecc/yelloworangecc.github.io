---
layout: post
title:  "linux的文件链接"
summary: "linux 的cp命令可以通过-l和-s分别创建硬链接和软链接(符号链接),链接的实质是目录中的展位符,通过创建链接可以在linux文件系统中维护同一个文件的多份拷贝(一个物理文件,多个虚拟文件),那么硬链接与软连接有什么区别呢?"
featured-img: os
# comments: true
---
linux 的cp命令可以通过-l和-s分别创建硬链接和软链接(符号链接),链接的实质是目录中的展位符,通过创建链接可以在linux文件系统中维护同一个文件的多份拷贝(一个物理文件,多个虚拟文件),那么硬链接与软连接有什么区别呢?

## 硬链接 ##
```
$ cp -l test1 test4
$ ls -il
total 16
1954886 drwxr-xr-x 2 rich rich 4096 Sep 1 09:42 dir1/
1954889 drwxr-xr-x 2 rich rich 4096 Sep 1 09:45 dir2/
1954793 -rw-r--r-- 2 rich rich 0 Sep 1 09:51 test1
1954794 -rw-r--r-- 1 rich rich 0 Sep 1 09:39 test2
1954888 -rw-r--r-- 1 rich rich 0 Dec 25 2008 test3
1954793 -rw-r--r-- 2 rich rich 0 Sep 1 09:51 test4
$
```
这个例子中,test4是test1的一个硬链接,通过在ls的-il选项我们可以查看实际文件的inode索引,由此可见硬链接与源文件具有相同的inode索引,这说明test1和test4实际上指示的是同一个文件.

## 软连接 ##
```
$ cp -s test1 test5
$ ls -il test*
total 16
1954793 -rw-r--r-- 2 rich rich 6 Sep 1 09:51 test1
1954794 -rw-r--r-- 1 rich rich 0 Sep 1 09:39 test2
1954888 -rw-r--r-- 1 rich rich 0 Dec 25 2008 test3
1954793 -rw-r--r-- 2 rich rich 6 Sep 1 09:51 test4
1954891 lrwxrwxrwx 1 rich rich 5 Sep 1 09:56 test5 -> test1
$
```
在这个例子中,test5是test1的一个软链接,与test4对比可见,软链接的inode索引不同,文件类型和权限不同,文件大小也不同,实质上软链接是一个新文件,文件的内容保存了关于源文件的信息而不是实际内容,通过这些信息文件系统可以跳转到源文件.

## 注意 ##
硬链接只能在同一个物理介质上创建,也就是说,硬链接和源文件要在同一个挂载点上.软链接则没有这种限制.
