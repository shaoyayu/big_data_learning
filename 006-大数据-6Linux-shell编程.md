# Shell编程-1

what>why>how学习方向

shell bash 

bash 解释器，启动器

用户的交互输入

文本文件输入

```shell
[root@MDNode01 shell]# ps -fe
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 03:27 ?        00:00:01 /sbin/init
root          2      0  0 03:27 ?        00:00:00 [kthreadd]
......
postfix    1050   1028  0 03:27 ?        00:00:00 qmgr -l -t fifo -u
root       1051      1  0 03:27 tty1     00:00:00 /sbin/mingetty /dev/tty1
root       1053      1  0 03:27 tty2     00:00:00 /sbin/mingetty /dev/tty2
root       1055      1  0 03:27 tty3     00:00:00 /sbin/mingetty /dev/tty3
root       1057      1  0 03:27 tty4     00:00:00 /sbin/mingetty /dev/tty4
root       1059      1  0 03:27 tty5     00:00:00 /sbin/mingetty /dev/tty5
root       1061      1  0 03:27 tty6     00:00:00 /sbin/mingetty /dev/tty6
root       1068    354  0 03:27 ?        00:00:00 /sbin/udevd -d
root       1069    354  0 03:27 ?        00:00:00 /sbin/udevd -d
root       1070    952  0 03:28 ?        00:00:03 sshd: root@pts/0 
root       1072   1070  0 03:28 pts/0    00:00:00 -bash
postfix    1317   1028  0 05:07 ?        00:00:00 pickup -l -t fifo -u
root       1733   1072  0 05:50 pts/0    00:00:00 ps -fe
[root@MDNode01 shell]# $$
-bash: 1072: command not found
[root@MDNode01 shell]# echo $$
1072

```



$$代表的是当前进程的bash的id

```shell
[root@MDNode01 shell]# cat hello.txt 
echo "hello world"
ls -l /
echo $$
[root@MDNode01 shell]# source hello.txt
hello world
total 94
dr-xr-xr-x.  2 root root  4096 Sep 25 05:22 bin
dr-xr-xr-x.  5 root root  1024 Jul 19 23:38 boot
drwxr-xr-x  18 root root  3700 Sep 25 03:27 dev
drwxr-xr-x. 74 root root  4096 Sep 25 05:22 etc
drwxr-xr-x.  2 root root  4096 Sep 23  2011 home
dr-xr-xr-x.  8 root root  4096 Sep 16 01:24 lib
dr-xr-xr-x. 10 root root 12288 Sep 25 05:22 lib64
drwx------.  2 root root 16384 Jul 19 23:35 lost+found
drwxr-xr-x.  2 root root  4096 Sep 23  2011 media
drwxr-xr-x.  2 root root  4096 Sep 23  2011 mnt
drwxr-xr-x.  3 root root  4096 Sep 16 01:50 opt
dr-xr-xr-x  84 root root     0 Sep 25 03:27 proc
dr-xr-x---.  3 root root  4096 Sep 19 03:49 root
dr-xr-xr-x.  2 root root 12288 Sep 16 01:31 sbin
drwxr-xr-x.  2 root root  4096 Jul 19 23:36 selinux
drwxr-xr-x.  2 root root  4096 Sep 23  2011 srv
drwxr-xr-x  13 root root     0 Sep 25 03:27 sys
drwxrwxrwt.  4 root root  4096 Sep 25 05:26 tmp
drwxr-xr-x. 14 root root  4096 Sep 19 03:47 usr
drwxr-xr-x. 17 root root  4096 Jul 19 23:36 var
1072
```



使用**source**加载脚本

与 

```shell
. hello.text
```



## 分为两种执行方式：

当前线程：

source 或者 . 的加载方式

子进程执行

/bin/bash 执行

```shell
[root@MDNode01 shell]# cat hello.txt 
#!/bin/bash
echo "hello world"
ls -l /
echo $$
[root@MDNode01 shell]# chmod +x hello.txt 
[root@MDNode01 shell]# ll
total 4
-rwxr-xr-x 1 root root 47 Sep 25 06:57 hello.txt
```

使用 chmod +x [文件] 给文件一个可执行的权限

```shell
[root@MDNode01 shell]# ./hello.txt 
hello world
total 94
dr-xr-xr-x.  2 root root  4096 Sep 25 05:22 bin
dr-xr-xr-x.  5 root root  1024 Jul 19 23:38 boot
drwxr-xr-x  18 root root  3700 Sep 25 03:27 dev
drwxr-xr-x. 74 root root  4096 Sep 25 05:22 etc
drwxr-xr-x.  2 root root  4096 Sep 23  2011 home
dr-xr-xr-x.  8 root root  4096 Sep 16 01:24 lib
dr-xr-xr-x. 10 root root 12288 Sep 25 05:22 lib64
drwx------.  2 root root 16384 Jul 19 23:35 lost+found
drwxr-xr-x.  2 root root  4096 Sep 23  2011 media
drwxr-xr-x.  2 root root  4096 Sep 23  2011 mnt
drwxr-xr-x.  3 root root  4096 Sep 16 01:50 opt
dr-xr-xr-x  85 root root     0 Sep 25 03:27 proc
dr-xr-x---.  3 root root  4096 Sep 19 03:49 root
dr-xr-xr-x.  2 root root 12288 Sep 16 01:31 sbin
drwxr-xr-x.  2 root root  4096 Jul 19 23:36 selinux
drwxr-xr-x.  2 root root  4096 Sep 23  2011 srv
drwxr-xr-x  13 root root     0 Sep 25 03:27 sys
drwxrwxrwt.  4 root root  4096 Sep 25 05:26 tmp
drwxr-xr-x. 14 root root  4096 Sep 19 03:47 usr
drwxr-xr-x. 17 root root  4096 Jul 19 23:36 var
1753
```

这样就可以执行文件了，

## 定义函数

```shell
[root@MDNode01 shell]# clear 
[root@MDNode01 shell]# fun(){
> echo "hello world"
> ls -l /
> $$
> }
[root@MDNode01 shell]# fun
hello world
total 94
dr-xr-xr-x.  2 root root  4096 Sep 25 05:22 bin
dr-xr-xr-x.  5 root root  1024 Jul 19 23:38 boot
drwxr-xr-x  18 root root  3700 Sep 25 03:27 dev
drwxr-xr-x. 74 root root  4096 Sep 25 05:22 etc
drwxr-xr-x.  2 root root  4096 Sep 23  2011 home
dr-xr-xr-x.  8 root root  4096 Sep 16 01:24 lib
dr-xr-xr-x. 10 root root 12288 Sep 25 05:22 lib64
drwx------.  2 root root 16384 Jul 19 23:35 lost+found
drwxr-xr-x.  2 root root  4096 Sep 23  2011 media
drwxr-xr-x.  2 root root  4096 Sep 23  2011 mnt
drwxr-xr-x.  3 root root  4096 Sep 16 01:50 opt
dr-xr-xr-x  84 root root     0 Sep 25 03:27 proc
dr-xr-x---.  3 root root  4096 Sep 19 03:49 root
dr-xr-xr-x.  2 root root 12288 Sep 16 01:31 sbin
drwxr-xr-x.  2 root root  4096 Jul 19 23:36 selinux
drwxr-xr-x.  2 root root  4096 Sep 23  2011 srv
drwxr-xr-x  13 root root     0 Sep 25 03:27 sys
drwxrwxrwt.  4 root root  4096 Sep 25 05:26 tmp
drwxr-xr-x. 14 root root  4096 Sep 19 03:47 usr
drwxr-xr-x. 17 root root  4096 Jul 19 23:36 var
-bash: 1072: command not found
[root@MDNode01 shell]# $$
-bash: 1072: command not found

```

这样就可以定义一个函数了；

这是一个当前的shell即当前的线程执行的，

```shell
[root@MDNode01 shell]# type fun
fun is a function
fun () 
{ 
    echo "hello world";
    ls --color=auto -l /;
    $$
}
```

fun是一个函数【function】，

常见的命令：

1、是一个在当前线程人为定义的function，

2、/bin/bash加载的

3、PATH目录下面，磁盘里面的某个可执行的程序。

## 文本流&重定向

#### 重定向

##### 输出

![](images\重定向.png)

修改线程的标准输出的位置

```shell

exec 6>&1
6和1都指指向0这个地方
exec 1> /dev/pts/1
将原来的1输出的位置0指向/dev/pts/1这个位置

```

不是命令

-- 程序本身的I/o

- 0：标准输入
- 1：标准输出
- 2：错误输出

-- 控制程序I/O位置



```shell
[root@MDNode01 ~]# ll -l / 1> ls.out
[root@MDNode01 ~]# cat ls.out 
total 94
dr-xr-xr-x.  2 root root  4096 Sep 25 05:22 bin
dr-xr-xr-x.  5 root root  1024 Jul 19 23:38 boot
drwxr-xr-x  18 root root  3700 Sep 25 03:27 dev
drwxr-xr-x. 74 root root  4096 Sep 25 05:22 etc
drwxr-xr-x.  2 root root  4096 Sep 23  2011 home
dr-xr-xr-x.  8 root root  4096 Sep 16 01:24 lib
dr-xr-xr-x. 10 root root 12288 Sep 25 05:22 lib64
drwx------.  2 root root 16384 Jul 19 23:35 lost+found
drwxr-xr-x.  2 root root  4096 Sep 23  2011 media
drwxr-xr-x.  2 root root  4096 Sep 23  2011 mnt
drwxr-xr-x.  3 root root  4096 Sep 16 01:50 opt
dr-xr-xr-x  84 root root     0 Sep 25 03:27 proc
dr-xr-x---.  3 root root  4096 Sep 25 08:05 root
dr-xr-xr-x.  2 root root 12288 Sep 16 01:31 sbin
drwxr-xr-x.  2 root root  4096 Jul 19 23:36 selinux
drwxr-xr-x.  2 root root  4096 Sep 23  2011 srv
drwxr-xr-x  13 root root     0 Sep 25 03:27 sys
drwxrwxrwt.  4 root root  4096 Sep 25 05:26 tmp
drwxr-xr-x. 14 root root  4096 Sep 19 03:47 usr
drwxr-xr-x. 17 root root  4096 Jul 19 23:36 var
```



将/目录下面的ll -l / 的输入出指向ls.out文件里面，

1>：代表的是覆盖输出

1>>：代表的是追加输出

2>：代表的是错误的输出

如：ll -l /abc 1> ls.out

这样的目录是不存在的，所以用的是2>输出的方式

```shell
[root@MDNode01 shell]# ls -l /good /usr 1>ls1.out 2>&1
[root@MDNode01 shell]# cat ls1.out 
ls: cannot access /good: No such file or directory
/usr:
total 80
dr-xr-xr-x.  2 root root 20480 Sep 25 05:22 bin
drwxr-xr-x.  2 root root  4096 Sep 23  2011 etc
drwxr-xr-x.  2 root root  4096 Sep 23  2011 games
drwxr-xr-x. 41 root root  4096 Sep 16 01:31 include
drwxr-xr-x   3 root root  4096 Sep 19 03:52 java
dr-xr-xr-x. 11 root root  4096 Sep 25 05:22 lib
dr-xr-xr-x. 40 root root 20480 Sep 25 05:22 lib64
drwxr-xr-x. 12 root root  4096 Sep 25 05:22 libexec
drwxr-xr-x. 12 root root  4096 Jul 19 23:36 local
dr-xr-xr-x.  2 root root  4096 Sep 25 05:22 sbin
drwxr-xr-x. 79 root root  4096 Sep 25 05:22 share
drwxr-xr-x.  4 root root  4096 Jul 19 23:36 src
lrwxrwxrwx.  1 root root    10 Jul 19 23:36 tmp -> ../var/tmp
```

**先给标准输出指向一个文件，在把错误输出指向标准输出**

与如下写法一样：程序会把1和2的流指向ls2.out的文件里面

```shell
ls -l /good /usr >& ls2.ot
ls -l /good /usr &> ls2.ot
```



##### 输入

read 命令

```shell
[root@MDNode01 shell]# read syy
##进入一个阻塞状态，等待用户的输入，每一个字符都会被检测，输入回车的时候结束输入；
shaoyayu.320.io                                           
##打印当前shell的一个变量
[root@MDNode01 shell]# echo $syy
shaoyayu.320.io
```



```shell
[root@MDNode01 shell]# read syy 0<<<"dsafdsaf"
[root@MDNode01 shell]# echo $syy
dsafdsaf
[root@MDNode01 shell]# read syy 0<<xx
> sdaf
> dsaf
> dsaf
##输入定义流的时候字符相等的时候表示结束；
> xx
[root@MDNode01 shell]# echo $syy
##遇到回车read的流会中断
sdaf
[root@MDNode01 shell]# vi 00.sh
[root@MDNode01 shell]# . 00.sh 
read读取到回车的时候结束
cat的0输入流不会哦
[root@MDNode01 shell]# cat 00.sh 
cat 0<<xx
read读取到回车的时候结束
cat的0输入流不会哦
xx
[root@MDNode01 shell]# 

```

![](images\Socket重定向.png)

### 变量 





```shell

```



### 引用&命令替换

```shell

```



### 退出状态&逻辑判断



### 表达式

### 流程控制








