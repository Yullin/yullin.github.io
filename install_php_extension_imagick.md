Title: install php extension imagick
Date: 2018-08-09 16:06
Modified: 2018-08-09 16:06
Category: TECH SKILLS
Tags: php, imagick, php extension
Slug: install-php-extension-imagick
Authors: Yullin

linux下配置imagick的步骤为(以centOS为例):

### 1. 安装ImageMagick

```
yum install ImageMagick-devel
/usr/local/imagemagick/bin/convert -sample 25%x25% a.jpg b.jpg #测试语句
```

### 2.  安装php的imagick扩展模块 (http://pecl.php.net/package/imagick)

```
wget http://pecl.php.net/get/imagick-3.1.0RC2.tgz
tar -zxvf imagick-3.1.0RC2.tgz
/usr/local/php/bin/phpize 					#在项目目录下运行phpize, phpize为项目生成合乎php使用的configure文件
./configure --with-php-config=/usr/local/php/bin/php-config --with-imagick=/usr/local/imagemagick	#php-config:获取php配置信息
make
make install
```

### 4. 配置php.ini

在php.ini中加入下面这句话

>extension=imagick.so
