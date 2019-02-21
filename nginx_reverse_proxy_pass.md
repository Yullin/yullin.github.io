Title: Nginx反向代理配置
Date: 2018-12-19 16:05  
Modified: 2018-12-19 16:05  
Category: TECH SKILLS  
Tags: nginx, reverse, proxy, 反向代理   
Slug: nginx-reverse-proxy-pass  
Authors: Yullin


这里主要记录Nginx服务器的反向代理proxy_pass配置方法中容易踩坑的地方,就是经常被提到的url的/问题的相关说明,需要的朋友可以参考下

## 普通反向代理
Nginx的普通的反向代理配置还是比较简单的，如：
```
location ~ /*
{
    proxy_pass http://192.168.1.12:8080;
}
```
或者可以
```
location /
{
    proxy_pass http://192.168.1.12:8080;
}
```
如果要配置一个相对复杂的反向代理,比如，将url中以/test/开头的请求转发到后台对应的某台server上  
可以在Nginx里设置一个变量，来临时保存/test/后面的路径信息
```
location ^~ /test/
{
    if ($request_uri ~ /test/(\d+)/(.+))
    {
        set $id $1;
        set $params $2;
    }
    proxy_pass http://backend$id.domain.com:5543/$params;
}
```
也可以先rewrite一下，然后再代理：
```
location ^~ /test/{
    rewrite /test/(\d+)/(.+) /$2?$args break;
    proxy_pass http://backend$1.domain.com:5543;
}
```
或者
```
location ~* /test/(\d+)/(.+)
{
    proxy_pass http://backend$1.domain.com:5543/$2?$args;
}
```
注意上面最后的`?$args`，表明把原始url最后的get参数也给代理到后台  
如果在proxy_pass中使用了变量（不管是主机名变量$1或后面的$2变量），则必须得加这段代码  
但如果pass_proxy后没用任何变量，则不需要加，它默认会把所有的url都给代理到后台，如：
```
location ~* /test/(\d+)/(.+)
{
    proxy_pass http://backend.domain.com:5543;
}
```
## url的/问题  
在nginx中配置proxy_pass时，当在后面的url加上了/，相当于是绝对根路径，nginx不会把location中匹配的路径部分代理走;如果没有/，则会把匹配的路径部分也给代理走。
 
下面四种情况分别用http://192.168.1.12/proxy/test.html 进行访问。  
* 第一种：
```
location /proxy/ {
    proxy_pass http://192.168.1.12:81/;
}
```
会被代理到http://192.168.1.12:81/test.html 这个url
 
* 第二种(相对于第一种，最后少一个 /)
```
location /proxy/ {
    proxy_pass http://192.168.1.12:81;
}
```
会被代理到http://192.168.1.12:81/proxy/test.html 这个url
 
* 第三种：
```
location /proxy/ {
    proxy_pass http://192.168.1.12:81/app/;
}
```
会被代理到http://192.168.1.12:81/app/test.html 这个url。

* 第四种情况(相对于第三种，最后少一个 / )：
```
location /proxy/ {
    proxy_pass http://192.168.1.12:81/app;
}
```
会被代理到http://192.168.1.12:81/apptest.html 这个url
 
从上面结果可以看出，准确说分两种情况：
1. http://192.168.1.12:81 (上面的第二种) 这种
2. http://127.0.0.1:81/.... （上面的第1，3，4种） 这种