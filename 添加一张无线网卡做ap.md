title: 添加一张无线网卡做AP
date: 2013-10-08 15:21:28
tags: 
    - Linux Tips
    - WiFi 
---
### 场景

刚换了办公室，没有 WiFi。加一个无线路由器、AP 都有点麻烦。找了一个旧的无线网卡安装在计算机上，准备自己做个 AP。

```
 |有线端口|==== |    台式机  +----无线网卡-++ ***WiFi*** | 手持设备 | 
```

<!--more-->


### 首先识别安装网卡

Google一下发现可用hostapd来做软ap。安装好hostapd准备配置发现没有无线网络的网络接口wlan。`lspci`可以发现网卡,但`dmesg`没有网络接口配置的信息。 原来按网络上一些贴子大都使用没有安装固件的步骤，那是因为装操作系统的时候，系统检查到了就直接安装了。在已经安装好的机器上直接插个网卡Linux不会自动把固件安装好，所以，通过lspci可以看见设备但不会自动出现什么`wlan0`网络接口。

通过`lspci` 可以看到安装的网卡的型号:
```
#lspci
..(略去)...
04:04.0 Network controller: Ralink corp. RT2561/RT61 802.11g PCI

```

显然应该去安装 Ralink 的固件。直接通过`aptitude search firmware` 可找到 `firmware-ralink` 包。
安装这个包以后`wlan0` 接口就出现了，说明设备被正确驱动！

### 用hostapd创建无线AP


照着[这篇文章](http://www.cyberciti.biz/faq/debian-ubuntu-linux-setting-wireless-access-point/)按部就班，`/etc/default/hostapd`添加一行：

```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
`hostapd.conf`的配置也很普通：

```
interface=wlan0
driver=nl80211
country_code=CN
ssid=omg-ap
hw_mode=g
channel=8
wpa=2
wpa_passphrase=mypassword
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
auth_algs=1
macaddr_acl=0
```

#### 一个坑

启动`hostapd`失败，说找不到wlan0。接口存在怎么会找不到呢？调整配置试了几次都失败！突然发现出错提示wlan0后面有空格， __难道hostapd取配置没有trim空格？__ 再看配置wlan0后面果然有个空格，果断删除，启动成功。

#### 另一个坑

`hostapd`启动起来，使用ipod搜索发现AP、连接、输入密码，认证失败！

查看日志

```
Oct  8 16:31:07 omg hostapd: wlan0: STA f4:1b:a1:5f:b3:cd IEEE 802.11: authenticated
Oct  8 16:31:07 omg hostapd: wlan0: STA f4:1b:a1:5f:b3:cd IEEE 802.11: associated (aid 1)
Oct  8 16:31:16 omg hostapd: wlan0: STA f4:1b:a1:5f:b3:cd IEEE 802.11: deauthenticated due to local deauth request
```

Google半天发现这个[贴子]（https://bbs.archlinux.org/viewtopic.php?id=151342），试试运行hostapd时候需要添加参数 `-e /dev/urandom`，认证成功！

于是在`/etc/default/hostapd`添加一行：

```
DAEMON_OPTS="-e /dev/urandom"
```


### 桥模式，还是NAT

_桥模式_简单的理解就是将有线信号变成无线信号，纯二层环境，台式机还要接入网络工作，所以桥模式只有放弃了。直接NAT了，DHCP也不做了，手动配置吧。

```
# echo 1 >/proc/sys/net/ipv4/ip_forward
# iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

iptables 真是方便啊，iPod可以上网了！
