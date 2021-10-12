# Route Map

## Conditional Matching
### ACLs
Like basic firewall and provide QOS etc.
#### Starndard ACLs
```
ip access-list standard {acl-number | acl-name}
[sequence] {permit | deny} {source} {source-wildcard}
```


#### Extended ACLs


```
ip access-list extended {acl-number | acl-name} 
[sequence] {permit | deny} protocol source source-wildcard destination destination-wildcard
```

### Prefix Matching
* le: <=
* ge: >=
* both: le and ge can also be used together

![](https://i.imgur.com/bpgVpET.png)

![](https://i.imgur.com/3j0Resf.png)

```
ip prefix-list prefix-list-name [seq sequence-number]
{permit | deny} high-order-bit-pattern/high-order-bit-count
[ge ge-value] [le le-value]
```

Example:

```
ip prefix-list RFC1918 seq 5 permit 192.168.0.0/13 ge 32
ip prefix-list RFC1918 seq 10 deny 0.0.0.0/0 ge 32
ip prefix-list RFC1918 seq 15 permit 10.0.0.0/8 le 32
ip prefix-list RFC1918 seq 20 permit 172.16.0.0/12 le 32
ip prefix-list RFC1918 seq 25 permit 192.168.0.0/16 le 32
```


## Route Maps

four components:
* Sequence number
* Conditional matching criterion
* Processing action
* Optional action

`route-map route-map-name [permit | deny] [sequence-number]`

```
route-map EXAMPLE permit 10
 match ip address ACL-ONE
! Prefixes that match ACL-ONE are permitted. Route-map completes processing upon a match
route-map EXAMPLE deny 20
 match ip address ACL-TWO
! Prefixes that match ACL-TWO are denied. Route-map completes processing upon a match
route-map EXAMPLE permit 30
 match ip address ACL-THREE
 set metric 20
! Prefixes that match ACL-THREE are permitted and modify the metric. Route-map completes
! processing upon a match
route-map EXAMPLE permit 40
! Because a matching criteria was not specified, all other prefixes are permitted
! If this sequence was not configured, all other prefixes would drop because of the
! implicit deny for all route-maps
```



## Conditional Forwarding of Packets

Policy-based routing(PBR):
* Routing by protocol typye(ICMP、TCP、UDP)
* Routing by source IP address, destination IP address, or both
* Manual assignment of different network paths to the same destination, based on tolerance for latency, link speed, or utilization for specific transient traffi

Drawback:
* Administrative burden in scalability
* Lack of network intelligence
* Troubleshooting complexity

### PBR Configuration

Example:
```
R2
ip access-list extended ACL-PBR
 permit ip 10.1.1.0 0.0.0.255 10.5.5.0 0.0.0.255
!
route-map PBR-TRANSIT permit 10
 match ip address ACL-PBR
 set ip next-hop 10.23.1.3
!
interface GigabitEthernet0/1
 ip address 10.12.1.2 255.255.255.0
 ip policy route-map PBR-TRANSIT
```
Route map 不會修改Routing table

### Local PBR

```
R1
ip access-list extended ACL-MANAGEMENT-LOCAL-PBR
 permit permit ip 172.16.14.0 0.0.0.255 any
!
route-map LOCAL-PBR permit 10
 match ip address ACL-LOCAL-PBR
 set ip next-hop 172.16.14.4
!
ip local policy route-map LOCAL-PBR
```