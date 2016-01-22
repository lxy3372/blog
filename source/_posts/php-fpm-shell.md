title: "简单的进程启动和关闭Shell脚本制作"
date: 2015-04-22 22:12:55
tags: [php,shell]
description: php-fpm.sh 启动脚本制作
author: riky

---

整个Shell相对简单，重点理解`awk` ，管道`|`,以及 `xargs kill -9` 应该就比较清晰了，
整个过程就是一个查询-&gt;筛选-&gt;然后kill-&gt;到在启动的过程，其他服务如Redis,MongoDB,Memecache以及Mysql和Apache都可以用类似的方式来制作简单的Shell脚本。
使用相对简单：`./php-fpm.sh start`
如果不能放到/bin/bash目录，可以采用软连接`ln -s`或者别名`alias fpm='/usr/local/bash/php-fpm.sh'`的方式来简化命令操作：
####别名：####
```bash
#命令行操作即可
#输出到.bashrc
echo "alias phpfpm='/usr/local/bash/php-fpm'">>~/.bashrc
#使其生效
soure ~/.bashrc
```

<!--more-->

- - -
脚本源码:
```bash
#!/bin/sh

#从终端接收第一个参数，系统本身默认当前shel为第0个参数$0
param=$1

#启动进程函数
start()
{
	#执行进程查询
	#ps aux  查询所用用户进程
	#grep -i "php-fpm"  查询进程为php-fpm的进程，不区分大小写
	#grep -v grep 移除查询结果中存在grep的进程，即忽略当前查询进程
	#awk '{print $2}' 打印第二列参数
    fpms=`ps aux | grep -i "php-fpm" | grep -v grep | awk '{print $2}'`

	#当前进程不为空  -n ： 字符串不为空
    if [ ! -n "$fpms" ]; then
		#启动进程
        php-fpm
        echo "PHP-FPM Start"
    else
        echo "PHP-FPM Already Start"
    fi
}

 #停止进程
stop()
{
    fpms=`ps aux | grep -i "php-fpm" | grep -v grep | awk '{print $2}'`
	#xargs 将  前面 $fpms 作为kill的参数，执行kill进程操作
    echo $fpms | xargs kill -9

    for pid in $fpms; do
		#正则匹配非数字
        if echo $pid | egrep -q '^[0-9]+$'; then
            echo "PHP-FPM Pid $pid Kill"
        else
            echo "$pid IS Not A PHP-FPM Pid"
        fi
    done
}

#switch 调用 
case $param in
    'start')
        start;;
    'stop')
        stop;;
    'restart')
        stop
        start;;
    *)
        echo "Usage: ./phpfpm.sh start|stop|restart";;
esac
```