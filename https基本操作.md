**查看OpenSSL的基本操作请移步至 OpenSSL.md**

ngx_http_ssl_module模块使nginx能够支持https

该模块**不默认编译**，使用该模块需要--with-http_ssl_module

该模块需要OpenSSL库

为了减少服务器负载，建议配置
```nginx
worker_processes auto;
http {
    server {
        listen              443 ssl;
        keepalive_timeout   70;

        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5;
        ssl_certificate     /usr/local/nginx/conf/cert.pem;
        ssl_certificate_key /usr/local/nginx/conf/cert.key;
        ssl_session_cache   shared:SSL:10m;
        ssl_session_timeout 10m;
    }
```

## 目录
* [ssl](#ssl)
* [ssl_buffer_size](#ssl_buffer_size)
* [ssl_protocols](#ssl_protocols)
* [ssl_certificate ](#ssl_certificate)
* [ssl_certificate_key](#ssl_certificate_key)
* [ssl_password_file](#ssl_password_file)
* [ssl_verify_client](#ssl_verify_client)
* [ssl_client_certificate](#ssl_client_certificate)
* [ssl_session_cache](#ssl_session_cache)
* [ssl_session_timeout](#ssl_session_timeout)

# ssl
```nginx
  Syntax:	ssl on | off;
  Default:	ssl off;
  Context:	http, server
```
该指令在1.15.0以上版本被淘汰，应该使用listen 443 ssl;这种配置来代替

# ssl_buffer_size 
```
    Syntax:	    ssl_buffer_size size;
    Default:    ssl_buffer_size 16k;
    Context:	http, server
```
定义发送数据buffer区域的大小

如果发送的数据较大，则使用默认的16k就好

如果发送的数据小，则使用4k

# ssl_protocols
```nginx
  Syntax:	ssl_protocols [SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2] [TLSv1.3];
  Default:	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  Context:	http, server
```
指定支持的协议版本

如果客户端使用的协议版本大于服务端支持的，则报错SSL23_GET_SERVER_HELLO:unsupported protocol


**TLSv1.1 TLSv1.2 仅仅在OpenSSL1.0.1以上版本才支持**

**TLSv1.3仅仅在OpenSSL1.1.1版本才支持**

# ssl_certificate 
```nginx
  Syntax:	ssl_certificate file;
  Default:	—
  Context:	http, server
```
给指定的主机提供PEM格式的证书

如果还需要提供中间证书，那么应该按照顺序先指定主证书，然后提供中间证书

从版本1.11.0开始，该指令可以指定多个不同类型的证书，如RSA和ECDSA
```nginx
server {
    listen              443 ssl;
    server_name         example.com;

    ssl_certificate     example.com.rsa.crt;
    ssl_certificate_key example.com.rsa.key;

    ssl_certificate     example.com.ecdsa.crt;
    ssl_certificate_key example.com.ecdsa.key;
    ...
}
```
# ssl_certificate_key 
```
  Syntax:	ssl_certificate_key file;
  Default:	—
  Context:	http, server
```
提供一个配置与ssl_certificate证书的秘钥

配置了秘钥后，每次重启都需要重新输入秘钥密码，除非使用指令ssl_password_file指定密码

# ssl_password_file
```
  Syntax:	ssl_password_file file;
  Default:	—
  Context:	http, server
```
在1.7.3版本中出现

提供一个与指定ssl_certificate_key秘钥文件的密码

**如果不指定该指令，并且秘钥有密码，reload即使输入了正确的密码，reload也不会生效**

其中可以提供多个密码，每一行一个，在使用密码的时候轮流尝试密码
```nginx
server {
    listen              443 ssl;
    server_name         example.com;

    ssl_certificate     example.com.rsa.crt;
    ssl_certificate_key example.com.rsa.key;
    ssl_password_file /tmp/openssl/pass;
    ...
}
```
# ssl_verify_client 

```
  Syntax:	ssl_verify_client on | off | optional | optional_no_ca;
  Default:	ssl_verify_client off;
  Context:	http, server
```
是否校验客户端证书，校验结果会保存在变量$ssl_client_verify中

参数optional会请求客户端证书，客户端提供了证书的时候校验，不提供不校验

参数optional_no_ca会请求客户端证书，但是客户端证书不需要被CA证书信任

该指令与ssl_client_certificate 配套使用，需要指定ssl_client_certificate 参数

# ssl_client_certificate 
```
  Syntax:	ssl_client_certificate file;
  Default:	—
  Context:	http, server
```
指定一个受信与CA的证书，这个证书用来做客户端证书校验

指定的证书在请求的时候**会**响应给给客户端

```nginx
server {
          listen       443 ssl;
          keepalive_timeout 70;
          ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
          ssl_ciphers         AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5;
          ssl_certificate     /tmp/openssl/nginx.crt;
          ssl_certificate_key /tmp/openssl/a.key;
          ssl_password_file /tmp/openssl/pass;
          ssl_verify_client on;                                                                                                     
          ssl_client_certificate /etc/pki/CA/cacert.pem;
          location / {
              return 200 123;
          }
      }
```

# ssl_trusted_certificate
```
  Syntax:	ssl_trusted_certificate file;
  Default:	—
  Context:	http, server
```
指定一个受信与CA的证书，这个证书用来做客户端证书校验

指定的证书在请求的时候**不会**响应给给客户端

## ssl_session_cache
```
    Syntax:	    ssl_session_cache off | none | [builtin[:size]] [shared:name:size];
    Default:	ssl_session_cache none;
    Context:	http, server
```
定义如何存储session

参数none

    nginx告诉客户端可以重用session，但是nginx不会存储session，实际上没有使用session
    
参数off

    nginx告诉客户端不可以重用session
    
参数builtin

    在OpenSSL中构建的缓存；由一个工作进程实现功能。缓存大小在会话中指定。如果没有给出大小，则等于20480个会话。使用内置缓存可能导致内存碎片化。

参数shared

    在所有工作进程之间共享的高速缓存。缓存大小以字节指定；一兆字节可以存储大约4000个会话。每个共享缓存应该具有任意的名称。同一名称的缓存可以在多个虚拟服务器中使用
```
    ssl_session_cache builtin:1000 shared:SSL:10m;
```
builtin和shared可以同时指定，但是使用shared会更加高效

## ssl_session_timeout
```
    Syntax:	    ssl_session_timeout time;
    Default:	ssl_session_timeout 5m;
    Context:	http, server
```
提供一个给客户端重用session的时间
