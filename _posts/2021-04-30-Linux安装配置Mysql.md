---
layout: mypost
title: Linux安装配置Mysql
categories: [Linux,Mysql]
---
###### 1、创建mysql用户组和mysql用户
```
groupadd mysql
useradd -g mysql mysql
passwd mysql
```
###### 2、将安装包上传到指定目录并解压
```
tar -zxvf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
```
###### 3、移动到/usr/local/mysql目录下
```
mkdir -p /usr/local/mysql 
cd mysql-5.7.34-linux-glibc2.12-x86_64
mv * /usr/local/mysql/
```
###### 4、创建mysql数据库存放目录并赋予权限

```
# 创建目录
mkdir -p  /data/mysql  

# 赋予权限
chown mysql:mysql -R /data/mysql   
```
###### 5、配置my.cnf
```
vim /etc/my.cnf

#配置内容如下
[mysqld]
bind-address=0.0.0.0
port=3306
user=mysql
basedir=/usr/local/mysql
datadir=/data/mysql
socket=/tmp/mysql.sock
log-error=/data/mysql/mysql.err
pid-file=/data/mysql/mysql.pid
#character config
character_set_server=utf8mb4
symbolic-links=0
explicit_defaults_for_timestamp=true
```
###### 6、初始化

```
cd /usr/local/mysql/bin/
./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/data/mysql/ --user=mysql --initialize
```
###### 7、先将mysql.server放置到/etc/init.d/mysql中
```
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
```
###### 8、启动数据库
```
service mysql start
```
###### 9、查看密码并修改密码
```
# 查看密码
cat /data/mysql/mysql.err

# 登录
cd /usr/local/mysql/bin
./mysql -u root -p 

# 修改密码
SET PASSWORD = PASSWORD('*****');
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
FLUSH PRIVILEGES; 

# 修改为远程访问
use mysql                 
update user set host = '%' where user = 'root';      
FLUSH PRIVILEGES;
```



