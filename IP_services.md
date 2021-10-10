---
tags: CCNP EXAM
---
# 350-401 IP services

## NTP
* RFC 958
* UDP port 123
* distributed client/server architecture
* 階層式架構(stratums)

![](https://i.imgur.com/CMs4nbT.png)

Configuration:

```
ntp server IP-ADDRESS [prefer] [source interface-id]
```

Viewing Status:
```
show ntp status
```

```
show ntp associations
```

NTP Peers:

用作備援使用:
![](https://i.imgur.com/7F4RZLW.png)
```
ntp peer IP-ADDRESS
```

## FHRP

### Object tracking
configure:
```
track 1 ip route 192.168.3.3/32 reachability
```

```
track 2 interface GigabitEthernGi0/1 line-protocol
```

### Hot Standby Rotuer Protocol

* Cisco proprietary protocol
* not support preemption by default
* Use UDP-based hello to dectect other routers
* Priority 預設為100，較高者選為acitve

版本比較:

|                   | HSRPv1                                     | HSRPv2                            |
|:----------------- | ------------------------------------------ |:--------------------------------- |
| Timers            | Does not support millisecond timer values  | Supports millisecond timer values |
| Group range       | 0~255                                      | 0~4095                            |
| Multicast address | 224.0.0.2                                  | 224.0.0.102                       |
| MAC address range | 0000.0C07.ACxy, xy為十六進制的group number | 0000.0C9F.F000 to 0000.0C9F.FFFF  |

#### Configure

1. 定義HSRP Instance
```
standby {ID} ip {vip-address}
```
2. 設定HSRP rotuer 可競爭為active router (Optional)
```
standby {ID} preempt
```
3. 設定HSRP Priority(Optional)
```
standby {instance-id} priority {priority}
```
4. 設定 HSRP MAC Address(Optional)
```
standby {instance-id} macaddress {mac-address}
```
5. 設定HSRP timers (Optional)
`standby {instance-id} timers {seconds | msec milliseconds}`
6. HSRP authentication(Optional)
```
standby {instance-id} authentication {text-password | text text-password | md5
{key-chain key-chain | key-string key-string}}
```

#### Viewing State
1. show standby brief
```
SW2# show standby brief
P indicates configured to preempt.
|
Interface Grp Pri P State Active Standby Virtual IP
Vl10 10 100 P Standby 172.16.10.3 local 172.16.10.1
```

```
SW3# show standby brief
P indicates configured to preempt.
|
Interface Grp Pri P State Active Standby Virtual IP
Vl10 10 100 P Active local 172.16.10.2 172.16.10.1
```

得知，目前SW3為active，SW2為standby。
show standby 可看到更詳細的資訊

#### Object tracking

```
SW2(config)# track 1 interface vlan 1 line-protocol
SW2(config-track)# interface vlan 10
SW2(config-if)# standby 10 priority 110
04:44:16.973: %HSRP-5-STATECHANGE: Vlan10 Grp 10 state Standby -> Active
SW2(config-if)# standby 10 track 1 decrement 20 # 若track 1 出現問題，則將該priority 降低
```

### Virtual Router Redundancy Protocol
工業標準協定，與HSRP有以下不同:
* Master router/backup router
* 預設啟用preemption
* VIP gateway MAC address:0000.5e00.01xx, XX為十六進制的group ID
* multicast address 224.0.0.18

VRRP 有v2(IPv4)和v3(IPv4、IPv6)版本

#### Configuration

![](https://i.imgur.com/aMEUjWn.png)
SW02:
```
interface Vlan10
 ip address 172.16.10.2 255.255.255.0
 vrrp 10 ip 172.16.10.1
 vrrp 10 priority 110 #Optional
 vrrp 10 authentication text CCNP #Optional
 vrrp 10 track 1 decrement 20 #Optional
```
SW03:

```
interface Vlan10
 ip address 172.16.10.3 255.255.255.0
 vrrp 10 ip 172.16.10.1
 vrrp 10 priority 110 #Optional
 vrrp 10 authentication text CCNP #Optional
 vrrp 10 track 1 decrement 20 #Optional
```

#### Viewing status

`show vrrp [brief]`

SW02:
```
SW2#show vrrp brief 
Interface          Grp Pri Time  Own Pre State   Master addr     Group addr
Vl10               10  110 3570       Y  Master  172.16.10.2     172.16.10.1    
```
SW03:
```
SW3#show vrrp brief 
Interface          Grp Pri Time  Own Pre State   Master addr     Group addr
Vl10               10  100 3609       Y  Backup  172.16.10.2     172.16.10.1  
```

#### Hierarchical VRRP configuration

在global configuration 啟用VRRPv3
```
fhrp version vrrp v3
```
SW02:
```
ip address 172.16.10.2 255.255.255.0
 vrrp 10 address-family ipv4
  track 1 decrement 20
  address 172.16.10.1 primary
  exit-vrrp
```

```
SW2#show vrrp brief 
  Interface          Grp  A-F Pri  Time Own Pre State   Master addr/Group addr
  Vl10                10 IPv4 100     0  N   Y  MASTER  172.16.10.2(local) 172.16.10.1
```

### Global Load Balancing Protocol

提供redundancy 和 Load-balancing.
GLBP 有以下兩個角色:

* Active virtual gateway(AVG)
每一個GLBP Group會有一AVG，負責回覆 ARP，Priority 最高則會被選擇為AVG

* Active virtual forwarder(AVF):
AVG 會蒐集AVF的資料，並根據Load Balancing設定來回覆ARP

#### Configuration

SW2
```
interface Vlan10
 ip address 172.16.10.2 255.255.255.0
 glbp 10 ip 172.16.10.1
 glbp 10 timers 3 6
 glbp 10 priority 150
 glbp 10 preempt
 glbp 10 weighting 70
 glbp 10 load-balancing weighted
```

load-balancing 有三種模式:

* Round robin
所有AVF輪流
* Weighted
權重
* Host dependent
以來源MAC address決定使用哪個AVF