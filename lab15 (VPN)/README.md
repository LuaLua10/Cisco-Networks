Лабораторная работа. VPN.
---------

Топология
---------

![](media/0735678569f8a389567856788c4e11e.png)

Задачи
---------

VPN. GRE. DmVPN
Настроить GRE между офисами Москва и С.-Петербург Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги
1. Настроите GRE между офисами Москва и С.-Петербург
2. Настроите DMVMN между Москва и Чокурдах, Лабытнанги
3. Все узлы в офисах в лабораторной работе должны иметь IP связность
4. План работы и изменения зафиксированы в документации

Решение
---------

#### Настроим GRE между офисами Москва и С.-Петербург

##### Конфигурация R15:

```
interface Tunnel1
 ip address 192.168.100.1 255.255.255.0
 ip mtu 1440
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/2
 tunnel destination 52.0.0.18
```

##### Конфигурация R18:

```
interface Tunnel1
 ip address 192.168.100.2 255.255.255.0
 ip mtu 1440
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/2
 tunnel destination 30.1.0.2
```

##### Проверим работу туннеля:

```
R18#ping 192.168.100.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.100.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

#### Настроите DMVMN между Москва и Чокурдах, Лабытнанги

> Маршрутизатор R14 - HUB, R27 и R28 - Spoke.

##### Конфигурация R14:

```
interface Tunnel1
 ip address 192.168.200.1 255.255.255.0
 no ip redirects
 ip mtu 1416
 ip nhrp authentication Test
 ip nhrp map multicast dynamic
 ip nhrp network-id 1001
 ip ospf network broadcast
 ip ospf hello-interval 30
 ip ospf priority 10
 tunnel source 101.0.0.2
 tunnel mode gre multipoint
 tunnel key 123
router ospf 1
 network 192.168.200.0 0.0.0.255 area 20
 ```

##### Сводная информацию по туннелям R14:

```
R14#show ip nhrp brief 
   Target             Via            NBMA           Mode   Intfc   Claimed 
192.168.200.2/32     192.168.200.2   52.0.0.34       dynamic  Tu1     <   >
192.168.200.3/32     192.168.200.3   52.0.0.26       dynamic  Tu1     <   >
```

##### Конфигурация R27:

```
interface Tunnel1
 ip address 192.168.200.2 255.255.255.0
 ip mtu 1416
 ip nhrp authentication Test
 ip nhrp map 192.168.200.1 101.0.0.2
 ip nhrp map multicast 101.0.0.2
 ip nhrp network-id 1001
 ip nhrp nhs 192.168.200.1
 ip nhrp registration no-unique
 tunnel source 52.0.0.34
 tunnel destination 101.0.0.2
 tunnel key 123
```

##### Сводная информацию по туннелям R27:

```
R27#show ip nhrp brief 
   Target             Via            NBMA           Mode   Intfc   Claimed 
192.168.200.1/32     192.168.200.1   101.0.0.2       static   Tu1     <   >
```

##### Конфигурация R28:

```
interface Loopback1
 ip address 28.28.28.28 255.255.255.255
 ip nat outside
 ip virtual-reassembly in
 ip policy route-map PBR1
interface Tunnel1
 ip address 192.168.200.3 255.255.255.0
 ip mtu 1416
 ip nhrp authentication Test
 ip nhrp map 192.168.200.1 101.0.0.2
 ip nhrp map multicast 101.0.0.2
 ip nhrp network-id 1001
 ip nhrp nhs 192.168.200.1
 ip nhrp registration no-unique
 ip ospf network broadcast
 ip ospf hello-interval 30
 ip ospf priority 0
 tunnel source Loop1
 tunnel destination 101.0.0.2
 tunnel key 123
router ospf 1
 network 192.168.200.0 0.0.0.255 area 20
```

##### Сводная информацию по туннелям R28:

```
R28#show ip nhrp brief 
   Target             Via            NBMA           Mode   Intfc   Claimed 
192.168.200.1/32     192.168.200.1   101.0.0.2       static   Tu1     <   >
```
