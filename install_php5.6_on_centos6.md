Title: install php5.6 on centos6  
Date: 2019-01-09 11:48  
Modified: 2019-01-09 11:48  
Category: TECH SKILLS  
Tags: php, centos6, php extension  
Slug: install-php5.6-on-centos6  
Authors: Yullin

本次安装系统环境为Centos6 x86_64
```
# cd /usr/local/src
# wget http://cn2.php.net/distributions/php-5.6.39.tar.gz
# tar xzf php-5.6.39.tar.xz
```

安装依赖包
```
# yum install gcc gcc-c++ bison bison-devel zlib-devel libmcrypt-devel mcrypt mhash-devel openssl-devel libxml2-devel libcurl-devel bzip2-devel readline-devel libedit-devel sqlite-devel
```

开始编译安装（这里先别急着去执行，先往文章后面看看）
```
# ./configure \
--prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--enable-inline-optimization \
--disable-debug \
--disable-rpath \
--enable-shared \
--enable-opcache \
--enable-fpm \
--with-fpm-user=www \
--with-fpm-group=www \
--with-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-gettext \
--enable-mbstring \
--with-iconv=/usr/local/libiconv \
--with-mcrypt \
--with-mhash \
--with-openssl \
--enable-bcmath \
--enable-soap \
--with-libxml-dir \
--enable-pcntl \
--enable-shmop \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-sockets \
--with-curl \
--with-zlib \
--enable-zip \
--with-bz2 \
--with-readline \

```

编译时报错：
```
configure: error: Please reinstall the iconv library.
```
安装iconv lib
```
# cd /usr/local/src/
# wget https://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.15.tar.gz
# tar xzf libiconv-1.15.tar.gz
# cd libiconv-1.15
# ./configure
# make && make install
```
继续编译，又报错：
```
configure: error: Don’t know how to define struct flock on this system, set –enable-opcache=no
```
在我加上`–-enable-opcache=no`之后，又报错：
```
configure: error: off_t undefined; check your library configuration
```
到这里，我的内心OS “WTF”  
找GOOGLE老师咨询了一下，原来是要这样处理一下：
```
# echo '/usr/local/lib64
/usr/local/lib
/usr/lib
/usr/lib64'>>/etc/ld.so.conf 
# ldconfig -v
```
据说这样处理之后，即便不改之前的opcache设置也正常，不过我没试，你有空倒是可以`--enable-opcache`再测试一次。  
修改之后是这样的：
```
# cat /etc/ld.so.conf
include ld.so.conf.d/*.conf
/usr/local/lib
/usr/lib
/usr/lib64
```
然后就可以一路畅快的完成configure了，你如果仔细去关注那飞快滚动的屏幕的话，会发现不少Warning，啊注意不是“FBI WARNING”，不过我都是略过了。
再接下来就是
```
make && make install
```

然后就是，嗯。。。收工。