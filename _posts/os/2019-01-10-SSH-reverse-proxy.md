---
layout: post
title:  "ssh反向代理配置"
summary: "利用树莓派和云主机搭建反向代理访问家庭网络"
featured-img: os
# comments: true
---

## 环境描述 ##
1. raspi:家庭网络中的raspberry pi主机
2. cloud:腾讯云主机
3. test:单位里的主机,用于测试

## 秘钥配置 ##
在raspi主机上运行ssh-keygen生成秘钥.
```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
```
将生成的id_rsa.pub文件中的内容(公钥)添加到cloud主机的`~/.ssh/authorized_keys`,如果没有该文件请新建.
查看cloud主机的`/etc/ssh/sshd_config`文件,确保公钥认证功能开启,本文环境中是默认开启的,如果为关闭状态,需要去掉#号注释并做修改.
```
#PubkeyAuthentication yes
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
```
这样建立ssh反向链接时就不在需要输入密码了.

## 开启GatewayPorts ##
继续修改sshd_config文件.
```
$ sudo emacs /etc/ssh/sshd_config
```
开启GatewayPorts.
```
GatewayPorts yes
```
重启sshd服务.
```
$ service sshd restart
```

## 建立反向链接 ##
在raspi主机上运行命令建立反向链接.
```
$ ssh -NR *:3333:localhost:22 ubuntu@172.81.237.92
```
这样我就可以在test主机上通过172.81.237.92的3333端口访问raspi主机上的ssh服务了.
