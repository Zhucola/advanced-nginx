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

nginx支持多种连接处理IO方法，特定IO方法的可用性取决于所使用的平台。如果平台支持多种IO方法，nginx会自动选择最有效的IO方法。但是，如果需要的话，也可以通过指令指定具体的IO方法

**select**

  标准方法。如果平台没有更有效的方法的话，则这个方法会自动被选择和构建
