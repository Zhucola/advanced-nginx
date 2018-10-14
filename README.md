base-nginx
=======================

一些nginx的基础操作与总结

## 目录
* [nginx如何处理一个请求](#nginx如何处理一个请求)


# nginx如何处理一个请求

nginx首先选定由那一个虚拟主机来处理请求

`
  server {
  
    listen 80;
    
    server_name example.org www.example.org;
  }
`
