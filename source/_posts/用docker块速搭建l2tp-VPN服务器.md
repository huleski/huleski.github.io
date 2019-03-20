---
title: 用docker块速搭建l2tp VPN服务器
categories: VPN
tags: VPN
date: 2019-03-20 18:57:23
---

参考: [docker快速搭建l2tp VPN服务器](https://blog.csdn.net/xindoo/article/details/52830609)

 docker就像一个大仓库, 我们只要从仓库中取出我们需要的应用(vpn服务器)运行起来就能用了。服务器是国外云服务器, 系统是CentOS7

搭建vpn服务器
-----------

**1. 获取 l2tp 的镜像**
```
docker pull fcojean/l2tp-ipsec-vpn-server
```

**2. 配置PSK，用户名和密码。先创建配置文件vpn.env**
```
vim vpn.env
```
将一下配置复制到vpn.env文件中。该配置psk为“abcdefg”，两个用户名user1和user2，密码都为123456
```
VPN_IPSEC_PSK=abcdefg
VPN_USER_CREDENTIAL_LIST=[{"login":"user1","password":"123456"},{"login":"user2","password":"123456"}]
```

**3. 加载 IPsec NETKEY 内核模块**
```
sudo modprobe af_key
```

**4. 启动镜像**

该命令加载env文件做配置文件，并将对应端口和服务器的端口做绑定。
注意：在执行该命令时当前目录下必须有之前创建的vpn.env文件，否则会报错。
```
docker run \
    --name vpn-server \
    --env-file ./vpn.env \
    -p 500:500/udp \
    -p 4500:4500/udp \
    -v /lib/modules:/lib/modules:ro \
    -d --privileged \
    fcojean/l2tp-ipsec-vpn-server
```

查看当前的vpn服务是否正常启动
```
docker logs vpn-server
```

有如下输出则表示已正常启动vpn服务器。

![](https://img-blog.csdn.net/20180708143943177?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vuc2VlbmJsYWRl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**5. 关闭防火墙，先查看防火墙状态**

如果防火墙未开启则跳过此部
```
systemctl stop firewalld
```
除了关闭防火墙，云服务器还需要配置访问规则, 即开放端口, 一定要记得**开放500端口和4500端口**, 这两个端口是vpn使用的.

配置客户端
--------
Windows10系统, 设置VPN:
1. 右键单击系统托盘中的无线/网络图标。 
2. 选择 打开网络与共享中心。 
3. 单击 设置新的连接或网络。 
4. 选择 连接到工作区，然后单击 下一步。 
5. 单击 使用我的Internet连接 (VPN)。 
6. 在 Internet地址 字段中输入你的 VPN 服务器 IP。 
7. 在 目标名称 字段中输入任意内容。单击 创建。 
8. 返回 网络与共享中心。单击左侧的 更改适配器设置。 
9. 右键单击新创建的 VPN 连接:

![](https://img-blog.csdn.net/20180707181238162?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vuc2VlbmJsYWRl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![](https://img-blog.csdn.net/20180707181543103?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vuc2VlbmJsYWRl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

10. 填写之前创建的vpn服务器ip, PSK, 用户名和密码, 点击保存. 注意VPN类型选择 [使用预共享秘钥的L2TP/IPSec PSK]
11. 以管理员身份启用命令提示符：执行以下两条命令

```
REG ADD HKLM\SYSTEM\CurrentControlSet\Services\PolicyAgent /v AssumeUDPEncapsulationContextOnSendRule /t REG_DWORD /d 0x2 /f
```

```
REG ADD HKLM\SYSTEM\CurrentControlSet\Services\RasMan\Parameters /v ProhibitIpSec /t REG_DWORD /d 0x0 /f
```

12.  设置VPN连接属性.请参考: [win10设置VPN连接](https://blog.csdn.net/qq_37729885/article/details/80577877) 
    
13.  至此, 设置完毕, 请关机重启, 再点击连接VPN, 成功! (下图为方便说明, 连接了两个VPN)

![](https://img-blog.csdn.net/20180707182606548?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Vuc2VlbmJsYWRl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

安卓手机不用设置可以直接连VPN, 但注意vpn类型选择 L2TP/IPSec PSK,** 建议先用安卓手机**（连WiFi，流量可能有意外）检测vpn是否搭建成功, 再尝试Windows10连接VPN

