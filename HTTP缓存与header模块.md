## 目录
* [HTTP缓存](#HTTP缓存)
  * [缓存控制](#缓存控制)
  * [缓存校验](#缓存校验)
  * [200状态码](#200状态码)
  * [304状态码](#304状态码)
  * [200fromDiskCache](#200fromDiskCache)
  * [具体缓存流程](#具体缓存流程)
* [header模块](#header模块)
  * [expires](#expires)
  * [add_header](#add_header)
* [Yii2框架HTTP缓存源码](#Yii2框架HTTP缓存源码)

# HTTP缓存
## 缓存控制
HTTP缓存分为缓存控制和缓存校验，缓存控制有Cache-Control和Pragma 
  
Pragma是旧产物，已经逐步抛弃，有些网站为了向下兼容还保留了这两个字段。如果一个报文中同时出现Pragma和Cache-Control时，以Pragma为准。同时出现Cache-Control和Expires时，以Cache-Control为准。即优先级从高到低是 Pragma -> Cache-Control -> Expires  
  
如果在请求header有如下参数  
```
Cache-Control: public,max-ae=86400
Pragma: no-cache
```
则Pragma的优先级更高

Cache-Control一般值为  
- no-cache，表示不管有没有缓存都去拿真实数据，不会发生304，就是强制刷新
- max-age=0，表示不管响应怎么设置，在重新获取数据前需要去校验ETag或者Last-Modified，校验通过就是304，就是在页面正常刷新
- max-age=自己设置的值，服务器响应客户端，表示要求客户端缓存多长时间

## 缓存校验
缓存校验有Last-Modified和ETag  

如果请求Cache-Control值为max-age=0，表示客户端要去服务端做资源校验，校验通过会发生304，使用本地缓存的资源，校验不通过的话，服务端将数据返回给客户端 
  
服务端在响应时候会有响应头Last-Modified，这是一个格林威治时间,表示资源最后的修改时间
```
Last-Modified: Thu, 01 Aug 2019 07:24:42 GMT
```
客户端在刷新页面时候，会发一个请求头If-Modified-Since，表示收到的上一次服务端给的Last-Modified
```
If-Modified-Since: Thu, 01 Aug 2019 07:24:42 GMT
```
当服务端会对比自己的Last-Modifed和客户端的If-Modified-Since,如果  

If-Modified-Since >= Last-Modifed  
  
那么服务端会直接响应304，响应body体长度为0，以下为一个304响应的nginx-access日志
```
192.168.124.1 - - [31/Jul/2019:13:43:46 +0800] "GET /a.css HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, l
ike Gecko) Chrome/73.0.3683.86 Safari/537.36"
```
但是Last-Modifed无法解决资源在一秒内连续修改的问题，一秒内连续修改后，客户端只会更新一次  

更好的解决方法是ETag，服务器会响应一个根据资源算出来的字符，如
```
ETag: "5d4293ba-19aa"
```
第二次客户端请求时候会携带If-None-Match请求头
```
If-None-Match: "5d4293ba-19aa"
```
如果  
  
If-None-Match == ETag

表示资源没有修改，服务端响应304  
  
如果同时有ETag和Last-Modify，则ETag的优先级会更高  

但是ETag也有问题，如果服务端是多节点集群，那么有可能A节点算出来的ETag和B节点的ETag可能不同，造成无法正常304响应，在nginx中可以关闭ETag
```
 etag off;
```
## 200状态码
200状态码会发生于浏览器第一次加载页面、强制刷新、304校验失败、资源缓存过期、浏览器禁用缓存情况  
  
如果是浏览器第一次加载，那么请求头不会有Cache-Control、If-None-Match、If-Modified-Since  
  
服务端正常响应200，正常将数据传给客户端
  
如果是强制刷新，那么浏览器会强制加请求头
```
Cache-Control: no-cache
Pragma: no-cache
```
表示需要服务端响应真实数据，不用做校验  
  
如果浏览器禁用了缓存Disable Cache，那么也会强制加请求头no-cache
## 304状态码
304状态码会发生于刷新页面情况  
  
刷新情况浏览器会强制加请求头
```
Cache-Control: max-age=0
```
  
表示需要浏览器校验，校验成功就是304，校验失败就是200
## 200fromDiskCache
发生过程
- 浏览器输入url
- 再开一个新窗口，输入url，发生200from disk cache  

由于访问静态资源，服务端通常都会响应Cache-Control:max-age，表示需要客户端缓存这个静态资源多长时间，如
```
Cache-Control: max-age=36536000
```
同一个浏览器新窗口再次访问会发生from disk cache  

from disk cache情况服务端不会收到请求
## 具体缓存流程
- 浏览器第一次加载url，不会有请求头Cache-Control、If-None-Match、If-Modified-Since,服务端响应200，响应头为
```
Cache-Control:max-age=86400   //告诉客户端缓存86400秒
Last-Modify:Wed,31 Jul 2019 04:05:42 GMT   //资源最后修改时间
ETage: "asdadsa"   //资源的ETag
```
- 刷新页面(服务端资源未修改)，请求头如下，发生304
```
If-Modified-Since:Wed,31 Jul 2019 04:05:42 GMT   //客户端保存的资源最后修改时间
ETag: "asdadsa"
```
- 刷新页面(服务端资源修改)，发生200
- 强制刷新，请求头，发生200
```
Cache-Control:no-cache
```
- 新打开一个页面，缓存86400秒内，发生200 from disk cache
- 新打开一个页面，缓存86400秒外，发生200

# header模块
## expires 
```
Syntax:	expires [modified] time;
expires epoch | max | off;
Default:	
expires off;
Context:	http, server, location, if in location
```
响应码为200, 201, 206, 301, 302, 303, 307, 308情况下会发这个响应头  
参数可以是正数或者负数，如果为负，则发送的头为
```
Cache-Control:no-cache
```
可以配置max，这样响应的就是，可以认为是永久缓存
```
Expires:Thu,31 Dec 2037 23:55:55 GMT 
Cache-Control:max-age=315360000
```
off参数可以禁用添加或者修改expire和Cache-Control响应  

# add_header
```
Syntax:	add_header name value [always];
Default:	—
Context:	http, server, location, if in location
```
响应码为200、201、204、206、301、302、303、304、307、308则添加指定的响应头  
如果当前级别没有add_header，则从上一个级别继承，仅仅当前级别没有的话  
如果定义了always，则不管响应码为多少都添加header  

