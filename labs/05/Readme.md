## Конфигурация DHCPv6

### Топология

![](lab04-1.svg)



### Таблица адресации

| Device | Interface | IPv6 Address          |
| ------ | --------- | --------------------- |
| R1     | G0/0/0/   | 2001:db8:acad:2::1/64 |
|        |           | fe80::1               |
|        | G0/0/1    | 2001:db8:acad:1::1/64 |
|        |           | fe80::1               |
| R2     | G0/0/0    | 2001:db8:acad:2::2/64 |
|        |           | fe80::2               |
|        | G0/0/1    | 2001:db8:acad:3::1/64 |
|        |           | fe80::1               |
| PC-A   | NIC       | DHCP                  |
| PC-B   | NIC       | DHCP                  |

### Цели

* **Часть 1: Построить сеть и сконфигурировать основные настройки устройств.**
* **Часть 2: Убедиться в назначении адреса SLAAC от маршрутизатора  R1**
* **Часть 3: Настроить и проверить Stateless DHCPv6 сервер на маршрутизаторе R1**
* **Часть 4: Настроить и проверить Stateful DHCPv6 сервер на маршрутизаторе R1**
* **Часть 5: Настроить и проверить DHCPv6 Relay на маршрутизаторе R2**

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

Отключение всех неиспользуемых портов на S2

```
interface range FastEthernet 0/2-4, FastEthernet 0/6-17, FastEthernet 0/19-24, GigabitEthernet 0/1-2
shutdown
```

#### Конфигурация базовых настроек на маршрутизаторах R1/R2.

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

Включение маршрутизации ipv6

`ipv6 unicast-routing`

Копирование текущей конфигурации в файл стартовой конфигурации<br>
`copy running-config startup-config`

#### Конфигурирование интерфейсов и маршрутизации на обоих маршрутизаторах R1 R2

Конфигурирование интерфейсов согласно таблице

На R1:

```
interface GigabitEthernet0/0/0
ipv6 address FE80::1 link-local
ipv6 address 2001:DB8:ACAD:2::1/64
exit
interface GigabitEthernet0/0/1
ipv6 address FE80::1 link-local
ipv6 address 2001:DB8:ACAD:1::1/64
```

На R2:

```
interface GigabitEthernet0/0/0
ipv6 address FE80::2 link-local
ipv6 address 2001:DB8:ACAD:2::2/64
exit
interface GigabitEthernet0/0/1
ipv6 address FE80::1 link-local
ipv6 address 2001:DB8:ACAD:3::1/64
```

Конфигурирование дефолтного маршрута

На R1:

`ipv6 route ::/0 2001:DB8:ACAD:2::2`

На R2:

`ipv6 route ::/0 2001:DB8:ACAD:2::1`

#### Убедиться в назначении адреса SLAAC от маршрутизатора  R1

Получение на компьютере PC-A ipv6-адреса методом SLAAC

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::201:43FF:FE06:961
   IPv6 Address....................: 2001:DB8:ACAD:1:201:43FF:FE06:961
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
```

Адрес сформирован с помощью алгоритма EUI-64

#### Настройка и проверка Stateless DHCPv6 сервера на маршрутизаторе R1

Рассмотрим конфигурацию ipv6 на PC-A более подробно

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0001.4306.0961
   Link-local IPv6 Address.........: FE80::201:43FF:FE06:961
   IPv6 Address....................: 2001:DB8:ACAD:1:201:43FF:FE06:961
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-EE-91-06-6B-00-01-43-06-09-61
   DNS Servers.....................: ::
                                     0.0.0.0
```

**Настройка маршрутизатора R1 для предоставления параметров Stateless DHCPv6 для PC-A**

Создание пула DHCPv6 R1-STATELESS. Привязка имени домена и адреса dns-сервера как часть настройки пула

```
ipv6 dhcp pool R1-STATELESS
dns-server 2001:db8:acad::254
domain-name STATELESS.com
```

Настройка интерфейса G0/0/1 на R1 для выставления флага OTHER и определения пула DHCPv6 для этого интерфейса

```
interface g0/0/1
ipv6 nd other-config-flag
ipv6 dhcp server R1-STATELESS
```

После перезагрузки интерфейса на PC-A проверим, получаются ли указанные настройки

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: STATELESS.com 
   Physical Address................: 0001.4306.0961
   Link-local IPv6 Address.........: FE80::201:43FF:FE06:961
   IPv6 Address....................: 2001:DB8:ACAD:1:201:43FF:FE06:961
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 2137386666
   DHCPv6 Client DUID..............: 00-01-00-01-EE-91-06-6B-00-01-43-06-09-61
   DNS Servers.....................: 2001:DB8:ACAD::254
                                     0.0.0.0
```

Имя домена и DNS-сервер получены клиентом PC-A.

#### Настройка и проверка Stateful DHCPv6 сервера на маршрутизаторе R1

Создание DHCPv6 пула на маршрутизаторе R1для сети 2001:db8:acad:3:aaaa::/80. Этот пул будет выдавать адреса для клиентов за интерфейсом G0/0/1 маршрутизатора R2. Также пул будет выдавать имя домена STATEFUL.COM и DNS-сервер 2001:db8:acad::254

Создание пула:

```
ipv6 dhcp pool R2-STATEFUL
address prefix 2001:db8:acad:3:aaa::/80
dns-server 2001:db8:acad::254
domain-name STATEFUL.com
```

Назначение созданного пула интерфейсу G0/0/0 маршрутизатора R1

```
interface g0/0/0
ipv6 dhcp server R2-STATEFUL
```

#### Настройка и проверка DHCPv6 Relay на маршрутизаторе R2

Проверка SLAAC-адреса на PC-B

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0006.2ABA.CDD5
   Link-local IPv6 Address.........: FE80::206:2AFF:FEBA:CDD5
   IPv6 Address....................: 2001:DB8:ACAD:3:206:2AFF:FEBA:CDD5
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-27-32-97-9E-00-06-2A-BA-CD-D5
   DNS Servers.....................: ::
                                     0.0.0.0
```

**Настройка маршрутизатора R2 как агента DHCPv6 Relay на интерфейсе G0/0/1**

Конфигурирование DHCPv6 Relay и добавление флага managed-confog-flag (для настройки Stateful DHCPv6)

```
interface g0/0/1
ipv6 nd managed-config-flag
ipv6 dhcp relay destination 2001:db8:acad:2::1 g0/0/0
```

После перезагрузки интерфейса на PC-B:

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: STATEFUL.com
   Physical Address................: 0006.2ABA.CDD5
   Link-local IPv6 Address.........: FE80::206:2AFF:FEBA:CDD5
   IPv6 Address....................: 2001:DB8:ACAD:3:AAAA:2AFF:FEBA:CDD5
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-27-32-97-9E-00-06-2A-BA-CD-D5
   DNS Servers.....................: 2001:DB8:ACAD::254
                                     0.0.0.0
```



