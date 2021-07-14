# 自建Bitwarden / Vaultwarden

## 教程

### 必备组建

- [github.com/fatedier/frp](https://github.com/fatedier/frp) - 内网穿透

- vaultwarden / bitwarden



### 安装

Docker-compose安装Bitwarden

```bash
version: '3'
services:
  vaultwarden:
    image: vaultwarden/server:latest
    restart: always
    ports:
      - "3080:80"
      - "3012:3012"
    environment:
      DOMAIN: 'https://bitwarden.haha.ai/'
      SIGNUPS_ALLOWED: 'true'
      WEBSOCKET_ENABLED: 'true'
    volumes:
      - ./data:/data
```



## 参考资料

- [使用WireGuard搭建私有网络](https://blog.gimo.me/post/setup-wireguard-vpn/)
- [自建 vaultwarden(a.k.a bitwarden_rs)](https://blog.gimo.me/post/self-host-vaultwarden/)

- [使用群晖搭建第三方 Bitwarden 密码服务器](https://ppgg.in/blog/10271.html)
  - 介绍群晖如何使用增量备份：[Hyper Backup套件](https://www.synology.com/zh-cn/knowledgebase/DSM/tutorial/Backup/How_to_back_up_your_data_to_cloud_services_with_Hyper_Backup)
- [群晖Docker + Caddy](https://github.com/sosandroid/docker-bitwarden_rs-caddy-synology)

