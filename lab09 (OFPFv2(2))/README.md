Лабораторная работа. OSPFv2. 
---------

Топология
---------

![](media/073df55cf8a389967d537a5c28c4e176e.png)

Задачи
---------
Настроить OSPF офисе Москва. Разделить сеть на зоны. Настроить фильтрацию между зонами.
1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone;
2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по-умолчанию;
3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию;
4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101;
5. План работы и изменения зафиксированы в документации.

```
Примечание: так как согластно выбранной архитектуре сети, коммутаторы SW4 и SW5 будут участвовать в маршрутизации,
на них будет запущен процесс OSPF. Коммутаторы будут находится в Area 10.
```

Решение
---------

#### Настроим OSPF на маршрутизаторах R14 и R15

##### Конфигурация R15:
 
```
router ospf 1
 router-id 15.15.15.15
 area 10 stub
 area 102 filter-list prefix DEAD_area101 in
 passive-interface Ethernet0/2
 network 15.15.15.15 0.0.0.0 area 0
 network 100.1.0.12 0.0.0.3 area 10
 network 100.1.0.16 0.0.0.3 area 10
 network 100.1.0.20 0.0.0.3 area 102
 network 100.1.0.40 0.0.0.3 area 0
```

##### Таблица маршрутизации R15:

```
R15#show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      14.0.0.0/32 is subnetted, 1 subnets
O        14.14.14.14 [110/11] via 100.1.0.41, 13:16:44, Ethernet1/0
      100.0.0.0/8 is variably subnetted, 19 subnets, 5 masks
O IA     100.1.0.0/30 [110/20] via 100.1.0.41, 13:16:44, Ethernet1/0
O        100.1.0.4/30 [110/20] via 100.1.0.14, 13:16:39, Ethernet0/1
O        100.1.0.8/30 [110/20] via 100.1.0.18, 13:16:39, Ethernet0/0
O        100.1.0.24/30 [110/20] via 100.1.0.14, 13:16:44, Ethernet0/1
O        100.1.0.28/30 [110/20] via 100.1.0.14, 13:16:44, Ethernet0/1
O        100.1.0.32/30 [110/20] via 100.1.0.18, 13:16:44, Ethernet0/0
O        100.1.0.36/30 [110/20] via 100.1.0.18, 13:16:44, Ethernet0/0
O        100.1.0.64/26 [110/21] via 100.1.0.18, 13:16:39, Ethernet0/0
                       [110/21] via 100.1.0.14, 13:16:34, Ethernet0/1
O        100.1.1.0/24 [110/21] via 100.1.0.18, 13:16:39, Ethernet0/0
                      [110/21] via 100.1.0.14, 13:16:34, Ethernet0/1
O        100.1.2.0/24 [110/21] via 100.1.0.18, 13:16:39, Ethernet0/0
                      [110/21] via 100.1.0.14, 13:16:34, Ethernet0/1
```

##### Конфигурация R14:

```
router ospf 1
 router-id 14.14.14.14
 area 10 stub
 area 101 stub no-summary
 passive-interface Ethernet0/2
 network 14.14.14.14 0.0.0.0 area 0
 network 100.1.0.0 0.0.0.3 area 101
 network 100.1.0.4 0.0.0.3 area 10
 network 100.1.0.8 0.0.0.3 area 10
 network 100.1.0.40 0.0.0.3 area 0
```


##### Таблица маршрутизации R14:

```
R14#show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      15.0.0.0/32 is subnetted, 1 subnets
O        15.15.15.15 [110/11] via 100.1.0.42, 13:09:00, Ethernet1/0
      100.0.0.0/8 is variably subnetted, 19 subnets, 5 masks
O        100.1.0.12/30 [110/20] via 100.1.0.6, 13:08:55, Ethernet0/0
O        100.1.0.16/30 [110/20] via 100.1.0.10, 13:08:55, Ethernet0/1
O IA     100.1.0.20/30 [110/20] via 100.1.0.42, 13:09:00, Ethernet1/0
O        100.1.0.24/30 [110/20] via 100.1.0.6, 13:08:55, Ethernet0/0
O        100.1.0.28/30 [110/20] via 100.1.0.6, 13:08:55, Ethernet0/0
O        100.1.0.32/30 [110/20] via 100.1.0.10, 13:08:55, Ethernet0/1
O        100.1.0.36/30 [110/20] via 100.1.0.10, 13:08:55, Ethernet0/1
O        100.1.0.64/26 [110/21] via 100.1.0.10, 13:08:55, Ethernet0/1
                       [110/21] via 100.1.0.6, 13:08:50, Ethernet0/0
O        100.1.1.0/24 [110/21] via 100.1.0.10, 13:08:55, Ethernet0/1
                      [110/21] via 100.1.0.6, 13:08:50, Ethernet0/0
O        100.1.2.0/24 [110/21] via 100.1.0.10, 13:08:55, Ethernet0/1
                      [110/21] via 100.1.0.6, 13:08:50, Ethernet0/0
```

#### Настроим OSPF на маршрутизаторах R12 и R13

##### Конфигурация R12:

```
router ospf 1
 router-id 12.12.12.12
 area 10 stub
 network 100.1.0.4 0.0.0.3 area 10
 network 100.1.0.12 0.0.0.3 area 10
 network 100.1.0.24 0.0.0.3 area 10
 network 100.1.0.28 0.0.0.3 area 10
```

##### Таблица маршрутизации R12:

```
R12#show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 100.1.0.13 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 100.1.0.13, 13:17:43, Ethernet0/3
                [110/11] via 100.1.0.5, 13:17:38, Ethernet0/2
      14.0.0.0/32 is subnetted, 1 subnets
O IA     14.14.14.14 [110/11] via 100.1.0.5, 13:17:38, Ethernet0/2
      15.0.0.0/32 is subnetted, 1 subnets
O IA     15.15.15.15 [110/11] via 100.1.0.13, 13:17:43, Ethernet0/3
      100.0.0.0/8 is variably subnetted, 18 subnets, 4 masks
O IA     100.1.0.0/30 [110/20] via 100.1.0.5, 13:17:38, Ethernet0/2
O        100.1.0.8/30 [110/20] via 100.1.0.5, 13:17:38, Ethernet0/2
O        100.1.0.16/30 [110/20] via 100.1.0.13, 13:17:43, Ethernet0/3
O IA     100.1.0.20/30 [110/20] via 100.1.0.13, 13:17:43, Ethernet0/3
O        100.1.0.32/30 [110/20] via 100.1.0.26, 13:17:33, Ethernet0/0
O        100.1.0.36/30 [110/20] via 100.1.0.30, 13:17:33, Ethernet0/1
O IA     100.1.0.40/30 [110/20] via 100.1.0.13, 13:17:43, Ethernet0/3
                       [110/20] via 100.1.0.5, 13:17:38, Ethernet0/2
O        100.1.0.64/26 [110/11] via 100.1.0.30, 13:17:33, Ethernet0/1
                       [110/11] via 100.1.0.26, 13:17:33, Ethernet0/0
O        100.1.1.0/24 [110/11] via 100.1.0.30, 13:17:33, Ethernet0/1
                      [110/11] via 100.1.0.26, 13:17:33, Ethernet0/0
O        100.1.2.0/24 [110/11] via 100.1.0.30, 13:17:33, Ethernet0/1
                      [110/11] via 100.1.0.26, 13:17:33, Ethernet0/0
```

##### Конфигурация R13:

```
router ospf 1
 router-id 13.13.13.13
 area 10 stub
 network 100.1.0.8 0.0.0.3 area 10
 network 100.1.0.16 0.0.0.3 area 10
 network 100.1.0.32 0.0.0.3 area 10
 network 100.1.0.36 0.0.0.3 area 10
```

##### Таблица маршрутизации R13:

```
R13#show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 100.1.0.17 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 100.1.0.17, 13:18:21, Ethernet0/2
                [110/11] via 100.1.0.9, 13:18:16, Ethernet0/3
      14.0.0.0/32 is subnetted, 1 subnets
O IA     14.14.14.14 [110/11] via 100.1.0.9, 13:18:16, Ethernet0/3
      15.0.0.0/32 is subnetted, 1 subnets
O IA     15.15.15.15 [110/11] via 100.1.0.17, 13:18:21, Ethernet0/2
      100.0.0.0/8 is variably subnetted, 18 subnets, 4 masks
O IA     100.1.0.0/30 [110/20] via 100.1.0.9, 13:18:16, Ethernet0/3
O        100.1.0.4/30 [110/20] via 100.1.0.9, 13:18:16, Ethernet0/3
O        100.1.0.12/30 [110/20] via 100.1.0.17, 13:18:21, Ethernet0/2
O IA     100.1.0.20/30 [110/20] via 100.1.0.17, 13:18:21, Ethernet0/2
O        100.1.0.24/30 [110/20] via 100.1.0.34, 13:18:16, Ethernet0/1
O        100.1.0.28/30 [110/20] via 100.1.0.38, 13:18:16, Ethernet0/0
O IA     100.1.0.40/30 [110/20] via 100.1.0.17, 13:18:21, Ethernet0/2
                       [110/20] via 100.1.0.9, 13:18:16, Ethernet0/3
O        100.1.0.64/26 [110/11] via 100.1.0.38, 13:18:16, Ethernet0/0
                       [110/11] via 100.1.0.34, 13:18:16, Ethernet0/1
O        100.1.1.0/24 [110/11] via 100.1.0.38, 13:18:16, Ethernet0/0
                      [110/11] via 100.1.0.34, 13:18:16, Ethernet0/1
O        100.1.2.0/24 [110/11] via 100.1.0.38, 13:18:16, Ethernet0/0
                      [110/11] via 100.1.0.34, 13:18:16, Ethernet0/1
```

#### Настроим OSPF на L3 коммутаторах SW4 и SW5

##### Конфигурация SW4:

```
router ospf 1
 router-id 4.4.4.4
 area 10 stub
 passive-interface default
 no passive-interface Ethernet1/0
 no passive-interface Ethernet1/1
 network 100.1.0.24 0.0.0.3 area 10
 network 100.1.0.32 0.0.0.3 area 10
 network 100.1.0.64 0.0.0.63 area 10
 network 100.1.1.0 0.0.0.255 area 10
 network 100.1.2.0 0.0.0.255 area 10
```

##### Таблица маршрутизации SW4:

    
```
SW4#show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 100.1.0.33 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/21] via 100.1.0.33, 13:18:46, Ethernet1/1
                [110/21] via 100.1.0.25, 13:18:36, Ethernet1/0
      14.0.0.0/32 is subnetted, 1 subnets
O IA     14.14.14.14 [110/21] via 100.1.0.33, 13:18:46, Ethernet1/1
                     [110/21] via 100.1.0.25, 13:18:36, Ethernet1/0
      15.0.0.0/32 is subnetted, 1 subnets
O IA     15.15.15.15 [110/21] via 100.1.0.33, 13:18:46, Ethernet1/1
                     [110/21] via 100.1.0.25, 13:18:36, Ethernet1/0
      100.0.0.0/8 is variably subnetted, 19 subnets, 4 masks
O IA     100.1.0.0/30 [110/30] via 100.1.0.33, 13:18:46, Ethernet1/1
                      [110/30] via 100.1.0.25, 13:18:36, Ethernet1/0
O        100.1.0.4/30 [110/20] via 100.1.0.25, 13:18:36, Ethernet1/0
O        100.1.0.8/30 [110/20] via 100.1.0.33, 13:18:46, Ethernet1/1
O        100.1.0.12/30 [110/20] via 100.1.0.25, 13:18:36, Ethernet1/0
O        100.1.0.16/30 [110/20] via 100.1.0.33, 13:18:46, Ethernet1/1
O IA     100.1.0.20/30 [110/30] via 100.1.0.33, 13:18:46, Ethernet1/1
                       [110/30] via 100.1.0.25, 13:18:36, Ethernet1/0
O        100.1.0.28/30 [110/20] via 100.1.0.25, 13:18:36, Ethernet1/0
O        100.1.0.36/30 [110/20] via 100.1.0.33, 13:18:46, Ethernet1/1
O IA     100.1.0.40/30 [110/30] via 100.1.0.33, 13:18:46, Ethernet1/1
                       [110/30] via 100.1.0.25, 13:18:36, Ethernet1/0
```

##### Конфигурация SW5:

```
router ospf 1
 router-id 5.5.5.5
 area 10 stub
 passive-interface default
 no passive-interface Ethernet1/0
 no passive-interface Ethernet1/1
 network 100.1.0.28 0.0.0.3 area 10
 network 100.1.0.36 0.0.0.3 area 10
 network 100.1.0.64 0.0.0.63 area 10
 network 100.1.1.0 0.0.0.255 area 10
 network 100.1.2.0 0.0.0.255 area 10
```

##### Таблица маршрутизации SW5:

```
SW5#show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 100.1.0.37 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/21] via 100.1.0.37, 13:19:24, Ethernet1/0
                [110/21] via 100.1.0.29, 13:19:14, Ethernet1/1
      14.0.0.0/32 is subnetted, 1 subnets
O IA     14.14.14.14 [110/21] via 100.1.0.37, 13:19:24, Ethernet1/0
                     [110/21] via 100.1.0.29, 13:19:14, Ethernet1/1
      15.0.0.0/32 is subnetted, 1 subnets
O IA     15.15.15.15 [110/21] via 100.1.0.37, 13:19:24, Ethernet1/0
                     [110/21] via 100.1.0.29, 13:19:14, Ethernet1/1
      100.0.0.0/8 is variably subnetted, 19 subnets, 4 masks
O IA     100.1.0.0/30 [110/30] via 100.1.0.37, 13:19:24, Ethernet1/0
                      [110/30] via 100.1.0.29, 13:19:14, Ethernet1/1
O        100.1.0.4/30 [110/20] via 100.1.0.29, 13:19:14, Ethernet1/1
O        100.1.0.8/30 [110/20] via 100.1.0.37, 13:19:24, Ethernet1/0
O        100.1.0.12/30 [110/20] via 100.1.0.29, 13:19:14, Ethernet1/1
O        100.1.0.16/30 [110/20] via 100.1.0.37, 13:19:24, Ethernet1/0
O IA     100.1.0.20/30 [110/30] via 100.1.0.37, 13:19:24, Ethernet1/0
                       [110/30] via 100.1.0.29, 13:19:14, Ethernet1/1
O        100.1.0.24/30 [110/20] via 100.1.0.29, 13:19:14, Ethernet1/1
O        100.1.0.32/30 [110/20] via 100.1.0.37, 13:19:24, Ethernet1/0
O IA     100.1.0.40/30 [110/30] via 100.1.0.37, 13:19:24, Ethernet1/0
                       [110/30] via 100.1.0.29, 13:19:14, Ethernet1/1
```

#### Настроим OSPF на маршрутизаторах R19 и R20

##### Конфигурация R19:

```
router ospf 1
 router-id 19.19.19.19
 area 101 stub
 network 100.1.0.0 0.0.0.3 area 101
```

##### Таблица маршрутизации R19:

```
R19#show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 100.1.0.1 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 100.1.0.1, 01:42:28, Ethernet0/0
```

##### Конфигурация R20:

```
router ospf 1
 router-id 20.20.20.20
 network 100.1.0.20 0.0.0.3 area 102
```

##### Таблица маршрутизации R20:

```
R20#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      100.0.0.0/8 is variably subnetted, 13 subnets, 4 masks
O IA     100.1.0.4/30 [110/30] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.8/30 [110/30] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.12/30 [110/20] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.16/30 [110/20] via 100.1.0.21, 01:21:57, Ethernet0/0
C        100.1.0.20/30 is directly connected, Ethernet0/0
L        100.1.0.22/32 is directly connected, Ethernet0/0
O IA     100.1.0.24/30 [110/30] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.28/30 [110/30] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.32/30 [110/30] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.36/30 [110/30] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.64/26 [110/31] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.1.0/24 [110/31] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.2.0/24 [110/31] via 100.1.0.21, 01:21:57, Ethernet0/0
```



