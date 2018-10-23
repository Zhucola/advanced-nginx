advanced-nginx
=======================
涉及
- nginx编译与安装
- nginx各模块基本操作与总结

联系作者
- weixin: 363260961
- QQ: 363260961

nginx交流群
-  935629460

使用版本
- CentOS Linux release 7.5.1804 (Core)
- nginx/1.12.2版本
- curl 7.61.0 (x86_64-pc-linux-gnu) libcurl/7.61.0 OpenSSL/1.0.2k zlib/1.2.7

## 目录
* [概述](#概述)
* [几种可选方法介绍](#几种可选方法介绍)
* [use](#use)

# 概述
nginx支持多种连接处理IO方法，特定IO方法的可用性取决于所使用的平台。如果平台支持多种IO方法，nginx会自动选择最有效的IO方法。但是，如果需要的话，也可以通过指令use指定具体的IO方法

configure编译选项中可以选择--without-poll_module、--without-pollt_module、--without-select_module、--without-select_module使其强制使用或者禁止使用某IO模型

如果使用--without-select_module，则不可以使用use select指令了，报错提示：invalid event type "select"
  
如果使用--with-select_module，则可以使用use epoll指令，最后nginx使用的是epoll的IO模型

# 几种可选方法介绍
**select**

  标准方法。如果平台没有更有效的方法的话，则这个方法会自动被选择和构建

**poll**

  标准方法。如果平台没有更有效的方法的话，则这个方法会自动被选择和构建
  
**kqueue**
  
  优先选择的有效方法（在FreeBSD 4.1+，OpenBSD 2.9+，NetBSD 2.0和macOS平台上）
  
**epoll**

  优先选择的有效方法（在Linux 2.6+平台上）
  
 **/dev/poll**

  优先选择的有效方法（在Solaris 7 11/99+，HP/UX 11.22+ (eventport)，IRIX 6.5.15+，和Tru64 UNIX 5.1A+平台上）

 **eventport**
 
  建议使用/dev/poll替代，该IO模型用于Solaris 10+ 
  
 # use
```
  Syntax:	use method;
  Default:	—
  Context:	events
```
```
  events{
    use select;
    #use epoll;
  }
```
通常不会指定，因为nginx会默认选择最优的方案
