# debian 服务端配置

基于 Debian 系统配置 sing-box 服务端，提供 VLESS 协议代理隧道连接，并支持 IPv6 客户端连接和 IPv6 优先出口

如果使用云服务器，请根据云厂商提供的网络访问控制功能，放行 IPv4 和 IPv6 的 TCP 443 端口

# 准备 sing-box 配置参数

UUID 生成：

```shell
sing-box generate uuid
```

Reality 密钥对生成：

```shell
sing-box generate reality-keypair
```

Short ID 生成：

```shell
openssl rand -hex 8
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
        "tag": "dns-google",
        "type": "tls",
        "server": "8.8.8.8"
      }
    ],
    "strategy": "prefer_ipv6",
    "final": "dns-google"
  },
  "inbounds": [
    {
      "type": "vless",
      "tag": "vless-reality",
      "listen": "::",
      "listen_port": 443,
      "users": [
        {
          "name": "<替换 name>",
          "uuid": "<替换 uuid>",
          "flow": "xtls-rprx-vision"
        }
      ],
      "tls": {
        "enabled": true,
        "server_name": "www.apple.com",
        "reality": {
          "enabled": true,
          "handshake": {
            "server": "www.apple.com",
            "server_port": 443
          },
          "private_key": "<替换 private_key>",
          "short_id": [
            "<替换 short_id>"
          ]
        }
      }
    }
  ],
  "outbounds": [
    {
      "type": "direct",
      "tag": "direct"
    }
  ],
  "route": {
    "default_domain_resolver": {
      "server": "dns-google",
      "strategy": "prefer_ipv6"
    },
    "final": "direct"
  }
}
```

`log` 子字段说明：

- `level`：日志级别，`warn` 表示记录警告及以上级别的日志（包括 error）
- `timestamp`：是否显示时间戳

`inbounds` 子字段说明：

- `users`：用户列表，支持配置多个用户
    - `name`：用户名称备注
    - `uuid`：UUID，每个用户建议使用独立的 UUID
    - `flow`：VLESS 流控模式
- `tls.reality`：Reality 配置
    - `private_key`：Reality 私钥
    - `short_id`：Short ID

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


