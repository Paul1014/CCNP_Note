---
tags: CCNP EXAM
---
# DMVPN Tunnel

## GRE Tunnel
[GRE Tunnel](https://hackmd.io/8V4ua_xiSWm6bWY8a5FS4g?view#Generic-Routing-EncapsulationGRE-Tunels)
DMVPN uses Multipoint GRE(mGRE)

## Next Hop Resolution Protocol

* RFC2332
* ARP-Like
* Client/server protocol: NHS and NHC
DMVPN 是使用mGRE 多個tunnel 方式建立，NHRP 則是用來 mapping 這些 ip位址。 DMVPN spoke(NHC) 會手動設定 DMVPN hub(NHS)位址。
NHC即可向NHS 註冊他們的 tunnel 和 NBMA位址


Message Types:


| Type         | Description                                                                                   |
| ------------ |:--------------------------------------------------------------------------------------------- |
| Registration | NHC -> NHS, let the hubs to nknow spoke's NMBA information                                    |
| Resolution   | Resolution reply the provices the IP tunnel IPaddress and NBMA IP address of the remote spoke |
| Redirect     | DMVPN Phase 3, notify the optimal path(spoke-to-spoke tunnel)                                 |
| Purge        | Remove a cached NHRP entry, NHS -> NHC                                                        |
| Error        | notify the sender of an NHRP packet that an error has occureed                                |

Message Extensions:

| Extension | Description |
| --------- |:----------- |
|   Responder address        |    用來決定回覆節點的位址         |
|   Forward transit NHS record        |  內涵 NHRP request packet            |
|   Reverse transit NHS record        |   內涵 NHRP rreply packet           |
|   Authentication        |     驗證 NHRP speaker間的資訊，明文傳輸        |
|Vendor private|This conveys vendor private information between NHRP speakers.|
|NAT | 當hub or spoker是藏在NAT裝置後面，透過該extension 宣告 NBMA address(inside local address) |

## DMVPN

### Phase 1: Spoke-to-Hub
Traffic 必須經過hub轉送
### Phase 2: Spoke-to-Spoke
提供spoke-to-spoke communication, 但不允許 summarization (next-hop preservation)
因此不支援不同的DMVPN network(multilevel hierachical DMVPN) 的 spoke-to-spoke連線

### Phase 3: Hierarchical Tree Spoke-to-Spoke
Phase 藉由加強NHRP與routing table的消息傳遞與互動，改善spoke-to-spoke的連線。
NHRP redirect(hub上啟用) message提供重要的資訊，使spoke 發起者能夠初始化 dest的解析

NHRP shortcut(spoke上啟用) 使Phase 3 可以支援summarication

![](https://i.imgur.com/sAXIWDC.png)
![](https://i.imgur.com/ToCXgmX.png)




## DMVPN 設定
Topology:
![](https://i.imgur.com/PKm8VKv.png)

mtu 建議1400
mss 建議1360

### Hub
```
interface Tunnel0
 ip address 192.168.124.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic #o. To support multicast or routing protocols
 ip nhrp network-id 1 #Network-id 1 用於辨別個不同的GRE Tunnel 
 ip nhrp redirect # 用於Phase 3 
 tunnel source Ethernet0/0
 tunnel mode gre multipoint  # 將Tunnel 設定成Multipoint mode
```



### Spoke
R2:
```
interface Tunnel0
 ip address 192.168.124.2 255.255.255.0
 no ip redirects
 ip nhrp map 192.168.124.1 10.13.1.1  #將NBMA 位址與 Tunnel IP位址對應
 ip nhrp map multicast 10.13.11 #用dynamic routing protocol 會需要設定 
 ip nhrp network-id 1
 ip nhrp nhs 192.168.124.1  #指定nhs 位址
 ip nhrp shortcut # 用於Phase 3
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
```

R4:
```
interface Tunnel0
 ip address 192.168.124.4 255.255.255.0
 no ip redirects
 ip nhrp map 192.168.124.1 10.13.1.1  #將NBMA 位址與 Tunnel IP位址對應
 ip nhrp map multicast 10.13.11 #用dynamic routing protocol 會需要設定 
 ip nhrp network-id 1
 ip nhrp nhs 192.168.124.1  #指定nhs 位址
 ip nhrp shortcut # 用於Phase 3
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
```

### Phase 3 實際應用
在hub router 上發布summarize route，並將next-hop 指回hub route
在透過 hub router 上設定 ip nhrp redircert 及在spoke 上設定 ip nhrp shortcut 
即可讓Spoke 查詢NHRP 後將封包直接送往另一個Spoke

R1 routing table:
```
D     192.168.0.0/18 is a summary, 00:40:06, Null0
D     192.168.20.0/24 [90/27008000] via 192.168.124.2, 00:38:52, Tunnel0
D     192.168.21.0/24 [90/27008000] via 192.168.124.2, 00:38:52, Tunnel0
D     192.168.22.0/24 [90/27008000] via 192.168.124.2, 00:38:52, Tunnel0
D     192.168.23.0/24 [90/27008000] via 192.168.124.2, 00:38:52, Tunnel0
D     192.168.40.0/24 [90/27008000] via 192.168.124.4, 00:38:52, Tunnel0
```

R2 & R4 routing table:
```
Gateway of last resort is not set

D     192.168.0.0/18 [90/27008000] via 192.168.124.1, 00:39:45, Tunnel0
```

R2 traceroute R4:
```
Tracing the route to 192.168.40.4

  1 192.168.124.4 12 msec 8 msec *
```

R2 show ip nhrp:
```
192.168.40.4/32 via 192.168.124.4
   Tunnel0 created 00:39:51, expire 01:20:17
   Type: dynamic, Flags: router 
   NBMA address: 10.43.1.4
```
### View DMVPN Tunnel status

`show dmvpn [detail]`

status:
* INTF: The line protocol of the DMVPN tunnel is down.
* IKE: DMVPN tunnels 設定IPsec 但IKE 建立還沒成功
* IPsec : IPsec SA 還沒建立成功
* NHRP: The DMVPN spoke router 還沒成功註冊
* Up: 正常運作

### View the NHRP Cache

`show ip nhrp [brief]`



| NHRP Mapping Entry | Description                                                  |
| ------------------ |:------------------------------------------------------------ |
| static             | 在DMVPN上手動新增的 ip nhrp map                              |
| dynamic            | 從NHRP NHS registration request到的                          |
| incomplete         | 當NHRP resolution request 正在運作時，在本地端儲存暫時的條目 |
| local              | 顯示本地mapping 資訊                                         |
| no-socket          | 尚未encryption                                               |
| NBMA address       | transport IP address                                         |

Flags:



| Flag     | Description                                                                      |
| -------- |:-------------------------------------------------------------------------------- |
| Used     | 指過去一分鐘內NHRP mapping有被拿來轉送 packet                                    |
| Implicit |                                                                                  |
| Unique   | 指NHRP mapping 清單不可被其他相同 tunnel IP address 但不同NBMA address的清單複寫 |
| Router   | NHRP mapping清單是從遠端router接收來的                                           |
| Rib      |                                                                                  |
| Nho      |                                                                                  |
| Nhop     | 給remote next-hop 的 NHRP mapping 清單                                           |

### IP NHRP Authentication
The password is stored in plaintext.
```
ip nhrp authentication password
```

### Unique IP NHRP Registration

NHC 會在註冊NHRP時，會要求NHS 保持 NBMA address 的獨特性，不可被其他 IP複寫。
當其他NHC 嘗試註冊相同的NBMA address則會出現錯誤。
該錯誤會發生在spoke為DHCP的狀況
解決方法為NHC 使用該指令:
` ip nhrp registration no-unique`

## Spoke-to-Spoke Communication

### Forming Spoke-to-Spoke Tunnel

![](https://i.imgur.com/OQegqzj.png)



## DMVPN Failure Detection and High Availability

NHRP default hold time:2 hours

### DMVPN Hub Redundancy

[設定可參考](https://networklessons.com/cisco/ccie-enterprise-infrastructure/dmvpn-dual-hub-single-cloud)


# Securing DMVPN

## IPsec
IPsec security architecture:
* Security protocols:AH, ESP
* Security associations:IEK SA(Control plane), IPsec SA(Data plane)
* Key management: IKEv1, IKEv2

[IPsec](https://hackmd.io/8V4ua_xiSWm6bWY8a5FS4g?view#IPsec-Fundamentals)



## IPsec Tunnel Protection

### ISAKMP Pre-Shared Key
1. ISAKMP policy:
```
crypto isakmp policy 1
 encr aes
 authentication pre-share
 hash sha
 group 2
```
2. ISAKMP key:
```
crypto isakmp key 6 DMVPN address 0.0.0.0 0.0.0.0
```
3. IPSEC transform set
```
crypto ipsec transform-set TS esp-aes esp-sha-hmac 
```
4. IPSEC Profile
```
crypto ipsec profile PF
 set transform-set TS 
```

### IKEv2 Pre-Shared key
1. IKEv2 Keyring
```
crypto ikev2 keyring DMVPN-KEYRING-INET
 peer ANY
 address 0.0.0.0 0.0.0.0
 pre-shared-key CISCO456
```
2. IKEv2 Profile
```
crypto ikev2 profile DMVPN-IKE-PROFILE-INET
 match fvrf any
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local DMVPN-KEYRING-INET # IKEv2 Profile
```
3. IPsec transform set
```
crypto ipsec transform-set AES256/SHA/TRANSPORT esp-aes 256 esp-sha-hmac
```
4. IPsec profile
```
crypto ipsec profile DMVPN-IPSEC-PROFILE-INET
 set transform-set AES256/SHA/TRANSPORT
 set ikev2-profile DMVPN-IKE-PROFILE-INET
```

查看狀態:
查看DMVPN狀態
```
R4#show dmvpn detail 
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
N - NATed, L - Local, X - No Socket
T1 - Route Installed, T2 - Nexthop-override
C - CTS Capable
# Ent --> Number of NHRP entries with same NBMA peer
NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface Tunnel0 is up/up, Addr. is 192.168.124.4, VRF "" 
   Tunnel Src./Dest. addr: 10.43.1.4/MGRE, Tunnel VRF ""
   Protocol/Transport: "multi-GRE/IP", Protect "PF" 
   Interface State Control: Disabled
   nhrp event-publisher : Disabled

IPv4 NHS:
192.168.124.1  RE priority = 0 cluster = 0
Type:Spoke, Total NBMA Peers (v4/v6): 1

# Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb    Target Network
----- --------------- --------------- ----- -------- ----- -----------------
    1 10.13.1.1         192.168.124.1    UP 02:03:49     S   192.168.124.1/32

          
Crypto Session Details: 
--------------------------------------------------------------------------------

Interface: Tunnel0
Session: [0x0CBF2718]
  Session ID: 2  
  IKEv2 SA: local 10.43.1.4/500 remote 10.13.1.1/500 Active 
          Capabilities:(none) connid:3 lifetime:23:53:18
  Crypto Session Status: UP-ACTIVE     # 代表正在啟用 IPsec
  fvrf: (none),Phase1_id: 10.13.1.1
  IPSEC FLOW: permit 47 host 10.43.1.4 host 10.13.1.1 
        Active SAs: 2, origin: crypto map
        Inbound:  #pkts dec'ed 105 drop 0 life (KB/Sec) 4271572/3198
        Outbound: #pkts enc'ed 105 drop 0 life (KB/Sec) 4271572/3198
   Outbound SPI : 0x51EA68B2, transform : esp-aes esp-sha256-hmac 
    Socket State: Open
```

查看 IPsec sa狀態

```
R4#show crypto ipsec sa

interface: Tunnel0
    Crypto map tag: Tunnel0-head-0, local addr 10.43.1.4

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (10.43.1.4/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (10.13.1.1/255.255.255.255/47/0) 
   current_peer 10.13.1.1 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 84, #pkts encrypt: 84, #pkts digest: 84
    #pkts decaps: 84, #pkts decrypt: 84, #pkts verify: 84
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 10.43.1.4, remote crypto endpt.: 10.13.1.1
     plaintext mtu 1426, path mtu 1476, ip mtu 1476, ip mtu idb Tunnel0
     current outbound spi: 0x51EA68B2(1374316722)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x562E547C(1445876860)
        transform: esp-aes esp-sha256-hmac ,
        in use settings ={Transport, } # transport mode
        conn id: 5, flow_id: SW:5, sibling_flags 80000000, crypto map: Tunnel0-head-0
        sa timing: remaining key lifetime (k/sec): (4271575/3295)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)  #
```