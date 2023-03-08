---
layout: mypost
title: Linux安装配置oh-my-zsh
categories: [Linux,zsh]
---
#### 一、源码安装

###### 1、安装zsh
```shell
yum install zsh
```

###### 2、clone oh-my-zsh项目，也可下载源码后直接上传
```shell
国外镜像 git clone https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
国内镜像 git clone https://gitee.com/mirrors/oh-my-zsh.git ~/.oh-my-zsh
```

###### 3、复制 .zshrc
```shell
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

###### 4、更改你的默认 Shell
```shell
chsh -s /bin/zsh
```

#### 二、命令安装
##### curl 安装
###### GitHub:
```shell
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
###### Gitee ( 国内镜像 )
```shell
sh -c "$(curl -fsSL https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh)"
```

##### wget 安装
###### GitHub:
```shell
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

###### Gitee ( 国内镜像 )
```shell
sh -c "$(wget -O- https://gitee.com/pocmon/mirrors/raw/master/tools/install.sh)"
```

#### 三、卸载
```shell
uninstall_oh_my_zsh
```
