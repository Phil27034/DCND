# EVPN Overlay L2 VLAN-based
Схема лабы:

<img width="623" alt="image" src="https://user-images.githubusercontent.com/116812447/212840735-c648b5e8-c271-452c-916b-215b462de33a.png">

Конфиг Leaf-1:

```
router bgp 64701
  router-id 10.10.1.3
  bestpath as-path multipath-relax
  reconnect-interval 12
  address-family ipv4 unicast
    redistribute direct route-map connected
    maximum-paths 64
  address-family l2vpn evpn
    retain route-target all
  neighbor 10.1.1.1
    remote-as 64600
    description Spine-1
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.1.2.1
    remote-as 64600
    description Spine-2
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.10.2.1
    remote-as 64600
    update-source loopback1
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
evpn
  vni 100 l2
    rd 10.10.2.3:100
    route-target import 100:100
    route-target export 100:100
    
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 100
    ingress-replication protocol bgp
    
```

Конфиг Spine-1:
```
router bgp 64600
  router-id 10.10.1.1
  bestpath as-path multipath-relax
  reconnect-interval 12
  address-family ipv4 unicast
    redistribute direct route-map connected
    maximum-paths 64
  address-family l2vpn evpn
    retain route-target all
  neighbor 10.1.1.2
    remote-as 64701
    description Leaf-1
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.1.1.6
    remote-as 64702
    description Leaf-2
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.1.1.10
    remote-as 64703
    description Leaf-3
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.10.2.3
    remote-as 64701
    update-source loopback1
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map SET_NEXT_HOP_UNCHANGED out
  neighbor 10.10.2.4
    remote-as 64702
    update-source loopback1
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map SET_NEXT_HOP_UNCHANGED out
      
 route-map SET_NEXT_HOP_UNCHANGED permit 10
  set ip next-hop unchanged
```

Таблица l2vpn маршрутов на leaf-1, представлены оба типа маршрутов 2 и 3:
 ```
leaf-1# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 49, Local Router ID is 10.10.1.3
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.10.2.3:100    (L2VNI 100)
*>e[2]:[0]:[0]:[48]:[0050.7966.682d]:[0]:[0.0.0.0]/216
                      10.10.2.4                                      0 64600 64702 i
*>l[2]:[0]:[0]:[48]:[0050.7966.683f]:[0]:[0.0.0.0]/216
                      10.10.2.3                         100      32768 i
*>l[3]:[0]:[32]:[10.10.2.3]/88
                      10.10.2.3                         100      32768 i
*>e[3]:[0]:[32]:[10.10.2.4]/88
                      10.10.2.4                                      0 64600 64702 i

Route Distinguisher: 10.10.2.4:100
*>e[2]:[0]:[0]:[48]:[0050.7966.682d]:[0]:[0.0.0.0]/216
                      10.10.2.4                                      0 64600 64702 i
*>e[3]:[0]:[32]:[10.10.2.4]/88
                      10.10.2.4                                      0 64600 64702 i
 ```
Таблица l2vpn маршрутов на leaf-2, представлены оба типа маршрутов 2 и 3:
```
leaf-2# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 53, Local Router ID is 10.10.1.4
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.10.2.3:100
*>e[2]:[0]:[0]:[48]:[0050.7966.683f]:[0]:[0.0.0.0]/216
                      10.10.2.3                                      0 64600 64701 i
*>e[3]:[0]:[32]:[10.10.2.3]/88
                      10.10.2.3                                      0 64600 64701 i

Route Distinguisher: 10.10.2.4:100    (L2VNI 100)
*>l[2]:[0]:[0]:[48]:[0050.7966.682d]:[0]:[0.0.0.0]/216
                      10.10.2.4                         100      32768 i
*>e[2]:[0]:[0]:[48]:[0050.7966.683f]:[0]:[0.0.0.0]/216
                      10.10.2.3                                      0 64600 64701 i
*>e[3]:[0]:[32]:[10.10.2.3]/88
                      10.10.2.3                                      0 64600 64701 i
*>l[3]:[0]:[32]:[10.10.2.4]/88
                      10.10.2.4                         100      32768 i
```

Пинг 192.168.1.3 > Leaf-2 > Leaf-1 > 192.168.1.2 работает:
```
VPCS> ping 192.168.1.2

84 bytes from 192.168.1.2 icmp_seq=1 ttl=64 time=21.790 ms
84 bytes from 192.168.1.2 icmp_seq=2 ttl=64 time=19.371 ms
84 bytes from 192.168.1.2 icmp_seq=3 ttl=64 time=19.969 ms
84 bytes from 192.168.1.2 icmp_seq=4 ttl=64 time=19.421 ms
84 bytes from 192.168.1.2 icmp_seq=5 ttl=64 time=18.218 ms
```
