
# IPv6 

## IPv6 Refresher

### Addressing

2001:0000:0000:00A1:0000:0000:0000:1E2A
=> 2001:0:0:A1::1E2A 


:: 只能出現一次，否則不曉得中間有多少位元

#### Address Architecture

* Unicast
* Multicast
* Anycast

##### Unicast

Generate an interface ID in different ways:
* Layer 2 address產生EUI-64格式的位址(link local用)
* Stateless Address Autoconfiguration(RFC 3041)
* DHCPv6
* Manual configuration
* Cryptographically Generated Addresses(CGAs)ㄝ, RFC3972, 使位址提供安全以及驗證

Unicast address 有以下分形式:
* The link-local scope(Layer 2 Domain): 
Layer 2 domain，位址內會含有MAC位址，前面16位元固為FE80
![](https://i.imgur.com/8yV5by7.png)

* The unique-local scope(Organization): 
IPv6私人位址，類似與IPv4 RFC 1918，不會被路由至外網
位址範圍: fc00::/7
![](https://i.imgur.com/iO6QLbG.png)


* The global scope(Internet):
可路由至外網的IP範圍，由IANA分發，位址 2000:: ~ 3FFF::

![](https://i.imgur.com/aF1agiD.png)

Special-Use Address:
* Unspecified address
用在沒有IPv6位址的裝置，位址為:: 
* Loopback address
位址為::1
* IPv4-compatible IPv6 address
此方法已經不使用
* IPv4-mapped IPv6 address
用於表示IPv4 node，位址為 ::FFFF:IPv4
ex: ::FFFF:192.168.100.10.1 => ::FFFF:C064:0A01

##### Anycast 

Anycast多使用在重要的網路資源(DNS root server, web, multicast RPs)
Anycast address format:
![](https://i.imgur.com/zaTbBSJ.png)


##### Muticast

![](https://i.imgur.com/TUrhtiH.png)
* T bit: 設1為非永久性的multicast address
* P bit: built based on a unicast prefix
* R bit: contain the RP address

IPv6 Multicast Scopes:
![](https://i.imgur.com/dBjTWXo.png)


###### Unicast-Prefix-Based Multicast Address
提供globally unique IPv6 multicast adresses

![](https://i.imgur.com/PwODexY.png)



Example:
Unicast Prefix: 2001\:100\:abc:1::/64
Unicast Prefix Scope Global: E
Choose Group ID: 11FF:11EE
Resulting Multicast Address: FF3E\:0040:2001\:100:abc\:1:11FF:11EE

###### Solicited-Node Multicast Address

Solicited-Node multicast address提供當local-link 上的host只知道他自己的 L3 unicast address

![](https://i.imgur.com/ZM3fd1s.png)

### Packet Format

![](https://i.imgur.com/WQIxDkQ.png)
* Flow label: 用於DiffServ, IntServ and RSVP2
* Next Header: 用來指向延伸的Header(upper-layer protocol)


與IPv4相比:
* Fixed length for the basic header
* Fragmentation is done only by the traffic source:
IPv6 使用Path MTU(PMTU) Discovery. IPv6的MTU不能小於1280bytes
* Header checksums are eliminated
在Layer 2 CRC以及Layer 4都有位元檢查，因此在IPv6 則把header checksum拿掉，增加路由效率

#### IPv6 Extension Headers

![](https://i.imgur.com/LioSCXg.png)

Extension Header:
* Hop-by-Hop Options Header
value of 0, 用於指定傳送參數到目的地路徑上每個router，它告知router必須對封包的內容做進一步處理
![](https://i.imgur.com/8LRv9o3.png)

每個node收到Hop-by-Hop 擴充表頭會進行該選項，若該選項無法理解，就會發出ICMP error message 給來源

* Destination Options Header
value of 60, 於目的節點需要額外處理的封包選項訊息，目的節點可能是中間目的主機（即routers）或者最後目的主機。
* Routing Header
value of 43, IPv6 發送端用來指定路由，將路徑上必須拜訪的主機列入清單
Routing Header 內含有Type field，有以下兩種:
	1. Type 0, RFC 2460，標頭裡含有必須拜訪的主機清單
	2. Type 2, RFC 3775，使用MIPv6，裏頭包含一個home address的unicast address，並啟用相對應的節點直接傳送給mobile node。
* Fragment Header
為了不讓Router 處理Fragment，在傳送封包前，來源端會先進行PMTU Discovery。 如果封包大小仍超出MTU時仍然需要切分，目的端可以透過ID進行重組

* Authentication Header
IPsec Authentication Header, 使用方法與IPv4相同
* Encapsulating Security Payload Header
IPsec ESP header. 使用方法與IPv4相同
* Mobility Header
用來mobile nodes溝通

![](https://i.imgur.com/Skqduap.png)

### ICMP for IPv6

![](https://i.imgur.com/5CRK5Sh.png)
Header value is 58
SA位址是用Source address selection algorithm計算出來

#### ICMP Error Message

* Destination Unreachable

Error code:
| Code | Short Description                                          | Explanation                             |
| ---- |:---------------------------------------------------------- | --------------------------------------- |
| 0    | No route to destination                                    | 因為沒有目的路由，所以封包被丟棄        |
| 1    | Communication with destination administratively prohibited | Filters blocked the packet              |
| 3    | Address unreachable                                        | Link-layer 位址無法被解析               |
| 4    | Port unreachable                                           | 目的TCP 或 UDP port不正確或目的地不存在 |


* Time Exceeded
Hop count 達到0則會回傳
* Packet Too Big
IPv6 packet-fragmentation process有所不同，不會讓每個Router 進行fragment，會使佔用太多資源
因此會需要一個feedback機制讓Router 通知來源封包在前往目的地路徑上需要fragment
PMTU Discovery 就是使用ICMP Packet Too Big error message來確認MTU大小

* Parameter Problem

| Code | Short Description                         | Explanation                                         |
| ---- |:----------------------------------------- | --------------------------------------------------- |
| 0    | Erroneous header field encountered        | The field pointed by the Pointer field is in error. |
| 1    | Unrecognized next header type encountered | The next header is not recognized.                  |
| 2    | Unrecognized IPv6 option encountered      | The IPv6 option is not recognized.                  |

#### Source Address Selection Algorithm

一個IPv6的節點通常會擁有多個unicast address，個別有不同的scope(link-local, global)
Dual-stack node甚至需要從IPv4或IPv6選擇。
SAS在MIPv6和VPNv6會詳細說明
![](https://i.imgur.com/wOWwwAx.png)

### Neighbor Discovery Protocol


IPv6 NDP Feature counterpart in IPv4:

| IPv6 ND Feature                                  | Short Description                                                                             | IPv4                                             |
|:------------------------------------------------ |:--------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| Router discovery                                 | Enable hosts to locate routers on attahed links                                               | ICMP Router Discovery, Routing protocol snooping |
| Prefix discovery                                 | learn prefixes                                                                                | not available                                    |
| Parameter discovery                              | learn parameters such as link MTU or hop limit                                                | PMTU discovery                                   |
| Address autoconfiguration                        | configure automatically an address                                                            | not available                                    |
| Address resolution                               | determini link-layer address for on-link destination                                          | ARP                                              |
| Next-hop determination                           | determine the next hop for a given destination                                                | ARP cache for on-link prefixes, Default router   |
| Neighbor Unreachability                          | detect that a neighbor is no longer reachable                                                 | Dead Gateway Detection                           |
| Duplicate address detection                      | determine addresses already in use                                                            | ARP with source=0                                |
| Redircet                                         | Enables routers to inform hosts of a better next hop on the link for a particular destination | ICMP Redirect                                    |
| Default router and more-specific route selection | Enable routers to inform multihomed hosts of better default routers                           |                                                  |
| Proxying nodes                                   | Accepts packets on behalf of other nodes                                                      | Proxy-ARP                                        |

Host and Router Roles with Regard to NDP:

![](https://i.imgur.com/1MBsDPv.png)

IPv6 ND是整個包在ICMPv6裡面，與IPv4不同(ARP, RARP都還在L3)
這使得可以方便提供security, multicast等功能
詳細文件可查看RFC 3971 

NDP 有以下message:
* Router solicitation (RS)
* Router advertisement (RA)
* Neighbor solicitation (NS)
* Neighbor advertisement (NA)
* Redirect
* Inverse neighbor solicitation (INS)
* Inverse neighbor advertisement (INA)

##### Comparison with IPv4
*  Router discovery 整合成IPv6內一部份功能，使host可以辨別他們的預設路由器
*  ND message含有額外的資訊(如MTU或link-layer address)，減少在link上交換的次數，舉例:
	- Router的Link-layer address 含著RA message，使所有在link上的節點都能收到
	- 目的link-layer address被插入至redirect message, 節省接收方需要額外adress-resolution
	- RA裡攜帶的MTU使所有node都使用一致的值
* IPv6 使用multicast groups(solicited-node multicast address)來address resolution，與IPv4 使用boradcasting不同。使IPv6大量主機的子網更易於管理。
* 一些新功能如address autoconfiguration and neighbor unreachability detection，簡化設定並改善封包傳遞的穩定性
* Router advertisements and Redirect message 攜帶路由器link-local的位址，使host與Roouter關聯更加穩定。在IPv4裡，host會因為不同網路的關係需要更改default gateway.
* IPv6 可以替ND message增加IP驗證及安全性，在IPv4中是沒有替ARP提供該功能的


##### Router and Prefix Discovery

router solicaitation(RS) and router advertisement(RA) 作為 router and prefix discovery
RA會自己定期發出給all-nodes，link local scope multicast address(FF02::1)，其內容含以下資訊:
* Default router的候選名單
* Autoconfiguration 的參數
* on-link上的prefixe

##### Address Resolution

NS和ND封包用來表示幾個重要的node operations:
* Link-layer address resolution
* Duplicate address detection(DAD)
* Neighbor unreachability detection(NUD)

##### Redirecting a Host to a Better Next Hop

![](https://i.imgur.com/LhFeY0m.png)

##### Neighbor Discovery Algorithms

###### Next-Hop Determination

![](https://i.imgur.com/9zRyhJU.png)

###### Default Router Selection

為特定主機的演算法，Router只依據routing protocol做出next-hop決定
IPv6不需要定義default gateway，但定義三種類型的host:

* Type A
Host無視defaut router，並在RA中的route資訊列出特定路由器清單，並執行router selection alogorithm選出路由器(主要是round robin)。
* Type B
與A相同，在RA中收到的default router清單依據權重排列(low, medium, high)，並依據他的權重來選擇
* Type C
Host載入routing table，當他們接收到有Route Information出現次數的選項，他們會載入default route(::/0)並對此router發出RA，當確定好Next-Hop並查詢off-link destination routing table時，會先前往reachable routers，再透過最常匹配prefix，最後使用router-prefernece

###### Neighbor Unreachability Detection

###### The State Machine for Reachability
neighbor cache 維持最近以傳輸過的neighbor清單
![](https://i.imgur.com/NUtXsWf.png)

* NA1—Receiving an NA with Solicited=0
* NA2—Receiving an NA with Solicited=1
* NA3—Receiving an NA with Solicited=1 and
	- Override=1 or
	- Override=0 and link-layer same as cached
* NA4—Receiving an NA with Solicited=1, Override=0, and link-layer different from cached
* NA5—Receiving an NA with Solicited=0, Override=1, and link-layer different from cached
* O—Receiving other ND packets, such as NS, RS, RA, redirect, with link-layer different from cached
* S—Sending packet
* T—Timeout (Note that each state has a timer, started when entering the state.)
* Te—Timeout with retries exhausted
* U—Upper-layer reachability confirmation

###### Autoconfiguration

使用NDP中的RA獲取prefix資訊並建立address，並在透過NS和NA去測試該位址是否已經在使用。

###### Summary
![](https://i.imgur.com/7Tu0npm.png)
![](https://i.imgur.com/6UTVNHJ.png)


## Delivering IPv6 Unicast Services

### IPv6 Provisioning

#### Stateless Autoconfiguration
根據Router的RA來獲得autoconfiguration所需資訊，需要以下三步驟:
* Discover a prefix used on the link
* Generate an interface ID
* Verify the uniqueness of the generated IPv6 address

#### Stateful DHCP

IPv4 DHCP 與 IPv6有些差異:
* DHCP-DISOVER and DHCP-OFFER 在IPv6不再使用，改使用DHCP-SOLICIT (c->s)和DHCP-ADVERTISE(s->c)
* IPv6 DHCP 內位址是由message option提供
* IPv6 host可向DHCP server一次取得多個位址

![](https://i.imgur.com/er6UoJn.png)

#### Router IPv6 Address Provisioning: Prefix Delegation

Prefix Delegation(PD)，使DHCP server直接發出prefix給Client，可用在ISP與Customer之間。
在IPv4網路內，ISP需要給Customer一個public IP，而Customer迫使內部環境使用NAT。
而該IP會是由ISP透過信件等方式告知Customer，再讓Customer自行設定。
若改以IPv6 PD方式，可直接自動分發給Customer，減少人為疏失。
可參考RFC 3769

![](https://i.imgur.com/QtDcrgE.png)
在PD protocol架構中，CE稱為Requesting Router(RR)，PE稱為Delegating Router(DR)

![](https://i.imgur.com/M0MIzbJ.png)

##### Requesting Router

RR 設置:
```
hostname CE
ipv6 unicast-routing
!
interface Serial2/0
 ! Upstream interface—ISP
 ipv6 address autoconfig default
 ipv6 dhcp client pd genpfx-foo
!
interface Ethernet0/0
 ! Downstream 1
 ipv6 address genpfx-foo 0:0:0:1::1/64
!
interface Ethernet1/0
 ! Downstream 2
 ipv6 address genpfx-foo 0:0:0:2::1/64
```

```
CE#show ipv6 interface Ethernet0/0
Ethernet0/0 is up, line protocol is up
 IPv6 is enabled, link-local address is FE80::A8BB:CCFF:FE02:8A00
 Global unicast address(es):
 2001:DB8:FF01:1::1, subnet is 2001:DB8:FF01:1::/64 [PRE]
 valid lifetime 2591041 preferred lifetime 603841
CE#show ipv6 interface Ethernet1/0
Ethernet1/0 is up, line protocol is up
 IPv6 is enabled, link-local address is FE80::A8BB:CCFF:FE02:8A01
 Global unicast address(es):
 2001:DB8:FF01:2::1, subnet is 2001:DB8:FF01:2::/64 [PRE]
 valid lifetime 2591005 preferred lifetime 603805
```

##### Delegating Router

DR 如從他可以派送的prefixe中獲取大量的address block? 有以下操作:
* Manual configuration with static binding
* Using a prefix pool
* Interaction with AAA
* DHCP

設定:
```
ipv6 unicast-routing
!
ipv6 dhcp pool pd-pool
 prefix-delegation pool pfx-pool
 dns-server 2001:DB8:100::1
!
ipv6 local pool pfx-pool 2001:DB8:FF00::/40 48
!
interface Serial2/0
 description Yet another customer
 ipv6 address 2001:DB8::1/64
 no ipv6 nd suppress-ra
 ipv6 dhcp server pd-pool
```

DHCP-PD 僅適合用在PE和CE之間作為delegating prefix的解決方法。

### IPv6 Network Access
![](https://i.imgur.com/xqmlUvs.png)


#### Native IPv6 Access

* Routed access
IPv6 packet are encapsulated in an Ehternet frame
* Bridge access
Permanent Virtual Circuit
![](https://i.imgur.com/1IFOtd5.png)
CPE: Customer premises equipment
* PPP encapsulated IPv6
![](https://i.imgur.com/QSseBLm.png)

#### Access over Tunnel

IPv6 over IPv4 tunnels 可以被用在各種類型網路，有以下原因會使用到他們:
* 快速又不花費的方法提供IPv6連線
* 提供跨越網路(中間非能管控到)傳送IPv6 traffic解決方法
* 在IPv6不支援等因素無法部屬的環境內，使其可以傳輸

##### Manaulaay Configured Tunnel
MCT 是第一個被發展出來的傳輸機制
MCT 是靜態設定提供point-to-point tunnel
![](https://i.imgur.com/TYDf473.png)

在Cisco IOS中，tunnel interface會自動關閉RA功能，若要重啟動輸入以下指令:
```
no ipv6 nd suppress-ra
```

##### Tunnel Broker and Tunnel Server

Tunnel Broker可以使在host上tunnel自動設定，使得各容易擴充
Tunnel server是Tunell Broker額外的功能，兩者都是提供擴充MCT的解決方法

###### Teredo

Teredo 是用來解決NAT導致IPv6 over IPv4 tunneling的問題，端點不需要Public IPv4位址即可穿隧。
It provides address assignment and hostto-host, automatic tunneling for unicast IPv6 connectivity when hosts are located behind IPv4 NAT(s)
使用IPv4 UDP port 3544

Teredo對provider不是常用的解決方法，且會喪失IPv6的特性


###### ISATAP
The Intra-Site Automatic Tunnel Addressing Protocol (ISATAP) specified in RFC 4214
ISATAP encapsulates IPv6 in IPv4 using ip-protocol 41

ISATAP format inteface ID:
first 32 bits: 0000:5EFE
```
IPv4 Address 200.15.15.1
IPv4 Address in hexadecimal format C80F:0F01
ISATAP-format Interface ID 0000:5EFE:C80F:0F01
```

![](https://i.imgur.com/mUQY8H5.png)

### IPv6 over the Backbone

Backbone network is single IPS infrastructure.
有以下方法在backbone上部屬IPv6:
* Daul-stack routers and links
* IPv6(adn IPv4) use a separate link layer
一些backbones使用layer 2技術(Frame Relay, ATM等)在edge router傳送IPv4流量。
為了建立IPv6連線，ISP router同樣可使用Layer 2架構，但需要與IPv4分隔出來
* IPv6 on the edges, and tunnels to traverse the backbone

#### IPv6 over IPv4 Tunnels

* IPv6 over GRE
![](https://i.imgur.com/EAtEIM4.png)
MCT一種，GRE是點對點的tunnel 技術，也因此難以擴充
* 6to4
6to4是一種automatic tunneling機制，RFC3056
![](https://i.imgur.com/XaaPdau.png)


### Translation Mechanisms(NAT-PT)

NAT-PT enables IPv6-only to communicate with IPv4-only

Header Translation IPv4 to IPv6:

| IPv6 Header Fields | Values                     |
|:------------------ | -------------------------- |
| Version            | 6                          |
| Traffic Class      | TOS                        |
| Flow Label         | 0                          |
| Payload Length     | Total length—header length |
| Next Header        | Protocol                   |
| Hop Limit          | TTL                        |
| Src address        | NAT-PT                     |
| Dst address        | NAT-PT                     |

IPv6 to IPv4:

| IPv6 Header Fields | Values                                      |
|:------------------ | ------------------------------------------- |
| Version            | 4                                           |
| IHL                | 5(no options)                               |
| TOS                | Traffic class                               |
| Total Length       | Payload length + length of the IPv4 header  |
| Identification     | 0                                           |
| Flasgs             | More Fragments Flag=0 Don’t Fragment Flag=1 |
| Fragment Offset    | 0                                           |
| TTL                | Hop Limit                                   |
| Protocol           | Next Header field                           |
| Src address        | NAT-PT                                      |
| Dst address        | NAT-PT                                      |
| Checksum           | Computed                                    |
| Options            | Not translated                              |

![](https://i.imgur.com/k0hBayw.png)

NAT-PT router config:
```
interface Ethernet0/0
 ipv6 address 3ffe:100:200:1::2/64
 ipv6 enable
 ipv6 nat
!
interface Ethernet1/0
 ip address 192.168.1.2 255.255.255.0
 ipv6 nat
!Entry for static mapping of v4 source->v6-
ipv6 nat v4v6 source 192.168.1.1 3ffe:b00:1::1
!Entry for mapping of v6 source->dynamic v4
ipv6 nat v6v4 source list pt1 pool v4pool
ipv6 nat v6v4 pool v4pool 10.50.10.1 10.50.10.10 prefix-length 24
ipv6 nat translation udp-timeout 600
ipv6 nat prefix 3ffe:b00:1::/96
!
ipv6 access-list pt1
 permit ipv6 3ffe:100:200:1::/64 any
```

NAT-PT 有許多如同NAT一樣的問題，因此若有其他解決方案則不會使用NAT-PT

