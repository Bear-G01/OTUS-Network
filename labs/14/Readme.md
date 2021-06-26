## OSPF

#### Цель:

##### Настроить OSPF офисе Москва Разделить сеть на зоны Настроить фильтрацию между зонами

- Маршрутизаторы R14-R15 находятся в зоне 0 - backbone
- Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по-умолчанию
- Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию
- Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101
- Настройка для IPv6 повторяет логику IPv4



#### Топология

![topology11](topology11.jpg)



#### Настройка OSPF для адресного пространства IPv4

Настроим маршрутизаторы согласно зонам

##### Настройка R14 (area 0 и area 101)

Включим процесс ospf

```
router ospf 1
```

Зададим router-id по логике: первый октет - номер зоны, последний октет - номер устройства, два средних октета - 0

```
router-id 0.0.0.14
```

Настроим все интерфейсы пассивными и включим необходимые

```
passive-interface default
no passive-interface Ethernet0/0
no passive-interface Ethernet0/1
no passive-interface Ethernet0/3
```

 Укажем, что маршрутизатору R14 известен маршрут по-умолчанию

```
default-information originate
```

Укажем, что area 101  это totally-stub area

```
area 101 stub no-summary
```

Настроим интерфейсы маршрутизатора

```
interface Ethernet0/0
 ip ospf 1 area 0

interface Ethernet0/1
 ip ospf 1 area 0

interface Ethernet0/3
 ip ospf network point-to-point
 ip ospf 1 area 101
```

Таблица соседства

```
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.13         1   FULL/BDR        00:00:37    172.16.32.2     Ethernet0/1
10.0.0.12         1   FULL/BDR        00:00:38    172.16.32.6     Ethernet0/0
101.0.0.19        0   FULL/  -        00:00:36    172.16.32.18    Ethernet0/3
```



##### Настройка R12 (area 0 и area 10)

Включим процесс ospf

```
router ospf 1
```

Зададим router-id

```
router-id 0.0.0.12
```

Настроим все интерфейсы пассивными и включим необходимые

```
passive-interface default
no passive-interface Ethernet0/0
no passive-interface Ethernet0/1
no passive-interface Ethernet0/2
no passive-interface Ethernet0/3
```

Настроим интерфейсы маршрутизатора

```
interface Ethernet0/0
 ip ospf 1 area 10

interface Ethernet0/1
 ip ospf 1 area 10

interface Ethernet0/2
 ip ospf 1 area 0

interface Ethernet0/3
 ip ospf 1 area 0
```

Таблица соседства

```
Neighbor ID     Pri   State           Dead Time   Address         Interface
0.0.0.15          1   FULL/BDR        00:00:35    172.16.32.9     Ethernet0/3
0.0.0.14          1   FULL/DR         00:00:37    172.16.32.5     Ethernet0/2
10.0.0.105        1   FULL/DR         00:00:31    172.16.32.30    Ethernet0/1
10.0.0.104        1   FULL/DR         00:00:31    172.16.32.26    Ethernet0/0
```

Настроим остальные маршрутизаторы аналогично R14 и R12

##### Настроим маршрутизатор R19 в totally-stub area, чтобы он получал только маршрут по-умолчанию

```markdown
router ospf 1
 area 101 stub
```

Предварительно настроив area 101 на R14 как totally stub

```
router ospf 1
 area 101 stub no-summary
```

```
Gateway of last resort is 172.16.32.17 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 172.16.32.17, 00:00:35, Ethernet0/0
      172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        172.16.32.16/30 is directly connected, Ethernet0/0
L        172.16.32.18/32 is directly connected, Ethernet0/0
```



##### Настроим фильтрацию подсети 172.16.32.16/30 из area 101 в area 102 на маршрутизаторе R15

Настроим prefix-list: исключаем 172.16.32.16/30 и разрешаем все остальные

```
ip prefix-list EX-AREA101 seq 5 deny 172.16.32.16/30
ip prefix-list EX-AREA101 seq 10 permit 0.0.0.0/0 le 32
```

Применяем ограничение для area 102 во входящем направлении

```
router ospf 1
 area 102 filter-list prefix EX-AREA101 in
```

Подсеть 172.16.32.16/30 из area 101 отфильтрована на R20

```
Gateway of last resort is 172.16.32.21 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 172.16.32.21, 03:44:09, Ethernet0/0
      172.16.0.0/16 is variably subnetted, 13 subnets, 3 masks
O IA     172.16.0.0/23 [110/31] via 172.16.32.21, 03:44:09, Ethernet0/0
O IA     172.16.8.0/23 [110/31] via 172.16.32.21, 03:44:09, Ethernet0/0
O IA     172.16.32.0/30 [110/30] via 172.16.32.21, 03:44:09, Ethernet0/0
O IA     172.16.32.4/30 [110/30] via 172.16.32.21, 03:44:09, Ethernet0/0
O IA     172.16.32.8/30 [110/20] via 172.16.32.21, 03:44:09, Ethernet0/0
O IA     172.16.32.12/30 [110/20] via 172.16.32.21, 03:44:09, Ethernet0/0
C        172.16.32.20/30 is directly connected, Ethernet0/0
L        172.16.32.22/32 is directly connected, Ethernet0/0
O IA     172.16.32.24/30 [110/30] via 172.16.32.21, 03:44:09, Ethernet0/0
O IA     172.16.32.28/30 [110/30] via 172.16.32.21, 03:44:09, Ethernet0/0
O IA     172.16.32.32/30 [110/30] via 172.16.32.21, 03:44:09, Ethernet0/0
O IA     172.16.32.36/30 [110/30] via 172.16.32.21, 03:44:09, Ethernet0/0
O E2     172.16.255.4/32 [110/20] via 172.16.32.21, 03:44:09, Ethernet0/0
```



#### Настройка OSPF для адресного пространства IPv6

Настроим OSPFv3 аналогично OSPFv2

На R14:

```
router ospfv3 1
 router-id 0.0.0.14
 area 101 stub no-summary
  passive-interface default
  no passive-interface Ethernet0/0
  no passive-interface Ethernet0/1
  no passive-interface Ethernet0/3
```

Настройка OSPFv3 на интерфейсах

```
interface Ethernet0/0
 ipv6 enable
 ipv6 ospf 1 area 0

interface Ethernet0/1
 ipv6 enable
 ipv6 ospf 1 area 0

interface Ethernet0/3
 ipv6 enable
 ipv6 ospf 1 area 101
 ipv6 ospf network point-to-point
```

Таблица соседства

```
 OSPFv3 Router with ID (0.0.0.14) (Process ID 1)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
10.0.0.13         1   FULL/BDR        00:00:36    6               Ethernet0/1
10.0.0.12         1   FULL/BDR        00:00:38    5               Ethernet0/0
172.16.32.18      0   FULL/  -        00:00:39    3               Ethernet0/3
```

На R12:

```
router ospfv3 1
 router-id 10.0.0.12
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
 no passive-interface Ethernet0/3
```

Настройка OSPFv3 на интерфейсах

```
interface Ethernet0/0
 ipv6 enable
 ipv6 ospf 1 area 10

interface Ethernet0/1
 ipv6 enable
 ipv6 ospf 1 area 10

interface Ethernet0/2
 ipv6 enable
 ipv6 ospf 1 area 0

interface Ethernet0/3
 ipv6 enable
 ipv6 ospf 1 area 0
```

Таблица соседства

```
OSPFv3 Router with ID (10.0.0.12) (Process ID 1)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
0.0.0.15          1   FULL/DR         00:00:33    4               Ethernet0/3
0.0.0.14          1   FULL/DR         00:00:36    3               Ethernet0/2
10.0.0.4          1   FULL/DR         00:00:34    13              Ethernet0/0
10.0.0.5          1   FULL/DR         00:00:35    14              Ethernet0/1
```

Укажем, что маршрутизаторам R14 и R15 известен маршрут по-умолчанию

```
(config)#ipv6 router ospf 1
(config-rtr)#default-information originate
```

Таблица маршрутизации на маршрутизаторе R12

```
OE2 ::/0 [110/1], tag 1
     via FE80::14, Ethernet0/2
     via FE80::15, Ethernet0/3
O   2001:1:16:1::/64 [110/20]
     via FE80::14, Ethernet0/2
C   2001:1:16:2::/64 [0/0]
     via Ethernet0/2, directly connected
L   2001:1:16:2::12/128 [0/0]
     via Ethernet0/2, receive
C   2001:1:16:3::/64 [0/0]
     via Ethernet0/3, directly connected
L   2001:1:16:3::12/128 [0/0]
     via Ethernet0/3, receive
O   2001:1:16:4::/64 [110/20]
     via FE80::15, Ethernet0/3
OI  2001:1:16:5::/64 [110/20]
     via FE80::14, Ethernet0/2
OI  2001:1:16:6::/64 [110/20]
     via FE80::15, Ethernet0/3
C   2001:1:16:7::/64 [0/0]
     via Ethernet0/0, directly connected
L   2001:1:16:7::12/128 [0/0]
     via Ethernet0/0, receive
C   2001:1:16:8::/64 [0/0]
     via Ethernet0/1, directly connected
L   2001:1:16:8::12/128 [0/0]
     via Ethernet0/1, receive
O   2001:1:16:9::/64 [110/20]
     via FE80::A8BB:CCFF:FE00:4001, Ethernet0/0
O   2001:1:16:10::/64 [110/20]
     via FE80::A8BB:CCFF:FE00:5011, Ethernet0/1
L   FF00::/8 [0/0]
     via Null0, receive
```

Area 101 настроена как totally-stub. Таблица маршрутизации на R19

```
OI  ::/0 [110/11]
     via FE80::14, Ethernet0/0
C   2001:1:16:5::/64 [0/0]
     via Ethernet0/0, directly connected
L   2001:1:16:5::19/128 [0/0]
     via Ethernet0/0, receive
L   FF00::/8 [0/0]
     via Null0, receive
```

##### Настроим фильтрацию подсети 172.16.32.16/30 из area 101 в area 102 для IPv6 на маршрутизаторе R15

```
(config)#ipv6 prefix-list EX-AREA101-V6 seq 5 deny 2001:1:16:5::/64
(config)#ipv6 prefix-list EX-AREA101-V6 seq 10 permit ::/0 le 128
....
(config-rtr)#area 102 filter-list prefix EX-AREA101 in
```

Таблица маршрутизации на маршрутизаторе R20. Префикс 2001:1:16:5::/64 исключён.

```
OE2 ::/0 [110/1], tag 1
     via FE80::15, Ethernet0/0
OI  2001:1:16:1::/64 [110/30]
     via FE80::15, Ethernet0/0
OI  2001:1:16:2::/64 [110/30]
     via FE80::15, Ethernet0/0
OI  2001:1:16:3::/64 [110/20]
     via FE80::15, Ethernet0/0
OI  2001:1:16:4::/64 [110/20]
     via FE80::15, Ethernet0/0
C   2001:1:16:6::/64 [0/0]
     via Ethernet0/0, directly connected
L   2001:1:16:6::20/128 [0/0]
     via Ethernet0/0, receive
OI  2001:1:16:7::/64 [110/30]
     via FE80::15, Ethernet0/0
OI  2001:1:16:8::/64 [110/30]
     via FE80::15, Ethernet0/0
OI  2001:1:16:9::/64 [110/30]
     via FE80::15, Ethernet0/0
OI  2001:1:16:10::/64 [110/30]
     via FE80::15, Ethernet0/0
L   FF00::/8 [0/0]
```

