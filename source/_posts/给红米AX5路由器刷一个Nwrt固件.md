---
title: 给红米AX5路由器刷一个Nwrt固件
categories:
  - 教程
  - 记录
tags:
  - IPv4
  - IPv6
  - 互联网
  - 刷机
  - 路由器
abbrlink: 58bdc802
date: 2026-01-17 18:48:32
---
# 0.前言
红米AX5路由器的内存只有256MiB，网上的其他教程都说只能刷精简版的固件用不了WiFi，我不信这么大内存（嵌入式设备上256M算大的）还跑不起来一个OpenWrt固件，在某只谈技术莫论政事的著名中国网络论坛（指恩山论坛）查找资料，找到Nwrt（基于OpenWrt的衍生开源固件）固件可用，刷机成功，大喜，遂作此文。

## 0.1 参考资料
[恩山论坛](https://www.right.com.cn/forum/thread-8278695-1-1.html)

[Github](https://github.com/janeblower/redmi_ax1800-ax5-ra67)

注：本教程未提到备份相关操作，请自行备份原厂固件，刷入Uboot和大分区后小米官方救砖工具也无能为力，一定要备份！

# 1.降级AX5的固件
对路由器进行刷机需要通过SSH刷入Uboot和大分区，而红米官方不支持直接开启SSH，我们需要通过原厂固件早期版本的漏洞来开启，将系统固件版本降级到1.0.26
操作：
1. **进入后台管理界面选择系统升级**  
2. 选择降级的固件1.0.26  
3. **等待ax5降级完成后进入管理界面，然后复制你的stok**
4. 填入到下面的链接里的STOK里面（记得删掉<>）
```plaintext
http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20nvram%20set%20ssh_en%3D1%3B%20nvram%20commit%3B%20sed%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%5C%22debug%5C%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%3B%20%2Fetc%2Finit.d%2Fdropbear%20start%3B
```
此时，浏览器显示  {"code":0}  即代表成功
然后修改ssh密码，要求同上，链接如下：
```plaintext
http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20echo%20-e%20'admin%5Cnadmin'%20%7C%20passwd%20root%3B
```
此时，打开你的终端，输入`ssh root@192.168.31.1`连接到路由器，密码是`admin`，显示ARE U OK的欢迎界面即为连接成功（如果你是Windows用户，请自行谷歌下载SSH连接工具）

# 2.刷入Uboot和过渡固件

## 2.1 刷入Uboot

用SCP协议把AX5_UBoot.bin上传到路由器的 /tmp 目录下，在本机终端执行，命令如下：
```bash
scp -O AX5_UBoot.bin root@192.168.31.1:/tmp/
```
上传成功后，在ssh终端窗口里输入这行命令执行
```bash
mtd write /tmp/AX5_UBoot.bin /dev/mtd7
```
此时，终端会输出如下信息且刷写是秒完成
```bash
root@AX5:/tmp$ mtd write AX5_UBoot.bin /dev/mtd7
Unlocking /dev/mtd7 …
Writing from AX5_UBoot.bin to /dev/mtd7 …
```

## 2.2 刷入大分区

同前文传入Uboot，用SCP传输大分区模块，执行命令上传文件

```bash
scp -O AX5_UBoot.bin root@192.168.31.1:/tmp/
```

然后在SSH终端敲下刷写命令：

```bash
mtd write /tmp/AX5_MIBIB.bin /dev/mtd1
```

这个刷入也是瞬时完成。

## 2.3 刷入过渡固件

>此处直接复制原教程文本：
>1. 进入Uboot后台方法：先把路由器断电，推荐直接拔路由器屁股的电源口而不是插座，然后用牙签或者其他工具按住路由器后面的RESET不要放开，然后插上电源，此时路由器会黄灯闪烁5次变为蓝灯，然后就松开RESET(松开牙签)。
>2. 准备根网线接到路由器的LAN口和电脑的网口，设置电脑的IP为192.168.1.X（随意不能是1）。==从此处开始到结束，网线要一直插着。==
>3. 在浏览器输入 192.168.1.1 进入Uboot。
>4. 然后选文件过渡固件（例如我选的 `openwrt-ipq60xx-Redmi_AX5-squashfs-nand-factory.ubi` ）
>5. 点击Update firmware等待写入。路由器指示灯开始闪烁，直到不闪烁亮蓝灯为止。
>
>来自参考资料

进入Uboot界面后，网页会显示如下界面：
![redmi-ax5-uboot](https://static.987632.xyz/img/20260117192819542.jpg)

选择过渡固件上传后，网页变为如下界面，稍等一两分钟，刷好会自动重启
![redmi-ax5-writing-transitional-firmware](https://static.987632.xyz/img/20260117192944520.jpeg)

过渡固件刷完重新打开页面的图像：
![redmi-ax5-transitional-firmware-ui-1536x668](https://static.987632.xyz/img/20260117193034548.jpg)

如果上面第五步操作之后路由器一直不亮蓝灯，则表明过渡固件写入失败，需要重新从第一步（路由器断电）开始操作。

## 2.4 升级 OpenWrt 为最终固件

操作方法就像参考资料里说的，在 192.168.1.1 的 web 界面上进行升级操作，点击按钮，上传最终固件，点击执行，等待路由器变蓝灯就好了。

>  这里我个人建议用 Chrome 浏览器，其他浏览器可能出现各种意想不到的问题，比如Firefox自动升级到HTTPS导致无法访问、360浏览器网页莫名其妙“走丢”、Safari浏览器不明所以的失败之类……烦，很烦，非常烦（恼）

### 2.4.0 升级需要多久？

大约半分钟到一分钟。如果 1 分钟之后路由器仍然不亮蓝灯，基本可以判断为升级失败，需要重新从 Uboot 界面那一步开始操作。

### 2.4.1 选什么固件？

放心，坑我都踩过，固件我都整理好了，折腾两周才搞好，链接中国大陆可用，你下载下来尽管刷就是了，不要恼。

- 大分区模块点此下载：https://static.987632.xyz/AX5_MIBIB.bin
- Uboot模块点此下载：https://static.987632.xyz/AX5_UBoot.bin
- 过渡固件下载：https://static.987632.xyz/transitional-firmware.ubi
- 最终固件下载：https://static.987632.xyz/Nwrt-firmware.bin

此处放一个最终固件登录页面的截图，初始用户是`root`，初始密码是`password`，不必多方打探。
![final-firmware](https://static.987632.xyz/img/20260117195242201.png)

SSH登录的欢迎界面如下图
![ssh-welcome](https://static.987632.xyz/img/20260117195412143.png)

这个固件因为内核新，很多东西操作起来方便。例如 SSH 自带支持 ed25519，不用额外添加 RSA 了。软件包可以自己按需删减和添加，扩展性很好。
