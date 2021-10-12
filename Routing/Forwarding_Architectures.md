# Forwarding Architectures

## Process Switching
Software switching or slow path.
當封包沒辦法經過CEF switch, 會再送往CPU中的ip_input，進行轉送


## Cisco Express Forwarding (CEF)
分成software-based(主要還是在CPU) 和 hardware-based(NPUs)
由FIB(從RIB上更新)和Adjacency table(Nexthop)組成


### Centralized Forwarding
統一由RP轉送


### Distributed Forwarding
當封包進入ingress line card, 會先在local forwarding engine進行查看，若outbound interface 是local，則會在直接從原介面出去，不是再送往RP處理。

## Stateful Switchover (SSO)

當一RP出現問題時，RIB的資料就會被reset，造成掉包的狀況，這段期間只能暫時使用CEF並等待第二個RP建立好RIB資料。

SSO 提供Redundancy功能，使兩RPs進行同步。
啟用nonstop forwarding(NSF) or nonstop routing(NSR)，可以在RP出問題時維持CEF 轉送並等待恢復

## SDM Templates

switching database manager(SDM)，可以被設定在c9k switch
如果有做stack，每台switch SDM需皆相同