
# BGP 

## BGP Fundamentals
Network Layer Reachability Information(NLRI): 一種含有 network prefix, prefix length 和任何特定的BGP PAs路由資訊的路由更新

### Autonomous Sytem Numbers (ASNs)

* 32 bits
* private ASNs: 64,512 to 65,535 and 4,200,000,00 to 4,294,967,294
### BGP session
Support Multi-hop sessions
BGP session are categorized into two types:
* iBGP:same AS
* eBGP:different AS

### Path Attributes

* Well-known manadatory: Include prefix advertisement
* Well-known discretionary: May or may not be included prefix advertisement
* Optional transitive
* Optional non-transitive

### Loop Prevention

AS_Path is used as a loop-prevention maechanism.

![](https://i.imgur.com/n9BHETo.jpg)


### Address Families

Every address family maintains a separate database and configuration for each protocol(address family plus sub-address family)
BGP incloudes an address family identifier(AFI) and subsequent address family identifier(SAFI) with every route advertisement to differentiate between theAFI and SAFI database.

### Inter-Router Communication

* BGP cannot discover neighbors dynamically
* Use TCP port 179 to communicate with other routers.
* BGP can support multi-hop neighbors
![](https://i.imgur.com/uZm5Cdg.jpg)

### BGP Messages

Packet Types:


| Type | Name         | Functional Overview                            |
| ---- | ------------ |:---------------------------------------------- |
| 1    | OPEN         | Sets up and establishes BGP adjacency          |
| 2    | UPDATE       | Advertise, updates, or withdraws routes        |
| 3    | NOTIFICATION | Indicates an error condition to a BGP neighbor |
| 4    | KEEPALIVE    | Ensures that BGP neighbors are still alive     |

#### OPEN
BGP OPEN message contains: the BGP version number, the ASN of the originating router, the hold time, the BGP identifier and other optional parameters that establish the seesion capabilities:

* Hold time: 收到UPDATE 或 KEEPALIVE hold timer 會重至為初始值，當 Hold time變為0 則會移除BGP neighbor，預設為 180秒
* BGP identifier: 使用32bit Router ID 辨識 BGP　router，RID 可使用在 loop-prevention mechanism，RID可手動或自動設定
#### KEEEPALIVE
利用每三分之一的hold timer時間交換KEEPALIVE message 

#### UPDATE
UPDATE meesage 包含了NLRI，可以更新BGP Router路由表
UPDATE 也像KEEPALIVE一樣能夠使Hold time初始化

#### NOTIFICATION
NOTIFICATION 用來告知 BGP session 上出現ERROR，如hold timer expiring, neighbor capabilites changing, or a BGP session reseet being requested等造成BGP connection關閉的原因


### BGP Neighbor States

BGP 使用 finite-state machine(FSM) 來維護整個BGP peers和運作狀態，BGP session可為以下六種狀態:

#### Idle
BGP 檢測start event並初始化和BGP peer TCP connection。當有錯誤導致BGP 第二次回到Idle時，會啟動ConnectRetry timer，60秒檢測一次。
#### Connect
當BGP initiates 完成TCP 三向交握連線，並開始建立 BGP session 送出OPEN message給neighbor，狀態變為OpenSent. 如果ConnectRetry timer在該Stage完成前先耗盡時，則會嘗試建立新的TCP connection, ConnectRetry timer初始化, 狀態也改為Active。
#### Active
BGP會持續嘗試建立TCP connection，值到BGP peer回應，但若TCP connection失敗狀態則會變回 Connect.
#### OpenSent 
OPEN Message 以傳送出去等待其他router回覆，當收到OPEN message時會檢查以下訊息:
* BGP version
* Source IP must math the IP address that is configured for the neighbor
* AS number must match
* BGP identifiers must be unique
* Security paramenter(password, TTL)

若以上資訊無誤，則會發出KEEPALIVE，state移至OpenConfirm。
如果有錯誤的話則發送NOTIFICATION 並狀態回到Idle
#### OpenConfirm
該狀態BGP router等待對方的 KEEEPALIVE or NOTIFICATION。
如果是KEEPALIVE，狀態變為Established。弱勢NOTIFICATION則回到Idle
#### Established
Established狀態代表已完成BGP session，BGP neighbors開始透過UPDATE message交換路由。如果hold timer 過期則又會回到Idle

![](https://i.imgur.com/EfxB45g.jpg)

## Basic BGP Configuration

在設定以前，最好以模組化的方式進行設定。
BGP router設定需要以下組件:
* BGP session parameters:ASN, authenication, keepalive timers, src and dst IP
* Address family initialization
* Activation of the address family on the BGP peer

BGP 設定步驟:
1.  初始化BGP prcess
```
Ruter(config)#rotuer bgp {as-number}
```
2. 設定BGP RID(optional)

```
Ruter(config-route)#bgp router-id {router-id}
```
3. 設定neighbor IP address and ASN
```
Ruter(config-route)#neighbor {ip-address} remote-as {as-number}
```
4. 設定 BGP session的source interface(optional)
```
Ruter(config-route)# neighbor {ip-address} update-source {interface-id}
```
5. Enailbe BGP authentication(Optional)
```
Ruter(config-route)# neighbor {ip-address} password {password}
```
6. 修改BGP timer(optional)
```
Ruter(config-route)#neighbor {ip-address} timers {keepalive holdtime} {min-holdtimer}
```
7. address family初始化
```
Ruter(config-route)#address-family {afi} {safi}
```
afi: ipv4, ipv6, l2vpn, etc.
safi: unicast, multicast, etc.
8. 啟用 addres family
```
Ruter(config-route)#neighbor {ip-address} activate
```
預設啟用IPv4和IPv6

### Verification of BGP session

![](https://i.imgur.com/fLs272V.jpg)



| Field    | Description                                             |
| -------- |:------------------------------------------------------- |
| Neighbor | BGP peer's IP address                                   |
| V        | BGP peer's version                                      |
| AS       | BGP peer's ASN                                          |
| MsgRcvd  | Count of mesg recived                                   |
| MsgSent  | Count of mesg sent                                      |
| TblVer   | Last version of the BGP database send to the peer       |
| InQ      | number of messages queued to be processed from the peer |
| OutQ     | number of messages queued to be send to the peer        |


```
show bgp {afi} {safi} neighbors {ipaddress}
```
Get the BGP neighbor session state, timer and other essential peering information.

### Prefix Advertisement

* Adj-RIB-in: 從neighbor送來的route暫存地方
* Loc-RIB: The actual routing information the router uses, developed from Adj-RIBs-In.
* Adj-RIB-out: The information the router chooses to send to neighboring routers

![](https://i.imgur.com/TAjLy9N.jpg)

install network prefixes int the BGP Loc-RIB
```
network {network} mask {subnet-mask} [route map {route-map-name}]
```
設定完network statement後, BGP 會開始搜尋 global RIB找到匹配的prefix, 可以是connected network 或是透過routing protocol的紀錄。
找到後會將其加入至 BGP Loc-RIB。
在loc-RIB 的route 都會被傳至bgp peer
所有route發布都會經過以下步驟：
1. 檢查NRLI可以在global RIB解析，如果不行則不會進行後續處理
2. 進行特定outbound policies檢查，如果無問題則會放到Adj-RIB-Out
3. 最後將NLRI 發布至BGP peer

![](https://i.imgur.com/zHoYp03.png)

BGP database 完整 processing:
![](https://i.imgur.com/RyN0PLd.png)

Display the contents of the Loc-RIB table
```
show bgp {afi} {safi}
```

## BGP session Types and Behaviors

* iBGP: Assign an AD of 200
* eBGP: Assign an AD of 20

### iBGP

iBGP通常使用在AS內有多個multiple routing policies 或傳送AS路由時所需:
![](https://i.imgur.com/A3cyj7v.png)

以上圖為例:
將R2與R4設定iBGP Peering，交換彼此連接的AS route。
但這樣設定R3也沒辦法路由至R1和R5。
也可以透過Redistributing 解決問題，但需要考量到以下幾點:
* Scalability: 在實際網路狀況下，IGP 沒辦法容下那麼大量的路由
* Custom routing
* Path attributes: 無法有由IGP Protocol管理，只有BGP能夠維護 path attribute

因此建議可將所有routers 建立 iBGP seesion(R2, R3 and R4)，建立成iBGP full mesh.


#### iBGP Full Mesh Requirement

AS_Path 擁有預防loop的機制，但只有在eBGP session上。在iBGP中並不會將ASN含在裡面。
在RFC 4271中，禁止NLRI 從一個iBGP peer傳至另一 iBGP peer，如下圖:

![](https://i.imgur.com/WWMUmuE.png)

為了解決該問題，可以透過以下方式:
![](https://i.imgur.com/DBEERvB.png)

#### Peering Using Loopback Address
BGP session 預設會使用primary IP address作為預設BGP ID。
但發生連線問題時，容易導致network connectivity lose，即便有其他path能抵達目的，如下圖:
![](https://i.imgur.com/xwi4g2a.png)

因此建議每台BGP router設定Loopback interface，並將其位置用IGP protocal宣告，若發生該狀況也能透過其他iBGP路徑前進。
設定指令:

```
neighbor {ip-address(通常是對方的iBGP)} update-source {interface-id}
```

### eBGP

eBGP有以下幾點和iBGP不同:

* TTL只設定1
* eBGP廣播的路由器修改了 BGP net hop IP位址
* eBGP廣播的路由器將ASN置放於AS_Path
* eBGP接收的Router會檢查收到的AS_Path內有無相同的ASN，若有則會將該NLRI丟棄

#### eBGP 和 iBGP 拓樸

相同AS內可為確保每個路由器皆會接收到路由(因從iBGP peer學到的路由不會轉送到另一個iBGP)，可嘗試以下作法:
* 建立Full mesh iBGP(但規模一大建立連線數就會翻倍)，可用Route Reflector解決
* 用IGP協定(OSPF、EIGRP)將同AS路由串起來

若是以BGP作為iBGP:
```
neighbor {ip-address} next-hop-self
```
可將其路由已修改next-hop傳給Neighbor

##### Route Reflector 

![](https://i.imgur.com/H2uCGFj.png)

因從iBGP peer學到的路由不會轉送到另一個iBGP。
若要R3學到R1的路由，在R2設定Route Reflector 設定
```
neighbor 10.12.1.1 route-reflector-client
```
![](https://i.imgur.com/3xiMoyu.png)


Route Reflector 使用兩屬性避免迴圈:
* ORIGINATOR_ID
* CLUSTER_LIST

```
R4# show bgp ipv4 unicast 10.1.1.0/24
! Output omitted for brevity
Paths: (1 available, best #1, table default)
Refresh Epoch 1
Local
10.12.1.1 from 10.34.1.3 (192.168.3.3)
Origin IGP, metric 0, localpref 100, valid, internal, best
Originator: 192.168.1.1, Cluster list: 192.168.3.3, 192.168.2.2
```

##### Confederation
RFC 3065，iBGP full mesh替代方案
Confederation 包含 sub-AS，稱為member ASs

![](https://i.imgur.com/ySkiw2L.png)

設定步驟:
1. route bgp {member-asn}
2. bgp confederation identifier {as-number}
3. bgp confederation peers {member-asn}  只有一Router 與其他AS 直接連接(如R2)

Confederation 從 iBGP session 分享狀態至 eBGP session，有以下幾點不同:

* AS_Path attribute 包含了subfield AS_CONFED_SEQUENCE. 只用於預防loop 但不拿來選擇最短AS_Path
* Route reflector 可以在member AS正常使用
* BGP MED、LOCAL_PREF  屬性也會傳給其他的member AS
* Next-hop address 當member ASs的router在進行交換時，不會被做改變
* 當route被宣告至confederation 以外的地方，AS_CONFED_SEQUENCE 就會被移除

```
R1-AS100# show bgp ipv4 unicast | begin Network
 Network Next Hop Metric LocPrf Weight Path
 *> 10.1.1.0/24 0.0.0.0 0 32768 ?
 *> 10.7.7.0/24 10.12.1.2 0 200 300 i
* 10.12.1.0/24 10.12.1.2 0 0 200 ?
 *> 0.0.0.0 0 32768 ?
 *> 10.23.1.0/24 10.12.1.2 0 0 200 ?
 *> 10.25.1.0/24 10.12.1.2 0 0 200 ?
 *> 10.46.1.0/24 10.12.1.2 0 200 ?
 *> 10.56.1.0/24 10.12.1.2 0 200 ?
 *> 10.67.1.0/24 10.12.1.2 0 200 ?
 *> 10.78.1.0/24 10.12.1.2 0 200 300 ?
```
```
R2# show bgp ipv4 unicast | begin Network
 Network Next Hop Metric LocPrf Weight Path
 *> 10.1.1.0/24 10.12.1.1 111 0 100 ?
 *> 10.7.7.0/24 10.67.1.7 0 100 0 (65200) 300 i
 *> 10.12.1.0/24 0.0.0.0 0 32768 ?
 * 10.12.1.1 111 0 100 ?
 *> 10.23.1.0/24 0.0.0.0 0 32768 ?
 * 10.25.1.0/24 10.25.1.5 0 100 0 (65200) ?
 *> 0.0.0.0 0 32768 ?
 *> 10.46.1.0/24 10.56.1.6 0 100 0 (65200) ?
 *> 10.56.1.0/24 10.25.1.5 0 100 0 (65200) ?
 *> 10.67.1.0/24 10.56.1.6 0 100 0 (65200) ?
 *> 10.78.1.0/24 10.67.1.7 0 100 0 (65200) 300 ?
Processed 8 prefixes, 10 paths
```
```
R4# show bgp ipv4 unicast 10.7.7.0/24
! Output omitted for brevity
BGP routing table entry for 10.7.7.0/24, version 504
Paths: (2 available, best #1, table default)
 Advertised to update-groups:
 3
 Refresh Epoch 1
 (65200) 300
 10.67.1.7 from 10.34.1.3 (192.168.3.3)
Origin IGP, metric 0, localpref 100, valid, confed-internal, best
Originator: 192.168.2.2, Cluster list: 192.168.3.3
rx pathid: 0, tx pathid: 0x0
 Refresh Epoch 1
 (65200) 300
 10.67.1.7 from 10.46.1.6 (192.168.6.6)
Origin IGP, metric 0, localpref 100, valid, confed-external
rx pathid: 0, tx pathid: 0
```

## Multiprotocol BGP for IPv6
RFC 4760定義以下新功能:
* New address family identifier(AFI) model
* New BGPv4 optional and nontransitive attributes:
    *  Multiprotocol reachable NLRI
    *  Multiprotocol unreachable NLRI

MP-BGP 延伸了 AFI和SAFI屬性，並且被放在路由表中:

* IPv4 unicast: AFI:1, SAFI:1
* IPv6 unicast: AFI:2, SAFI:1

### IPv6 簡易設定方式

在設定時建議使用global 位址設定
純IPv6 BGP 可停用 ipv4-unicast
![](https://i.imgur.com/oTkc0lK.png)

```
R1
router bgp 65100
 bgp router-id 192.168.1.1
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 2001:DB8:0:12::2 remote-as 65200
 !
address-family ipv6
 neighbor 2001:DB8:0:12::2 activate
```

### IPv6 Summary
`aggregate-address {prefix/prefix-length} [summary-only] [as-set]`
```
router bgp 65200
 bgp router-id 192.168.2.2
 bgp log-neighbor-changes
 neighbor 2001:DB8:0:12::1 remote-as 65100
 neighbor 2001:DB8:0:23::3 remote-as 65300
 !
 address-family ipv4
 no neighbor 2001:DB8:0:12::1 activate
 no neighbor 2001:DB8:0:23::3 activate
 exit-address-family
 !
 address-family ipv6
 bgp scan-time 6
 network 2001:DB8::2/128
 network 2001:DB8:0:12::/64
 aggregate-address 2001:DB8::/59 summary-only
 neighbor 2001:DB8:0:12::1 activate
 neighbor 2001:DB8:0:23::3 activate
 exit-address-family
```

### IPv6 over IPv4

可僅使用一BGP session 交換IPv4或IPv6 session

```
R1
router bgp 65100
 bgp router-id 192.168.1.1
 no bgp default ipv4-unicast
 neighbor 10.12.1.2 remote-as 65200
 !
address-family ipv6 unicast
 redistribute connected
 neighbor 10.12.1.2 activate
```

```
R2
router bgp 65200
 bgp router-id 192.168.2.2
 no bgp default ipv4-unicast
 neighbor 10.12.1.1 remote-as 65100
 neighbor 10.23.1.3 remote-as 65300
address-family ipv6 unicast
 bgp scan-time 6
 network 2001:DB8::2/128
 network 2001:DB8:0:12::/64
 aggregate-address 2001:DB8::/62 summary-only
 neighbor 10.12.1.1 activate
 neighbor 10.23.1.3 activate
```

![](https://i.imgur.com/b2pZr1h.png)

注意底下 sho bgp ipv6 unicast，Next Hop 為::FFF:xx.xx.xx.xx
僅代表ipv4與ipv6對應，但在ipv6 上沒辦法真的辨識，因此這些路由還未能被載入RIB

![](https://i.imgur.com/avkfIdA.png)

如要解決該問題，可透過手動設定BGP route map 

# Advanced BGP

## Route Summarization

### Aggregate address

![](https://i.imgur.com/xoGlXfJ.png)
R1 - R2 - R3
```
aggregate-address {network} {subnet-mask} [summary-only] [as-set]
```

summary-only 會只發佈出summary過的路由

###  The Atomic Aggreagate Attribute

當BGP router 總和一筆route，他並不會將在總合前的AS_Path發佈出去

比如Figure 12-1，從R1 summarize route，送到R3時AS_Path紀載65100、65200。若從R2 summarize route，只會有65200

### Route Aggregation with AS_SET

AS_SET 功能可以保留路由在被Aggregation前的AS_PATH

## BGP Route Filtering and Manipulation
複習BGP Route Policy Processing
![](https://i.imgur.com/Ti55VEi.png)

IOS XE 提供以下方法去 filtering route:
* Distribution list
* Prefix list
* AS_Path ACL/filitering
* Route map

### Distribution List Filtering

原 BGP Table
```
Network Next Hop Metric LocPrf Weight Path
 *> 10.3.3.0/24 10.12.1.2 33 0 65200 65300 3003 ?
* 10.12.1.0/24 10.12.1.2 22 0 65200 ?
 *> 0.0.0.0 0 32768 ?
 *> 10.23.1.0/24 10.12.1.2 333 0 65200 ?
 *> 100.64.2.0/25 10.12.1.2 22 0 65200 ?
 *> 100.64.2.192/26 10.12.1.2 22 0 65200 ?
 *> 100.64.3.0/25 10.12.1.2 22 0 65200 65300 300 ?
 *> 192.168.1.1/32 0.0.0.0 0 32768 ?
 *> 192.168.2.2/32 10.12.1.2 22 0 65200 ?
 *> 192.168.3.3/32 10.12.1.2 3333 0 65200 65300 ?
```
Distrbution list:
```
R1
ip access-list extended ACL-ALLOW
 permit ip 192.168.0.0 0.0.255.255 host 255.255.255.255
 permit ip 100.64.0.0 0.0.255.0 host 255.255.255.128
!
router bgp 65100
 address-family ipv4
 neighbor 10.12.1.2 distribute-list ACL-ALLOW in
```

```
*> 10.12.1.0/24 0.0.0.0 0 32768 ?
 *> 100.64.2.0/25 10.12.1.2 22 0 65200 ?
 *> 100.64.3.0/25 10.12.1.2 22 0 65200 65300 300 ?
 *> 192.168.1.1/32 0.0.0.0 0 32768 ?
 *> 192.168.2.2/32 10.12.1.2 22 0 65200 ?
 *> 192.168.3.3/32 10.12.1.2 3333 0 65200 65300 ?
```

#### Prefix List filtering

```
R1# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
R1(config)# ip prefix-list RFC1918 seq 10 permit 10.0.0.0/8 le 32
R1(config)# ip prefix-list RFC1918 seq 20 permit 172.16.0.0/12 le 32
R1(config)# ip prefix-list RFC1918 seq 30 permit 192.168.0.0/16 le 32
R1(config)# router bgp 65100
R1(config-router)# address-family ipv4 unicast
R1(config-router-af)# neighbor 10.12.1.2 prefix-list RFC1918 in
```

```
*> 10.3.3.0/24 10.12.1.2 33 0 65200 65300 3003 ?
* 10.12.1.0/24 10.12.1.2 22 0 65200 ?
 *> 0.0.0.0 0 32768 ?
 *> 10.23.1.0/24 10.12.1.2 333 0 65200 ?
 *> 192.168.1.1/32 0.0.0.0 0 32768 ?
 *> 192.168.2.2/32 10.12.1.2 22 0 65200 ?
 *> 192.168.3.3/32 10.12.1.2 3333 0 65200 65300 ?
```

#### AS_Path Filtering
##### regex

可以使用正規化 regular expressions(regex) 去解析大數字的AS



| Modifier                | Description                                               |
|:----------------------- |:--------------------------------------------------------- |
| _ (underscore)          | Matches a space                                           |
| ^ (caret)               | Indicates the start of the string                         |
| $ (dollar sign)         | Indicates the end of the string                           |
| [] (brackets)           | Matches a single character or nesting within a range      |
| - (hyphen)              | Indicates a range of numbers in brackets                  |
| [^] (caret in brackets) | Excludes the characters listed in brackets                |
| () (parentheses)        | Used for nesting of search patterns                       |
| &#124; ( pipe)          | Provides or functionality to the query                    |
| . (period)              | Matches a single character, including a space             |
| * (asterisk)            | Matches zero or more characters or patterns               |
| + (plus sign)           | Matches one or more instances of the character or pattern |
| ? (question mark)       | Matches one or no instances of the character or pattern   |

Example:

![](https://i.imgur.com/vZa8ZHs.png)

 
```
R2# show bgp ipv4 unicast
! Output omitted for brevity
 Network Next Hop Metric LocPrf Weight Path
*> 172.16.0.0/24 172.32.23.3 0 0 300 80 90 21003 2100 i
*> 172.16.4.0/23 172.32.23.3 0 0 300 878 1190 1100 1010 i
*> 172.16.16.0/22 172.32.23.3 0 0 300 779 21234 45 i
*> 172.16.99.0/24 172.32.23.3 0 0 300 145 40 i
*> 172.16.129.0/24 172.32.23.3 0 0 300 10010 300 1010 40 50 i
*> 192.168.0.0/16 172.16.12.1 0 0 100 80 90 21003 2100 i
*> 192.168.4.0/23 172.16.12.1 0 0 100 878 1190 1100 1010 i
*> 192.168.16.0/22 172.16.12.1 0 0 100 779 21234 45 i
*> 192.168.99.0/24 172.16.12.1 0 0 100 145 40 i
*> 192.168.129.0/24 172.16.12.1 0 0 100 10010 300 1010 40 50 i
```
```
R2# show bgp ipv4 unicast regexp 1[14]
! Output omitted for brevity
 Network Next Hop Metric LocPrf Weight Path
*> 172.16.4.0/23 172.32.23.3 0 0 300 878 1190 1100 1010 i
*> 172.16.99.0/24 172.32.23.3 0 0 300 145 40 i
*> 192.168.4.0/23 172.16.12.1 0 0 100 878 1190 1100 1010 i
*> 192.168.99.0/24 172.16.12.1 0 0 100 145 40 i
```

##### AS_Path ACL

如以下指令輸出:
```
R2# show bgp ipv4 unicast neighbors 10.12.1.1 advertised-routes | begin Network
 Network Next Hop Metric LocPrf Weight Path
 *> 10.3.3.0/24 10.23.1.3 33 0 65300 3003 ?
 *> 10.12.1.0/24 0.0.0.0 0 32768 ?
 *> 10.23.1.0/24 0.0.0.0 0 32768 ?
 *> 100.64.2.0/25 0.0.0.0 0 32768 ?
 *> 100.64.2.192/26 0.0.0.0 0 32768 ?
 *> 100.64.3.0/25 10.23.1.3 3 0 65300 300 ?
 *> 192.168.2.2/32 0.0.0.0 0 32768 ?
 *> 192.168.3.3/32 10.23.1.3 333 0 65300 ?
```

R2 有些BGP route 是從10.23.1.3學到的，但R2功能僅是用於AS間的連線測試，不想學習到Route時:

```
R2
ip as-path access-list 1 permit ^$
!
router bgp 65200
 address-family ipv4 unicast
 neighbor 10.12.1.1 filter-list 1 out
 neighbor 10.23.1.3 filter-list 1 out
```
輸出變成如下:
```
R2# show bgp ipv4 unicast neighbors 10.12.1.1 advertised-routes | begin Network
 Network Next Hop Metric LocPrf Weight Path
 *> 10.12.1.0/24 0.0.0.0 0 32768 ?
 *> 10.23.1.0/24 0.0.0.0 0 32768 ?
 *> 100.64.2.0/25 0.0.0.0 0 32768 ?
 *> 100.64.2.192/26 0.0.0.0 0 32768 ?
 *> 192.168.2.2/32 0.0.0.0 0 32768 ?
Total number of prefixes 5
```

### Route Maps

```
neighbor {ip-address} route-map {route-map-name} {in | out}
```
原 BGP table:
```
 Network Next Hop Metric LocPrf Weight Path
 *> 10.1.1.0/24 0.0.0.0 0 32768 ?
 *> 10.3.3.0/24 10.12.1.2 33 0 65200 65300 3003 ?
* 10.12.1.0/24 10.12.1.2 22 0 65200 ?
 *> 0.0.0.0 0 32768 ?
 *> 10.23.1.0/24 10.12.1.2 333 0 65200 ?
 *> 100.64.2.0/25 10.12.1.2 22 0 65200 ?
 *> 100.64.2.192/26 10.12.1.2 22 0 65200 ?
 *> 100.64.3.0/25 10.12.1.2 22 0 65200 65300 300 ?
 *> 192.168.1.1/32 0.0.0.0 0 32768 ?
 *> 192.168.2.2/32 10.12.1.2 22 0 65200 ?
 *> 192.168.3.3/32 10.12.1.2 3333 0 65200 65300 ?

```

```
R1
ip prefix-list FIRST-RFC1918 permit 192.168.0.0/16 le 32
ip as-path access-list 1 permit _65200$
ip prefix-list SECOND-CGNAT permit 100.64.0.0/10 le 32
!
route-map AS65200IN deny 10
 description Deny any RFC1918 networks via Prefix List Matching
 match ip address prefix-list FIRST-RFC1918
route-map AS65200IN permit 20
 description Change local preference for AS65200 originate route in 100.64.x.x/10
 match ip address prefix-list SECOND-CGNAT
 match as-path 1
 set local-preference 222
route-map AS65200IN permit 30
 description Change the weight for AS65200 originate routes
 match as-path 1
 set weight 65200
route-map AS65200IN permit 40
 description Permit all other routes un-modified
!
router bgp 65100
 address-family ipv4 unicast
 neighbor 10.12.1.1 route-map AS65200IN in
```

後:

```
# show bgp ipv4 unicast | b Network
 Network Next Hop Metric LocPrf Weight Path
 *> 10.1.1.0/24 0.0.0.0 0 32768 ?
 *> 10.3.3.0/24 10.12.1.2 33 0 65200 65300 3003 ?
r> 10.12.1.0/24 10.12.1.2 22 65200 65200 ?
 r 0.0.0.0 0 32768 ?
 *> 10.23.1.0/24 10.12.1.2 333 65200 65200 ?
 *> 100.64.2.0/25 10.12.1.2 22 222 0 65200 ?
 *> 100.64.2.192/26 10.12.1.2 22 222 0 65200 ?
 *> 100.64.3.0/25 10.12.1.2 22 0 65200 65300 300 ?
 *> 192.168.1.1/32 0.0.0.0 0 32768 ?
```

### Clearing BGP Connections
Depending on the change to the BGP route manipulation technique, the BGP session may
need to be refreshed to take effect.

Two methods:
1. Hard reset: 
```
clear ip bgp {ip-address}
```
2. Soft rest: 
```
clear ip bgp {ip-address} soft
```

## BGP Communities

BGP Communities 是一個Attribute，作用就像是在Prefix上加上一個Label，讓其他Router 收到這條Prefix根據Lable值做其他事情

```
neighbor {IP} send-community
```


### Well-Known Communities

* Internet
* No-Advertise
* No_Export
* Local-AS

#### The No_Advertise BGP Community
![](https://i.imgur.com/MdqE3aI.png)
告訴收到這條prefix 的 route 不要將這條Prefix發布出去
```
set community no-advertise
```
```
R2# show bgp 10.1.1.0/24
! Output omitted for brevity
BGP routing table entry for 10.1.1.0/24, version 18
Paths: (1 available, best #1, table default, not advertised to any peer)
 Not advertised to any peer
 Refresh Epoch 1
 100, (received & used)
 10.1.12.1 from 10.1.12.1 (192.168.1.1)
Origin IGP, metric 0, localpref 100, valid, external, best
Community: no-advertise
```

#### The No_Export BGP Community
收到這條Prefix的Router不能發給其他的AS
`set community no-export`
![](https://i.imgur.com/utRp0hx.png)

#### The Local-AS(No_Export_Subconfed) BGP Community
收到這條Prefix的Router只能發布給同個Confederation中的AS
```
set community local-as
```
![](https://i.imgur.com/2DK6luS.png)

### Maxiumu Prefix

用於限制可從BGP peer學到多少Prefix

`neighbor {ip} maximum-prefix {prefix-count}
[warning-percentage] [restart time] [warning-only].`


當 BGP peer 宣布超過的route，會使 neighbor 狀態變成 Idle (PfxCt)

## Configuration Scalability

### IOS Peer Groups
用於使設定更容易閱讀

```
router bgp 100
 no bgp default ipv4-unicast
 neighbor AS100 peer-group
 neighbor AS100 remote-as 100
 neighbor AS100 update-source Loopback0
 neighbor 192.168.2.2 peer-group AS100
 neighbor 192.168.3.3 peer-group AS100
 neighbor 192.168.4.4 peer-group AS100
address-family ipv4
 neighbor AS100 next-hop-self
 neighbor 192.168.2.2 activate
 neighbor 192.168.3.3 activate
 neighbor 192.168.4.4 activate
 exit-address-family
```

### IOS Peer Templates
Peer group限制所有neighbor 有相同的outbound routing policy
IOS peer template 允許可重複使用設定
There are two types of BGP peer templates:
* Peer session: for theBGP session
* Peer policy: for the address family policy

```
router bgp 100
 template peer-policy TEMPLATE-PARENT-POLICY
 route-map FILTERROUTES in
 inherit peer-policy TEMPLATE-CHILD-POLICY 20
 exit-peer-policy
 !
 template peer-policy TEMPLATE-CHILD-POLICY
 maximum-prefix 10
 exit-peer-policy
 
 bgp log-neighbor-changes
 neighbor 10.12.1.2 remote-as 200
 !
 address-family ipv4
 neighbor 10.12.1.2 activate
 neighbor 10.12.1.2 inherit peer-policy TEMPLATE-PARENT-POLICY
 exit-address-family
```

# BGP Path Selection

## BGP Best Path
The following list provides the attributes that the BGP best-path algorithm uses for the
process of selecting the best route. These attributes are processed in the order listed:

1. Weight(highest)
2. Local preference(highest)
3. route originated by the local router
4. the path with the shorter Accmulated Interior Gateway Protocol(AIGP) metric
5. AS_Path(shortest)
6. the best origin code
7. the lowset multi-exit discriminator(MED)
8. eBGP > iBGP
9. the path throught the closest IGP neighbor
10. the oldest route for eBGP path
11. the lowest neighbor BGP RID
12. the lowest neighbor IP address

![](https://i.imgur.com/FwfMxjx.png)

### Weight
* BGP wight is a Cisco-defined attribute.
* 16-bit value (0~65535)
* The value is not advertised to other routers
* Higher weight is preferred.
```
set weight {weight} # in a route map set
neighbor {IP} wight {weight}
```


### Local Preference

* A well-known discretionary path attribute
* 32-bit value(0 through 4,294,967,295)
* The value is not advertise between eBGP peers
* Higher value is preferred
* Default value 100

```
bgp default local-preference {value}
set local-preference # route map
neighbor {IP} local-preference {Value}
```

若有相同route，在 show bgp ipv4 unicast 會只顯示 best path
但不代表router discard該route 來源
只是放在Loc-RIB database 等待使用，等待比較完後確定該route 不是best path則會退回給原本的Router

### Local Originated int the Network or Aggregate Advertisement
以下幾點可辨認route 是由本地端發起:

* 從本地端發起的route
* Network 在本地端被Aggregated
* Route 被 BGP peer接收

### Accumulated Interior Gateway Protocol(AIGP)
* Optional nontransitive path attribute
* AIGP metric 32-bit
* A path with an AIGP metric is preferred
* Lower AIGP metric is preferred
通常IGP協定都是使用lowest-path metric作為最短路徑，但沒辦法提供BGP 可擴展性的功能
AIGP 提供BGP 在多個ASs環境中，各AS擁有自己的IGP domain，去維護和計算一個path metric概念植

![](https://i.imgur.com/cJ07PkC.png)

`set aigp-metric {igp-metric | metric} #Route map`

### Shortest AS_Path

* A shorter AS_Path is preferred
* Prepending ASNs to AS_Path makes the AS_Path longer

`set as-path prepend {as-number} # route map`

### Origin Type

* Well-known mandatory
* In Cisco router using the network statment: i(for IGP) origin, ? (incomplete) redistributed

Preference order:
1. IGP origin (Most)
2. Exterior Gateway Protocol (EGP) origin
3. Incomplete origin (Least)

`set origin {igp | incomplete}`

### Multi-Exit Discriminator

* 32-bit value called metriBGP sets
* influence traffic flows inbound from a different AS
* A lower MED is preferred over a higher MED
* If the MED value is not set, the default is 0
可從eBGP session中接收並發給其他iBGP，但不會再傳給其他AS

`set metric {metric}. #route map` 

#### Missing MED Behavior
`bgp bestpath med missing-as-worst # in BGP Configuration`
#### Always Compare MED

`bgp always-compare-med # in BGP Configuration`

Enable this feature on all BGP routers in the AS, or routing loops can occur.

#### BGP Edterministic
![](https://i.imgur.com/xyFj6U2.png)
若沒啟用 bgp deterministicmed ，還是會保留R4的route(因為他是最先被設定的)

```
 bgp deterministicmed  # is recommended for all BGP deployments in the same AS.
```

### eBGP over iBGP

1. eBGP peers (most desirable)
2. Confederation member AS peers
3. iBGP peers (least desirable

### Lowest IGP metric
The next decision step is to use the lowest IGP cost to the BGP next-hop address.
![](https://i.imgur.com/we8xtLa.png)

### Prefer the Oldest EBGP Path

偏好建立最久的BGP session

### Router ID
select the best path using the lowestrouter ID of the advertising EBGP router
如果Route 是從RR(route reflector)接收，Originator ID 會替換掉該 Router ID

### Minimun Cluster List Length

* non-transitive BGP attribute
* Route reflectors use the cluster-id attribute as a loop-prevention mechanism
* Cluster ID 不會在AS間發布，只會套用在locally

![](https://i.imgur.com/F6aqFfJ.png)
如上圖，RR2會選擇直接從R3發布過來的

## BGP  Equal-Cost Multipath

啟用 BGP multipathing 不會影響 best-path algorithm 或改變 path advertisement。
若要啟用 ECMP 以下值必須相同:
* Weight
* Local preference
* AS_Path length
* AS_Path conten(although confederations can contain a different AS_CONFED_SEQ path)
* Origin
* MED
* Advertisement method (iBGP or eBGP) (If the prefix is learned from an iBGP advertisement, the IGP cost must match for iBGP and eBGP to be considered equal.)

`maximum-paths {number-paths}`