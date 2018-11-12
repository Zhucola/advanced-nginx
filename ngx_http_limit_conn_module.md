
联系作者
- weixin: 363260961
- QQ: 363260961

**编写不易，转载请注明出处https://github.com/Zhucola/advanced-nginx**

使用版本
- CentOS Linux release 7.5.1804 (Core)
- nginx/1.12.2版本
- curl 7.61.0 (x86_64-pc-linux-gnu) libcurl/7.61.0 OpenSSL/1.0.2k zlib/1.2.7

对于一些服务器流量异常、负载过大，甚至是大流量的恶意攻击访问等，进行并发数的限制；该模块可以根据定义的键来限制每个键值的连接数，只有那些正在被处理的请求（这些请求的头信息已被完全读入）所在的连接才会被计数。
## 目录
* [limit_conn_zone](#limit_conn_zone)
* [limit_conn_status](#limit_conn_status)
* [并发限制于error_page的结合使用](#并发限制于error_page的结合使用)

## limit_conn_zone 
```
  Syntax:	limit_conn_zone key zone=name:size;
  Default:	—
  Context:	http
```
设置一个共享区间，用于存储各种各样的key的连接状态

该命令和limit_zone命令一起配合使用

size的大小最小为32k（32K），否则会报错
```
nginx: [emerg] zone "zone=test:30k" is too small in /usr/local/nginx/conf/nginx.conf:8
```

参数key可以是文本、变量和他们的组合
```
  http{
    limit_conn_zone $remote_addr zone=test:10m;
    limit_conn_status 503;
    server {
      listen 80;
      root /tmp;
      location / {
          fastcgi_pass http://127.0.0.1:9000;
          include fastcgi.conf;
          limit_conn test 5;
      }
    }
  }
```
以上配置的意思就是定一个名字是test，大小为10m的共享区间，该共享区间用以记录将请求的头信息已被完全读入的请求，记录的键为$remote_addr

比如并发情况下：
1.请求$remote_addr为127.0.0.1，则test区间127.0.0.1的键自增请求数为1
2.请求$remote_addr为127.0.0.1，则test区间127.0.0.1的键自增请求数为2
3.请求$remote_addr为127.0.0.1，则test区间127.0.0.1的键自增请求数为3
4.请求$remote_addr为192.168.1.1，则test区间192.168.1.1的键自增请求数为1

当$remote_addr并发请求数达到命令limit_conn test 5;所指定的大小5时,第5个并发返回503 Service Temporarily Unavailable

可以创建5个xshell窗口，在/tmp目录下创建文件a.php，a.php中sleep(50);，也就是阻塞50秒

5个窗口的$remote_addr为127.0.0.1，模拟5个并发操作，在第6个并发创建的时候，会返回503

```
  http{
    limit_conn_zone $uri zone=test:10m;
    limit_conn_status 503;
    server {
      listen 80;
      root /tmp;
      location / {
          fastcgi_pass http://127.0.0.1:9000;
          include fastcgi.conf;
          limit_conn test 5;
      }
    }
  }
```
也可以同时限制$uri的并发数量为5
比如：
1.请求$uri为/a，则test区间/a的键自增请求数为1
2.请求$uri为/a，则test区间/a的键自增请求数为2
3.请求$uri为/a，则test区间/a的键自增请求数为3
4.请求$uri为/b，则test区间/b的键自增请求数为1

```
  http{
    limit_conn_zone $HTTP_AAA zone=test:10m;
    limit_conn_status 503;
    server {
      listen 80;
      root /tmp;
      location / {
          fastcgi_pass http://127.0.0.1:9000;
          include fastcgi.conf;
          limit_conn test 5;
      }
    }
  }
```
也可以指定在请求中带head头为AAA的并发数量

```
  curl 'http://127.0.0.1/a.php' -H 'AAA: 123'
```

也就是说，如果heade头AAA值为123，那么最多接受5个并发；如果有请求没有heade头AAA，那么没有并发限制

**注意**
如果用IP作为键限制，则应该使用$binary_remote_addr替代$remote_addr，因为$remote_addr参数长度在7至15个字节，而$binary_remote_addr长度在IPV4下总是为4字节，在IPV6下总是为16字节

如果区域存储已用完，服务器将将错误返回给所有其他请求

## limit_conn_status 
```
  Syntax:	limit_conn_status code;
  Default:	limit_conn_status 503;
  Context:	http, server, location
```
设置状态代码以响应拒绝的请求返回

注意这个code值应该在400和599之间

503的响应码具体信息为
```
<html>
<head><title>503 Service Temporarily Unavailable</title></head>
<body bgcolor="white">
<center><h1>503 Service Temporarily Unavailable</h1></center>
<hr><center>nginx/1.14.1</center>
</body>
</html>
```

## 并发限制于error_page的结合使用

使用ngx_http_limit_conn_module模块来做并发限频，默认情况下会返回http code503
```
<html>
<head><title>503 Service Temporarily Unavailable</title></head>
<body bgcolor="white">
<center><h1>503 Service Temporarily Unavailable</h1></center>
<hr><center>nginx/1.14.1</center>
</body>
</html>
```
这种非程序员能看懂的提示语不是我们想要的，所以我们应该结合error_page命令来与并发限频一起使用

```
http {
  include       mime.types;
  limit_conn_zone $uri$HTTP_id zone=test:32k;
  server {
      listen       80;
      server_name  localhost;
      error_page 599 = /response;
      location / {
          limit_conn_status 599;
          limit_conn test 10;
          root /tmp;
          fastcgi_pass 127.0.0.1:9000;
          include fastcgi.conf;
      }
      location /response {
          default_type  application/json;
          return 200 '{"code":1,"message":"请求超限"}';
      }
  }
}
```
以上配置中，如果请求$uri$HTTP_id并发达到了11，则limit_conn_status会返回599响应码，又因为我们有error_page指令，所以被转到location /response，最后返回
```
< HTTP/1.1 200 OK
< Server: nginx/1.14.1
< Date: Mon, 12 Nov 2018 14:56:30 GMT
< Content-Type: application/json
< Content-Length: 35
< Connection: keep-alive
< 
* Connection #0 to host 127.0.0.1 left intact
{"code":1,"message":"请求超限"}
```
