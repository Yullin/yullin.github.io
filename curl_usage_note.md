Title: curl usage note
Date: 2018-11-02 11:08
Modified: 2018-11-02 11:08
Category: TECH SKILLS
Tags: curl, usage, get, post
Slug: curl-usage-note
Authors: Yullin

CURL可以详细打印出请求过程每一步所消耗的时间，对于我们日常的排查故障非常有用。  
下面说一下具体的使用方法：  
### 1.建立一个命令格式文件  
```
\n 
　　　　　　　time_namelookup:  %{time_namelookup}\n
               time_connect:  %{time_connect}\n
            time_appconnect:  %{time_appconnect}\n
           time_pretransfer:  %{time_pretransfer}\n
              time_redirect:  %{time_redirect}\n
         time_starttransfer:  %{time_starttransfer}\n
                            ----------\n
                 time_total:  %{time_total}\n
\n
```
说明：  
time_namelookup：DNS解析域名时间，把域名--->ip的时间  
time_connect：TCP连接的时间，三次握手的时间  
time_appconnect：SSL|SSH等上层连接建立的时间  
time_pretransfer：从请求开始到到响应开始传输的时间  
time_redirect：从开始到最后一个请求事务的时间  
time_starttransfer：从请求开始到第一个字节将要传输的时间  
time_total：总时间

### 2.命令使用方法

  curl -w "@curl" -o /dev/null -s -d "username=aaa&password=bbb" https://xxx.xxx.com/webapp/xxx/login

 -w：从文件中读取信息打印格式

 -o：输出的全部信息

 -s：不打印进度条

 -d：参数

### 3.简单的方式

curl -o /dev/null -s -w '%{time_connect}:%{time_starttransfer}:%{time_total}\n' 'http://www.baidu.com'

忽略ssl认证  并输出结果

curl -k -XPOST -w '\n%{time_connect}:%{time_starttransfer}:%{time_total}\n' www.baidu.com

### 4.当Content-Type:application/json，requestBody:{id: 93, status: 2}

 curl -w "@curl" -H "Content-Type:application/json" -X PUT --data '{id: 93, status: 2}' http:/xxxxxxxxxxx/changeStatus
 
 
 
 -----------------------------------------------------------------------
 
 curl 命令提供了 -w 参数，这个参数在 manpage 是这样解释的：
```
-w, --write-out <format>
              Make curl display information on stdout after a completed transfer. The format is a string that may contain plain text mixed with any number of variables. The  format
              can  be  specified  as  a literal "string", or you can have curl read the format from a file with "@filename" and to tell curl to read the format from stdin you write
              "@-".

              The variables present in the output format will be substituted by the value or text that curl thinks fit, as described below. All variables are specified  as  %{vari‐
              able_name} and to output a normal % you just write them as %%. You can output a newline by using \n, a carriage return with \r and a tab space with \t.
```
它能够按照指定的格式打印某些信息，里面可以使用某些特定的变量，而且支持 \n 、 \t 和 \r 转义字符。提供的变量很多，比如 status_code 、 local_port 、 size_download 等等，这篇文章我们只关注和请求时间有关的变量（以 time_ 开头的变量）。

先往文本文件 curl-format.txt 写入下面的内容：
```
➜  ~ cat curl-format.txt
    time_namelookup:  %{time_namelookup}\n
       time_connect:  %{time_connect}\n
    time_appconnect:  %{time_appconnect}\n
      time_redirect:  %{time_redirect}\n
   time_pretransfer:  %{time_pretransfer}\n
 time_starttransfer:  %{time_starttransfer}\n
                    ----------\n
         time_total:  %{time_total}\n
```
那么这些变量都是什么意思呢？我解释一下：

time_namelookup ：DNS 域名解析的时候，就是把 https://zhihu.com 转换成 ip 地址的过程
time_connect ：TCP 连接建立的时间，就是三次握手的时间
time_appconnect ：SSL/SSH 等上层协议建立连接的时间，比如 connect/handshake 的时间
time_redirect ：从开始到最后一个请求事务的时间
time_pretransfer ：从请求开始到响应开始传输的时间
time_starttransfer ：从请求开始到第一个字节将要传输的时间
time_total ：这次请求花费的全部时间
我们先看看一个简单的请求，没有重定向，也没有 SSL 协议的时间：
```
➜  ~ curl -w "@curl-format.txt" -o /dev/null -s -L "http://cizixs.com"
    time_namelookup:  0.012
       time_connect:  0.227
    time_appconnect:  0.000
      time_redirect:  0.000
   time_pretransfer:  0.227
 time_starttransfer:  0.443
                    ----------
         time_total:  0.867
```
可以看到这次请求各个步骤的时间都打印出来了，每个数字的单位都是秒（seconds），这样可以分析哪一步比较耗时，方便定位问题。这个命令各个参数的意义：

-w ：从文件中读取要打印信息的格式
-o /dev/null ：把响应的内容丢弃，因为我们这里并不关心它，只关心请求的耗时情况
-s ：不要打印进度条
从这个输出，我们可以算出各个步骤的时间：

DNS 查询：12ms
TCP 连接时间：pretransfter(227) - namelookup(12) = 215ms
服务器处理时间：starttransfter(443) - pretransfer(227) = 216ms
内容传输时间：total(867) - starttransfer(443) = 424ms
来个比较复杂的，访问某度首页，带有中间有重定向和 SSL 协议：
```
➜  ~ curl -w "@curl-format.txt" -o /dev/null -s -L "https://baidu.com"
    time_namelookup:  0.012
       time_connect:  0.018
    time_appconnect:  0.328
      time_redirect:  0.356
   time_pretransfer:  0.018
 time_starttransfer:  0.027
                    ----------
         time_total:  0.384
```
可以看到 time_appconnect 和 time_redirect 都不是 0 了，其中 SSL 协议处理时间为 328-18=310ms 。而且 pretransfer 和 starttransfer 的时间都缩短了，这是重定向之后请求的时间。