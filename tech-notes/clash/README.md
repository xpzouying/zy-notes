# Clash旁路由

之前一直使用OpenWRT，但是由于：

1. OpenWRT版本众多，有很多版本中可能被人植入一些乱七八糟的东西。

2. OpenWRT里面功能太多太杂，几乎大部分功能我本人不需要，所以准备尝试直接使用Clash。

下列简单介绍一下在局域网内如何使用`Clash Premium`搭建家庭科学上网。


## 准备

需要下列前提：

- 家里一台VM。我使用Esxi安装了一个Ubuntu作为Clash运行的机器。

- 下载[Clash Premium](https://github.com/Dreamacro/clash/releases/tag/premium)。

- [Country.mmdb](https://github.com/Dreamacro/maxmind-geoip/releases/latest/download/Country.mmdb) - IP地址数据库。

- `config.yaml` - Clash的配置文件。


上列工作准备好后，将：clash、config.yaml、Country.mmdb放在一个目录下，我放在了`~/clash/`下，


## 运行

进入到Clash文件夹中，启动。

```bash
./clash -d . 
```

## 配置（新）

参考这里：[Clash-Linux](https://github.com/yuanlam/Clash-Linux)


1、首先关闭ubuntu server的自身的dns

```bash
sudo vim /etc/systemd/resolved.conf

# 设置为no，其他的不变
DNSStubListener=no

# 重启
reboot
```


2、配置iptables


我的ip地址是：`192.168.50.8`

```bash
#!/bin/bash

# 修改成自己的IP地址
ClashHost=192.168.50.8

iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

iptables -t nat -N clash
iptables -t nat -N clash_dns

iptables -t nat -A PREROUTING -p tcp --dport 53 -d 198.18.0.0/24 -j clash_dns
iptables -t nat -A PREROUTING -p udp --dport 53 -d 198.18.0.0/24 -j clash_dns
iptables -t nat -A PREROUTING -p tcp -j clash

iptables -t nat -A clash_dns -p udp --dport 53 -d 198.18.0.0/24 -j DNAT --to-destination ${ClashHost}:53
iptables -t nat -A clash_dns -p tcp --dport 53 -d 198.18.0.0/24 -j DNAT --to-destination ${ClashHost}:53

iptables -t nat -A clash -d 1.0.0.0/8 -j ACCEPT
iptables -t nat -A clash -d 10.0.0.0/8 -j ACCEPT
iptables -t nat -A clash -d 100.64.0.0/10 -j ACCEPT
iptables -t nat -A clash -d 127.0.0.0/8 -j ACCEPT
iptables -t nat -A clash -d 169.254.0.0/16 -j ACCEPT
iptables -t nat -A clash -d 172.16.0.0/12 -j ACCEPT
iptables -t nat -A clash -d 192.168.0.0/16 -j ACCEPT
iptables -t nat -A clash -d 224.0.0.0/4 -j ACCEPT
iptables -t nat -A clash -d 240.0.0.0/4 -j ACCEPT
iptables -t nat -A clash -d 192.168.1.2/32 -j ACCEPT

# iptables -t nat -A clash -p tcp --dport 22 -d ${ClashHost}/32 -j ACCEPT

iptables -t nat -A clash -p tcp -j REDIRECT --to-ports 7892
```


## 设置开机启动

1、创建service文件

```bash
sudo vim /etc/systemd/system/clash.service
```

2、内容如下

注意需要root权限运行。

```
[Unit]
Description=clash daemon

[Service]
Type=simple
User=root
ExecStart=/home/zy/clash/clash -d /home/zy/clash
Restart=on-failure

[Install]
WantedBy=multi-user.target
```


3、重新加载

```bash
sudo systemctl daemon-reload

sudo systemctl start clash.service

# 开机自启动
sudo systemctl enable clash.service
```


4、运维

```bash

# 查看日志
sudo journalctl -u clash.service -f
```


## 开机加载iptables规则

```bash
# 保存规则
sudo iptables-save > /etc/iptables.rules

```


## 设置家庭其他机器网络

- 网关/Gateway - 设置为clash的ip

- DNS - 设置为：198.18.0.1。clash配置文件中的配置，会走`fake-ip`策略。


## 配置 （旧）

按照[haoel陈皓老师的配置](https://github.com/haoel/haoel.github.io#74-%E8%AE%BE%E7%BD%AE-iptables-%E8%BD%AC%E5%8F%91)，设置iptables。

iptables配置贴到下面：

```
iptables -t nat -N CLASH
iptables -t nat -A CLASH -d 10.0.0.0/8 -j RETURN
iptables -t nat -A CLASH -d 127.0.0.0/8 -j RETURN
iptables -t nat -A CLASH -d 169.254.0.0/16 -j RETURN
iptables -t nat -A CLASH -d 172.16.0.0/12 -j RETURN
iptables -t nat -A CLASH -d 192.168.0.0/16 -j RETURN
iptables -t nat -A CLASH -d 224.0.0.0/4 -j RETURN
iptables -t nat -A CLASH -d 240.0.0.0/4 -j RETURN
iptables -t nat -A CLASH -p tcp -j REDIRECT --to-ports 7892
```

这样，在clash本机就可以请求外网了。


**局域网请求配置**

配置局域网内的其他设备通过该机器上网。（以本人的Macbook为例）

配置：IP地址的`网关`和`DNS Server`为：clash所在机器的IP地址。

此时，会发现Clash日志中，并没有接收到局域网的请求。需要进行下列iptables设置：

```bash
# 拦截外部tcp数据并交给clash规则链处理
iptables -t nat -A PREROUTING -p tcp -j CLASH
```

路由表每次开机都会恢复到默认值，可以使用下列进行持久化：

```bash
sudo apt install iptables-persistent

sudo dpkg-reconfigure iptables-persistent
```


## 路由器恢复

```bash
iptables -t nat -D PREROUTING -p tcp -j CLASH
iptables -t nat -D OUTPUT -p udp --dport 53 -j CLASH_DNS
iptables -t nat -D PREROUTING -p udp --dport 53 -j CLASH_DNS
iptables -t nat -F CLASH
iptables -t nat -X CLASH
iptables -t nat -F CLASH
iptables -t nat -X CLASH_DNS
```


参考资料：

- [左耳朵](https://github.com/haoel/haoel.github.io)

- [Clash-Linux](https://github.com/yuanlam/Clash-Linux) - Linux上面如何部署Clash。

- [Docker+Clash 部署透明“网关”的实现](https://zhuanlan.zhihu.com/p/423684520)