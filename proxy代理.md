## 目录 
* [proxy_bind](#proxy_bind)
* [proxy_pass](#proxy_pass)
* [proxy_set_header](#proxy_set_header)

# proxy_bind
```
Syntax:	proxy_bind address [transparent] | off;
Default:	—
Context:	http, server, location
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
