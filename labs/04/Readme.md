# Имплементация DHCPv4

### Топология

![Topology](lab04.svg)

### Таблица адресации
| Устройство | Интерфейс    | IP-адрес      | Маска подсети    | Маршрут по умолчанию |
| ---------- | ----------- | -----------  | --------------- | -------------------- |
| R1         | G0/0/0      | 10.0.0.1     | 255.255.255.252 | N/A                  |
|            | G0/0/1      | N/A          | N/A             |                      |
|            | G0/0/1.100  | 192.168.1.1  | 255.255.255.192 |                      |
|            | G0/0/1.200  | 192.168.1.65 | 255.255.255.224 |                      |
|            | G0/0/1.1000 | N/A          | N/A             |                      |
| R2         | G0/0/0      | 10.0.0.2     | 255.255.255.252 | N/A                  |
|            | G0/0/1      | 192.168.1.97 | 255.255.255.240 |                      |
| S1         | VLAN200     | 192.168.1.2  | 255.255.255.192 | 192.168.1.1          |
| S2         | VLAN1       | 192.168.1.66 | 255.255.255.224 | 192.168.1.65         |
| PC-A       | NIC         | DHCP         | DHCP            | DHCP                 |
| PC-B       | NIC         | DHCP         | DHCP            | DHCP                 |

### Таблица VLAN


| VLAN ID | Имя        | Назначенный интерфейс         |
| ------- | ---------- | --------------------------  |
| 1       | N/A        | S2: F0/18                   |
| 100     | Clients    | S1: F0/6                    |
| 200     | Management | S1: VLAN 200                |
| 999     | Parking_Lot| S1: F0/1-4, F0/7-24, G0/1-2 |
| 1000    | Native     | N/A                         |

### Цели
* Часть 1. Построить сеть и сконфигурировать основные настройки устройств.
* Часть 2. Настроить и проверить 2 DHCPv4-сервера на R1.
* Часть 3. Настроить и проверить DHCP Relay на R2.


### Часть 1. Конфигурация базовых настроек на маршрутизаторах R1/R2.

Подключение к маршрутизатору и вход в привилегированный режим:<br>
`enable`

Вход в режим конфигурации:<br>
`configure terminal`

Указание имени маршрутизатора:<br>
`hostname R1` - на R1<br>
`hostname R2` - на R2

Отключение разрешения имен DNS:<br>
`no ip domain-lookup`

Установка паролей на вход в привилегированный режим, консольному и терминальному подключенииям. Включение шифрования паролей в виде простого текста:

```
enable secret class
line console 0
password cisco
login
exit
line vty 0 4
password cisco
login
exit
service password-encryption
```

Установка баннера при подключении

```
banner motd C
===============================================================================
        Connections are logged. Unauthorized access is prohibited.           
===============================================================================
C
```

Копирование текущей конфигурации в файл стартовой конфигурации<br>
`copy running-config startup-config`

Установка времени на маршрутизаторе
```
configure terminal
clock set HH:MM:SS MONTH DD YEAR
```

#### Конфигурирование меж-VLAN маршрутизации на R1
 
Включение интерфейса G0/0/1
```
int G0/0/1
no shutdown
```

Конфигурирование подинтерфейсов для каждого VLAN согласно таблице
```
interface G0/0/1.100
description Default Gateway for VLAN 100
encapsulation dot1Q 100
ip address 192.168.1.1 255.255.255.192
exit
interface G0/0/1.200
description Default Gateway for VLAN 200
encapsulation dot1Q 200
ip address 192.168.1.65 255.255.255.224
exit
interface G0/0/1.1000
description Native VLAN
encapsulation dot1Q 1000
end
```

Проверка работы подинтерфейсов<br>
`show ip interface brief`

#### Настройка интерфейсов G0/0/1, G0/0/0 на R2 и статических маршрутов на R1/R2

Настройка IP-адреса на R2 G0/0/1

```
interface G0/0/1
ip address 192.168.1.97 255.255.255.240
no shutdown
```

Настройка IP-адреса на интерфейсах G0/0/0 маршрутизаторов R1/R2

R1:
```
interface G0/0/0
ip address 10.0.0.1 255.255.255.252
no shutdown
```

R2:
```
interface G0/0/0
ip address 10.0.0.2 255.255.255.252
no shutdown
```
Настройка маршрута по умолчанию на маршрутизаторах R1/R2

`ip route 0.0.0.0 0.0.0.0 10.0.0.2` - R1<br>
`ip route 0.0.0.0 0.0.0.0 10.0.0.1` - R2

Проверка корректности настройки статических маршрутов (доступности интерфейса R2 G0/0/1 c R1)
```
R1#ping 192.168.1.97

Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```
Копирование текущей конфигурации в файл стартовой конфигурации<br>
`copy running-config startup-config`

#### Конфигурация базовых настроек на коммутаторах S1 S2.

Подключение к коммутатору и вход в привилегированный режим:<br>
`enable`

Вход в режим конфигурации:<br>
`configure terminal`

Указание имени коммутатора:<br>
`hostname S1` - на S1 <br>
`hostname S2` - на S2

Отключение разрешения имен DNS:<br>
`no ip domain-lookup`

Установка паролей на вход в привилегированный режим, консольному и терминальному подключенииям. Включение шифрования паролей в виде простого текста:

```
enable secret class
line console 0
password cisco
login
end
line vty 0 15
password cisco
login
exit
service password-encryption
```

Установка баннера при подключении

```
banner motd C
===============================================================================
        Connections are logged. Unauthorized access is prohibited.           
===============================================================================
C
```

Копирование текущей конфигурации в файл стартовой конфигурации<br>
`copy running-config startup-config`

Установка времени на коммутаторе<br>
`clock set HH:MM:SS MONTH DD YEAR`

#### Создание VLAN на S1

Создание VLAN и назначение имени на S1

```
configure terminal
vlan 100
name Clients
exit
vlan 200
name Management
exit
vlan 999
name ParkingLot
exit
```
Назначение IP-адреса на VLAN 200. Настройка шлюза по умолчанию на S1:
```
interface vlan 200
ip address 192.168.1.2 255.255.255.192
exit
ip default-gateway 192.168.1.1
```

Назначение IP-адреса на S2 VLAN 1. Настройка шлюза по умолчанию на S2:
```
interface vlan 1
ip address 192.168.1.66 255.255.255.224
exit
ip default-gateway 192.168.1.65
```

Назначение всех неиспользуемых портов на VLAN 999 ParkingLot на S1
```
interface range FastEthernet 0/2-4, FastEthernet 0/7-24, GigabitEthernet 0/1-2
switchport mode access
switchport access vlan 999
shutdown
```
Отключение всех неиспользуемых портов на S2

```
interface range FastEthernet 0/2-4, FastEthernet 0/6-17, FastEthernet 0/19-24, GigabitEthernet 0/1-2
shutdown
```
#### Назначение VLAN интерфейсам коммутаторов

S1

```
interface FastEthernet 0/6
switchport mode access
switchport access vlan 100
```

S2
```
interface FastEthernet 0/18
switchport mode access
switchport access vlan 1
```
#### Настройка вручную интерфейса S1 F0/5 как 802.1Q trunk.

Изменение режима работы порта
```
interface FastEthernet 0/5
switchport mode trunk
```
Изменение native VLAN на 1000<br>
`switchport trunk native vlan 1000`

Разрешение прохождения VLAN 100, 200, 1000 через trunk<br>
`switchport trunk allowed vlan 100, 200, 1000`

Копирование текущей конфигурации в файл стартовой конфигурации<br>
`copy running-config startup-config`

Проверка статуса trunk
```
S1#show interfaces trunk 
Port        Mode         Encapsulation  Status        Native vlan
Fa0/5       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/5       100,200,1000

Port        Vlans allowed and active in management domain
Fa0/5       100,200

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/5       100,200
```

Возможные адреса DHCP, если бы он был включен:

со стороны PC-A: 192.168.1.3-192.168.1.62
со стороны PC-B: 192.168.1.98-192.168.1.110

### Часть 2. Настройка проверка двух DHCPv4-серверов на маршрутизаторе R1

#### Конфигурирование двух DHCP-пулов для двух поддерживаемых подсетей.

Исключение первых пяти используемых адресов каждого DHCP-пула
```
ip dhcp excluded-address 192.168.1.1 192.168.1.5
ip dhcp excluded-address 192.168.1.97 192.168.1.102
```
Создание DHCP-пула. Определение обслуживаемой пулом сети. Установка имени домена и шлюза по умолчанию.

```
ip dhcp pool POOL-Clients
network 192.168.1.0 255.255.255.192
default-router 192.168.1.1
domain-name ccna-lab.com
```
```
ip dhcp pool R2_Client_LAN
network 192.168.1.96 255.255.255.240
default-router 192.168.1.97
domain-name ccna-lab.com
```
Конфигурирование времени аренды адреса недоступно в Packet Tracer.

Копирование текущей конфигурации в файл стартовой конфигурации<br>
`copy running-config startup-config`

Проверка настройки DHCP-сервера:

```
Pool POOL-Clients :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 62
 Leased addresses               : 1
 Excluded addresses             : 9
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.1          192.168.1.1      - 192.168.1.62      1    / 9     / 62

Pool R2_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 14
 Leased addresses               : 0
 Excluded addresses             : 9
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.97         192.168.1.97     - 192.168.1.110     0    / 9     / 14
```
```
R1#show ip dhcp binding 
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
```

#### Попытка получения IP-адреса от DHCP на PC-A

```
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: ccna-lab.com
   Link-local IPv6 Address.........: FE80::2E0:A3FF:FE82:D9D8
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.6
   Subnet Mask.....................: 255.255.255.192
   Default Gateway.................: ::
                                     192.168.1.1
```
```
Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
```

### Часть 3. Конфигурирование и проверка DHCP Relay на маршрутизаторе R2

#### Настройка маршрутизатора R2 как DHCP relay для LAN на интерфейсе G0/0/1

Настройка адреса IP-helper на интерфейсе G0/0/1 с указанием адреса интерфейса R1 G0/0/0

```
interface G0/0/0
ip helper-address 10.0.0.1
```
Копирование текущей конфигурации в файл стартовой конфигурации<br>
`copy running-config startup-config`

Проверка получения IP-адреса на интерфейсе компьютера PC-B. Проверка соединения с интерфейсом R1 G0/0/1
```
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: ccna-lab.com
   Physical Address................: 0001.C956.A00C
   Link-local IPv6 Address.........: FE80::201:C9FF:FE56:A00C
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.103
   Subnet Mask.....................: 255.255.255.240
   Default Gateway.................: ::
                                     192.168.1.97
   DHCP Servers....................: 10.0.0.1
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-60-BD-57-67-00-01-C9-56-A0-0C
   DNS Servers.....................: ::
                                     0.0.0.0
```
```
Pinging 192.168.1.97 with 32 bytes of data:

Reply from 192.168.1.97: bytes=32 time<1ms TTL=255
```

Проверка выданных IP-адресов на маршрутизаторе R1
```
R1#show ip dhcp binding 
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      00E0.A382.D9D8           --                     Automatic
192.168.1.103    0001.C956.A00C           --                     Automatic
```
*команда ip dhcp server statistics недоступна в Packet Tracer* 