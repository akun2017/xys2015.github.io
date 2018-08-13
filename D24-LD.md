====================================      
  
第24天 周1 20180813 授课老师-李泳谊    
作者: 邢永胜         
  
====================================   
  
# 定时任务回顾  
  
1. 定时备份  
2. 定时执行命令  
3. 书写排错流程  
4. 企业故障案例: 定时任务没有定向到空或追加到文件  
5. 企业故障案例: 定时任务和运行脚本，PATH环境没有正确添加，导致命令找不到  
  
## 如何让1个脚本开机自启动（面试题）  
  
方法1: 把命令放到 /etc/rc.local  
在CentOS 7中使用时，需要给/etc/rc.d/rc.local加上执行权限  
  
方法2: chkconfig  
  
    1) 脚本存放在  
    vim /etc/init.d/oldboyd  
    echo oldboy  
  
    1) 脚本要有执行权限  
    chmod +x vim /etc/init.d/oldboyd  
  
    1) 在开头增加chkconfig要求的格式  
    对脚本增加`# chkconfig: 2345 99 98`，  
    其中2345是运行级别，  
    99 表示启动的顺序，第99号，  
    98 表示关机顺序  
    自己写的脚本，一般都用99 99  
      
    1) 加入到chkconfig管理  
    chkconfig --add oldboyd  
  
    1) 检查chkconfig | grep oldboy  
  
    演示如下:  
    [root@as4k ~]# cat /etc/init.d/oldboyd   
    # chkconfig: 2345 99 98  
    echo oldboy-chkconfig-hello  
    [root@as4k ~]# chmod +x /etc/init.d/oldboyd  
    [root@as4k ~]# chkconfig --add oldboyd  
    [root@as4k ~]# chkconfig | grep oldboyd  
    oldboyd        	0:off	1:off	2:on	3:on	4:on	5:on	6:off  
    [root@as4k ~]# /etc/init.d/oldboyd  
    oldboy-chkconfig-hello  
      
显示系统脚本的执行顺序      
    -(减号)表示默认开启不启动，可以手动让其启动  
    [root@as4k ~]# cat  /etc/init.d/* | grep 'chkconfig:' | tail -10  
    # chkconfig:	- 12 87  
    # chkconfig: - 13 99  
    # chkconfig: 2345 12 88  
    # chkconfig: 345 1 99  
    # chkconfig: - 65 10  
    # chkconfig: - 99 01  
    # chkconfig: 2345 55 25  
    # chkconfig: - 85 15  
    # chkconfig: 12345 01 99  
    # chkconfig: 12345 26 75  
  
  
## 小结  
  
面试题: 如何让1个脚本开机启动  
1 /etc/rc.local  
2 chkconfig  
3 如何通过chkconfig管理  
    放在/etc/init.d/ 且有执行权限  
    符合chkconfig格式  
  
# 用户管理  
  
## useradd 添加用户  
  
小题1:   
添加1个用户lidao999，uid为999，禁止远程登录系统，不创建家目录。  
此题也可描述为，创建一个uid为999的虚拟用户。  
  
解答:  
    useradd -u 999 -s /sbin/nologin -M lidao999  
解释:  
    -M 不创建家目录  
    -s 指定解释器  
    -u 指定uid  
    -g 指定主要组名称  
演示:  
    [root@as4k ~]# useradd -u 999 -s /sbin/nologin -M lidao999  
    [root@as4k ~]# id lidao999  
    uid=999(lidao999) gid=999(lidao999) groups=999(lidao999)  
    [root@as4k ~]# grep lidao999 /etc/passwd  
    lidao999:x:999:999::/home/lidao999:/sbin/nologin  
    [root@as4k ~]# ls -l /home/lidao999  
    ls: cannot access /home/lidao999: No such file or directory  
    [root@as4k init.d]# su - lidao999  
    su: warning: cannot change directory to /home/lidao999: No such file or directory  
    This account is currently not available.  
  
小题2:  
添加1个虚拟用户mysql，用户和用户组的uid都是666  
    [root@as4k ~]# useradd -u 666 -s /sbin/nologin -M mysql  
    [root@as4k ~]# id mysql  
    uid=666(mysql) gid=666(mysql) groups=666(mysql)  
      
## userdel 删除用户  
  
userdel username  
    默认不会删除用户老家  
    -r 删除用户的老家与邮箱等相关文件，连锅端。  
  
一般我们不会直接，把一个用户删除，因为难保，用户的家目录里没有重要文件，如果  
该用户不在使用，只需在/etc/passwd里，把该用户的注释掉即可。（前面加#）  
  
    # steam:x:606:609:this_is_my_boss:/home/steam:/bin/bash  
    [root@as4k init.d]# id steam  
    id: steam: No such user  
  
## usermod  修改用户信息  
  
    参数与uesradd类似  
  
# 组的管理  
  
## 主要组和附加组  
      
组分为: 主要组，附加组。使用useradd不加参数创建用户时，默认就是创建一个  
和用户名同名的主要组，主要组不能被删除，只能被修改。  
  
## 修改主要组和附加组 uesrmod -G -g  
  
-g 更改的是主要组  
把lidao999的主要组更改为root  
    # id lidao999  
    uid=999(lidao999) gid=999(lidao999) groups=999(lidao999)  
    # usermod -g oldboy lidao999  
    # id lidao999  
    uid=999(lidao999) gid=500(oldboy) groups=500(oldboy)  
      
更改附加组 -G  
为lidao999添加oldboy,www附加组  
    # id lidao999  
    uid=999(lidao999) gid=999(lidao999) groups=999(lidao999)  
    # usermod -G oldboy,www lidao999  
    # id lidao999  
    uid=999(lidao999) gid=999(lidao999) groups=999(lidao999),500(oldboy),506(www)  
  
清空附加组  
    # id lidao999  
    uid=999(lidao999) gid=999(lidao999) groups=999(lidao999),500(oldboy),506(www)  
    # usermod -G '' lidao999  
    # id lidao999  
    uid=999(lidao999) gid=999(lidao999) groups=999(lidao999)  
    # 可以看到，清空附加组，主要组不受影响  
  
## passwd 修改密码  
  
passwd username (only root can use this syntax)  
passwd  
  
passwd默认是交互式修改密码，普通用户只能修改自己的密码并且需要原密码，root用户可  
修改所有人的密码，不需要原密码，包括修改root用户自己的。  
  
--stdin  非交互式修改密码 适用于脚本批量修改密码  
    echo 123456 | passwd --stdin oldboy  
  
# 查询用户信息  
  
id       显示用户信息，uid，gid，所属组，顺便还能判断用户是否存在  
w        显示谁登陆到系统，并且看他们在干什么  
last     所有用户每次的登录情况  
lastlog  所有用户最近一次的登录情况  
  
     
# 用户和组相关文件  
  
[root@as4k ~]# ls -l /etc/passwd /etc/group /etc/shadow /etc/gshadow   
-rw-r--r-- 1 root root 1077 Aug 12 23:55 /etc/group    # 存放用户信息  
---------- 1 root root  863 Aug 12 23:55 /etc/gshadow  # 用户组信息，每个组里有多少用户  
-rw-r--r-- 1 root root 2059 Aug 12 23:55 /etc/passwd   # 用户密码信息   
---------- 1 root root 2055 Aug 13 00:21 /etc/shadow   # 用户组密码信息  
  
老男孩IT教育-/etc/passwd每一列的含义  
https://www.processon.com/view/link/5a4ecf47e4b01acda589aa65  
  
## 骨架目录 /etc/skel  
  
每个新用户的家目录模板  
[root@as4k ~]# ls -la /etc/skel/  
total 20  
drwxr-xr-x.  2 root root 4096 Jul 11 20:49 .  
drwxr-xr-x. 79 root root 4096 Aug 13 00:33 ..  
-rw-r--r--.  1 root root   18 Mar 23  2017 .bash_logout  
-rw-r--r--.  1 root root  176 Mar 23  2017 .bash_profile  
-rw-r--r--.  1 root root  124 Mar 23  2017 .bashrc  
  
## useradd new_username 创建用户流程 待完成  
  
uesradd oldboy   
1 创建家目录 修改权限 所有者  
2 创建同名主要组  
3 把/etc/skel下面所有文件，复制到/home/oldboy/目录里  
  
配置文件回顾:  
    # 全局 - 所有用户生效  
        /etc/profile  
        /etc/bashrc  
    # 局部 - 单个用户生效  
        ~/.bashrc    
        ~/.bash_profile    
    # 如果矛盾，局部生效  
  
# sudo（临时成为root）  
  
oldboy用户想查看系统日志，4种解决方案:  
    1. sudo         # 正是我们需要的  
    2. cat加上suid   # 给大了  
    3. 日志加上r权限  # 给大了  
    4. 给root密码    # 给大了  
  
sudo，尚方宝剑，让普通用户以root运行某个命令  
sudo -k 清空缓存密码  
sudo -l 查看当前用户能够使用的特殊命令  
  
## 授予oldboy用户ls, cat, echo权限  
  
编辑:  visudo === vim /etc/sudoers  
类似:  crontab -e  === vim /var/spool/cron/root  
visudo有语法检查，另外直接编辑/etc/sudoers，需要w!，方能强制保存。  
  
实现步骤:  
  
1 visudo打开，sudo权限配置文件，找到90，91行，内容大概如下  
    90 ## Allow root to run any commands anywhere  
    91 root    ALL=(ALL)       ALL  
  
2 在第92行写入如下内容，并保存  
    90 ## Allow root to run any commands anywhere  
    91 root    ALL=(ALL)       ALL  
    92 oldboy  ALL=(ALL)       /bin/ls, /bin/cat, /bin/echo  
  
    # 这里需要使用命令的绝对路径  
    $ \which ls cat echo  
    /bin/ls  
    /bin/cat  
    /bin/echo  
  
3 切换到oldboy用户进行测试  
  
查看有哪些高级权限oldboy用，一开始需要oldboy用户的密码，无关内容已略去  
    [oldboy@as5k ~]$ sudo -l  
    User oldboy may run the following commands on this host:  
        (ALL) /bin/ls, (ALL) /bin/cat, (ALL) /bin/echo  
  
查看root用户的家目录  
    [oldboy@as5k ~]$ ls /root/  
    ls: cannot open directory /root/: Permission denied  
    [oldboy@as5k ~]$ sudo ls /root/  
    alex.txt	 data	hello2	   install.log  
    anaconda-ks.cfg  hello	hello.txt  
  
查看/etc/shadow  
    [oldboy@as5k ~]$ ls -l /etc/shadow  
    ----------. 1 root root 846 Jul 16 05:31 /etc/shadow  
    [oldboy@as5k ~]$ cat /etc/shadow  
    cat: /etc/shadow: Permission denied  
    [oldboy@as5k ~]$ sudo cat /etc/shadow | head -3  
    root:$6$rwzYQnccX/Ia9wdB0:17723:0:99999:7:::  
    bin:*:17246:0:99999:7:::  
    daemon:*:17246:0:99999:7:::  
  
## 无需密码 /bin/* !  
  
无需密码:  
    oldboy  ALL=(ALL) NOPASSWD: /bin/ls, /bin/cat, /bin/echo  
    oldboy  ALL=(ALL) NOPASSWD: ALL # 运行所有命令，而且不需要输入密码。  
有了NOPASSWD，执行sudo，就无需再输入密码了。  
  
/bin/*  
给上/bin/下所有命令root权限  
    oldboy ALL=(ALL) /bin/*  
/bin所有权限，排除危险权限  
    oldboy ALL=(ALL) /bin/*, !/bin/rm, !/bin/su  
  
  
## NOPASSWD 针对单条命令 待完成  
  
# 跳板机  
  
用户审计 行为审计  
记录用户的操作  
跳板机，堡垒机，检查站，收费站  
用来当作连接实际服务器的中介，可以记录用户的操作。  
实现跳板机的硬件: 齐治堡垒机  
开源软件: jumpserver  
自己写: python  shell脚本  
  
# 命令行提示符变成 -bash-4.1$ 故障 ？  
  
每个用户的家目录里都有，该用户的配置文件，它们一般是以.开头的，其中就有控制  
命令提示符的配置文件，把这些文件全部删除，就会出现本故障。但是如果系统配置  
/etc/profile，配置了命令提示符，则不会出现上述故障，解决方法就是把/etc/skel  
里的初始配置文件，复制一份到用户家目录里。  
  
    [root@as4k steam2]# ls -la  
    total 20  
    drwx------   2 steam2 steam 4096 Aug 10 05:47 .  
    drwxr-xr-x. 25 root   root  4096 Aug 10 09:06 ..  
    -rw-r--r--   1 steam2 steam   18 Mar 23  2017 .bash_logout  
    -rw-r--r--   1 steam2 steam  176 Mar 23  2017 .bash_profile  
    -rw-r--r--   1 steam2 steam  124 Mar 23  2017 .bashrc  
    [root@as4k steam2]# \rm .bash*  
    [root@as4k steam2]# ls -la  
    total 8  
    drwx------   2 steam2 steam 4096 Aug 13 00:54 .  
    drwxr-xr-x. 25 root   root  4096 Aug 10 09:06 ..  
    [root@as4k steam2]# su - steam2  
    -bash-4.1$ cp /etc/skel/.bash* /home/steam2/  
    -bash-4.1$ ls -la /home/steam2/  
    total 20  
    drwx------   2 steam2 steam  4096 Aug 13 00:55 .  
    drwxr-xr-x. 25 root   root   4096 Aug 10 09:06 ..  
    -rw-r--r--   1 steam2 oldboy   18 Aug 13 00:55 .bash_logout  
    -rw-r--r--   1 steam2 oldboy  176 Aug 13 00:55 .bash_profile  
    -rw-r--r--   1 steam2 oldboy  124 Aug 13 00:55 .bashrc  
    -bash-4.1$ logout  
    [root@as4k steam2]# su - steam2  
  
# echo pwd | bash  
  
命令解释器  
[root@as4k ~]# echo pwd | bash  
/root  
  
[root@as4k ~]# cat /etc/shells  
/bin/sh  
/bin/bash  
/sbin/nologin  
/bin/dash  # Ubuntu / debian 默认的命令解释器  
/bin/tcsh  # tcsh csh Unix使用  
/bin/csh  
  
# 总结-24天  
  
用户管理  
作业  
故障案例  
  
# 预习  
  
如何保护系统  
磁盘  
  
# 本结作业  
  
## 定时任务作业 待完成  
  
每天晚上12点打包备份/etc/目录  
1.打包备份到/backup目录   
2.删除7天之前的备份 find  
3.保留每周1的备份   find  
  
## 批量添加用户 待完成  
  
批量添加3个用户stu01,stu02....stu10,并设置123456(禁止使用for,while等循环)   
批量添加10个用户stu01,stu02....stu10,并设置8位随机密码(禁止使用for,while等循环)   
  
  
