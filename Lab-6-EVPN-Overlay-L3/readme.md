# Настройка VxLAN EVPN L3

## Задание
Настроить маршрутизацию в рамках Overlay между клиентами: assymetric и symmetric подходами

## Метод решения
- Настриваем Underlay на eBGP
- Настраиваем Overlay Control PLane на MP-BGP eVPN c использованием loopback на каждом свитче
- Настраиваем VLAN интерфейсы, добавляем в них физические интерфейсы, создаем SVI интерфейсы
- Создаем MAC VRF, настраиваем RD, RT
- Создаем IP VRF, добавляем туда SVI интерфейсы, настраиваем RD, RT и импорт, экспорт в таблицу evpn
- Настраиваем nve интерфейc, виртуальный MAC адрес и anycast gateway

## Схема стенда

<img width="1612" alt="image" src="https://user-images.githubusercontent.com/116812447/216768217-c13f5d80-8997-4c34-ab15-6a17c4cdb50b.png">

## План IP адресации

 Адресное пространство для Underlay сети распределено следующим образом:   
 10.X.Y.Z/30  
 X - номер ЦОДа   
 Y - номер Spine   
 Z - номер в порядке очереди, первый номер всегда принадлежит Spine
 
| Spine  | Leaf | Network |
| ------------- | ------------- | ----------- |
| Spine 1 - 10.1.1.1  | Leaf 1 - 10.1.1.2  | 10.1.1.0/30 |
| Spine 1 - 10.1.1.5 | Leaf 2 - 10.1.1.6  | 10.1.1.4/30 |
| Spine 1 - 10.1.1.9 | Leaf 3 - 10.1.1.10  | 10.1.1.8/30 |
| Spine 2 - 10.1.2.1  | Leaf 1 - 10.1.2.2  | 10.1.2.0/30 |
| Spine 2 - 10.1.2.5 | Leaf 2 - 10.1.2.6  | 10.1.2.4/30 |
| Spine 2 - 10.1.2.9 | Leaf 3 - 10.1.2.10  | 10.1.2.8/30 |

Резервные сети:  
10.1.2.12/30  
10.1.2.16/30  
10.1.2.20/30  

Адреса для loopback интерфейсов:  
10.10.X.Y  
X: 1 - Underlay, 2 - Overlay  
Y: В порядке очереди  
| Device | Underlay Loopback | Overlay loopback |  
| ------------- | ------------- | ----------- |
| Spine 1  | 10.10.1.1/32  | 10.10.2.1/32 |
| Spine 2 | 10.10.1.2/32  | 10.10.2.2/32 |
| Leaf 1| 10.10.1.3/32  | 10.10.2.3/32 |
| Leaf 2| 10.10.1.4/32  | 10.10.2.4/32 |
| Leaf 3| 10.10.1.5/32  | 10.10.2.5/32 |

Резервные адреса:  
10.10.1.6/32 - 10.10.1.10/32  
10.10.2.6/32 - 10.10.2.10/32

## План адресации AS
- Нумерация берется из Private Range
- Спайны - 64600
- Лифы - 64701-64703

## Настройки Leaf-1
```
#Виртуальный MAC
fabric forwarding anycast-gateway-mac 0001.0002.0003

#Сопоставление VLAN и MAC
vlan 1,100,200
vlan 100
  vn-segment 100
vlan 200
  vn-segment 200
  
#Настройка IP VRF  
vrf context OTUS
  rd 10.10.2.3:1000
  address-family ipv4 unicast
    route-target import 1000:1000 evpn
    route-target export 1000:1000 evpn
 
#Настройка VLAN интерфейсов и  nve интерфейса 
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

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback0
  member vni 100
    ingress-replication protocol bgp
  member vni 200
    ingress-replication protocol bgp

#Настройка физических интерфейсов и loopback
interface Ethernet1/1
  no switchport
  ip address 10.1.1.2/30
  no shutdown

interface Ethernet1/2
  switchport access vlan 100

interface Ethernet1/3
  switchport access vlan 200
  
interface loopback0
  ip address 10.10.2.3/32
  

#Настройка BGP
router bgp 64701
  router-id 10.10.2.3
  address-family ipv4 unicast
    network 10.10.2.3/32
  neighbor 10.1.1.1
    remote-as 64600
    address-family ipv4 unicast
  neighbor 10.10.2.1
    remote-as 64600
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended

#Настройка MAC VRF 
evpn
  vni 100 l2
    rd 10.10.2.3:100
    route-target import 100:100
    route-target export 100:100
  vni 200 l2
    rd 10.10.2.3:200
    route-target import 200:200
    route-target export 200:200
 ```
