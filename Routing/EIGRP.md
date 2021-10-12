
# EIGRP

Enhance distance vector routing.

## EIGRP Terminology


| Term                  | Definition                       |
|:--------------------- | -------------------------------- |
| Successor route       | 擁有抵達目的最低成本路徑的路由   |
| Successor             | 指第一個 next-hop router         |
| Feasible distance     | FD是抵達目的總計成本             |
| Reported distance     | RD值從發布出來的router的FD值累加 |
| Feasibility condition | Backup route                     |

## Configure modes
### Classic mode
```
router eigrp [as-number] 
    network [ip-address] [netmask]
```

### Name mode
```
Router eigrp [process-name]
    address-family {ipv4 | ipv6} {unicast | vrf-name} autonomous-system [as-number]
    network [ip-address] [netmask]
```

interface 設定也在 name Eigrp name mdoe中設定

```
af-interface ethernet 0/0
```


## Topology Table
* Network prefix
* EIGRP neighbors that have advertised that prefix
* Metrics
* Values used for calculating the metric(Load, reliability, delay, bandwidth)

![](https://i.imgur.com/qOPEcMX.jpg)

P(Passive): 為topology 穩定狀態
A(Active): Topology改變時正在計算新路徑

## EIGRP Neighbors

* IP Number: 88
* group address: 224.0.0.10
* multicast MAC address: 01-00-5E-00-00-0A

### Packet Type:



| Type | Name | Function |
| -------- | -------- | -------- |
| 1     | Hello     | 用於discovery neighbors    |
| 2     | Request     | 從鄰居中得到特定資訊  |
| 1     | Update   |  傳送Routing和可靠性資訊給其他鄰居   |
| 1     | Query     | 在網路收斂期間，尋找替代路徑    |
| 1     | Reply     | response to a query packet    |

## Path metric calculation
主要以Bandwidth和Total Delay為主
![](https://i.imgur.com/dXrrQiI.jpg)

### Load Balancing
equal-cost multipathing (ECMP).
EIGRP 支援 unequal-cost load balancing, 並由EIGRP Variance 操縱

公式:

$\frac{Feasible Successor FD}{FDSuccesor Route FD}$ >= Variance

## Failure detection and timers

* Hello timer: 5 sec (slow interface:60 sec)
* Hold time: 15 sec(3 times hello interval) 

### Convergence
![](https://i.imgur.com/ydOHVOB.jpg)

1. R2 發現link failure, 但R2沒有feasible succesor，因此將 10.1.1.0/24 狀態改為active並對鄰居發送Query
2. R3收到後，因為沒有其他的鄰居並回覆給R2沒有該路由。R4 收到也將該路由標記為Active並發給R5 Query
3. R5 收到後，因R5的Successor還存在，因此reply 10.1.1.0/24 路由給R2
4. R4收到R5的回覆，承認該封包，將狀態改為Passive並計算好新的Metrics給R2
5. R2收到R4 relpy，也承認該封包，將該筆路由install 並設定為Passive

#### SIA Timers

Stuck in Active(SIA)，發生在路由器之間使用較慢的interface時，收斂時間過長。

```
Router(config-router)#timers active-time 2(mins)
```

### Route summarization

Route summarization 可縮減路由表和query domain
EIGRP 預設關閉 route sumarization

```
Router(config-router)#auto summary
```

手動summize:
```
Router(config-if)#ip summary-address eigrp {AS number} {Netowrk} {subnet-mask}
```

#### Leak-map

假設有以下network:
192.168.0.0/24
192.168.1.0/24
192.168.2.0/24
192.168.3.0/24
經過summary後變為192.168.0.0/22

Leak-map 可以針對Summarization進行微條整，將特定路由發出
```
access-list 1 permit 192.168.0.0 0.0.0.255
route-map EIGRP-LEAK
match ip address 1
ip summary-address eigrp 1  192.168.0.0 255.255.252.0 leak-map EIGRP-LEAK
```

## EIGRP Stub Router

多用在Branch Office 或 Edge，路由器接收到Query時皆reply"沒辦法抵達"

```
eigrp stub
```

### Stub site function

* Eigrp neighbors 在WAN link上時不傳送queris
* EIGRP STUB 允許downstream接受和發布network prefix穿過WAN
* EIGRP stub site防止 stub site變成transit site

```
router eigrp EIGRP-NAMED
 address-family ipv4 unicast autonomous-system 100
  af interface serial1/0
    sub-site wan-interface
```

## Split Horizon

Disabled:
![](https://i.imgur.com/P6ST30f.jpg)

Enable:
![](https://i.imgur.com/NAoMnv8.jpg)

Split Horizon 可預防 routing loop
EIGRP 預設啟用 Split Horizon

在使用DMVPN時需要將split horizon關閉

## Route Manipulation


## Offset Lists

在RIP 是使用hop-count計算
EIGRP Offset = 256*Offset Delay

## Trouble shooting command
*  lists ipv4 address, local interface, hello and hold time 
```
show ip eigrp neighbors 
```

* show AS mismatch packet

```
debug eigrp packets #
```

* Mismatched K values

```
show ip protocols
```

## EIGRPv6

* Protocol ID 88
* Multicast link-local scoped address FF02::A


| EIGRP Packet | Source | Destination | Purpose |
| -------- | -------- | -------- |-------- |
| Hello     | Link-local address     | FF02::A    | Neighbor discovery and keepalive     |
| Acknowledgement     | Link-local address     | Link-local address    | Acknoleges receipt of an update     |
| Query     | Link-local address     | FF02:A    | Request for route infromation during a topology change event     |
| Reply     | Link-local address     | Link-local address    | A response to a query message     |
| Update     | Link-local address     | Link-local address    | Adjacency forming     |
| Update     | Link-local address     | FF02::A    | Topology change |

* Support Classic AS and Named mode

* Command:


| Command | Description | 
| -------- | -------- | 
| show ipv6 eigrp interface [interface-id] [detail]     |  Disaplys the EIGRPv6 interface    | 
| show ipv6 eigrp neighbors    |  Disaplys the EIGRPv6 neighbors    | 
| show ipv6 route eigrp      |  Disaplys only EIGRP IPv6 routes    | 
| show ipv6 protocols   |  Disaplys the current state of hte active routing protocol processes    | 


