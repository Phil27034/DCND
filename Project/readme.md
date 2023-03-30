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
- Anycast gateway
- Virtual MAC
- Суммаризация маршрутов на Border Leaf перед отправкой на внешние устройства

### Конфиг Leaf

### Конфиг Spine

### Конфиг Border Leaf
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

## Связь с внешними сетями: Firewall, Campus

## Связь с Интернетом