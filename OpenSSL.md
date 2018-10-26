
联系作者
- weixin: 363260961
- QQ: 363260961

**编写不易，转载请注明出处，谢谢**

使用版本
- CentOS Linux release 7.5.1804 (Core)
- nginx/1.12.2版本
- curl 7.61.0 (x86_64-pc-linux-gnu) libcurl/7.61.0 OpenSSL/1.0.2k zlib/1.2.7
- OpenSSL 1.0.2p  14 Aug 2018
## 目录
* [随机数](#随机数)
* [秘钥RSA](#秘钥RSA)
  * [查看RSA秘钥公私钥、结构](#查看RSA秘钥公私钥、结构)
  * [查看RSA秘钥是否被篡改](#查看RSA秘钥是否被篡改)
 * [秘钥DSA](#秘钥DSA)
    * [查看DSA秘钥公私钥、结构](#查看DSA秘钥公私钥、结构)
# 随机数
openssl rand ...
- \-out file  指定随机数输出文件
- \-rand file 指定随机数种子文件 
- \-base64    编码为base64
- \-hex       编码为hex
- num         随机数长度
```openssl
  openssl rand 30
  openssl rand -rand /root/.rnd 30
```
输出结果为二级制
```openssl
  openssl rand -hex 30
  openssl rand -base64 30
```
输出结果为不二级制

# 秘钥RSA
openssl genrsa用于生成RSA私钥，**不会生成公钥，因为公钥提取自私钥**

openssl genrsa 
- \-out filename 将指定的秘钥保存至filenme文件，若未指定则为标准输出
- numbites 秘钥长度，该项必须为命令行最后一项
- \-des等  加密私钥的算法
- \-passout args 加密私钥文件的密码

密码是可选项，但是强烈推荐使用密码。密码可以安全的存储、传输和备份受保护的秘钥。但是可能每次重启web服务器都需要重新输入密码。如果需要更加安全，建议投资硬件
```openssl
  openssl genrsa –aes128 –out a.key 2048
  openssl genrsa -des3 -out a.key  -passout pass:1234 2048
```
# 查看RSA秘钥公私钥、结构
```openssl
  openssl genrsa -des3 -out a.key  -passout pass:1234 2048
  openssl rsa -text -in a.key    # 查看结构
  openssl rsa -in a.key -pubout -out a_pub.key -passin pass:1234  #查看公钥
  openssl rsa -in a.key -out a_priv.key   #查看未加密的私钥
```
# 查看RSA秘钥是否被篡改
```openssl
  openssl genrsa -des3 -out a.key  -passout pass:1234 2048
  openssl rsa -in a.key -check -passin pass:1234
```
# 秘钥DSA
DSA秘钥生成分为两步，1.生成DSA参数2.秘钥创建
```openssl
  openssl dsaparam -genkey 2048 | openssl dsa -out d.key -aes128
```
# 查看DSA秘钥公私钥、结构
```openssl
  openssl dsaparam -genkey 2048 | openssl dsa -out d.key -aes128
  openssl dsa -text -in d.key  # 查看结构
  openssl dsa -in d.key -pubout -out d_pub.key  #查看公钥
  openssl dsa -in d.key -out d_prive.key  #查看私钥
  
```
