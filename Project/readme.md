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
- Firewall траснлирует в фабрику адреса удаленных сетей

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
!
interface Ethernet0/0.3
 encapsulation dot1Q 3
 ip address 192.168.6.2 255.255.255.252

 router bgp 64800
 bgp log-neighbor-changes
 network 1.1.1.0 mask 255.255.255.0
 network 1.1.2.0 mask 255.255.255.0
 network 1.1.3.0 mask 255.255.255.0
 neighbor 192.168.5.1 remote-as 64705
 neighbor 192.168.6.1 remote-as 64704
```

## Связь с Интернетом
- AS 64800 - 159.33.0.0/24
- Тестовый адрес для подключения из Интернет - 159.33.0.100
- Адрес натируется в 192.168.1.2, который находится в фабрике в VRF OTUS - Frontend
- Все обращения фабрики в интернет проходят через процедуру SNAT для соотвествующего провайдера
- Firewall отправляет в фабрику шлюз по умолчанию
