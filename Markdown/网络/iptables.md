```bash
# 自动将数据包的源IP地址替换为出口网卡的IP地址
# 只能在 POSTROUTING 链中使用
# 主要实现网络地址转换，让内网设备通过网关访问外网
iptables -t nat -I POSTROUTING -j MASQUERADE


# 1. 指定出口网卡
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# 2. 指定源地址范围
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE

# 3. 指定端口范围（可选）
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE --to-ports 1024-65535
```


### 表和链的关系与优先级
优先级顺序：当数据包经过时，iptables 按以下顺序检查表（优先级从高到低）：
```
raw → mangle → nat → filter
```
（同一数据包会依次经过这些表的对应链，但并非所有表都会处理该数据包）

#### 规则匹配逻辑：
+ 每个链中的规则按从上到下的顺序匹配，一旦匹配到某条规则，就执行规则定义的动作（如允许、拒绝、跳转），后续规则不再处理。
+ 若没有匹配到任何规则，则执行链的默认策略（如 DROP 或 ACCEPT）。



### 问题

#### 本机网络问题

```bash
# 允许本地回环地址可以正常使用
iptables -I INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
```

```bash
# #允许已建立的或相关连的通行
iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```