---
tags: CCNP EXAM
---
# QoS

## 影響網路品質下降因素
1. Lack of bandwidth

2. Latency and jitter
    *  Propagation delay: 指物理傳送時間(一般網路線、光纖、衛星等)
    *  Serialization delay: 封包至link上傳送時間，s = packet size(bits)/ line speed(bps)
    *  Processing delay: 指設備處理時間(CPU、封包交換、路由等)
    *  Delay variation (Jitter): 多為變數、根據當下網路環境擁塞狀況(network congestion)影響傳送時間。
3. Packet loss

## QoS models

1.Best effort
2.Integrated Services(IntServ): 
 * end-to-end 頻寬預留
 * Resourc Reservation Protocol(RSVP) : 
![](https://i.imgur.com/AUwH0P0.png)
[設定文件](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/qos_rsvp/configuration/15-mt/qos-rsvp-15-mt-book/config-rsvp.html)

3.Differentiated Services(DiffServ):
*  Classes and marks
    [設定文件](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/qos_classn/configuration/15-mt/qos-classn-15-mt-book/qos-classn-ethr-cos-mtch-mark.pdf)

### Classification and Marking
在進入QoS機制前，IP traffic需先進行分類(classification)，再進行標記(marking)，讓其他網路裝置也能套用QoS機制。

#### Classification
##### 方法:
* Internal: QoS groups
* Layer 1: Physical interface, subinterface or port
* Layer 2: MAC address and 802.1q CoS bits
* Layer 2.5: MPLS Experimental(EXP) bits
* Layer 3: DSCP, IPP and src/dst IP address.
* Layer 4: TCP or UDP ports
* Layer 7: Next Generation Network-Based Application Recognition(NBAR2)

#####  Layer 7 Classification
NBAR2 運作模式:
* Protocaol Discovery: 獲得即時及時(real-time)流量並辨別應用程式(Applications)
* Modular QoS CLI(MQC): 將不同應用程式流量進行分類

#### Marking
##### 方法:
* Internal: QoS groups
* Layer 2: MAC address and 802.1q CoS bits
* Layer 3: DSCP, IPP

##### Layer 2 Marking
![](https://i.imgur.com/y9w1NA5.png)
* Priority Code Point(PCP): 用於標記封包於特定CoS，值從0(lowest)至7(hightest)。但在non-802.1Q link 或是 Layer 3網路不起作用，因此會需要其他對應方式，如:IP Precendence Type of Services(ToS)
* Drop Eligible Indicator(DEI): 預設為0，設為1時，代表網路出現擁塞則丟棄該frame
* VLAN Identifier(VLAN ID)

##### Layer 3 Marking
確保Marking在non 802.1Q或是Layer 3環境下可使用，分別有IPP和DSCP標準
![](https://i.imgur.com/Bf4Xpl0.png)
* IPv4 IP Precedence(IPP):僅使用前3 bits，值從0至7，6和7保留internal network使用，目前較顯少人使用。
* DiffServ Field: 可用在IPv4和IPv6，前6 bits為DSCP(0 to 63)，和2 bits Congestion Notification(ECN)

#####  DSCP Per-Hop Behaviors(PHB)

Four PHBs:
* Class Selector(CS) PHB: 較過時的DiffServ field, 兼容於IPP
![](https://i.imgur.com/uDcD0tn.png)

* Default Forwarding(DF)PHB: Default best-effort
![](https://i.imgur.com/7vmLoFp.png)

* Assured Forwarding(AF) PHB: 
只針對Throughtput有要求，不限制dely和jitter。
![](https://i.imgur.com/GdRSV2v.png)
分為四種AF class，class number不代表優先權，每個class 被視為 independently 並被放置不同的queues。(但若經過不認得DSCP的設備時，AF4則會被當作最高值)
Drop Probability 2 bits(1 to 3)，在網路發生擁塞時，數值越高越優先被丟棄。

![](https://i.imgur.com/vVc2E7j.png)



* Expedited Forwarding(EF) PHB:
low-loss, low-latency, low-jitter,多用在VoIP。
EF DSCP Value為101110(十進制:46)

下表為 DSCP PHBs 所有值:
![](https://i.imgur.com/dDJOyX7.png)


##### Scavenger Class

通常將商業價值性較低的流量放置的class(如:遊戲、影音平台等)
DSCP value使用CS1 

## Policing and Shaping

Policing: 將超過的流量直接丟棄，多用在edge
Shaping: 將超過的流量放置buffer，多用在service provider(SP)

![](https://i.imgur.com/IDCZSZ4.png)

### Token Bucket Algorithms

![](https://i.imgur.com/9ak1VMk.png)

參數定義:
* Committed Information Rate(CIR): 為ISP提供傳輸速度，單位為bps。
* Committed Time Interval(Tc): 間隔時間，單位多為milliseconds(ms)。 Tc = (Bc[bits]/CIR[bps]) * 1000
* Committed Burst Size(Bc): 可調整參數設定，最大值為token bucket的size，單位為bytes。
* Token: 單位為1byte
* Token bucket: 以CIR的速率補足token。

下圖舉例:
![](https://i.imgur.com/kt9QIlW.png)
interface speed = 10 Mbps
CIR = 2Mbps
Bc = 0.5Mb
Tc = Bc/CIR = 250 m
Tcs(一秒中有的Tc數量)= 1000/ Tc = 4

Tc 的建議範圍: 8ms to 125ms
而real-time raffic 建議 8ms to 10ms

### Types of Policers

* Single-rate two-color marker/policer
* Single-rate three-color marker/policer (srTCM)
* Two-rate three-color marker/policer (trTCM)

#### Single-Rate Two-Color Markers/Policers

* Single token bucket algorithm
* 超過CIR則直接丟棄封包

![](https://i.imgur.com/HH20UWL.png)

#### Single-rate three-color marker/policer (srTCM)

* two token buckets(RFC2697):當第一個bucket(Bc)滿之後，接下來的流量會往第二個bucket(Be)存放
* Exceeding 和 Violating 流量依據Bc to Be的亂數token而不同
* two bucket 演算法會造成一些TCP retransmission 且更使頻寬使用更有效率，通常使用AF class(AFx1, AFx2, AFx3)
![](https://i.imgur.com/HHuUWuW.png)
![](https://i.imgur.com/z6TGL2U.png)

#### Two-Rate Three-Color Markers/Policers (trTCM)

* RFC2698 與 RFC2697相似，但同時有兩個token bucket(Bc和Be)，以不同的 token rate 注入。
![](https://i.imgur.com/FJBA19w.png)

* 使用CIR和Peak Information Rate(PIR)，PIR必須大於等於CIR
![](https://i.imgur.com/4MWyZN9.png)

#### Congestion Management

Congestion Management 是由 Queuing 和 Scheduling組成。
Queuing 也就是 buffering。
Congestion 是由 實體介面(interface)的queuing algorithm檢測，也就是指 transmit ring(Tx-ring or TxQ)額滿。
主要發生Congestion 原因:
* 進入介面快於輸出介面
* 輸出介面接收來自多個進入介面的封包

Queuing algorithms(大多不適用於 modern rich-media network):

* FIFO
* Round robin: 亂數選擇queues做傳輸，因此不會有starve產生
* Weighted round robin(WRR): 有prioritization的Round robin
* Custom queuing(CQ): 是Cisco 實作的一種WRR，內包含16對round-robin和FIFO queues。可自行定義每個Queues 的traffic type和 bandwidth。CQ會造成長時間的延遲。
* Priority queing(PQ):將queues分成四種等級，但容易使最低等級的產生starved
* Weighted fair queuing(WFQ): 藉由IPP權重自動分配流量的頻寬，雖然可為real-time traffic提供高優先權，但沒辦法為個別的flow保證頻寬。
以下演算法適用於現代網路環境:
* Class-based weighted fair queuing(CBWFQ):可使用達256 queues，延伸WFQ的功能，當封包完成classification後會被分配到屬於的class，該class也會被assign bandwidth, weight, queue limit 和 maximum packet limit。被指派的頻寬的最低值為congestion發生時的流量，queue limit值為允許封包可進入buffer的最大值。CBWFQ指是用於非real-time traffic
* Low-latency queuing(LLQ): 為CBWFQ結合PQ，更符合real-time traffic，當其中有一class沒被使用，則會被分享出來。
![](https://i.imgur.com/jYKyDPe.png)
CBWFQ保證每一個class的bandwidth，LLQ則是替class建立priority，LLQ也能真設定上下限的流量。

#### Congestion-Avoidance Tools
Tail drop:當buffer滿了，所有想要進入的封包都被丟棄。tail drop必須避免於TCP traffic。
Random early detection(RED):當buffer快滿之前，隨機丟棄即將進來的封包。
Weight RED(WRED):由Cisco 實作，可針對traffic 的優先權(IPP、DSCP)進行丟包，WRED也會使用 IP Explicit Congestion Notification(ECN) bits 去告知目前傳輸上遇到擁塞，可用於告知啟用ECN 的端點設備，使降低transmission rates