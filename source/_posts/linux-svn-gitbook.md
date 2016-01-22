title: "Linux笔记：SVN"
date: 2015-12-01 15:31:27
tags: [svn]
description: linux学习笔记，svn和gitbook
author: riky

---

由于公司木有运维，什么事都需要自己动手来做。在一步步搭建开发环境的过程中总是会遇到很多选择和坑。首当其冲的便是版本控制的选择：Git和SVN。
虽然我个人偏向于Git（GitLab或者Gitblit，之后会提到这些，毕竟是我自己经常使用的）。今天说说SVN：
安装就不用说了，除了一些较为复杂的软件，安装命令都是容易想得到的：
```bash
sudo apt-get install subversion
```

用SVN是大家讨论后得出的结果，我也觉得有两点在SVN上比较好用：
1. 利用SVN部署DEV环境的web服务
2. 利用SVN部署WIKI,这里我采用的是[GitBook](http://www.gitbook.com)。(这是个不错的玩意儿)

先记录下SVN安装SVN之后一些必要的操作：

####1. 创建仓库
```bash
sudo svnadmin create /data/svn/web  #缺省，细节参数未曾研究
```
<!--more-->
之后会在该目录下生成一下目录大致如下：
```bash
.
├── conf  #配置文件目录
│   ├── authz #用户和组配置
│   ├── hooks-env.tmpl
│   ├── passwd #用户账号密码
│   └── svnserve.conf  #svn服务配置(项目和系统相关)
├── db
├── format
├── hooks #hooks这个很重要
│   ├── post-commit.tmpl
│   ├── post-lock.tmpl
│   ├── post-revprop-change.tmpl
│   ├── post-unlock.tmpl
│   ├── pre-commit.tmpl
│   ├── pre-lock.tmpl
│   ├── pre-revprop-change.tmpl
│   ├── pre-unlock.tmpl
│   ├── start-commit.tmpl
│   └── test.log
├── locks
└── README.tx
```
######1.1 conf基本配置
- authz 配置用户组和读写权限
```bash
#添加：
[groups]
dev = dev,riky #用户所属组
[/]
@dev = rw #组配置权限(简单的配置)
```
- passwd 配置账号和密码
```bash
#配置用户账号密码
[users]
riky = riky
dev = dev
```
- svnserve.conf 这个文件我没有做什么修改

酱紫基本svn服务就可以起起来了。
```bash
#svn服务器目录/data/svn/web/  端口 3691
sudo svnserve -d -T -r /data/svn/web/ --listen-port 3691
```

######1.2 利用hooks自动部署Web和WIKI
服务端启动了就可以检出到服务器web目录（`cd /data/web`）：
```bash
svn checkout svn://192.168.1.1:3691 #检出整个目录
#然后提交几个测试文件上去即可
touch test.txt
svn add test.txt
svn ci * -m"init"
```
这样我们就可以利用hooks了：
- 将上述hooks中post-commit.tmpl 重命名为post-commit
- 编辑添加以下：
```bash
export LANG=en_US.UTF-8
SVN=/usr/bin/svn
cd /data/web  #web客户端根目录
$SVN update
```

保存之后，在其他电脑上检出svn，并提交php文件，就会在当前目录自动更新，实现SVN提交代码即可在Web服务器上生效了。

####2. 开机启动
为了避免停机后出现的麻烦,可以将svn启动加入到启动脚本中：
在/etc/init.d/下创建svn.sh:
```bash
#/bin/bash

svnserve -d --listen-port 3691 -T -r /data/svn/web/
#如果有多个svn仓库，可以加入多个
```
在/etc/rs.local中添加如下：
```bash
/etc/init.d/svn.sh #新加入，需放在exit前
exit 0
```
重启服务器即可生效。
以上就是最近在整服务器的时候一些摸索出来的配置，至于准不准备我也不太确定！总之，之后看到的时候还是能有一些帮助的，一些更细节性的配置以后在修正。