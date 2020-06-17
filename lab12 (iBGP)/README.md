Лабораторная работа. iBGP.
---------

Топология
---------

![](media/073df55cf8a383467346537a5c28c4e11e.png)

Задачи
---------

Настроить iBGP в офисе Москва Настроить iBGP в сети провайдера Триада Организовать полную IP связанность всех сетей
1. iBGP в офисом Москва между маршрутизаторами R14 и R15
2. Настроите iBGP в провайдере Триада
3. Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас.
4. В офисе С.-Петербург работает протокол iBGP. (Не использовать протокол OSPF)
5. Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно
6. Все сети в лабораторной работе должны иметь IP связность
7. План работы и изменения зафиксированы в документации

Решение
---------

#### Настроим iBGP на маршрутизаторах AS 1001 (R14/15)

##### Конфигурация R14:

```
interface Loopback0
 ip address 14.14.14.14 255.255.255.255
 ipv6 address FE80::1:14 link-local
 ipv6 address 2001:AAAA:1001:14::1/128
 ipv6 ospf 10 area 0
router ospf 1
 network 14.14.14.14 0.0.0.0 area 0
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 network 100.1.0.0 mask 255.255.252.0
 neighbor 15.15.15.15 remote-as 1001
 neighbor 15.15.15.15 update-source Loopback0
 neighbor 15.15.15.15 next-hop-self
 neighbor 101.0.0.1 remote-as 101
 neighbor 101.0.0.1 route-map AS_PREP_AS101 out
ip route 100.1.0.0 255.255.252.0 Null0
route-map AS_PREP_AS101 permit 10
 set as-path prepend 1001 1001 1001 1001
```

##### Таблица маршрутизации R14:

```
R14#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      20.0.0.0/22 is subnetted, 1 subnets
B        20.42.0.0 [20/0] via 101.0.0.1, 2d02h
      30.0.0.0/28 is subnetted, 1 subnets
B        30.1.0.0 [200/0] via 15.15.15.15, 23:06:52
      52.0.0.0/26 is subnetted, 1 subnets
B        52.0.0.0 [20/0] via 101.0.0.1, 2d02h
      101.0.0.0/8 is variably subnetted, 3 subnets, 3 masks
B        101.0.0.0/28 [20/0] via 101.0.0.1, 2d02h
```

##### Таблица BGP R22 (в результате работы AS Prepend маршрут до сети 100.1.0.0/22 (AS1001) идет через AS301)

```
R22#show ip bgp
BGP table version is 16, local router ID is 22.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   20.42.0.0/22     101.0.0.6                              0 301 520 2042 i
 *>                   101.0.0.10                             0 520 2042 i
 *   30.1.0.0/28      101.0.0.2                              0 1001 1001 1001 1001 1001 301 i
 *                    101.0.0.10                             0 520 301 i
 *>                   101.0.0.6                0             0 301 i
 *   52.0.0.0/26      101.0.0.6                              0 301 520 i
 *>                   101.0.0.10               0             0 520 i
 *   100.1.0.0/22     101.0.0.10                             0 520 301 1001 i
 *>                   101.0.0.6                              0 301 1001 i
 *                    101.0.0.2                0             0 1001 1001 1001 1001 1001 i
 *>  101.0.0.0/28     0.0.0.0                  0         32768 i
```

##### Конфигурация R15:

```
interface Loopback0
 ip address 15.15.15.15 255.255.255.255
 ipv6 address FE80::1:15 link-local
 ipv6 address 2001:AAAA:1001:15::1/128
 ipv6 ospf 10 area 0
router ospf 1
 network 15.15.15.15 0.0.0.0 area 0
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 network 100.1.0.0 mask 255.255.252.0
 neighbor 14.14.14.14 remote-as 1001
 neighbor 14.14.14.14 update-source Loopback0
 neighbor 14.14.14.14 next-hop-self
 neighbor 30.1.0.1 remote-as 301
ip route 100.1.0.0 255.255.252.0 Null0
ip prefix-list DEAD_area101 seq 5 deny 100.1.0.0/30
ip prefix-list DEAD_area101 seq 10 permit 0.0.0.0/0 le 32
ipv6 prefix-list DEAD_area101v6 seq 5 deny 2001:AAAA:1001:1::/64
ipv6 prefix-list DEAD_area101v6 seq 10 permit ::/0 le 128
```

##### Таблица маршрутизации R15:

```
R15#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      20.0.0.0/22 is subnetted, 1 subnets
B        20.42.0.0 [20/0] via 30.1.0.1, 23:06:18
      30.0.0.0/8 is variably subnetted, 3 subnets, 3 masks
B        30.1.0.0/28 [20/0] via 30.1.0.1, 23:06:18
      52.0.0.0/26 is subnetted, 1 subnets
B        52.0.0.0 [20/0] via 30.1.0.1, 23:06:18
      101.0.0.0/28 is subnetted, 1 subnets
B        101.0.0.0 [200/0] via 14.14.14.14, 23:05:32
```

#### Настроите iBGP в провайдере Триада (R23-R26)

> Для провайдера Триада параллельно настроим IGP (OSPF)
> R23 будит выступать в роле route-reflector

##### Конфигурация R23:

```
interface Loopback0
 ip address 23.23.23.23 255.255.255.255
router ospf 1
 router-id 23.23.23.23
 passive-interface default
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
 network 23.23.23.23 0.0.0.0 area 0
 network 52.0.0.0 0.0.0.3 area 0
 network 52.0.0.12 0.0.0.3 area 0
router bgp 520
 bgp log-neighbor-changes
 network 52.0.0.0 mask 255.255.255.192
 neighbor AS520 peer-group
 neighbor AS520 remote-as 520
 neighbor AS520 update-source Loopback0
 neighbor AS520 route-reflector-client
 neighbor AS520 next-hop-self
 neighbor 24.24.24.24 peer-group AS520
 neighbor 25.25.25.25 peer-group AS520
 neighbor 26.26.26.26 peer-group AS520
 neighbor 101.0.0.9 remote-as 101
ip route 52.0.0.0 255.255.255.192 Null0
```

##### Таблица маршрутизации R23:

```
R23#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      20.0.0.0/22 is subnetted, 1 subnets
B        20.42.0.0 [200/0] via 24.24.24.24, 2d02h
      30.0.0.0/28 is subnetted, 1 subnets
B        30.1.0.0 [200/0] via 24.24.24.24, 2d03h
      100.0.0.0/22 is subnetted, 1 subnets
B        100.1.0.0 [200/0] via 24.24.24.24, 2d02h
      101.0.0.0/8 is variably subnetted, 3 subnets, 3 masks
B        101.0.0.0/28 [20/0] via 101.0.0.9, 2d03h
```

##### Конфигурация R24:

```
interface Loopback0
 ip address 24.24.24.24 255.255.255.255
router ospf 1
 router-id 24.24.24.24
 passive-interface default
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
 network 24.24.24.24 0.0.0.0 area 0
 network 52.0.0.0 0.0.0.3 area 0
 network 52.0.0.4 0.0.0.3 area 0
 network 52.0.0.16 0.0.0.3 area 0
router bgp 520
 bgp router-id 24.24.24.24
 bgp log-neighbor-changes
 network 52.0.0.0 mask 255.255.255.192
 neighbor 23.23.23.23 remote-as 520
 neighbor 23.23.23.23 update-source Loopback0
 neighbor 23.23.23.23 next-hop-self
 neighbor 30.1.0.5 remote-as 301
 neighbor 52.0.0.18 remote-as 2042
ip route 52.0.0.0 255.255.255.192 Null0
```

##### Таблица маршрутизации R24:

```
R24#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      20.0.0.0/22 is subnetted, 1 subnets
B        20.42.0.0 [20/0] via 52.0.0.18, 2d02h
      30.0.0.0/8 is variably subnetted, 3 subnets, 3 masks
B        30.1.0.0/28 [20/0] via 30.1.0.5, 2d03h
      100.0.0.0/22 is subnetted, 1 subnets
B        100.1.0.0 [20/0] via 30.1.0.5, 2d02h
      101.0.0.0/28 is subnetted, 1 subnets
B        101.0.0.0 [200/0] via 23.23.23.23, 2d03h
```

##### Конфигурация R25:

```
interface Loopback0
 ip address 25.25.25.25 255.255.255.255
router ospf 1
 router-id 25.25.25.25
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/2
 network 25.25.25.25 0.0.0.0 area 0
 network 52.0.0.8 0.0.0.3 area 0
 network 52.0.0.12 0.0.0.3 area 0
 network 52.0.0.28 0.0.0.3 area 0
 network 52.0.0.32 0.0.0.3 area 0
router bgp 520
 bgp router-id 25.25.25.25
 bgp log-neighbor-changes
 network 52.0.0.0 mask 255.255.255.192
 neighbor 23.23.23.23 remote-as 520
 neighbor 23.23.23.23 update-source Loopback0
 neighbor 23.23.23.23 next-hop-self
ip route 52.0.0.0 255.255.255.192 Null0
```

##### Таблица маршрутизации R25:

```
R25#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      20.0.0.0/22 is subnetted, 1 subnets
B        20.42.0.0 [200/0] via 24.24.24.24, 2d02h
      30.0.0.0/28 is subnetted, 1 subnets
B        30.1.0.0 [200/0] via 24.24.24.24, 2d03h
      100.0.0.0/22 is subnetted, 1 subnets
B        100.1.0.0 [200/0] via 24.24.24.24, 2d02h
      101.0.0.0/28 is subnetted, 1 subnets
B        101.0.0.0 [200/0] via 23.23.23.23, 2d03h
```

##### Конфигурация R26:

```
interface Loopback0
 ip address 26.26.26.26 255.255.255.255
router ospf 520
 router-id 26.26.26.26
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/2
 network 26.26.26.26 0.0.0.0 area 0
 network 52.0.0.4 0.0.0.3 area 0
 network 52.0.0.8 0.0.0.3 area 0
 network 52.0.0.20 0.0.0.3 area 0
 network 52.0.0.24 0.0.0.3 area 0
router bgp 520
 bgp router-id 26.26.26.26
 bgp log-neighbor-changes
 network 52.0.0.0 mask 255.255.255.192
 neighbor 23.23.23.23 remote-as 520
 neighbor 23.23.23.23 update-source Loopback0
 neighbor 23.23.23.23 next-hop-self
 neighbor 52.0.0.22 remote-as 2042
ip route 52.0.0.0 255.255.255.192 Null0
```

##### Таблица маршрутизации R26:

```
R26#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      20.0.0.0/22 is subnetted, 1 subnets
B        20.42.0.0 [20/0] via 52.0.0.22, 00:02:30
      30.0.0.0/28 is subnetted, 1 subnets
B        30.1.0.0 [200/0] via 24.24.24.24, 00:02:30
      100.0.0.0/22 is subnetted, 1 subnets
B        100.1.0.0 [200/0] via 24.24.24.24, 00:02:30
      101.0.0.0/28 is subnetted, 1 subnets
B        101.0.0.0 [200/0] via 23.23.23.23, 00:02:30
```

#### Настроим iBGP на маршрутизаторах AS 2042 (R16-R18,R32)

> R18 будит выступать в роле route-reflector

> На R16 и R17 порты e0/0 и e0/2 обьединим в Bridge и созданим BVI интерфейсы

> На созданных BVI интерфейсах настроим протокол VRRP


##### Конфигурация R18:

```
interface Loopback0
 ip address 18.18.18.18 255.255.255.255
router bgp 2042
 bgp router-id 18.18.18.18
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 network 20.42.0.0 mask 255.255.252.0
 neighbor AS2042 peer-group
 neighbor AS2042 remote-as 2042
 neighbor AS2042 update-source Loopback0
 neighbor AS2042 route-reflector-client
 neighbor AS2042 next-hop-self
 neighbor 16.16.16.16 peer-group AS2042
 neighbor 17.17.17.17 peer-group AS2042
 neighbor 32.32.32.32 peer-group AS2042
 neighbor 52.0.0.17 remote-as 520
 neighbor 52.0.0.21 remote-as 520
 maximum-paths 2
ip route 16.16.16.16 255.255.255.255 20.42.0.6
ip route 17.17.17.17 255.255.255.255 20.42.0.2
ip route 20.42.0.0 255.255.252.0 Null0
ip route 20.42.0.8 255.255.255.252 20.42.0.6
ip route 32.32.32.32 255.255.255.255 20.42.0.6
```

##### Таблица маршрутизации R18:

```
R18#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      20.0.0.0/8 is variably subnetted, 9 subnets, 4 masks
B        20.42.1.0/24 [200/0] via 16.16.16.16, 04:47:54
B        20.42.2.0/24 [200/0] via 16.16.16.16, 04:47:54
B        20.42.3.0/24 [200/0] via 16.16.16.16, 04:47:54
      30.0.0.0/28 is subnetted, 1 subnets
B        30.1.0.0 [20/0] via 52.0.0.21, 3d01h
                  [20/0] via 52.0.0.17, 3d01h
      52.0.0.0/8 is variably subnetted, 5 subnets, 3 masks
B        52.0.0.0/26 [20/0] via 52.0.0.21, 3d01h
                     [20/0] via 52.0.0.17, 3d01h
      100.0.0.0/22 is subnetted, 1 subnets
B        100.1.0.0 [20/0] via 52.0.0.21, 3d01h
                   [20/0] via 52.0.0.17, 3d01h
      101.0.0.0/28 is subnetted, 1 subnets
B        101.0.0.0 [20/0] via 52.0.0.21, 3d01h
                   [20/0] via 52.0.0.17, 3d01h
```

##### Конфигурация R17:

```
bridge irb

interface Loopback0
 ip address 17.17.17.17 255.255.255.255

interface Ethernet0/0.5
 description Manag
 encapsulation dot1Q 5
 bridge-group 1

interface Ethernet0/0.10
 description VID10
 encapsulation dot1Q 10
 bridge-group 2

interface Ethernet0/0.11
 description VID11
 encapsulation dot1Q 11
 bridge-group 3

interface Ethernet0/1
 ip address 20.42.0.2 255.255.255.252
 ipv6 address FE80::17 link-local
 ipv6 address 2001:AAAA:2042:1::2/64

interface Ethernet0/2.5
 description Manag
 encapsulation dot1Q 5
 bridge-group 1

interface Ethernet0/2.10
 description VID10
 encapsulation dot1Q 10
 bridge-group 2

interface Ethernet0/2.11
 description VID11
 encapsulation dot1Q 11
 bridge-group 3

interface BVI1
 description Manag
 ip address 20.42.1.1 255.255.255.0
 ipv6 address FE80::17 link-local
 ipv6 address 2001:AAAA:2042:4::1/64
 vrrp 3 description VRRP_Manag
 vrrp 3 ip 20.42.1.1
 vrrp 3 authentication cisco

interface BVI2
 description VID10
 ip address 20.42.2.1 255.255.255.0
 ipv6 address FE80::17 link-local
 ipv6 address 2001:AAAA:2042:5::1/64
 vrrp 1 description VRRP_VID10
 vrrp 1 ip 20.42.2.1
 vrrp 1 authentication cisco

interface BVI3
 description VID11
 ip address 20.42.3.2 255.255.255.0
 ipv6 address FE80::17 link-local
 ipv6 address 2001:AAAA:2042:6::1/64
 vrrp 2 description VRRP_VID11
 vrrp 2 ip 20.42.3.1
 vrrp 2 authentication cisco

router bgp 2042
 bgp router-id 17.17.17.17
 bgp log-neighbor-changes
 network 20.42.0.0 mask 255.255.252.0
 network 20.42.1.0 mask 255.255.255.0
 network 20.42.2.0 mask 255.255.255.0
 network 20.42.3.0 mask 255.255.255.0
 neighbor 18.18.18.18 remote-as 2042
 neighbor 18.18.18.18 update-source Loopback0
 neighbor 18.18.18.18 next-hop-self

ip route 16.16.16.16 255.255.255.255 20.42.0.1
ip route 18.18.18.18 255.255.255.255 20.42.0.1
ip route 20.42.0.0 255.255.252.0 Null0
ip route 32.32.32.32 255.255.255.255 20.42.0.1

bridge 1 protocol ieee
bridge 1 route ip
bridge 2 protocol ieee
bridge 2 route ip
bridge 3 protocol ieee
bridge 3 route ip
```

##### Таблица маршрутизации R17:

```
R17#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      16.0.0.0/32 is subnetted, 1 subnets
S        16.16.16.16 [1/0] via 20.42.0.1
      17.0.0.0/32 is subnetted, 1 subnets
C        17.17.17.17 is directly connected, Loopback0
      18.0.0.0/32 is subnetted, 1 subnets
S        18.18.18.18 [1/0] via 20.42.0.1
      20.0.0.0/8 is variably subnetted, 9 subnets, 4 masks
S        20.42.0.0/22 is directly connected, Null0
C        20.42.0.0/30 is directly connected, Ethernet0/1
L        20.42.0.2/32 is directly connected, Ethernet0/1
C        20.42.1.0/24 is directly connected, BVI1
L        20.42.1.1/32 is directly connected, BVI1
C        20.42.2.0/24 is directly connected, BVI2
L        20.42.2.1/32 is directly connected, BVI2
C        20.42.3.0/24 is directly connected, BVI3
L        20.42.3.2/32 is directly connected, BVI3
      30.0.0.0/28 is subnetted, 1 subnets
B        30.1.0.0 [200/0] via 18.18.18.18, 00:08:06
      32.0.0.0/32 is subnetted, 1 subnets
S        32.32.32.32 [1/0] via 20.42.0.1
      52.0.0.0/26 is subnetted, 1 subnets
B        52.0.0.0 [200/0] via 18.18.18.18, 00:08:06
      100.0.0.0/22 is subnetted, 1 subnets
B        100.1.0.0 [200/0] via 18.18.18.18, 00:08:06
      101.0.0.0/28 is subnetted, 1 subnets
B        101.0.0.0 [200/0] via 18.18.18.18, 00:08:06
```

##### Конфигурация R16:

```
bridge irb

interface Loopback0
 ip address 16.16.16.16 255.255.255.255

interface Ethernet0/0.5
 description Manag
 encapsulation dot1Q 5
 bridge-group 1

interface Ethernet0/0.10
 description VID10
 encapsulation dot1Q 10
 bridge-group 2

interface Ethernet0/0.11
 description VID11
 encapsulation dot1Q 11
 bridge-group 3

interface Ethernet0/1
 ip address 20.42.0.6 255.255.255.252
 ipv6 address FE80::16 link-local
 ipv6 address 2001:AAAA:2042:2::2/64

interface Ethernet0/2.5
 description Manag
 encapsulation dot1Q 5
 bridge-group 1

interface Ethernet0/2.10
 description VID10
 encapsulation dot1Q 10
 bridge-group 2

interface Ethernet0/2.11
 description VID11
 encapsulation dot1Q 11
 bridge-group 3

interface Ethernet0/3
 ip address 20.42.0.9 255.255.255.252
 ipv6 address FE80::16 link-local
 ipv6 address 2001:AAAA:2042:3::1/64

interface BVI1
 description Manag
 ip address 20.42.1.2 255.255.255.0
 ipv6 address FE80::16 link-local
 ipv6 address 2001:AAAA:2042:4::2/64
 vrrp 3 description VRRP_Manag
 vrrp 3 ip 20.42.1.1
 vrrp 3 authentication cisco

interface BVI2
 description VID10
 ip address 20.42.2.2 255.255.255.0
 ipv6 address FE80::16 link-local
 ipv6 address 2001:AAAA:2042:5::2/64
 vrrp 1 description VRRP_VID10
 vrrp 1 ip 20.42.2.1
 vrrp 1 authentication cisco

interface BVI3
 description VID11
 ip address 20.42.3.1 255.255.255.0
 ipv6 address FE80::16 link-local
 ipv6 address 2001:AAAA:2042:6::2/64
 vrrp 2 description VRRP_VID11
 vrrp 2 ip 20.42.3.1
 vrrp 2 authentication cisco

router bgp 2042
 bgp router-id 16.16.16.16
 bgp log-neighbor-changes
 network 20.42.0.0 mask 255.255.252.0
 network 20.42.1.0 mask 255.255.255.0
 network 20.42.2.0 mask 255.255.255.0
 network 20.42.3.0 mask 255.255.255.0
 neighbor 18.18.18.18 remote-as 2042
 neighbor 18.18.18.18 update-source Loopback0
 neighbor 18.18.18.18 next-hop-self

ip route 17.17.17.17 255.255.255.255 20.42.0.5
ip route 18.18.18.18 255.255.255.255 20.42.0.5
ip route 20.42.0.0 255.255.252.0 Null0
ip route 32.32.32.32 255.255.255.255 20.42.0.10

bridge 1 protocol ieee
bridge 1 route ip
bridge 2 protocol ieee
bridge 2 route ip
bridge 3 protocol ieee
bridge 3 route ip
```

##### Таблица маршрутизации R16:

```
R16#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      30.0.0.0/28 is subnetted, 1 subnets
B        30.1.0.0 [200/0] via 18.18.18.18, 00:32:51
      52.0.0.0/26 is subnetted, 1 subnets
B        52.0.0.0 [200/0] via 18.18.18.18, 00:32:51
      100.0.0.0/22 is subnetted, 1 subnets
B        100.1.0.0 [200/0] via 18.18.18.18, 00:32:51
      101.0.0.0/28 is subnetted, 1 subnets
B        101.0.0.0 [200/0] via 18.18.18.18, 00:32:51
R16#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      16.0.0.0/32 is subnetted, 1 subnets
C        16.16.16.16 is directly connected, Loopback0
      17.0.0.0/32 is subnetted, 1 subnets
S        17.17.17.17 [1/0] via 20.42.0.5
      18.0.0.0/32 is subnetted, 1 subnets
S        18.18.18.18 [1/0] via 20.42.0.5
      20.0.0.0/8 is variably subnetted, 11 subnets, 4 masks
S        20.42.0.0/22 is directly connected, Null0
C        20.42.0.4/30 is directly connected, Ethernet0/1
L        20.42.0.6/32 is directly connected, Ethernet0/1
C        20.42.0.8/30 is directly connected, Ethernet0/3
L        20.42.0.9/32 is directly connected, Ethernet0/3
C        20.42.1.0/24 is directly connected, BVI1
L        20.42.1.2/32 is directly connected, BVI1
C        20.42.2.0/24 is directly connected, BVI2
L        20.42.2.2/32 is directly connected, BVI2
C        20.42.3.0/24 is directly connected, BVI3
L        20.42.3.1/32 is directly connected, BVI3
      30.0.0.0/28 is subnetted, 1 subnets
B        30.1.0.0 [200/0] via 18.18.18.18, 00:32:54
      32.0.0.0/32 is subnetted, 1 subnets
S        32.32.32.32 [1/0] via 20.42.0.10
      52.0.0.0/26 is subnetted, 1 subnets
B        52.0.0.0 [200/0] via 18.18.18.18, 00:32:54
      100.0.0.0/22 is subnetted, 1 subnets
B        100.1.0.0 [200/0] via 18.18.18.18, 00:32:54
      101.0.0.0/28 is subnetted, 1 subnets
B        101.0.0.0 [200/0] via 18.18.18.18, 00:32:54
```

##### Конфигурация R32:

```
interface Loopback0
 ip address 32.32.32.32 255.255.255.255
router bgp 2042
 bgp router-id 32.32.32.32
 bgp log-neighbor-changes
 network 20.42.0.0 mask 255.255.252.0
 neighbor 18.18.18.18 remote-as 2042
 neighbor 18.18.18.18 update-source Loopback0
 neighbor 18.18.18.18 next-hop-self
ip route 16.16.16.16 255.255.255.255 20.42.0.9
ip route 17.17.17.17 255.255.255.255 20.42.0.9
ip route 18.18.18.18 255.255.255.255 20.42.0.9
ip route 20.42.0.0 255.255.252.0 Null0
ip route 20.42.0.4 255.255.255.252 20.42.0.9
```

##### Таблица маршрутизации R16:

```
R32#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      16.0.0.0/32 is subnetted, 1 subnets
S        16.16.16.16 [1/0] via 20.42.0.9
      17.0.0.0/32 is subnetted, 1 subnets
S        17.17.17.17 [1/0] via 20.42.0.9
      18.0.0.0/32 is subnetted, 1 subnets
S        18.18.18.18 [1/0] via 20.42.0.9
      20.0.0.0/8 is variably subnetted, 4 subnets, 3 masks
S        20.42.0.0/22 is directly connected, Null0
S        20.42.0.4/30 [1/0] via 20.42.0.9
C        20.42.0.8/30 is directly connected, Ethernet0/0
L        20.42.0.10/32 is directly connected, Ethernet0/0
      30.0.0.0/28 is subnetted, 1 subnets
B        30.1.0.0 [200/0] via 18.18.18.18, 00:08:16
      32.0.0.0/32 is subnetted, 1 subnets
C        32.32.32.32 is directly connected, Loopback0
      52.0.0.0/26 is subnetted, 1 subnets
B        52.0.0.0 [200/0] via 18.18.18.18, 00:08:16
      100.0.0.0/22 is subnetted, 1 subnets
B        100.1.0.0 [200/0] via 18.18.18.18, 00:08:16
      101.0.0.0/28 is subnetted, 1 subnets
B        101.0.0.0 [200/0] via 18.18.18.18, 00:08:16
```