## 目录
* [HTTP缓存](#HTTP缓存)
  * [缓存控制](#缓存控制)
  * [缓存校验](#缓存校验)
  * [304状态码](#304状态码)
  * [200 from disk cache](#200 from disk cache)
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


