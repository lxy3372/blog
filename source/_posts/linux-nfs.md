title: "linux笔记：NFS的使用"
date: 2015-12-03 17:42:06
tags: [linux,NFS]
description: ubuntu下NFS搭建
author: riky

---
填坑不易，且填且珍惜！幸亏最近自己有整理相关的文档，有些东西自己在脑海里整理过一遍写出来，基本就不会忘记了，即使是有些偏差还是可以修正，加深印象。

先说说为什么会使用上NFS（Network File System, 作为一个小白程序猿，之前真的压根就没有听说过这玩意儿，我果然很菜），网站之前的服务器部署一直只有一台服务器（lnmp一键安装了），随然流量的增大，除了做一些软性的优化之外（缓存），我逐渐将Redis和Mysql迁移到了另一台服务器，以减少主站的压力。在此基础上考虑到可能会推一些引流的活动，以备用为目的申请了一台备用主机。以此为契机，我开始研究Nginx的负载均衡（之后整理下负载均衡的配置），成功的将主站服务器和备用 服务器纳入集群中。那这时候问题就来了：
1. 备用主机没有外网IP，网站一些爬虫无法请求到外网；
2. 因为没有使用第三方存储，上传的图片一致存储在主站的静态资源目录，以images.rikyliu.com的形式去访问。虽然通过nginx转发到备用主机可以上传上去，但是此主机没有外网IP，无法访问其静态资源。

虽然这两个问题都可以用nginx的proxy_pass来转发到主站服务器解决，但是当一些接口逻辑涉及到文件处理时却不是很方便，于是我找到了NFS来解决此问题：
![服务器架构](http://7xi3xm.com1.z0.glb.clouddn.com/server.jpg)
<!--more-->

如图所示，存储图片的主机开机NFS Server服务，将文件存储目录设置NFS 网络共享盘。然后备用主机将NFS Server的共享盘挂载在当前服务器的同一目录，从而实现从不同服务器上传文件，都可以进行存储和读取。

以下是（Ubuntu）基本的安装是使用：
##主服务器：服务端（内网：192.168.1.1）:
安装：
```bash
apt-get install apt-get install nfs-kernel-server
```
编辑:vim /etc/exports
```bash
#参数：
#共享目录   ip(权限)
/data/upload 192.168.1.2(rw,sync,no_root_squash,no_subtree_check,insecure)
/data/web/webapi_v2/upload 
#权限参数详情：

#rw  可读写

#sync 同步

#no_root_squash   登入 NFS 主机使用分享目录的使用者，如果是 root 的话，那么对于这个分享的目录来说，他就具有 root 的权限！这个项目『极不安全』，不建议使用！
#root_squash：在登入 NFS 主机使用分享之目录的使用者如果是 root 时，那么这个使用者的权限将被压缩成为匿名使用者，通常他的 UID 与 GID 都会变成 nobody 那个系统账号的身份。

#sync 资料同步写入到内存与硬盘当中 
#async 资料会先暂存于内存当中，而非直接写入硬盘 

#insecure 允许从这台机器过来的非授权访问

#no_subtree_check: 即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；
#subtree_check: 若输出目录是一个子目录，则nfs服务器将检查其父目录的权限；
```
重启nfs-kernel-server
具体参数可参考[cnblogs](http://www.cnblogs.com/lykyl/archive/2013/06/14/3136921.html)

##备用服务器：客户端（内网：192.168.1.2）

安装：
```bash
apt-get install nfs-common
```
挂载:
```bash
mount -t nfs 192.168.1.1:/data/upload  /data/upload #将192.168.1.1主机/data/upload 目录 挂载到目录/data/upload 下
```
酱紫，进入/data/upload 目录就可以看到192.168.1.1对应目录下的文件了，具体权限 请参考上面的参数设置即可。

这样就可以了么？还不够，如果主机重启了的话，还得手动挂载，所以需要在开机启动中设置自动挂载：
```bash
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
UUID=af414ad8-9936-46cd-b074-528854656fcd / ext4 errors=remount-ro 0 1
192.168.1.1:/data/upload /data/upload nfs defaults 0 0
```

以上就是最近我自己在使用NFS过程中得到的一些经验，如果有不对的地方，请及时指正，谢谢！