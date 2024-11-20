# wireguard 组网

## 1. 搭建

### 安装

[官网](https://www.wireguard.com/install/)

### 服务端
> 带有公网 ip

#### 1. 开启 ip 转发
打开配置文件 `/etc/sysctl.conf` 配置以下字段
```bash
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

保存配置文件后应用更改
```bash
sudo sysctl -p
```

#### 2. 生成密钥对

```bash
wg genkey | tee server_private.key | wg pubkey > server_public.key
```
其中，`server_private.key` 是私钥， `server_public.key` 是服务端公钥

#### 3. 更改配置文件

更改配置文件 `/etc/wireguard/wg0.conf`，如果没有则新建
```ini
[Interface]
Address = <网段>
SaveConfig = true
ListenPort = <服务端口> # 服务端口需要开放 udp
PrivateKey = <服务端私钥>

[Peer]
PublicKey = <客户端公钥>
AllowedIPs = <转发到该客户端的 ip / 网段>
```

如果有多个客户端，可以在在下面新增 peer 配置

#### 4. 运行
> 如果更改配置文件后需要重启服务才能生效
启动服务端
```bash
sudo wg-quick up wg0
```

查看运行状态
```bash
sudo wg show
```

停止运行
```bash
sudo wg-quick down wg0
```

### 客户端

#### 1. 生成密钥对
```bash
wg genkey | tee client_private.key | wg pubkey > client_public.key
```
其中，`client_private.key` 是私钥， `client_public.key` 是服务端公钥

#### 3. 更改配置文件

更改配置文件 `client.conf`，如果没有则新建
```ini
[Interface]
Address = <客户端 ip> # 需要和服务端同一网段
PrivateKey = <客户端私钥>
ListenPort = 62340

[Peer]
PublicKey = <服务端公钥>
Endpoint = <服务端ip:port>
AllowedIPs = <转发到服务端的网段>
PersistentKeepalive = 25
```
#### 4. 启动
根据不同平台启动 wireguard 并导入配置文件


