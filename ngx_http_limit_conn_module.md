
联系作者
- weixin: 363260961
- QQ: 363260961

**编写不易，转载请注明出处https://github.com/Zhucola/advanced-nginx**

使用版本
- CentOS Linux release 7.5.1804 (Core)
- nginx/1.12.2版本
- curl 7.61.0 (x86_64-pc-linux-gnu) libcurl/7.61.0 OpenSSL/1.0.2k zlib/1.2.7

对于一些服务器流量异常、负载过大，甚至是大流量的恶意攻击访问等，进行并发数的限制；该模块可以根据定义的键来限制每个键值的连接数，只有那些正在被处理的请求（这些请求的头信息已被完全读入）所在的连接才会被计数。
## 目录
* [nginx如何处理一个请求](#nginx如何处理一个请求)
