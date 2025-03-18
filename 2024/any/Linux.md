Linux
2022年2月24日
9:26


[[toc]]


========================

# ubuntu  安装  deb包

apt-get  install -y  ./typora_0.11.18_amd64.deb

-y  是自动同意安装。
必须带路径，不然就去仓库  根据名字搜索，而不是  安装本地软件包

========================

scp ..  怎么表示windows路径，用  /e  从linux下载下来了，但是E盘找不到。其他的  要么  提示  不是目录，要么提示无法解析hostname

直接winscp。

用.  可以。。这个会下载到  目前所在的文件夹下。就是：
C:\Users\ssss\dll>scp root@123.123.123:/root/file/ad.txt .
这个会把ad.txt  下载到  C盘的  Users/ssss/dll  文件夹下。
绝对路径好像没有办法。

========================

# ls  -ltr  
详细信息，按时间倒序，反转

========================

# useradd passwd
useradd  用户名
passwd  用户名

useradd  参数：
-u  UID,  指定UID，必须  >= 500,  且没有其他用户使用。
-g GID/GROUPNAME，指定默认组，可以是GID  或  GROUPNAME，
-G  GOUPS：  指定额外组
-c COMMENT  ，用户的注释信息
-d PATH  ，  用户的  home  目录。
       -d 目录 指定用户主目录，如果此目录不存在，则同时使用-m选项，可以创建主目录。
       -g 用户组 指定用户所属的用户组。
       -G 用户组，用户组 指定用户所属的附加组。
       -s Shell文件 指定用户的登录Shell。
       -u 用户号 指定用户的用户号，如果同时有-o选项，则可以重复使用其他用户的标识号。

用户创建后，会在  home  下  增加一个  用户名的  文件夹。
。。不过  我在home  下没有看到  新增用户的  文件夹，而且  su  新用户，也没有  文件夹。

。。也没有  cat /etc/password  文件。这个应该是保存  密码的，看网上说  这个里面  会有  新用户的，  但是  这个文件都没有。。  应该是  cat /etc/passwd  。这个有

userdel username  删除用户username
rm -rf username   删除用户username所在目录   。。  感觉有点。。。

userdel -r ztl  删除  ztl  用户的  系统文件，主要是  /etc/passwd, /etc/shadown. /etc/group ,还有  主目录。

su username  切换到刚新建的用户。  exit退出。

# groupadd groupdel groupmod
groupadd  用户组
groupdel  yonghuzu

groupadd  选项  用户组
选项是  -g GID，指定新用户组的  组标识号。  -o,一般与-g  一起使用，表示新用户组的  id  可以和  系统已有的用户组  id  相同

groupmod

cat /etc/group  可以看到  所有组和  id
修改用户权限  有一个简单的方法：  直接  vi /etc/password  进行修改。

关于uid：
0  表示  管理员  (root)
1-500  是  系统用户
501 -  65535  是普通用户。

# groups usermod
groups   查看当前登录用户的同组人员
groups  yonghuming,  查看  yonghuming  这个用户的  所在组，及组内成员

usermod  选项  用户名
常用选项，-c,-d,-m,-g,-G,-s,-u,-o  等。  这些选项和  useradd  中一致。

passwd 选项 用户名

可使用的选项：
    -l 锁定口令，即禁用账号。
    -u 口令解锁。
    -d 使账号无口令。
    -f 强迫用户下次登录时修改口令。

passwd -d sam   删除sam  用户的密码，登录的时候就不需要  输入密码了。

========================

========================

# chgrp
change group，  用于  变更文件或目录的  所属群组。
和chown  不同，chgrp  允许  普通用户改变  文件所属的组，只要该用户是  该组的一员。

chgrp [-cfhRv][--help][--version][所属群组][文件或目录...] 或 chgrp [-cfhRv][--help][--reference=<参考文件或目录>][--version][文件或目录...]

-c 或 --changes：效果类似"-v"参数，但仅回报更改的部分。
-f 或 --quiet 或 --silent： 　不显示错误信息。
-h 或 --no-dereference： 　只对符号连接的文件作修改，而不改动其他任何相关文件。
-R 或 --recursive： 　递归处理，将指定目录下的所有文件及子目录一并处理。
-v 或 --verbose： 　显示指令执行过程。
--help： 　在线帮助。
--reference=<参考文件或目录>： 　把指定文件或目录的所属群组全部设成和参考文件或目录的所属群组相同。
--version： 　显示版本信息。

[root@localhost test]# ll
---xrw-r-- 1 root root 302108 11-13 06:03 log2012.log
[root@localhost test]# chgrp -v bin log2012.log
[root@localhost test]# ll
---xrw-r-- 1 root bin  302108 11-13 06:03 log2012.log
将 log2012.log 文件由 root 群组改为 bin 群组。

[root@localhost test]# ll
---xrw-r-- 1 root bin  302108 11-13 06:03 log2012.log
-rw-r--r-- 1 root root     61 11-13 06:03 log2013.log
[root@localhost test]#  chgrp --reference=log2012.log log2013.log
[root@localhost test]# ll
---xrw-r-- 1 root bin  302108 11-13 06:03 log2012.log
-rw-r--r-- 1 root bin      61 11-13 06:03 log2013.log
改变文件 log2013.log 的群组属性，使得文件 log2013.log 的群组属性和参考文件 log2012.log 的群组属性相同。

========================

# chmod
change  mode,  控制用户对文件的权限  的命令

linux/unix  的文件调用权限  分为3级：  文件所有者owner，用户组group，其他用户

![](../_resources/feec46709fcf44b7ab9eac6aca5dd782.jpg)

只有owner  和  超级用户  有  修改文件或目录  的权限。
可以使用绝对模式  ，符号模式  指定文件的权限。

![](../_resources/f8a3f808b40049e9b55d38caf74c51cf.png)

chmod [-cfvR] [--help] [--version] mode file...

mode  权限设定，格式如下：
[ugoa...][[+-=][rwxX]...][,...]

u代表owner，g代表与该文件的owner  一个用户组，o代表其他人，a代表3者都是。
+  代表增加权限，  -代表取消权限，  =  表示唯一设定权限
r，可读，w可写，x可执行，X  只有当文件是目录文件，或者其他类型的用户有可执行权限时，才将文件权限设置为  可执行。

其他参数
-c  如果该文件权限确实已经更改，才显示其更改动作。
-f  如果该文件权限无法
-v  显示权限变更的详细资料
-R  对目前目录下的所有文件和子目录  进行相同的权限变更
--help
--version

|     |     |     |     |
| --- | --- | --- | --- |
| rwx | 111 | 7   | 读写执行 |
| rw- | 110 | 6   | 读写  |
| r-x | 101 | 5   | 读  执行 |
| r-- | 100 | 4   | 读   |
| -wx | 011 | 3   | 写  执行 |
| -w- | 010 | 2   | 写   |
| --x | 001 | 1   | 执行  |
| --- | 000 | 0   | 无   |

765的解释：
7：  111 rwx，  属主  的权限
6：  110 rw-，  属组  的权限
5：  101 r-x，  其他用户的权限

将文件 file1.txt 设为所有人皆可读取 :
chmod ugo+r file1.txt

将文件 file1.txt 设为所有人皆可读取 :
chmod a+r file1.txt

将文件 file1.txt 与 file2.txt 设为该文件拥有者，与其所属同一个群体者可写入，但其他以外的人则不可写入 :
chmod ug+w,o-w file1.txt file2.txt

为 ex1.py 文件拥有者增加可执行权限:
chmod u+x ex1.py

将目前目录下的所有文件与子目录皆设为任何人可读取 :
chmod -R a+r *

chmod a=rwx file
chmod 777 file
2个相同效果

chmod ug=rwx,o=x file
chmod 771 file
效果相同

若用 chmod 4755 filename 可使此程序具有 root 的权限。

========================

# chown
，change  owner，用于设置  文件所有者  和  文件关联组的  命令。

需要root权限。

chown [-cfhvR] [--help] [--version] user[:group] file...
    user : 新的文件拥有者的使用者 ID
    group : 新的文件拥有者的使用者组(group)
    -c : 显示更改的部分的信息
    -f : 忽略错误信息
    -h :修复符号链接
    -v : 显示详细的处理信息
    -R : 处理指定目录以及其子目录下的所有文件
    --help : 显示辅助说明
    --version : 显示版本

把 /var/run/httpd.pid 的所有者设置 root：
chown root /var/run/httpd.pid

将文件 file1.txt 的拥有者设为 runoob，群体的使用者 runoobgroup :
chown runoob:runoobgroup file1.txt

将当前前目录下的所有文件与子目录的拥有者皆设为 runoob，群体的使用者 runoobgroup:
chown -R runoob:runoobgroup *

把 /home/runoob 的关联组设置为 512 （关联组ID），不改变所有者：
chown :512 /home/runoob

========================

========================

# service logstash status
展现的信息不够

## journalctl -xe | grep logstash

## journalctl -xe | grep logstash > 1.txt

========================

# arch
获得x86_64  或  AARCH64

========================

# rpm -ivh xxx.rpm

## rpm -qa | grep xxx

## rpm -qi xxx
这个有  name  属性

## rpm -e
上面的name

========================

# tar -zxvf filebeat.tar.gz -C /opt/filebeat

========================

# ps -ef|grep 123123
获得pid为123123的信息。可以获得软件的地址。

========================

========================

========================

========================

========================

========================

========================

========================

========================

========================

========================

已使用 Microsoft OneNote 2016 创建。


----------
# ubuntu安装指定版本的软件
ubuntu22，安装gcc，安装g++，g++时，libc6-dev 报错，它依赖了2.35-0ubuntu3，但是ubuntu22安装了 2.35-0ubuntu3.1.

使用命令： `sudo apt intstall libc6=2.35-0ubuntu3` 来降版本




# 清理磁盘

清理日志文件
`sudo rm -rf /var/log/*`

清理临时文件
`sudo rm -rf /tmp/*`

清理软件缓存
`sudo apt-get clean`

检查大文件
`sudo find / -type f -size +100M`

清理无用软件
`sudo apt-get autoremove`


# man
man 后面的数字含义：
|数字|含义|
|--|--|
|1 |用户操作的命令，如ls |
|2 |内核系统调用的程序接口 |
|3 |C库函数程序接口 |
|4 |特殊文件 |
|5 |文件格式 |
|6 |游戏和娱乐 |
|7 |其他杂项 |
|8 |系统管理命令 |

不是所有系统都包含以上数值


# 压缩指定时间后的日志文件

`tar -zcvf aaa.tar.gz --after-date='2 days ago' /log/app/`

这个是 2天前的 0点之后。


# find

名字
`find . -name "Abc"`  // 名字，名称完整匹配。
`find . -iname "abc"`     // 忽略大小写
。。find 默认当前路径
`find . -iname "*abc"`

`find . -iname "pom\.xml"`

大于500mb的文件
`find . -size +500M`

小于5m
`find . -size -5M`

深度
`find -maxdepth 2 -size +500M`

按时间划分，下面是 1天内 修改的
`find -mtime -1`
`find -mmin -10`     // 10min内修改的

1分钟内 创建 的
`find -cmin -1`

查找目录，directory
`find -type d`      // f, l：  file，符号




# 文件系统

## ext4
大多数linux的默认文件系统。

日志文件系统，可以跟踪文件在磁盘上的位置并记录对磁盘的更改。
ext4非常高效和可靠


## btrfs

没有同样可靠的历史记录

Btrfs最显著的特点是其写时复制（COW）方法，它在修改数据之前将数据复制到磁盘的另一个位置。由于采用了写时复制方法，Btrfs大大减小了数据损坏的风险。

对数据块和元数据进行校验和，这是防止数据损坏的另一项预防措施


## zfs

特性：汇集存储

大多数文件系统使用单独的文件管理器，而这两个组件在汇集存储系统中合并在一起。

如果您有==多个磁盘驱动器==，这是一个很好的功能，因为它可以==将它们的存储容量合并为一个统一的文件系统==。

ZFS还具有与Btrfs相同的许多功能（包括COW、快照和数据校验和验证），使您可以放心地知道您的数据是有效和完整的。


## reiser4

Reiser4以其高效的日志记录和小文件存储而脱颖而出。它还具有原子性，即仅允许文件更改完全完成或完全不进行，并防止部分更改导致损坏。


## xfs

内部划分成分配组使其能够同时运行多个I/O操作，这使得它在多个处理器或核心并行运行时成为一个很好的选择。
它还包括xfsdump和xfsrestore，这两个都是有用的文件备份和恢复工具。


## jfs


JFS是一个日志文件系统，但它只记录元数据，并以略高的写入速度为代价，对文件恢复的彻底性有所减少。



# 文件目录层次

BV1zzQnYtEaQ


二进制文件目录
- /bin/，OS核心程序，如mount ls cd，在用户登录之前加载。
- /usr/local/bin，管理员安装的可执行文件
- /usr/bin，OS的非核心程序，大部分用户程序。 usr是 unix system resource
- /sbin，需要root权限的管理工具，如 iptable，sshd

当一个二进制文件出现在多个目录中时，你可以调整 PATH 变量中 路径的顺序 来 调用想要的二进制文件

---

库
- /lib，包含了 /bin 和 /sbin 中二进制文件 所需 的 共享库文件
- /usr/lib，包含 /usr 下的 bin所需的共享库

通过调整 LD_LIBRARY_PATH 中路径顺序，来改变 库文件的搜索顺序

---

配置
- /etc，所有配置，包括 fstab，ssh_config

---

用户数据
- /home
- /root

---

日志，缓存 (快速变化的数据)
- /var，特别是 log目录

---

易失性运行时信息
- /run， 如 systemd详情，用户session，日志守护进程

系统服务 通过 run 目录 利用 套接字进行实时通信


---

系统监控
- /proc，通过通信通道 检查OS的整体状态， CPU信息，文件挂载点
- /sys，展示较低层级的内核和硬件对象，对 设备，模块，网络堆栈，VF虚拟文件 等 进行精细监控和配置




# strace
跟踪 系统调用






# java CPU占用太高，查询步骤

3个命令
1. top
2. top -H -p 进程ID   // 可以找到线程ID
3. printf '0x%x\n' 线程ID   // 转为16进制
4. jstack 进程ID | grep 0x线程ID -A 20   // 显示20行



# ==shell==

## 多个文件总大小 awk

`ls *jpg -l | awk '{sum += $5} END {print sum / 1024 / 1024 " MB"}'`



