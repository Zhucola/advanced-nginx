## 目录
* [随机数](#随机数)

# 随机数
openssl rand ...
- \-out file 指定随机数输出文件
- \-rand file 指定随机数种子文件 
- \-base64  编码为base64
- \-hex  编码为hex
- num  随机数长度
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
