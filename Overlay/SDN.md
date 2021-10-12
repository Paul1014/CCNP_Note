# SDN (350-401)

## Virtualization

### Server Virtualization

#### Virtual Machines
Hypervisor:
* Type 1: 直接運行在system hardware. bare metal, native
* Type 2: 運行在 OS上
![](https://i.imgur.com/NpdXOIL.png)


### Network Functions Virtualization

NFV和server virtualization, cloud environments有以下優點:
* 減少資金及維運成本支出
* Faster time to market(TTM)，VM和 container更容易部屬
* Ability to scale up/out and down/in capacity on demand (elasticity)
* Openness to the virtual appliance market and pure software networking vendors
* Opportunities to test and deploy new innovative services virtually and with lower risk

ETSI 定義的 NFV 架構框架:
![](https://i.imgur.com/Z3U1c1J.png)

* NFVI: NFV Infrastructure，在VNFs中所有部屬的hardware 和 software
* Virtualized Infrastructure Manager: VIM，負責管理所有硬體及虛擬化資源。
* Element Managers: EMs, 負責VNF 功能管理。提供VNF的fault, configuration, accounting, performance, and security(FCAPS)功能
* Management and Orchestration: orchestration負責VNF network services creating, maintaining and tearing down. 
* Operations Support System(OSS)/Business Support System(BSS):  OSS通常運行在SP或是大型企業網路，幫助maintaining network inventory, provisioning new services, configuring network devices, and resolving network issues. OSS 通常會和BSS結合來改善Customer experience.

#### VNF Performance

Terminology:
* Input/output: 電腦系統與外界溝通橋梁，資料接收以及送出的地方
* I/O device: 外部裝置如鍵盤滑鼠
* Interrupt request(IRQ): 藉由I/o device發出的硬體訊號給CPU。CPU收到IRQ會保存目前的狀態，並暫時丟棄現在做的事去執行和設備有關的interrupt handler routine
* Device driver: 控制I/O 裝置以及允許CPU與I/O裝置溝通的電腦程式
* Direct memory access(DMA): 一種記憶體存取方法，使I/O裝置可以直接在主記憶體接受或傳送資料，不必透過CPU，提升速度
* Kernel and user space: 
![](https://i.imgur.com/OSRb285.png)

在虛擬環境中，hypervisors 負責將 pNIC和vNIC 透過 vSwitch 串聯一起:
![](https://i.imgur.com/KxtsmWH.png)
從pNIC傳送資料至VM的步驟如下:
1. pNIC接收資料後將其放入Rx queue(ring buffers)
2. pNIC 透過DMA方式送出packet和packet descriptor 到主記憶體，packet descriptor 內容只有 memory location and size of the packet.
3. pNIC 送出IRQ給CPU
4. CPU 將控制權轉給送出IRQ的pNIC，接受該封包並移到network stack
5. 封包資料從socket receive buffer 複製到OVS virtual switch
6. OVS 封包轉送至VM動作，需要切換kernel and user space
7. 封包抵達vNIC 的Rx queue
8. vNIC 透過DMA方式將packet and packet descriptor到 virtual memory
9. vNIC 送出IRQ給vCPU
10. vCPU將其控制權給該vNIC並將封包移動到network stack
11. 該封包資料複製並傳送到 VM的應用程式上
所有傳送封包的步驟接如上，可發現會需要CPU處理多個 interrupt
使用high-speed NIC也會增加interrupt數量。
為避免過載，有以下 I/O技術被發展出來:
* OVS Data Plane Development Kit(OVS-DPDK)
* PCI passthrough
* Single-root I/O virtualization(SR-IOV)

##### OVS-DPDK

![](https://i.imgur.com/9R6VNO9.png)

OVS-DPDK 運作在user space，略過kernel space，當資料到達OVS可以直接被交換到特定的VNF

##### PCI Passthrough
允許VNF直接連接實體PCI裝置，但此方法會因pNIC的數量限制
![](https://i.imgur.com/MAHfqVb.png)

PCI passthrough 有以下好處:
* one-to-one的對應
* 略過hypervisor
* 直接存取I/O資源
* 降低CPU使用率
* 減少系統延遲
* 增加I/O throughput

##### SR-IOV
PCI passthrough 加強版，與許多個VNF共享一個pNIC
SR-IOV支援兩種模式:
* Virtual Ethernet Bridge (VEB): Traffic between VNFs attached to the same pNIC is
hardware switched directly by the pNIC
* Virtual Ethernet Port Aggregator (VEPA):Traffic between VNFs attached to the
same pNIC is switched by an external switch

![](https://i.imgur.com/YqdiDOa.png)


### Cisco Enterprise NEtwork Functions Virtualization (ENFV)

Cisco ENFV solution 是建立在 ETSI NFV 架構上，提供以下幾點好處:
* 減少硬體使用量
* 減少硬體到廠服務的需求
* 提供上架新服務、重大更新的簡單操作
* 透過Cisco DNA Center集中化管理
* 透過進階的虛擬化技術(virtual machine moves, snapshots, and upgrades) 增加網路運作彈性
* Supports Cisco SD-WAN cEdge and vEdge virtual router onboarding
* Supports third-party VNFs

#### Cisco ENFV Solution Architecture

* Management and Orchestration(MANO)
* VNFs
* Network Functions Virtualization Infrastructure Software(NFVIS)
* Hardware resource

![](https://i.imgur.com/eRq4hXr.png)

## Foundational Network Programmability Concepts

### Data Models and Supporting Protocol

* YANG modeling language
* Network Configuration Protocol(NETCONF)
* RESTCONF

API method:
![](https://i.imgur.com/3hQ08f7.png)


#### YANG Data Models

EXAMPLE:
![](https://i.imgur.com/aqHwP2J.png)

#### NETCONF
RFC 4741 and RFC 6241
runs over SSH, TLL and Simple Object Access Protocol(SOAP)
NETCONF可以辨識 configuration data and operational data

SNMP 和 NETCONF比較:
![](https://i.imgur.com/bgt2T6Q.png)
ENTCONF Model:
![](https://i.imgur.com/I3VZDRo.png)
NETCONF Operations:
![](https://i.imgur.com/vVYwobM.png)

#### RESTCONF

* GET
* POST
* PUT
* DELETE
* OPTIONS