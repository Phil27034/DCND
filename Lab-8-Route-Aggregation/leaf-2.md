```
!Command: show running-config
!Running configuration last done at: Sun Mar  5 14:35:15 2023
!Time: Sun Mar  5 19:38:30 2023

version 9.3(10) Bios:version  
hostname leaf-2
vdc leaf-2 id 1
  limit-resource vlan minimum 16 maximum 4094
  limit-resource vrf minimum 2 maximum 4096
  limit-resource port-channel minimum 0 maximum 511
  limit-resource u4route-mem minimum 248 maximum 248
  limit-resource u6route-mem minimum 96 maximum 96
  limit-resource m4route-mem minimum 58 maximum 58
  limit-resource m6route-mem minimum 8 maximum 8

nv overlay evpn
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

no password strength-check
username admin password 5 $5$PLPFPN$TbtzYdPw8oY1gmrXMra7S3Ud1vGE.5/TK3LcsrCcdn8  role network-admin
ip domain-lookup
copp profile strict
snmp-server user admin network-admin auth md5 37476FC8D281731195F9DC04771DEFA4F7F5 priv 005431DD90CC21188CD8924F7A0FB9CCF9EC localizedV2key
rmon event 1 log trap public description FATAL(1) owner PMON@FATAL
rmon event 2 log trap public description CRITICAL(2) owner PMON@CRITICAL
rmon event 3 log trap public description ERROR(3) owner PMON@ERROR
rmon event 4 log trap public description WARNING(4) owner PMON@WARNING
rmon event 5 log trap public description INFORMATION(5) owner PMON@INFO

fabric forwarding anycast-gateway-mac 0001.0002.0003
vlan 1,100,200,300,500,600
vlan 100
  vn-segment 100
vlan 200
  vn-segment 200
vlan 300
  vn-segment 300
vlan 500
  vn-segment 500
vlan 600
  vn-segment 600

vrf context OTUS
  vni 300
  rd 10.10.2.4:300
  address-family ipv4 unicast
    route-target import 300:300
    route-target import 300:300 evpn
    route-target export 300:300
    route-target export 300:300 evpn
vrf context OTUS2
  vni 500
  rd 10.10.2.4:500
  address-family ipv4 unicast
    route-target import 500:500
    route-target import 500:500 evpn
    route-target export 500:500
    route-target export 500:500 evpn
vrf context management
hardware access-list tcam region racl 512
hardware access-list tcam region arp-ether 256 double-wide


interface Vlan1

interface Vlan100
  no shutdown
  vrf member OTUS
  ip address 192.168.1.1/24
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member OTUS
  ip address 192.168.2.1/24
  fabric forwarding mode anycast-gateway

interface Vlan300
  no shutdown
  vrf member OTUS
  ip forward

interface Vlan400

interface Vlan500
  no shutdown
  vrf member OTUS2
  ip forward

interface Vlan600
  no shutdown
  vrf member OTUS2
  ip address 192.168.4.1/24
  fabric forwarding mode anycast-gateway

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 100
    ingress-replication protocol bgp
  member vni 200
    ingress-replication protocol bgp
  member vni 300 associate-vrf
  member vni 500 associate-vrf
  member vni 600
    ingress-replication protocol bgp

interface Ethernet1/1
  no switchport
  ip address 10.1.1.6/30
  no shutdown

interface Ethernet1/2
  switchport access vlan 100

interface Ethernet1/3
  switchport access vlan 200

interface Ethernet1/4
  switchport access vlan 600

interface Ethernet1/5

interface Ethernet1/6

interface Ethernet1/7

interface Ethernet1/8

interface Ethernet1/9

interface Ethernet1/10

interface Ethernet1/11

interface Ethernet1/12

interface Ethernet1/13

interface Ethernet1/14

interface Ethernet1/15

interface Ethernet1/16

interface Ethernet1/17

interface Ethernet1/18

interface Ethernet1/19

interface Ethernet1/20

interface Ethernet1/21

interface Ethernet1/22

interface Ethernet1/23

interface Ethernet1/24

interface Ethernet1/25

interface Ethernet1/26

interface Ethernet1/27

interface Ethernet1/28

interface Ethernet1/29

interface Ethernet1/30

interface Ethernet1/31

interface Ethernet1/32

interface Ethernet1/33

interface Ethernet1/34

interface Ethernet1/35

interface Ethernet1/36

interface Ethernet1/37

interface Ethernet1/38

interface Ethernet1/39

interface Ethernet1/40

interface Ethernet1/41

interface Ethernet1/42

interface Ethernet1/43

interface Ethernet1/44

interface Ethernet1/45

interface Ethernet1/46

interface Ethernet1/47

interface Ethernet1/48

interface Ethernet1/49

interface Ethernet1/50

interface Ethernet1/51

interface Ethernet1/52

interface Ethernet1/53

interface Ethernet1/54

interface Ethernet1/55

interface Ethernet1/56

interface Ethernet1/57

interface Ethernet1/58

interface Ethernet1/59

interface Ethernet1/60

interface Ethernet1/61

interface Ethernet1/62

interface Ethernet1/63

interface Ethernet1/64

interface mgmt0
  vrf member management

interface loopback0
  ip address 10.10.2.4/32

interface loopback1
  ip address 10.10.1.5/32
icam monitor scale

line console
line vty
boot nxos bootflash:/nxos.9.3.10.bin sup-1
router bgp 64702
  router-id 10.10.2.4
  address-family ipv4 unicast
    network 10.10.1.5/32
    network 10.10.2.4/32
  neighbor 10.1.1.5
    remote-as 64600
    address-family ipv4 unicast
  neighbor 10.10.2.1
    remote-as 64600
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      allowas-in 3
      send-community
      send-community extended
evpn
  vni 100 l2
    rd 10.10.2.4:100
    route-target import 100:100
    route-target export 100:100
  vni 200 l2
    rd 10.10.2.4:200
    route-target import 200:200
    route-target export 200:200
  vni 600 l2
    rd 10.10.2.4:600
    route-target import 600:600
    route-target export 600:600

no logging console
```
