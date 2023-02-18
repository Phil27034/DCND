# Настройка vPC

## Схема
<img width="1366" alt="image" src="https://user-images.githubusercontent.com/116812447/219690642-19d22a0b-c3c1-4cc0-9c83-032477724f94.png">

## Настройка vPC на основном узле Leaf-1
```
feature vpc

vpc domain 8
  #Активация Peer switch - оба узла отвечают на BPDU
  peer-switch
  
  #Приоритет для основной ноды
  role priority 5
  
  #Настройка peer keep alive link
  peer-keepalive destination 172.16.16.1 source 172.16.16.2 vrf VPC
  
  #Задержка поднятия vPC после перезагрузки устройства
  delay restore 10
  
  #Включение Peer gateway - резервный узел может маршрутизировать пакеты предназначенные основному узлу
  peer-gateway
  
  #Если роутер подключен к 1 свитчу, его траффик будет проходить корректно
  layer3 peer-router
  
  #Синхронизация ARP таблицы между узлами
  ip arp synchronize

#LAG для подколючения peer-link
interface port-channel1
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

#LAG для подключения конечного хоста
interface port-channel101
  switchport access vlan 100
  vpc 8
  
 interface loopback0
  ip address 10.10.2.3/32
  ## Virtual VTEP ID - Anycast VTEP
  ip address 10.10.2.6/32 secondary
  
 #Интерфейс peer keep alive
 interface Ethernet1/4
  no switchport
  vrf member VPC
  ip address 172.16.16.2/30
  no shutdown

#Включение VRF VPC для peer keep alive
vrf context VPC

#Интерфейс подключаемый к конечному узлу
interface Ethernet1/2
  switchport access vlan 100
  channel-group 101 mode active
  
 #Интерфейс для peer link
 interface Ethernet1/6
  switchport mode trunk
  channel-group 1 mode active
  
#Второй интерфейс для peerlink
interface Ethernet1/5
  switchport mode trunk
  channel-group 1 mode active
```

На Leaf-2 настройки абсалютно идентичны.

## Настройки Port-Channel на конечном узле (Nexus)
```
ip route 0.0.0.0/0 192.168.1.1

interface Vlan2
  no shutdown
  ip address 192.168.1.2/24

interface port-channel1
  switchport access vlan 2

interface Ethernet1/1
  switchport access vlan 2
  channel-group 1 mode active

interface Ethernet1/2
  switchport access vlan 2
  channel-group 1 mode active
```

## Проверка настроек и работоспособности на Leaf-1
Состояние vPC
```
leaf-1# show vpc
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 8
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : primary, operational secondary
Number of vPCs configured         : 1
Peer Gateway                      : Enabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 10s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Enabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     1,100,200,300


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
8     Po101         up     success     success               100

```
## Проверка связи со всеми узлами с узла Client-1
```
switch# ping 192.168.1.1
PING 192.168.1.1 (192.168.1.1): 56 data bytes
64 bytes from 192.168.1.1: icmp_seq=0 ttl=254 time=27.412 ms
64 bytes from 192.168.1.1: icmp_seq=1 ttl=254 time=8.777 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=254 time=7.359 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=254 time=8.414 ms
64 bytes from 192.168.1.1: icmp_seq=4 ttl=254 time=7.899 ms

--- 192.168.1.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 7.359/11.972/27.412 ms
switch# ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2): 56 data bytes
64 bytes from 192.168.1.2: icmp_seq=0 ttl=255 time=3.742 ms
64 bytes from 192.168.1.2: icmp_seq=1 ttl=255 time=9.352 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=255 time=5.014 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=255 time=2.425 ms
64 bytes from 192.168.1.2: icmp_seq=4 ttl=255 time=5.631 ms

--- 192.168.1.2 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 2.425/5.232/9.352 ms
switch# ping 192.168.1.3
PING 192.168.1.3 (192.168.1.3): 56 data bytes
64 bytes from 192.168.1.3: icmp_seq=0 ttl=254 time=29.024 ms
64 bytes from 192.168.1.3: icmp_seq=1 ttl=254 time=36.468 ms
64 bytes from 192.168.1.3: icmp_seq=2 ttl=254 time=26.779 ms
64 bytes from 192.168.1.3: icmp_seq=3 ttl=254 time=33.06 ms
64 bytes from 192.168.1.3: icmp_seq=4 ttl=254 time=27.185 ms

--- 192.168.1.3 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 26.779/30.503/36.468 ms
switch# ping 192.168.2.2
PING 192.168.2.2 (192.168.2.2): 56 data bytes
64 bytes from 192.168.2.2: icmp_seq=0 ttl=253 time=25.401 ms
64 bytes from 192.168.2.2: icmp_seq=1 ttl=253 time=14.12 ms
64 bytes from 192.168.2.2: icmp_seq=2 ttl=253 time=26.393 ms
64 bytes from 192.168.2.2: icmp_seq=3 ttl=253 time=23.502 ms
64 bytes from 192.168.2.2: icmp_seq=4 ttl=253 time=23.743 ms

--- 192.168.2.2 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 14.12/22.631/26.393 ms
switch# ping 192.168.2.3
PING 192.168.2.3 (192.168.2.3): 56 data bytes
64 bytes from 192.168.2.3: icmp_seq=0 ttl=252 time=21.053 ms
64 bytes from 192.168.2.3: icmp_seq=1 ttl=252 time=19.934 ms
64 bytes from 192.168.2.3: icmp_seq=2 ttl=252 time=27.832 ms
64 bytes from 192.168.2.3: icmp_seq=3 ttl=252 time=21.514 ms
64 bytes from 192.168.2.3: icmp_seq=4 ttl=252 time=29.598 ms

--- 192.168.2.3 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 19.934/23.986/29.598 ms
```

## Проверка маршрутов на Leaf-2
Все MAC адреса видны за Anycast VTEP
```
switch# show l2route mac all

Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote (V):vPC link
(Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
(S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
(Pf):Permanently-Frozen, (Orp): Orphan

Topology    Mac Address    Prod   Flags         Seq No     Next-Hops
----------- -------------- ------ ------------- ---------- ---------------------------------------
100         5009.0000.1b08 BGP    SplRcv        0          10.10.2.6 (Label: 100)
100         aabb.cc00.6000 Local  L,            0          Eth1/2
200         aabb.cc00.4000 BGP    SplRcv        0          10.10.2.6 (Label: 200)
200         aabb.cc00.7000 Local  L,            0          Eth1/3
300         5002.0000.1b08 VXLAN  Rmac          0          10.10.2.6
```
