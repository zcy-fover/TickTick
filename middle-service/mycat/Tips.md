#### 高可用 VIP
**高可用性HA** (High Availability) 尽量缩短系统日常的维护操作和突发的系统奔溃导致的停机时间，提高系统和应用的可用性。数据库通常通过主从关系来实现。

数据库实现高可用，数据库服务搭建主从关系，主库 IP、从库 IP、读写 VIP、只读 VIP，使用这几个 IP 地址都可以连接对应的数据库服务。VIP 如何寻址到对应的服务器，使用的是 TCP/IP 协议的 ARP 协议。当 物理数据库无法使用时，动态将 VIP 对应的 MAC 地址替换成从库