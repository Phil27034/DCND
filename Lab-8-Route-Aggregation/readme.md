# Оптимизация таблиц маршрутизации

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
- На спайне используется опция allow as in так как к нему приходят маршруты с файерволла с его собственной AS в пути
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
 
