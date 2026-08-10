# debian 安装和管理

在 Debian 系统中安装和管理 sing-box，为后续服务端和客户端配置提供运行环境

# 环境准备

更新软件包索引：

```shell
apt update
```

升级已安装的软件包：

> 根据实际情况运行

```shell
apt upgrade -y
```

安装工具：

```shell
apt install curl vim -y
```

# 安装 sing-box

添加 sing-box 软件源密钥：

```shell
mkdir -p /etc/apt/keyrings

curl -fsSL https://sing-box.app/gpg.key -o /etc/apt/keyrings/sagernet.asc

chmod a+r /etc/apt/keyrings/sagernet.asc
```

添加 sing-box 软件源：

```shell
echo '
Types: deb
URIs: https://deb.sagernet.org/
Suites: *
Components: *
Enabled: yes
Signed-By: /etc/apt/keyrings/sagernet.asc
' | tee /etc/apt/sources.list.d/sagernet.sources
```

更新软件包索引：

```shell
apt update
```

安装 sing-box：

```shell
apt install sing-box -y
```

查看 sing-box 版本：

```shell
sing-box version
```

# 管理 sing-box 服务

重启 sing-box 服务：

```shell
systemctl restart sing-box
```

查看 sing-box 状态：

```shell
systemctl status sing-box
```

停止 sing-box 服务：

```shell
systemctl stop sing-box
```

注册开机启动：

```shell
systemctl enable sing-box
```


