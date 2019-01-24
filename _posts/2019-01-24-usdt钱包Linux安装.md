---
layout:     post
title:      记一次USDT安装流程
subtitle:   附精简流程
date:       2019-01-24
author:     CDz
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Golang
---


# 躺坑安装
usdt git地址:https://github.com/OmniLayer/omnicore

依赖:

```
$yum install libtool
$yum install gcc
$yum install gcc-g++
$yum install make
$yum install autoconfig
$yum install automake
```

先安装git:

```
yum install git
yum apt-get install pkg-config
```

usdt核心库:

```
git clone https://github.com/OmniLayer/omnicore.git
cd omnicore/
```

编译:

```
./autogen.sh 	//configuration failed, please install autoconf first 没有安装autoconf,先安装autoconf
./configure
make
```

//解压命名
tar -zxvf filename

安装autoconf和automake:
```
yum install -y  autoconf 
yum install automake
```

报错:
```
Makefile.am:8: error: Libtool library used but 'LIBTOOL' is undefined
Makefile.am:8:   The usual way to define 'LIBTOOL' is to add 'LT_INIT'
Makefile.am:8:   to 'configure.ac' and run 'aclocal' and 'autoconf' again.
Makefile.am:8:   If 'LT_INIT' is in 'configure.ac', make sure
Makefile.am:8:   its definition is in aclocal's search path.
/usr/share/automake-1.13/am/depend2.am: error: am__fastdepCXX does not appear in AM_CONDITIONAL
/usr/share/automake-1.13/am/depend2.am:   The usual way to define 'am__fastdepCXX' is to add 'AC_PROG_CXX'
/usr/share/automake-1.13/am/depend2.am:   to 'configure.ac' and run 'aclocal' and 'autoconf' again
/usr/share/automake-1.13/am/depend2.am: error: AMDEP does not appear in AM_CONDITIONAL
/usr/share/automake-1.13/am/depend2.am:   The usual way to define 'AMDEP' is to add one of the compiler tests
/usr/share/automake-1.13/am/depend2.am:     AC_PROG_CC, AC_PROG_CXX, AC_PROG_OBJC, AC_PROG_OBJCXX,
/usr/share/automake-1.13/am/depend2.am:     AM_PROG_AS, AM_PROG_GCJ, AM_PROG_UPC
/usr/share/automake-1.13/am/depend2.am:   to 'configure.ac' and run 'aclocal' and 'autoconf' again
Makefile.am: error: C++ source seen but 'CXX' is undefined
Makefile.am:   The usual way to define 'CXX' is to add 'AC_PROG_CXX'
Makefile.am:   to 'configure.ac' and run 'autoconf' again.
autoreconf: automake failed with exit status: 1
```
报错原因很清楚没有LIBTOOL这个包

解决:
```
yum install libtool
```

然后再执行:
```
./autogen.sh //成功
./configure	//报错
```

报错
```
checking for Berkeley DB C++ headers... no
configure: error: libdb_cxx headers missing, Omni Core requires this library for wallet functionality (--disable-wallet to disable wallet functionality)
```
报错没有Berkeley DB这个数据库

**安装Berkeley DB:**
Ubuntu安装:
```
sudo add-apt-repository ppa:bitcoin/bitcoin
sudo apt-get update
sudo apt-get install libdb4.8-dev libdb4.8++-dev
```

CentOS下载成功但是编译失败
```
Since there is no libdb++ in official CentOs repo, I've done the following to fix this problem

Download sources of BerkeleyDb 4.8 //安装BerkeleyDb 地址 https://github.com/tinybike/get-bdb-4.8 只安装不按其中步骤按下面步骤
Go to /build_unix/
../dist/configure --prefix=/usr/local --enable-cxx
make
(as root) make `install`
```

CentOS下这个方法有效:
```
last,the resolution is:

wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libdb4-4.8.30-13.el7.x86_64.rpm
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libdb4-devel-4.8.30-13.el7.x86_64.rpm
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libdb4-cxx-4.8.30-13.el7.x86_64.rpm
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libdb4-cxx-devel-4.8.30-13.el7.x86_64.rpm
and

rpm -ivh libdb4-4.8.30-13.el7.x86_64.rpm
rpm -ivh libdb4-devel-4.8.30-13.el7.x86_64.rpm
rpm -ivh libdb4-cxx-4.8.30-13.el7.x86_64.rpm 
rpm -ivh libdb4-cxx-devel-4.8.30-13.el7.x86_64.rpm﻿​
```

-------

安装完成后继续检查:
```
./configure	//报错
```
继续报错:
```
checking for boostlib >= 1.47.0... configure: We could not detect the boost libraries (version 1.47 or higher). If you have a staged boost library (still not installed) please specify $BOOST_ROOT in your environment and do not give a PATH to --with-boost option.  If you are sure you have boost installed, then check your version number looking in <boost/version.hpp>. See http://randspringer.de/boost for more documentation.
checking whether the Boost::System library is available... no
checking whether the Boost::Filesystem library is available... no
checking whether the Boost::Program_Options library is available... no
checking whether the Boost::Thread library is available... no
checking whether the Boost::Chrono library is available... no
checking whether the Boost::Unit_Test_Framework library is available... no
checking for dynamic linked boost test... no
checking for mismatched boost c++11 scoped enums... ok
configure: error: No working boost sleep implementation found.
```

```
yum install boost-devel (old boost-devel-1.41.0-18.el6.x86_64)
```

报错:
```
configure: error: openssl  not found.
```

```
yum install openssl-devel
```

报错:
```
configure: error: libevent not found.
```

```
yum install libevent-devel
```

执行:
```
./configure
make
```

编译完成后:

```
cd /src
./omnicored
```

启动失败 但是会生成`~/.bitcoin/`目录

创建`bitcoin.conf`文件

```
tnet=1
server=1
rpcuser=
rpcpassword=
rpcallowip=116.25.41.18
rpcport=7778
txindex=1
datacarriersize=80
logtimestamps=1
omnidebug=tally
omnidebug=packets
omnidebug=pending
```

```
nohup ./omnicored --datadir=/data/btcdata --conf=/root/.bitcoin/bitcoin.conf -testnet&
```

# 总结(精简流程)
1.usdt核心库:
```
git clone https://github.com/OmniLayer/omnicore.git
cd omnicore/
```

2.先安装依赖的包
```
$yum install libtool
$yum install gcc
$yum install gcc-g++
$yum install make
$yum install autoconfig
$yum install automake
$yum install libtool
yum install boost-devel (old boost-devel-1.41.0-18.el6.x86_64)
yum install openssl-devel
yum install libevent-devel
```

3.安装数据库libdb4:
```
last,the resolution is:

wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libdb4-4.8.30-13.el7.x86_64.rpm
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libdb4-devel-4.8.30-13.el7.x86_64.rpm
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libdb4-cxx-4.8.30-13.el7.x86_64.rpm
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libdb4-cxx-devel-4.8.30-13.el7.x86_64.rpm
and

rpm -ivh libdb4-4.8.30-13.el7.x86_64.rpm
rpm -ivh libdb4-devel-4.8.30-13.el7.x86_64.rpm
rpm -ivh libdb4-cxx-4.8.30-13.el7.x86_64.rpm 
rpm -ivh libdb4-cxx-devel-4.8.30-13.el7.x86_64.rpm﻿​
```

4.编译:
```
./autogen.sh
./configure
make
```

5.编译完成后:
```
cd /src
./omnicored
```
会报错,但是会生成`~/.bitcoin/`目录

6.创建`bitcoin.conf`文件
```
tnet=1
server=1
rpcuser=
rpcpassword=
rpcallowip=116.25.41.18
rpcport=7778
txindex=1
datacarriersize=80
logtimestamps=1
omnidebug=tally
omnidebug=packets
omnidebug=pending
```

7.启动
```
nohup ./omnicored --datadir=/data/btcdata --conf=/root/.bitcoin/bitcoin.conf -testnet&
```