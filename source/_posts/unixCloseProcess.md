---
title: Mac环境下关闭指定端口进程
date: 2016-12-19 22:26:31
tags: [Linux, shell]
category: Linux
---

> 一次Webapck打包进度卡在70%,`Ctrl+C`退出后,重启Webpack发现一直提示端口被占用.

### 查看指定端口进程
查找`8001`端口被哪个进程占用:
```bash
# lsof -i:8001
```
结果显示
```
COMMAND  PID   USER    FD   TYPE             DEVICE SIZE/OFF NODE NAME
node     1577  relign  24u  IPv4 0xf4c52151f2395907      0t0  TCP *:vcom-tunnel (LISTEN)
```
我们可以看到一个node进程占用了`8001`端口.
输出各列信息的意义如下:
* `COMMAND` 进程的名称,也就是启动的程序名
* `PID` 进程的ID
* `USER` 进程属主的名字
* `FD` 文件描述符,应用程序通过文件描述符识别该文件.如`CWD`,`txt`等
* `TYPE` 文件的类型,如`DIR`、`REG`等
* `DEVICE` 指定磁盘的名称
* `SIZE` 文件的大小
* `NODE` 索引节点(文件在磁盘上的标识)
* `NAME` 打开文件的确切名称

<!--more-->
### 关闭进程
```
# kill -9 1577
```
终止一个进程或终止一个正在运行的程序,一般是通过`kill`、`killall`、`pkill`、`xkill`等进行.比如一个程序已经死掉,但又不能退出,这时就应该考虑应用这些工具.

#### kill命令

`kill`命令可通过进程ID(PID)给进程发信号.默认情况下,`kill`命令会向命令行中列出的全部PID发送一个
`TERM`信号.遗憾的是你只能用PID而不能用命令名,所以`kill`命令有时并不好用.

要发送进程信号,你必须是进程的属主或登录为`root`用户.
```
$ kill 1577
```
`TERM`信号告诉进程可能的话就停止运行.不过,如果有不服管教的进程,那它通常会忽略这个请求.如果要强制终止,可以用`kill -9 [PID]`.

#### killall命令

`killall`通过程序的名字,直接杀死所有进程.