Title: install and config openresty
Date: 2018-08-30 10:38
Modified: 2018-08-30 10:38
Category: TECH SKILLS
Tags: openresty
Slug: install-and-config-openresty
Authors: Yullin

### 安装配置openresty

1. 下载源码  
```
cd /usr/local/src
wget https://openresty.org/download/openresty-1.13.6.2.tar.gz
```

2. 解压后编译安装  
```
yum install pcre-devel openssl-devel gcc curl  #需要事先准备好需要的依赖包
tar xzf openresty-1.13.6.2.tar.gz
cd openresty-1.13.6.2
 ./configure --prefix=/usr/local/openresty --with-luajit --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-debug
gmake -j2   ##由于是双核CPU，所以编译时添加-j2的参数
gmake install
```
