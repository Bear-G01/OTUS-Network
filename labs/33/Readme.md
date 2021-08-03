## Виртуальные частные сети - VPN



### Цель:

##### Настроить GRE между офисами Москва и С.-Петербург Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги

1. Настроить GRE между офисами Москва и С.-Петербург
2. Настроить DMVMN между Москва и Чокурдах, Лабытнанги
3. Все узлы в офисах в лабораторной работе должны иметь IP связность



##### Топология туннелей:

![](lab33.svg)

Настройка GRE туннеля и маршрутизации OSPF внутри туннеля

На R15:

```
interface Tunnel300
 ip address 10.255.0.1 255.255.255.0
 ip ospf network point-to-point
 ip ospf 1 area 17
 tunnel source 98.0.31.2
 tunnel destination 198.0.52.10
```

```
router ospf 1
no passive-interface Tunnel 300
```

на R18:

```
interface Tunnel300
 ip address 10.255.0.2 255.255.255.0
 ip ospf network point-to-point
 ip ospf 1 area 17
 tunnel source 198.0.52.10
 tunnel destination 98.0.31.2
```

```
router ospf 1
no passive-interface Tunnel 300
```

Настроим редистрибуцию маршрутов из EIGRP в OSPF на маршрутизаторе R18

```
router ospf 1
 redistribute eigrp 10 metric-type 1 subnets
```

Проверим доступность с компьютера в Москве 172.16.0.10 до компьютера в Санкт-Петербурге 172.17.0.10

```
VPCS> trace 172.17.0.10
trace to 172.17.0.10, 8 hops max, press Ctrl+C to stop
 1   172.16.0.4   1.449 ms  0.727 ms  0.845 ms
 2   172.16.32.33   1.287 ms  1.292 ms  1.657 ms
 3   172.16.32.9   1.860 ms  2.064 ms  1.811 ms
 4   10.255.0.2   3.052 ms  2.768 ms  3.726 ms
 5   172.17.32.2   3.623 ms  3.465 ms  3.229 ms
 6   172.17.32.10   3.674 ms  4.048 ms  4.816 ms
 7   *172.17.0.10   6.564 ms (ICMP type:3, code:3, Destination port unreachable)

VPCS> ping 172.17.0.10

84 bytes from 172.17.0.10 icmp_seq=1 ttl=58 time=4.324 ms
84 bytes from 172.17.0.10 icmp_seq=2 ttl=58 time=3.753 ms
84 bytes from 172.17.0.10 icmp_seq=3 ttl=58 time=4.933 ms
84 bytes from 172.17.0.10 icmp_seq=4 ttl=58 time=4.135 ms
84 bytes from 172.17.0.10 icmp_seq=5 ttl=58 time=4.462 ms
```



Настройка туннелей DMVPN и маршрутизации OSPF внутри туннеля:

На R14 (HUB)

```
interface Tunnel200
 ip address 10.65.0.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication OTUS
 ip nhrp map multicast dynamic
 ip nhrp network-id 102
 ip nhrp holdtime 600
 ip tcp adjust-mss 1360
 ip ospf network broadcast
 ip ospf 1 area 0
 tunnel source 188.0.1.2
 tunnel mode gre multipoint
```

```
router ospf 1
 no passive-interface Tunnel200
```

На R27

```
interface Tunnel200
 ip address 10.65.0.2 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication OTUS
 ip nhrp map 10.65.0.1 188.0.1.2
 ip nhrp map multicast 188.0.1.2
 ip nhrp network-id 102
 ip nhrp holdtime 600
 ip nhrp nhs 10.65.0.1
 ip tcp adjust-mss 1360
 ip ospf network broadcast
 ip ospf priority 0
 ip ospf 1 area 0
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
```

```
router ospf 1
 no passive-interface Tunnel200
```

На R28

```
interface Tunnel200
 ip address 10.65.0.3 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication OTUS
 ip nhrp map 10.65.0.1 188.0.1.2
 ip nhrp map multicast 188.0.1.2
 ip nhrp network-id 102
 ip nhrp holdtime 600
 ip nhrp nhs 10.65.0.1
 ip tcp adjust-mss 1360
 ip ospf network broadcast
 ip ospf priority 0
 ip ospf 1 area 0
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
```

```
router ospf 1
 no passive-interface Tunnel200
```

Таблица маршрутизации на R14:

```
R14#sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 172.16.32.6 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 172.16.32.6, 10:03:12, Ethernet0/0
                [110/1] via 172.16.32.2, 10:03:12, Ethernet0/1
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C        10.65.0.0/24 is directly connected, Tunnel200
L        10.65.0.1/32 is directly connected, Tunnel200
O IA     10.255.0.0/24 [110/1020] via 172.16.32.6, 10:13:00, Ethernet0/0
                       [110/1020] via 172.16.32.2, 10:13:00, Ethernet0/1
      12.0.0.0/32 is subnetted, 1 subnets
O        12.12.12.12 [110/11] via 172.16.32.6, 12:40:21, Ethernet0/0
      13.0.0.0/32 is subnetted, 1 subnets
O        13.13.13.13 [110/11] via 172.16.32.2, 12:40:21, Ethernet0/1
      14.0.0.0/32 is subnetted, 1 subnets
C        14.14.14.14 is directly connected, Loopback0
      15.0.0.0/32 is subnetted, 1 subnets
O        15.15.15.15 [110/21] via 172.16.32.6, 10:26:23, Ethernet0/0
                     [110/21] via 172.16.32.2, 10:26:23, Ethernet0/1
      172.16.0.0/16 is variably subnetted, 17 subnets, 3 masks
O IA     172.16.0.0/23 [110/21] via 172.16.32.6, 12:40:21, Ethernet0/0
                       [110/21] via 172.16.32.2, 12:40:21, Ethernet0/1
O IA     172.16.8.0/23 [110/21] via 172.16.32.6, 12:40:21, Ethernet0/0
                       [110/21] via 172.16.32.2, 12:40:21, Ethernet0/1
C        172.16.32.0/30 is directly connected, Ethernet0/1
L        172.16.32.1/32 is directly connected, Ethernet0/1
C        172.16.32.4/30 is directly connected, Ethernet0/0
L        172.16.32.5/32 is directly connected, Ethernet0/0
O        172.16.32.8/30 [110/20] via 172.16.32.6, 12:40:21, Ethernet0/0
O        172.16.32.12/30 [110/20] via 172.16.32.2, 12:40:21, Ethernet0/1
C        172.16.32.16/30 is directly connected, Ethernet0/3
L        172.16.32.17/32 is directly connected, Ethernet0/3
O IA     172.16.32.20/30 [110/30] via 172.16.32.6, 10:26:18, Ethernet0/0
                         [110/30] via 172.16.32.2, 10:26:18, Ethernet0/1
O IA     172.16.32.24/30 [110/20] via 172.16.32.6, 12:40:21, Ethernet0/0
O IA     172.16.32.28/30 [110/20] via 172.16.32.6, 12:40:21, Ethernet0/0
O IA     172.16.32.32/30 [110/20] via 172.16.32.2, 12:40:21, Ethernet0/1
O IA     172.16.32.36/30 [110/20] via 172.16.32.2, 12:40:21, Ethernet0/1
O E2     172.16.255.4/32 [110/20] via 172.16.32.6, 12:40:21, Ethernet0/0
                         [110/20] via 172.16.32.2, 12:40:21, Ethernet0/1
O        172.16.255.19/32 [110/11] via 172.16.32.18, 12:40:21, Ethernet0/3
      172.17.0.0/16 is variably subnetted, 5 subnets, 3 masks
O E1     172.17.0.0/23 [110/1040] via 172.16.32.6, 10:12:44, Ethernet0/0
                       [110/1040] via 172.16.32.2, 10:12:44, Ethernet0/1
O E1     172.17.8.0/23 [110/1040] via 172.16.32.6, 10:12:44, Ethernet0/0
                       [110/1040] via 172.16.32.2, 10:12:44, Ethernet0/1
O E1     172.17.32.0/27 [110/1040] via 172.16.32.6, 10:12:44, Ethernet0/0
                        [110/1040] via 172.16.32.2, 10:12:44, Ethernet0/1
O E1     172.17.32.0/30 [110/1040] via 172.16.32.6, 10:12:44, Ethernet0/0
                        [110/1040] via 172.16.32.2, 10:12:44, Ethernet0/1
O E1     172.17.32.4/30 [110/1040] via 172.16.32.6, 10:12:44, Ethernet0/0
                        [110/1040] via 172.16.32.2, 10:12:44, Ethernet0/1
      172.18.0.0/16 is variably subnetted, 3 subnets, 2 masks
O IA     172.18.0.0/24 [110/1011] via 10.65.0.3, 00:00:07, Tunnel200
O IA     172.18.8.0/24 [110/1011] via 10.65.0.3, 00:00:07, Tunnel200
O IA     172.18.32.0/30 [110/1010] via 10.65.0.3, 00:01:37, Tunnel200
      172.19.0.0/24 is subnetted, 1 subnets
O        172.19.0.0 [110/1010] via 10.65.0.2, 09:31:23, Tunnel200
      188.0.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        188.0.1.0/30 is directly connected, Ethernet0/2
L        188.0.1.2/32 is directly connected, Ethernet0/2
```

Проверим доступность всех узлов сети от компьютера 172.19.0.10 в Лабытнанги до остальных офисов

Москва

```
VPCS> ping 172.16.0.10

84 bytes from 172.16.0.10 icmp_seq=1 ttl=60 time=10.865 ms
84 bytes from 172.16.0.10 icmp_seq=2 ttl=60 time=4.572 ms
84 bytes from 172.16.0.10 icmp_seq=3 ttl=60 time=5.013 ms
84 bytes from 172.16.0.10 icmp_seq=4 ttl=60 time=7.477 ms
84 bytes from 172.16.0.10 icmp_seq=5 ttl=60 time=3.935 ms

VPCS> trace 172.16.0.10
trace to 172.16.0.10, 8 hops max, press Ctrl+C to stop
 1   172.19.0.1   0.367 ms  0.269 ms  0.343 ms
 2   10.65.0.1   3.642 ms  8.774 ms  8.409 ms
 3   172.16.32.2   4.454 ms  3.742 ms  3.783 ms
 4   172.16.32.38   3.050 ms  4.693 ms  3.717 ms
 5   *172.16.0.10   5.803 ms (ICMP type:3, code:3, Destination port unreachable)
```

Санкт-Петербург

```
VPCS> ping 172.17.0.10

84 bytes from 172.17.0.10 icmp_seq=1 ttl=57 time=8.439 ms
84 bytes from 172.17.0.10 icmp_seq=2 ttl=57 time=5.947 ms
84 bytes from 172.17.0.10 icmp_seq=3 ttl=57 time=5.107 ms
84 bytes from 172.17.0.10 icmp_seq=4 ttl=57 time=5.279 ms
84 bytes from 172.17.0.10 icmp_seq=5 ttl=57 time=5.961 ms

VPCS> trace 172.17.0.10
trace to 172.17.0.10, 8 hops max, press Ctrl+C to stop
 1   172.19.0.1   0.384 ms  0.650 ms  0.582 ms
 2   10.65.0.1   3.894 ms  8.370 ms  3.249 ms
 3   172.16.32.6   4.564 ms  3.400 ms  3.623 ms
 4   172.16.32.9   5.439 ms  3.893 ms  4.091 ms
 5   10.255.0.2   12.176 ms  5.168 ms  4.810 ms
 6   172.17.32.2   5.721 ms  4.959 ms  4.809 ms
 7   172.17.32.10   10.686 ms  4.978 ms  5.484 ms
 8   *172.17.0.10   7.205 ms (ICMP type:3, code:3, Destination port unreachable)
```

Чокурдах

```
VPCS> ping 172.18.0.10

84 bytes from 172.18.0.10 icmp_seq=1 ttl=60 time=6.796 ms
84 bytes from 172.18.0.10 icmp_seq=2 ttl=61 time=2.064 ms
84 bytes from 172.18.0.10 icmp_seq=3 ttl=61 time=6.120 ms
84 bytes from 172.18.0.10 icmp_seq=4 ttl=61 time=7.219 ms
84 bytes from 172.18.0.10 icmp_seq=5 ttl=61 time=6.046 ms

VPCS> trace 172.18.0.10
trace to 172.18.0.10, 8 hops max, press Ctrl+C to stop
 1   172.19.0.1   0.541 ms  0.821 ms  0.651 ms
 2   10.65.0.3   1.584 ms  1.486 ms  1.695 ms
 3   172.18.32.2   1.540 ms  1.705 ms  2.214 ms
 4   *172.18.0.10   1.811 ms (ICMP type:3, code:3, Destination port unreachable)
```

