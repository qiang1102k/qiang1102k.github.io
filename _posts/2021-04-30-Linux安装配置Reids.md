---
layout: mypost
title: Linux安装配置Redis
categories: [Linux,Redis]
---
###### 1、将压缩包上传到指定目录并解压
```shell
tar -zxvf redis-6.2.1.tar.gz
```

###### 2、编译并安装redis
```shell
cd redis-6.2.1
make
cd src
make install
```

###### 3、安装完成后usr/local/bin中会生成相关的命令文件，进入安装目录复制redis.conf文件至usr/local/bin
```shell
mv redis.conf /usr/local/bin
```

###### 4、设置redis后台启动
```shell
daemonize yes
```