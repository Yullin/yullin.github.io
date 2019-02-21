Title: configure elasticsearch logstash kibana 
Date: 2018-10-26 11:20
Modified: 2018-10-26 11:20
Category: TECH SKILLS
Tags: elk, elasticsearch, logstash, kibana, ingest node
Slug: config-elasticsearch-logstash-kibana
Authors: Yullin

本文主要记录ELK的测试环境的搭建过程   
### 1.环境以及架构 
Jumper|系统版本|IP地址|主机名|角色
------|------|:------:|-----|---
d1000|CENTOS 7|10.2.8.30|centosT-AutomELK-8030|Kibana
d1000|CENTOS 7|10.2.8.31|centosT-AutomELK-8031|Logstash
d1000|CENTOS 7|10.2.8.32|centosT-AutomELK-8032|ElasticSearch

本次的所有环境均是基于CENTOS7系统进行搭建。  
首先是准备环境，安装JAVA运行环境JDK，需要先上传下载好的JDK,这里我下载的是rpm安装包。
由于安全原因，不能直接上传，只能先上传到d1000.intsig.net的FTP服务器上，然后通过wget命令进行下载。  
```
wget http://10.2.4.11/bin_deng/packagename.rpm  
rpm -ivh packagename.rpm
```
### 2.安装配置Elasticsearch
首先需要下载最新版本：  
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.3.1.tar.gz  
```
下载后解压至`/usr/local/`目录下，解压后的路径即为`/usr/local/elasticsearch-6.3.1/`，做一个软链接
```
ln -s /usr/local/elasticsearch-6.3.1 /usr/local/elasticsearch
```
修改配置文件`/usr/local/elasticsearch/config/elasticsearch.yml`，需要修改以下几项：
```  
network.host: 10.2.8.32  #设置为本机外网可访问的IP
http.port: 9200          #设置监听端口为9200  
cluster.name: my-es      #设置集群名字
node.name: node-1        #设置节点名字
path.data: /data/es_data #设置数据文件存储目录
path.logs: /data/es_logs #设置日志文件目录
```
修改完成后，启动elasticsearch  
```
/usr/local/elasticsearch/bin/elasticsearch  
```
启动直接就报错了，报错信息如下：  
```
Exception in thread "main" java.nio.file.AccessDeniedException: /usr/local/elasticsearch/config/jvm.options
        at java.base/sun.nio.fs.UnixException.translateToIOException(UnixException.java:90)
        at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:111)
        at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:116)
        at java.base/sun.nio.fs.UnixFileSystemProvider.newByteChannel(UnixFileSystemProvider.java:215)
        at java.base/java.nio.file.Files.newByteChannel(Files.java:369)
        at java.base/java.nio.file.Files.newByteChannel(Files.java:415)
        at java.base/java.nio.file.spi.FileSystemProvider.newInputStream(FileSystemProvider.java:384)
        at java.base/java.nio.file.Files.newInputStream(Files.java:154)
        at org.elasticsearch.tools.launchers.JvmOptionsParser.main(JvmOptionsParser.java:58)
```
经过查询是由于执行该命令的用户没有运行jvm的权限导致该报错，所以我们新建一个elk用户:
```
sudo useradd elk  
```
然后修改目录归属：
```
sudo chown -R elk.elk /usr/local/elasticsearch* 
sudo chown -R elk.elk /data/es_*               #修改/data/es_data和/data/es_log目录的权限 
sudo su
```
在root用户下运行elasticsearch也会报错`can not run elasticsearch as root`，所以我们需要切换到elk用户下面再启动elasticsearch
```
su elk
/usr/local/elasticsearch/bin/elasticsearch -d  #-d参数是后台运行
```
修改之后仍然还是有报错，报错如下：
```
ERROR: [1] bootstrap checks failed
[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```
需要在/etc/sysctl.conf中修改配置
```
vm.max_map_count=262144
```
或者在命令行中执行`sysctl -w vm.max_map_count=262144`  
如此折腾一番之后，可以正常运行elasticsearch了。

### 3.安装配置Logstash  
* *需要注意的一点是新版的logstash-6.3.2不支持最新版的jdk-10.0.2，需要降为上一个稳定版jdk-8u181。否则会报错`Unrecognized VM option 'UseParNewGC'`参考链接https://github.com/elastic/logstash/issues/9345*
    
下载最新版本的Logstash:
```
sudo wget https://artifacts.elastic.co/downloads/logstash/logstash-6.3.2.tar.gz
sudo tar xzf logstash-6.3.2.tar.gz -C /usr/local/
sudo ln -s logstash-6.3.2 logstash
```
在`/esr/local/logstash/config`目录下面新建log.conf配置文件，并添加以下修改内容：
```
input {
  stdin {
    type => "syslog"
  }
  #file {
  #  type => "hadoop-yarnlog"
  #  path => "/usr/local/hadoop/logs/yarn-hadoop-resourcemanager-m000.log"
  #}
}

output {
  elasticsearch {
    hosts => "10.2.8.32:9200"
    index => "logstash-%{type}-%{+YYYY.MM.dd}"
    template_overwrite => true
  }
}
```
关于logstash配置文件的解释可以参考 [csdn blog][a] 的说明 

[a]: https://blog.csdn.net/king7950/article/details/61196447 ""

关于logstash的启动命令如下：
```
sudo /usr/local/logstash/bin/logstash -f /usr/local/logstash/config/log.conf > /dev/null 2>&1
```
这样的方式启动主要是为了防止其产生不必要的输出，占用磁盘存储。

### 4.安装配置Kibana
到官网进行下载最新版本：
```
cd /usr/local/src/
sudo wget https://artifacts.elastic.co/downloads/kibana/kibana-6.3.2-linux-x86_64.tar.gz
```
然后解压缩到`/usr/local/`目录下面
```
sudo tar xzf kibana-6.3.2-linux-x86_64.tar.gz -C /usr/local/
cd /usr/local/
sudo ln -s kibana-6.3.2-linux-x86_64 kibana
```
修改配置文件`/usr/local/kibana/config/kibana.yml`中的以下参数：
```
server.port: 5601                           #kibana的监听端口
server.host: "10.2.8.30"                    #kibana的绑定IP
elasticsearch.url: "http://10.2.8.32:9200"  #kibana的后端数据查询链接
```
修改完成后启动kibana到后台
```
sudo /usr/local/kibana/bin/kibana > /dev/null 2>&1 &
```
