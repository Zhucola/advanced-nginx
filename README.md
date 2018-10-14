advanced-nginx
=======================
涉及
- nginx编译与安装
- nginx各模块基本操作与总结
- openssl私钥、CSR、CRT、自签CA的搭建


联系作者
- weixin: 363260961

使用版本
- CentOS Linux release 7.5.1804 (Core)
- nginx/1.12.2版本
- curl 7.61.0 (x86_64-pc-linux-gnu) libcurl/7.61.0 OpenSSL/1.0.2k zlib/1.2.7

## 目录
* [nginx如何处理一个请求](#nginx如何处理一个请求)
* [ngx_http_core_module](#ngx_http_core_module)
    * [merge_slashes](#merge_slashes)
    * [location](#location)


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
* ^~   前缀匹配，**匹配成功则停止**
* =   精准匹配，**匹配成功则停止**
* /abc    常规字符串匹配，**如果有多个能匹配的话，使用匹配最长的那个**
* @    定义一个命名的location，使用在内部定向时，如error_page

优先级：(从高到低)
* =
* ^~
* \~ \*  \~ 
* /abc


路径匹配在URI规范化以后进行，所谓规范化，就是先将URI中形如"%XX"的编码字符进行编码，再解析URI中的相对路径"."和".."部分，另外还可能会压缩相邻的两个或多个斜线成为一个斜线(需要merge_slashes on;)
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

可以使用前缀字符串或者正则表达式定义路径。使用正则表达式需要在路径开始添加"~\*"前缀 (不区分大小写)，或者"~"前缀(区分大小写)。为了根据请求URI查找路径，nginx先检查前缀字符串定义的路径 (前缀路径)，在这些路径中找到能最精确匹配请求URI的路径。然后nginx按在配置文件中的出现顺序检查正则表达式路径， 匹配上某个路径后即停止匹配并使用该路径的配置，否则使用最大前缀匹配的路径的配置。


