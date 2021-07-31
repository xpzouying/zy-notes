# Nginx 配置文件

## 配置1 - xray回落使用

特色：

1. listen unix文件，提供回落使用。

```
server {
    listen 80 reuseport default_server;
    listen [::]:80 reuseport default_server;
    return 301 https://demo.example.com;
}
server {
    listen 80;
    listen [::]:80;
    server_name demo.example.com;
    return 301 https://$host$request_uri;
}
server {
    listen unix:/dev/shm/nginx_unixsocket/default.sock default_server;
    listen unix:/dev/shm/nginx_unixsocket/h2.sock http2 default_server;
    return 301 https://demo.example.com;
}
server {
    listen unix:/dev/shm/nginx_unixsocket/default.sock;
    listen unix:/dev/shm/nginx_unixsocket/h2.sock http2;
    server_name demo.example.com;
    add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; preload" always;
    location / {
        proxy_set_header X-Forwarded-For 127.0.0.1;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://unix:/dev/shm/cloudreve_unixsocket/cloudreve.sock;
        client_max_body_size 0;
    }
}
```

对应的xray配置。配置文件中设置`fallbacks`，如果不是xray协议，那么直接进入到回落的配置。

结合上面的nginx配置，走domain socket连接。

```json
{
    "log": {
        "loglevel": "none"
    },
    "inbounds": [
        {
            "port": 443,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "12345-1234-1234-1234-0123456789",
                        "flow": "xtls-rprx-direct"
                    }
                ],
                "decryption": "none",
                "fallbacks": [
                    {
                        "alpn": "h2",
                        "dest": "/dev/shm/nginx_unixsocket/h2.sock"
                    },
                    {
                        "dest": "/dev/shm/nginx_unixsocket/default.sock"
                    }
                ]
            },
            "streamSettings": {
                "network": "tcp",
                "security": "xtls",
                "xtlsSettings": {
                    "alpn": [
                        "h2",
                        "http/1.1"
                    ],
                    "minVersion": "1.2",
                    "cipherSuites": "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256:TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384:TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256",
                    "certificates": [
                        {
                            "certificateFile": "/usr/local/nginx/certs/example.cer",
                            "keyFile": "/usr/local/nginx/certs/example.key",
                            "ocspStapling": 3600
                        }
                    ]
                }
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom"
        }
    ]
}
```