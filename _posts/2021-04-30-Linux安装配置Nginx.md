---
layout: mypost
title: Linux安装配置Nginx
categories: [Linux,Nginx]
---
###### 1、将压缩包上传到指定目录并解压
```shell
tar -zxvf nginx-1.16.1.tar.gz
```

###### 2、设置配置信息
```shell
./configure --prefix=/usr/local/nginx
```

###### 3、执行./configure报错: the HTTP rewrite module requires the PCRE library
```shell
yum -y install pcre-devel
yum -y install openssl openssl-devel
```

###### 4、编译和安装
```shell
make
make install
```

###### 5、查看配置信息是否正确
```shell
/usr/local/nginx/sbin/nginx -t
```

###### 6、启动nginx
```shell
/usr/local/nginx/sbin/nginx
```

###### 7、停止nginx
```
从容停止Nginx：kill -QUIT 主进程号
快速停止Nginx：kill -TERM 主进程号
强制停止Nginx：pkill -9 nginx
```


