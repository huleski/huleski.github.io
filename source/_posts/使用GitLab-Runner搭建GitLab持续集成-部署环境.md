---
title: 使用GitLab-Runner搭建GitLab持续集成/部署环境
categories: GitLab
tags: 
    - GitLab-CI
    - GitLab
date: 2019-02-26 11:22:47
---

相关背景
------
1. GitLab
> 是一套基于Ruby开发的开源Git项目管理应用，其提供的功能和Github类似，不同的是GitLab提供一个GitLab CE社区版本，用户可以将其部署在自己的服务器上，这样就可以用于团队内部的项目代码托管仓库。

2. GitLab CI
> 是GitLab 提供的持续集成服务(从8.0版本之后，GitLab CI已经集成在GitLab中了)，只要在你的仓库根目录下创建一个.gitlab-ci.yml 文件， 并为该项目指派一个Runner，当有合并请求或者Push操作时，你写在.gitlab-ci.yml中的构建脚本就会开始执行。

3. GitLab Runner
> 是配合GitLab CI进行构建任务的应用程序，GitLab CI负责yml文件中各种阶段流程的执行，而GitLab Runner就是具体的负责执行每个阶段的脚本执行，一般来说GitLab Runner需要安装在单独的机器上通过其提供的注册操作跟GitLab CI进行绑定，当然，你也可以让其和GitLab安装在一起，只是有的情况下，你代码的构建过程对资源消耗十分严重的时候，会拖累GitLab给其他用户提供政策的Git服务。

4. 持续集成/部署环境
> 持续集成是程序开发人员在频繁的提交代码之后，能有相应的环境能对其提交的代码自动执行构建(Build)、测试(Test),然后根据测试结果判断新提交的代码能否合并加入主分支当中,而持续部署也就是在持续集成之后自动将代码部署(Deploy)到生成环境上

开启GitLab可持续集成功能, 你需要通过如下两步启用GitLab CI功能


1. 为你的项目配置一个GitLab Runner
2. 新建一个.gitlab-ci.yml文件在你项目的根目录

创建GitLab Runner以及配置
------------------------

拉取官方镜像, alpine版镜像体积比较小, 也可以使用latest版
```
docker pull gitlab/gitlab-runner:alpine
```

启动gitlab-runner容器
```
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:alpine
```

执行下面命令注册一个runner :
```
docker exec -it gitlab-runner gitlab-ci-multi-runner register
```

接下来出现以下内容, 根据提示输入
```
Please enter the gitlab-ci coordinator URL:
# 示例：http://10.12.2.22
Please enter the gitlab-ci token for this runner:
# xxxxxx
Please enter the gitlab-ci description for this runner:
# 示例：test
Please enter the gitlab-ci tags for this runner (comma separated):
# 示例：test
Please enter the executor: docker, parallels, shell, kubernetes, docker-ssh, ssh, virtualbox, docker+machine, docker-ssh+machine:
# ssh
Please enter the SSH server address (e.g. my.server.com):
# 10.12.2.22
Please enter the SSH server port (e.g. 22):
# 22  
Please enter the SSH user (e.g. root):
# root
Please enter the SSH password (e.g. docker.io):
# 123456
Please enter path to SSH identity file (e.g. /home/user/.ssh/id_rsa):

Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

说明： 
 1. gitlab ci 的地址以及token，从你要配置该runner到哪个项目，就去gitlab下该项目首页右侧设置—》CI/CD Pipelines—》Specific Runners下可以找到。 
 2. gitlab-ci tags这个很重要，在项目构建流程yaml文件里面指定tag，就是匹配使用哪个tag的runner，这里我定义了test，回头再配置文件里面就指定这个tag。 
 3. executor：执行者可以有很多种，这里我们使用ssh, 登录进入后再构建。 

如果想修改注册信息, 可以编辑文件 `vim /srv/gitlab-runner/config/config.toml`, 内容如下:
```
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "wta-admin"
  url = "http://10.12.2.22/"
  token = "JKLASJDFIAOSKJ"
  executor = "ssh"
  [runners.ssh]
    user = "root"
    password = "123456"
    host = "10.12.2.22"
    port = "22"
    builds_dir = "/home/runner/builds"   # 设置构建路径
    cache_dir = "/home/runner/caches"    # 设置构建缓存路径

  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
```

修改完记得重启docker
```
docker restart gitlab-runner
```

gitlab-runner已经配置完成。

在gitlab项目的根目录新建.gitlab-ci.yml文件
---------------------------------

gitlab-ci.yml文件是用来配置GitLab CI进行构建流程的配置文件，其采用YAML语法,所以你需要额外注意要用空格来代替缩进，而不是Tabs。
.gitlab-ci.yml文件如下。查看[详细配置](https://segmentfault.com/a/1190000011890710)或[官方配置](https://docs.gitlab.com/ce/ci/yaml/README.html)

```
# 利用caches字段来指定下面将要进行的job任务中需要共享的文件目录,如果没有，
# 每个Job开始的时候，GitLab Runner都会删掉.gitignore里面的文件
cache:
  key: ${CI_BUILD_REF_NAME}
  paths:
    - target

# 利用stages关键字来定义持续构建过程中的三个阶段: package、build_docker、deploy_docker
# 1. 所有 Stages 会按照顺序运行，即当一个 Stage 完成后，下一个 Stage 才会开始
# 2. 只有当所有 Stages 完成后，该构建任务 (Pipeline) 才会成功
# 3. 如果任何一个 Stage 失败，那么后面的 Stages 不会执行，该构建任务 (Pipeline) 失败
stages:
  - package
  - build_docker
  - deploy_docker

############################### maven打包 ###############################
# 定义一个叫package的Job任务, package为job名, 可随意命名。下同
# stage字段声明属于package阶段，这个job会第一个执行，执行一些环境的初始化工作。
# script字段指定该任务执行的内容, 由于是CentOS, 此处执行shell语句。下同
package:
  stage: package
  tags:                       #这里的tags一定要属于注册时填的tags中，下面同理
    - test
  script:
    - echo "begining to execute package project"
    - docker stop test && docker rm test && docker rmi test:1.0
    - mvn clean install -Dmaven.test.skip=true
    - cp -f target/*.jar /data/
  artifacts:
    paths:
      - target/*.jar    # 将maven构建成功的jar包作为构建产出导出，可在下一个stage的任务中使用 目前没卵用

############################### 构建镜像 ############################### 
build_docker:
  stage: build_docker
  script:
    - echo "begining to execute build project"
    - docker build -t test:1.0 /data/
  tags:
    - test

############################### 部署运行 ############################### 
# only字段指定需要执行的所在分支或者标签
deploy_docker:
  stage: deploy_docker
  script:
    - echo "begining to execute deploy project"
    - docker run -d -p 80:80 --restart always --name=test test:1.0
    - echo "dev部署成功, 嘻嘻嘻......"
  only:
    - dev
  tags:
    - test

deploy_docker:
  stage: deploy_docker
  script:
    - echo "begining to execute deploy project"
    - docker run -d -p 8080:80 --restart always --name=test test:1.0
    - echo "master部署成功, 嘻嘻嘻......"
  only:
    - master
  tags:
    - test
```

创建完成后push到gitlab, 此时打开项目首页的Piplines标签页，会发现一个状态标识为pending的构建任务, gitlab-CI搭建完成