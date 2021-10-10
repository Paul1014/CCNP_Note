# Wireless(350-401)

## Wireless Signals and Modulation
### Basic wireless theory

#### frequency
LAN communication frequency:
* 2.4G: 2.4 to 2.4835 GHz
* 5G: 
5.150 to 5.250
5.250 to 5.350
5.470 to 5.725
5.725 to 5.825

2.4G 和 5G頻帶(band)內還被分出小的頻帶，稱為Channel
![](https://i.imgur.com/RJ32rAU.png)

Cneter frequency 定義在band中的channel位置，其範圍稱之為bandwidth:
![](https://i.imgur.com/ajGSqE5.png)

理想狀態為，signal bandwidth 小於 channel width，讓不同的信號盡可能傳送所有channel，使其無機會overlap:
![](https://i.imgur.com/BxB7hrm.png)

然而，實際狀況上還是會需要overlapping:

![](https://i.imgur.com/BHmGjeq.png)
如上圖，雖然頻道間的bandwidth變小了，但在該頻率範圍能容納更多的channel

#### Phase
相位
![](https://i.imgur.com/cAwdFNg.png)

#### Wavelength
波長，單位為lambda(λ)
2.4 GHz為4.92 inches
5 GHz為2.36 inches
![](https://i.imgur.com/japJaV7.png)

#### RF Power and dB
RF signal 強度是以坡峰(amplitude)判斷:
![](https://i.imgur.com/VgDy5ko.png)
RF signal強度單位為watts(W)

db 是用來方便比較強度等級，公式如下:
$$ dB = 10log_{10} \left( \frac{P2}{P1} \right)   $$
通常P2代表source強度P1則是reference value使其比較

dB Laws:

* Law of Zero: P2和P1相同，比數出來為1， 
$$ dB = 10log_{10}(1) $$
此時dB=0

* Law of 3s: P2與P1比數為2 or 1/2
$$ dB = 10log_{10}(2) $$
因此dB = 3 (若1/2為-3)
* Law of 10s: P2與P1比數為10 or 1/10
dB = 10 or -10

![](https://i.imgur.com/zQqntUF.png)

#### Comparing Power Against a Reference: dBm

![](https://i.imgur.com/jqK8A94.png)

$$ db = 10log_{10}left( \frac{0.000031623mW}{100mW})=-65dB $$

直接拿W運算有點困難，可以先將100mW和0.000031623mW變成db(P1=1m)

![](https://i.imgur.com/h8jaMn1.png)

Tx=20 dBm
netloss = 65 dBm
Rx=-45 dBm

Effective isotropic radiated power(EIRP)，測量單位為dBm
從傳送端強度，經由cable訊號衰減，到天線傳送至Rx
![](https://i.imgur.com/dB2bPvN.png)

![](https://i.imgur.com/IxC6m2J.png)


#### Free Space Path Loss

當RF信號由天線(antenna)傳送，穿過開放空間訊號逐漸減弱
![](https://i.imgur.com/Oew9l7v.png)

計算方法:
$$ FSPL(dB)=20log_{10}(d)+20log_{10}(f)+32.44$$
d: distance(km)
f: frequency(MHz)

![](https://i.imgur.com/jmrUZGG.png)

#### Power Levels at the Receiver

Received signal strength indicator(RSSI)
RSSI 值範圍 0 to 255，最弱到最強，單位為dBm
每個receiver 都有sesitivity level:
![](https://i.imgur.com/rnFIH1g.png)

Signal-to-noise ratio(SNR)，單位dB ，值越高越好
![](https://i.imgur.com/vWXoosN.png)

#### Carrying Data Over an RF Signal

spread spectrum(擴頻):

* Direct sequence spread spectrum(DSSS):
使用在2.4GHz band上
* Orthogonal Frequency Division Multiplexing(OFDM):
2.4和5 GHz都可已使用

#### Maintaining AP-Clinet Compatibility

802.11 標準:
![](https://i.imgur.com/7gddi2t.png)

#### Using Multiple Radios to Scale Performance

一個裝置有single-in, single-out(SISO) system
多個transimitter和receivers:
mulitple-input, multiple-out(MIMO) system
![](https://i.imgur.com/OzutiEt.png)

#### Maximizing the AP–Client Throughput
提高SNR和RSSI 值可加快資料傳輸
802.11 有簡單的方法去依照目前的訊號強度，改變coding scheme
稱為 dynamic rate shifiting(DRS)

![](https://i.imgur.com/YmbE0mm.png)


## Wireless Infrastructure


Cisco AP運作兩種模式: Autonomous or lightweight
依據載入的code image。
Autonomous 可以自己獨立設定，Lightweight則需要wireless LAN controllers(WLC)
### Wireless LAN Topologies
#### Autonomous Topology

![](https://i.imgur.com/VTaMJON.png)
Autonomous AP 提供完整的功能，standalone basic service sets(BSS)
每個AP都必須設定mgmt vlan並設定IP，且client roaming只支援Layer 2 domain.
使用Autonomous AP 雖然會使infra複雜難於管理，但有個好處是可以縮短data path.
![](https://i.imgur.com/rdocpB7.png)

#### Lightweight Topology
Lightweight AP必須透過WLC管理，才有完全的功能。
AP和WLC透過CAPWAP tunnel 加入，control plane 和 data plane的資料都會使用該tunnel。
將網路內的AP都加入至WLC，這種架構稱為 centralized 或 unified wireless LAN topology.
![](https://i.imgur.com/o4faJvO.png)

unified wireless LAN topology 有個問題是可能會使data path過長:
![](https://i.imgur.com/PCZocdq.png)

有個做法是使用 embedded WLC switch，讓WLC在Access layer，縮短data path。
最多可支援200 AP
![](https://i.imgur.com/WRUYMxd.png)

也可以使access 的AP 成為WLC，稱為 Mobility Express:
![](https://i.imgur.com/QxiHkTO.png)

### Pairing Lightweight APs and WLCs

Cisco lightweight AP被設計成 touch free，也就是只要開箱後接上網路線，無需做其他設定，只要能使其和WLC連線正常即可加入

#### AP state

![](https://i.imgur.com/3nT8Bcx.png)



##### Discovering a WLC

AP使用下列方法尋找WLC:
* Prior knowledge of WLCs
* DHCP and DNS information to sugeest some controllers
* Broadcast on the local subnet to solicit controllers

為了尋找WLC, AP 會送出 unicst CAPWAP Discovery Request 給 controller IP(UDP port 5246)，或是發出broadcast。
如果WLC存在則會回復CAPWAP Discovery Response
整個過程如下:

1. AP在本地LAN發出 CAPWAP Discovery Request，等待WLC回傳Response

> 如果AP和WLC在不同subnet，則需要設定relay:
```
router(config)# ip forward-protocol udp 5246
router(config)# interface vlan n
router (config-int)# ip helper-address WLC1-MGMT-ADDR
router(config-int)# ip helper-address WLC2-MGMT-ADDR
```
2. 一個AP最多可以啟動三個WLC，並存在NVRAM，當AP重新開機時這些紀錄還是會在。
3. DHCP server 提供AP IP address並送出DHCP option 43 建議 WLC 地址列表。
4. AP 嘗試解析CISCO-CAPWAP-CONTROLLER.local-domain(local-domain是從DHCP學來的)，如果解析到IP，controller則會嘗試與該IP聯繫
5. 如果到此步驟還是沒成功，AP會自己reset並再跑一次Discovery流程

##### Selecting a WLC
當AP 完成WLC discovery，開始擇一WLC嘗試加入，送出CAPWAP Join Request 並等待Response，此時 AP和WLC會建立DTLS tunnel去加密CAPWAP control message
過程如下:
1. 當AP先前就有加入過一controller，則會被設定成primed with a primary, secondary and tertiary controllers，陸續嘗試加入這些controllers

2.如果AP沒有任何候選controller，他會嘗試找一個。若有一個controller被設定成master controller，則會回覆AP request

3. AP會嘗試加入least-loaded wlc，使其load balancing

##### Maintaining WLC Availability

AP 預設會每30秒發出keepalive message 給WLC，假如沒收到WLC回覆，則會再發出四個keepalive message(每三秒一次)，若無回應則會快速轉至尋找其他WLC。

##### Cisco AP Modes

* Local:
default lightweight mode，為特定頻道提供一或多個功能BSSs，在這段期間不會轉送資料，AP會去掃描其他頻道的雜訊、discover rogue devices和比對 IDS事件
* Monitor:
AP尚未轉送資料，但允許充當傳感器，The AP checks for IDS events, detects rogue access points, and determines
the position of stations through location-based services
* FlexConnect:
在遠端的AP可以在不同SSID和VLAN間交換資料
* Sniffer:
* Rogue detector:
AP將自己致力於detecting rogue devices, 購過蒐集有線和無線的MAC address來尋找rogue devices
* Bridge
兩AP設定成bridge可以連接倆分隔地點，多個AP bridge可以形成indoor or out door mesh network.
* Flex+Bridge:
* SE-Connect

### Leveraging Antennas for Wireless Coverage
利用天線來增加無限的覆蓋程度。

####　Radiation Patterns
![](https://i.imgur.com/pnxVBHJ.png)
XY Plane known as H plane(Horizontal)
XZ Plane known as E Plnae(elevation)

![](https://i.imgur.com/HShsbJW.png)

#### Gain
Gain，是用來測量天線如何有效將RF能量集中在某個方向

![](https://i.imgur.com/YTBllYW.png)


#### Beamwidth
Beamwidth，天波束寬，用來作為天線焦點的度量


![](https://i.imgur.com/Q5mEwzU.png)



#### Polarization
![](https://i.imgur.com/2DqvSqG.png)

#### Omnidirectional Antennas

Omnidirectional 常見的類型為 dipole:
it has a relatively low gain
![](https://i.imgur.com/D4JdDIF.png)
dipole: 甜甜圈形狀
2 to 5 dBi
![](https://i.imgur.com/vLok6lA.png)

Cisco wireless access point(APs)也是使用Omnidirectional Antennas

#### Directional Antennas
Patch antennas: egg-shaped pattern
6 to 8 dBi at 2.4 GHz, 7 to 10 dBi at 5 GHz
![](https://i.imgur.com/cUrYfQL.png)

![](https://i.imgur.com/9UQ53MP.png)

Yagi Antenna:
10 to 14 dBi
![](https://i.imgur.com/lA33670.png)
![](https://i.imgur.com/6JDuCL2.png)

Dish antennas:
20 to 30 dBi, the highest gain of all the wireless LAN antennas
arabolic shape

![](https://i.imgur.com/NAA3xhr.png)
![](https://i.imgur.com/ynPgGJL.png)


## Understanding Wireless Roaming and Location Services

### Overview

#### Roaming Between Autonomous APs

![](https://i.imgur.com/9VvjCNc.png)
Client-1 會持續 scan channels和探測candidate APs
當Client-1連接AP 1，假使Client-1 移動至AP 2的範圍，Client-1會發現AP 1的訊號逐漸減弱，並嘗試連線到AP 2，此時在 infra上，MAC address對應的port會有異動(從AP1到AP2)
![](https://i.imgur.com/IgvtU5c.png)

#### Intracontroller Roaming

剛才Roaming範例為AP是autonomous模式，在Lightweight AP 和WLC無線網路架構，是透過CAPWAP tunnels。
![](https://i.imgur.com/HF3C8ir.png)
當Clinet-1 從AP1移動至AP2時，只有在Controller上有變動
![](https://i.imgur.com/8pR6Jav.png)

在Client設備Romaing時，reassociation 會處理以下設定:

* DHCP
* Client authentication:
Cisco controllers 提供三種技術簡化在Roaming時的驗證時間:
	1. Cisco Centralized Key Management(CCKM)
	選擇一個controller 維護clinet 和 其AP的key對應的Database，並在clinet roaming時提供。Clinet必須支援 Cisco Compatible Extensions(CCX)
	2. Key caching: client設備皆會維護在先前AP連線的金鑰，但數量最多八組
	3. 802.11r: client裝置可以快取一部份的 authentication server的金鑰，將其傳遞給接下來roaming的AP

### Roaming Between Centralized Controllers
在大型無線網路內，可能有多個controller，AP會個別分佈在其中。
會使的client 從一controller roaming 至其他controller。
隨著網路規模成長，AP roaming可以藉由組織controller 擴展至mobility groups.


#### Layer 2 Roaming

Client 設備從一AP移動至不同controller的AP，稱為intercontroller roam.

![](https://i.imgur.com/5gagW4C.png)
![](https://i.imgur.com/wPjqxT8.png)


#### Layer 3 Roaming


![](https://i.imgur.com/2qtbxzF.png)

如上圖，Client-1 在AP -21取得VLAN100網段，在Client-1要移動至AP-2時，應問AP-2是發出VLAN200的網段，與VLAN100不同，勢必會重新取得IP。

Layer 3 intercontroller roam會建立original controller與client之間的tunnel，使其保留原本的IP

![](https://i.imgur.com/DJZb2ES.png)

#### Scaling Mobility with Mobility Groups

將兩centralized controllers設定在同個mobility group，client 則可在這中間快速roaming。
若在不同mobility group，Client roaming效率就會變差(IP、Authenticate都會需要重抓)。

controller間必須互相trust，才有辦法在其中間Roaming
![](https://i.imgur.com/rqVYJkz.png)


## Authenticating Wireless Clients



### Authenticating with Pre-Shared Key

Wi-Fi Protected Access(WPA) versions, WPA1, WPA2 or WPA3
two client authentication:
1. Pre-Shared Key(PSK) aka. Personal mode
2. 802.1x aka. Enterprise mode

WPA1和WP2皆有可能被竊聽並字典攻擊，WPA3使用Simultaneous Authentication of Equals(SAE)，來預防此問題。

### Authenticating with EAP

Extensible Authentication Protocol(EAP)，提供更有彈性的驗證框架。
EAP可以做來其他Authentication 的延伸功能。
![](https://i.imgur.com/ePUwf8G.png)
* Supplicant: Client device that is requesting access
* Authenticator: The network device that provides access to the network
* Authentication Server(AS): The device that takes user or client credentials and
permits or denies network access based on a user database and policies (usually
a RADIUS server)

EAP有很多種類:LEAP, EAP-FAST,PEAP 和 EAP-TLS

![](https://i.imgur.com/waOfm1O.png)

# 考試筆記

FlexConnect (previously known as Hybrid Remote Edge Access Point or H-REAP) is a wireless solution for branch office and remote office deployments