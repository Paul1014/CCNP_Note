# OSPF
* IP Protocol: 89
* Two multicast address:
    1. ALLSPFRouters: IP: 224.0.0.5 / MAC: 01:00:5E:00:00:05
    2. SLLDRouters: IP: 224.0.0.6/  MAC: 01:00:5E:00:00:06

## Packet Types:


| Type | Packet Name | Functional Overview |
| -------- | -------- | -------- |
| 1     | Hello     | Discovering and maintaing neighbors     |
| 2     | Database description(DBD) or (DDP)     | Exchang when an OSPF adjacency is first being formed. 只包含LSA表頭的部分  |
| 3     | Link-state request(LSR)     |   在交換 type 2封包後，Router會知道自身Database缺少那些LSA，因此會在發起 request取得最新LSA  |
| 4     | Link-state update (LSU)     | 包含LSA完整的資料 |
| 5     | Link-state ack     |     確認鏈路狀態更新封包訊息 

### Hello Packets

Field:
![](https://i.imgur.com/bY4gUns.jpg)

#### Neighbors

| State    | Description                                                                                |
| -------- |:------------------------------------------------------------------------------------------ |
| Down     | Initial state, meaning that the router has notreceived any hello packet                    |
| Attempt  | 該狀態出現於NBMA( non-broadcast multi-access)網路環境，代表router還在嘗試溝通中            |
| Init     | Router 已收到hello packet但還沒建立雙向的連線                                              |
| 2-Way    | 雙線的連線已經建立，BDR和DR也在該階段產生                                                  |
| ExStart  | first state in forming na adjacency, identify the master or slave for LSDB synchronization |
| Exchange | Exchange link state by using DBD packet                                                    |
| Loading  | ask more recent LSAs in Exchange state.                                                    |
| Full     | Neighbors已建立完畢                                                                        |

![](https://i.imgur.com/Mew6xEx.png)

### DR and BDR　運作
在multi-access network中，可允許多個OSPF router 在同一個network sgement。這時需要主要router 來掌控 LSA update，
稱為DR。
流程如下:
![](https://i.imgur.com/f7W8YkO.jpg)

## OSPF CONFIGURATION
Network statement:
```
router ospf 1
    network 10.0.0.0 0.0.0.255 area 0
    network 192.168.20.0 0.0.0.255 area 0
```
Interface-Specific Configuration:
```
interface GigabitEthernet 0/0
    ip add 10.0.0.1 255.255.255.0
    ip ospf [process-id] area [area-id]
```

### Requirements for Neigbor Adjacency

* 需使用獨特的RID
* MTU需要相同，OSPF protocol 不支援 fragmentation
* area ID 相同
* 啟用DR
* OSPF hello 和 dead timers設定一致

## Default Route Advertisement

OSPF支援將預設路由發布出去
```
rotuer ospf 1
    network 10.0.0.0 0.0.0.255 area0
    default-information originate
```

## Common Ospf Optimizations

### Link Costs

$\frac{Reference Bandwidth}{Interface Bandwidth}$

Reference default: 100M

```
ip ospf cost 1-65535
```

### Failure Detection

Dead timer reaches 0, the neighbor state is changed to down.

### DR Placement

以下為DR選舉的過程:
先比較Prioriy(越大越好)再比較Router ID(越大越好)
設定:
```
interface gigaethernet 0/1
    ip ospf priority [number]
```

### OSPF Network Type



| Type                | Description             | DR/BDR | Timers                                 |               
| ------------------- |:----------------------- | ------ | -------------------------------------- | 
| Broadcst            | 所有設定接在Ethernet 上 | Yes    | hello:10 <br>  waite:40 <br> Dead:40   |                                                        
| Non-Broadcst        | Frame Relay上           | Yes    | hello:30 <br>  waite:120 <br> Dead:120 |                                                       
| Point to Point      | Frame Relay、GRE、PPP   | No     | hello:10 <br>  waite:40 <br> Dead:40   |                                                      
| Point-to-Multipoint | 使用在hub-and-spoke 上(HDLC)  | No     |  Hello:10 <br>  waite:40 <br> Dead:40    |                


## AREAS

* Area 0 called backbone. all areas must connect to Area 0.
* Area border routers(ABRs) between Area 0 and another OSPF area.

### OSPF Route Types
* intra-area routes : O
* interarea routes: O IA

### LINK-STATE ANNOUNCEMENTS


| Name                     | Description                             |
|:------------------------ | --------------------------------------- |
| Type 1 Router LSA        | 只用在同個area內                        |
| Type 2 Network LSA       | 在Multi-access中由DR轉發的              |
| Type 3 Summary LSA       | 由area 0發布network prefixes 至其他area |
| Type 4 ASBR Summary LSA  | 發給特定ASBR的summary LSA               |
| Type 5 AS external LSA   | 宣告redistributed的route                |
| Type 7 NSSA external LSA | 在NSSAs內歡告redistributed routes       |

Type 1,2 ,3 用在intra-area和interarea

#### Type 1 LSA
* addvertise router, age ,sequence, link count, and link ID.
* correlate information for neighbor router



| Description                   | Link Type | Link ID Value           | Link Data            |
| ----------------------------- | --------- |:----------------------- |:-------------------- |
| P2P link(IP address assigned) | 1         | Neighbor RID            | Interface IP address |
| P2P link(using IP unnumbered) | 1         | Neighbor RID            | MIB II IfIndex value |
| Link to transit network       | 2         | Interface address or DR | Interface address    |
| Link to stub network          | 3         | Network address         | Subnet mask          |
| Virtual link                  | 4         | Neighbor RID            | Interface IP address |
* Network link types:
    1. Transit
    2. Point-to-point: not use DR
    3. Stub: NO neighbor adjacencies are established on the link.

#### Type 2 LSA
* The DR 廣播 Type 2 LSA 並辨識所有在同個網路的路由器
* show ip ospf database network 可查看type 2 詳細資訊
* 如果DR有異動話，會產生新的Type 2 LSA，使整個網路重新跑一次SPF
#### Type 3 LSA
* 由ABR 收到各area的type 1 LSA作成Type 3並發布給其他area
* 如果從Area 0(Backbone)發出來的Type 3，ABR則會產生新的 type 3給nonbackbone area

![](https://i.imgur.com/ZtmJ3CV.jpg)
* 如果是由Type 1 LSA製作出的Type 3 LSA，Metric 則是用原本Type 1的
* 如果是由Type 3 LSA製作出的Type 3 LSA，Metric 則用總計的量
* 如果是由Area 0的Type 3 LSA製作出的Type 3 LSA，則用到ABR的成本加上LSA裡面總成本
* 只會從Area 0發布出去

##### Type 3 LSA Fields


| Field              | Description                                   |
|------------------ | --------------------------------------------- |
| Link ID            | Network number                                |
| Advertising Router | RID of the router advertising the route (ABR) |
| Network Mask       | Prefix length for the advertised network      |
| Metric             | Metric for the LSA                            |

#### Type 5 LSA
當一路由紀錄是被reditributed 進入OSPF，ASBR則會發出Type 5 LSA給整個OSPF domain。
![](https://i.imgur.com/Pd7cT57.jpg)
以上圖為例，R6 將 static route redistributing 到OSPF domain. 

路由顯示上會是 O E1 or O E2
##### Type 5 LSA Fields

| Field              | Description                                           |
|:------------------ |:----------------------------------------------------- |
| Link ID            | External network number                               |
| Network Mask       | Subnet mask for the external networ                   |
| Advertising Router | RID of the router advertising the route (ASBR)        |
| Metric   type      | OSPF eternal metric type (Type 1 O E1 or Type 2 O E2) |
| Metric             | Metric upon reditribution                             |
| External Route Tag | 用來溝通AS 邊界或其他相對資訊防止routing loop               |

#### Type 4 LSA

用來提供ASBR的位置，會是由ASBR發布Type 5後第一個接收到的ABR發布下去。
ABR 只會為每一個ASBR發布一次Type 4 LSA，儘管ASBR 有上千條的Type 5。

#### Type 7 LSA

Use to not-so-stubby areas (NSSAs) advertise the external route to backbone area, reduce the LSDB in area.

路由顯示上會是 O N1 or O N2

#### LSA Type Summary
![](https://i.imgur.com/oXz509Y.jpg)



## OSPF Stubby Areas
### Stub Area

Stub area 不允許Type 5 LSA
因此在ABR收到Type 5 LSA時，會透過Type 3 LSA產生 default route(0.0.0.0/0) 給Stub Area
在Stub area內的Router 都必須設定成stub

```
area area-id stub
```



### Totally Stubby Areas
將Type 3 LSA, Type 4 LSA 和 Type 5 LSA的route 直接產生一個 default route 給 stub area

```
area area-id stub no-summary
```
no-summary 設定在stub area 的router上，ABR上只要設定
```
area area-id stub
```
### Not-so-stubby area (NSSAs)
與 stub area一樣禁止Type 4和 Type 5的LSA，但允許 redistributaion of external routes 進入
當 ASBR要將external routes 傳入 NSSA時，會將Type 5 換成 Type 7，當Type 7 LSA 抵達 ABR時，在將轉換成Type 5.

![](https://i.imgur.com/4zwjWLx.jpg)

在NSSA 內的router 都必須設定 nssa option，不然會沒辦法組成鄰居。
如果NSSA 需要 default route 可以用:
```
R3
    area area-id nssa default-information-originate
```

```
R4
    area area-id nssa
    redistribute connected metric-type 1 subnets
```


### Totally NSSAs

如同 totally stubbly area，會將 3,4,5會成default route。
因此在ABR上只要設定如下
```
    area area-id no-summary
```


## Discontiguous Networks

如下圖為例:

![](https://i.imgur.com/TNWyrFC.jpg)
因為Type 3 LSA都是要經過Area 0，因此R4還是會先經過Area 0才底到Area 12的R1，不會穿越過Area 23

若要解決discontiguous 問題可使用Virtual link

### Virtual Link
```
area [area-id] virtual-link [router-id]
```

需要建立Virtual Link 的Area，不得為Stub area



## PATH SELECTION

調整 interface cost:
```
ip ospf cost 1-65535
```

### Intra-area
通常OSPF 都會預設優先走 Intra-area 的route，儘管彼此cost非常小。



### Interarea

![](https://i.imgur.com/EQsZteD.jpg)

### External Route
分成Type 1 和 Type 2
兩者不同地方為:
* Type 1 路由優先於Type 2
* Type 1 的metric 等同於 redistribution metric 加上到ASBR總路徑metric
* Type 2 metric僅等同於 redistribution metric

#### E1 and N1 External Route
Type 1 路由用 redistribution 路徑加上抵達ASBR最短路徑計算抵達被宣告的網路
當 O E1 和 O N1路由同時進入ABR 的RIB時，會載入 O N1

設定指令:
```
router ospf 1
    redistribute {protocol} {asnumber} metric {value} metric-type 1 subnets
```

#### E2 and N2 External Route
在宣告時直接給定一metric

```
redistribute {protocol} {asnumber} metric {value} subnets
```

## SUMMARIZATION OF ROUTES


### Interarea Summarization
![](https://i.imgur.com/XYCxm62.jpg)
Metric 也是選最少的發布
command:
```
area [area-id] range [network] [subnet-mask] [advertise|not-advertise] [cost]
```
### External Summarization

![](https://i.imgur.com/BuS1BWY.jpg)

```
router ospf 1
    summary-adress 172.16.0.0 255.255.255.240.0
```

## Router Filtering

通常發生在ABR上

### Filtering with summarization
用 not-advertise 取消發布summary route

### Area filtering

![](https://i.imgur.com/TWWkUk1.jpg)

設定方式:

![](https://i.imgur.com/sa4tOVt.jpg)

### Local OSPF Filtering
![](https://i.imgur.com/VluASF1.jpg)

可在local router上篩選掉該路由，但還是會被發佈出去

# OSPFv3

* support IPv4 and IPv6
* New LSA type for IPv6
* Remove the IP prefix information in the OSPF packet header
* Router ID must be manually assign.
* Authentication use IPsec extension headers in the IPv6 packet.

## OSPFv3 Link-State Advertisement

* Protocol ID 89
* OSPFv3 LSA Types


| LS Type | Name              | Description                                                                                              |
| ------- | ----------------- |:-------------------------------------------------------------------------------------------------------- |
| 0x2001  | Router            | 每一個路由器產生 router LSA，其中內含interface state 和 cost                                             |
| 0x2002  | Network           | 由DR產生 network LSA並宣告給所有在同個網路上的路由器，包含DR本身                                         |
| 0x2003  | Interarea prefix  | ABR 產生 interarea prefix LSA 告訴前往其他area 的IPv6 address prefixes 的route                           |
| 0x2004  | Interarea router  | ABR產生的LSA 告知其他Area ASBR的位址                                                                     |
| 0x4005  | AS external       | ASBR 宣告default route 或是透過redistribution 的其他協定路由                                             |
| 0x2007  | NSSA              | ASBR 位於 not-so-stubby area 發布 route redistributed into the area                                      |
| 0x0008  | Link              | Sheared only between neighbors on the same link                                                          |
| 0x2009  | Intra-area prefix | Advertise one or more IPv6 prefixes that are associated with a router, stub, or transit network segament |

## OSPv3 Communication

FF02::05 : ALLSPFRouters
FF02::06 : ALLDRouters

![](https://i.imgur.com/tpKH8Tm.jpg)

## OSPFv3 configuration

和以往不同，network是直接宣告在 interface上

1. 啟用IPv6 Routing
```
Router#ipv6 unicast-routing 
```
2. assign the router ID to the OSPF process, router ID 32 bits
```
router ospfv3 1
    router-id 192.168.1.1
```
3. Enable OSPFv3 on an interface
```
interface Loopback 0
    ipv6 address 2001:DB8::1/128
    ospfv3 1 ipv6 area 0
```
### OSPFv3 Verification

```
show ospfv3 ipv6 neighbor
show ospfv3 interface [interfacename]
show ip route ospfv3
```

### Summarization

```
Router(config)# router ospfv3 1
Router(config-router)# address-family ipv6 unicast
Router(config-router-af)# area 0 range 2001:db8:0:0::/65
```

### Network type

OSPFv3 可支援和OSPFv2的Network type，假如介面被設定成broadcast且被指成DR state，也能將它改成 point-to-point
該設定需要兩端router皆改

```
ospfv3 network point-to-point
```

## IPv4 Support In OSPFv3
Ensure that the IPv4 interface has an IPv6 address(Global or Link local)

```
ospfv3 process-id ipv4 area area-id
```