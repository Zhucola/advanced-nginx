
## 目录
* [ngx_http_core_module.md](https://github.com/Zhucola/advanced-nginx/blob/master/ngx_http_core_module.md)

    一些nginx的基本命令，入门nginx的基本操作应该从此开始
* [OpenSSL.md](https://github.com/Zhucola/advanced-nginx/blob/master/OpenSSL.md)

    OpenSSL入门，ca证书搭建，openssl.cnf配置文件详解
* [ngx_http_ssl_module.md](https://github.com/Zhucola/advanced-nginx/blob/master/ngx_http_ssl_module.md)

    nginx的https配置，查看此目录应该有一定的openssl、https协议基础

* [连接处理方法IO.md](https://github.com/Zhucola/advanced-nginx/blob/master/连接处理方法IO.md)

    nginx的IO 模型简单介绍，仅仅介绍如何设置、切换，网络IO模型详细讲解不在本次nginx总结中
    
* [日志管理.md](https://github.com/Zhucola/advanced-nginx/blob/master/日志管理.md)

    nginx的日志管理，涉及error、access日志、日志结构等
    
* [命令行与热部署操作.md](https://github.com/Zhucola/advanced-nginx/blob/master/命令行与热部署操作.md)

    nginx的信号管理（如平滑重启、启动、停止、重新创建日志文件）和热部署操作
    
* [ngx_http_limit_conn_module.md](https://github.com/Zhucola/advanced-nginx/blob/master/ngx_http_limit_conn_module.md)

    直接使用nginx进行并发限流，如以$uri为键，限制相同请求$uri的并发数量为100，在请求超限下返回自定义http_code，或者配合error_page指定来做进一步超限处理，可以将并发限流的操作从后端服务器转移至nginx，缓解后端服务器压力
