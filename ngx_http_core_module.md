
联系作者
- weixin: 363260961
- QQ: 363260961

**编写不易，转载请注明出处https://github.com/Zhucola/advanced-nginx**

使用版本
- CentOS Linux release 7.5.1804 (Core)
- nginx/1.12.2版本
- curl 7.61.0 (x86_64-pc-linux-gnu) libcurl/7.61.0 OpenSSL/1.0.2k zlib/1.2.7

## 目录
* [nginx如何处理一个请求](#nginx如何处理一个请求)
* [ngx_http_core_module](#ngx_http_core_module)
    * [default_type](#default_type)
    * [types](#types)
    * [root](#root)
    * [alias](#alias)
    * [error_page](#error_page)
    * [try_files](#try_files)
    * [merge_slashes](#merge_slashes)
    * [location](#location)
    * [server_tokens](#server_tokens)
    * [client_max_body_size](#client_max_body_size)
    * [client_header_timeout](#client_header_timeout)   
    * [etag](#etag)
    * [return](#return)
# nginx如何处理一个请求

nginx首先选定由那一个虚拟主机来处理请求
```nginx
server {
    listen 80;
    server_name a.com b.com;
    location / {
        return 200 "server_name is a.com b.com";
    }
}
server {
    listen 80;
    server_name c.com d.com;
    location / {
        return 500 "server_name is c.com d.com";
    }
}
```
在这个配置中，nginx**仅仅**检查请求的host头以决定该请求由那个虚拟主机来处理，如果host头没有匹配任意一个虚拟主机，或者请求中根本没有包含host头，那nginx会将请求分发到定义在此端口上的默认虚拟主机，第一个被列出的虚拟主机就是nginx的默认虚拟主机。也可以显式的某个虚拟主机为默认虚拟主机(default_server从0.8.21开始使用，在以前版本使用default代替)
```curl
    curl 'http://127.0.0.1' -H 'host: a.com'
```
以上请求的host被指定为a.com，所以匹配到server_name a.com b.com，返回http_code 200，消息体server_name is a.com b.com
```curl
    curl 'http://127.0.0.1' -H 'host: c.com'
```
以上请求的host被指定为c.com，所以匹配到server_name c.com d.com，返回http_code 500，消息体server_name is c.com d.com
```curl
    curl 'http://127.0.0.1' -H 'host: xxx.com'
```
以上请求的host被指定为xxx.com，没有server_name与其匹配，所以nginx会将请求分发到定义在此端口上的默认虚拟主机(第一个被列出的)，返回http_code 200，消息体server_name is a.com b.com

如果将nginx配置改为
```nginx
server {
    listen 80;
    server_name a.com b.com;
    location / {
        return 200 "server_name is a.com b.com";
    }
}
server {
    listen 80 default_server;
    server_name c.com d.com;
    location / {
        return 500 "server_name is c.com d.com";
    }
}
```
```curl
    curl 'http://127.0.0.1' -H 'host: xxx.com'
```
以上请求的host被指定为xxx.com，没有server_name与其匹配，所以nginx会将请求分发到定义在此端口上的默认虚拟主机(default_server定义的)，返回http_code 500，消息体server_name is c.com d.com
```nginx
server {
    listen 80;
    server_name "";
    return 444;
}
server {
    listen 80;
    server_name c.com d.com;
    location / {
        return 500 "server_name is c.com d.com";
    }
}
```
设置主机名为空字符串以匹配未定义Host头的请求，而且返回了一个nginx特有的，非http标准码444，可以用来关闭连接
```curl
    curl 'http://127.0.0.1' -H 'host: xxx.com'
```
返回curl: (52) Empty reply from server

不是一定要返回444，可以根据自身的业务需求来处理逻辑，比如我要返回"没有host头与之匹配"
```nginx
server {
    listen 80;
    server_name "";
    return 200 "没有host头与之匹配";
}
server {
    listen 80;
    server_name c.com d.com;
    location / {
        return 500 "server_name is c.com d.com";
    }
}
```
从0.8.48版本开始，server_name "" 以成为主机名的默认设置，所以可以省略server_name ""
```nginx
server {
    listen 192.168.1.111:80;
    server_name a.com;
    ...
}
server {
    listen 192.168.1.112:80;
    server_name b.com;
    ...
}
```
上面的配置中，nginx首先检查请求的IP地址和端口是否匹配某个server块中的listen指令配置。接着nginx继续测试请求host头是否匹配这个server块中的某个server_name值，如果没有匹配则将这个请求交给默认主机。

**默认服务器是监听端口的属性，所以不同的监听端口可以设置不同的默认服务器**

# ngx_http_core_module
## default_type
```
   Syntax:	default_type mime-type;
   Default:	default_type text/plain;
   Context:	http, server, location
```
   该指令应该和types指令配合学习
   
   定义**默认**响应类型(Content-Type)，**默认**的意思就是文件的扩展名不在nginx定义的MIME映射表里
   
   例外：如请求一个*.php文件，php扩展名不在MIME映射表，不会走default_type，因为php主动响应了Content-Type
   
```nginx
   root /tmp;
   index a.html;
   default_type text/plain;
   location / {
      return 200 123;
   }
```
```
   curl -v 'http://127.0.0.1'
```
以上会输出服务器响应结果，可见响应Content-Type: text/plain
```nginx
   root /tmp;
   index a.html;
   default_type a/b;
   location / {
      return 200 123;
   }
```
```
   curl -v 'http://127.0.0.1'
```
以上会输出服务器响应结果，可见响应Content-Type: a/b
```nginx
   root /tmp;
   index a.html;
   default_type a/b;
   location / {
      # 这里什么都不写
   }
```
```
   curl -v 'http://127.0.0.1'
```
执行命令echo "file is a.html" > /tmp/a.html，请求后响应Content-Type: text/html，不会执行default_type命令，因为会查看a.html，文件后缀html与imme.types里面的text/html     html htm shtml;相匹配

## types
```
   Syntax:	types { ... }
   Default:	types {
         text/html  html;
         image/gif  gif;
         image/jpeg jpg;
      }
   Context:	http, server, location
```
设置文件扩展名和响应的MIME类型的映射表，可以将多个扩展名映射到同一种类型

nginx都会有一行include mime.types;的，可以去mime.types里面查看映射信息，里面包含了足够多的类型

如果不想使用mime.types，而为所有请求都响应唯一的Content-Type，可以使用如下配置
```nginx
   location / {
      types {}
      default_type application/json;
   }
```

## root
```
   Syntax:	root path;
   Default:	
   root html;
   Context:	http, server, location, if in location
```
仅仅是将uri拼到root值的后面

path值可以是变量，但是不能是$document_root和$realpath_root；因为$document_root和$realpath_root是根据root或者alias来定义的

如果nginx的编译路径是/usr/local/nginx，则默认的root位置是/usr/local/nginx/html
```nginx
   root /tmp;
   location /a {
      return 200 $request_filename;
   }
```
```curl
curl 'http://127.0.0.1/a/b'
```
会输出/tmp/a/b

## alias
```
   Syntax:	alias path;
   Default:	—
   Context:	location
```
path值可以是变量，但是不能是$document_root和$realpath_root；因为$document_root和$realpath_root是根据root或者alias来定义的

指定一个指定路径的替换路径

alias和rewrite不能同时出现
```nginx
   alias /tmp;
   location /a {
      return 200 $request_filename;
   }
```
```curl
curl 'http://127.0.0.1/a/b'
```
会输出/tmp/b
```nginx
   alias /tmp;
   merge_slashes on;
   location /a/ {
      return 200 $request_filename;
   }
```
```curl
curl 'http://127.0.0.1/a/b'
```
会输出/tmpb

## error_page
```
   Syntax:	error_page code ... [=[response]] uri;
   Default:	—
   Context:	http, server, location, if in location
```
  为错误定义显示的URI或者状态码
```nginx
   root /tmp;
   location /a {
      error_page 404 /b;   
   }
   location /b {
      return 200 123;
   }
```
如果请求uri为/a，出现404，则uri变为/b，重新查找location，返回123；对于http_code，即使return 200，最后的结果也是404
```nginx
   root /tmp;
   location /a {
      error_page 404 /b;   
   }
```
如果请求uri为/a，出现404，则uri变为/b，查找/tmp/b，返回/tmp/b的文件内容；对于http_code，即使return 200，最后的结果也是404

```nginx
   location / {
      error_page 500 502 503 504 /50x.html;   
   }
```
对于http_code500 502 503 504，使用error_page

```nginx
   location /a {
      error_page 404 =200 /200.html;   
   }
```

如果发生404，则error_page执行后的响应码是200，**注意404后面有一个空格**

```nginx
   location /a {
      error_page 404 = /200.php;   
   }
```
如果URI将被发送到一个被代理的服务器处理，或者发送到一个FastCGI服务器处理，这些后端服务器又返回了不同的响应码，那么这些响应码可以由根据被处理后的结果展现，以上配置的最终响应码取决于/200.php，**注意等号两面都有空格**

```nginx
   location /a {
      error_page 404 http://127.0.0.1:81/a;   
   }
   location /b {
      error_page 404 =200 http://127.0.0.1:81/b;   
   }
```
也可以进行重定向
```nginx
   location /a {
      error_page 404 @back;   
   }
   location @back {
      return 200 $uri;
   }
```
如果不希望error_page改变uri，可以将错误转到一个命名路径，比如uri /a发生404，则跳转到location @back，最后返回/a，uri不会改变

**如果不存在location @back，则会发生500 Internal Server Error**

## try_files
```
   Syntax:	try_files file ... uri;
            try_files file ... =code;
   Default:	—
   Context:	server, location
```
按照指定顺序检查文件是否存在，并且使用第一个找到的文件来处理请求。文件路径是根据root和alias指令，将file参数拼接而成

如果找不到，则按最后一个参数指定的uri进行内部跳转
```
   server{
      root /tmp;
      index index.html;
      location / {
         try_files "/a" "/b";
      }
   }
```
```
   curl 'http://127.0.0.1'
```
在/tmp中没有index.html，创建echo "123" > a，uri为"/",try_files会找/tmp/a，这个文件，返回文件内容123

如果/tmp/a文件不存在，则会将请求uri变成/b,重新查找location，以上配置会返回http_code500，因为进入查找死循环

不能将"/a"改为"a"，这样会查找/tmpa文件，也不能将"/b"改为"b"，这样会将uri变成"b"，正确的uri应该是"/b"

```
   server{
      root /tmp;
      index index.html;
      location / {
         try_files "/a" "/b" "/c" "/d";
      }
   }
```
以上配置，会查找/tmp/a、/tmp/b、/tmp/c文件，如果有一个找到则停止查找；如果都找不到则uri变成/d

```
   server{
      root /tmp;
      index index.html;
      location / {
         try_files "/a" "/d" =404;
      }
   }
```
以上配置，会查找/tmp/a文件，如果都找不到则uri变成/d，并且响应码为http_code 404

## merge_slashes
```
   Syntax:	merge_slashes on | off;
   Default:	merge_slashes on;
   Context:	http, server
```
   开启或者关闭将请求中URI相邻的两个或者更多斜线合并成一个的功能  
   压缩URI对于前缀匹配和正则匹配的正确性是很重要的，没有开启这个功能时，请求//script/one.php不能被location /scripts/匹配
   如果URI中包含base64编码内容，必须将斜线压缩调整成off，因为base64编码本身会使用"/"字符，然而出于安全方面的考虑，最好还是不要关闭压缩
```nginx
   merge_slashes on;
   location / {
      return 200 $uri;
   }
```
```curl
curl 'http://127.0.0.1/a//b'
```
会输出/a/b

## location
```
   Syntax:	location [ = | ~ | ~* | ^~ ] uri { ... }    location @name { ... }
   Default:	—
   Context:	server, location
```
根据请求的URI来设置配置
* \~    执行一个正则匹配，区分大小写，**匹配成功则停止**
* \~ \* 执行一个正则匹配，不区分大小写，**匹配成功则停止**
* ^~   前缀匹配，区分大小写，**匹配成功则停止**
* =   精准匹配，区分大小写，**匹配成功则停止**
* /abc    常规字符串匹配，**如果有多个能匹配的话，使用匹配最长的那个**
* @    定义一个命名的location，使用在内部定向时，如error_page

优先级：(从高到低)
* =
* ^~
* \~ \*    \~ 
* /abc

```nginx
   location = /ab {  
      return 200 "= /ab"
   }
   location /a {  
      return 200 "/a"
   }
   location /ab {  
      return 200 "/ab"
   }
```
```curl
curl 'http://127.0.0.1/a'
```
返回http_code 200,消息体/a
```nginx
   location /a {  
      return 200 "/a"
   }
   location /ab {  
      return 200 "/ab"
   }
```
```curl
curl 'http://127.0.0.1/ab'
```
返回http_code 200,消息体/ab
```nginx
   location /a {  
      return 200 "/a"
   }
   location /ab {  
      return 200 "/ab"
   }
```
```curl
curl 'http://127.0.0.1/ab'
```
返回http_code 200,消息体/ab
```nginx
   location = /ab {  
      return 200 "= /ab"
   }
   location ^~ /ab {  
      return 200 "^~ /ab"
   }
```
```curl
curl 'http://127.0.0.1/ab'
```
返回http_code 200,消息体= /ab
```nginx
   location ~ /ab {  
      return 200 "~ /ab"
   }
   location ~* /ab {  
      return 200 "~* /ab"
   }
```
```curl
curl 'http://127.0.0.1/aB'
```
返回http_code 200,消息体~\* ab

**=与^~不支持正则**
```nginx
   location = /abc$ {  # $不是正则的结束标志
      return 200 $uri;
   }
```
可以使用前缀字符串或者正则表达式定义路径。使用正则表达式需要在路径开始添加"\~\*"前缀 (不区分大小写)，或者"\~"前缀(区分大小写)。为了根据请求URI查找路径，nginx先检查前缀字符串定义的路径 (前缀路径)，在这些路径中找到能最精确匹配请求URI的路径。然后nginx按在配置文件中的出现顺序检查正则表达式路径， 匹配上某个路径后即停止匹配并使用该路径的配置，否则使用最大前缀匹配的路径的配置

**注意**

**^~ /a与/a不能同时写，会报错**
```nginx
   location = /a {  
      return 200 $uri;
   }
   location ^~ /a {  
      return 200 $uri;
   }
```
```nginx
nginx: [emerg] duplicate location "/a" 
```
路径匹配在URI**规范化**以后进行，所谓规范化，就是先将URI中形如"%XX"的编码字符进行编码，再解析URI中的相对路径"."和".."部分，另外还可能会压缩相邻的两个或多个斜线成为一个斜线(需要merge_slashes on;)
```nginx
   location /abc+ {
      return 200 $uri;
   }
```
```curl
curl 'http://127.0.0.1/abc%2B'
```
因为请求URI会规范化，所以将/abc%2B解析成/abc+，返回http_code 200,消息体/abc+
```nginx
   location /a {
      return 200 $uri;
   }
```
```curl
curl 'http://127.0.0.1/b/../a'
```
因为请求URI会规范化，所以将/b/../a解析成/a，返回http_code 200,消息体/a
路径可以嵌套
```nginx
   location /{
      location /a {
         return 200 "a";
      }
      location /b {
         return 200 "b";
      }
   }
```
```curl
curl 'http://127.0.0.1/a'
```
返回http_code 200,消息体/a

如果请求"\/"出现频繁，定义"location \= \/"可以提高这些请求的处理速度， 因为查找过程在第一次比较以后即结束
```nginx
   server {
      listen 80;
      location /a/ {
         proxy_pass http://127.0.0.1:81/b;
      }
   }
   server {
      listen 81;
      location / {
         return 200 $uri;
      }
   }
```
```curl
curl 'http://127.0.0.1/a'
```
返回http_code 301
```nginx
   server {
      listen 80;
      location /a {
         proxy_pass http://127.0.0.1:81/b;
      }
   }
   server {
      listen 81;
      location / {
         return 200 $uri;
      }
   }
```
```curl
curl 'http://127.0.0.1/a'
```
返回http_code 200，响应体/b

```nginx
   root /tmp;
   index index.css;
   server {
      listen 80;
      location / {
       
      }
      location /index.css {
         return 200 123;
      }
   }
```
请求uri为/，则uri变为/index.css，查找location /index.css，即使有/tmp/index.css，最后也会返回return 200 123

## server_tokens
```
   Syntax:	server_tokens on | off | build | string;
   Default: server_tokens on;
   Context:	http, server, location
```
开启或者关闭在响应头中输出nginx版本号
```
   HTTP/1.1 200 OK
   Server: nginx/1.14.0
```
参数build，如果编译时候configure -–build=8，则
```
   HTTP/1.1 200 OK
   Server: nginx/1.14.0 (8)
```
参数off会不显示具体nginx版本
```
   HTTP/1.1 200 OK
   Server: nginx
```
参数string在商业版本中使用

## client_max_body_size
```
   Syntax:	client_max_body_size size;
   Default:	client_max_body_size 1m;
   Context:	http, server, location
```

设置允许客户端请求正文的最大长度，请求长度由Content-Length请求头指定。如果超过设定值，将返回413(Request Entity Too Large)。浏览器不能正确显示这个错误。size为0可以使nginx不检查客户端请求正文长度

当实际传的正文大于nginx限制，但是重写Content-Length，不会报错

```
   curl 'http://127.0.0.1' -H 'Content-Length: 1234
```
默认值1m的可接受最大Content-Length为1048576

## client_header_timeout
```
   Syntax:	client_header_timeout time;
   Default:	client_header_timeout 60s;
   Context:	http, server
```
定义读取客户端请求头部的超时。如果客户端在这段时间内没有传送完整的头部到nginx， nginx将返回错误408 (Request Time-out)到客户端

## etag
```
   Syntax:	etag on | off;
   Default:	etag on;
   Context:	http, server, location
```
开启或者关闭静态文件自动计算etag响应头

只是在静态文件中有效

默认情况下，请求静态文件会响应etag头，如ETag: "5bc887e1-2"

## return
```
   Syntax:	return code [text];
               return code URL;
               return URL;
   Default:	—
   Context:	server, location, if

```
停止处理并返回指定code给客户端，返回非标准状态码444可以直接关闭连接而不返回响应头
```
   server {
      listen 80;
      location / {
         return 444;
      }
   }
```
请求127.0.0.1:80，请求过程为
```
* Rebuilt URL to: http://127.0.0.1/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1
> User-Agent: curl/7.61.0
> Accept: */*
> 
* Empty reply from server
* Connection #0 to host 127.0.0.1 left intact
curl: (52) Empty reply from server
```
```
   server {
      listen 80;
      location / {
         return 444 "abc";
      }
   }
```
请求127.0.0.1:80，请求过程为
```
* Rebuilt URL to: http://127.0.0.1/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1
> User-Agent: curl/7.61.0
> Accept: */*
> 
< HTTP/1.1 444 
< Server: nginx/1.12.2
< Date: Tue, 06 Nov 2018 13:04:21 GMT
< Content-Type: text/plain
< Content-Length: 3
< Connection: keep-alive
< 
* Connection #0 to host 127.0.0.1 left intact
abc
```

可以在指令中指定重定向的URL（0.8.42版本之后），状态码为301、302、303和307，或者指定响应体文本

return http://uri  响应码为302

0.7.51版本以前只能返回204、400、402-406、408、410、411、413、416和500-504

直到1.1.16和1.0.13版本，307才被认为是一种重定向

1.13.0可以返回308
```
   server {
      listen 80;
      location / {
         return http://127.0.0.1:81;
      }
   }
```
请求127.0.0.1:80，会返回响应码302，如果请求支持重定向（如 curl -L ），则重定向到127.0.0.1:81

**注意**

对于return重定向，要么写成return http:\/\/url这种形式，
