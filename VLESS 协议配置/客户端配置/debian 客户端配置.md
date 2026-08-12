# debian 客户端配置

基于 Debian 系统配置 sing-box 客户端，通过 VLESS 协议建立代理隧道连接，并使用 IPv6 优先出口

仅指定域名、指定IP、私有IP走本地出口，其余默认走代理出口

# 准备 sing-box 配置参数

- `UUID`：服务端配置的 UUID
- `服务器 IP/域名`：服务器的 IPv4 地址、IPv6 地址或域名，三者任选其一
- `public_key`：服务端生成的 Reality 公钥
- `short_id`：服务端配置的 Short ID

# 建立直连规则文件

创建统一管理的文件夹：

```shell
mkdir -p /etc/sing-box/rules
```

## 建立域名直连规则文件

编写域名直连规则文件：

```shell
vim /etc/sing-box/rules/direct-domains.json
```

域名直连规则文件内容（自动匹配子域名）：

```json
{
  "version": 3,
  "rules": [
    {
      "domain_suffix": [
        "bilibili.com",
        "hdslb.com",
        "bilivideo.com",
        "bilivideo.cn",
        "b23.tv"
      ]
    },
    {
      "domain_suffix": [
        "aliyun.com",
        "aliyuncs.com",
        "alibabacloud.com",
        "aliyuncdn.com",
        "alicdn.com"
      ]
    }
  ]
}
```

## 建立 ip 直连规则文件

编写 ip 直连规则文件：

```shell
vim /etc/sing-box/rules/direct-ips.json
```

ip 直连规则文件内容：

```json
{
  "version": 3,
  "rules": [
    {
      "ip_cidr": [
        "<IPv4 地址>/32"
      ]
    },
    {
      "ip_cidr": [
        "<IPv6 地址>/128"
      ]
    }
  ]
}
```

# 编辑 sing-box 配置

编辑 sing-box 配置文件：

```shell
vim /etc/sing-box/config.json
```

> 先清空文件内容

sing-box 配置内容：

```json
{
  "log": {
    "level": "warn",
    "timestamp": true
  },
  "dns": {
    "servers": [
      {
        "tag": "ali-dns",
        "type": "udp",
        "server": "223.5.5.5"
      },
      {
        "tag": "google-dns",
        "type": "tls",
        "server": "8.8.8.8",
        "detour": "proxy"
      }
    ],
    "rules": [
      {
        "rule_set": "direct-domains",
        "server": "ali-dns"
      }
    ],
    "final": "google-dns",
    "strategy": "prefer_ipv6"
  },
  "inbounds": [
    {
      "type": "tun",
      "tag": "tun-in",
      "address": [
        "10.255.255.1/30",
        "fdfe:dcba:9876::1/126"
      ],
      "auto_route": true,
      "auto_redirect": true,
      "strict_route": true,
      "mtu": 1480,
      "stack": "system"
    }
  ],
  "outbounds": [
    {
      "type": "vless",
      "tag": "proxy",
      "server": "<替换服务器 IP/域名>",
      "server_port": 443,
      "domain_resolver": "ali-dns",
      "uuid": "<替换 UUID>",
      "flow": "xtls-rprx-vision",
      "packet_encoding": "xudp",
      "tcp_fast_open": true,
      "tls": {
        "enabled": true,
        "server_name": "www.apple.com",
        "utls": {
          "enabled": true,
          "fingerprint": "chrome"
        },
        "reality": {
          "enabled": true,
          "public_key": "<替换 public_key>",
          "short_id": "<替换 short_id>"
        }
      }
    },
    {
      "type": "direct",
      "tag": "direct"
    }
  ],
  "route": {
    "default_domain_resolver": "ali-dns",
    "rule_set": [
      {
        "tag": "direct-domains",
        "type": "local",
        "format": "source",
        "path": "/etc/sing-box/rules/direct-domains.json"
      },
      {
        "tag": "direct-ips",
        "type": "local",
        "format": "source",
        "path": "/etc/sing-box/rules/direct-ips.json"
      }
    ],
    "rules": [
      {
        "action": "sniff"
      },
      {
        "protocol": "dns",
        "action": "hijack-dns"
      },
      {
        "network": "icmp",
        "outbound": "direct"
      },
      {
        "rule_set": "direct-domains",
        "outbound": "direct"
      },
      {
        "rule_set": "direct-ips",
        "outbound": "direct"
      },
      {
        "ip_is_private": true,
        "outbound": "direct"
      }
    ],
    "final": "proxy",
    "auto_detect_interface": true
  }
}
```

语法检查：

```shell
sing-box check -c /etc/sing-box/config.json
```

> 没有输出说明语法通过

# 应用 sing-box 配置

重启 sing-box：

```shell
systemctl restart sing-box
```

实时查看 sing-box 服务日志：

```shell
journalctl -u sing-box -f
```

查看是否应用成功：

```shell
curl -6 ping0.cc

curl -4 ping0.cc
```

> 会返回服务器的 IPv6 和 IPv4


