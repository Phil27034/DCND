# Построение сети Underlay на базе протокола IS-IS

## Схема адресации NET параметра

49.0001.000X.0000.000Y.00
- 0001 - area 1 - у всех одинаково
- X - 1 для спайна, 2 для Leaf
- Y - в порядке очереди

Конфиг для Leaf-1
```
feature isis

router isis isis
  net 49.0001.0002.0000.0001.00
  is-type level-2

interface loopback0
  ip router isis isis

interface Ethernet1/1
  ip router isis isis

interface Ethernet1/2
  ip router isis isis
```
Конфиги для других устройств аналогичны.

Все loopback доступны с spine-1
```
spine-1# show ip route
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.1.1.0/30, ubest/mbest: 1/0, attached
    *via 10.1.1.1, Eth1/1, [0/0], 06:00:21, direct
10.1.1.1/32, ubest/mbest: 1/0, attached
    *via 10.1.1.1, Eth1/1, [0/0], 06:00:21, local
10.1.1.4/30, ubest/mbest: 1/0, attached
    *via 10.1.1.5, Eth1/2, [0/0], 06:00:21, direct
10.1.1.5/32, ubest/mbest: 1/0, attached
    *via 10.1.1.5, Eth1/2, [0/0], 06:00:21, local
10.1.1.8/30, ubest/mbest: 1/0, attached
    *via 10.1.1.9, Eth1/3, [0/0], 06:00:21, direct
10.1.1.9/32, ubest/mbest: 1/0, attached
    *via 10.1.1.9, Eth1/3, [0/0], 06:00:21, local
10.1.2.0/30, ubest/mbest: 1/0
    *via 10.1.1.2, Eth1/1, [115/80], 03:06:11, isis-isis, L2
10.1.2.4/30, ubest/mbest: 1/0
    *via 10.1.1.6, Eth1/2, [115/80], 04:47:53, isis-isis, L2
10.1.2.8/30, ubest/mbest: 1/0
    *via 10.1.1.10, Eth1/3, [115/80], 05:05:09, isis-isis, L2
10.10.1.1/32, ubest/mbest: 2/0, attached
    *via 10.10.1.1, Lo0, [0/0], 06:01:12, local
    *via 10.10.1.1, Lo0, [0/0], 06:01:12, direct
10.10.1.2/32, ubest/mbest: 3/0
    *via 10.1.1.2, Eth1/1, [115/81], 00:01:54, isis-isis, L2
    *via 10.1.1.6, Eth1/2, [115/81], 00:01:54, isis-isis, L2
    *via 10.1.1.10, Eth1/3, [115/81], 00:01:54, isis-isis, L2
10.10.1.3/32, ubest/mbest: 1/0
    *via 10.1.1.2, Eth1/1, [115/41], 03:06:11, isis-isis, L2
10.10.1.4/32, ubest/mbest: 1/0
    *via 10.1.1.6, Eth1/2, [115/41], 04:47:53, isis-isis, L2
10.10.1.5/32, ubest/mbest: 1/0
    *via 10.1.1.10, Eth1/3, [115/41], 05:05:09, isis-isis, L2
```
