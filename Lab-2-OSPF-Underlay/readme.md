# Настройка OSPF underlay

Протокол OSPF и Loopback интерфейсы настроены настроены на всех устройствах.  
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
Все Loopback интерфейсы доступны со Spine-1:
```
spine-1# show ip route ospf-underlay | grep /32
10.10.1.2/32, ubest/mbest: 3/0
10.10.1.3/32, ubest/mbest: 1/0
10.10.1.4/32, ubest/mbest: 1/0
10.10.1.5/32, ubest/mbest: 1/0
```
