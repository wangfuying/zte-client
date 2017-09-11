# 城院校园网Linux客户端 v3.0 Beta

### 简介
本客户端为Linux中兴认证客户端，已适配东莞理工学院城市学院。

参考Dot1x

### 编译
需要安装以下运行库:
libcurl, libleptonica, tesseract


#### For Debian Jessie
```
apt-get install libcurl4-openssl-dev libleptonica-dev tesseract-ocr-dev tesseract-ocr-eng gcc git make cmake
git clone https://github.com/yzsme/zte-client.git
cd zte-client
cmake CMakeLists.txt
make
cp ./zte_client /usr/sbin/zte-client
```

#### 交叉编译
Debian Jessie可参考此处安装Toolchains: [CrossToolchains#For_jessie_.28Debian_8.29](https://wiki.debian.org/CrossToolchains#For_jessie_.28Debian_8.29) 

OpenWRT/PandoraBox可到官网下载Toolchains

以Debian 9 Stretch，目标架构mipsel为例

下载源码: 
```
apt-get install build-essential crossbuild-essential-mipsel autoconf libtool pkg-config git

cd /usr/src

MAKE_JOBS=32

CURL_VERSION="7.54.1"
LEPT_VERSION="1.74.4"
TESSERACT_VERSION="3.05.01"
LIBJPEG_VERSION="1.5.2"
wget -O- https://curl.haxx.se/download/curl-${CURL_VERSION}.tar.bz2 | tar -xjvf-
wget -O- http://www.leptonica.com/source/leptonica-${LEPT_VERSION}.tar.gz | tar -zxvf-
wget -O- https://github.com/tesseract-ocr/tesseract/archive/${TESSERACT_VERSION}.tar.gz | tar -zxvf-
wget -O- https://github.com/libjpeg-turbo/libjpeg-turbo/archive/${LIBJPEG_VERSION}.tar.gz | tar -zxvf-
```
编译curl: 
```
cd /usr/src/curl-${CURL_VERSION}
CC=/usr/bin/mipsel-linux-gnu-gcc ./configure \
--host=mipsel \
--prefix=/usr/local/curl-mipsel \
--disable-debug \
--disable-ares \
--disable-rt \
--disable-largefile \
--disable-libtool-lock \
--disable-ftp \
--disable-ldap \
--disable-ldaps \
--disable-rtsp \
--disable-proxy \
--disable-dict \
--disable-telnet \
--disable-tftp \
--disable-pop3 \
--disable-imap \
--disable-smb \
--disable-smtp \
--disable-gopher \
--disable-manual \
--disable-ipv6 \
--disable-sspi \
--disable-crypto-auth \
--disable-ntlm-wb \
--disable-tls-srp \
--disable-unix-sockets \
--disable-soname-bump \
--without-zlib \
--without-winssl \
--without-darwinssl \
--without-ssl \
--without-gnutls \
--without-polarssl \
--without-mbedtls \
--without-cyassl \
--without-nss \
--without-axtls \
--without-libpsl \
--without-libmetalink \
--without-libssh2 \
--without-librtmp \
--without-winidn \
--without-libidn2 \
--without-nghttp2
make -j${MAKE_JOBS} && make -j${MAKE_JOBS} install
```
编译libjpeg:
```
cd /usr/src/libjpeg-turbo-${LIBJPEG_VERSION}
autoreconf -fiv
CC=/usr/bin/mipsel-linux-gnu-gcc \
./configure \
--host=mipsel \
--prefix=/usr/local/libjpeg-mipsel
make -j${MAKE_JOBS} && make -j${MAKE_JOBS} install
```
编译leptonica:
```
cd /usr/src/leptonica-${LEPT_VERSION}
CC=/usr/bin/mipsel-linux-gnu-gcc \
CFLAGS="-I/usr/local/libjpeg-mipsel/include/ -L/usr/local/libjpeg-mipsel/lib/" \
./configure \
--host=mipsel \
--prefix=/usr/local/leptonica-mipsel \
--without-zlib \
--without-libpng \
--without-giflib \
--without-libtiff \
--without-libwebp \
--without-libopenjpeg
make -j${MAKE_JOBS} && make -j${MAKE_JOBS} install
```
编译tesseract:
```
cd /usr/src/tesseract-${TESSERACT_VERSION}
./autogen.sh
C=/usr/bin/mipsel-linux-gnu-gcc \
CXX=/usr/bin/mipsel-linux-gnu-g++ \
PKG_CONFIG_PATH="/usr/local/leptonica-mipsel/lib/pkgconfig/" \
./configure \
--host=mipsel \
--disable-largefile \
--disable-graphics \
--enable-embedded \
--prefix=/usr/local/tesseract-mipsel
make -j${MAKE_JOBS} && make -j${MAKE_JOBS} install
```
最后参考以下命令编译客户端，输出ELF文件zte-client
```
cd /usr/src
git clone https://github.com/yzsme/zte-client.git
cd zte-client
mkdir -p mipsel-build
cd mipsel-build
mipsel-linux-gnu-gcc -I/usr/local/tesseract-mipsel/include -I/usr/local/curl-mipsel/include/ -I/usr/local/leptonica-mipsel/include/ ../main.c ../src/zte.c ../src/dhcpClient.c ../src/exception.c ../src/webAuth.c -c
mipsel-linux-gnu-g++ main.o zte.o dhcpClient.o exception.o webAuth.o /usr/local/curl-mipsel/lib/libcurl.a /usr/local/leptonica-mipsel/lib/liblept.a /usr/local/tesseract-mipsel/lib/libtesseract.a /usr/local/libjpeg-mipsel/lib/libjpeg.a -lpthread -static-libstdc++ -static-libgcc -lrt -o zte-client
```
查看文件信息
```
bash - # file zte-client
./zte-client: ELF 32-bit LSB shared object, MIPS, MIPS32 rel2 version 1 (SYSV), dynamically linked, interpreter /lib/ld.so.1, for GNU/Linux 3.2.0, BuildID[sha1]=d1ff072bf2f621498368883f7e4c5cbdaae75449, not stripped

bash - # mipsel-linux-gnu-readelf -d ./zte-client 

Dynamic section at offset 0x23c contains 32 entries:
  Tag        Type                         Name/Value
 0x00000001 (NEEDED)                     Shared library: [libpthread.so.0]
 0x00000001 (NEEDED)                     Shared library: [librt.so.1]
 0x00000001 (NEEDED)                     Shared library: [libm.so.6]
 0x00000001 (NEEDED)                     Shared library: [libc.so.6]
 0x00000001 (NEEDED)                     Shared library: [ld.so.1]
 0x0000000c (INIT)                       0x870fc
 0x0000000d (FINI)                       0x604670
 0x00000019 (INIT_ARRAY)                 0x71c640
 0x0000001b (INIT_ARRAYSZ)               564 (bytes)
 0x00000004 (HASH)                       0x36c
 0x00000005 (STRTAB)                     0x24a2c
 0x00000006 (SYMTAB)                     0xab3c
 0x0000000a (STRSZ)                      269369 (bytes)
 0x0000000b (SYMENT)                     16 (bytes)
 0x70000035 (MIPS_RLD_MAP_REL)           0x724264
 0x00000015 (DEBUG)                      0x0
 0x00000003 (PLTGOT)                     0x724520
 0x00000011 (REL)                        0x69b64
 0x00000012 (RELSZ)                      32720 (bytes)
 0x00000013 (RELENT)                     8 (bytes)
 0x70000001 (MIPS_RLD_VERSION)           1
 0x70000005 (MIPS_FLAGS)                 NOTPOT
 0x70000006 (MIPS_BASE_ADDRESS)          0x0
 0x7000000a (MIPS_LOCAL_GOTNO)           6788
 0x70000011 (MIPS_SYMTABNO)              6639
 0x70000012 (MIPS_UNREFEXTNO)            43
 0x70000013 (MIPS_GOTSYM)                0x18cb
 0x6ffffffb (FLAGS_1)                    Flags: PIE
 0x6ffffffe (VERNEED)                    0x69a44
 0x6fffffff (VERNEEDNUM)                 4
 0x6ffffff0 (VERSYM)                     0x66666
 0x00000000 (NULL)                       0x0
```


### 使用说明
```
必要参数(使用-l, -r参数不要求以下参数):

	-u, --zteuser		中兴认证用户名
	-p, --ztepass		中兴认证密码
	-d, --device		指定网卡设备，默认为第一个可用网卡设备


可选参数:

	-w, --webuser		天翼认证用户名
	-k, --webpass		天翼认证密码
	-f, --pidfile		pid文件路径，默认为/tmp/zte-client.pid
	-m, --logfile       日志文件路径，前台运行模式下默认输出到标准输出，后台运行模式下默认重定向到/dev/null
	-i, --dhcp		指定DHCP客户端，可以选择dhclient, udhcpc或none(代表不启用DHCP客户端)，默认为dhclient
	-b, --daemon		以守护进程模式运行
	-r, --reconnect		重新连接
	-l, --logoff		注销
	-h, --help		显示帮助信息
```
### 示例
进行中兴认证与天翼认证，并以守护进程模式运行:
```
/usr/sbin/zte-client --zteuser username --ztepass password --webuser webusername --webpass webpassword --device eth0 --daemon
```

注销:
```
/usr/sbin/zte-client -l
```

重新进行认证:
```
/usr/sbin/zte-client -r
```

### 参考资料
Dot1x

802.11x协议信息

### 链接
[MIT License](https://opensource.org/licenses/MIT)

[Zhensheng Yuan's weblog: http://zhensheng.im](http://zhensheng.im)

### 协议
**The MIT License (MIT)**

Copyright (c) 2016 Zhensheng Yuan

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
