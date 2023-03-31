# Проектная работа
Все конфигурационные файлы находятся в папке проекта.

## Схема стенда

<img width="1751" alt="image" src="https://user-images.githubusercontent.com/116812447/228843950-8427ee16-29a5-45e1-bfbd-a48077c771bb.png">

## Underlay
- Протокол маршрутизации - eBGP
- Router-ID задается вручную
- Leaf анонсирует BGP EVPN loopback, NVE loopback, NVE vPC virtual loopback
- Spine анонсирует только свой loopback для построения соседств BGP EVPN
- BFD используется по дизайну, но не стенде, так как виртуальные Nexus не поддерживают

### Нумерация автономных систем
- Нумерация берется из Private Range
- Спайны - 64600
- Лифы - 64701-64703

### Конфиг Spine

```
router bgp 64600
  router-id 10.10.2.1
  address-family ipv4 unicast
    network 10.10.2.1/32
  neighbor 10.1.1.2
    remote-as 64701
    address-family ipv4 unicast
  neighbor 10.1.1.6
    remote-as 64702
    address-family ipv4 unicast
  neighbor 10.1.1.10
    remote-as 64701
    address-family ipv4 unicast
  neighbor 10.1.1.14
    remote-as 64703
    address-family ipv4 unicast
```

### Конфиг Leaf

```
feature bgp

router bgp 64701
  router-id 10.10.2.5
  address-family ipv4 unicast
    network 10.10.1.2/32
    network 10.10.1.3/32
    network 10.10.2.5/32
  neighbor 10.1.1.9
    remote-as 64600
    address-family ipv4 unicast
```

## Overlay
- Симметричная маршрутизация
- Активирован Anycast gateway
- Virtual MAC
- Суммаризация маршрутов на Border Leaf перед отправкой на внешние устройства
- BGP EVPN update идут с loopback0
- Включен multihop
- Включены address-family l2vpn, communities, extended communities
- Назначены L2 VNI для каждого VLAN
- Каждому L2 VNI назначен RD, RT
- Созданы VLAN для L3 VNI
- Созданы неообходимые VRF, им назначены RD, RT, L3 VNI, в VRF помещены нужные интерфейсы
- Создан NVE интерфейс, в качестве метода поиска узлов выбран BGP, все VNI прописаны с методом обработки BUM траффика - ingress replication
- На Spine применена директива retain route-target all
- На Spine применен route-map, который сохраняет адрес next-hop

### Конфиг Leaf

```

fabric forwarding anycast-gateway-mac 0001.0002.0003

vlan 1,100,200,300,400,500

vlan 100
  vn-segment 100
vlan 200
  vn-segment 200
vlan 300
  vn-segment 300
vlan 400
  vn-segment 400
vlan 500
  vn-segment 500

router bgp 64701
  router-id 10.10.2.5
  neighbor 10.10.2.1
    remote-as 64600
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
evpn
  vni 100 l2
    rd 10.10.2.5:100
    route-target import 100:100
    route-target export 100:100
  vni 200 l2
    rd 10.10.2.5:200
    route-target import 200:200
    route-target export 200:200
  vni 400 l2
    rd 10.10.2.5:400
    route-target import 400:400
    route-target export 400:400
vrf context OTUS
  rd 10.10.2.5:300
  address-family ipv4 unicast
    route-target import 300:300
    route-target import 300:300 evpn
    route-target export 300:300
    route-target export 300:300 evpn
vrf context OTUS2
  rd 10.10.2.3:500
  address-family ipv4 unicast
    route-target import 500:500
    route-target import 500:500 evpn
    route-target export 500:500
    route-target export 500:500 evpn

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 100
    ingress-replication protocol bgp
  member vni 200
    ingress-replication protocol bgp
  member vni 300 associate-vrf
  member vni 400
    ingress-replication protocol bgp
  member vni 500 associate-vrf
```

### Конфиг Spine

```
router bgp 64600
  router-id 10.10.2.1
   address-family l2vpn evpn
    retain route-target all

neighbor 10.10.2.3
    remote-as 64701
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map SET_NEXT_HOP_UNCHANGED out
  neighbor 10.10.2.4
    remote-as 64702
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map SET_NEXT_HOP_UNCHANGED out
  neighbor 10.10.2.5
    remote-as 64701
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map SET_NEXT_HOP_UNCHANGED out
  neighbor 10.10.2.7
    remote-as 64703
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      allowas-in 3
      send-community
      send-community extended
      route-map SET_NEXT_HOP_UNCHANGED out
```

## vPC

Конфиг

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

## Связь с Firewall
- Cвязь реализована в виде 2 саб интерфейсов - по одному для каждого VRF
- Маршруты фабрики отправляемые на файерволл суммаризуются
- На Border Leaf в VRF Backed END - OTUS 2 редистрибьютятся маршруты OSPF из кампуса
- Firewall транслирует в фабрику адреса удаленных сетей и шлюз по умолчанию

### Конфиг Border Leaf

```
interface Ethernet1/2.2
  encapsulation dot1q 2
  vrf member OTUS
  ip address 192.168.5.1/30
  no shutdown

interface Ethernet1/2.3
  encapsulation dot1q 3
  vrf member OTUS2
  ip address 192.168.6.1/30
  no shutdown

router bgp 64703
  router-id 10.10.2.7

  vrf OTUS
    address-family ipv4 unicast
      aggregate-address 192.168.1.0/24 as-set summary-only
      aggregate-address 192.168.2.0/24 as-set summary-only
    neighbor 192.168.5.2
      remote-as 64800
      local-as 64705 no-prepend replace-as
      address-family ipv4 unicast
    
  vrf OTUS2
    address-family ipv4 unicast
      redistribute ospf CAMPUS route-map BGP
      aggregate-address 192.168.3.0/24 as-set summary-only
      aggregate-address 192.168.4.0/24 as-set summary-only
    neighbor 192.168.6.2
      remote-as 64800
      local-as 64704 no-prepend replace-as
      address-family ipv4 unicast
```

### Конфиг Firewall

```
interface Loopback0
 ip address 1.1.1.1 255.255.255.0
!
interface Loopback1
 ip address 1.1.2.1 255.255.255.0
!
interface Loopback2
 ip address 1.1.3.1 255.255.255.0

 interface Ethernet0/0.2
 encapsulation dot1Q 2
 ip address 192.168.5.2 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/0.3
 encapsulation dot1Q 3
 ip address 192.168.6.2 255.255.255.252

 interface Ethernet0/2
 ip address 3.3.3.1 255.255.255.252
 ip nat outside
 ip virtual-reassembly in


 router bgp 64800
 bgp log-neighbor-changes
 network 1.1.1.0 mask 255.255.255.0
 network 1.1.2.0 mask 255.255.255.0
 network 1.1.3.0 mask 255.255.255.0
 network 159.33.0.0 mask 255.255.255.0
 neighbor 3.3.3.2 remote-as 10000
 neighbor 3.3.3.2 route-map BLOCK_INTERNAL out
 neighbor 192.168.5.1 remote-as 64705
 neighbor 192.168.5.1 default-originate
 neighbor 192.168.5.1 route-map BLOCK_EXTERNAL out
 neighbor 192.168.6.1 remote-as 64704
 neighbor 192.168.6.1 default-originate
 neighbor 192.168.6.1 distribute-list 1 out

ip nat inside source list 2 interface Ethernet0/2 overload
ip nat inside source static 192.168.1.2 159.33.0.100
ip route 0.0.0.0 0.0.0.0 3.3.3.2
ip route 159.33.0.0 255.255.255.0 192.168.5.1
!
!
ip prefix-list BLOCK_EXTERNAL seq 10 deny 159.33.0.0/24
ip prefix-list BLOCK_EXTERNAL seq 20 permit 0.0.0.0/0 le 32
!
ip prefix-list BLOCK_INTERNAL seq 25 permit 159.33.0.0/24
access-list 1 deny   159.33.0.0 0.0.0.255
access-list 1 permit any
access-list 2 permit 192.168.0.0 0.0.255.255
!
route-map BLOCK_EXTERNAL permit 10
 match ip address prefix-list BLOCK_EXTERNAL
!
route-map BLOCK_INTERNAL permit 10
 match ip address prefix-list BLOCK_INTERNAL

```

## Связь с Интернетом
- AS 64800 - 159.33.0.0/24
- Тестовый адрес для подключения из Интернет - 159.33.0.100
- Адрес натируется в 192.168.1.2, который находится в фабрике в VRF OTUS - Frontend
- Все обращения фабрики в интернет проходят через процедуру SNAT для соотвествующего провайдера
- Firewall отправляет в фабрику шлюз по умолчанию - во все VRF
- В сторону провайдеров Firewall отправляет только внешнюю сеть 159.33.0.0/24
- Внутрь внешняя сеть не транслируется
- От провайдеров Firewall блокирует любые апдейты, шлюз по умолчанию в сторону провайдеров настроен статически

Таблица маршрутизации провайдера ISP1, получен от Firewall только маршрут внешних сетей:
```
Router#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      3.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        3.3.3.0/30 is directly connected, Ethernet0/0
L        3.3.3.2/32 is directly connected, Ethernet0/0
      159.33.0.0/24 is subnetted, 1 subnets
B        159.33.0.0 [20/0] via 3.3.3.1, 14:16:46
```

Пинг с провайдера 1 до внешнего корпоративного адреса, который транслируется в фабрику:

```
Router#ping 159.33.0.100
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 159.33.0.100, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 43/70/87 ms
```

Доступ в Интернет из фабрики, VRF OTUS:

```
host-5#ping 3.3.3.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 3.3.3.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 14/22/42 ms
```

## Связь с кампусом
- Кампус соеденен с VRF OTUS2 по OSPF
- VRF OTUS2 отправляет в Кампус все маршруты фабрики и удаленных сетей
- Кампус отправляет в OTUS2 все кампусные сети
- Кампус не получает от VRF OTUS2 шлюз по умолчанию, у него свой выход в Интернет

Конфиг кампуса

```
interface Loopback0
 ip address 2.1.1.1 255.255.255.0
!
interface Loopback1
 ip address 2.1.2.1 255.255.255.0
!
interface Loopback2
 ip address 2.1.3.1 255.255.255.0

interface Ethernet0/1
 ip address 192.168.7.2 255.255.255.252
 ip ospf network point-to-point

 router ospf 2
 network 2.0.0.0 0.255.255.255 area 0
 network 192.168.7.0 0.0.0.3 area 0

```

 Таблица маршрутов роутера Кампуса, где есть все маршруты до фабрики, удаленных сетей и нет шлюза по умолчанию

 ```
campus_router#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      1.0.0.0/24 is subnetted, 3 subnets
O E2     1.1.1.0 [110/1] via 192.168.7.1, 00:05:16, Ethernet0/1
O E2     1.1.2.0 [110/1] via 192.168.7.1, 00:05:04, Ethernet0/1
O E2     1.1.3.0 [110/1] via 192.168.7.1, 00:00:04, Ethernet0/1
      2.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
C        2.1.1.0/24 is directly connected, Loopback0
L        2.1.1.1/32 is directly connected, Loopback0
C        2.1.2.0/24 is directly connected, Loopback1
L        2.1.2.1/32 is directly connected, Loopback1
C        2.1.3.0/24 is directly connected, Loopback2
L        2.1.3.1/32 is directly connected, Loopback2
O E2  192.168.1.0/24 [110/1] via 192.168.7.1, 00:52:52, Ethernet0/1
O E2  192.168.2.0/24 [110/1] via 192.168.7.1, 00:09:16, Ethernet0/1
O E2  192.168.3.0/24 [110/1] via 192.168.7.1, 00:09:11, Ethernet0/1
O E2  192.168.4.0/24 [110/1] via 192.168.7.1, 00:08:59, Ethernet0/1
      192.168.7.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.7.0/30 is directly connected, Ethernet0/1
L        192.168.7.2/32 is directly connected, Ethernet0/1
 ```

 Таблица маршрутов VRF OTUS2, где есть все кампусные сети

 ```
border-leaf# show ip route vrf OTUS2 | grep 2\.1\.
2.1.1.1/32, ubest/mbest: 1/0
2.1.2.1/32, ubest/mbest: 1/0
2.1.3.1/32, ubest/mbest: 1/0
 ```

Пинг из VRF OTUS до всех сетей Кампуса

```
host-5#ping ip 2.1.1.1 repeat 1
Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 2.1.1.1, timeout is 2 seconds:
!
Success rate is 100 percent (1/1), round-trip min/avg/max = 113/113/113 ms
host-5#ping ip 2.1.2.1 repeat 1
Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 2.1.2.1, timeout is 2 seconds:
U
Success rate is 0 percent (0/1)
host-5#ping ip 2.1.3.1 repeat 1
Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 2.1.3.1, timeout is 2 seconds:
U
Success rate is 0 percent (0/1)
host-5#
```

Пинг из VRF OTUS2 до всех сетей кампуса

```
host-6#ping ip 2.1.1.1 repeat 1
Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 2.1.1.1, timeout is 2 seconds:
!
Success rate is 100 percent (1/1), round-trip min/avg/max = 120/120/120 ms
host-6#ping ip 2.1.1.1 repeat 1
*Mar 31 12:25:26.798: %CDP-4-DUPLEX_MISMATCH: duplex mismatch discovered on Ethernet0/0 (not full duplex), with leaf-2(9LBFM56FPTE) Ethernet1/4 (full duplex).
host-6#ping ip 2.1.2.1 repeat 1
Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 2.1.2.1, timeout is 2 seconds:
!
Success rate is 100 percent (1/1), round-trip min/avg/max = 34/34/34 ms
host-6#ping ip 2.1.3.1 repeat 1
Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 2.1.3.1, timeout is 2 seconds:
!
Success rate is 100 percent (1/1), round-trip min/avg/max = 38/38/38 ms
```