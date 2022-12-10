# Пострение Underlay сети на базе eBGP
 - Router-Id - настроен вручную, такой же как Underlay Loopback
 - Таймеры - Hello и Hold - 3 и 9 секунд соотвественно
 - Reconnect таймер - 12 секунд
 - address-family ipv4 unicast
 - Распространяются только маршруты до Loopback на лифах

## Схема сети

## Нумерация автономных систем
- Нумерация берется из Private Range
- Спайны - 64600
- Лифы - 64701-64703

 ## Конфиг Спайна-1
 ```
 ```
 
 ## Конфиг Лифа-1
 ```
feature bgp

router bgp 64701
  router-id 10.10.1.3
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    redistribute direct route-map connected
    maximum-paths 64
  neighbor 10.1.1.1
    remote-as 64600
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.1.2.1
    remote-as 64600
    timers 3 9
    address-family ipv4 unicast
 ```
 
 ## Таблица маршрутизации Спайна-1
 ```
 ```
 
 
