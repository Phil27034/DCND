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
