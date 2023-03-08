---
title: SSH无密码登录
categories: SSH
tags: SSH
date: 2023-03-08 18:46:17
---

环境: 本机Linux, 远程Linux
- 首先我们在本机Linux系统上生成一对SSH Key：SSH密钥和SSH公钥．密钥保存在本机Linux系统上。
- 然后公钥上传到远程Linux服务器．之后我们就能在本机Linux无密码SSH登录远程Linux了．SSH密钥就好比是你的身份证明．

### 本机Linux生成秘钥

在本机Linux命令行使用 `ssh-keygen` 生成密钥和公钥, 也可以添加参数:

```bash
ssh-keygen -b 4096 -t rsa
```

- -b是指定生成秘钥长度为4096, 默认是2048
- -t表示加密类型, 默认是RSA加密

生成SSH Key的过程中会要求你指定一个文件来保存密钥，按Enter键使用默认的文件就行了．然后需要输入一个密码来加密你的SSH Key．密码至少要5位长度．

生成过程如下:

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/home/.ssh/id_rsa): 　按Enter键
Enter passphrase (empty for no passphrase): 　　输入一个密码
Enter same passphrase again: 　　再次输入密码
Your identification has been saved in ~/.ssh/id_rsa.
Your public key has been saved in ~/.ssh/id_rsa.pub.
The key fingerprint is:
e1:dc:ab:ae:b6:19:b0:19:74:d5:fe:57:3f:32:b4:d0 matrix@vivid
The key's randomart image is:
+---[RSA 4096]----+
| .. |
| . . |
| . . .. . |
| . . o o.. E .|
| o S ..o ...|
| = ..+...|
| o . . .o .|
| .o . |
| .++o |
+-----------------+
```
SSH密钥会默认保存在 `~/.ssh/id_rsa`文件中．SSH公钥保存在 `~/.ssh/id_rsa.pub` 文件中．

### 远程连接Linux

使用`ssh-copy-id`命令将SSH公钥上传到远程Linux服务器

```bash
ssh-copy-id username@remote-server
```

输入远程用户密码后, SSH公钥就会自动上传了．SSH公钥保存在远程Linux服务器的 `~/.ssh/authorized_keys` 文件中．

上传完成后，SSH登录就不需要再次输入密码了．但是首次使用SSH Key登录时需要输入一次SSH密钥的加密密码．（只需要输入一次，将来会自动登录，不再需要输入密钥的密码）

测试一下远程ssh查看系统版本:
```bash
ssh -p 22 username@remote-server "cat /etc/redhat-release"
```

OK! 以后使用scp命令来传送文件时也不需要输入密码了．
