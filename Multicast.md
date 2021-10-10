# Multicast

Internet Group Management Protocol(IGMP): Layer 2
Protocol Independent Multicast(PIM): Layer 3


## Multicast Addressing

IP Class D range : 224.0.0.0/4
![](https://i.imgur.com/1jQQfzp.png)

well-know local multicast address:
![](https://i.imgur.com/oJYU3IN.png)

### L2 multicast address

Mutlicast mac address 轉換:
![](https://i.imgur.com/5S5sWd9.png)
8th bit:1(multicast)
25th bit always 0

## IGMP

IGMP 用於recivers 加入multicast group 的協定

### IGMPv2

Message Format:
![](https://i.imgur.com/oMIMBLX.png)

* Type:
    * Version 2 membership report(0x16): 常見在IGMP加入, receiver 加入群組或是回覆 local router' membership quer message
    * Version 1 membership report(0x12):  用於和v1相容
    * V2 leave group(0x17)
    * General membership query(0x11): 定期送至224.0.0.1並確認有無其他receivers, gorup address is 0.0.0.0
    * Group specific query(0x11): 回覆要離開群組的receiver
* Max response time: 僅只有在type 0x11會使用，他可以設定在送出responding report的最大時間
* Checksum: 驗證校正用
* Group address: 0.0.0.0(發出加入)或是特定的group address
IGMP membership report = IGMP join(非正規講法)

IGMP 中會選出一最小interface IP address的router作為 IGMP querier.
其他router 會等待接收 querier的membership query report。
當querier異常時，其他router 會等待60秒的確定沒接收到querier的封包時則會觸發 querier election.

### IGMPv3

延伸IGMPv2功能，新增multicast source filtering.
IGMPv3 可backward compatible(V1,V2)

### IGMP Snooping

為優化傳送並最小化flooding, switch必須設定multicast 的流量給會使用到的receivers.

a multicast MAC address is never used as a source MAC
address. Switches treat multicast MAC addresses as unknown frames and flood them out all ports

cisco switch使用以下方法防止multicast flooding:
* IGMP snooping
* Static MAC address entries

> Even with IGMP snooping enabled, some multicast groups are still flooded on all
ports (for example, 224.0.0.0/24 reserved addresses).

![](https://i.imgur.com/2SIM9FU.png)

IGMP snooping 預設是啟動的

## PIM

### PIM Distribution Trees

Mutlicast rotuer 建立distribution tree去定義 IP multicast traffic 到recivers的網路路徑
distribution trees有兩種類型:Source trees(shortest path trees)and shared trees

#### Source Trees

也稱SPT, 指multicast distribution tree 中的source root, 使用最短路徑向下傳至receivers.

![](https://i.imgur.com/tGmSQg1.png)

#### Shared Trees

使一router 變成 redezous point(RP), 因此也稱 RP trees
![](https://i.imgur.com/6DXrTB8.png)

### PIM Terminology
![](https://i.imgur.com/nS0hP9W.png)

* Reverse Path Forwarding(RPF) interface: with lowest-cost path to the source(SPT) or the RP. 
* RPF neighbor: RPF interface上的 PMI neighbor.
* Multicast Forwarding Information Base(MFIB): 一種forwarding table. 用來快送轉速 multicast forwarding information.
* Last-hop router (LHR)
* First-hop router (FHR)
* Multicast state: The multicast state is composed of the entries found in the mroute table (S, G, IIF, OIF, and so on).

目前PIM 有以下運作模式:
* PIM Dense Mode (PIM-DM)
* PIM Sparse Mode (PIM-SM)
* PIM Sparese Dense Mode
* PIM Source Specific Multicast (PIM-SSM)
* PIM Bidirectional Mode(Bidir-PIM)


![](https://i.imgur.com/c4bieWX.png)

### PIM Dense Mode

Mutlicast group 密集的分布在整個網路中(大部分的host都需要 multicast packet)

PIM-DM 預設將所有interface flooding traffic. 如果在LHR 沒有任何receiver 則會發出 prun message 給source.
![](https://i.imgur.com/h0Rlu8x.png)

![](https://i.imgur.com/qW0TwT2.png)

prune 會在三分鐘後失效，因為 multicast traffic 又會relooded 整個 distribution trees.
PMI-DM 僅適合用在小型網路

### PIM Sparese MODE

與PIM-DM相反，用於multicast group 較稀少的網路環境。
和PIM-DM，利用routing table找出RPF。

#### PIM Shared and Source Path Trees
和PIM-DM flood and prune 的implicit join 模式不同
PIM-SM會以 explicit join，如下圖:
![](https://i.imgur.com/x1wUEFs.png)

As picture, two trees aree created: an SPT from the FHR to the RP (S,G) and a Shared tree from the RP to the LHR (*,G)

##### Shared Tree Join

如PIM-SM 的圖， 連接到LHR的Reciver A發出 IGMP join(address G).
LHR 知道該G 群組的RP為R2，因此送出(*,G)
當RP收到時，代表已經建立了Shared Tree至 receiver A.

##### Source Registration

當 Group G 來源送出一封包，連接該source的FHR必須負責與RP註冊該來源位址，並請求RP建立一個回到FHR的tree

FHR會封裝 來源的multicast data，稱作register message，並透過單向的PIM tunnel 送至RP。
RP收到該register meeage後，若沒有啟用shared tree(因為RP對此Mutli-group不感興趣)則會直接發回給FHR register stop message(不透過tunnel).
但若在RP上啟用了shared tree, 則會將此multicast packet 向下傳送至shared tree，並送出(S,G) Join 給multi-source 建立SPT。
如果和RP間有多個hop，則所有經過的router都會建立(S,G)的SPT。
(*,G)也一樣

當RP開始直接接收muti-source的資料後，則會向FHR發出register top message，此時開始multicast traffic 開始從source 根據SPT向下傳送至RPT(shared tree)至receiver.

PIM register tunnel會根據 RP上的valid RPF path繼續啟用

##### PIM SPT Switchover

PIM-SM 允許LHR替換shared tree 至 SPT的來源。

![](https://i.imgur.com/wMy6mN8.png)

如上圖: 在FHR收到RP來的first multicast packet後。
LHR會發現自身路由表上有最快到 multi-source的路徑。
因此會送出 (S,G)PIM 到FHR並形成SPT, 若有需要則會將PRF interface設定往FHR的SPT介面。然後發出PIM prune message給RP。

##### Designated Routers

當有多個PMI-SM router 從在同一個LAN中，則會透過PIM hello mesage 選出 DR，並免送出多個Multicast traffic 給LAN 或RP。
FHR DR負責封裝unicast register message 給RP
LHR DR負責送出PIM join 和 prune message給RP

DR hold time預設是3.5倍的hello time(105秒)

#### Reverse Path Forwarding
RPF 是用來預防loop 並確保multicast traffic抵達正確的interface
RPF 功能如下:
* 當router 使用一interface接收multicast packet, 該interface用用來送給 source unicast packet，表示該multicast 抵達RPF interface
* 當multicast封包抵達RPF interface，router會將該封包從存在於outgoing interface list(OIL)中的inteface 傳送出去
* 如果封包不是從RPF接收，則會被丟棄

PIM uses multicast source tree between the source and the LHR and between the source the RP
Shared tree between RP and LHR.
RPF check皆會有所不同:
* 當PIM router 接收(S,G), router開始進行RPF check
* 如果PIM router 沒有explicit source-tree state，則會被視為shared-tree state.並開始針對RP address進行RPF check.

#### PIM Forwarder
![](https://i.imgur.com/SWznXqy.png)

如圖，R2和R3透過各自的RPF Interface接收到相同的(S,G) traffic
並將該封包往下傳送OIF。
R2和R3開始比較各自至soure的AD值和route metric。
勝出的router就會作為該LAN的PIM forwarder(如R3)


![](https://i.imgur.com/C0NZcm4.png)

### Rendezous Points

在PIM-SM 中，會選擇一個或多個router運作RPs。
RP會在shared tree選出1 root point
設定方式可以是static或auto-rp, PIM bootstrat router(BSR)

#### Static RP

手動設定RP相對簡單，但在大型複雜的網路環境中會增加管理上成本。

#### Auto-RP
Auto-RP 是cisco專屬機制，好處如下:
* easy to use multiple RPs within a network to serve different group ranges.
* allows load splitting among different RPs.
* simplifies RP placement according to the locations of group participants.
*  prevents inconsistent manual static RP configurations that might cause connectivity problems
*  Multiple RPs can be used to serve different group ranges or to serve as backups for each other.
*  The Auto-RP mechanism operates using two basic components, candidate RPs (C-RPs) and RP mapping agents (MAs).

##### Candidate RPs
C-RPs, 會透過RP announcement messages發布自願成為RP
預設時間為60秒，並使用 224.0.1.39(cisco-RP-announce)
當有多個C-RPs，則會以highest IP address優先

##### RP Mapping Agents
RP MAs, 加入224.0.1.39接收RP announcements, 儲存Anncouncements中的 group-to-RP mapping cache資訊

RP MA會發布給224.0.1.40(cisco-RP-Discovery) RP maaping
預設為60秒，MA announcements 包含已選出的RP和group-to-RP mappings，使所有加入224.0.1.40 router存儲該RP mapping在他們的自己的cache.
多個multiple RP MA 可以被設定成redundancy，會各自發出自己的 group-to-RP mapping 給 PIM domain 中的router.

![](https://i.imgur.com/vxYvLEF.png)


#### PIM Bootstrap Router

BSR 在RFC 5059中被定義，非專屬協定，提供fault-tolerant(容錯), automated RP discovery 和 distribution
BSR 基本上和Auto-RP功能相同，但BSR 是PIM v2規範的一部份，group-to-RP 包含以下:
* Multicast group range
* RP priority
* RP address
* Hash mask length
* SM/Bidir flag

BSR message 使用 224.0.0.13, TTL 為 1
多個Candidate BSRs(C-BSRs)會被部屬在PIM domain預防單一個點出錯，藉由送出 BSR message 中的priority(highest)選出BSR

##### Candidate RPs

A router that is configured as a candidate RP (C-RP) receives the BSR messages
該BSR message含有當前active BSR位址，C-RP則會發出 candidate RP advertisement(C-RP-Adv) message 給BS



![](https://i.imgur.com/fvSQTWA.png)
