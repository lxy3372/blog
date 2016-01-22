title: "Linux笔记:创建用户和私钥登陆"
date: 2015-11-30 16:08:45
tags: [linux,ssh,adduser,ssh-copy-id]
description: Linux运维笔记
author: riky

---
最近侥幸获得一台低配版的阿里云主机，虽然是低配但是练练手也是挺好的。自从进入“某爱”之后，我开始不再只是程序猿了，居然还是“搞机”的运维狗，无奈啊，公司的云主机都是我一个一个Google然后配起来的，虽然不是很标准，但最起码还是能用的。趁此机会将之前倒腾过的部分技能重新梳理下，免得到时候又给忘记了。

####注册新用户
通过Root登录之后第一件事就是创建一个属于自己的用户，然后把root这个账号登录权限关闭了。防止被攻击（之前在不懂事的时候就被攻击了）。
```bash
adduser username#后续输入密码，即可（可填项跳过）
```
这个地方我之前踩过一个坑，那就是** `adduser` ** 和 **`useradd`**是有区别的，区别在于useradd仅仅只是创建了一个用户，没有密码。而`adduser`会为用户做许多准备工作：
1.	需要输入密码
2.	建立与当前创建用户名相同的group
3.	会在home目录下建立用户目录，用户登陆之后的默认目录就是这个，`cd ~`也是回到这个目录
4.	最重要的是可以通过SSH登陆了，其他就不太清楚了

酱紫，基本上就可以用自己的账号密码登陆了`ssh user@ip`。
<!--more-->

####SSH私钥登陆
1. 先生成本地密钥
```bash
ssh-keygen -C "your email" #命令后续存在交互，一般回车忽略
```
主目录下`~/.ssh`文件夹中会生成对应`id_rsa`(私钥)和`id_rsa.pub`(公钥)

2. 上传公钥到服务器
网上较多的说法是scp上传id_rsa.pub到服务器上`~/.ssh/authorized_keys`,然后更改该文件权限为600即可，我试过并不好使。最简洁的方式是使用`ssh-copy-id`即可。
```bash
ssh-copy-id  -i id_rsa.pub  user@ip
```
酱紫就可以直接用私钥登陆了。

3. 关闭root登陆和密码认证
为了服务器安全，我经常选择关闭掉root的登陆权限和密码登陆权限，更改两项配置即可`vim /etc/ssh/sshd_config`：
```nginx
PermitRootLogin no  #文件最后两行 将yes改为no
SyslogFacility AUTHPRIV
PasswordAuthentication no  #最后一行 将yes改为no
```
保存后重启即可`sudo reboot`
** 这里需要记下的一点的是，修改改文件需要管理员权限，而经过上述创建的用户，并不是sudoers也就是无法通过使用sudo来更改此文件 **

4. 配置sudoers权限
该文件配置项还是比较多的，我只是想用户能用即可，其他的坑待以后来填，已root用户身份登陆`su`，编辑`/etc/sudoers`:
```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL
username ALL=(ALL:ALL) ALL #新添加
```
保存后就可以使用了，以上就是一些创建用户的基本上要做的（个人走过的路而已，已在高速上大神勿喷）。
**注： 关闭密码登陆而使用私钥登陆之后，请做好私钥备份。免得到时候私钥弄丢了，进不去就悲剧了**