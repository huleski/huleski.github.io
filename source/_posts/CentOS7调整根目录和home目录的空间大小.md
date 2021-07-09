---
title: CentOS7调整根目录和home目录的空间大小
categories: CentOS7
tags: CentOS7
date: 2021-07-09 14:22:34
---

当安装完 CentOS7 操作系统，发现磁盘分区大小错误，或者后期使用过程发现 `/home` 还剩余很多空间，`/` 下空间不足，需要将 `/home` 下空间重新分配给 `/` 目录。

### 查看分区空间和格式 

```bash
[root@localhost portainer]$ df -hT
文件系统                    类型    容量    已用  可用  已用% 挂载点
devtmpfs                 devtmpfs  3.8G     0   3.8G    0%     /dev
tmpfs                     tmpfs    3.8G     0   3.8G    0%     /dev/shm
tmpfs                     tmpfs    3.8G   51M   3.8G    2%     /run
tmpfs                     tmpfs    3.8G     0   3.8G    0%     /sys/fs/cgroup
/dev/mapper/centos-root    xfs     50G    27G    23G    56%    /
/dev/sda1                  xfs    1014M  193M   822M    19%    /boot
/dev/mapper/centos-home    xfs     873G   33M   873G    1%     /home
```

这里 `/dev/mapper/centos-root` 就是根目录, 只有50G太小了, 而 `/dev/mapper/centos-home` 有800多G, 因此可以分一部分 `/home` 给 `/`目录

可以看到 /home 分区是 xfs 格式，这里特别注意：

1. ext2/ext3/ext4文件系统的调整命令是resize2fs（增大和减小都支持）

```text
lvextend -L 120G /dev/mapper/centos-home //增大至120G
lvextend -L +20G /dev/mapper/centos-home //增加20G
lvreduce -L 50G /dev/mapper/centos-home //减小至50G
lvreduce -L -8G /dev/mapper/centos-home //减小8G
resize2fs /dev/mapper/centos-home //执行调整
```

2. xfs文件系统的调整命令是xfs_growfs（只支持增大）

```text
lvextend -L 120G /dev/mapper/centos-home //增大至120G
lvextend -L +20G /dev/mapper/centos-home //增加20G
xfs_growfs /dev/mapper/centos-home //执行调整
```

就是说：**xfs文件系统只支持增大分区空间的情况，不支持减小的情况**

硬要减小的话，只能在减小后将逻辑分区重新通过 `mkfs.xfs` 命令重新格式化才能挂载上，这样的话这个逻辑分区上原来的数据就丢失了。如果有重要文件就先复制备份到其他地方, 然后再进行下面的操作

```bash
# 卸载 /home 分区 (注意: 不要在home目录下执行这个操作)
umount /home

# 将 /home 分区减小600G（根据自己实际情况设定大小）
lvreduce -L -600G /dev/mapper/centos-home

# 格式化 /home 分区
mkfs.xfs /dev/mapper/centos-home -f

# 重新挂载 /home 分区
mount /dev/mapper/centos-home /home/

# 查看剩余空间
vgdisplay

# 上面空余的 600G 分到 / 分区下
lvextend -L +600G /dev/mapper/centos-root
```

再次查看分区, 分配成功

```bash
[root@localhost portainer]$ df -h
文件系统                  容量   已用  可用   已用%   挂载点
devtmpfs                 3.8G     0  3.8G    0%     /dev
tmpfs                    3.8G     0  3.8G    0%     /dev/shm
tmpfs                    3.8G   51M  3.8G    2%     /run
tmpfs                    3.8G     0  3.8G    0%     /sys/fs/cgroup
/dev/mapper/centos-root  650G   27G  624G    5%     /
/dev/sda1               1014M  193M  822M   19%     /boot
/dev/mapper/centos-home  273G   33M  273G    1%     /home
```
