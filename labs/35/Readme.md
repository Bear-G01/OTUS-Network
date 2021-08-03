## IPSec over DmVPN

#### Цель:

Настроить GRE поверх IPSec между офисами Москва и С.-Петербург Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги

1. Настроить GRE поверх IPSec между офисами Москва и С.-Петербург
2. Настроить DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги
3. Все узлы в офисах в лабораторной работе должны иметь IP связность

#### Решение:

##### Настроить GRE поверх IPSec между офисами Москва и С.-Петербург

На маршрутизаторах R15 и R18

Настроим политику IPSEC

```
crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
```

Настроим ключ с ожиданием для любого адреса и keepalive

```
crypto isakmp key OTUS-KEY address 0.0.0.0 0.0.0.0
crypto isakmp keepalive 10
```

Настроим transform-set и укажем, что IPSEC работает в режиме транспорта:

```
crypto ipsec transform-set AES-256 esp-aes 256 esp-sha-hmac
 mode transport
```

Настройка профиля IPSEC

```
crypto ipsec profile GRE-IPSEC
 set transform-set AES-256
```

Применим профиль на туннельном интерфейсе каждого маршрутизатора:

```
int Tunnel 300
 tunnel protection ipsec profile GRE-IPSEC
```

Проверка доступности:

```
VPCS> ping 172.17.0.10

84 bytes from 172.17.0.10 icmp_seq=1 ttl=58 time=5.353 ms
84 bytes from 172.17.0.10 icmp_seq=2 ttl=58 time=5.033 ms
84 bytes from 172.17.0.10 icmp_seq=3 ttl=58 time=3.377 ms
84 bytes from 172.17.0.10 icmp_seq=4 ttl=58 time=4.160 ms
84 bytes from 172.17.0.10 icmp_seq=5 ttl=58 time=4.369 ms

VPCS> trace 172.17.0.10
trace to 172.17.0.10, 8 hops max, press Ctrl+C to stop
 1   172.16.0.4   0.866 ms  0.782 ms  0.528 ms
 2   172.16.32.33   0.989 ms  1.251 ms  1.635 ms
 3   172.16.32.9   1.813 ms  1.541 ms  1.625 ms
 4   10.255.0.2   3.157 ms  3.149 ms  3.196 ms
 5   172.17.32.2   3.185 ms  3.506 ms  3.733 ms
 6   172.17.32.10   3.248 ms  3.301 ms  3.260 ms
 7   *172.17.0.10   4.250 ms (ICMP type:3, code:3, Destination port unreachable)
```



##### Настроить DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги

На маршрутизаторах R14, R27, R28

Настроим политику IPSEC

```
crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
```

Настроим ключ с ожиданием для любого адреса и keepalive

```
crypto isakmp key OTUS-KEY address 0.0.0.0 0.0.0.0
crypto isakmp keepalive 10
```

Настроим transform-set и укажем, что IPSEC работает в режиме транспорта:

```
crypto ipsec transform-set AES-256 esp-aes 256 esp-sha-hmac
 mode transport
```

Настройка профиля IPSEC

```
crypto ipsec profile DMVPN
 set transform-set AES-256
```

Применим профиль на туннельном интерфейсе каждого маршрутизатора:

```
int Tunnel 300
 tunnel protection ipsec profile DMVPN
```

Проверим туннель DMVPN на R14

```
R14#sh dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
	N - NATed, L - Local, X - No Socket
	# Ent --> Number of NHRP entries with same NBMA peer
	NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
	UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel200, IPv4 NHRP Details
Type:Hub, NHRP Peers:2,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 198.0.52.42           10.65.0.2    UP 09:27:52     D
     1 198.0.52.34           10.65.0.3    UP 06:06:56     D
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

