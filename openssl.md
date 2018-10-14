## 目录
* [随机数](#随机数)
* [秘钥RSA](#秘钥RSA)
  * [查看RSA秘钥公私钥、结构](#查看RSA秘钥公私钥、结构)
  * [查看RSA秘钥是否被篡改](#查看RSA秘钥是否被篡改)
 * [秘钥DSA](#秘钥DSA)
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

openssl genrsa ...
\ -\out filename 将指定的秘钥保存至filenme文件，若未指定则为标准输出
\ numbites 秘钥长度，该项必须为命令行最后一项
\ \-des等  加密私钥的算法
\ \-passout args 加密私钥文件的密码

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
