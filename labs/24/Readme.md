## iBGP

#### Цель:

Настроить iBGP в офисе Москва

Настроить iBGP в сети провайдера Триада

Организовать полную IP связанность всех сетей

#### Задание:

1. Настроить iBGP в офисе Москва между маршрутизаторами R14 и R15
2. Настроить iBGP в провайдере Триада
3. Настроить офиса Москва так, чтобы приоритетным провайдером стал Ламас
4. Настроить офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно
5. Все сети в лабораторной работе должны иметь IP связность

#### Топология:



#### Решение:

##### Настроить iBGP в офисе Москва между маршрутизаторами R14 и R15

Настроим iBGP на loopback-интерфейсах маршрутизаторов R14 и R15

```
R14(config)# interface lo0
R14(config-if)# ip add 14.14.14.14
R14(config-if)# ip ospf 1 area 0
R14(config-if)# ipv6 address FE80::14 link-local
R14(config-if)# ipv6 address 14:14:14:14::14/128
R14(config-if)# ipv6 ospf 1 area 0
.........
R14(config)# router bgp 1001
R14(config-router)# neighbor 15:15:15:15::15 remote-as 1001
R14(config-router)# neighbor 15:15:15:15::15 update-source Loopback0
R14(config-router)# neighbor 15.15.15.15 remote-as 1001
R14(config-router)# neighbor 15.15.15.15 update-source lo0
R14(config-router)# address-family ipv4
R14(config-router-af)# neighbor 15.15.15.15 activate
R14(config-router)# address-family ipv6
R14(config-router-af)# neighbor 15:15:15:15::15 activate

```

```
R15(config)# interface lo0
R15(config-if)# ip add 15.15.15.15
R15(config-if)# ip ospf 1 area 0
R15(config-if)# ipv6 address FE80::14 link-local
R15(config-if)# ipv6 address 15:15:15:15::15/128
R15(config-if)# ipv6 ospf 1 area 0
.........
R15(config)# router bgp 1001
R15(config-router)# neighbor 14:14:14:14::14 remote-as 1001
R15(config-router)# neighbor 14:14:14:14::14 update-source Loopback0
R15(config-router)# neighbor 14.14.14.14 remote-as 1001
R15(config-router)# neighbor 14.14.14.14 update-source lo0
R15(config-router)# address-family ipv4
R15(config-router-af)# neighbor 14.14.14.14 activate
R15(config-router)# address-family ipv6
R15(config-router-af)# neighbor 14:14:14:14::14 activate
```

Проверка установления соседства:

```
R14#sh ip bgp summary

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
15.15.15.15     4         1001      29      32       23    0    0 00:19:55       11
```

```
R15#sh ip bgp summary
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
14.14.14.14     4         1001      34      31       13    0    0 00:21:04        0
```



##### Настроить iBGP в провайдере Триада

Настроим iBGP в Триаде поверх IS-IS на loopback-интерфейсах.

Настроим Route Reflector на маршрутизаторе R25

```
R25(config)# interface Loopback0
R25(config-if)# ip address 25.25.25.25 255.255.255.255
R25(config-if)# ip router isis
R25(config-if)# ipv6 address FE80::25 link-local
R25(config-if)# ipv6 address 2001:520::25/128
R25(config-if)# ipv6 router isis

....

R25(config)# router bgp 520
R25(config-router)# neighbor RRC peer-group
R25(config-router)# neighbor RRC remote-as 520
R25(config-router)# neighbor RRC update-source Loopback0
R25(config-router)# neighbor RRC6 peer-group
R25(config-router)# neighbor RRC6 remote-as 520
R25(config-router)# neighbor RRC6 update-source Loopback0
R25(config-router)# neighbor 23.23.23.23 peer-group RRC
R25(config-router)# neighbor 24.24.24.24 peer-group RRC
R25(config-router)# neighbor 26.26.26.26 peer-group RRC
R25(config-router)# neighbor 2001:520::23 peer-group RRC6
R25(config-router)# neighbor 2001:520::24 peer-group RRC6
R25(config-router)# neighbor 2001:520::26 peer-group RRC6

R25(config-router)# address-family ipv4
R25(config-router-af)#  neighbor RRC route-reflector-client
R25(config-router-af)#  neighbor RRC next-hop-self
R25(config-router-af)#  neighbor 23.23.23.23 activate
R25(config-router-af)#  neighbor 24.24.24.24 activate
R25(config-router-af)#  neighbor 26.26.26.26 activate

R25(config-router)# address-family ipv6
R25(config-router-af)#  neighbor RRC6 route-reflector-client
R25(config-router-af)#  neighbor 2001:520::23 activate
R25(config-router-af)#  neighbor 2001:520::24 activate
R25(config-router-af)#  neighbor 2001:520::26 activate
```

Настроим маршрутизатор R23

```
R23(config)# interface Loopback0
R23(config-if)# ip address 23.23.23.23 255.255.255.255
R23(config-if)# ip router isis
R23(config-if)# ipv6 address FE80::23 link-local
R23(config-if)# ipv6 address 2001:520::23/128
R23(config-if)# ipv6 router isis

....

R23(config)# router bgp 520
R23(config-router)# neighbor 25.25.25.25 remote-as 520
R23(config-router)# neighbor 25.25.25.25 update-source Loopback0
R23(config-router)# neighbor 2001:520::25 remote-as 520
R23(config-router)# neighbor 2001:520::25 update-source Loopback0

R23(config-router)# address-family ipv4
R23(config-router-af)#  neighbor 25.25.25.25 activate
R23(config-router-af)#  neighbor 25.25.25.25 next-hop-self

R23(config-router)# address-family ipv6
R23(config-router-af)#  neighbor 2001:520::25 activate
```

Настроим маршрутизаторы R24 и R26 аналогично маршрутизатору R23

Проверим состояние соседства и таблицу BGP на маршрутизаторе R25

```
R25#sh ip bgp summ
BGP router identifier 25.25.25.25, local AS number 520
BGP table version is 12, main routing table version 12
11 network entries using 1540 bytes of memory
12 path entries using 960 bytes of memory
4/4 BGP path/bestpath attribute entries using 576 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3124 total bytes of memory
BGP activity 20/0 prefixes, 28/6 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
23.23.23.23     4          520     278     284       12    0    0 04:09:53        4
24.24.24.24     4          520     283     283       12    0    0 04:09:52        5
26.26.26.26     4          520     282     283       12    0    0 04:10:03  
```



```
R25#sh ip bgp
BGP table version is 12, local router ID is 25.25.25.25
     Network          Next Hop            Metric LocPrf Weight Path
*>i 8.8.2.1/32       24.24.24.24              0    100      0 301 i
*>i 8.8.2.2/32       23.23.23.23              0    100      0 101 i
*>i 98.0.31.0/30     24.24.24.24              0    100      0 301 i
*>i 188.0.1.0/30     23.23.23.23              0    100      0 101 i
*>i 188.0.101.4/30   23.23.23.23              0    100      0 101 i
* i                  24.24.24.24              0    100      0 301 i
*>i 198.0.52.0/30    23.23.23.23              0    100      0 i
*>i 198.0.52.4/30    24.24.24.24              0    100      0 i
*>i 198.0.52.8/30    24.24.24.24              0    100      0 i
*>i 198.0.52.12/30   26.26.26.26              0    100      0 i
*>  198.0.52.16/30   0.0.0.0                  0         32768 i
*>  198.0.52.24/30   0.0.0.0                  0         32768 i
```

Проверим таблицу BGP на маршрутизаторе R26

```
R26#sh ip bgp
BGP table version is 13, local router ID is 26.26.26.26
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path

 *>i 8.8.2.1/32       24.24.24.24              0    100      0 301 i
 *>i 8.8.2.2/32       23.23.23.23              0    100      0 101 i
 *>i 98.0.31.0/30     24.24.24.24              0    100      0 301 i
 *>i 188.0.1.0/30     23.23.23.23              0    100      0 101 i
 *>i 188.0.101.4/30   23.23.23.23              0    100      0 101 i
 *>i 198.0.52.0/30    23.23.23.23              0    100      0 i
 *>i 198.0.52.4/30    24.24.24.24              0    100      0 i
 *>i 198.0.52.8/30    24.24.24.24              0    100      0 i
 *   198.0.52.14                               0             0 2042 i
 *   198.0.52.12/30   198.0.52.14              0             0 2042 i
 *>                   0.0.0.0                  0         32768 i
 *>i 198.0.52.16/30   25.25.25.25              0    100      0 i
 *>i 198.0.52.24/30   25.25.25.25              0    100      0 i
```



##### Настроить офиса Москва так, чтобы приоритетным провайдером стал Ламас

Настроим на маршрутизаторе R15 атрибут local preference равным 200

```
R15(config)# router bgp 1001
R15(config-router)# bgp default local-preference 200
```

Проверим атрибут с маршрутизатора R14 с помощью таблицы BGP
```
R14#sh ip bgp
     Network          Next Hop            Metric LocPrf Weight Path
 *>i 8.8.2.1/32       98.0.31.1                0    200      0 301 i
 *                    188.0.1.1                              0 101 301 i
 *>i 8.8.2.2/32       98.0.31.1                0    200      0 301 101 i
 *                    188.0.1.1                0             0 101 i
 *>i 98.0.31.0/30     15.15.15.15              0    200      0 i
 *                    188.0.1.1                              0 101 301 i
 r>i 188.0.1.0/30     98.0.31.1                0    200      0 301 101 i
 r                    188.0.1.1                0             0 101 i
 *>i 188.0.101.4/30   98.0.31.1                0    200      0 301 i
 *                    188.0.1.1                0             0 101 i
 *>i 198.0.52.0/30    98.0.31.1                0    200      0 301 101 i
 *                    188.0.1.1                0             0 101 i
 *>i 198.0.52.4/30    98.0.31.1                0    200      0 301 i
 *                    188.0.1.1                              0 101 301 i
 *>i 198.0.52.8/30    98.0.31.1                0    200      0 301 520 i
 *                    188.0.1.1                              0 101 520 i
 *>i 198.0.52.12/30   98.0.31.1                0    200      0 301 520 i
 *                    188.0.1.1                              0 101 520 i
 *>i 198.0.52.16/30   98.0.31.1                0    200      0 301 520 i
 *                    188.0.1.1                              0 101 520 i
 *>i 198.0.52.24/30   98.0.31.1                0    200      0 301 520 i
 *                    188.0.1.1                              0 101 520 i
```

   

##### Настроить офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно

Для настройки прохождения трафика по двум линкам одновременно необходимо на маршрутизаторе R18 настроить параметр maximum-paths равный 2.

```
R18(config)# router bgp 2042

R18(config-router)# address-family ipv4
R18(config-router-af)# maximum-paths 2

R18(config-router)# address-family ipv6
R18(config-router-af)# maximum-paths 2
```

Настроим loopback-интерфейсы на маршрутизаторах R22 и R21 провайдеров Киторн и Ламас 8.8.2.2 и 8.8.2.1

Проверим маршрутизацию до loopback-интерфейсов и до маршрутизатора R14 и R15 в Москве

Киторн:

```
R18#trace 8.8.2.2
Type escape sequence to abort.
Tracing the route to 8.8.2.2
VRF info: (vrf in name/id, vrf out name/id)
  1 198.0.52.9 1 msec
    198.0.52.13 0 msec
    198.0.52.9 1 msec
  2 10.0.52.25 1 msec
    10.0.52.1 0 msec
    10.0.52.25 0 msec
  3 198.0.52.2 [AS 520] 1 msec
    10.0.52.9 1 msec *
```

Ламас

```
R18#trace 8.8.2.1
Type escape sequence to abort.
Tracing the route to 8.8.2.1
VRF info: (vrf in name/id, vrf out name/id)
  1 198.0.52.13 1 msec
    198.0.52.9 0 msec
    198.0.52.13 1 msec
  2 198.0.52.6 [AS 520] 1 msec
    10.0.52.17 0 msec *
```

Москва R14

```
R18#trace 2001:1:101:1001::14
Type escape sequence to abort.
Tracing the route to 2001:1:101:1001::14

  1 2001:1:520:1043::26 0 msec
    2001:1:520:1042::24 1 msec
    2001:1:520:1043::26 0 msec
  2 2001:1:520:1::23 1 msec
    2001:1:520:4::25 0 msec
    2001:1:520:1::23 1 msec
  3 2001:1:520:2::23 1 msec
    2001:1:520:101::22 2 msec
    2001:1:520:2::23 1 msec
  4 2001:1:101:1001::14 [AS 101] 2 msec
    2001:1:520:101::22 1 msec
    2001:1:101:1001::14 1 msec
```

Москва R15

```
R18#trace 2001:1:301:1001::15
Type escape sequence to abort.
Tracing the route to 2001:1:301:1001::15

  1 2001:1:520:1043::26 0 msec
    2001:1:520:1042::24 0 msec
    2001:1:520:1043::26 1 msec
  2 2001:1:520:301::21 [AS 520] 20 msec
    2001:1:520:3::24 0 msec
    2001:1:520:301::21 0 msec
  3 2001:1:520:301::21 [AS 520] 1 msec
    2001:1:301:1001::15 1 msec
    2001:1:520:301::21 1 msec
```



##### Все сети в лабораторной работе должны иметь IP связность

Проверим связность всех сетей

Москва R15 - Санкт-Петербург R18

```
R15#ping 198.0.52.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 198.0.52.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R15#ping 198.0.52.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 198.0.52.14, timeout is 2 seconds:
!!!!!

R15#ping 2001:1:520:1042::18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1:520:1042::18, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R15#ping 2001:1:520:1043::18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1:520:1043::18, timeout is 2 seconds:
!!!!!
```

Санкт-Петербург R18 - Лабытнанги R27

```
R18#ping 198.0.52.26
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 198.0.52.26, timeout is 2 seconds:
!!!!!

R18>ping 2001:1:520:1061::27
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1:520:1061::27, timeout is 2 seconds:
!!!!!
```

Санкт-Петербург R18 - Чокурдах R28

```
R18#ping 198.0.52.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 198.0.52.18, timeout is 2 seconds:
!!!!!
R18#ping 198.0.52.22
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 198.0.52.22, timeout is 2 seconds:
!!!!!

R18#ping 2001:1:520:1051::28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1:520:1051::28, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R18#ping 2001:1:520:1052::28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1:520:1052::28, timeout is 2 seconds:
!!!!!
```

Москва R15 - Лабытнанги R27

```
R15#ping 198.0.52.26
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 198.0.52.26, timeout is 2 seconds:
!!!!!

R15#ping 2001:1:520:1061::27
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1:520:1061::27, timeout is 2 seconds:
!!!!!
```

Москва R15 - Чокурдах R28

```
R15#ping 198.0.52.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 198.0.52.18, timeout is 2 seconds:
!!!!!
R15#ping 198.0.52.22
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 198.0.52.22, timeout is 2 seconds:
!!!!!

R15#ping 2001:1:520:1051::28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1:520:1051::28, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R15#ping 2001:1:520:1052::28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1:520:1052::28, timeout is 2 seconds:
!!!!!
```

Лабытнанги R27 - Чокурдах R28

```
R27#ping 198.0.52.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 198.0.52.18, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R27#ping 198.0.52.22
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 198.0.52.22, timeout is 2 seconds:
!!!!!

R27#ping 2001:1:520:1051::28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1:520:1051::28, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R27#ping 2001:1:520:1052::28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1:520:1052::28, timeout is 2 seconds:
!!!!!
```

