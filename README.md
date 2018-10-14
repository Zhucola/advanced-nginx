base-nginx
=======================

一些nginx的基础操作与总结

使用nginx/1.12.2版本

## 目录
* [nginx如何处理一个请求](#nginx如何处理一个请求)


# nginx如何处理一个请求

nginx首先选定由那一个虚拟主机来处理请求
```nginx
server {
    listen 80;
    server_name a.org www.b.org;
    ...
}
server {
    listen 80;
    server_name b.org www.b.org;
    ...
}
```
在这个配置中，nginx仅仅检查请求的host头以决定该请求由那个虚拟主机来处理，如果host头没有匹配任意一个虚拟主机，或者请求中根本没有包含host头，那nginx会将请求分发到定义在此端口上的默认虚拟主机，第一个被列出的虚拟主机就是nginx的默认虚拟主机。也可以显式的某个虚拟主机为默认虚拟主机(default_server从0.8.21开始使用，在以前版本使用default代替)
