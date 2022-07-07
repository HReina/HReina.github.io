---
title: 部署ceph集群
date: 2022-03-14 20:58:06
tags: 
    - 云计算
categories:
    - 云计算
    - Ceph
comments: true
---

> ​             本文是在基于5台vmare虚拟机，系统为centos7环境上部署安装ceph集群的整个过程。

官方文档：[安装 Ceph — Ceph 文档](https://docs.ceph.com/en/latest/install/)

<!-- more -->

###### 准备环境

5台vm，系统:centos7，至少1核1G内存每台，每台node角色的机器至少挂载1块不少于5G的空闲盘为
osd存储。

| 主机名      | IP              | 作用                   |
| :---------- | --------------- | ---------------------- |
| admin       | 192.168.xxx.101 | admin--安装ceph-deploy |
| node1       | 192.168.xxx.102 | mon/mgr/osd            |
| node2       | 192.168.xxx.103 | osd                    |
| node2       | 192.168.xxx.104 | osd                    |
| ceph-client | 192.168.xxx.105 | client                 |

1. 给三台node节点添加一块大小5G以上的磁盘。
2. IP地址中的xx需要根据自己的vmware分配的nat网段来确定
3. 所有节点修改主机名并相互解析
4. 关闭所有机器的防火墙和selinux
5. 所有节点创建普通用户并设置密码--所有节点都操作

```
[root@admin ~]# systemct1 stop fi rewal1d
[root@admin ~]# systemct1 disable firewalld
[root@admin ~]# setenforce 0
[root@admin ~]# sed -i 's/enforcing/disab1ed/' /etc/selinux/config
[root@admin ~]# useradd cephu
[root@admin ~]# echo 1 | passwd --stdin cephu
[root@admin ~]# hostnamect1 set- hostname admin
[root@admin ~]# cat >> /etc/hosts <<eof
192.168.72.101 admin
192.168.72.102 nodel
192.168.72.103 node2
192.168.72.104 node3
192.168.72.105 ceph-client
eof
```

  6.确保各 Ceph 节点上新创建的用户都有 sudo 权限--所有节点操作

```bash
[root@admin ~]# visudo
root ALL=(ALL) ALL
cephu ALL=(root) NOPASSWD:ALL
```

7. 实现ssh无密码登录（admin节点操作）

```bash
[root@admin ~]# su - cephu
[cephu@admin ~]$ ssh-keygen
[cephu@admin ~]$ ssh-copy-id cephu@ceph-client
[cephu@admin ~]$ ssh-copy-id cephu@node1
[cephu@admin ~]$ ssh-copy-id cephu@node2
[cephu@admin ~]$ ssh-copy-id cephu@node3
```

8. 在admin节点用root用户添加~/.ssh/config配置文件，并进行如下设置，这样 ceph-deploy 就能用
   你所建的用户名登录 Ceph 节点了

  ```bash
  [root@admin ~]# mkdir ~/.ssh
  [root@admin ~]# vim ~/.ssh/config
  Host node1
  Hostname node1
  User cephu
  Host node2
  Hostname node2
  User cephu
  Host node3
  Hostname node3
  User cephu
  ```

   9.添加下载源，安装ceph-deploy（admin节点，root用户）

```bash
[root@admin ~]# vim /etc/yum.repos.d/ceph.repo
[ceph-noarch]
name=Ceph noarch packages
baseurl=https://download.ceph.com/rpm-luminous/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
[root@admin ~]# yum makecache
[root@admin ~]# yum update
[root@admin ~]# vim /etc/yum.conf
keepcache=1
[root@admin ~]# yum install -y ceph-deploy
```

10. 安装 ntp（所有节点）

* 选择任何一台机器当ntp时间服务器，其他的节点当时间服务器的客户端跟服务器同步时间

```bash
[root@admin ~]# yum install -y ntp
[root@admin ~]# vim /etc/ntp.conf
# 有4行server的位置，把那4行server行注释掉，填写以下两行
server 127.127.1.0
fudge 127.127.1.0 stratum 10
[root@admin ~]# systemctl start ntpd
[root@admin ~]# systemctl enable ntpd

```

* 其他所有节点

```bash
[root@ceph-client ~]# yum install ntpdate -y
[root@ceph-client ~]# ntpdate admin
```

###### 部署ceph集群

   没有特别说明以下所有操作均是在admin节点，cephu用户下执行

1. 创建cephu操作的目录，所有ceph-deploy命令操作必须在该目录下执行

```bash
[root@admin ~]# su - cephu
[cephu@admin ~]$ mkdir my-cluster
```

2. 创建集群

   首先在这里需要先下载一个包并安装否则会报错

```bash
[cephu@admin ~]$ wget
https://files.pythonhosted.org/packages/5f/ad/1fde06877a8d7d5c9b60eff7de2d452
f639916ae1d48f0b8f97bf97e570a/distribute-0.7.3.zip
[cephu@admin ~]$ unzip distribute-0.7.3.zip
[cephu@admin ~]$ cd distribute-0.7.3
[cephu@admin distribute-0.7.3]$ sudo python setup.py install
```

​        创建集群

```bash
[cephu@admin ~]$ cd my-cluster/
[cephu@admin my-cluster]$ ceph-deploy new node1
[cephu@admin my-cluster]$ ls
ceph.conf ceph-deploy-ceph.log ceph.mon.keyring
```

3. 安装luminous-12.2.13在(脚本方式在admin节点)

   目标：在node1,node2,node3三个节点上安装ceph和ceph-radosgw主包

   **方法1：利用官方脚本全自动安装**

   * 脚本会帮助node1,node2,node3创建epel源和ceph源，且自动安装ceph和ceph-radosgw主包
   * 这一步时间很长，容易超时，可以利用手动安装

```bash
[cephu@admin my-cluster]$ ceph-deploy install --release luminous node1 node2 node3
```

​       如果ceph和ceph-radosgw安装不上，则采用方法2

​       测试是否安装完成：分别在node1 node2 node3中确认安装版本为12.2.13

```bash
[cephu@node1 ~]$ ceph --version
ceph version 12.2.13 (584a20eb0237c657dc0567da126be145106aa47e) luminous
(stable)
```

​       **方法2：手动部署安装三台机器分别创建：三台node节点相同操作**

      * 安装epel源

```bash
[root@node1 ~]# yum install -y epel-release
```

      * 创建ceph源

```bash
[root@node1 ~]# vim /etc/yum.repos.d/ceph.repo
[Ceph]
name=Ceph packages for $basearch
baseurl=http://mirrors.aliyun.com/ceph/rpm-luminous/el7/$basearch
enabled=1
gpgcheck=0
type=rpm-md
gpgkey=https://mirrors.aliyun.com/ceph/keys/release.asc
priority=1
[Ceph-noarch]
name=Ceph noarch packages
baseurl=http://mirrors.aliyun.com/ceph/rpm-luminous/el7/noarch
enabled=1
gpgcheck=0
type=rpm-md
gpgkey=https://mirrors.aliyun.com/ceph/keys/release.asc
priority=1
[ceph-source]
name=Ceph source packages
baseurl=http://mirrors.aliyun.com/ceph/rpm-luminous/el7/SRPMS
enabled=1
gpgcheck=0
type=rpm-md
gpgkey=https://mirrors.aliyun.com/ceph/keys/release.asc
priority=1
[cephu@node1 ~]$ sudo yum install ceph ceph-radosgw -y
```

​        测试是否安装完成：分别在node1 node2 node3中确认安装版本为12.2.13

```bash
[cephu@node1 ~]$ ceph --version
ceph version 12.2.13 (584a20eb0237c657dc0567da126be145106aa47e) luminous
(stable)
```

4. 初始化mon：admin节点--cephu用户执行

```bash
[cephu@admin my-cluster]$ ceph-deploy mon create-initial
```

5. 赋予各个节点使用命令免用户名权限

```bash
[cephu@admin my-cluster]$ ceph-deploy admin node1 node2 node3
```

6. 安装ceph-mgr：只有luminous才有，为使用dashboard做准备

```bash
 [cephu@admin my-cluster]$ ceph-deploy mgr create node1
```

7. 添加osd，各个节点上提供存储空间的磁盘大小不能太小，最好5G以上，注意检查你的磁盘名字

```bash
[cephu@admin my-cluster]$ ceph-deploy osd create --data /dev/sdb node1
[cephu@admin my-cluster]$ ceph-deploy osd create --data /dev/sdb node2
[cephu@admin my-cluster]$ ceph-deploy osd create --data /dev/sdb node3
```

​    命令中/dev/sdb是在各个节点上为osd准备的空闲磁盘（无需分区格式化，如果有分区需要指定具体              	分区），通过如下命令查看

```bash
[cephu@admin my-cluster]$ ssh node1 lsblk -f
```

​    最后通过如下命令查看集群状态，如果显示health_ok，3个osd up就成功了

```bash
[cephu@admin my-cluster]$ ssh node1 sudo ceph -s
cluster:
   id: 8b765619-0fe7-4826-a79e-d0d0be9c2931
   health: HEALTH_OK

services:
   mon: 1 daemons, quorum node1
   mgr: node1(active)
   osd: 3 osds: 3 up, 3 in

data:
   pools: 0 pools, 0 pgs
   objects: 0 objects, 0B
   usage: 3.01GiB used, 12.0GiB / 15.0GiB avail
   pgs:

```

###### Dashboard的配置

在node1上操作，把ceph-mgr和ceph-mon安装在同一个主机上，最好只有一个ceph-mgr

```bash
[root@node1 ~]# su - cephu
```

1. 创建管理域秘钥

```bash
[cephu@node1 ~]$ sudo ceph auth get-or-create mgr.node1 mon 'allow profile
mgr' osd 'allow *' mds 'allow *'
[mgr.node1]
key = AQCn735fpmEODxAAt6jGd1u956wBIDyvyYmruw==
```

2. 开启 ceph-mgr 管理域

```bash
[cephu@node1 ~]$ sudo ceph-mgr -i node1
```

3. 查看ceph的状态：确认mgr的状态为active

```bash
[cephu@node1 ~]$ sudo ceph status
cluster:
  id: 8b765619-0fe7-4826-a79e-d0d0be9c2931
  health: HEALTH_OK

services:
  mon: 1 daemons, quorum node1
  mgr: node1(active, starting)
  osd: 3 osds: 3 up, 3 in

data:
  pools: 0 pools, 0 pgs
  objects: 0 objects, 0B
  usage: 3.01GiB used, 12.0GiB / 15.0GiB avail
  pgs:
```

4. 打开dashboard模块

```bash
[cephu@node1 ~]$ sudo ceph mgr module enable dashboard
```

5. 绑定开启dashboard模块的ceph-mgr节点的ip地址，ip地址为mgr节点的ip地址,也就是node1的ip
地址

```bash
[cephu@node1 ~]$ sudo ceph config-key set mgr/dashboard/node1/server_addr
192.168.72.102
```

6. web登录：浏览器地址栏输入 mgr地址:7000

![ceph](/images/ceph.png)

###### 配置客户端使用rbd

创建块设备之前需要创建存储池，存储池相关命令需要在mon节点执行--也就是规划好的node1节点

1. 创建存储池

```bash
[cephu@node1 ~]$ sudo ceph osd pool create rbd 128 128
pool 'rbd' created
```

> 少于5个osd，pg数量为128
> 5-10个osd，pg数量为512
> 10-50个osd，pg数量为4096

2. 初始化存储池

```bash
[cephu@node1 ~]$ sudo rbd pool init rbd
```

3. 准备客户端client：客户端操作
    另备一台主机，系统centos7用来作为client。主机名为client，ip：192.168.xx.105。修改hosts文件实现和admin节点的主机名互通。

4. 为client安装ceph

```bash
[root@ceph-client ~]# yum -y install python-setuptools
```

​      需要进行ceph集群环境内容
​      需要安装luminous-12.2.13的第2中方式手动安装ceph

5. 在admin节点赋予client使用命令免用户名权限

```bash
[cephu@admin my-cluster]$ ceph-deploy admin ceph-client
```

6. 修改client下该文件的读权限
7. 修改client下的ceph配置文件：这一步是为了解决映射镜像时出错问题

```bash
[cephu@ceph-client ~]$ sudo sh -c 'echo "rbd_default_features = 1" >>
/etc/ceph/ceph.conf'
```

8. client节点创建块设备镜像：单位是M，这里是4个G

```bash
 [cephu@ceph-client ~]$ rbd create foo --size 4096
```

9. client节点映射镜像到主机

```bash
[cephu@ceph-client ~]$ sudo rbd map foo --name client.admin
```

10. client节点mount块设备

```bash
[cephu@ceph-client ~]$ sudo mkfs.ext4 -m 0 /dev/rbd/rbd/foo
[cephu@ceph-client ~]$ sudo mkdir /mnt/ceph-block-device
[cephu@ceph-client ~]$ sudo mount /dev/rbd/rbd/foo /mnt/ceph-block-device
[cephu@ceph-client ~]$sudo mount /dev/rbd/rbd/foo /mnt/ceph-block-device
[cephu@ceph-client ceph-block-device]$ sudo touch test.txt
```

客户端重起之后，设备需要重新做一下映射(第9步)，不然可能会卡死

