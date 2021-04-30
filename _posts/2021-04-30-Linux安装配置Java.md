---
layout: mypost
title: Linux安装配置Java
categories: [Linux,Java]
---
###### 1、下载jdk安装包并上传到指定目录
###### 2、解压压缩包

```
tar -zxvf jdk-8u281-linux-i586.tar.gz
```
###### 3、配置环境变量

```
vi /etc/profile

export JAVA_HOME=/soft/jdk1.8.0_281
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JRE_HOME=$JAVA_HOME/jre

source /etc/profile
```
###### 4、问题解决
###### bash: ./java: /lib/ld-linux.so.2: bad ELF interpreter报错

```
sudo yum install glibc.i686
```
