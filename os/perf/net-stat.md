## 网络测试

一些 WEB 端服务:

- `https://ipinfo.io/`, IP 信息及地理位置
- `https://www.pingdom.com/` 网站的运行时间和响应速度
- `https://dnschecker.org/` DNS 记录查询, 以及 DNS 缓存传播检测
- `https://lookup.icann.org/en` 在线 WhoIS

工具的大致分类：

 功能 | Unix 工具 | Windows 工具 | 新工具 | 网络层次 
 -----|-----------|--------------| -----------| --------------
本机网卡配置 | ip | ipconfig  
本机连接探测 | ss | netstat | rustnet
DNS 查询 | dig, nslookup, whois 
网络路径 | ping, traceroute | | trippy | 传输层以下
端口可达性 | nc, ncat | 
TLS/HTTP | curl | wget | xh  | 应用层 
抓包 (DPI) | wireshark, ngrep 

其他功能：

* 网络代理配置 mihomo
* 网络性能测试：带宽、延时、吞吐
* 渗透测试工具

## 连通问题排查思路

1. (L1) 确认主机网卡状态 `ifconifg`，如 DOWN、RX/TX error 
2. 查看链路错误 `ip -s link`，如果存在大量错误，怀疑交换机、网线、网卡驱动。
3. (L2) 检查同网段 ARP 响应 `arp -n` ，无响应说明 VLAN 错误。
4. (L3) 检查 `ip addr` 
5. 检查路由是否正确 `ip route get <IP>`，如是否走向预期网关，并检查网关状态。
6. `ping <IP>`，如不通，检查是否有 ACL 规则或防火墙规则
7. 跨网段时，`traceroute <IP>` ，如果某跳丢失，说明中间设备有问题 
8. (L4) 检查服务端口 `ss -lntp | grep <Port>` 是否在监听，尝试本机连接
9. 如果是应用的问题，尝试 `tcpdump` 抓包，查看 `TCP` 报文行为。
10. 检查 TCP 端口冲突，或者连接数限制
11. (L5) 用 `htop` 查看是否应用卡死，查看应用日志

## `ip addr`

```bash 
$ ip addr
5: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1492 qdisc mq state UP group default qlen 1000
    link/ether c8:6e:08:fb:f2:89 brd ff:ff:ff:ff:ff:ff
    altname enxc86e08fbf289
    inet 192.168.1.13/24 brd 192.168.1.255 scope global noprefixroute eth2
       valid_lft forever preferred_lft forever
    inet6 2409:8a3c:518e:a760:ed98:b4b:3c53:f11/128 scope global nodad noprefixroute
       valid_lft forever preferred_lft forever
    inet6 2409:8a3c:518e:a760:2165:e207:ce66:1c96/64 scope global nodad deprecated noprefixroute
       valid_lft forever preferred_lft 0sec
    inet6 fe80::bc9c:b52b:1927:29eb/64 scope link nodad noprefixroute
       valid_lft forever preferred_lft forever
6: eth3: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether c8:6e:08:fb:f2:8a brd ff:ff:ff:ff:ff:ff
    altname enxc86e08fbf28a
```

* `LOWER_UP` 物理网线连通
* `UP` 网卡驱动正常

## `ss`

* -t/u 查询 TCP/UDP 连接
* -p 显示进程信息，如 PID 和进程名
* -l 过滤 LISTEN 状态的套接字
* -n 不显示域名，显示 `ip：port`

```bash 
$ # 监听端口的进程
$ ss -ltnp
State        Recv-Q     Send-Q    Local Address:Port    Peer Address:Port   Process
LISTEN       0          20        127.0.0.1:25           0.0.0.0:*
LISTEN       0          1000      10.255.255.254:5       0.0.0.0:*
LISTEN       0          20        [::1]:25               [::]:*                  
```

当 Socket 处于 `Established` 状态时，

* `Recv-Q` 表示缓冲区中还未被读取的字节数
* `Send-Q` 表示缓冲区中还没被远端主机确定的字节数

当 Socket 处于 `Listen` 时，

* `Recv-Q` 表示全连接队列的长度
* `Send-Q` 表示全连接队列的最大长度。全连接队列是指完成三次握手后，还没有被 `accept()` 取走的连接的存储队列。

```c
socket();
bind();
listen(fd, ...); // 内核开始监听

while(true) {
	int conn_fd = accept(fd, ...); // 从全连接队列获取一个 TCP 连接，
	// 如 (src_ip, src_port, dst_ip, dst_port) 
}
```

## PenTest

* nmap
* sql-map
* burpsuite
