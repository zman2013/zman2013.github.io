---
title: Wireshark解密TLS报文
tags:
  - decrypt
  - tcp
  - tcpdump
  - wireshark
id: '304'
categories:
  - - tcp
date: 2019-05-01 21:51:57
---

# 解密TLS报文

## 1. 预备知识

1.  拥有服务器端证书对应的私钥
2.  加密算法不能使DHE、ECDHE
3.  报文必须完整，从TCP握手开始

## 2. 操作步骤

### 2.1 抓取报文

#### 2.1.1 Linux服务器上抓取tcp报文
```shell
tcpdump -i eth0  port 10176 -w server.cap -C 50
# -i 指定网卡名
# -w 输出到文件 
# -C 文件分割大小，单位M
# port 指定监听端口
```

#### 2.1.2 Android上抓取tcp报文（如果无特殊需要抓一方的报文就够了）
```shell
tcpdump -i any -p port 8883 -vv -s 0 -w sdcard/client.pcap
# -i   指定网卡名
# -w   输出到文件 
# -C   文件分割大小，单位M
# port 指定监听端口
# -vv  输出详细报文
# -s 0 不截取数据包
```

### 2.2 设置Wireshark（v3.0.1）

#### 2.2.1 添加证书私钥

如果是nginx做的反向代理，一般证书会配置在nginx这一层。 Preferences -> Protocols -> TLS 
[![wireshark_tls](./Screen-Shot-2019-05-01-at-8.56.23-PM-750x410.png)](./Screen-Shot-2019-05-01-at-8.56.23-PM.png)

添加私钥参数 IP Address, 服务器IP Port, 服务器端口 Protocol http（如果是http：小写，非https, Key File 选择key文件，格式pem       
**注意**：私钥格式为pem，样式如下： 
```
-----BEGIN RSA PRIVATE KEY----- 
... 
-----END RSA PRIVATE KEY-----
```

#### 2.2.2 使用Wireshark打开.pcap文件

如果tcp报文完整（从tcp握手开始），并且私钥正确，而且加密算法不是DHE/ECDHE，你应该已经看到解密后的内容了。 

# 排查网络丢包、异常数据

_有时遇到网络丢包的情况，也可能遇到client收到server异常数据的情况，两者的处理方式是一样的_

## 1. 禁止Wireshark重排TCP seq和ack

Preferences -> Protocols -> TLS [![wireshadow_tcp](./Screen-Shot-2019-05-01-at-9.17.41-PM-750x410.png)](./Screen-Shot-2019-05-01-at-9.17.41-PM.png)

## 2. client和server相同时间段内抓包

## 3. 对比client和server的报文

1.  在一端定位到异常报文，获取tcp对应seq
2.  在另一端查询是否有相同seq的数据且时间相同

## 4. Go on

剩下的就是根据报文内容具体分析了：比如会不会是连接超时，http2 server主动发送了goaway，然后断开了连接。 -

# Q&A

Q1：如何判断当前TLS使用的加密算法 
A：在TLS的Server Hello报文中查Cipher Suite，如下图 [![tls_cipher_suite](./Screen-Shot-2019-05-01-at-9.29.36-PM-750x410.png)](./Screen-Shot-2019-05-01-at-9.29.36-PM.png) 

Q2：如何指定TLS使用的加密算法 
A1：客户端指定，客户端可以通过参数的方式设定，以okhttpclient为例
```java
OkHttpClient.Builder okHttpClientBuilder = new OkHttpClient.Builder();
ConnectionSpec spec = new ConnectionSpec.Builder(ConnectionSpec.MODERN\_TLS)
                .tlsVersions(TlsVersion.TLS\_1\_2)
                .cipherSuites(
                        CipherSuite.TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256,
                        CipherSuite.TLS\_RSA\_WITH\_AES\_128\_CBC\_SHA,
                        CipherSuite.TLS\_RSA\_WITH\_AES\_256\_CBC\_SHA,
                        CipherSuite.TLS\_RSA\_WITH\_3DES\_EDE\_CBC\_SHA)
                .build();
        okHttpClientBuilder.connectionSpecs(Collections.singletonList(spec));

OkHttpClient okHttpClient = okHttpClientBuilder.build();
```

A2：服务端通过配置的方式设定，以nginx为例
```
ssl\_prefer\_server\_ciphers on;
ssl\_ciphers AES128-GCM-SHA256;
```

Q3：nginx http2\_module连接空闲超时时间是多少？ 
A：3min 

Q4：如何判断wireshark解密成功了？ 
A：观察是否http请求的url已经变成明文；http payload也是明文了（界面下方可以切换tab） 

Q5：DHE算法为什么不能解密？ 
A：DHE算法符合完全正向保密协议，中间人即使得到私钥也破解不了密文。 


# Links：
1.  [tcpdump command](https://www.tcpdump.org/manpages/tcpdump.1.html)
2.  [https://packetpushers.net/using-wireshark-to-decode-ssltls-packets/](https://packetpushers.net/using-wireshark-to-decode-ssltls-packets/)
3.  [openssl ciphers](https://www.openssl.org/docs/man1.0.2/man1/ciphers.html)
4.  [tcp status](https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.1.0/com.ibm.zos.v2r1.halu101/constatus.htm)
5.  [http2 goaway](https://http2.github.io/http2-spec/#GOAWAY)
6.  [nginx http2 module](http://nginx.org/en/docs/http/ngx_http_v2_module.html#http2_idle_timeout)