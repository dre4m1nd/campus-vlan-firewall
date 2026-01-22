# campus-vlan-firewall
《校园办公区与教学区网络VLAN隔离与防火墙边界防护》的配置信息
### 附录1：各设备配置信息

1. 二层交换机 SW-Office（办公区）

```plain
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW-Office
SW-Office(config)#vlan 10
SW-Office(config-vlan)#name Office_Area
SW-Office(config-vlan)#exit
SW-Office(config)#interface range FastEthernet0/1-20
SW-Office(config-if-range)#switchport mode access
SW-Office(config-if-range)#switchport access vlan 10
SW-Office(config-if-range)#switchport port-security
SW-Office(config-if-range)#switchport port-security violation shutdown
SW-Office(config-if-range)#no shutdown
SW-Office(config-if-range)#exit
SW-Office(config)#interface FastEthernet0/24
SW-Office(config-if)#switchport mode trunk
SW-Office(config-if)#switchport trunk allowed vlan 10
SW-Office(config-if)#no negotiation auto
SW-Office(config-if)#no shutdown
SW-Office(config-if)#exit
SW-Office(config)#exit
SW-Office#write memory
```

#### 2. 二层交换机 SW-Teach（教学区）
```plain
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW-Teach
SW-Teach(config)#vlan 20
SW-Teach(config-vlan)#name Teach_Area
SW-Teach(config-vlan)#exit
SW-Teach(config)#interface range FastEthernet0/1-20
SW-Teach(config-if-range)#switchport mode access
SW-Teach(config-if-range)#switchport access vlan 20
SW-Teach(config-if-range)#switchport port-security
SW-Teach(config-if-range)#switchport port-security violation restrict
SW-Teach(config-if-range)#no negotiation auto
SW-Teach(config-if-range)#no shutdown
SW-Teach(config-if-range)#exit
SW-Teach(config)#interface FastEthernet0/24
SW-Teach(config-if)#switchport mode trunk
SW-Teach(config-if)#switchport trunk allowed vlan 20
SW-Teach(config-if)#no negotiation auto
SW-Teach(config-if)#no shutdown
SW-Teach(config-if)#exit
SW-Teach(config)#exit
SW-Teach#write memory
```

#### 3. 三层交换机 SW-Core（核心路由）
```plain
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW-Core
SW-Core(config)#vlan 10
SW-Core(config-vlan)#name Office_Area
SW-Core(config-vlan)#vlan 20
SW-Core(config-vlan)#name Teach_Area
SW-Core(config-vlan)#vlan 30
SW-Core(config-vlan)#name DMZ_Area
SW-Core(config-vlan)#exit
SW-Core(config)#interface Vlan10
SW-Core(config-if)#ip address 192.168.10.1 255.255.255.0
SW-Core(config-if)#no shutdown
SW-Core(config-if)#exit
SW-Core(config)#interface Vlan20
SW-Core(config-if)#ip address 192.168.20.1 255.255.255.0
SW-Core(config-if)#no shutdown
SW-Core(config-if)#exit
SW-Core(config)#interface Vlan30
SW-Core(config-if)#ip address 192.168.30.1 255.255.255.0
SW-Core(config-if)#no shutdown
SW-Core(config-if)#exit
SW-Core(config)#interface FastEthernet0/1
SW-Core(config-if)#switchport mode trunk
SW-Core(config-if)#switchport trunk allowed vlan 10
SW-Core(config-if)#no negotiation auto
SW-Core(config-if)#no shutdown
SW-Core(config-if)#exit
SW-Core(config)#interface FastEthernet0/2
SW-Core(config-if)#switchport mode trunk
SW-Core(config-if)#switchport trunk allowed vlan 20
SW-Core(config-if)#no negotiation auto
SW-Core(config-if)#no shutdown
SW-Core(config-if)#exit
SW-Core(config)#interface FastEthernet0/3
SW-Core(config-if)#switchport mode access
SW-Core(config-if)#switchport access vlan 30
SW-Core(config-if)#no shutdown
SW-Core(config-if)#exit
SW-Core(config)#interface FastEthernet0/24
SW-Core(config-if)#no switchport
SW-Core(config-if)#ip address 192.168.0.2 255.255.255.0
SW-Core(config-if)#no shutdown
SW-Core(config-if)#exit
SW-Core(config)#ip routing
SW-Core(config)#ip route 0.0.0.0 0.0.0.0 192.168.0.1
SW-Core(config)#exit
SW-Core#write memory
```

#### 4. 防火墙 Cisco ASA 5505（安全防护）
```plain
ASA>enable
ASA#configure terminal
ASA(config)#clear configure dhcpd
ASA(config)#license activation key 0xd41d8cd98f00b204e9800998ecf8427e
ASA(config)#exit
ASA#reload
ASA>enable
ASA#configure terminal
ASA(config)#interface Vlan1
ASA(config-if)#nameif inside
ASA(config-if)#security-level 100
ASA(config-if)#ip address 192.168.0.1 255.255.255.0
ASA(config-if)#no shutdown
ASA(config-if)#exit
ASA(config)#interface Vlan2
ASA(config-if)#nameif dmz
ASA(config-if)#security-level 50
ASA(config-if)#ip address 192.168.30.2 255.255.255.0
ASA(config-if)#no shutdown
ASA(config-if)#exit
ASA(config)#interface Vlan3
ASA(config-if)#nameif outside
ASA(config-if)#security-level 0
ASA(config-if)#ip address 202.100.1.2 255.255.255.0
ASA(config-if)#no shutdown
ASA(config-if)#exit
ASA(config)#interface Ethernet0/0
ASA(config-if)#switchport access vlan 3
ASA(config-if)#no shutdown
ASA(config-if)#exit
ASA(config)#interface Ethernet0/1
ASA(config-if)#switchport access vlan 1
ASA(config-if)#no shutdown
ASA(config-if)#exit
ASA(config)#interface Ethernet0/2
ASA(config-if)#switchport access vlan 2
ASA(config-if)#no shutdown
ASA(config-if)#exit
ASA(config)#interface range Ethernet0/3-7
ASA(config-if-range)#shutdown
ASA(config-if-range)#exit
ASA(config)#route inside 192.168.10.0 255.255.255.0 192.168.0.2 1
ASA(config)#route inside 192.168.20.0 255.255.255.0 192.168.0.2 1
ASA(config)#route dmz 192.168.30.0 255.255.255.0 192.168.30.1 1
ASA(config)#route outside 100.64.0.0 255.255.255.0 202.100.1.1 1
ASA(config)#access-list inside_access_in extended permit tcp 192.168.10.0 255.255.255.0 host 192.168.30.10 eq 80
ASA(config)#access-list inside_access_in extended permit tcp 192.168.10.0 255.255.255.0 host 192.168.30.10 eq 443
ASA(config)#access-list inside_access_in extended deny ip 192.168.20.0 255.255.255.0 192.168.10.0 255.255.255.0
ASA(config)#access-list inside_access_in extended permit ip any any
ASA(config)#access-group inside_access_in in interface inside
ASA(config)#access-list outside_access_in extended deny icmp 100.64.0.0 255.255.255.0 host 192.168.30.10 icmp-type echo-request
ASA(config)#access-list outside_access_in extended permit tcp any host 192.168.30.10 eq 80
ASA(config)#access-list outside_access_in extended permit icmp any any icmp-type echo-reply
ASA(config)#access-group outside_access_in in interface outside
ASA(config)#access-list dmz_access_in extended deny ip any 192.168.10.0 255.255.255.0
ASA(config)#access-list dmz_access_in extended permit ip any any
ASA(config)#access-group dmz_access_in in interface dmz
ASA(config)#exit
ASA#write memory
```

#### 5. 路由器 Cisco 2911（外网出口）
```plain
Router>enable
Router#configure terminal
Router(config)#hostname R-Internet
R-Internet(config)#interface GigabitEthernet0/0
R-Internet(config-if)#ip address 202.100.1.1 255.255.255.0
R-Internet(config-if)#no shutdown
R-Internet(config-if)#exit
R-Internet(config)#interface GigabitEthernet0/1
R-Internet(config-if)#ip address 100.64.0.1 255.255.255.0
R-Internet(config-if)#no shutdown
R-Internet(config-if)#exit
R-Internet(config)#ip route 192.168.0.0 255.255.255.0 202.100.1.2
R-Internet(config)#ip route 192.168.10.0 255.255.255.0 202.100.1.2
R-Internet(config)#ip route 192.168.20.0 255.255.255.0 202.100.1.2
R-Internet(config)#ip route 192.168.30.0 255.255.255.0 202.100.1.2
R-Internet(config)#ip routing
R-Internet(config)#exit
R-Internet#write memory
```

### 附录2：设备与终端IP配置表

| 设备类型 | 设备名称 | IP地址 | 子网掩码 | 网关地址 | VLAN ID | 部署位置 |
| --- | --- | --- | --- | --- | --- | --- |
| 服务器 | 财务服务器 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 | 10 | 办公区 |
| 服务器 | 资源服务器 | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 | 30 | DMZ区 |
| 终端PC | 办公区PC | 192.168.10.20 | 255.255.255.0 | 192.168.10.1 | 10 | 办公区 |
| 终端PC | 教学区PC | 192.168.20.20 | 255.255.255.0 | 192.168.20.1 | 20 | 教学区 |
| 攻击机 | 外网攻击机1 | 100.64.0.10 | 255.255.255.0 | 100.64.0.1 | - | 外网 |
| 攻击机 | 教学区攻击机2 | 192.168.20.30 | 255.255.255.0 | 192.168.20.1 | 20 | 教学区 |
| 三层交换机 | SW-Core Vlan10 | 192.168.10.1 | 255.255.255.0 | - | 10 | 核心层 |
| 三层交换机 | SW-Core Vlan20 | 192.168.20.1 | 255.255.255.0 | - | 20 | 核心层 |
| 三层交换机 | SW-Core Vlan30 | 192.168.30.1 | 255.255.255.0 | - | 30 | 核心层 |
| 三层交换机 | SW-Core Fa0/24 | 192.168.0.2 | 255.255.255.0 | 192.168.0.1 | - | 核心层 |
| 防火墙 | ASA Inside | 192.168.0.1 | 255.255.255.0 | - | 1 | 安全边界 |
| 防火墙 | ASA DMZ | 192.168.30.2 | 255.255.255.0 | - | 2 | 安全边界 |
| 防火墙 | ASA Outside | 202.100.1.2 | 255.255.255.0 | - | 3 | 安全边界 |
| 路由器 | R-Internet G0/0 | 202.100.1.1 | 255.255.255.0 | - | - | 外网出口 |
| 路由器 | R-Internet G0/1 | 100.64.0.1 | 255.255.255.0 | - | - | 外网出口 |

