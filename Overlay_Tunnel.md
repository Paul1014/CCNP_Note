# Overlay Tunnels


## Generic Routing Encapsulation(GRE) Tunels

GRE Tunnel 多用於連接不同網段的網路，或是用於VPN環境
其原理是local router在原始封包上加上新的header(也就是Encapsulation)，到達remote router後在進行de-encapsulation。
![](https://i.imgur.com/Ng8jPy4.png)
GRE 支援 IPv4和IPv6 underlay or overlay 網路環境

GRE 設定:
![](https://i.imgur.com/Iy20TXh.png)

GRE MTU:
通常網路的MTU是1500, 因為GRE會在原本的IP header上再次封裝,
 需根據tunnel type去進行調整:
 
![](https://i.imgur.com/MtOIAxG.png)


### Tunneling vs Encapsulation
Encapsulation: 在特定的 protocol stack 的每一層資料增加header(如一般主機會將html data加上TCP(Layer4),IP(Layer3),Ethernet(Layer2)的header 並逐層增加)
Tunneling: 從眾多協定中選擇其一將資料封裝並傳送至外網。Tunneling 可以傳送 Lower-layer 和 Same-layer的協定
Tunneling 的三個組件:
1. Passenger protocol: The protocol that you are encapsulating. For example, IPv4 and IPv6 protocols
2. Carrier protocol: The protocol that encapsulates. For example, generic routing encapsulation (GRE) and Multiprotocol Label Switching (MPLS).
3. Transport protocol: The protocol that carries the encapsulated protocol. The main transport protocol is IP.

## IPsec Fundamentals
### Security Services:
* Peer Authentication:PSK, Digital certificate
* Data confidentiality:DES, 3DES, AES(recoomended)
* Data integrity:HMAC、SHA、MD5(not use)
* Replay detection
![](https://i.imgur.com/3bFSQYh.png)


### Authentication Header
* Provide data integrith, authentication and replay detection.
* Make sure the data packet has not been modified during transport
* Use IP Protocol 51.
* Not support encryption(data confidentiality) and NAT traversal

### Encapsulating Security Payload(ESP)

Packet transport:
* Tunnel mode: 加密整個原始封包並插入新的IPsec haeder。新header 可用來routing 並提供overlay 功能
* Transport: 只加密payload。

![](https://i.imgur.com/Qh7sCly.png)


Diffie-Hellman(DH):非對稱式交換協定，利用AES加密的psk給peers 驗證。
DH group refers to the length of the key, where the larger the modulus, more secuity.
Cisco recoommends avoiding DH groups 1,2, and 5 and instead use 14 and higher.
RSA signatures
Pre-Shared Key

### Transform Sets
* Authenication header transform
* ESP encryption transform
* ESP authenication transform
* IP compreession transform

![](https://i.imgur.com/Ubqua16.png)

### Internet Key Exchange
用於驗證兩端的Security associations(SAs)，也成為IKE tunnel
IKEv1(RFC 2409)
IKEv2(RFC 7296)

#### IKEv1
Internet Security Association Key Management Protocol(ISAKMP) use UDP prot 500

two phase of key negotiation for IKE and IPsec SA:
* Phase 1: Establishes a bidirectional SA, Known as an ISAKMP SA.(IKE SA)
initiator:將初始化SA協議的端點
responder
Phase 1 協議可使用兩種模式: main mode(MM), aggressive mode(AM)
* Phase 2: 利用Phase 1 SA to establishe unidirectional IPsec SAs
The method used to establish the IPsec AS is known as quick mode.
##### MM
Operate more safe, but take longer than AM.

The mode consists of 6 msg exchnages:
1. MM1: 1st msg that initiator sends SA proposal to responder. SA proposal 包含以下資訊:
Hash algorithm
Encryption algorithm
Authenication method
DH group
Lifetime(default 24h)
2. MM2: Send the SA proposal is matched from responder.
3. MM3: initiator starts the DH key exchange.
4. MM4: responder sends its own key to the initiator.
5. MM5: the initiator starts authentication
6. MM6: responder send back packet and authenticates the session. The ISAKMP SA is established.

##### AM

1. AM1: the initiator send all the informaiton(MM1,MM3,MM5)
2. AM2: the responder send MM2,MM4,MM6
3. AM3: the initiator send the authentication(MM5)

##### QM
Use a three-message exchange:
1. QM1: 任一方開始multiple IPsec SAs 單向交換訊息，內容包含在Phase 1時一至同意的 encryption and integrith 演算法
2. QM2: 從reponder發出符合QM1中的IPsec parameters
3. QM3: 這時已經有兩條unidirectional IPsec SAs在兩端點之中


#### IKEv2

IKEv2 間化 IKEv1，變得更有效率。
最大差別是SA的建立方法，IKEv2 中將通訊request和respones稱為exhcanges 或是 request/response pairs.

1. The first exchange, IKE_SA_INIT, 等同於IKEv1的MM1 to MM4，但v2僅只使用一個 request/response pair.
2. The second exchange, IKE_AUTH, 等同於MM5 to MM6和QMM1, QM2,僅只使用一個 request/response pair.

IKEv2總共只使用4個msg完成bidirectional IKE SA和unidirectional IPsec SAs.
如果需要額外的IPsec SA,IKEv2也只使用兩個msg(a request/response pair), CREATE_CHILD_SA exchange


![](https://i.imgur.com/RG5josO.png)





## Cisco Location/ID Separation Protocol(LISP)
LSIP Topology:
![](https://i.imgur.com/z19vYpw.png)




LISP 由以下components組成:
* Endpoint identifier(EID)
在LISP site內的端點IP位址
* LISP Site
LISP router and EIDs存在的地方
* Ingress tunnel router (ITR)
指從EIDs接收到的LISP封裝的IP封包並轉至對外的LISP router
* Egress tunnel router (ETR)
與ITR相反，從外部接收到的LISP封裝的IP封包轉至給EIDs的LISP router
* Tunnel router (xTR)
指同時運作ITR和ETR的router
* Proxy ITR(PITR)
從非LISP site傳送資料到EIDs
* Proxy ETR(PETR)
從EIDs傳送資料到非LISP site
* Proxy xTR(PxTR)
通時運作PITR和PETR
* LSIP router
* Routing locator(RLOC)
An RLOC is an IPv4 or IPv6 address of an ETR that is
Internet facing or network core facing.
* Map server(MS)
從ETR上學習EID-to-Prefix mapping資訊，並儲存 EID-to-RLOC mapping database
* Map resolver(MR)
收到從ITR 發出的LISP 封裝的map request，並回答對應的ETR，紀錄需要先查詢MS
* Map server/map resolver(MS/MR)

### LISP Architecture and Protocols

* LISP routing architecture
在傳統路由架構，endpoint只要一變動位置，其IP位指也會變動。
但LISP將IP位指拆分成EID和RLOC，使endpoint可以在site和site之間roam
* LISP Control Plane
![](https://i.imgur.com/7QYMzge.png)
與DNS相似，LISP Router會詢問EID的位置(MAP request)，再由MS回覆其所在的RLOC(Map Reply)
* LISP Data Plane
![](https://i.imgur.com/rSz9Ocx.png)
LISP packet: RFC 6830
* Outer LISP IP header: 由ITR封裝EID IP位址
* Outer LISP UDP header:port使用4341，用於改善load sharing
* Instance ID: 24-bit value用於網路虛擬化，換句話說他為VRF和VPN啟用虛擬化及分割，像是MPLS 網路中的VPN ID
* Original IP header: 給EID接收

### LISP Operation

* Map registration and map notify
![](https://i.imgur.com/MYyrAYW.png)
	1. ETR傳送map register message給MS，註冊他的EID(10.1.2.0/24)，也帶上RLOC位址(100.64.2.2)。ETR預設會回覆map request message，但也可以在當時register 中啟用proxy map，讓MS幫忙回覆
	2. MS 送出map notify message給ETR，確認map register 已經接收到並執行，UDP port 4342(both)
* Map request and map reply
![](https://i.imgur.com/foEcxh9.png)

當LISP site的Endpoint想連線到其他LISP site，會經歷以下步驟
	1. host1 透過DNS解析搜尋到hosts2的IP 位址(10.1.2.2)，並將封包送至Gateway並轉送到ITR上(這中間都是為一般routing)
	2. ITR收到該封包後，查詢FIB資料，並根據以下規則判斷:
		(1) 該是否符合預設路由且沒有其他往host2的路由
			是，則進行步驟(2)
			否，則用符合的route 轉送
		(2) 來源IP為址是否在本地 map cache有註冊EID?
			是，接著下步驟
			否，則直接轉送
	3. ITR 送出封裝的map request(UDP:4342) 給MR，source port則是ITR隨機。
	4. 因為MR和MS在同個裝置上運作，因此MS 直接將該map request ETR；假使MR和MS在不同裝置上，MR會將ITR發來的map request 發給MS，MS會在轉發該request給ETR
	5. ETR回傳ITR map reply message(包含EID-to-RLOC)，source port為UDP4342，destiation為ITR當時發出request的port
	6.ITR將EID-to-RLOC mapping載入本地cache，並準備開始轉送LISP traffic
* LISP data path
![](https://i.imgur.com/v83Il1L.png)
當ITR已經接收到EDI-to-RLOC，準備開始從host1傳送資料至host2，以下步驟為封裝及解封裝過程
	1. ITR收到來自EDI host 1到hosts 2的封包
	2. ITR會將該封包封裝outer header，並加上RLOC IP address。封包使用UDP 4341進行轉送
	3. ETR收到後解封裝傳給host 2
* Proxy ETR
![](https://i.imgur.com/YPqO0hf.png)

當EID傳送非LISP site內的目的位址時，會需要有PETR協助轉送，如以下步驟:
	1. 由ITR接收到host1傳送的封包(目的位址非LISP site)
	2. ITR向MR發出map request
	3. Mapping database 上查無該對應為址，並回傳ITR negative map reply，並讓ITR將其non-LISP prefix 及PETR加至mapping cache和FIB
	4. ITR重新送出LISP 封裝封包給PETR
	5. PETR解封裝後將其封包送至non-lisp 目的位址
* Proxy ITR
![](https://i.imgur.com/VFEHeTn.png)
	1. PITR從外部收到至host1 的封包
	2. PITR 送出 map request 給MR詢問host1
	3. MS/MR將其map request 轉給ETR
	4. ETR送出map reply(EID-to-RLOC) 給PITR
	5. PITR將LISP封包轉送給ETR
	6. ETR收到後解封裝轉送給host1
	
## Virtual Extensible Local Area Network(VXLAN)

主機虛擬化對傳統的網路環境有很大的需求，一台實體機上有許多的VM且都有各自的MAC address，使得在傳統Layer 2 網路上有以下問題:
* 12-bit VLAN ID不夠使用
* 龐大的mac address tables
* STP blocks links to avoid loops, and this results in a large number of disabled links, which is unacceptable.
* ECMP 不支援
* 機器移轉(Host mobility)難以進行

VXLAN 是 overlay data plane 封裝格式，延伸Layer 2 和 Layer 3 overlay network，並使用 MAC-in-IP/UDP tunneling
![](https://i.imgur.com/JWOK8Tk.png)

VXLAN destaination port : 4789
Linux default: 8472

VXLAN 擁有24-bit network identifier(VNI)
為了在Layer 3 網路上發現VNIs，必須使用 virtual tunnel endpoints(VTEPs)
VTEP有兩個介面:
![](https://i.imgur.com/LbD53Nt.png)

* Local LAN interface: 提供兩個local host橋接
* IP interface: 面對 core network，該介面的IP為址方便辨識VTEP並將VXLAN traffic封裝解封裝

VXLAN為data plane protocol，control plane可以選擇以下搭配:
* multicast underlay
* unicast VXLAN tunnnels
* MP-BGP EVPN 
* LISP control plane

LISP 為VXLAN preferred choice
Cisco SD-Access則是使用VXLAN(data)和LISP(control)的實例。
![](https://i.imgur.com/MHzdnQH.png)

如上圖，LISP僅址有封裝Layer 3位址。
VXLAN將original ethernet header(MAC-in-IP)封裝進去