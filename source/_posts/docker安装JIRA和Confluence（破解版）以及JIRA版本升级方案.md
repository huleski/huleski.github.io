---
title: docker安装JIRA和Confluence（破解版）以及JIRA版本升级方案
categories: JIRA
tags: 
    - JIRA
    - docker
date: 2019-07-08 16:41:58
---

> 本文将演示通过Docker安装JIRA和Confluence，并破解过程。
本文只做个人学习研究之用，不得用于商业用途！

## 安装JIRA

### 1. 获取资源

- Docker镜像 [Github链接](https://github.com/cptactionhank)
- 补丁工具(atlassian-agent.jar) [Github链接](https://github.com/pengzhile/atlassian-agent/releases/download/v1.2/atlassian-agent-v1.2.zip)  原链接已失效, 附百度云:

> 百度网盘地址：
> 
> 链接：https://pan.baidu.com/s/17zNwlp3sd1PLSCxPVjDwfQ 
> 
> 提取码：b84z 
> 
> 复制这段内容后打开百度网盘手机App，操作更方便哦

### 2. 制作Docker破解容器

编写Dockerfile文件：

```bash
FROM cptactionhank/atlassian-jira-software:7.12.3

USER root

# 将代理破解包加入容器
COPY "atlassian-agent.jar" /opt/atlassian/jira/

# 设置启动加载代理包
RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/jira/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/jira/bin/setenv.sh
```

将下载好的`atlassian-agent.jar`文件放在Dockerfile同目录下，例如：

```bash
- JIRA
    |-Dockerfile
    |-atlassian-agent.jar
```

构建镜像, 执行命令(注意后面有一个点)

```bash
docker build -t jira/jira:v7.12.3 .
```

构建成功后会显示 'Successfully built...` 字样

启动容器，执行命令：

```bash
docker run -d -p 10086:8080 \
    -v /home/jira/data:/var/atlassian/jira \
    --restart always --name=jira \
    --health-cmd="curl --silent --fail localhost:8080 || exit 1" \
    jira/jira:v7.12.3
```

访问: `http://127.0.0.1:10086,可见JIRA配置页面

### 3. 破解

在见到JIRA配置页面后, 进行相关配置, 当要求输入许可证时

![进入JIRA配置页面](http://pubgmjp23.bkt.clouddn.com/10973635-4168e67779acb0b2_%E7%9C%8B%E5%9B%BE%E7%8E%8B.web.jpg)

复制服务器ID: `BY9B-GWD1-1C78-K2DE`, 在存放`atlassian-agent.jar`的目录下执行命令, 生成许可证:

```bash
# 需替换邮箱（test@test.com）、名称（JIRA）、
# 访问地址（http://192.168.0.1）、服务器ID（BY9B-GWD1-1C78-K2DE）
# 为你的信息

java -jar atlassian-agent.jar -d -m test@test.com -n JIRA -p jira -o http://192.168.0.89 -s BY9B-GWD1-1C78-K2DE
```

![生成许可证](http://pubgmjp23.bkt.clouddn.com/xukezheng.jpg)

复制下面生成的一长串许可证填写到页面中, 完成破解

## 安装 Confluence（6.13.0)

### 1. 编写Dockerfile文件

```bash
FROM cptactionhank/atlassian-confluence:6.13.0

USER root

# 将代理破解包加入容器
COPY "atlassian-agent.jar" /opt/atlassian/confluence/

# 设置启动加载代理包
RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh
```

### 2. 构建镜像

将下载好的`atlassian-agent.jar`文件放在Dockerfile同目录下，例如：

```bash
- Confluence
    |-Dockerfile
    |-atlassian-agent.jar
```

构建镜像, 执行命令(注意后面有一个点)

```bash
docker build -f Dockerfile -t confluence/confluence:6.13.0 .
```

启动容器，执行命令：

```bash
docker run -d -p 8090:8090 confluence/confluence:6.13.0
```

访问`http://127.0.0.1:8090`,参照JIRA的安装流程，进行操作。

生成confluence许可方法可参照前面JIRA的破解过程, 这里不再赘述

## JIRA版本升级

可以参考[官方文档](https://confluence.atlassian.com/jirakb/startup-check-jira-data-version-too-low-to-be-upgraded-872266914.html)

![升级步骤](http://pubgmjp23.bkt.clouddn.com/jira.png)

由于之前安装的是jira6.4版本的, 现在需要升级到7.12.3版本, 根据官方提示需要先升级到7.0版本的jira再过渡升级到7.12.3

### 1. 备份数据

**a. 备份数据库**
   
   登录jira6.4版本, 进入设置 -> 系统 -> 备份系统, 输入备份的文件名字: 比如jira

![备份系统](http://pubgmjp23.bkt.clouddn.com/2E6%7D47Y%29@KMCUP3%25%295B%29%60%7BO.png)

点击备份后会在备份界面提示的某个路径下生成一个`jira.zip`数据库的备份文件

**b . 备份图片**

复制 `/var/atlassian/application-data/jira/data`下的`attachments`和`avatars`文件夹出来,备份好

### 2. 升级过渡版

如果不过数据库文件过渡处理, 在jira v7.12.3中是无法使用的

获取JIRA v7.0.11 镜像并启动

```bash
docker pull dchevell/jira-software:7.0.11
docker run -d -p 8080:8080 dchevell/jira-software:7.0.11
```

启动后访问 `http://127.0.0.1:8080`, 开始配置, 当需要许可证时, 上面的方法破解不了, 由于只是做一个过度版本, 可以去官网注册然后申请一个30天有效期的许可证, 然后输入进去就好了

同样在备份系统的同级目录中点击恢复系统

![恢复系统](http://pubgmjp23.bkt.clouddn.com/ertysdysdrtysert.png)

按照提示将前面备份好的文件放到他指定目录下,点击复原, 如果需要填写许可证那就填写许可证在复原

复原成功后再重复上一步骤进行备份
备份后将在 docker容器 `/var/atlassian/jira/export`目录下生成一个`.zip`数据库的备份文件

### 3. 升级最终版

按照上面的步骤把jira v7.12.3运行起来, 并将数据库备份文件放入恢复界面指定的目录之下进行数据复原.

把前面图片备份文件分别放入`/var/atlassian/application-data/jira/data/`下即可

升级成功!