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

