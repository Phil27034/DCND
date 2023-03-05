# Оптимизация таблиц маршрутизации
Конфиги всех устройств представлены в папке

## Схема

<img width="2404" alt="image" src="https://user-images.githubusercontent.com/116812447/222961373-bc19c581-b5de-4029-bd9f-c42b84eef3bc.png">

## Описание сети
2 VRF:
- OTUS
  - 192.168.1.0/24
  - 192.168.2.0/24
- OTUS2
  - 192.168.3.0/24
  - 192.168.4.0/24

## Особенности маршрутизации
- L2, L3 связность внутри VRF осуществляется средствами фабрики
- Связь между VRF осуществляется средством внешнего Firewall
- За Firewall находятся 3 внешних сети:
  - 1.1.1.0/24
  - 1.1.2.0/24
  - 1.1.3.0/24
- Внешние сети транслируются в фабрику
- Сети фабрики транслуируются во вне
- Для пиринга с файерворлом border-leaf использует подмену номера AS
- На спайне и всех лифах кроме бордера используется опция allow as in так как к ним приходят маршруты с файерволла с его собственной AS в пути
- Border-leaf отправляет в сторону файерволла суммарные сетевые маршруты вместо кучи мрашрутов до каждого хоста
- Файерволл в свою очередь топравляет в фабрику один суммаризованный маршрут, вместо того чтобы отправлять каждую сеть отдельно
 
 ## Настройки
 Border-Leaf - суммаризация маршрутов в сторону файерволла
 ```
 vrf OTUS
    address-family ipv4 unicast
      aggregate-address 192.168.1.0/24 as-set summary-only
      aggregate-address 192.168.2.0/24 as-set summary-only
    neighbor 192.168.5.2
      remote-as 64800
      local-as 64705 no-prepend replace-as
      address-family ipv4 unicast
    neighbor 192.168.6.2
  vrf OTUS2
    address-family ipv4 unicast
      aggregate-address 192.168.3.0/24 as-set summary-only
      aggregate-address 192.168.4.0/24 as-set summary-only
    neighbor 192.168.6.2
      remote-as 64800
      local-as 64704 no-prepend replace-as
      address-family ipv4 unicast
 ```
 
 Файерволл - суммаризация маршрутов в сторону border-leaf
 ```
 router bgp 64800
 bgp log-neighbor-changes
 network 1.1.0.0 mask 255.255.0.0
 network 1.1.1.0 mask 255.255.255.0
 network 1.1.2.0 mask 255.255.255.0
 network 1.1.3.0 mask 255.255.255.0
 aggregate-address 1.1.0.0 255.255.0.0 as-set summary-only
 neighbor 192.168.5.1 remote-as 64705
 neighbor 192.168.6.1 remote-as 64704
 ```
 
 ## Проверка связи
 Ping с Host-1 до всех хостов в фабрике, а также до сетей за файерволлом
 ```
 switch# ping 192.168.2.2
PING 192.168.2.2 (192.168.2.2): 56 data bytes
64 bytes from 192.168.2.2: icmp_seq=0 ttl=253 time=11.258 ms
64 bytes from 192.168.2.2: icmp_seq=1 ttl=253 time=8.812 ms
64 bytes from 192.168.2.2: icmp_seq=2 ttl=253 time=10.627 ms
64 bytes from 192.168.2.2: icmp_seq=3 ttl=253 time=11.411 ms
64 bytes from 192.168.2.2: icmp_seq=4 ttl=253 time=11.356 ms

--- 192.168.2.2 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 8.812/10.692/11.411 ms
switch# ping 192.168.3.2
PING 192.168.3.2 (192.168.3.2): 56 data bytes
64 bytes from 192.168.3.2: icmp_seq=0 ttl=249 time=52.523 ms
64 bytes from 192.168.3.2: icmp_seq=1 ttl=249 time=40.784 ms
64 bytes from 192.168.3.2: icmp_seq=2 ttl=249 time=68.24 ms
64 bytes from 192.168.3.2: icmp_seq=3 ttl=249 time=61.553 ms
64 bytes from 192.168.3.2: icmp_seq=4 ttl=249 time=49.282 ms

--- 192.168.3.2 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 40.784/54.476/68.24 ms
switch# ping 192.168.1.3
PING 192.168.1.3 (192.168.1.3): 56 data bytes
64 bytes from 192.168.1.3: icmp_seq=0 ttl=254 time=22.234 ms
64 bytes from 192.168.1.3: icmp_seq=1 ttl=254 time=20.268 ms
64 bytes from 192.168.1.3: icmp_seq=2 ttl=254 time=16.242 ms
64 bytes from 192.168.1.3: icmp_seq=3 ttl=254 time=17.697 ms
64 bytes from 192.168.1.3: icmp_seq=4 ttl=254 time=15.479 ms

--- 192.168.1.3 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 15.479/18.384/22.234 ms
switch# ping 192.168.2.3
PING 192.168.2.3 (192.168.2.3): 56 data bytes
64 bytes from 192.168.2.3: icmp_seq=0 ttl=252 time=32.007 ms
64 bytes from 192.168.2.3: icmp_seq=1 ttl=252 time=19.658 ms
64 bytes from 192.168.2.3: icmp_seq=2 ttl=252 time=30.63 ms
64 bytes from 192.168.2.3: icmp_seq=3 ttl=252 time=24.237 ms
64 bytes from 192.168.2.3: icmp_seq=4 ttl=252 time=35.347 ms

--- 192.168.2.3 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 19.658/28.375/35.347 ms
switch# ping 192.168.4.2
PING 192.168.4.2 (192.168.4.2): 56 data bytes
64 bytes from 192.168.4.2: icmp_seq=0 ttl=249 time=43.323 ms
64 bytes from 192.168.4.2: icmp_seq=1 ttl=249 time=54.935 ms
64 bytes from 192.168.4.2: icmp_seq=2 ttl=249 time=61.39 ms
64 bytes from 192.168.4.2: icmp_seq=3 ttl=249 time=42.461 ms
64 bytes from 192.168.4.2: icmp_seq=4 ttl=249 time=44.996 ms

--- 192.168.4.2 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 42.461/49.421/61.39 ms
switch# ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1): 56 data bytes
64 bytes from 1.1.1.1: icmp_seq=0 ttl=252 time=37.793 ms
64 bytes from 1.1.1.1: icmp_seq=1 ttl=252 time=29.305 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=252 time=25.276 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=252 time=29.385 ms
64 bytes from 1.1.1.1: icmp_seq=4 ttl=252 time=30.336 ms

--- 1.1.1.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 25.276/30.418/37.793 ms
switch# ping 1.1.2.1
PING 1.1.2.1 (1.1.2.1): 56 data bytes
64 bytes from 1.1.2.1: icmp_seq=0 ttl=252 time=34.04 ms
64 bytes from 1.1.2.1: icmp_seq=1 ttl=252 time=32.442 ms
64 bytes from 1.1.2.1: icmp_seq=2 ttl=252 time=47.714 ms
64 bytes from 1.1.2.1: icmp_seq=3 ttl=252 time=38.645 ms
64 bytes from 1.1.2.1: icmp_seq=4 ttl=252 time=29.605 ms

--- 1.1.2.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 29.605/36.489/47.714 ms
switch# ping 1.1.3.1
PING 1.1.3.1 (1.1.3.1): 56 data bytes
64 bytes from 1.1.3.1: icmp_seq=0 ttl=252 time=17.586 ms
64 bytes from 1.1.3.1: icmp_seq=1 ttl=252 time=23.792 ms
64 bytes from 1.1.3.1: icmp_seq=2 ttl=252 time=24.743 ms
64 bytes from 1.1.3.1: icmp_seq=3 ttl=252 time=20.615 ms
64 bytes from 1.1.3.1: icmp_seq=4 ttl=252 time=27.097 ms

--- 1.1.3.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 17.586/22.766/27.097 ms
```

## Суммаризация маршрутов
Таблица маршрутизации на Leaf-1 vrf OTUS  
Видны суммарные маршруты для внешних сетей, а также маршруты до сетей в другом vrf
```
leaf-1(config)# show ip bgp vrf OTUS
BGP routing table information for VRF OTUS, address family IPv4 Unicast
BGP table version is 19, Local Router ID is 192.168.1.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>e1.1.0.0/16         10.10.1.6                                      0 64600 64703 64800 i
*>e192.168.1.3/32     10.10.1.5                                      0 64600 64702 i
*>e192.168.2.3/32     10.10.1.5                                      0 64600 64702 i
*>e192.168.3.0/24     10.10.1.6                                      0 64600 64703 64800 64704 64600 64701 i
*>e192.168.4.0/24     10.10.1.6                                      0 64600 64703 64800 64704 64600 64702 i
```

Таблица маршрутизации на файерволле  
Видны маршруты до сетей, вместо хостовых маршрутов
```
Router#show ip bgp
BGP table version is 92, local router ID is 192.168.6.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  1.1.0.0/16       0.0.0.0                       100  32768 i
 s>  1.1.1.0/24       0.0.0.0                  0         32768 i
 s>  1.1.2.0/24       0.0.0.0                  0         32768 i
 s>  1.1.3.0/24       0.0.0.0                  0         32768 i
 *>  192.168.1.0      192.168.5.1                            0 64705 {64600,64701,64702} i
 *>  192.168.2.0      192.168.5.1                            0 64705 {64600,64701,64702} i
 *>  192.168.3.0      192.168.6.1                            0 64704 64600 64701 i
 *>  192.168.4.0      192.168.6.1                            0 64704 64600 64702 i
```
