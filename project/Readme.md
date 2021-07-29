### Модернизация сети компании. Применение архитектуры DMVPN - Dual hub, dual cloud. Включение динамической маршрутизации iBGP внутри туннеля DMVPN. Использование IPSec для защиты данных в туннеле.



#### Топология



![](/Users/leonov/Documents/OTUS/PROJECT/project.svg)



Настройка туннеля DMVPN Single hub, dual cloud с VRF

Настройка протокола IPSec:

```
crypto keyring VRF_PSK vrf FVRF
 pre-shared-key address 0.0.0.0 0.0.0.0 key OTUS-KEY
!
!
crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
!
crypto ipsec transform-set AES-256 esp-aes 256 esp-sha-hmac
 mode transport
!
crypto ipsec profile DMVPN
 set transform-set AES-256
```

Настройка туннельных интерфейсов

HUB:

```
R16(config)#int tunnel 100
R16(config-if)#tunnel mode gre multipoint
R16(config-if)#ip address 10.64.0.1 255.255.255.0
R16(config-if)#tunnel source e0/1
R16(config-if)#ip mtu 1400
R16(config-if)#ip tcp adjust-mss 1360
R16(config-if)#ip nhrp network-id 100
R16(config-if)#ip nhrp authentication OTUS
R16(config-if)#ip nhrp map multicast dynamic
R16(config-if)#tunnel key 101
R16(config-if)#tunnel protection ipsec profile DMVPN shared
```

```
interface Tunnel200
 ip address 10.65.0.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication OTUS
 ip nhrp map multicast dynamic
 ip nhrp network-id 200
 ip nhrp holdtime 600
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
 tunnel key 102
 tunnel vrf FVRF
 tunnel protection ipsec profile DMVPN shared
```

SPOKE:

```
 interface Tunnel100
 ip address 10.64.0.3 255.255.255.0
 ip mtu 1400
 ip nhrp authentication OTUS
 ip nhrp map multicast 89.208.35.213
 ip nhrp map 10.64.0.1 89.208.35.213
 ip nhrp network-id 100
 ip nhrp holdtime 600
 ip nhrp nhs 10.64.0.1
 ip tcp adjust-mss 1360
 tunnel source Ethernet1/0
 tunnel destination 89.208.35.213
 tunnel key 101
 tunnel vrf FVRF1
 tunnel protection ipsec profile DMVPN
```

```
interface Tunnel200
 ip address 10.65.0.3 255.255.255.0
 ip mtu 1400
 ip nhrp authentication OTUS
 ip nhrp map multicast 89.208.35.213
 ip nhrp map 10.65.0.1 89.208.35.213
 ip nhrp network-id 200
 ip nhrp holdtime 600
 ip nhrp nhs 10.65.0.1
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/3
 tunnel destination 89.208.35.213
 tunnel key 102
 tunnel vrf FVRF2
 tunnel protection ipsec profile DMVPN
```

Проверка подлючение на хабе:

```
R16(config-if)#do sh dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
	N - NATed, L - Local, X - No Socket

	# Ent --> Number of NHRP entries with same NBMA peer

NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting

	UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel100, IPv4 NHRP Details
Type:Hub, NHRP Peers:3,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb

----- --------------- --------------- ----- -------- -----

     1 212.44.128.106        10.64.0.2    UP 00:07:47     D
     1 89.109.236.2          10.64.0.3    UP 03:52:45     D
     1 89.249.249.147        10.64.0.4    UP 04:09:14     D

Interface: Tunnel200, IPv4 NHRP Details
Type:Hub, NHRP Peers:3,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb

----- --------------- --------------- ----- -------- -----

     1 185.116.203.35        10.65.0.2    UP 02:48:20     D
     1 94.228.196.245        10.65.0.3    UP 04:09:42     D
     1 89.249.249.147        10.65.0.4    UP 04:09:02     D
```



Настройка динамической маршрутизации iBGP с помощью приватной автономной системы 65100. На хабе используется Route Reflector

HUB:

```
router bgp 65100
 bgp router-id 16.16.16.16
 bgp log-neighbor-changes
 bgp listen range 10.64.0.0/15 peer-group DMVPN-SPOKES
 network 192.168.15.0
 redistribute connected
 redistribute static
 neighbor DMVPN-SPOKES peer-group
 neighbor DMVPN-SPOKES remote-as 65100
 neighbor DMVPN-SPOKES update-source Tunnel100
 neighbor DMVPN-SPOKES route-reflector-client
 
 address-family ipv4 vrf FVRF
  neighbor 89.208.35.1 remote-as 12695
  neighbor 89.208.35.1 activate
```

SPOKE:

```
router bgp 65100
 bgp router-id 7.7.7.7
 bgp log-neighbor-changes
 network 10.181.0.0 mask 255.255.255.0
 network 10.181.1.0 mask 255.255.255.0
 network 10.181.2.0 mask 255.255.255.0
 redistribute connected
 redistribute static
 neighbor 10.64.0.1 remote-as 65100
 neighbor 10.65.0.1 remote-as 65100
 !
 address-family ipv4 vrf FVRF1
  network 89.109.236.0 mask 255.255.255.248
  neighbor 89.109.236.1 remote-as 25515
  neighbor 89.109.236.1 activate
 exit-address-family
 !
 address-family ipv4 vrf FVRF2
  network 94.228.196.240 mask 255.255.255.248
  neighbor 94.228.196.244 remote-as 48293
  neighbor 94.228.196.244 activate
```

Проверка соседства BGP:

```
R16#sh ip bgp all summ
For address family: IPv4 Unicast
BGP router identifier 16.16.16.16, local AS number 65100
BGP table version is 57, main routing table version 57
14 network entries using 1960 bytes of memory
37 path entries using 2960 bytes of memory
4/4 BGP path/bestpath attribute entries using 576 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 5640 total bytes of memory
BGP activity 41/17 prefixes, 89/42 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
*10.64.0.2      4        65100      51      59       57    0    0 00:40:34        6
*10.64.0.3      4        65100      53      61       57    0    0 00:41:50        8
*10.64.0.4      4        65100     314     341       57    0    0 04:40:59        3
*10.65.0.2      4        65100      50      58       57    0    0 00:39:23        6
*10.65.0.3      4        65100      55      60       57    0    0 00:41:20        8
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
*10.65.0.4      4        65100       6      15       57    0    0 00:00:12        3

* Dynamically created based on a listen range command
  Dynamically created neighbors: 6, Subnet ranges: 1

BGP peergroup DMVPN-SPOKES listen range group members:
  10.64.0.0/15

For address family: VPNv4 Unicast
BGP router identifier 16.16.16.16, local AS number 65100
BGP table version is 11, main routing table version 11
10 network entries using 1520 bytes of memory
10 path entries using 800 bytes of memory
6/6 BGP path/bestpath attribute entries using 912 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3376 total bytes of memory
BGP activity 41/17 prefixes, 89/42 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
89.208.35.1     4        12695     328     314       11    0    0 04:42:57       10
```

6 соседей iBGP и один сосед eBGP

Таблицы маршрутизации на хабе:

```
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 15 subnets, 3 masks

C        10.64.0.0/24 is directly connected, Tunnel100
L        10.64.0.1/32 is directly connected, Tunnel100
C        10.65.0.0/24 is directly connected, Tunnel200
L        10.65.0.1/32 is directly connected, Tunnel200
B        10.180.0.0/24 [200/20] via 10.180.32.2, 00:44:01
B        10.180.1.0/24 [200/20] via 10.180.32.6, 00:44:01
B        10.180.32.0/30 [200/0] via 10.64.0.2, 00:44:06
B        10.180.32.4/30 [200/0] via 10.64.0.2, 00:44:06
B        10.181.0.0/24 [200/20] via 10.181.32.2, 00:45:17
B        10.181.1.0/24 [200/20] via 10.181.32.6, 00:45:17
B        10.181.2.0/24 [200/20] via 10.181.32.10, 00:45:17
B        10.181.32.0/30 [200/0] via 10.64.0.3, 00:45:22
B        10.181.32.4/30 [200/0] via 10.64.0.3, 00:45:22
B        10.181.32.8/30 [200/0] via 10.64.0.3, 00:45:22
B        10.185.0.0/24 [200/0] via 10.64.0.4, 04:44:31
      192.168.15.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.15.0/24 is directly connected, Ethernet0/0
L        192.168.15.2/32 is directly connected, Ethernet0/0
```

```
Gateway of last resort is 89.208.35.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 89.208.35.1
      89.0.0.0/8 is variably subnetted, 5 subnets, 4 masks
B        89.0.25.0/29 [20/0] via 89.208.35.1, 04:46:25
B        89.109.236.0/29 [20/0] via 89.208.35.1, 04:46:25
C        89.208.35.0/24 is directly connected, Ethernet0/1
L        89.208.35.213/32 is directly connected, Ethernet0/1
B        89.249.249.128/27 [20/0] via 89.208.35.1, 04:46:25
      94.0.0.0/29 is subnetted, 2 subnets
B        94.0.48.0 [20/0] via 89.208.35.1, 04:46:25
B        94.228.196.240 [20/0] via 89.208.35.1, 04:46:25
      185.0.0.0/29 is subnetted, 1 subnets
B        185.0.60.0 [20/0] via 89.208.35.1, 04:46:25
      185.116.0.0/24 is subnetted, 1 subnets
B        185.116.203.0 [20/0] via 89.208.35.1, 04:46:25
      212.0.32.0/29 is subnetted, 1 subnets
B        212.0.32.0 [20/0] via 89.208.35.1, 04:46:25
      212.44.128.0/30 is subnetted, 1 subnets
B        212.44.128.104 [20/0] via 89.208.35.1, 04:46:25
```

Проверка связи между офисами и облаком:

Между компьютером в Москве и компьютером на Складе

```
VPCS> ping 10.185.0.10

84 bytes from 10.185.0.10 icmp_seq=1 ttl=60 time=6.591 ms
84 bytes from 10.185.0.10 icmp_seq=2 ttl=60 time=5.569 ms
84 bytes from 10.185.0.10 icmp_seq=3 ttl=60 time=5.815 ms
84 bytes from 10.185.0.10 icmp_seq=4 ttl=60 time=5.538 ms
84 bytes from 10.185.0.10 icmp_seq=5 ttl=60 time=5.012 ms
```

```
VPCS> trace  10.185.0.10
trace to 10.185.0.10, 8 hops max, press Ctrl+C to stop
 1   10.180.1.1   1.400 ms  0.985 ms  1.392 ms
 2   10.180.32.5   1.776 ms  1.568 ms  2.235 ms
 3   10.64.0.1   7.842 ms  3.243 ms  3.573 ms
 4   10.64.0.4   5.074 ms  5.021 ms  4.555 ms
 5   *10.185.0.10   6.431 ms (ICMP type:3, code:3, Destination port unreachable)
```

Между производством и облаком:

```
VPCS> ping 192.168.15.11

84 bytes from 192.168.15.11 icmp_seq=1 ttl=61 time=6.210 ms
84 bytes from 192.168.15.11 icmp_seq=2 ttl=61 time=3.455 ms
84 bytes from 192.168.15.11 icmp_seq=3 ttl=61 time=3.740 ms
84 bytes from 192.168.15.11 icmp_seq=4 ttl=61 time=3.322 ms
84 bytes from 192.168.15.11 icmp_seq=5 ttl=61 time=3.469 ms
```

```
VPCS> trace 192.168.15.11
trace to 192.168.15.11, 8 hops max, press Ctrl+C to stop
 1   10.181.0.1   0.933 ms  0.795 ms  1.137 ms
 2   10.181.32.1   1.407 ms  1.083 ms  1.163 ms
 3   10.64.0.1   3.379 ms  3.227 ms  3.284 ms
 4   *192.168.15.11   3.951 ms (ICMP type:3, code:3, Destination port unreachable)
```

Между складом и облаком:

```
VPCS> ping 192.168.15.11

84 bytes from 192.168.15.11 icmp_seq=1 ttl=62 time=5.630 ms
84 bytes from 192.168.15.11 icmp_seq=2 ttl=62 time=2.261 ms
84 bytes from 192.168.15.11 icmp_seq=3 ttl=62 time=2.719 ms
84 bytes from 192.168.15.11 icmp_seq=4 ttl=62 time=2.872 ms
84 bytes from 192.168.15.11 icmp_seq=5 ttl=62 time=2.555 ms
```

```
VPCS> trace 192.168.15.11
trace to 192.168.15.11, 8 hops max, press Ctrl+C to stop
 1   10.185.0.1   0.959 ms  0.896 ms  0.786 ms
 2   10.64.0.1   2.045 ms  1.687 ms  1.858 ms
 3   *192.168.15.11   2.439 ms (ICMP type:3, code:3, Destination port unreachable)
```



#### Реализация схемы  Dual Hub, Dual Cloud

Для реализации схемы  Dual Hub, Dual Cloud  добавим в облако 2 маршрутизатора:

![](project-dualcloud-2.svg)



Настроим  IPSec на новом маршрутизаторе R44

```
crypto keyring VRF_PSK vrf FVRF
 pre-shared-key address 0.0.0.0 0.0.0.0 key OTUS-KEY
!
!
crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
!
crypto ipsec transform-set AES-256 esp-aes 256 esp-sha-hmac
 mode transport
!
crypto ipsec profile DMVPN
 set transform-set AES-256
```

Перенастроим второй туннельный интерфейс Tunnel200 с маршрутизатора R16 на маршрутизатор R44

```
interface Tunnel200
 ip address 10.65.0.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication OTUS
 ip nhrp map multicast dynamic
 ip nhrp network-id 200
 ip nhrp holdtime 600
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
 tunnel key 102
 tunnel vrf FVRF
 tunnel protection ipsec profile DMVPN shared
```

 На Spoke

В настройке туннельного интерфейса Tunnel200 



Необходимо изменить tunnel source для туннельного интерфейса Tunnel200

```
 interface Tunnel100
 ip address 10.64.0.3 255.255.255.0
 ip mtu 1400
 ip nhrp authentication OTUS
 ip nhrp map multicast 89.208.35.213
 ip nhrp map 10.64.0.1 89.208.35.213
 ip nhrp network-id 100
 ip nhrp holdtime 600
 ip nhrp nhs 10.64.0.1
 ip tcp adjust-mss 1360
 tunnel source Ethernet1/0
 tunnel destination 89.208.35.213
 tunnel key 101
 tunnel vrf FVRF1
 tunnel protection ipsec profile DMVPN
```

изменить адреса в строках 

```
 ip nhrp map multicast 89.208.32.1
 ip nhrp map 10.64.0.1 89.208.32.1
```

