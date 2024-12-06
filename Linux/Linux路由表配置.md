```shell
echo "1 middleman" >> /etc/iproute2/rt_tables 
ip route add 0.0.0.0/0 dev wg2 table middleman 
ip rule add from <NodeA对Client的allowed-ips> lookup middleman
```

这段代码的作用是配置 Linux 系统中的路由表和策略路由，使流量通过特定的接口 `wg2` 来处理。下面是逐行解释：

### 代码详解

1. **`echo "1 middleman" >> /etc/iproute2/rt_tables`**
    
    - 作用：在系统的路由表配置文件 `/etc/iproute2/rt_tables` 中添加一个新的自定义路由表，编号为 `1`，名字为 `middleman`。
    - 路由表编号和名字的对应关系定义了新的路由表，以便后续可以通过名字引用它。
    - 例如，`middleman` 路由表的 ID 为 1。
2. **`ip route add 0.0.0.0/0 dev wg2 table middleman`**
    
    - 作用：在 `middleman` 路由表中添加一条默认路由，表示将所有的流量（`0.0.0.0/0`）通过 `wg2` 接口发送。
    - `wg2` 是一个 WireGuard 接口，在此场景下通常用于加密流量或 VPN 连接。
3. **`ip rule add from <NodeA对Client的allowed-ips> lookup middleman`**
    
    - 作用：定义一条策略路由规则，将来自 `<NodeA对Client的allowed-ips>` 的流量指定使用 `middleman` 路由表。
    - `<NodeA对Client的allowed-ips>` 需要替换为具体的 IP 地址范围，通常是一个子网（如 `192.168.1.0/24`）。

### 整体流程

- 为了灵活管理不同接口和流量的路由策略，自定义了 `middleman` 路由表，并配置让特定流量从 `wg2` 接口通过。
- 此方法适用于多路由场景，例如：
    - 配置策略路由实现不同的出口路由。
    - 在 VPN 配置中强制指定流量走特定通道。
    - 或者，用于特定网络的中间代理（例如流量中转、拦截等）。

### 注意事项

1. **`<NodeA对Client的allowed-ips>` 替换**
    
    - 替换为具体的 IP 地址或子网，例如 `192.168.10.0/24` 或单一 IP `192.168.10.10`。
    - 确保这个范围对应的是你希望通过 `wg2` 接口路由的流量。
2. **权限要求**
    
    - 需要 `root` 权限来执行上述命令。
3. **持久化**
    
    - 如果希望配置在系统重启后仍然生效，需将这些命令加入到启动脚本中，例如 `/etc/rc.local` 或对应的服务管理器（如 `systemd`）。


```
ip route show table 0
```

###### ChatGPT 说：

ChatGPT

`ip route show table 0` 是一个用于显示所有路由表内容的命令。它可以展示系统中默认路由表和自定义路由表的所有内容。

### 命令详解

- **`ip route`**：显示路由表内容。
- **`show`**：显示指定的路由表内容。
- **`table 0`**：表示显示 **所有路由表的内容**，包括主路由表、默认路由表 (`local`)，以及用户定义的表。

### 输出示例

运行 `ip route show table 0` 后，你可能会看到类似以下内容：

sql

复制代码

`local     broadcast 192.168.1.255 dev eth0 proto kernel scope link src 192.168.1.100     local 192.168.1.100 dev eth0 proto kernel scope host src 192.168.1.100     broadcast 192.168.1.0 dev eth0 proto kernel scope link src 192.168.1.100 main     default via 192.168.1.1 dev eth0 proto dhcp src 192.168.1.100 metric 100     192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100 metric 100 middleman     default dev wg2`

### 输出说明

1. **`local` 路由表**：
    
    - 包含与本地网络相关的特殊路由，例如广播地址、本地地址等。
    - 用于本地通信（如本机向自己发送数据）。
2. **`main` 路由表**：
    
    - 系统的默认路由表，通常包含主要路由信息，例如默认网关和本机子网。
3. **`middleman` 路由表**（自定义）：
    
    - 刚才通过 `ip route add ... table middleman` 添加的路由表。
    - 示例中，它定义了一个默认路由，所有流量通过 `wg2` 接口。

### 使用场景

- 检查所有路由表的配置，调试多路由问题。
- 验证自定义路由表的规则是否生效。
- 确保策略路由与路由表配置一致。

### 注意事项

- 如果你没有定义额外的路由表，输出可能只显示系统默认的 `local` 和 `main` 路由表。
- 检查输出时，要确保路由表的规则与 `ip rule` 配置的优先级和匹配条件一致。


```
ip route show table 0 | grep default
```

###### ChatGPT 说：

ChatGPT

`ip route show table 0 | grep default` 的作用是从所有路由表中筛选出包含默认路由 (`default`) 的条目。这是调试和检查默认路由配置时常用的命令。

---

### 示例输出

运行命令后的可能输出：

plaintext

复制代码

`default via 192.168.1.1 dev eth0 proto dhcp src 192.168.1.100 metric 100 default dev wg2 table middleman`

---

### 输出解释

1. **`default via 192.168.1.1 dev eth0`**
    
    - 默认路由通过网关 `192.168.1.1`，使用接口 `eth0`。
    - `proto dhcp` 表示该路由由 DHCP 配置。
    - `src 192.168.1.100` 指定默认的源 IP 地址。
    - `metric 100` 是路由的优先级（较低的值表示更高的优先级）。
2. **`default dev wg2 table middleman`**
    
    - 默认路由指向 `wg2` 接口。
    - 属于 `middleman` 自定义路由表。
    - 没有明确的网关，流量直接从接口发送。

---

### 用途

- **检查默认路由配置：** 确保流量正确地分流到指定的接口或网关。
- **调试多路由环境：** 验证策略路由是否将流量正确匹配到自定义路由表。
- **优化路由优先级：** 通过查看 `metric` 值，调整默认路由的优先级。

---

### 注意事项

1. **多表路由的匹配优先级**
    
    - 默认情况下，主路由表 (`main`) 的规则优先级较高。使用 `ip rule` 指定的规则，可以改变匹配条件。
2. **可能无输出**
    
    - 如果没有自定义默认路由，或自定义路由表未配置默认路由，命令可能不会有任何输出。
3. **持久化配置**
    
    - 临时添加的路由规则在系统重启后可能会丢失，需要在配置文件中进行持久化。

如果你对输出有疑问，或需要更详细的分析，提供上下文我可以进一步帮助！


# 持久化配置
在 Linux 系统中，临时添加的路由和策略路由规则会在系统重启后丢失。为了使配置持久化，可以根据使用的系统类型和需求，通过以下方式实现：

---

## **方法 1：编辑网络配置文件（推荐）**

### 在 Debian/Ubuntu 系列系统中

1. **修改接口配置文件**  
    编辑 `/etc/network/interfaces` 文件（或其他网络管理工具的配置文件，例如 `Netplan` 或 `NetworkManager`）。
    
    示例：
    
    plaintext
    
    复制代码
    
    `auto wg2 iface wg2 inet manual     pre-up ip route add 0.0.0.0/0 dev wg2 table middleman     pre-up ip rule add from <NodeA对Client的allowed-ips> lookup middleman`
    
2. **重启网络服务**：
    
    bash
    
    复制代码
    
    `sudo systemctl restart networking`
    

---

### 在 RHEL/CentOS 系列系统中

1. **添加路由规则到接口配置文件**  
    编辑 `/etc/sysconfig/network-scripts/route-wg2`：
    
    plaintext
    
    复制代码
    
    `0.0.0.0/0 dev wg2 table middleman`
    
2. **添加策略路由规则**  
    编辑 `/etc/sysconfig/network-scripts/rule-wg2`：
    
    plaintext
    
    复制代码
    
    `from <NodeA对Client的allowed-ips> table middleman`
    
3. **重启网络服务**：
    
    bash
    
    复制代码
    
    `sudo systemctl restart network`
    

---

## **方法 2：使用启动脚本**

1. **编辑 `/etc/rc.local` 文件**  
    如果系统启用了 `rc.local`，可以在文件中添加路由命令：
    
    bash
    
    复制代码
    
    `echo "1 middleman" >> /etc/iproute2/rt_tables ip route add 0.0.0.0/0 dev wg2 table middleman ip rule add from <NodeA对Client的allowed-ips> lookup middleman`
    
2. **赋予可执行权限**：
    
    bash
    
    复制代码
    
    `sudo chmod +x /etc/rc.local`
    
3. **启用 `rc-local` 服务**（某些系统默认禁用）：
    
    bash
    
    复制代码
    
    `sudo systemctl enable rc-local sudo systemctl start rc-local`
    

---

## **方法 3：使用 Systemd 服务**

1. **创建自定义服务文件**  
    在 `/etc/systemd/system/custom-routing.service` 中创建文件：
    
    plaintext
    
    复制代码
    
    `[Unit] Description=Custom Routing Configuration After=network.target  [Service] Type=oneshot ExecStart=/usr/bin/bash -c '     echo "1 middleman" >> /etc/iproute2/rt_tables &&     ip route add 0.0.0.0/0 dev wg2 table middleman &&     ip rule add from <NodeA对Client的allowed-ips> lookup middleman ' RemainAfterExit=true  [Install] WantedBy=multi-user.target`
    
2. **启用并启动服务**：
    
    bash
    
    复制代码
    
    `sudo systemctl daemon-reload sudo systemctl enable custom-routing sudo systemctl start custom-routing`
    

---

## **方法 4：直接修改系统文件**

1. **修改 `/etc/iproute2/rt_tables` 文件**  
    将 `1 middleman` 添加到该文件末尾。
    
2. **在 `/etc/network/if-up.d/` 中添加脚本**  
    创建脚本文件 `/etc/network/if-up.d/routing`：
    
    bash
    
    复制代码
    
    `#!/bin/bash if [ "$IFACE" = "wg2" ]; then     ip route add 0.0.0.0/0 dev wg2 table middleman     ip rule add from <NodeA对Client的allowed-ips> lookup middleman fi`
    
3. **赋予脚本可执行权限**：
    
    bash
    
    复制代码
    
    `sudo chmod +x /etc/network/if-up.d/routing`
    

---

## **验证持久化**

完成持久化配置后，重启系统并检查是否生效：

bash

复制代码

`ip route show table middleman ip rule show`