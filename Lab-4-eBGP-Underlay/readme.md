# Пострение Underlay сети на базе eBGP
 - Router-Id - настроен вручную, такой же как Underlay Loopback
 - Таймеры - Hello и Hold - 3 и 9 секунд соотвественно
 - Reconnect интервал - 12 секунд
 - address-family ipv4 unicast
 - Распространяются только маршруты до Loopback на лифах

## Схема сети
<img width="1142" alt="image" src="https://user-images.githubusercontent.com/116812447/207006848-f7a1c966-b143-4f34-abc3-816545c7e882.png">

## Нумерация автономных систем
- Нумерация берется из Private Range
- Спайны - 64600
- Лифы - 64701-64703

 ## Конфиг Спайна-1
 ```
feature bgp

router bgp 64600
  router-id 10.10.1.1
  bestpath as-path multipath-relax
  reconnect-interval 12
  address-family ipv4 unicast
    maximum-paths 64
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
 ```
 
 ## Конфиг Лифа-1
 ```
feature bgp

router bgp 64701
  router-id 10.10.1.3
  bestpath as-path multipath-relax
  reconnect-interval 12
  address-family ipv4 unicast
    redistribute direct route-map connected
    maximum-paths 64
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
 ```
 
 ## Таблица маршрутизации Спайна-1
 Отображены маршруты до всех Loopback на всех Лифах
 ```
 spine-1(config)# show ip route bgp
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.10.1.3/32, ubest/mbest: 1/0
    *via 10.1.1.2, [20/0], 00:31:01, bgp-64600, external, tag 64701
10.10.1.4/32, ubest/mbest: 1/0
    *via 10.1.1.6, [20/0], 00:24:55, bgp-64600, external, tag 64702
10.10.1.5/32, ubest/mbest: 1/0
    *via 10.1.1.10, [20/0], 00:02:48, bgp-64600, external, tag 64703
 ```
 
 
