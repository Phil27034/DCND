# Настройка OSPF underlay

Протокол OSPF и Loopback интерфейсы настроены на всех устройствах согласно плану IP адресации.  
Конфигурация на всех устройствах одинаковая, ниже приведен конфиг Spine-1.

```
spine-1# sh run ospf

!Command: show running-config ospf
!Running configuration last done at: Wed Nov  9 19:11:28 2022
!Time: Sat Nov 12 11:47:07 2022

version 9.3(10) Bios:version
feature ospf

router ospf underlay
  router-id 10.10.1.1

interface loopback0
  ip router ospf underlay area 0.0.0.0

interface Ethernet1/1
  ip ospf authentication message-digest
  ip ospf authentication key-chain ospf
  ip ospf network point-to-point
  ip router ospf underlay area 0.0.0.0

interface Ethernet1/2
  ip ospf authentication message-digest
  ip ospf authentication key-chain ospf
  ip ospf network point-to-point
  ip router ospf underlay area 0.0.0.0

interface Ethernet1/3
  ip ospf authentication message-digest
  ip ospf authentication key-chain ospf
  ip ospf network point-to-point
  ip router ospf underlay area 0.0.0.0
```
Все маршруты доступны на Spine-1:
```
spine-1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.1.0/30, ubest/mbest: 1/0, attached
    *via 10.1.1.1, Eth1/1, [0/0], 4d06h, direct
10.1.1.1/32, ubest/mbest: 1/0, attached
    *via 10.1.1.1, Eth1/1, [0/0], 4d06h, local
10.1.1.4/30, ubest/mbest: 1/0, attached
    *via 10.1.1.5, Eth1/2, [0/0], 4d04h, direct
10.1.1.5/32, ubest/mbest: 1/0, attached
    *via 10.1.1.5, Eth1/2, [0/0], 4d04h, local
10.1.1.8/30, ubest/mbest: 1/0, attached
    *via 10.1.1.9, Eth1/3, [0/0], 2d17h, direct
10.1.1.9/32, ubest/mbest: 1/0, attached
    *via 10.1.1.9, Eth1/3, [0/0], 2d17h, local
10.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.2, Eth1/1, [110/80], 2d19h, ospf-underlay, intra
10.1.2.4/30, ubest/mbest: 1/0
    *via 10.1.1.6, Eth1/2, [110/80], 2d19h, ospf-underlay, intra
10.1.2.8/30, ubest/mbest: 1/0
    *via 10.1.1.10, Eth1/3, [110/80], 2d17h, ospf-underlay, intra
10.10.1.1/32, ubest/mbest: 2/0, attached
    *via 10.10.1.1, Lo0, [0/0], 2d16h, local
    *via 10.10.1.1, Lo0, [0/0], 2d16h, direct
10.10.1.2/32, ubest/mbest: 3/0
    *via 10.1.1.2, Eth1/1, [110/81], 2d19h, ospf-underlay, intra
    *via 10.1.1.6, Eth1/2, [110/81], 2d19h, ospf-underlay, intra
    *via 10.1.1.10, Eth1/3, [110/81], 2d17h, ospf-underlay, intra
10.10.1.3/32, ubest/mbest: 1/0
    *via 10.1.1.2, Eth1/1, [110/41], 2d19h, ospf-underlay, intra
10.10.1.4/32, ubest/mbest: 1/0
    *via 10.1.1.6, Eth1/2, [110/41], 4d03h, ospf-underlay, intra
10.10.1.5/32, ubest/mbest: 1/0
    *via 10.1.1.10, Eth1/3, [110/41], 2d17h, ospf-underlay, intra
```
Пинг доходит до всех устройств:
```
spine-1# ping 10.10.1.3
PING 10.10.1.3 (10.10.1.3): 56 data bytes
64 bytes from 10.10.1.3: icmp_seq=0 ttl=254 time=9.393 ms
64 bytes from 10.10.1.3: icmp_seq=1 ttl=254 time=3.052 ms
64 bytes from 10.10.1.3: icmp_seq=2 ttl=254 time=2.827 ms
64 bytes from 10.10.1.3: icmp_seq=3 ttl=254 time=4.52 ms
64 bytes from 10.10.1.3: icmp_seq=4 ttl=254 time=2.83 ms

--- 10.10.1.3 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 2.827/4.524/9.393 ms
spine-1# ping 10.10.1.4
PING 10.10.1.4 (10.10.1.4): 56 data bytes
64 bytes from 10.10.1.4: icmp_seq=0 ttl=254 time=5.936 ms
64 bytes from 10.10.1.4: icmp_seq=1 ttl=254 time=4.271 ms
64 bytes from 10.10.1.4: icmp_seq=2 ttl=254 time=2.682 ms
64 bytes from 10.10.1.4: icmp_seq=3 ttl=254 time=3.095 ms
64 bytes from 10.10.1.4: icmp_seq=4 ttl=254 time=2.186 ms

--- 10.10.1.4 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 2.186/3.634/5.936 ms
spine-1# ping 10.10.1.5
PING 10.10.1.5 (10.10.1.5): 56 data bytes
64 bytes from 10.10.1.5: icmp_seq=0 ttl=254 time=8.018 ms
64 bytes from 10.10.1.5: icmp_seq=1 ttl=254 time=3.747 ms
64 bytes from 10.10.1.5: icmp_seq=2 ttl=254 time=4.168 ms
64 bytes from 10.10.1.5: icmp_seq=3 ttl=254 time=3.899 ms
64 bytes from 10.10.1.5: icmp_seq=4 ttl=254 time=4.968 ms

--- 10.10.1.5 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 3.747/4.96/8.018 ms
spine-1# ping 10.10.1.2
PING 10.10.1.2 (10.10.1.2): 56 data bytes
64 bytes from 10.10.1.2: icmp_seq=0 ttl=253 time=8.002 ms
64 bytes from 10.10.1.2: icmp_seq=1 ttl=253 time=5.761 ms
64 bytes from 10.10.1.2: icmp_seq=2 ttl=253 time=6.41 ms
64 bytes from 10.10.1.2: icmp_seq=3 ttl=253 time=4.029 ms
64 bytes from 10.10.1.2: icmp_seq=4 ttl=253 time=4.033 ms

--- 10.10.1.2 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 4.029/5.646/8.002 ms
```
