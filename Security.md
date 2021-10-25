---
tags: CCNP EXAM
---
# 350-401 Security

## Secure Network Access Control
### Network Security Design for Threat Defense
Cisco SAFE:
* Management
* Security intelligence
* Compliance
* Segmentation
* Threat defense
* Secure services

Cisco security solution:
* Cisco Talos: threat intelligence
* Cisco Threat Grid: static file analysis and dynamic file analysis(sandbox)
* Cisco Advanced Malware Protection(AMP)
* Cisco AnyConnect: endpoint software
* Cisco Umbrella: DNS
* Cisco Web Security Appliance(WSA)
* Cisco Email Security Appliance (ESA)


### Network Access Control(NAC)

#### 802.1x

Components:
* Extensible Authentication Protocol(EAP):
* EAP method(EAP type)
* EAP over LAN(EAPoL)
* RADIUS protocol

roles:
* Supplicant: 裝在endpoint上的軟體，透過EAPoL提供艷幟給Authenticator(EX: Cisco AnyConnect)
* Authenticator: Network Acess device(NAD)，如Switch、WLC，將Supplicant來的EAP 封包，封裝成RADIUS 封包給Authentication server
* Authentication server: A RADIUS server 驗證 Client
![](https://i.imgur.com/nqyF99u.png)

802.1X process:
![](https://i.imgur.com/qAqZa0u.png)
1. 當authenticator注意到有一個port開始有資料進來，開始發送EAP-request/identify frames進行authentication process。
2. authenticator 轉送supplicant和autehntication server之間的EAP message，並等待EAP method被選擇
3. 如果驗證成功，authentication server會回傳 RADIUS access-accept message ，並帶上EAP-sussce message和authorization (ACL)，當完成 authenticator 就會開啟該port

##### EAP Methods
* EAP challenge-based authentication method
	* EAP-MD5: 僅使用 MD5 演算法hash驗證，但無法提供雙向驗證。
* EAP TLS authentication method
	* EAP-TLS:使用PKI certificate 驗證機制提供supplicant 和 authentication server雙向驗證
* EAP tunneled TLS authentication methods(outer)
	* EAP Flexible Authentication via Secure Tunneling(EAP-FAST): 與PEAP相似，由Cisco開發取代PEAP，提供更快速的重新驗證並支援faster wirless roaming。EAP-FAST使用protected access credentials(PACs)來支援快速重新驗證，有點像是cookie並存在本機上。
	* EAP Tunneled Transport Layer Security(EAP-TTLS):功能相似於PEAP但不像PEAP廣泛支援，EAP-TTLS 可支援 傳統Password Authentication Protocol(PAP), Challenge Handshake Authentication Protocol(CHAP) 和 Microsoft Challenge Handshake Authentication Protocol(MS-CHAP)
	* Protected EAP(PEAP): 僅只有authentication server需要certificate，減少EAP設定上的管理成本。PEAP會形成加密的TLS tunnel 在supplicant and the authentication server之間。當tunnel建立後，PEAP 會使用一inner mthods 透過outer PEAP TLS tunnel 對supplicant 進行身分驗證:
	
* EAP inner authentication methods
	* EAP Generic Token Card(EAP-GTC): PEAPv1, 由Cisco 建立，作為MSCHAPv2替代方案。
	* EAP Microsoft Challenge Handshake Authentication Protocol Version 2(EAP-MSCHAPv2): PEAPv0，使用username和password或是computer name 和 password，也在微軟的AD服務上使用。 
	* EAP TLS: 因為需要建立PKI，設定較複雜，很少被使用。
  	
EAP Inner authentication methods 在 PEAP, EAP-FAST, EAP-TTLS tunnel 傳送。

##### EAP Chaining

EAP-FAST含有EAP Chaining 選項，可支援單個TLS通道內機器與使用者的驗證。這允許連線公司網路設備的使用者分配權限


#### MAC Authentication Bypass(MAB)
使用endpoint的MAC address進行port-based access control
Process:
![](https://i.imgur.com/N9A6wTq.png)
1. switch 每30秒送出EAPoL identity request message給endpoint(共執行三次)，進行初始化驗證。等待三次timeouts，switch 會辨別出該endpoint 沒有supplicant並透過MAB 進行驗證
2. Switch 會啟用該port並開始MAB 驗證，等待Switch 學習到來源MAC位址。學習到位址後則會先丟棄該endpoint封包，並使用該ednpoint mac位址發出RADIUS access-request message作為識別。RADIUS server收到該message會開始MAC 驗證
3. 等待RAIDUS server確認該裝置可以取得權限，會回傳 RADIUS response 給authenticator並允許該endpoint連線網路，其中也包含dACLs, dVLANs 和 SGT tags在authorization options中
> 若802.1x沒有啟用，則會在port linkup直接運行步驟2
dACLs: Downloadable ACLs
dVLAN: Dynamic VLAN assignment
SGR tags: Security Group Tags tags

#### Web Authentication(WebAuth)

WebAuth 有兩種類型:
* Local Web Authentication
* Centralized Web Authentication with CISCO ISE
#### Enhanced Flexible Authentication (FlexAuth)

FlexAuth可支援多個authentication method同時執行(先前都需要等802.1x執行完才可以)，使client可以更快速的連線。
FlexAuth是Cisco Identity-Based Networking Services(IBNS) 2.0 intergrated solution的其中重要component，提供 authentication, acl and user policy enforcement.


#### Cicso IBNS 2.0

* Enhanced FlexAuth(Access Session Manager)
* Cisco Common Classification Policy Language(C3PL)
* Cisco ISE

#### Cisco TrustSec
TrustSec使用 SGT tags來維護firewall rules and ACLs
使用 SGT tags 進行 ingress tagging and egress filtering 以強制訪問 control policy.

Cisco TrustSec 設定有三個階段:
1. Ingress classification
2. Propagation
3. Egress enforcement

##### Ingress Classification
指派SGT tags給 users, endpoint 或是其它要進入TrustSec network的資源，主有有以下分配方式:
*  Dynamic assignment: 使用802.1x, MAB or WebAuth時將authorization option帶入
*  Static assignment: SGT tags可透過在 SGT-capable網路設備上 statically mapped，以下可以擇一指派:
	* IP to SGT tag
	* Subnet to SGT tag
	* VLAN to SGT tag
	* Layer 2 interface to SGT tag
	* Layer 3 logical interface to SGT tag
	* Port to SGT tag
	* Port profile to SGT tag
SGT capable可以從 Cisco ISE上下載

##### Propagation

Propagation 是將 mappings資訊傳送給 TrustSec 網路設備，並該依據SGT tags執行政策
有兩種可用的 propagating methods:
* Inline tagging
* Cisco-created protcoal SGT Exchange Protocol(SXP)

###### Inline tagging
將SGT tag插入至frame，允許upstream 裝置讀取並應用policy。Native tagging 完全獨立於Layer 3 協定。
Native tagging的缺點是只有ASIC支援TrustSec Cisco裝置可以使用，其他不支援的裝置在接收到該tagged frame則會被丟棄
![](https://i.imgur.com/2vFYd1d.png)
###### SXP propagation
SXP 是 TCP-based peer-to-peer protocl，用在不支援inline tagging 的網路設備。
使IP-to-SGT mappings可以與 non-inilne tagging的裝置溝通。
送出IP-to-SGT bindings裝置稱為 speaker，IP-to-SGT binding receiver 稱為 listener.
SXP 可支援single-hop or multi-hop:

![](https://i.imgur.com/rDKaDft.png)

###### Egress Enforcement

在Classification 和 propagation 執行完後， 則可以開始在 TrustSec 網路中的egress point執行政策

主要分為兩種方法:
* Security Group ACL(SGACL):
![](https://i.imgur.com/mNJJxAd.png)

* Security Group Firewall(SGFW)

SGACL 使用情境如下:
![](https://i.imgur.com/D9qQ5Ru.png)


#### MACsec

MACsec is an IEEE 802.1AE standards-based Layer 2 hop-by-hop encryption method
只有在兩MACsec peers端點上流量進行enrypted，在switch 處理時是沒加密狀態
![](https://i.imgur.com/j13gf8O.png)
所有參與MACsec communications都必須支援MAC中新增加的2個field(802.1AE header, Intergrity Check Value)

有兩種 MACsec 密鑰機制可用：
* Security Association Protocol(SAP): Cisco 專用協定
* MACsec Key Agreement(MKA) protocol








## Network Device Access Control and Infrastructure Security

### Termianl Lines and Password Protection

```
service password-encryption # 密碼加密
```

### Authentication, Authorization, and Accounting(AAA) 

* Authentication:驗證身分
* Authorization:給予權限
* Accounting:追蹤及記錄使用資訊

#### TACASC+

* TCP port 49
* 將authentication, authorixation, accounting 功能獨立
![](https://i.imgur.com/k56Z8no.png)


##### 設定
1. 啟用 AAA function
```
aaa new-model
```
2. 設定TACACS+ server
舊版:
```
tacacs-server host { hostname | host-ip-address } key key-string 
```
新:
```
tacacs server name
address ipv4 { hostname | host-ip-address }
key key-string
```
3. 建立AAA group # optional
```
aaa group server tacacs+ group-name
server name server-name
```
4. 啟用 AAA login authentication # 啟用AAA登入驗證
```
aaa authentication login {LIST NAME} group tacacs+ local
```
5. 啟用 AAA 授權 EXEC 
```
aaa authorization exec {LIST NAME} group tacacs+ local
```
6. 啟用 AAA 授權 console
```
aaa authorization console　# 若是在 line vty 則不用啟用
```
7. 啟用 AAA 授權指令
```
aaa authorization commands {privilege level} { default | custom-
 list-name } method1 
```
8. 授權在global配置模式的權限
```
aaa authorization config-commands
```
9. 啟用 Login accounting
```
aaa accounting exec { default | custom-list-name } method1
```
10. 啟用 command accounting
```
aaa accounting commands {privilege level} { default | custom-list-name }
 method1
```





當AAA server 沒有回應時，登入的AAA 使用者 command authorization 還是會嘗試詢問AAA server。
可使用 if-autbenticated 參數，避免該狀況
```
aaa authorization commands 0 default group ISE-TACACS+ if-authenticated
```
#### RADIUS

* IETF standard AAA protocoal
* support EAP 
* RADIUS needs to return all authorization parameters in a single reply

![](https://i.imgur.com/rzaii2I.png)





### Zone-Based Fireall

### Control Plane Policing (CoPP)
CoPP policy是用來控制Control Plane 封包進入CPU

#### 設定

1. 設定ACL
2. 設定Class map
3. 設定Policy map

##### ACL for CoPP
所有網路流量都必須先定義，透過ACL可以和class map對應

```
ip access-list extended ACL-CoPP-ICMP
 permit icmp any any echo-reply
 permit icmp any any ttl-exceeded
 permit icmp any any unreachable
 permit icmp any any echo
 permit udp any any range 33434 33463 ttl eq 1
!
ip access-list extended ACL-CoPP-IPsec
 permit esp any any
 permit gre any any
 permit udp any eq isakmp any eq isakmp
 permit udp any any eq non500-isakmp
 permit udp any eq non500-isakmp any
!
ip access-list extended ACL-CoPP-Initialize
 permit udp any eq bootps any eq bootpc
!
ip access-list extended ACL-CoPP-Management
 permit udp any eq ntp any
 permit udp any any eq snmp
 permit tcp any any eq 22
 permit tcp any eq 22 any established
!
ip access-list extended ACL-CoPP-Routing
 permit tcp any eq bgp any established
 permit eigrp any host 224.0.0.10
 permit ospf any host 224.0.0.5
 permit ospf any host 224.0.0.6
 permit pim any host 224.0.0.13
 permit igmp any any 
```

##### Class Maps for CoPP
Example:
```
class-map match-all CLASS-CoPP-IPsec
 match access-group name ACL-CoPP-IPsec
class-map match-all CLASS-CoPP-Routing
 match access-group name ACL-CoPP-Routing
class-map match-all CLASS-CoPP-Initialize
 match access-group name ACL-CoPP-Initialize
class-map match-all CLASS-CoPP-Management
 match access-group name ACL-CoPP-Management
class-map match-all CLASS-CoPP-ICMP
 match access-group name ACL-CoPP-ICMP
```

##### CoPP Map Policy
Example:
```
policy-map POLICY-CoPP
 class CLASS-CoPP-ICMP
 police 8000 conform-action transmit exceed-action transmit
 violate-action drop
```

套用:

```
control-plane
 service-policy input POLICY-CoPP
```

### IPv6 First-Hop security (300-410)

#### Router Advertisement(RA) Guard
IPv6 RA 用於IPv6網路環境內，主機設定 stateless DHCP設定會向網路內的router 發出 Solicitaion，再由router 回覆Advertise
```
ipv6 nd raguard attach-policy
```
#### DHCPv6 Guard
DHCPv6 Guard can block DHCPv6 reply and advertisement messages
```
ipv6 dhcp guard attach-policy 
```
#### Binding Table
顯示 連接的IPv6 Neighbors 資訊，　包含link-layer address

#### IPv6 ND Inspection/ IPv6 Snooping
建立binding table 

```
ipv6 snooping policy
```
#### Source Guard
Layer 2 snooping interface, 驗證 IPV6來源流量, 如果介面無法辨識該流量位指, 則會封鎖
類似於 mac address security

```
ipv6 source-guard 
```

### uRPF (300-410)

#### Introduction
Unicast Reverse Path Forwarding: 用於讓路由器檢測 spoofed addres(欺騙位址)
啟用在Router上，驗證封包來源IP位址，若未指不符則丟棄該封包
uRPF主要有三種模式:strict, loose and VRF
主要討論strict和loose

* Strict mode:
Router 會使用同一個Interface去接收以及回覆封包，EX:
![](https://i.imgur.com/8YNHXRY.jpg)
該模式可能會使正常的流量被丟棄，尤其是在非同步的routing path的狀況下。

Strict模式在企業網路中，可以被使用在access layer或是branch office。

* Loose mode:
在Loose模式下，source address必須出現在routing table上(allow-default)。
也可以使用acess list允許範圍。
另外如果source address在路由表上egress是Null interface，則會被丟棄。

Loose模式在企業網路中，可以使用在與default route連接的uplink。


#### Example

使用uRPF功能，必須先啟用CEF

Example:
```
interface gi 0/0
ip verify unicast source reachable-via {rx | any} [allow-default]
[allow-self-ping] [list]
```
rx: strict
any: loose

驗證:

```
Router#show cef interface gigabitEthernet 0/1
GigabitEthernet0/1 is up (if_number 3)
  Corresponding hwidb fast_if_number 3
  Corresponding hwidb firstsw->if_number 3
  Internet address is 10.14.1.1/24
  ICMP redirects are always sent
  Per packet load-sharing is disabled
  IP unicast RPF check is enabled
  (略 ...)
```