---
layout: post
title:  "GNUroot debian android ssh设置"
date:   22017-09-09 21:18:43 +0800
categories: gnuroot debian
---



## SSH设置
```
# 安装openssh
apt-get install openssh-server
# 打开文件/etc/ssh/sshd_conifg,设置一下属性
PermitRootLogin yes 
UsePAM no
UsePrivilegeSeparation no 
```

## 设置语言环境
```
# 查看语言库
locale -a
# 安装locales
apt-get install locales
# 生成语言文件
locale-gen zh_CN.UTF-8
dpkg-reconfigure locales

vi /etc/default/locale 
LANG=zh_CN.UTF-8

# 设置环境变量vi ~/.bash_profile
exoprt LANG="zh_CN.UTF-8"
# 生效
source .bash_profile
```

## 设置时间同步
```
# 时间（时区）查看
date -R
# 安装时间同步程序
apt-get install ntpdate
# 同步时间
ntpdate time.nist.gov
# 选择时区（只是选择，选择后的结果需要copy到环境变量）
tzselect
# 设置环境变量vi ~/.bash_profile
TZ='Asia/Shanghai'; export TZ
```

## 命令别名
```
# 设置环境变量vi ~/.bash_profile
alias l='ls -alhF'
alias la='ls -AFh'
alias ll='ls -lhAF'
```

