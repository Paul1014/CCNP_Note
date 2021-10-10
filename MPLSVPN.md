# VRF, MPLS and MPLS Layer 3 VPNs

## VRF

### Creating and Verifing VRF Instances
Create VRF:
```
R1(config)# ip vrf RED
R1(config-vrf)# exit
R1(config)# ip vrf GREEN
R1(config-vrf)# exit
R1(config)# ip vrf BLUE
```
Verifying VRF:
```
R1# show ip vrf
 Name     Default RD     Interfaces
 BLUE      <not set>
 GREEN     <not set>
 RED       <not set>
```
Assing the interface to the VRF Instances
```
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# ip vrf forwarding RED
R1(config-if)# interface gigabitEthernet 1/0
R1(config-if)# ip vrf forwarding GREEN
```
### Configuring EIGRP for Multiple VRF Instances

```
R1# configure terminal
R1(config)# router eigrp VRFEXAMPLE # Name mode
R1(config-router)# address-family ipv4 vrf RED autonomous-system 10
R1(config-router-af)# network 10.0.1.1 0.0.0.0
R1(config-router-af)# network 10.0.12.1 0.0.0.0
```

## MPLS

MPLS 是一種以標籤(labels)取代Layer 3 的封包轉送方式。
MPLS is not much faster than traditional IP routing.
MPLS 可以減少在 core router上的轉送負擔，使其更有效率

LIB: Label Information Base
LFIB: Label Forwarding Information Base
LDP: Label Distribution Protocol
LSP: label-switched path
LSRs: label switching routers

### MPLS LIB and LFIB
![](https://i.imgur.com/NAUF1rk.png)

MPLS-Enable Router 使用 label distribution protocol(LDP)交換標籤。當標籤交換後則會建立LIB，在透過LFIB將封包交換出去



### Label Switching Routers

![](https://i.imgur.com/qSDVFKe.png)

R1和R5為Edge LSR(介於MPLS 和 non-MPLS domain)

### Label-Switched Path
LSP 指 labeled packet 單向路徑(因來回所傳輸的路徑可能不同)
![](https://i.imgur.com/P19gz5b.png)

### Labels 構造

使MPLS 在環境中運行，標籤會被加入至封包中，位置介於Layer 2 和 Layer 3之間:
![](https://i.imgur.com/70qw4ir.png)

其構造:
![](https://i.imgur.com/u2W3fZZ.png)

Label: 20 bits, 0-15是保留不使用的
EXP: 3 bits, 用於QoS
S: 1 bit, 封包有多個標籤時，用來表示該標籤會最底層
TTL: 8 bits

### Label Distribution Protocol

為了建立LSP，Label 必須在LSR間被分享，因此LDP 可負責該任務。
LDP 會發送一 hello packet，dest. multicast address 224.0.0.2
, UDP port:646，所有在該link上的Router 皆會接受。
建立好neighbor後，使用TCP 646 port交換 label information.

![](https://i.imgur.com/kxQ5V9Y.png)

透過交換Label information 建立LIB

### Label Switching

根據建立好的 IP routing tabel 和 LIB，Router 會建立好LFIB 和 FIB。
![](https://i.imgur.com/i4eKIsj.png)

### Penultimate Hop Popping

PHP，多用於在MPLS Domain 倒數第二個的Router，如下圖:
![](https://i.imgur.com/s0wf2fJ.png)

一般來說在封包進入到R5時，會先查看LIFB確認標籤，執行pop後再查看FIB表將封包傳出。但依照目前網路環境，R5確定是直接連接10.0.0.0/24
因此可以在R4的時候就先將該封包的標籤pop掉，經過R5時直接查看FIB轉送即可。

## MPLS Layer 3 VPNS

![](https://i.imgur.com/ZM8PNpW.png)

MPLS Domain: P-Network
Customer sites: C-Network
Customer router known as CE(Customer edge), do not run MPLS
CE connect to the PE(Provider edge) router of the MPLS domain.

在此環境中，Customer A 和 Customer B可能會重複使用到private 網段，為應付此需求，PE router上必須使用VRF instances:
![](https://i.imgur.com/bCoLYdF.png)

PE和CE建立IGP 交換route information後，PE會將該route redistribute 至 MP-BGP，以便與其他PE進行交換:
![](https://i.imgur.com/3ezOuFv.png)

特別注意的是，只有PE會設定BGP，P routers 不參與其中。
MP-BGP 建立必須依賴IGP，因此PE與P router之間也需要先建立好routing protocol 確認連線，再PE透過MP-BGP交換CE路由

### MPLS Layer 3 VPNv4 Address

Route Distinguisher(RD) 用於辨識各個Customer Route
RD 在PE per-customer VRF instance 被使用

![](https://i.imgur.com/5nAiYbi.png)
example: 65000:100, 172.16.10.0:200


VPNv4被MP-IBGP 鄰居交換

![](https://i.imgur.com/1fqnzbS.png)

1. CE和PE交換動態路由資訊
2. PE 將 customer-specific route放置各別VRF
3. 將VRF table redistributed into MP-BGP
4. PE 開始與他的MP-IBGP peering交換 VPNv4 route 
5. PE 再將VPNv4 route redistribute 回到VRF table
6. 最後PE再與CE交換路由資訊

### MPLS Layer 3 VPN Label Stack
VPN Label 主要用在 MP-IBGP Peer，中間MPLS路由還是會以LDP Label:
![](https://i.imgur.com/X3T2qm5.png)

![](https://i.imgur.com/vKJRgHY.png)
