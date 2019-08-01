## 目录
* [HTTP缓存](#HTTP缓存)
  * [缓存控制](#缓存控制)
  * [缓存校验](#缓存校验)
  * [304状态码](#304状态码)
  * [200fromDiskCache](#200fromDiskCache)
* [header模块](#header模块)

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
