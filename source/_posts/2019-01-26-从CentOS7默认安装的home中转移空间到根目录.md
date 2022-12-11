---
title: 从CentOS7默认安装的/home中转移空间到根目录/
urlname: 1b9fc5729ba59c7025d746556f3c6057
date: '2019-01-26 10:16:32 +0800'
layout: post
categories: Centos
key: 20190126
tags:
  - centos
---

转自 [https://blog.csdn.net/evandeng2009/article/details/49814097](https://blog.csdn.net/evandeng2009/article/details/49814097)

# 一、基础概念

Cent0S 7 默认启用 LVM2（Logical Volume Manager），把机器的一块硬盘分为两个区 sda1 和 sda2，其中分区 sda1 作为系统盘/boot 挂载，少量空间；sda2 作为一个物理卷并且完全作为逻辑卷组 VG(Volume Group）centos，在这个逻辑卷组 centos 中建立三个逻辑卷 LV（Logical Volume）root 和 home 还有 swap，分别挂载到根目录/和/home 以及 swap。而两个分区 sda1 和 sda2 上都建立了文件系统 XFS，文件系统 XFS 作为 RedHat 的默认文件系统也有它的考虑，成为继 ext3，ext4 之后的主流文件系统。
几个概念的关系：M 个物理硬盘 HD 或者物理硬盘中的分区一起组建为一个逻辑卷组 VG 及存储池，在卷组 VG 中创建 N 个逻辑卷 LV，在一个逻辑卷 LV 中创建文件系统比如 xfs。物理硬盘/分区、逻辑卷有最小基本寻址单元，CentOS7 默认的大小为 4MB，二者一一对应，类似于链接或者变量引用，但是一个二者关系并非一直不变，因为物理硬盘可能发生变化而逻辑卷自动调整。创建卷组和逻辑卷，会类似于创建分区一样在磁盘开始位置写入卷的信息 VGDA（卷组描述符区域，Volume Group Descriptor Area）用于识别。逻辑卷的好处在于屏蔽物理底层支撑，可自由扩展变更，而不用担心硬盘或者分区的物理空间局限，也就不会存在为了扩展分区大小而去备份/扩展分区重新格式化硬盘等问题。
HD/分区--通过 pvcreate->PV--通过 vgcreate(vgchange)/vgextend-->VG--通过 lvcreate/lvextend-->LV--通过 mkfs-->FS--通过 xfs_growfs 等-->df 磁盘生效
但是关键点在于，CentOS 7 默认安装时/home 占用太多空间，根目录相较而言就小得多(只有 50G)，而 OpenStack 安装以及存储的东西都在根目录下。上传几个镜像说不定就把你的根目录空间耗尽。不像其他文件系统 ext3，ext4 或者 reiserfs 等，有命令（resize2fs，resize_reiserfs）直接支持缩小文件系统的大小，**默认安装的 xfs 支持扩展增大但是不支持缩小空间！**

# 二、步骤概览

所以下面要做的步骤大概为：（直接 root 用户登录系统，本机或者 ssh root 过去，如果使用当前普通用户会遇到点不必要的麻烦）

1. 备份/home/用户文件，要是没啥内容则忽略这步（为什么非要这个/home，删掉直接用 root？还是保留它，它存在也有道理的，再说生产环境还是不要只用 root）
2. umount /home 卸载并 lvremove 删除这个 home 逻辑卷，释放它的空间，vgdisplay 查看卷组可用空间大小
3. lvcreate 新建一个新的 home 卷，并在其上 mkfs 建立 xfs 文件系统，(分配挂载到/home - 不用更改/etc/fstab，重启即可，) 拷贝回来之前的内容

(这个时候空余的空间随便你分配，可以再建立别的逻辑卷，或者直接空闲下来以后使用，也可以直奔主题的走下面的第四步)

4. 把之前的 home 逻辑卷释放并分配新卷 home 之后剩下的空间，lvextend 分配给 root 卷，并用命令 xfs_growfs 扩展它的文件系统空间

# 三、相关命令

看看大概的命令有哪些

#### pv

> pvchange pvck pvcreate pvdisplay pvmove pvremove pvresize pvs pvscan

##### vb

1. vgcfgbackup vgchange vgconvert vgdisplay vgextend vgimportclone vgmknodes vgremove vgs
2. vgsplit vgcfgrestore vgck vgcreate vgexport vgimport vgmerge vgreduce vgrename
3. vgscan

##### lv

1. lvchange lvcreate lvextend lvmchange lvmdiskscan lvmetad lvmsar lvremove lvresize lvscan
2. lvconvert lvdisplay lvm

# 四、默认安装情况

##### df -h //查看磁盘使用情况

##### cat /etc/fstab

##### vgdisplay //查看逻辑卷组情况

##### lvdisplay //查看逻辑卷情况，默认三个，root、home 和交换空间 swap

# 五、操作步骤

#### 备份/home 中的用户数据

> [root[\_@\_localhost ](/localhost) /]# mkdir /backup && mv /home/\* /backup
> [root[\_@\_localhost ](/localhost) /]# ls /home/

#### 卸载这个/home 并删除逻辑卷 home

> umount /home

#### df -h //查看磁盘情况

> Filesystem Size Used Avail Use% Mounted on
> /dev/mapper/centos-root 50G 17G 34G 33% /
> devtmpfs 3.9G 0 3.9G 0% /dev
> tmpfs 3.9G 84K 3.9G 1% /dev/shm
> tmpfs 3.9G 9.0M 3.9G 1% /run
> tmpfs 3.9G 0 3.9G 0% /sys/fs/cgroup
> /dev/sda1 494M 133M 362M 27% /boot

#### lvremove /dev/centos/home //删除逻辑卷 home

> Do you really want to remove active logical volume home? [y/n]: y
> Logical volume "home" successfully removed

#### vgdisplay //查看卷组可用空间

#### 新建一个卷 home，fdisk 格式化为 8e 格式，文件系统还是搞为 xfs（同样挂载到/home）

> lvcreate -L 50G -n home centos //L 表示大小，默认单位为 M；n 表示卷名；这里的 centos 是 CentOS7 安装系统的时候就默认建立好的卷组名
> WARNING: xfs signature detected on /dev/centos/home at offset 0. Wipe it? [y/n]: y
> Wiping xfs signature on /dev/centos/home.
> Logical volume "home" created.

#### lvdisplay //查看逻辑卷 home

#### vgdisplay //再次查看卷组空间大小

#### [# vgchange -ay centos //可选步骤：激活卷组 centos，使得这个新建的 home 逻辑卷生效（用 vgchange 而不用 lvchange）]

> 3 logical volume(s) in volume group "centos" now active

##### mkfs -t xfs /dev/centos/home //在新建的逻辑卷 home 上建立 xfs 文件系统

##### mount /dev/centos/home /home/ //把这个新逻辑卷 home 挂到之前的文件夹/home 中去，直接重启用 fstab 来挂载也行

##### df -h //现在来查看磁盘使用情况

##### mv /backup/\* /home/ //再把之前拷出来的东西拷回新建的/home 中，拷贝完成就可以直接用这个普通用户来桌面登录系统了，不用重启

##### 最后再把释放出来多余的空间分配给 root 卷并 xfs_growfs 扩展文件系统

lvextend -L +823G /dev/centos/root //把剩下的 823G 现在分配给 root 卷，剩下那点渣渣空间让它闲着；+号表示在原来的基础上额外增加，不要加好则设定为具体额度
Size of logical volume centos/root changed from 50.00 GiB (12800 extents) to 873.00 GiB (223488 extents).
Logical volume root successfully resized

##### lvdisplay //查看逻辑卷和卷组情况

#### df -h //不使用 xfs_growfs 扩展文件系统，磁盘是不认得多的空间的

##### xfs_growfs /dev/centos/root //扩展 root 卷

##### df -h //再看 root 大小已经生效

# 六、命令汇总

简单总结下，就是下面几条命令：

1. mkdir /backup
2. mv /home/\* /backup/
3. umount /home
4. lvremove /dev/centos/home
5. lvcreate -L 50G -n home cents
6. mkfs -t xfs /dev/centos/home
7. mv /backup/\* /home/
8. lvextend -L +xxxG /dev/centos/root
9. xfs_growfs root
10. rm -rf /backup
