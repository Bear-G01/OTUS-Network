## BGP. Фильтрация

### Цель:

Настроить фильтрацию для офиса в Москве; 

Настроить фильтрацию для офиса в С.-Петербург.

### Задачи:

1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path);
2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list);
3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по-умолчанию;
4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по-умолчанию и префикс офиса С.-Петербург;
5. Все сети в лабораторной работе должны иметь IP связность;



#### Решение:

#### Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path);

Для эмуляции транзитного трафика с маршрутизаторов R22 и R21 отключим внешние линки

![](L:\OTUS\3.25.BGP-Filter\lab25-1.svg)



На маршрутизаторе R21 настроен loopback-интерфейс с адресом 8.8.2.1. На маршрутизаторе R22 настроен loopback-интерфейс с адресом 8.8.2.2

На маршрутизаторе R22 получаем префикс 8.8.2.1/32 через AS 1001 (Москва)

```
R22(config-if)#do sh ip bgp
BGP table version is 32, local router ID is 10.1.0.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path

 *>  8.8.2.1/32       188.0.1.2                              0 1001 301 i
 *>  8.8.2.2/32       0.0.0.0                  0         32768 i
 *>  98.0.31.0/30     188.0.1.2                              0 1001 i
 *>  188.0.1.0/30     0.0.0.0                  0         32768 i
```

Запретим прохождение транзитного трафика на маршрутизаторах R14 и R15 через локальную AS, разрешив только нетранзитный трафик
R14

```
R14(config)# ip as-path access-list 1 permit ^$
R14(config)# ip as-path access-list 1 deny .*
R14(config)# router bgp 1001
R14(config-router)# address-family ipv4
R14(config-router-af)# neighbor 188.0.1.1 filter-list 1 out
```
Для IPv6

```
R14(config)# ipv6 as-path access-list 1 permit ^$
R14(config)# ipv6 as-path access-list 1 deny .*
R14(config)# router bgp 1001
R14(config-router)# address-family ipv6
R14(config-router-af)#neighbor 2001:1:101:1001::22 filter-list 1 out
```
R15

```
R15(config)# ip as-path access-list 1 permit ^$
R15(config)# ip as-path access-list 1 deny .*
R15(config)# router bgp 1001
R14(config-router)# address-family ipv4
R15(config-router-af)# neighbor 98.0.31.1 filter-list 1 out
```
Для IPv6

```
R15(config)# ipv6 as-path access-list 1 permit ^$
R15(config)# ipv6 as-path access-list 1 deny .*
R15(config)# router bgp 1001
R15(config-router)# address-family ipv6
R15(config-router-af)# neighbor 2001:1:301:1001::21 filter-list 1 out
```

После фильтрации транзитного трафика маршрутизатор R22  больше не получает префикс 8.8.2.1/32 

```
R22>sh ip bgp
BGP table version is 33, local router ID is 10.1.0.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path

 *>  8.8.2.2/32       0.0.0.0                  0         32768 i
 *>  98.0.31.0/30     188.0.1.2                              0 1001 i
 *>  188.0.1.0/30     0.0.0.0                  0         32768 i
```



### Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list)



![](L:\OTUS\3.25.BGP-Filter\lab25-2.svg)



Ограничим рассылку всех апдейтов, кроме апдейтов с префиксами Санкт-Петербурга

```
R18(config)#ip prefix-list TRANSIT-DENIED seq 10 permit 198.0.52.8/30
R18(config)#ip prefix-list TRANSIT-DENIED seq 15 permit 198.0.52.12/30
R18(config)#router bgp 2042
R18(config-router)#address-family ipv4
R18(config-router-af)#neighbor 198.0.52.9 prefix-list TRANSIT-DENIED out
R18(config-router-af)#neighbor 198.0.52.13 prefix-list TRANSIT-DENIED out
```

Для IPv6

```
R18(config)#ipv6 prefix-list TRANSIT-DENIED-V6 seq 10 permit 2001:1:520:1042::/64
R18(config)#ipv6 prefix-list TRANSIT-DENIED-V6 seq 15 permit 2001:1:520:1043::/64
R18(config)#router bgp 2042
R18(config-router)#address-family ipv6
R18(config-router)#address-family ipv6
R18(config-router-af)#neighbor 2001:1:520:1042::24 sec 10 prefix-list TRANSIT-DENIED-V6 out
R18(config-router-af)#neighbor 2001:1:520:1043::26 sec 15 prefix-list TRANSIT-DENIED-V6 out
```



### Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по-умолчанию



Настроим prefix-list чтобы с маршрутизатора R22 приходили апдейты только маршрута по-умолчанию и установим анонсирование соседу R14 маршрута по-умолчанию



```
R22(config)#ip prefix-list DEFAULT-ONLY seq 5 permit 0.0.0.0/0
R22(config)#router bgp 101
R22(config-router)#address-family ipv4
R22(config-router-af)#neighbor 188.0.1.2 prefix-list DEFAULT-ONLY out
R22(config-router-af)#neighbor 188.0.1.2 default-originate
```

Удалим предварительно настроенный маршрут по-умолчанию на маршрутизаторе R14

```
R14(config)#no ip route 0.0.0.0 0.0.0.0 188.0.1.1
```

Проверим таблицу BGP

```
R14(config)#do sh ip bgp
BGP table version is 58, local router ID is 10.0.1.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path

 *>  0.0.0.0          188.0.1.1                              0 101 i
 *>i 8.8.2.1/32       98.0.31.1                0    200      0 301 i
 *>i 8.8.2.2/32       98.0.31.1                0    200      0 301 101 i
 *>i 98.0.31.0/30     15.15.15.15              0    200      0 i
 r>i 188.0.1.0/30     98.0.31.1                0    200      0 301 101 i
 *>i 188.0.101.4/30   98.0.31.1                0    200      0 301 i
 *>i 198.0.52.0/30    98.0.31.1                0    200      0 301 520 i
 *>i 198.0.52.4/30    98.0.31.1                0    200      0 301 i
 *>i 198.0.52.8/30    98.0.31.1                0    200      0 301 520 i
 *>i 198.0.52.12/30   98.0.31.1                0    200      0 301 520 i
 *>i 198.0.52.16/30   98.0.31.1                0    200      0 301 520 i
 *>i 198.0.52.20/30   98.0.31.1                0    200      0 301 520 i
 *>i 198.0.52.24/30   98.0.31.1                0    200      0 301 520 i
```

Получаем маршрут по-умолчанию от маршрутизатора R22

Произведем аналогичные настройки для IPv6

R22

```
R22(config)#ipv6 prefix-list DEFAULT-ONLY-V6 seq 5 permit ::/0
R22(config)#router bgp 101
R22(config-router)#address-family ipv6
R22(config-router-af)#neighbor 2001:1:101:1001::14 prefix-list DEFAULT-ONLY-V6 out
R22(config-router-af)#$01:1:101:1001::14 def
R22(config-router-af)#neighbor 2001:1:101:1001::14 default-originate
```

Удалим предварительно настроенный статический маршрут по-умолчанию на R14

```
R14(config)# no ipv6 route ::/0 2001:1:101:1001::22
```

R14

```
R14#sh ip bgp ipv6 unicast
BGP table version is 59, local router ID is 10.0.1.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  ::/0             2001:1:101:1001::22
                                                              0 101 i
 * i 2001:1:101:301::/64
                       2001:1:301:1001::21
                                                0    200      0 301 i
 * i 2001:1:101:1001::/64
                       2001:1:301:1001::21
                                                0    200      0 301 101 i
 *>i 2001:1:301:1001::/64
                       2001:1:301:1001::21
                                                0    200      0 301 i
 * i 2001:1:520:101::/64
                       2001:1:301:1001::21
                                                0    200      0 301 520 i
 * i 2001:1:520:301::/64
                       2001:1:301:1001::21
                                                0    200      0 301 i
 * i 2001:1:520:1042::/64
                       2001:1:301:1001::21
                                                0    200      0 301 520 i
 * i 2001:1:520:1043::/64
                       2001:1:301:1001::21
                                                0    200      0 301 520 i
 * i 2001:1:520:1051::/64
                       2001:1:301:1001::21
                                                0    200      0 301 520 i
 * i 2001:1:520:1052::/64
                       2001:1:301:1001::21
                                                0    200      0 301 520 i
 * i 2001:1:520:1061::/64
                       2001:1:301:1001::21
                                                0    200      0 301 520 i
```

Получаем на R14 маршрут по-умолчанию с R22



### Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по-умолчанию и префикс офиса С.-Петербург

Настроим маршрутизатор R21 аналогично маршрутизатору R22 и добавим в prefix-list разрешающие правила для префиксов Санкт-Петербурга

```
R21(config)#ip prefix-list DEFAULT-ONLY-PETER seq 5 permit 0.0.0.0/0
R21(config)#ip prefix-list DEFAULT-ONLY-PETER seq 10 permit 198.0.52.8/30
R21(config)#ip prefix-list DEFAULT-ONLY-PETER seq 15 permit 198.0.52.12/30
R21(config)#router bgp 301
R21(config-router)#address-family ipv4
R21(config-router-af)#neighbor 98.0.31.2 prefix-list DEFAULT-ONLY-PETER out
R21(config-router-af)#neighbor 98.0.31.2 default-originate
```

Проверим таблицу BGP на маршрутизаторе R15

```
R15#sh ip bgp
BGP table version is 67, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path

 *>  0.0.0.0          98.0.31.1                              0 301 i
 *>  98.0.31.0/30     0.0.0.0                  0         32768 i
 *>  198.0.52.8/30    98.0.31.1                              0 301 520 i
 *>  198.0.52.12/30   98.0.31.1                              0 301 520 i
```

Произведем аналогичные настройки для IPv6

```
R21(config)#ipv6 prefix-list DEFAULT-ONLY-PETER-V6 seq 5 permit ::/0
R21(config)#ipv6 prefix-list DEFAULT-ONLY-PETER-V6 seq 10 permit 2001:1:520:1042::/64
R21(config)#ipv6 prefix-list DEFAULT-ONLY-PETER-V6 seq 15 permit 2001:1:520:1043::/64
R21(config)#router bgp 301
R21(config-router)#address-family ipv6
R21(config-router-af)#neighbor 2001:1:301:1001::15 prefix-list DEFAULT-ONLY-PETER-V6 out
R21(config-router-af)#neighbor 2001:1:301:1001::15 default-originate
```

Проверим таблицу BGP на маршрутизаторе R15

```
R15#sh ip bgp ipv6 unicast
BGP table version is 48, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  ::/0             2001:1:301:1001::21
                                                              0 301 i
 *>  2001:1:520:1042::/64
                       2001:1:301:1001::21
                                                              0 301 520 i
 *>  2001:1:520:1043::/64
                       2001:1:301:1001::21
                                                              0 301 520 i

```

### Все сети в лабораторной работе должны иметь IP связность

Проверим связность измененных сетей

Доступность маршрутизаторов R14 и R15 в московском офисе из офиса Санкт-Петербурга

```
R18#ping 188.0.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 188.0.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/2 ms
R18#ping 98.0.31.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 98.0.31.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

IPv6

```
R18#ping 2001:1:301:1001::15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1:301:1001::15, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/8/21 ms
R18#ping 2001:1:101:1001::14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:1:101:1001::14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

