---
layout: post
title:  "交叉编译小米路由器2（R2D）shadowsocks"
date:   2018-01-02 17:33:01 +0800
categories: shadowsocks R2D 交叉编译 小米路由器
---

# 交叉编译小米路由器2（R2D）shadowsocks
## 背景：

> 我是使用 https://shadowsocks.la/ 提供的shadowsocks服务，由于最近服务商把shadowsocks加密方式换成了google的[chacha20-ietf-poly1305]，但是小米路由器（R2D）内置的ss-redir版本旧不支持。问题就来了，我一直在小米路由器做的透明代理（简单来说既是连接到这个路由的所有请求都自动走shadowsocks服务）就失效了;所以我需要更新R2D的ss-redir以便支持[chacha20-ietf-poly1305]加密;google后没有找到现成的，相关的文章也少，最后只能自己交叉编译一个，同时把编译过程记录下来，方便大家使用;

## 小白党，直接使用我编译：
> 最新版本openwrt-shadowsock下载链接

## 折腾党，我们手把手重新编译一次：

>编译清单
>项目地址：https://github.com/shadowsocks/shadowsocks-libev
>编译环境：ubuntu
>小米路由器SDK：http://bigota.miwifi.com/xiaoqiang/sdk/tools/package/sdk_package.zip
>编译工具：make git autoconf libtool 
>编译核心依赖：TLS pcre libsodium libev c-ares


下面的操作都在自己的home目录进行，我是临时用docker构建了个ubuntu编译的，所以都在/root目录下
1、下载小米路由器SDK
```shell
wget http://bigota.miwifi.com/xiaoqiang/sdk/tools/package/sdk_package.zip
unzip sdk_package.zip
```

2、设置环境变量
```shell
export PATH=$PATH:/root/miwifi/sdk_package/toolchain/bin
export CC=arm-xiaomi-linux-uclibcgnueabi-gcc
export CXX=arm-xiaomi-linux-uclibcgnueabi-g++`
#核心依赖包编译后放在这里
mkdir /root/miwifi/ss_lib
export SS_LIB=/root/miwifi/ss_lib
```

3、安装基础依赖包
```shell
apt install -y make git autoconf libtool
```

4、下载编译安装核心依赖包

编译mbedtls
```shell
cd /root/miwifi/repo
wget --no-check-certificate https://tls.mbed.org/download/mbedtls-2.6.0-gpl.tgz
tar -xzvf mbedtls-2.6.0-gpl.tgz
cd mbedtls-2.6.0
make -j$(nproc) DESTDIR=$SS_LIB CC=arm-xiaomi-linux-uclibcgnueabi-gcc AR=arm-xiaomi-linux-uclibcgnueabi-ar LD=arm-xiaomi-linux-uclibcgnueabi-ld install
```

编译pcre
```shell
cd /root/miwifi/repo
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.41.tar.gz
tar -xzvf pcre-8.41.tar.gz
cd pcre-8.41
./configure --host=arm-xiaomi-linux-uclibcgnueabi --prefix=$SS_LIB --disable-shared --enable-utf8 --enable-unicode-properties
make -j$(nproc) install
```

编译libsodium
```shell
cd /root/miwifi/repo
wget --no-check-certificate https://download.libsodium.org/libsodium/releases/libsodium-1.0.13.tar.gz
tar -xzvf libsodium-1.0.13.tar.gz
cd libsodium-1.0.13
./configure --host=arm-xiaomi-linux-uclibcgnueabi --prefix=$SS_LIB --disable-ssp --disable-shared
make -j$(nproc) install
```

编译libev
```shell
cd /root/miwifi/repo
wget http://dist.schmorp.de/libev/libev-4.24.tar.gz
tar -xzvf libev-4.24.tar.gz
cd libev-4.24
./configure --host=arm-xiaomi-linux-uclibcgnueabi --prefix=$SS_LIB --disable-shared
make -j$(nproc) install
```

编译c-ares
```shell
cd /root/miwifi/repo
wget --no-check-certificate https://c-ares.haxx.se/download/c-ares-1.13.0.tar.gz
tar -xzvf c-ares-1.13.0.tar.gz
cd c-ares-1.13.0
./configure LDFLAGS=-static  --host=arm-xiaomi-linux-uclibcgnueabi --prefix=$SS_LIB
```

5、正式编译shadowsocks-libev 
```shell
mkdir /root/miwifi/ss
cd /root/miwifi/repo
git clone https://github.com/shadowsocks/shadowsocks-libev
cd shadowsocks-libev
git submodule init
git submodule update
./autogen.sh
./configure LIBS="-lpthread -lm" LDFLAGS="-Wl,-static -static -static-libgcc -L$SS_LIB/lib" CFLAGS="-I$SS_LIB/include" --host=arm-xiaomi-linux-uclibcgnueabi --prefix=/root/miwifi/ss --disable-ssp --disable-documentation --with-mbedtls=$SS_LIB --with-pcre=$SS_LIB --with-sodium=$SS_LIB
make -j$(nproc) install
```

编译完成就可以看到/root/miwifi/ss目录下有编译好的文件，copy到小米路由器上跑就好了;