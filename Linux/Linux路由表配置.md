```shell
echo "1 middleman" >> /etc/iproute2/rt_tables ip route add 0.0.0.0/0 dev wg2 table middleman ip rule add from <NodeA对Client的allowed-ips> lookup middleman
```