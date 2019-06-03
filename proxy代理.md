## 目录 
* [proxy_connect_timeout](#proxy_connect_timeout)
* [proxy_method](#proxy_method)
* [proxy_http_version](#proxy_http_version)
* [proxy_pass_request_body](#proxy_pass_request_body)
* [proxy_pass](#proxy_pass)
* [proxy_set_header](#proxy_set_header)
* [underscores_in_headers](#underscores_in_headers)
* [获取代理模式下的真实用户ip](#获取代理模式下的真实用户ip)

# proxy_connect_timeout
```
Syntax:	proxy_connect_timeout time;
Default:	
proxy_connect_timeout 60s;
Context:	http, server, location
```
设置于代理服务器的连接超时时间，这个时间不能超过75秒
# proxy_method
```
Syntax:	proxy_method method;
Default:	—
Context:	http, server, location

```
指定代理方法而不是使用客户端请求的方法

如果client c -> proxy server s1 -> server s2

使用get方式请求client，那么s2收到的请求方式是get

如果在proxy server s1指定
```
  proxy_method POST;
```
那么收到的请求方式就是POST

# proxy_http_version
```
Syntax:	proxy_http_version 1.0 | 1.1;
Default:	
proxy_http_version 1.0;
Context:	http, server, location
```
1.1版本推荐在keep alive下使用

需要在每一个被使用到的proxy都设置
# proxy_pass_request_body
```
Syntax:	proxy_pass_request_body on | off;
Default:	
proxy_pass_request_body on;
Context:	http, server, location
```
决定是否传送原始body给代理服务器

单独将proxy_pass_request_body off还是会将body传给server端，需要
```
location /x-accel-redirect-here/ {
    proxy_method GET;
    proxy_pass_request_body off;
    proxy_set_header Content-Length "";
    proxy_pass ...
}
```
# proxy_pass
```
Syntax:	proxy_pass URL;
Default:	—
Context:	location, if in location, limit_except
```
设置一个代理协议和地址

协议可以是http或者https，地址可以带端口和uri
```
  server {
    listen 80;
    location / {
      proxy_pass http://127.0.0.1:81;
    }
  }
  server {
    listen 81;
    location / {
      return 200 $request_uri;
    }
  }
```
请求80端口uri为/abc，将会proxy_pass到81端口，输出/abc

```
  server {
    listen 80;
    location /abc {
      proxy_pass http://127.0.0.1:81/xy;
    }
  }
  server {
    listen 81;
    location / {
      return 200 $request_uri;
    }
  }
```
请求80端口uri为/abcd，将会proxy_pass到81端口，输出/xyd，就是实际请求的uri(/abcd)，与location的/abc做差集，结果为d,将d拼上proxy_pass的uri(/xy)，最后代理过去的uri为/abcd
```
  server {
    listen 80;
    location /abc {
      proxy_pass http://127.0.0.1:81;
    }
  }
  server {
    listen 81;
    location / {
      return 200 $request_uri;
    }
  }
```
请求80端口uri为/abcd，将会proxy_pass到81端口，输出/abcd，因为proxy_pass没有默认的uri，所以不做差集处理

```
  server {
    listen 80;
    location /abc {
      proxy_pass http://127.0.0.1:81/;
    }
  }
  server {
    listen 81;
    location / {
      return 200 $request_uri;
    }
  }
```
请求80端口uri为/abcd，将会proxy_pass到81端口，输出/，因为proxy_pass有默认的uri为/，所以做差集处理

```
  server {
    listen 80;
    location /xxx {
      rewrite ^/xxx /yyy break;
      proxy_pass http://127.0.0.1:81;
    }
  }
  server {
    listen 81;
    location / {
      return 200 $request_uri;
    }
  }
```
请求80端口uri为/xxx，将会proxy_pass到81端口，因为有rewrite并且是break，所以改变了uri，输出/yyy

```
  server {
    listen 80;
    location /a {
      proxy_pass http://127.0.0.1:81/oooooo;
    }
  }
  server {
    listen 81;
    location / {
      return 200 $request_uri;
    }
  }
```
请求80端口uri为/abc，将会proxy_pass到81端口，uri做差集，最后输出结果为/oooooobc

# proxy_set_header
```
Syntax:	proxy_set_header field value;
Default:	
proxy_set_header Host $proxy_host;
proxy_set_header Connection close;
Context:	http, server, location
```
允许重新定义或者追加header头到proxy服务器，可以包含文本、变量和他们的组合
指令会继承于上一个级别，如果当前级别没有proxy_set_header被定义
```
  server {
    listen 80;
    proxy_set_header AA aa;
    location / {
      proxy_pass http://127.0.0.1:81;
    }
  }
  server {
    listen 81;
    location / {
      return 200 $HTTP_AA;
    }
  }
```
请求80端口uri为/，会输出aa，因为80的location /里面没有定义proxy_set_header，所以会继承上一个级别的proxy_set_header
```
  server {
    listen 80;
    proxy_set_header AA aa;
    location / {
      proxy_set_header BB bb;
      proxy_pass http://127.0.0.1:81;
    }
  }
  server {
    listen 81;
    location / {
      return 200 $HTTP_AA;
    }
  }
```
请求80端口uri为/，会输出空
```
  server {
    listen 80;
    location / {
      proxy_set_header BB bb;
      proxy_pass http://127.0.0.1:81;
    }
  }
  server {
    listen 81;
    location / {
       proxy_set_header BB xx;
       proxy_pass http://127.0.0.1:82;
    }
  }
  server {
    listen 82;
    location / {
      return 200 $HTTP_BB;
    }
  }
```
请求80端口uri为/，会输出xx，相当于81端口对head头BB做了重写
```
  server {
    listen 80;
    location / {
      proxy_set_header AA aa;
      proxy_pass http://127.0.0.1:81;
    }
  }
  server {
    listen 81;
    location / {
       proxy_set_header BB bb;
       proxy_pass http://127.0.0.1:82;
    }
  }
  server {
    listen 82;
    location / {
      return 200 $HTTP_AA;
    }
  }
```
请求80端口uri为/，可以获取到第一次proxy添加的header头，所以输出aa
# underscores_in_headers
```
Syntax:	underscores_in_headers on | off;
Default:	
underscores_in_headers off;
Context:	http, server
```
自定义的header头变量是否可以带下划线
```
server {
    listen 81;
    underscores_in_headers off;
    location / {
       proxy_set_header aa $http_a_b;
       proxy_pass http://127.0.0.1:82;
    }
  }
  server {
    listen 82;
    location / {
      return 200 $HTTP_AA;
    }
  }
```
```
  curl 'http://127.0.0.1:81' -H 'a_b: 123'
```
请求将获取不到aa的header头，需要将underscores_in_headers改为on这样才能获取到$http_a_b
```
server {
    listen 81;
    underscores_in_headers on;
    location / {
       proxy_set_header aa $http_a_b;
       proxy_pass http://127.0.0.1:82;
    }
  }
  server {
    listen 82;
    location / {
      return 200 $HTTP_AA;
    }
  }
```
```
  curl 'http://127.0.0.1:81' -H 'a_b: 123'
```
输出123

```
server {
    listen 81;
    underscores_in_headers on;
    location / {
       return 200 $HTTP_AA_A;
    }
  }
```
```
  curl 'http://127.0.0.1' -H 'AA_A: 123'
```
输出123
```
server {
    listen 81;
    underscores_in_headers off;
    location / {
       return 200 $HTTP_AA_A;
    }
  }
```
```
  curl 'http://127.0.0.1' -H 'AA-A: 123'
```
输出123
# 获取代理模式下的真实用户ip
- Client C -> Server S  S获取的remote_addr为C的ip
- Client C1 -> proxy Server S1 -> proxy Server S2 -> Server S3     S3获取的remote_addr为S2的

在proxy_pass情况下，应该添加
```
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```
这样Server S3获取的HTTP_X_FORWARDED_FOR为Client C1,Server S1  S3获取的remote_addr为S2的，真实的客户端ip为HTTP_X_FORWARDED_FOR最左端的

也可以在proxy Server S1中添加
```
  proxy_set_header X-Real-IP $remote_addr;
```

这样在Server S3中获取到的HTTP_X_REAL_IP就是client的ip

Client C1 -> proxy Server S1 -> proxy Server S2 -> Server S3需要在每一个proxy里面都写proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;，如果proxy Server S2中没写，则Server S3获取到的就是client c1，remote_addr为s1的
