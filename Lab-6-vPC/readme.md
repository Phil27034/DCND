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
  
  delay restore 10
  
  #Включение Peer gateway - резервный узел может маршрутизировать пакеты предназначенные основному узлу
  peer-gateway
  
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
