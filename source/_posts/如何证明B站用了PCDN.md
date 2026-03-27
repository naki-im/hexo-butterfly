---
title: 如何证明B站用了PCDN
date: 2026-03-04 22:19:16
categories:
 - 记录
tags:
 - B站
 - IPv4
 - CDN
 - PCDN
 - SSL
 - TLS
 - 互联网
---

# 0.引言

众所周知，中国大陆地区的带宽成本很高（商用宽带相对家宽而言），而流媒体平台又是流量大户，平台们看着高额流量账单，心里郁闷，就有人想出个损招：直接把用户的设备当CDN节点行不行呢？——还真行，都发展成产业了。
这个损招就是ISP闻风丧胆誓要得而诛之的**PCDN（P2P-CDN, Peer-to-Peer Content Delivery Network，点对点内容分发网络）**。

我一看视频，B站似乎有PCDN？分析一下，确实有可能。

# 1.分析视频流的URL

我在B站看视频，按开F12键调到Firefox的网络面板，看到一堆HTTP请求，全是*206 Partial Content*的响应状态码，不用想，肯定是视频流。点开一看这个URL，为什么这么怪？
![Firefox-debug](https://cdn.jsdmirror.cn/gh/naki-im/img/img/20260304225922517.png)

看一个请求，如下：

```http
GET /v1/resource/36347775999-1-30232.m4s?agrr=0&build=0&buvid=358B11A0-5640-65CA-BD1C-00D6A88F982109942infoc&bvc=vod&bw=91605&cdnid=2123&deadline=1772639389&dl=0&e=ig8euxZM2rNcNbdlhoNvNC8BqJIzNbfqXBvEqxTEto8BTrNvN0GvT90W5JZMkX_YN0MvXg8gNEV4NC8xNEV4N03eN0B5tZlqNxTEto8BTrNvNeZVuJ10Kj_g2UB02J0mN0B5tZlqNCNEto8BTrNvNC7MTX502C8f2jmMQJ6mqF2fka1mqx6gqj0eN0B599M%3D&f=u_0_0&gen=playurlv3&lrs=46&mid=1385932271&nbs=1&nettype=0&og=cos&oi=0x24098a20a84c3be444b327045f61aecc&orderid=0%2C3&os=bcache&platform=pc&qn_dyeid=95143d9c8cc5b9af0070040f69a8387d&sign=e2ddb4&traceid=trMfRpDHoftTTP_0_e_N&uipk=5&uparams=e%2Cuipk%2Ctrid%2Cdeadline%2Cgen%2Cog%2Cplatform%2Cmid%2Cnbs%2Coi%2Cos&upsig=3a86d0dba89fbe72d2ea234999b69b69 HTTP/1.1
Host: xy223x109x235x91xy.mcdn.bilivideo.cn:8082
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br, zstd, identity
Range: bytes=966822-1024147
Origin: https://www.bilibili.com
Sec-GPC: 1
Connection: keep-alive
Referer: https://www.bilibili.com/
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site
Pragma: no-cache
Cache-Control: no-cache
```

很奇怪啊，为什么这个Host带非标端口8082？

# 2.分析CDN域名对应IP地址

## 2.1 使用dig查询域名对应IP地址

看这个域名，很奇怪。
`xy223x109x235x91xy.mcdn.bilivideo.cn`这个域名看起来就有一股临时地址的味。

DNS解析一下这个域名，找到IP地址是`223.109.235.91`,看起来就有一股家宽的味（中国大陆地区家宽IP地址有相当数量的IP地址在`223.0.0.0/8`里）

````bash
naki@naki-im:~/docs/naki.im/hexo-butterfly$ dig @223.5.5.5 A xy223x109x235x91xy.mcdn.bilivideo.cn

; <<>> DiG 9.20.15-1~deb13u1-Debian <<>> @223.5.5.5 A xy223x109x235x91xy.mcdn.bilivideo.cn
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 41478
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;xy223x109x235x91xy.mcdn.bilivideo.cn. IN A

;; ANSWER SECTION:
xy223x109x235x91xy.mcdn.bilivideo.cn. 1800 IN A	223.109.235.91

;; Query time: 103 msec
;; SERVER: 223.5.5.5#53(223.5.5.5) (UDP)
;; WHEN: Wed Mar 04 22:25:09 CST 2026
;; MSG SIZE  rcvd: 81
````

等会，这域名有问题！提取数字：
`xy223x109x235x91xy.mcdn.bilivideo.cn`
数字字段分别是：`223`、 `109`、 `235`、 `91`
解析出的IP是`223.109.235.91`
破案了，域名里面藏IP！

## 2.2 在IP2Location查询IP归属

查询地址：https://www.ip2location.com/
打开查询页面，显示的信息如下：
（欲要查看，请[点击此处](https://www.ip2location.com/223.109.235.91)）

![ip2location-result](https://cdn.jsdmirror.cn/gh/naki-im/img//img/20260304231221488.png)

可以看出的信息：
归属地：江苏南京
运营商：中国移动
ASN：AS56046 China Mobile Communications Corporation
*用途：(MOB) Mobile ISP*

看：用途是移动ISP，也就是说，这些IP是分配给移动网络使用的，包括家庭宽带、4G/5G上网用户等。也就是说，这个IP是一个（或者多个）家庭用户使用的IP地址，从而可以推断，B站的这个域名的背后大概率是PCDN。
