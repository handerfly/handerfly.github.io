#linux中ulimit作用
一、作用

Linux对于每个用户，系统限制其最大进程数。为提高性能，可以根据设备资源情况，设置各linux 用户的最大进程数。

复制代码
ulimit主要是用来限制进程对资源的使用情况的，它支持各种类型的限制，常用的有：

内核文件的大小限制

进程数据块的大小限制

Shell进程创建文件大小限制

可加锁内存大小限制

常驻内存集的大小限制

打开文件句柄数限制

分配堆栈的最大大小限制

CPU占用时间限制用户最大可用的进程数限制

Shell进程所能使用的最大虚拟内存限制
复制代码
 

有个疑问：服务器资源是有限制的，limits.conf会限制资源，但是对于ulimit来说限制是粒度是哪些？  

1）用户 ? 进程？ 

2）如果是进程的限制，系统中会有多个进程，那所有的进程同时需要受限于用户或整个系统的资源限制。

 

 

二、用法

ulimit使用的基本格式为：ulimit [options] [limit]

具体的options参数含义如下表所示：

复制代码
选项 含义

-a 显示当前系统所有的limit资源信息。 

-H 设置硬资源限制，一旦设置不能增加。

-S 设置软资源限制，设置后可以增加，但是不能超过硬资源设置。

-c 最大的core文件的大小，以 blocks 为单位。

-f 进程可以创建文件的最大值，以blocks 为单位.

-d 进程最大的数据段的大小，以Kbytes 为单位。

-m 最大内存大小，以Kbytes为单位。

-n 查看进程可以打开的最大文件描述符的数量。

-s 线程栈大小，以Kbytes为单位。

-p 管道缓冲区的大小，以Kbytes 为单位。

-u 用户最大可用的进程数。

-v 进程最大可用的虚拟内存，以Kbytes 为单位。

-t 最大CPU占用时间，以秒为单位。

-l 最大可加锁内存大小，以Kbytes 为单位。
复制代码
 其中ulimit -n用于限制进程能够打开的文件描述符的最大数目。因为任何设备在linux下都是文件，通信的接口也有专门的接口文件负责，所以linux下进程tcp链接的最大并发量也受限于该值。

复制代码
[root@centos5 ~]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
max nice                        (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 4096
max locked memory       (kbytes, -l) 32
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
max rt priority                 (-r) 0
stack size              (kbytes, -s) 10240
cpu time               (seconds, -t) unlimited
max user processes              (-u) 4096
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited||<
复制代码
其中 "open files (-n) 1024 "是Linux操作系统对一个进程打开的文件句柄数量的限制(也包含打开的SOCKET数量,可影响MySQL的并发连接数目)。

这个值可用ulimit 命令来修改,但ulimit命令修改的数值只对当前登录用户的目前使用环境有效,系统重启或者用户退出后就会失效.

 

有时候为了方便起见，也可以将用户资源的限制统一由一个文件来配置，这个文件就是/etc/security/limits.conf，该文件不但能对指定用户的资源进行限制，还能对指定组的资源进行限制。该文件的使用规则如下：

 <domain> <type> <item> <value> 

其中：

domain表示用户或者组的名字，还可以使用 * 作为通配符，表示任何用户或用户组。

Type 表示限制的类型，可以有两个值，soft 和 hard，分别表示软、硬资源限制。软限制是指限制用户同时打开的文件数目，硬限制是指系统根据硬件资源（主要指内存）计算出来的最多可打开的文件数目。通常软限制低于硬限制；建议不要将软限制和硬限制修改过大。

item 表示需要限定的资源名称，常用的有nofile、cpu、stack等。分别表示最大打开句柄数、占用的cpu时间、最大的堆栈大小。

value 表示限制各种资源的具体数值。

除了limits.conf文件之外，还有一个/etc/security/limits.d目录，可以将资源限制创建一个文件放到这个目录中，默认系统会首先去读取这个目录下的所有文件，然后才去读取limits.conf文件。所有资源限制设置完成后，退出shell终端，再次登录shell终端后，ulimit设置即可自动生效。

 注意：各种配置的生效方式。 

 

对于需要做许多 socket 连接并使它们处于打开状态的Java 应用程序而言，最好通过使用 ulimit -n xx 修改每个进程可打开的文件数，缺省值是 1024。

ulimit -n 4096 将每个进程可以打开的文件数目加大到4096，缺省为1024

其他建议设置成无限制（unlimited）的一些重要设置是：
数据段长度：ulimit -d unlimited
最大内存大小：ulimit -m unlimited
堆栈大小：ulimit -s unlimited
CPU 时间：ulimit -t unlimited
虚拟内存：ulimit -v unlimited

 

 

配置生效方式

    暂时地，适用于通过 ulimit 命令登录 shell 会话期间。
    永久地，通过将一个相应的 ulimit 语句添加到由登录 shell 读取的文件中， 即特定于 shell 的用户资源文件，如：
1)、解除 Linux 系统的最大进程数和最大文件打开数限制：
        vi /etc/security/limits.conf
        # 添加如下的行
        * soft noproc 11000
        * hard noproc 11000
        * soft nofile 4100
        * hard nofile 4100
      说明：* 代表针对所有用户，noproc 是代表最大进程数，nofile 是代表最大文件打开数
2)、让 SSH 接受 Login 程式的登入，方便在 ssh 客户端查看 ulimit -a 资源限制：
        a、vi /etc/ssh/sshd_config
            把 UserLogin 的值改为 yes，并把 # 注释去掉
        b、重启 sshd 服务：
              /etc/init.d/sshd restart
3)、修改所有 linux 用户的环境变量文件：
    vi /etc/profile
    ulimit -u 10000
    ulimit -n 4096
    ulimit -d unlimited
    ulimit -m unlimited
    ulimit -s unlimited
    ulimit -t unlimited
    ulimit -v unlimited
 保存后运行#source /etc/profile 使其生效

 

在使用ulimit时，有以下几种使用方法：

（1）在用户环境变量中加入

如果用户使用的是bash，那么就可以在用户目录的环境变量文件.bashrc或者.bash_profile中加入“ulimit -u 128”来限制用户最多可以使用128个进程。

（2）在应用程序的启动脚本中加入

如果应用程序是tomcat，那么就可以在tomcat的启动脚本startup.sh脚本中加入“ulimit -n 65535”来限制用户最多可以使用65535个文件描述符。

（3）直接在shell命令终端执行ulimit命令

这种方法的资源限制仅仅在执行命令的终端生效，退出或者关闭终端后，设置失效，并且这个设置不影响其它shell终端。

 

公司服务器需要调整 ulimit的stack size 参数调整为unlimited 无限，使用ulimit -s unlimited时只能在当时的shell见效，重开一个shell就失效了。。于是得在/etc/profile 的最后面添加ulimit -s unlimited 就可以了，source /etc/profile使修改文件生效。


如果碰到类似的错误提示ulimit: max user processes: cannot modify limit: 不允许的操作 ulimit: open files: cannot modify limit: 不允许的操作
为啥root用户是可以的？普通用户又会遇到这样的问题？
看一下/etc/security/limits.conf大概就会明白。
linux对用户有默认的ulimit限制，而这个文件可以配置用户的硬配置和软配置，硬配置是个上限。超出上限的修改就会出“不允许的操作”这样的错误。
在limits.conf加上
*        soft    noproc 10240
*        hard    noproc 10240
*        soft    nofile 10240
*        hard    nofile 10240
就是限制了任意用户的最大进程数和文件数为10240。

 

如何设置普通用户的ulimit值
1、vim /etc/profile
增加 ulimit -n 10240
source /etc/profile 重新启动就不需要运行这个命令了。
2、修改/etc/security/limits.conf
增加
*      hard     nofile     10240   
\\限制打开文件数10240
3、测试，新建普通用户，切换到普通用户使用ulit -a 查看是否修改成功。
 

查看系统级别资源限制：

1.系统级别：

sysctl -a (-a:显示当前所有可用的值)

系统总限制：cat /proc/sys/fs/file-max 等一系列值，修改/etc/sysctl.conf 中也可以控制

 /proc/sys/fs/file-nr，可以看到整个系统目前使用的文件句柄数量

 

修改此硬限制的方法是修改/etc/rc.local脚本，在脚本中添加如下行：

echo 8000037 > /proc/sys/fs/file-max

这是让Linux在启动完成后强行将系统级打开文件数硬限制设置为800037。

这表明这台Linux系统最多允许同时打开(即包含所有用户打开文件数总和)800037个文件，是Linux系统级硬限制，所有用户级的打开文件数限制都不应超过这个数值。该值是Linux系统在启动时根据系统硬件资源状况计算出来的最佳的最大同时打开文件数限制，如果没有特殊需要，不应该修改此限制，除非想为用户级打开文件数限制设置超过此限制的值。

 

查找文件句柄问题的时候，lsof可以很方便看到某个进程开了那些句柄.也可以看到某个文件/目录被什么进程占用了.

 

2.session 设置：

ulimit -a # 查看所有的 

ulimit -S -n1024 #设置当前会话的打开文件数软连接数为 1024.

ulimit -H -n1024 #设置当前会话的打开文件数硬连接数为 1024.

ulimit -n 996 #设置当前会话的打开文件数硬&&软连接数都为 1024.

 

3.设置用户(只针对用户的每个进程)：

#<domain> <type> <item> <value>

#

* soft nofile 32768

* hard nofile 65536

 

ulimit -n vs. file-max ? 
简单的说, ulimit -n控制进程级别能够打开的文件句柄的数量, 而max-file表示系统级别的能够打开的文件句柄的数量。

ulimit -n的设置在重启机器后会丢失，因此需要修改limits.conf的限制，limits.conf中有两个值soft和hard，soft代表只警告，hard代表真正的限制

Cat /etc/security/limits.conf

*               soft    nofile          150000  
*               hard    nofile          150000  
这里我们把soft和hard设置成一样的。

“cat /proc/sys/fs/file-max”，或“sysctl -a | grep fs.file-max”查看系统能打开的最大文件数。查看和设置例如：

复制代码
[root@vm014601 ~]# sysctl -a |grep fs.file-max  
fs.file-max = 200592  
[root@vm014601 ~]# echo "fs.file-max = 2005920" >> /etc/sysctl.conf   
[root@vm014601 ~]# sysctl -p  
[root@vm014601 ~]# cat /proc/sys/fs/file-max                          
2005920
复制代码
file-nr是只读文件，第一个数代表了目前分配的文件句柄数；第二个数代表了系统分配的最大文件句柄数；比如线上系统查看结果：

复制代码
# cat /proc/sys/fs/file-max  
1106537  
# cat /proc/sys/fs/file-nr       
1088  0       1106537  
# lsof | wc -l  
1506
复制代码
可以看到file-nr和lsof的值不是很一致，但是数量级一致。为什么会不一致？原因如下： 

写到lsof是列出系统所占用的资源,但是这些资源不一定会占用打开文件号的。如共享内存,信号量,消息队列,内存映射.等,虽然占用了这些资源,但不占用打开文件号.

我曾经在前端机上很长时间都无法得到lsof | wc -l 的结果，这个时候可以通过file-nr粗略的估算一下打开的文件句柄数。
