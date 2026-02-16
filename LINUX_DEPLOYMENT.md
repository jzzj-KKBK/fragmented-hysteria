# Linux amd64 部署指南

## 编译信息
- 编译时间：2026/2/16 3:28
- 文件大小：29,275,090 字节
- 架构：Linux amd64
- Go版本：1.26.0

## 文件位置
编译好的二进制文件位于：`build/hysteria-linux-amd64`

## 移植到Linux系统的步骤

### 1. 传输文件到Linux服务器
```bash
# 使用scp传输文件
scp build/hysteria-linux-amd64 user@your-server:/tmp/

# 或者使用其他传输工具如rsync
```

### 2. 在Linux服务器上设置权限
```bash
# 移动到合适的位置
sudo mv /tmp/hysteria-linux-amd64 /usr/local/bin/hysteria

# 设置可执行权限
sudo chmod +x /usr/local/bin/hysteria

# 验证版本
hysteria version
```

### 3. 配置系统服务（可选）
创建systemd服务文件 `/etc/systemd/system/hysteria.service`：
```ini
[Unit]
Description=Hysteria VPN Service
After=network.target

[Service]
Type=simple
User=nobody
Group=nogroup
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE CAP_NET_RAW
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE CAP_NET_RAW
NoNewPrivileges=true
ExecStart=/usr/local/bin/hysteria server --config /etc/hysteria/config.yaml
Restart=on-failure
RestartSec=5s
LimitNOFILE=infinity

[Install]
WantedBy=multi-user.target
```

### 4. 创建配置文件
创建 `/etc/hysteria/config.yaml`：
```yaml
listen: :443
acme:
  domains:
    - your-domain.com
  email: your-email@example.com
auth:
  type: password
  password: your-secure-password
```

### 5. 启动服务
```bash
sudo systemctl daemon-reload
sudo systemctl enable hysteria
sudo systemctl start hysteria
sudo systemctl status hysteria
```

## 测试连接

### 客户端配置
创建客户端配置文件 `client.yaml`：
```yaml
server: your-domain.com:443
auth: your-secure-password
tls:
  sni: your-domain.com
  insecure: false
socks5:
  listen: 127.0.0.1:1080
```

### 运行客户端
```bash
hysteria client --config client.yaml
```

## 调试和故障排除

### 1. 检查日志
```bash
# 查看服务日志
sudo journalctl -u hysteria -f

# 查看详细日志
sudo journalctl -u hysteria -f -o cat
```

### 2. 测试网络连接
```bash
# 测试端口连通性
nc -zv your-domain.com 443

# 使用curl测试
curl -v https://your-domain.com
```

### 3. 抓包分析（如果仍有连接问题）
```bash
# 在服务器端抓包
sudo tcpdump -i any port 443 -w hysteria.pcap

# 在客户端抓包
sudo tcpdump -i any port 1080 -w client.pcap

# 使用Wireshark分析
wireshark hysteria.pcap
```

## 性能优化建议

1. **调整内核参数**：
```bash
# 增加文件描述符限制
echo "fs.file-max = 1000000" >> /etc/sysctl.conf
echo "net.core.rmem_max = 134217728" >> /etc/sysctl.conf
echo "net.core.wmem_max = 134217728" >> /etc/sysctl.conf
sudo sysctl -p
```

2. **启用BBR拥塞控制**：
```bash
echo "net.core.default_qdisc = fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control = bbr" >> /etc/sysctl.conf
sudo sysctl -p
```

## 安全注意事项

1. 使用强密码或证书认证
2. 定期更新软件版本
3. 配置防火墙规则
4. 启用日志监控
5. 考虑使用Docker容器化部署

## 联系支持
如果在部署过程中遇到问题，请检查：
1. 防火墙设置
2. 域名解析
3. TLS证书状态
4. 服务日志中的错误信息