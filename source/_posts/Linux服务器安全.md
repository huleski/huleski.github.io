---
title: Linux服务器安全
categories: Linux
tags: Linux
date: 2023-03-17 10:35:36
---

### SSH访问控制配置

编辑文件 `vi /etc/ssh/sshd_config`添加下面一行:

```bash
# 只允许192.168.0.1+root用户登录
AllowUsers root@192.168.0.1
```

重启ssh: `systemctl restart sshd`

### 设置密码强度策略
在`/etc/pam.d/password-auth`和`/etc/pam.d/system-auth`文件中修改'password requisite pam_pwquality.so'那一行为：
```text
password requisite pam_pwquality.so try_first_pass type= retry=5 difok=3 minlen=8 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1
```

> 参数说明：     
> retry=5（登录或修改密码失败时可以重试的次数,默认值是1，如果用户输入的密码强度不够就退出）        
> difok=3（允许新、旧密码相同字符的个数）       
> minlen=8 （最小长度为8）     
> ucredit = -1 （至少包含一位大写字母）            
> lcredit = -1 （至少包含一位小写字母）         
> ocredit = -1 （至少包含一个特殊字符)

### 设置身份鉴别策略
设置用户登录失败处理功能，可采取结束会话、限制非法登录次数和自动退出等措施。

**注意: CentOS7中使用pam_tally2模块, CentOS8中使用pam_faillock模块**

下面以CentOS7为例, 编辑文件`vim /etc/pam.d/sshd`添加以下内容(需要重启ssh): 
```bash
auth  required  pam_tally2.so   deny=5 even_deny_root unlock_time=600 root_unlock_time=600
```

> deny 设置普通用户和root用户连续错误登陆的最大次数，超过最大次数，则锁定该用户     
> even_deny_root 也限制root用户；     
> unlock_time 设定普通用户锁定后，多少时间后解锁，单位是秒；     
> root_unlock_time 设定root用户锁定后，多少时间后解锁，单位是秒；

```bash
# 查看用户登录失败的次数
pam_tally2 -u 用户名

# 解锁指定用户(登录成功后，这个用户登录错误值将清零)
pam_tally2 -r -u 用户名
```

### 设置登陆会话超时
30分钟无操作，自动退出会话。
```bash
vim /etc/profile
TMOUT=1800 #30分钟超时
source /etc/profile
```

### 安装防恶意代码软件
安装Linux病毒查杀软件Clamav
```bash
# 安装
yum -y install zlib-devel openssl-devel clamav* clamd*
# 更新病毒库
freshclam

#重启freshclam服务
systemctl  start  clamav-freshclam.service
#查看服务状态
systemctl status clamav-freshclam.service
#开机启动程序
systemctl enable clamav-freshclam.service 
#停止freshclam服务
systemctl stop clamav-freshclam.service

# 扫描 /data 目录指令：
clamscan -ri /data  
#扫描参数：
#-r/--recursive[=yes/no]        递归所有文件        
#--log=FILE/-l FILE             增加扫描报告          
#--copy[路径]                    将受感染的文件复制到[路径]             
#--move [路径]                   将受感染的文件移动到[路径]       
#--remove [路径]                 删除受感染的文件       
#--quiet                        只输出错误消息      
#--infected/-i                  只输出感染文件       
#--suppress-ok-results/-o       跳过扫描OK的文件       
#--bell                         扫描到病毒文件发出警报声音      
#--unzip(unrar)                 解压压缩文件扫描 
```
定时杀毒, 编辑 `vim /etc/crontab`,加上一下内容:
```bash
# 让服务器每天晚上2点定时更新和杀毒
0 2 * * * /usr/bin/freshclam --quiet
30 2 * * * /usr/bin/clamscan  -ri /data --move=/tmp
```
重启定时任务: `systemctl restart crond`

### 隐藏Nginx软件版本号信息
在不隐藏的情况下，我们可以通过http响应头查看Nginx的版本号

过在nginx.conf的http段中加入 `server_tokens off` 来隐藏我们的版本号

