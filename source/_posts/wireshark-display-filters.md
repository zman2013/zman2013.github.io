---
title: wireshark-display-filters
date: 2021-05-21 22:33:34
tags:
---

# 常用 filter
1. 过滤源
  ip.src_host == 127.0.0.1
2. 过滤目标
  ip.dst_host == 127.0.0.1
3. 过滤端口
  tcp.port == 443
4. 过滤flag
  tcp.flags.syn==1
5. 逻辑运算
(ip.src_host == 127.0.0.1 || ip.src_host == 127.0.0.2) && tcp.port == 443  && tcp.flags.syn==1
