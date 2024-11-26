
# 系统进程


## 【1】、进程基本概述


当我们运行一个程序，那么我们将运行的程序叫进程


​ PS1：当程序运行为进程后，系统会为该进程分配内存，以及进程运行的身份和权限


​ PS2：在进程运行的过程中，服务器上会有各种状态来表示当前进程的指标信息


程序和进程的区别


​ 程序是数据和指令的集合，是一个静态的概念，比如/bin/ls /bin/cp 等二进制文件，同时程序可以长期存在系统中


​ 进程是程序运行的过程，是一个动态的概念，进程是存在生命周期概念的，也就是说进程随着程序的终止而销毁，不会永久存在系统中


程序的生命周期


​ 一个对象从无到有、从有到无过程称为生命周期


![img](https://img2023.cnblogs.com/blog/3475099/202411/3475099-20241125223446616-1832121283.jpg)


当父进程接收到任务调度时，会通过fock派生子进程来处理，子进程会继承父进程衣钵（相当于完全复制了一份父进程的信息）


1. 子进程在处理任务代码时，父进程会进入等待状态中，在一个进程中一般都是子进程去干活
2. 子进程在处理任务代码后，会执行退出，然后唤醒父进程来回收子进程的资源
3. 如果子进程在处理任务过程中，父进程退出，子进程没有退出，子进程没有被父进程管理，则变为**僵尸进程**
4. 每个进程都有自己的PID号，子进程称为PPID


祖先进程


1. pid\=1的进程
2. 在系统启动时，会首先启动一个祖先进程，在启动别的进程



```
root@xu-ubuntu:~/scripts# pstree -p
systemd(1)─┬─ModemManager(935)─┬─{ModemManager}(960)

[root@kylin-xu ~]# pstree -p 
systemd(1)─┬─NetworkManager(874)─┬─{NetworkManager}(882)

```

## 【2】、监控进程状态


程序在运行后，我们需要了解进程的运行状态，查看进程的状态分为:静态和动态两种方式


### 1、ps命令


使用ps命令查看当前的进程状态（静态）


ps –auxf常用组合方式查看进程、PID、占用cpu百分比，占用内存百分比，状态、执行的命令等


* \-a 显示所有终端机下执行的进程，除了阶段作业领导者之外
* \-u 以用户为主的格式来显示进程状况
* \-x 显示所有进程，不以终端机来区分
* \-f 用ASCII字符显示树状结构，表达进程间的相互关系


![image-20241125150128940](https://img2023.cnblogs.com/blog/3475099/202411/3475099-20241125223446322-175654733.png)




| **标志** | **意义** |
| --- | --- |
| USER | 该 process 属于那个使用者账号的 |
| PID | 该 process 的号码 |
| %CPU | 该 process 使用掉的 CPU 资源百分比 |
| %MEM | 该 process 所占用的物理内存百分比 |
| VSZ | 该 process 使用掉的虚拟内存量 (Kbytes) |
| RSS | 该 process 占用的固定的内存量 (Kbytes) |
| TTY | 该 process 是在那个终端机上面运作，若与终端机无关，则显示 ?。另外， tty1\-tty6 是本机上面的登入者程序，若为 pts/0 等等的，则表示为由网络连接进主机的程序。 |
| STAT | 该程序目前的状态 |
| START | 该 process 被触发启动的时间 |
| TIME | 该 process 实际使用 CPU 运作的时间 |
| COMMAND | 该程序的实际指令\[]内核态进程 无\[] 用户进程 |




| STAT基本状态 | 描述 | STAT状态\+符号 | 描述 |
| --- | --- | --- | --- |
| R | 进程运行 | s | 进程是控制进程， Ss进程的领导者，父进程 |
| S | 可中断睡眠 | \< | 进程运行在高优先级上，S\<优先级较高的进程 |
| T | 进程被暂停 | N | 进程运行在低优先级上，SN优先级较低的进程 |
| D | 不可中断睡眠 | \+ | 当前进程运行在前台，R\+该表示进程在前台运行 |
| Z | 僵尸进程 | l | 进程是多线程的，Sl表示进程是以线程方式运行 |



```
案例1：
1）在终端1上运行vim
vim test
2）在终端2上运行ps命令查看状态
[root@kylin-xu ~]# ps aux | grep vim
root      434314  0.2  0.3 227272  6960 pts/1    S+   18:27   0:00 vim test.txt
3）在终端1上挂起vim命令按下: ctrl+z
4) 回到终端2再次运行ps命令查看状态
[root@kylin-xu ~]# ps aux | grep vim
root      434314  0.0  0.3 227272  6960 pts/1    T    18:27   0:00 vim test.txt

```


```
案例2
PS命令查看不可中断状态进程
使用tar打包文件时，可以通过中断不断查看状态，由S+,R+变为D+
[root@xu ~]# ps axu|grep tar|grep -v grep
root     14289  2.6  0.1 124268  1888 pts/2    S+   10:56   0:01 tar zcf etc.tar.gz /etc/ /usr/ /var
[root@xu ~]# ps axu|grep tar|grep -v grep
root     14289  2.7  0.2 124380  2240 pts/2    R+   10:56   0:01 tar zcf etc.tar.gz /etc/ /usr/ /var
 [root@xu ~]# ps axu|grep tar|grep -v grep
root     14289  2.9  0.2 124916  2724 pts/2    D+   10:56   0:01 tar zcf etc.tar.gz /etc/ /usr/ /var

```

### 2、top命令


使用top命令查看当前的进程状态 **动态**




| **任务** | **含义** |
| --- | --- |
| Tasks:73 total | 当前进程的总数 |
| 2 running | 正在运行的进程数 |
| 71 sleeping | 睡眠的进程数 |
| 0 stopped | 停止的进程数 |
| 0 zombie | 僵尸进程数 |
| %Cpu(s): 49\.2 us | 系统用户进程使用CPU百分比 |
| 5\.7 sy | 内核进程占用CPU百分比，内核是于硬件进行交互 |
| ni | 调整过优先级进程占用百分比 |
| 45\.2 id | 空闲CPU的百分比 |
| 0\.0 wa | CPU等待IO完成的时间 |
| 0\.0 hi | 硬中断，占的CPU百分比 |
| 0\.0 si | 软中断，占的CPU百分比 |
| 0\.0 st | 比如虚拟机占用物理CPU的时间 |


什么是中断？


中断就是终止当前在做的事情 去执行另一段程序


**会保留现场**，执行的那段程序做完之后会在回来执行刚来尚未完成的部分


![image-20241125160520458](https://img2023.cnblogs.com/blog/3475099/202411/3475099-20241125223445944-1729402762.png)


软中断和硬中断的区别




|  | 软中断 | 硬中断 |
| --- | --- | --- |
| 是否有随机性 突发性 | 否 | 是 |
| 是否有中断响应周期 | 无 | 是 |
| 中断类型号的提供方法 | 固定或由指令提供 | 由中断控制器提供，硬件提供 |


## 【3】、管理进程状态


当程序运行为进程后，如果希望停止进程，怎么办呢？那么此时我们可以使用linux的kill命令对进程发送关闭新号，当然除了kill 还有killall pkill


使用kill –l列出当前系统所支持的信号


![image-20241125161406952](https://img2023.cnblogs.com/blog/3475099/202411/3475099-20241125223445622-870624090.png)


虽然Linux信号很多，但是我们仅仅使用最常用的3个信号


1） SIGHUP 重新加载配置文件 1 不停机维护


2） SIGKILL 强制杀死进程 9 **不管内存中是否有没有数据，直接杀进程，可能会丢文件，类似拔电源**


3） SIGTERM 终止进程，**默认kill使用该信号** 15 **如果内存中还有和当前进程相关的信息，将内存中的信息写入磁盘后，在杀死进程**



```
[root@kylin-xu ~]# ps -ef | grep httpd
root      272889       1  0 06:45 ?        00:00:03 /usr/sbin/httpd -DFOREGROUND
apache    272972  272889  0 06:45 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache    272973  272889  0 06:45 ?        00:00:11 /usr/sbin/httpd -DFOREGROUND
apache    272974  272889  0 06:45 ?        00:00:12 /usr/sbin/httpd -DFOREGROUND
apache    272975  272889  0 06:45 ?        00:00:11 /usr/sbin/httpd -DFOREGROUND
[root@kylin-xu ~]# kill -9 272889
[root@kylin-xu ~]# ps aux | grep httpd

```


```
# 修改httpd端口，使用kill -1 重新加载，不停机维护
root@kylin-xu ~]# ss -tunlp | grep httpd
tcp     LISTEN   0        128                    *:80                  *:*       users:(("httpd",pid=470759,fd=4),("httpd",pid=470758,fd=4),("httpd",pid=470757,fd=4),("httpd",pid=470213,fd=4))

vim /etc/httpd/conf/httpd.conf 

[root@kylin-xu ~]# kill -1 470213
[root@kylin-xu ~]# ss -tunlp | grep httpd
tcp     LISTEN   0        128                    *:8080                *:*       users:(("httpd",pid=471610,fd=9),("httpd",pid=471609,fd=9),("httpd",pid=471608,fd=9),("httpd",pid=470213,fd=9))

```

**killall命令**



```
# 可以根据进程名字杀死进程
[root@kylin-xu ~]# killall httpd
[root@kylin-xu ~]# ps -ef | grep httpd
root      474240  430910  0 19:41 pts/1    00:00:00 grep httpd

```

**pkill命令**



```
    使用pkill踢出从远程登录到本机的用户，终止pts1上所有进程，用户强制退出
[root@lzy ~]# pkill -9 -t pts/1

```

## 【4】、后台进程


### 1、临时后台进程


1） 什么是后台进程


​ 运行的进程默认在终端前台运行，一旦关闭终端，进程随着结束，此时希望进程在后台运行不退出，这样关闭终端也不影响进程的正常运行


2） 如何把程序放在后台



```
# 把进程放入后台运行 使用 &
[root@kylin-xu ~]# sleep 5000 &
[1] 480069
[root@kylin-xu ~]# ps aux | grep sleep
root      480069  0.0  0.0 212380   760 pts/1    S    19:54   0:00 sleep 5000
root      480151  0.0  0.0 213156   892 pts/1    R+   19:54   0:00 grep sleep

# jobs查看后台进程
[root@kylin-xu ~]# jobs
[1]+  运行中               sleep 5000 &

# 调回前台执行
[root@kylin-xu ~]# fg %1
sleep 5000


# ctrl + c 终止

# CTRL + z 后台停止，bg 进程在后台执行
[root@kylin-xu ~]# sleep 5000 
^Z
[1]+  已停止               sleep 5000
[root@kylin-xu ~]# bg %1
[1]+ sleep 5000 &

```

放入后台的进程，如果再终端断开连接后 ，重新连上进程会消失，我们现在不想让他消失


### 2、持久化后台进程


**screen的使用 常用**



> screen的原理：开辟一个新的bash space ，将进程放入新的 bash space 中，



```
yum install -y screen

# 开创新的 bash ---- -S  bash空间名称
[root@kylin-xu ~]# screen -S wget_install
# 进入了screen 开辟的新的bash，在新的bash中执行命令，就相当于在后台执行命令了
[root@kylin-xu ~]# sh 1.sh 

# ctrl +a+d 表示退出当前bash空间，放入后台，会到前台
[root@kylin-xu ~]# ps aux | grep wget
root      491586  0.0  0.1 228240  2360 ?        Ss   20:27   0:00 SCREEN -S wget_install
root      491633  1.0  0.6 227348 12832 pts/1    S+   20:27   0:01 wget https://pkg.jenkins.io/redhat-stable/jenkins-2.190.1-1.1.noarch.rpm --no-check-certificate

# 通过screen创建的后台进程在断开终端后也会正常执行
ctrl + d，重连终端，进程依旧存在
[root@kylin-xu ~]# ps aux | grep wget
root      491586  0.0  0.1 228240  2360 ?        Ss   20:27   0:00 SCREEN -S wget_install


```


```
--list：查看当前有哪些bash空间
[root@kylin-xu ~]# screen -list
There is a screen on:
        491586.wget_install     (Detached)
1 Socket in /run/screen/S-root.

-r：进入指定的bash space	，可以通过进程号或bash space name
[root@kylin-xu ~]# screen -r  491586

```

**nohup**



```
[root@kylin-xu ~]# nohup sleep 300 &
[1] 494330

```

## 【5】、进程优先级


1） 在启动进程时，为不同的进程使用不同的调度策略


* nice 值越高 表示优先级越低，例如\+19 该进程容易将CPU使用量让给其他进程
* nice值越低 表示优先级越高，例如\-20, 改进程更不倾向于让出CPU


使用top或ps敏玲查看进程的优先级


使用top可以查看nice优先级 NI:实际nice级别，默认是0 动态修正CPU调度。范围（\-20\~19）。越大，cpu调度越一般，越小，cpu调度越偏向它。一般用于后台进程，调整也是往大了调，用来给前台进程让出CPU资源


PR: 优先级 显示nice值，PR默认是20，越小，优先级越高。修改nice可以同时修改PR \-20映射到0， \+19映射到39


![image-20241125170420489](https://img2023.cnblogs.com/blog/3475099/202411/3475099-20241125223445189-1734319279.png)


nice指定程序的优先级，语法格式nice \-n 优先级数字 进程名称


  * [系统进程](#%E7%B3%BB%E7%BB%9F%E8%BF%9B%E7%A8%8B)
* [【1】、进程基本概述](#tid-CMrF5a)
* [【2】、监控进程状态](#tid-pFWdfh)
* [1、ps命令](#tid-aG6Ny2)
* [2、top命令](#tid-jBfQnM)
* [【3】、管理进程状态](#tid-5XtiQm)
* [【4】、后台进程](#tid-pjf3kC)
* [1、临时后台进程](#tid-tbmdty):[wgetCloud机场](https://longdu.org)
* [2、持久化后台进程](#tid-kWcZ8x)
* [【5】、进程优先级](#tid-hmyapD)

   ![](https://github.com/cnblogs_com/blogs/825724/galleries/2406923/o_240629122743_PixPin_2024-05-29_10-50-58.png)    - **本文作者：** [我亦无他，为手熟尔](https://github.com)
 - **本文链接：** [https://github.com/xuruizhao/p/18568958](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
