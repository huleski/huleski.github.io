---
title: cordova入门教程
categories: cordova
tags: cordova
date: 2019-07-29 09:33:23
---

简介 ( [Cordova官网](https://cordova.apache.org/)/[Cordova中文网](https://cordova.axuer.com/))
----

> Apache Cordova是一个开源的移动开发框架。允许你用标准的web技术-HTML5,CSS3和JavaScript做跨平台开发。 应用在每个平台的具体执行被封装了起来，并依靠符合标准的API绑定去访问每个设备的功能，比如说：传感器、数据、网络状态等。

简单来说 Cordova 就是一个能将`html/js/css`打包成各个平台应用功能的框架, 原理是他内置了一个浏览器, 然后把H5显示出来, 并能够打包成不同平台的App, 目前支持的平台有:

- Android
- Blackberry 10
- iOS
- OS X
- Ubuntu
- Windows
- WP8

安装环境 ( /data/cordova目录下操作 )
------

### 1. 安装JDK

下载`jdk-8u162-linux-x64.tar.gz` 并解压到当前目录

```bash
tar -xzvf jdk-8u162-linux-x64.tar.gz
```

编辑配置文件 `vim /etc/profile` 在最后添加:

```bash
#####  Java environment  #####
export JAVA_HOME=/data/cordova/jdk1.8.0_162
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export  PATH=${JAVA_HOME}/bin:$PATH
```

使配置立即生效:

```bash
source /etc/profile
```

### 2. 安装Android SDK

下载 Android SDK并解压 (可以在[这里](https://www.androiddevtools.cn/)下载)

```bash
wget http://dl.google.com/android/android-sdk_r24.4.1-linux.tgz
tar -xzf android-sdk_r24.4.1-linux.tgz 
```

编辑配置文件 `vim /etc/profile` 并在文件最后添加:

```bash
#####  Android environment  #####
export ANDROID_HOME=/data/cordova/android-sdk-linux
export PATH=$ANDROID_HOME/tools:$PATH
```

使配置立即生效:

```bash
source /etc/profile
```

其他可能用到的命令:

```bash
### 查看可用的组件:
android list sdk --all
### 安装安卓依赖工具包
android update sdk -u --all --filter 1,2,3,5,11,12,22,23,24,25,26,27,28,29,45,88,89
```

### 3. 安装gradle

在 `https://gradle.org/releases/` 中复制对应版本gradle下载地址,在/data/cordova中下载并解压:

```bash
wget https://services.gradle.org/distributions/gradle-3.3-bin...(替换为你复制的那个下载地址)
unzip  gradle-3.3-linux.zip
```

编辑配置: `vim /etc/profile` 在文件最后添加:

```bash
### gradle environment
export GRADLE_HOME=/data/cordova/gradle-3.3
export PATH=$GRADLE_HOME/bin:$PATH
```

使配置立即生效:

```bash
source /etc/profile
```

### 4. 安装node.js

下载nodejs并解压:

```bash
wget http://nodejs.org/dist/v10.16.0/node-v10.16.0-linux-x64.tar.gz
tar -zxf node-v10.16.0-linux-x64.tar.gz
```

编辑配置文件 `vim /etc/profile` 在文件最后添加:

```bash
### nodejs environment
export NODE_HOME=/data/cordova/node-v10.16.0-linux-x64
export PATH=$NODE_HOME/bin:$PATH  
```

使配置立即生效: 

```bash
source /etc/profile
```

检查node版本命令: `node -v`
检查npm 版本命令: `npm -v`

最后`/etc/profile`文件添加的配置为:

```bash
### Java environment
JAVA_HOME=/data/gradle/jdk1.8.0_152
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH

### Nodejs environment
export NODE_HOME=/data/cordova/node-v8.9.4-linux-x64
export PATH=$NODE_HOME/bin:$PATH

### Android environment
export ANDROID_HOME=/data/cordova/android-sdk-linux
export PATH="$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/tools/bin"

### gradle environment
export GRADLE_HOME=/data/cordova/gradle-3.3
export PATH=$PATH:$GRADLE_HOME/bin
```

### 5. 安装Cordova

执行命令

```bash
npm install -g cordova --registry https://registry.npm.taobao.org
```

创建Cordova项目
-------------

```bash
## 可以指定应用ID和应用名: cordova create project_name app_id app_name
cordova create hello

## 进入项目路径
cd hello

## 添加Android平台
cordova platform add android --save
```

修改安卓平台构建文件的maven地址为阿里云镜像: `vim  platforms/android/build.gradle` (可以跳过, 如果构建失败再来配置)

```gradle
// 将buildscript和allprojects中repositories 的内容都替换成:
repositories {
    jcenter()
    google()
    maven {
        url "http://maven.aliyun.com/nexus/content/groups/public/"
    }
}
```

把写好的H5文件放入www文件夹下

自定义app logo和启动画面需要添加插件 ([参考文章](https://blog.csdn.net/lc_style/article/details/78401105)):

```bash
cordova plugin add cordova-plugin-splashscreen
```

打包安装App

```bash
cordova build android
```

若打包过程中下载依赖时timeout, 则需要番蔷

打包成功会出现 `BUILD SUCCESSFUL` , 打包后的文件为: `platforms/android/app/build/outputs/apk/debug/app-debug.apk`

Cordova打包release版本

```bash
## 打包未签名的apk包
cordova build android --release
```

```bash
## 生成秘钥
keytool -genkey -v -keystore ~/myKey.keystore -alias myKey -keyalg RSA -validity 20000 
```
> keytool 秘钥工具
> 
> -keystore D:/myKey.keystore 表示生成的证书及其存放路径，如果直接写文件名则默认生成在用户当前目录下；
> 
> -alias myKey 表示证书的别名是 `myKey`,不写这一项的话证书名字默认是mykey；
> 
> -keyalg RSA 表示采用的RSA算法；
> 
> -validity 20000表示证书的有效期是20000天。


```bash
## 签名
jarsigner -verbose -keystore ~/myKey.keystore -signedjar name.apk app-release-unsigned.apk ~/myKey.keystore
```
> jarsigner 签名工具
> 
> name.apk 需要生成的apk名字
> 
> app-release-unsigned.apk 待签名的apk
