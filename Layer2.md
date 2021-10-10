# Layer 2

## IEEE 802.1D STP
### Port states:
* Disalbed
* Blocking: only receive BPDUs
* Listening: send and receive BPDUs
* Learning: can modiiiify the MAC address table, but still dose not forward any other network traffic besides BPDUs.
* Forwarding: forward all network traffic and can update the MAC address table.
* Broken
### Port Types
* Root port(RP)
* Designated port(DP)
* Blocking port
### Root Bridge selection
1. Bridge ID 最小的(Priority =>　MAC address)成為root
2. Root cost
3. Bridge ID
4. Port ID
## RSTP (802.1W)
Use synchronization process (need full-duplex)


### Port States
* Discarding
* Learning
* Forwarding
### Port Roles
* Root port
* Designated port
* Alternate port
* Backup port: 僅存在同台switch上有多條Link
## Topology Change
* Portfast: skip Learning and Listening to Forwarding. When port receive BPDU, the port state will enter Listening and Learning.
* UplinkFast: Select the blocking port is standy when the root port is shutdown.
* BackboneFast: Since root port send RLQ and recive RLQ reply to make sure the root siwtch is on.

## STP Protection mechanisms

* Root Guard: prevents a configured port from becoming a root port. The state show ErrDisabled.
* STP Portfast: 多用於end devices, 通常會和BPDU Guard一起設定
* BPDU Guard: 預防其他設備發送BPDU，可防止他人偷接switch。
* BPDU Filter: 防止local switch發送BPDU
* STP Loop Guard: 預防沒收到BPDU(指hardware)時造成loop
* Unidirectional Link Detection(UDLD): 用於預防port state 正常但實際上有出現資料遺漏的狀況。

## MST
* 解決RSTP消耗過多硬體資源
* 增加可管理性
* 規劃上可以達到不同VLAN分流

### 區域 Region
Switch 比較以下參數判斷是否為同一個Region:
* Configuration Name
* Revision Number
* VLAN and Instance mapping

### Common and Internal Spanning Tree (CIST)
出現於有兩個Region 以上的STP架構，會將所有Region內，Bridge ID最小的switch作為CIST。

### 與RSTP/STP 共存

MST會自行判BPDU，若remote switch設定RSTP，則將訊息拆成 Per VLAN發送。但CIST無法由RSTP/STP的switch擔任。

## VTP
模式分為Server、Client、Transport和Off。
只有Server能夠異動vlan database。
當domain內有兩個server，其中 revision number 最高的會被選為master，另一server就會與其同步。

## DTP
用於自動trunk協定，Desriable 主動，auto為被動


## EtherChannel Bundle

### LACP
multicast MAC address 0180:C200:0002
Port mode:Passive、Active


### PAgP
multicast MAC address 0100:0CCC:CCCC
port mode:Auto、Desirable