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


## 配置

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

Clash接收DNS的处理。（假设clash dns配置监听了1053）

```bash
#在nat表中新建一个clash_dns规则链
iptables -t nat -N CLASH_DNS
#清空clash_dns规则链
iptables -t nat -F CLASH_DNS
#重定向udp流量到本机1053端口
iptables -t nat -A CLASH_DNS -p udp -j REDIRECT --to-port 1053
#抓取本机产生的53端口流量交给clash_dns规则链处理
iptables -t nat -I OUTPUT -p udp --dport 53 -j CLASH_DNS
#拦截外部upd的53端口流量交给clash_dns规则链处理
iptables -t nat -I PREROUTING -p udp --dport 53 -j CLASH_DNS
```


参考资料：

- [左耳朵](https://github.com/haoel/haoel.github.io)

- [Docker+Clash 部署透明“网关”的实现](https://zhuanlan.zhihu.com/p/423684520)