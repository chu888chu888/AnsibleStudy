# 概述

大家知道什么叫做云计算吗？事实上，目前并没有一个确定的定义。然而概括来讲，所谓的云计算，指的就是把你的软件和服务统一部署在数据中心，统一管理，从而实现高伸缩性。
###云计算具有以下特性:
+ 虚拟化和自动化
+ 服务器，存储介质，网络等资源都可以随时替换
+ 所有的资源都由云端统一管理
+ 高度的伸缩性以满足业务需求
+ 集中于将服务传递给业务.


## 安装

### 使用国内镜像的问题
在国内使用各种开源项目,都逃脱不过一个翻墙的问题,我们安装ansible也是一样的,但是这里我提出一个方法就是使用国内镜像的方法,这个方法更快都好.

### 临时使用

```
sudo aptitude install python-dev
sudo pip install ansible -i https://pypi.tuna.tsinghua.edu.cn/simple
sudo pip install markupsafe -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 修改配置文件
为了修改默认的镜像源，在我的 Linux系统中，需要修改/root/.pip/pip.conf。因为使用pip时，
总是添加sudo运行。

```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

## 第一例子

### 解禁Ubuntu 上的root用户
为了保证分发的实验的优先级别,我们用root用户

```
ubuntu安装好后，root初始密码（默认密码）不知道，需要设置。
1、先用安装时候的用户登录进入系统
2、输入：sudo passwd  按回车
3、输入新密码，重复输入密码，最后提示passwd：password updated sucessfully
此时已完成root密码的设置
4、输入：su root
切换用户到root
```

### 解禁Ubuntu SSH下root登录问题

```
SSH服务器，可以通过SSH协议来访问远程服务器，代替telnet和ftp。但是ubuntu默认是不启用root用户也不允许root远程登录的。所以需要先启用root用户
 
启用root用户：sudo passwd root      //修改密码后就启用了。
 
安装OpenSSH server：
1. 使用apt命令安装openssh server
$ sudo apt-get install openssh-server
 
 
2. 可以对 openssh server进行配置
$ sudo vi /etc/ssh/sshd_config
找到PermitRootLogin no一行，改为PermitRootLogin yes
找#PermitRootLogin without-password 并注释掉
#PermitRootLogin without-password
 
3. 重启 openssh server
$ sudo service ssh restart
 
4. 客户端如果是ubuntu的话，则已经安装好ssh client,可以用下面的命令连接远程服务器。
$ ssh xxx.xxx.xxx.xxx
如果是windows系统的话，可以使用SSH Secure Shell等ssh软件进行远程连接。
```

### 添加ssh互信


```
root@hadoopmaster:~# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
0f:f7:b7:e6:1e:cb:88:7b:39:06:80:fe:76:47:32:f7 root@hadoopmaster
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|                 |
|       .         |
|      . .        |
|     .  S..      |
|      .  ++.o    |
|       .  .*.oo  |
|        o ..*+E+ |
|       . .o=.=*  |
+-----------------+



root@hadoopmaster:~/.ssh# ls
id_rsa  id_rsa.pub  known_hosts
root@hadoopmaster:~/.ssh# cat ./id_rsa.pub >>./authorized_keys
root@hadoopmaster:~/.ssh# ls
authorized_keys  id_rsa  id_rsa.pub  known_hosts

root@hadoopmaster:~# scp ~/.ssh/id_rsa.pub root@hadoopslave1:/root/
root@hadoopslave1's password:
id_rsa.pub                                                                            100%  399     0.4KB/s   00:00
root@hadoopmaster:~# scp ~/.ssh/id_rsa.pub root@hadoopslave2:/root/
The authenticity of host 'hadoopslave2 (192.168.1.166)' can't be established.
ECDSA key fingerprint is b6:a9:7d:5b:8c:92:0b:39:f2:fb:d8:69:49:df:b0:22.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'hadoopslave2,192.168.1.166' (ECDSA) to the list of known hosts.
root@hadoopslave2's password:
id_rsa.pub                                                                            100%  399     0.4KB/s   00:00
root@hadoopmaster:~#

mkdir ~/.ssh       # 如果不存在该文件夹需先创建，若已存在则忽略
cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
rm ~/id_rsa.pub    # 用完就可以删掉了

```

### 建立ansible配置目录

```
如果ansible目录不存在,就需要自已建立了
root@hadoopmaster:~# cd /etc/ansible/
root@hadoopmaster:/etc/ansible# ls
hosts

hosts目录内容

[hadoop]
192.168.1.159
192.168.1.76
192.168.1.166
```

### 执行ansible 命令

```
root@hadoopmaster:/etc/ansible# ansible hadoop -m command -a 'uptime'
192.168.1.159 | SUCCESS | rc=0 >>
 16:00:44 up 20 days,  5:25,  2 users,  load average: 0.14, 0.05, 0.05

192.168.1.166 | SUCCESS | rc=0 >>
 16:00:44 up 20 days,  5:25,  1 user,  load average: 0.02, 0.02, 0.05

192.168.1.76 | SUCCESS | rc=0 >>
 16:00:44 up 20 days,  5:25,  1 user,  load average: 0.00, 0.01, 0.05
```

###常用模块

1 setup模块,查看远程主机基本信息

```
root@hadoopmaster:/etc/ansible# ansible hadoop -m setup
```
2 ping 用来测试远程主机的运行状态

```
root@hadoopmaster:/etc/ansible# ansible hadoop -m ping
192.168.1.166 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}


192.168.1.76 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.1.159 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
3 file模块

```
相关选项如下：
force：需要在两种情况下强制创建软链接，一种是源文件不存在，但之后会建立的情况下；另一种是目标软链接已存在，需要先取消之前的软链，然后创建新的软链，有两个选项：yes|no
group：定义文件/目录的属组
mode：定义文件/目录的权限
owner：定义文件/目录的属主
path：必选项，定义文件/目录的路径
recurse：递归设置文件的属性，只对目录有效
src：被链接的源文件路径，只应用于state=link的情况
dest：被链接到的路径，只应用于state=link的情况
state：
       directory：如果目录不存在，就创建目录
       file：即使文件不存在，也不会被创建
       link：创建软链接
       hard：创建硬链接
       touch：如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间
       absent：删除目录、文件或者取消链接文件
```

例子

```
远程文件符号链接创建
ansible storm_cluster -m file -a "src=/etc/resolv.conf dest=/tmp/resolv.conf state=link"

远程文件信息查看
ansible storm_cluster -m command -a "ls –al /tmp/resolv.conf"

远程文件符号链接删除
ansible storm_cluster -m file -a "path=/tmp/resolv.conf state=absent"

远程文件信息查看
ansible storm_cluster -m command -a "ls -al /tmp/resolv.conf"
```

4 copy模块

```
复制文件到远程主机
相关选项如下：
backup：在覆盖之前，将源文件备份，备份文件包含时间信息。有两个选项：yes|no
content：用于替代“src”，可以直接设定指定文件的值
dest：必选项。要将源文件复制到的远程主机的绝对路径，如果源文件是一个目录，那么该路径也必须是个目录
directory_mode：递归设定目录的权限，默认为系统默认权限
force：如果目标主机包含该文件，但内容不同，如果设置为yes，则强制覆盖，如果为no，则只有当目标主机的目标位置不存在该文件时，才复制。默认为yes
others：所有的file模块里的选项都可以在这里使用
src：被复制到远程主机的本地文件，可以是绝对路径，也可以是相对路径。如果路径是一个目录，它将递归复制。在这种情况下，如果路径使用“/”来结尾，则只复制目录里的内容，如果没有使用“/”来结尾，则包含目录在内的整个内容全部复制，类似于rsync。
```

例子

```
示例：
## 将本地文件“/etc/ansible/ansible.cfg”复制到远程服务器
# ansible storm_cluster -m copy -a "src=/etc/ansible/ansible.cfg dest=/tmp/ansible.cfg owner=root group=root mode=0644"

## 远程文件信息查看
# ansible storm_cluster -m command -a "ls -al /tmp/ansible.cfg"


```

5 command模块

```
## 在远程主机上执行命令
相关选项如下：
creates：一个文件名，当该文件存在，则该命令不执行
free_form：要执行的linux指令
chdir：在执行指令之前，先切换到该目录
removes：一个文件名，当该文件不存在，则该选项不执行
executable：切换shell来执行指令，该执行路径必须是一个绝对路径
```

例子

```
# ansible storm_cluster -m command -a "uptime"

```