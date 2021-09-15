---
layout: mypost
title: Linux安装配置Oracle
categories: [Linux,Oracle]
---
### 1、oracle数据库下载地址
```
Oracle Database 11g Release 2 (11.2.0.1.0) for Linux x86-64

http://download.oracle.com/otn/linux/oracle11g/R2/linux.x64_11gR2_database_1of2.zip

http://download.oracle.com/otn/linux/oracle11g/R2/linux.x64_11gR2_database_2of2.zip
```

### 2、解压
```
unzip linux.x64_11gR2_database_1of2.zip
unzip linux.x64_11gR2_database_2of2.zip
```

### 3、关闭selinux
```
vim /etc/selinux/config
```
查看状态
```
setenforce 0
```
![img_1.png](img_1.png)

### 4、关闭防火墙
```
systemctl restart firewalld.service
systemctl list-unit-files|grep firewalld.service
systemctl disable firewalld.service
```

### 5、安装Oracle 11g依赖包
```
yum install gcc make binutils gcc-c++ compat-libstdc++-33elfutils-libelf-devel elfutils-libelf-devel-static ksh libaio libaio-develnumactl-devel sysstat unixODBC unixODBC-devel pcre-devel -y
```

### 6、添加安装用户和用户组
```
groupadd oinstall
groupadd dba
useradd -g oinstall -G dba oracle
passwd oracle
```

### 7、修改内核参数配置文件
```
vi /etc/sysctl.conf
```
添加以下内容
```
# oracle setting
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 1073741824
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
```

### 8、修改用户的限制文件
修改/etc/security/limits.conf文件：
```
vi /etc/security/limits.conf
```
添加以下内容
```
oracle           soft    nproc           2047
oracle           hard    nproc           16384
oracle           soft    nofile          1024
oracle           hard    nofile          65536
oracle           soft    stack           10240
```

修改/etc/pam.d/login文件：
```
vi /etc/pam.d/login
```
添加以下内容
```
# oracle setting
session    required     /lib64/security/pam_limits.so
session    required     pam_limits.so
```

修改/etc/profile文件
```
vi /etc/profile
```
添加以下内容
```
# oracle setting
if [ $USER = "oracle" ]; then
    if [ $SHELL = "/bin/ksh" ]; then
        ulimit -p 16384
        ulimit -n 65536
    else
        ulimit -u 16384 -n 65536
    fi
fi
```

### 9、创建安装目录和设置文件权限
```
mkdir -p /data/oracle/product/11.2.0
mkdir /data/oracle/oradata
mkdir /data/oracle/inventory
mkdir /data/oracle/fast_recovery_area
chown -R oracle:oinstall /data/oracle
chmod -R 775 /data/oracle
```

### 10、设置oracle用户环境变量
```
su -l oracle
vi .bash_profile
```
添加以下内容
```
ORACLE_BASE=/data/oracle
ORACLE_HOME=$ORACLE_BASE/product/11.2.0
ORACLE_SID=orcl
PATH=$PATH:$ORACLE_HOME/bin
export ORACLE_BASE ORACLE_HOME ORACLE_SID PATH
```

生效环境变量
```
source .bash_profile
```

### 11、编辑静默安装响应文件
复制安装文件夹response到当前oracle用户的家目录下：
```
cp -R /app/database/response/ .
cd response/
vi db_install.rsp
```

需要设置的选项如下：
```
oracle.install.option=INSTALL_DB_SWONLY

ORACLE_HOSTNAME=CentOS

UNIX_GROUP_NAME=oinstall

INVENTORY_LOCATION=/data/oracle/inventory

SELECTED_LANGUAGES=en,zh_CN

ORACLE_HOME=/data/oracle/product/11.2.0

ORACLE_BASE=/data/oracle

oracle.install.db.InstallEdition=EE

oracle.install.db.DBA_GROUP=dba

oracle.install.db.OPER_GROUP=dba

DECLINE_SECURITY_UPDATES=true
```

### 12、根据响应文件静默安装Oracle11g
```
cd /app/database/
./runInstaller -silent -responseFile /home/oracle/response/db_install.rsp -ignorePrereq
```

交换空间不足的问题
![img_2.png](img_2.png)

解决方法
切换到root用户
```
free
dd if=/dev/zero of=swapfile bs=1024 count=500000
mkswap swapfile
swapon swapfile
```
![img_3.png](img_3.png)

开始Oracle在后台静默安装。安装过程中，如果提示[WARNING]不必理会，此时安装程序仍在后台进行，如果出现Successfully Setup Software，则安装程序已经停止了

出现以下提示时，表示安装成功了
![img_4.png](img_4.png)

按照要求执行脚本，打开终端，退出到root身份登录，执行脚本：
```
sh /data/oracle/inventory/orainstRoot.sh
sh /data/oracle/product/11.2.0/root.sh
```

### 13、以静默方式配置监听
重新使用oracle用户登录
```
netca /silent /responseFile /home/oracle/response/netca.rsp
```

注意此处，必须使用/silent /responseFile格式，而不是-silent -responseFile，因为是静默安装

成功运行后，在/data/oracle/product/11.2.0/network/admin中生成listener.ora和sqlnet.ora

通过netstat命令可以查看1521端口正在监听

### 14、以静默方式建立新库，同时也建立一个对应的实例
```
vi /home/oracle/response/dbca.rsp
```
参考以下内容配置
```
[GENERAL]

# oracle版本，不能更改
RESPONSEFILE_VERSION = "11.2.0"

# Description   : Type of operation
OPERATION_TYPE = "createDatabase"

[CREATEDATABASE]

# Description   : Global database name of the database
# 全局数据库的名字=SID+主机域名# 第三方工具链接数据库的时候使用的service名称
GDBNAME = "orcl.test"

# Description   : System identifier (SID) of the database
# 对应的实例名字
SID = "orcl"

# Description   : Name of the template
# 建库用的模板文件
TEMPLATENAME = "General_Purpose.dbc"

# Description   : Password for SYS user
# SYS管理员密码
SYSPASSWORD = "123456"

# Description   : Password for SYSTEM user
# SYSTEM管理员密码
SYSTEMPASSWORD = "123456"

# Description   : Password for SYSMAN user
# SYSMAN管理员密码
SYSMANPASSWORD = "123456"

# Description   : Password for DBSNMP user
# DBSNMP管理员密码
DBSNMPPASSWORD = "123456"

# Description   : Location of the data file's
# 数据文件存放目录
DATAFILEDESTINATION =/data/oracle/oradata

# Description   : Location of the data file's
# 恢复数据存放目录
RECOVERYAREADESTINATION=/data/oracle/fast_recovery_area

# Description   : Character set of the database
# 字符集，重要!!! 建库后一般不能更改，所以建库前要确定清楚。
# (CHARACTERSET = "AL32UTF8" NATIONALCHARACTERSET= "UTF8")
CHARACTERSET = "ZHS16GBK"

# Description   : total memory in MB to allocate to Oracle
# oracle内存1638MB,物理内存2G*80%
TOTALMEMORY = "1638"
```

进行静默配置：
```
dbca -silent -responseFile /home/oracle/response/dbca.rsp
```
![img_5.png](img_5.png)

建库后进行实例进程检查：
```
ps -ef | grep ora_ | grep -v grep
```
![img_6.png](img_6.png)

查看监听状态：
```
lsnrctl status
```
![img_7.png](img_7.png)

### 15、命令行模式静默删除
首先查看dbca的帮助信息
```
dbca -help
```

修改/home/oracle/response/dbca.rsp文件里以下几个参数，下面三个参数根据建库实际情况进行修改：
```
OPERATION_TYPE = "deleteDatabase"
SOURCEDB = "orcl"
SYSDBAUSERNAME = "sys"
SYSDBAPASSWORD = "123456"
```

然后运行：
```
dbca -silent -responseFile /home/oracle/response/dbca.rsp
```

各参数含义如下:
```
-silent 表示以静默方式删除
-responseFile 表示使用哪个响应文件,必需使用绝对路径
RESPONSEFILE_VERSION 响应文件模板的版本,该参数不要更改
OPERATION_TYPE 安装类型,该参数不要更改
SOURCEDB 数据库名,不是全局数据库名,即不包含db_domain
```

很简单数据库卸载完成了，请注意，只是数据库卸载完了，数据库软件还是在的。

### 16、使用DBCA卸载数据库
```
dbca -silent -delete Database -responseFile dbca.rsp
a.选项-silent表示静默安装，免安装交互，大部分安装信息也不输出
b.选项-responseFile指定应答文件，要求用绝对路径
```



