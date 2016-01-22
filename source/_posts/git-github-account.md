title: "客户端设置多个github账号"
date: 2015-04-22 22:33:31
tags: [Git,GitHub]
description: 客户端git配置多个github账号

---

最近由于需要在自己的电脑上办公，需要远程连接公司的Gitblit。但本机已经简单的配置GitHub的一些账号信息。在两个账号切换的过程中会有些麻烦，于是Google下，有了以下的解决方案：

####（参考: http://www.cnblogs.com/sosoft/p/3477578.html）####

###步骤一：用`ssh-keygen`命令生成一组新的id_rsa_new和id_rsa_new.pub。 ###

```language
ssh-keygen -t rsa -C "new email"
```
平时我们都是直接回车，默认生成id_rsa和id_rsa.pub。这里特别需要注意，出现提示输入文件名的时候要输入与默认配置不一样的文件名，比如： id_rsa_new。

###步骤二：配置~/.ssh/config文件，以我自己的机器为例。###

```language
#Default Git

Host chdxy# 比如 chdxy

HostName IP Address #一般采用对应域名即可  github.com

User think #该用户为github账户用户名  如  CHDTEAM

IdentityFile ~/.ssh/id_rsa

#Second Git   同上配置采用一样的即可

Host secondgit

HostName IP Address #域名也可

User think

IdentityFile ~/.ssh/id_rsa_second
```
Host就是每个SSH连接的单独代号，IdentityFile告诉SSH连接去读取哪个私钥。

###步骤三：执行ssh-agent让ssh识别新的私钥。###
<!--more-->
```language
# 多个账号的ssh-key 都需要加进来

#最好先测试下是否连接成功  ssh -T git@chdxy   （一般默认为git@github.com 此处设置了Host，采用自己账号的Host即可）

ssh-add ~/.ssh/id_rsa_new
ssh-add ~/.ssh/id_rsa
```
该命令如果报错：Could not open a connection to your authentication agent.无法连接到ssh agent，可执行ssh-agent bash命令后再执行ssh-add命令。
以后，在clone或者add remote的时候，需要把config文件中的host代替git@remoteaddress中的remoteaddress。

同时，你可以通过在特定的repo下执行下面的命令，生成区别于全局设置的user.name和user.email。
```language
git config user.name "newname"

git config user.email "newemail"

#git config --global --unset user.name 取消全局设置

#git config --global --unset user.email 取消全局设置

#在同一机器不同目录下克隆远程同一个repo
```
然后执行对应的clone操作就可以了

```shell
cd /home/user1

git clone git@chdxy/chdxy:xxx.git   #格式  git clone git@Host:用户名/具体.git   即把默认的github.com 改成 “Host”

cd /home/user2
git clone git@secondgit:User/xxx.git
```
上面的两条clone命令，虽然关联到同一个repo，却是通过不同ssh连接，当然也是不同的git账号。
注意：以上不要设置全局的user 和email 直接设置当前仓库的user 和email即可 （去掉—global，如果已设置加上—unset参数即可）

接下来的各种操作（pull、push、fetch和普通的一样）

```shell
git push origin master
```