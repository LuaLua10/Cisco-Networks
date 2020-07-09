Лабораторная работа. IPSec.
---------

Топология
---------

![](media/0735678569f8a389567856788c4e11e.png)

Задачи
---------

IPSec over DmVPN
Настроить GRE поверх IPSec между офисами Москва и С.-Петербург Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги
1. Настроите GRE поверх IPSec между офисами Москва и С.-Петербург
2. Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги
3. Все узлы в офисах в лабораторной работе должны иметь IP связность
4. План работы и изменения зафиксированы в документации

Решение
---------

#### Настроим GRE поверх IPSec между офисами Москва и С.-Петербург

##### Конфигурация R15:

```
crypto isakmp policy 10
 encr 3des
 hash md5 
 authentication pre-share
 group 2  
crypto isakmp key cisco address 52.0.0.18      
      
crypto ipsec transform-set SET esp-des 
 mode transport
     
crypto map MAP1 10 ipsec-isakmp 
 set peer 52.0.0.18
 set transform-set SET 
 match address 100

interface Tunnel1
 ip address 192.168.100.1 255.255.255.0
 ip mtu 1440
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/2
 tunnel destination 52.0.0.18
 
interface Ethernet0/2
 crypto map MAP1

ip route 20.42.0.0 255.255.252.0 tunnel1 
access-list 100 permit gre host 30.1.0.2 host 52.0.0.18
```

##### Сводная информация о туннели R15:

```
R15#show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
52.0.0.18       30.1.0.2        QM_IDLE           1004 ACTIVE
```

```
IPv6 Crypto ISAKMP SA

R15#show crypto ipsec sa

interface: Ethernet0/2
    Crypto map tag: MAP1, local addr 30.1.0.2

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (30.1.0.2/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (52.0.0.18/255.255.255.255/47/0)
   current_peer 52.0.0.18 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 50, #pkts encrypt: 50, #pkts digest: 50
    #pkts decaps: 23, #pkts decrypt: 23, #pkts verify: 23
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 30.1.0.2, remote crypto endpt.: 52.0.0.18
     path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/2
     current outbound spi: 0xD729A1E4(3609829860)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x9178A74A(2440603466)
        transform: esp-des ,
        in use settings ={Transport, }
        conn id: 11, flow_id: SW:11, sibling_flags 80004000, crypto map: MAP1
        sa timing: remaining key lifetime (k/sec): (4337552/1779)
        IV size: 8 bytes
        replay detection support: N
        Status: ACTIVE(ACTIVE)
          
     inbound ah sas:
          
     inbound pcp sas:
          
     outbound esp sas:
      spi: 0xD729A1E4(3609829860)
        transform: esp-des ,
        in use settings ={Transport, }
        conn id: 12, flow_id: SW:12, sibling_flags 80004000, crypto map: MAP1
        sa timing: remaining key lifetime (k/sec): (4337548/1779)
        IV size: 8 bytes
        replay detection support: N
        Status: ACTIVE(ACTIVE)
          
     outbound ah sas:
          
     outbound pcp sas:
```

##### Конфигурация R18:

```
crypto isakmp policy 10
 encr 3des
 hash md5 
 authentication pre-share
 group 2  
crypto isakmp key cisco address 30.1.0.2       
      
crypto ipsec transform-set SET esp-des 
 mode transport
         
crypto map MAP1 10 ipsec-isakmp 
 set peer 30.1.0.2
 set transform-set SET 
 match address 100

interface Tunnel1
 ip address 192.168.100.2 255.255.255.0
 ip mtu 1440
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/2
 tunnel destination 30.1.0.2

interface Ethernet0/2
 crypto map MAP1

ip route 100.1.0.0 255.255.252.0 Tunnel1
access-list 100 permit gre host 52.0.0.18 host 30.1.0.2
```

##### Сводная информация о туннели R18:

```
R18#show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
52.0.0.18       30.1.0.2        QM_IDLE           1004 ACTIVE

IPv6 Crypto ISAKMP SA
```

```
R18#show crypto ipsec sa

interface: Ethernet0/2
    Crypto map tag: MAP1, local addr 52.0.0.18

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (52.0.0.18/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (30.1.0.2/255.255.255.255/47/0)
   current_peer 30.1.0.2 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 23, #pkts encrypt: 23, #pkts digest: 23
    #pkts decaps: 50, #pkts decrypt: 50, #pkts verify: 50
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 52.0.0.18, remote crypto endpt.: 30.1.0.2
     path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/2
     current outbound spi: 0x9178A74A(2440603466)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0xD729A1E4(3609829860)
        transform: esp-des ,
        in use settings ={Transport, }
        conn id: 11, flow_id: SW:11, sibling_flags 80000000, crypto map: MAP1
        sa timing: remaining key lifetime (k/sec): (4202820/2077)
        IV size: 8 bytes
        replay detection support: N
        Status: ACTIVE(ACTIVE)
          
     inbound ah sas:
          
     inbound pcp sas:
          
     outbound esp sas:
      spi: 0x9178A74A(2440603466)
        transform: esp-des ,
        in use settings ={Transport, }
        conn id: 12, flow_id: SW:12, sibling_flags 80000000, crypto map: MAP1
        sa timing: remaining key lifetime (k/sec): (4202824/2077)
        IV size: 8 bytes
        replay detection support: N
        Status: ACTIVE(ACTIVE)
          
     outbound ah sas:
          
     outbound pcp sas:
```

##### Трасировка между VPC1 и VPC8

```
VPCS> trace 20.42.2.10
trace to 20.42.2.10, 8 hops max, press Ctrl+C to stop
 1   100.1.1.1   0.576 ms  0.422 ms  0.403 ms
 2   100.1.0.33   0.733 ms  0.647 ms  0.577 ms
 3   100.1.0.17   1.044 ms  0.829 ms  0.800 ms
 4   192.168.100.2   1.911 ms  2.033 ms  1.886 ms
 5   20.42.0.6   2.082 ms  1.885 ms  2.229 ms
 6   *20.42.2.10   2.235 ms (ICMP type:3, code:3, Destination port unreachable)
```

#### Настроим DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги

> R14 - HUB, R27,R28 - Spoke

##### Конфигурация R14:

```
crypto isakmp policy 10
 encr 3des
 hash md5 
 authentication pre-share
 group 2  
crypto isakmp key cisco address 0.0.0.0        
       
crypto ipsec transform-set SET esp-des 
 mode transport
     
crypto ipsec profile DMVPN
 set transform-set SET 

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
 tunnel protection ipsec profile DMVPN

router ospf 1
 network 192.168.200.0 0.0.0.255 area 20
```

##### Сводная информация о туннели R14:

```
R14#show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
101.0.0.2       52.0.0.34       QM_IDLE           1001 ACTIVE
101.0.0.2       52.0.0.26       QM_IDLE           1002 ACTIVE

IPv6 Crypto ISAKMP SA
```

```
R14#show crypto ipsec sa

interface: Tunnel1
    Crypto map tag: Tunnel1-head-0, local addr 101.0.0.2

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (101.0.0.2/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (52.0.0.26/255.255.255.255/47/0)
   current_peer 52.0.0.26 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 229, #pkts encrypt: 229, #pkts digest: 229
    #pkts decaps: 212, #pkts decrypt: 212, #pkts verify: 212
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 101.0.0.2, remote crypto endpt.: 52.0.0.26
     path mtu 1500, ip mtu 1500, ip mtu idb (none)
     current outbound spi: 0x39D033A9(969946025)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x5614EC94(1444211860)
        transform: esp-des ,
        in use settings ={Transport, }
        conn id: 55, flow_id: SW:55, sibling_flags 80000000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4266540/2610)
        IV size: 8 bytes
        replay detection support: N
        Status: ACTIVE(ACTIVE)
          
     inbound ah sas:
          
     inbound pcp sas:
          
     outbound esp sas:
      spi: 0x39D033A9(969946025)
        transform: esp-des ,
        in use settings ={Transport, }
        conn id: 56, flow_id: SW:56, sibling_flags 80000000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4266540/2610)
        IV size: 8 bytes
        replay detection support: N
        Status: ACTIVE(ACTIVE)
          
     outbound ah sas:
          
     outbound pcp sas:
          
   protected vrf: (none)
   local  ident (addr/mask/prot/port): (101.0.0.2/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (52.0.0.34/255.255.255.255/47/0)
   current_peer 52.0.0.34 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 3025, #pkts encrypt: 3025, #pkts digest: 3025
    #pkts decaps: 52, #pkts decrypt: 52, #pkts verify: 52
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0
          
     local crypto endpt.: 101.0.0.2, remote crypto endpt.: 52.0.0.34
     path mtu 1500, ip mtu 1500, ip mtu idb (none)
     current outbound spi: 0x6A179D90(1779932560)
     PFS (Y/N): N, DH group: none
          
     inbound esp sas:
      spi: 0x28B63D6(42689494)
        transform: esp-des ,
        in use settings ={Transport, }
        conn id: 53, flow_id: SW:53, sibling_flags 80000000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4335159/1898)
        IV size: 8 bytes
        replay detection support: N
        Status: ACTIVE(ACTIVE)
          
     inbound ah sas:
          
     inbound pcp sas:
          
     outbound esp sas:
      spi: 0x6A179D90(1779932560)
        transform: esp-des ,
        in use settings ={Transport, }
        conn id: 54, flow_id: SW:54, sibling_flags 80000000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4335151/1898)
        IV size: 8 bytes
        replay detection support: N
        Status: ACTIVE(ACTIVE)
          
     outbound ah sas:
          
     outbound pcp sas:
```

##### Конфигурация R27:

```
crypto isakmp policy 10
 encr 3des
 hash md5 
 authentication pre-share
 group 2  
crypto isakmp key cisco address 0.0.0.0        
      
crypto ipsec transform-set SET esp-des 
 mode transport
       
crypto ipsec profile DMVPN
 set transform-set SET 

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
 tunnel protection ipsec profile DMVPN
```

##### Сводная информация о туннели R27:

```
R27#show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
101.0.0.2       52.0.0.34       QM_IDLE           1001 ACTIVE

IPv6 Crypto ISAKMP SA
```

```
R27#show crypto ipsec sa

interface: Tunnel1
    Crypto map tag: Tunnel1-head-0, local addr 52.0.0.34

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (52.0.0.34/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (101.0.0.2/255.255.255.255/47/0)
   current_peer 101.0.0.2 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 52, #pkts encrypt: 52, #pkts digest: 52
    #pkts decaps: 3034, #pkts decrypt: 3034, #pkts verify: 3034
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 52.0.0.34, remote crypto endpt.: 101.0.0.2
     path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/0
     current outbound spi: 0x28B63D6(42689494)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x6A179D90(1779932560)
        transform: esp-des ,
        in use settings ={Transport, }
        conn id: 51, flow_id: SW:51, sibling_flags 80004000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4174358/1684)
        IV size: 8 bytes
        replay detection support: N
        Status: ACTIVE(ACTIVE)
          
     inbound ah sas:
          
     inbound pcp sas:
          
     outbound esp sas:
      spi: 0x28B63D6(42689494)
        transform: esp-des ,
        in use settings ={Transport, }
        conn id: 52, flow_id: SW:52, sibling_flags 80004000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4174367/1684)
        IV size: 8 bytes
        replay detection support: N
        Status: ACTIVE(ACTIVE)
          
     outbound ah sas:
          
     outbound pcp sas:
```

##### Конфигурация R28:

```
crypto isakmp policy 10
 encr 3des
 hash md5 
 authentication pre-share
 group 2  
crypto isakmp key cisco address 0.0.0.0        
      
crypto ipsec transform-set SET esp-des 
 mode transport
      
crypto ipsec profile DMVPN
 set transform-set SET 

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
 tunnel source 52.0.0.26
 tunnel destination 101.0.0.2
 tunnel key 123
 tunnel protection ipsec profile DMVPN

router ospf 1
 router-id 28.28.28.28
 passive-interface Ethernet0/0
 passive-interface Ethernet0/1
 network 101.1.0.0 0.0.3.255 area 20
 network 192.168.200.0 0.0.0.255 area 20

ip access-list standard VPN
 permit 100.1.0.0 0.0.3.255

route-map PBR1 permit 5
 match ip next-hop VPN
 set interface Tunnel1
```

##### Сводная информация о туннели R28:

```
R28#show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
101.0.0.2       52.0.0.26       QM_IDLE           1001 ACTIVE

IPv6 Crypto ISAKMP SA
```

```
R28#show crypto ipsec sa

interface: Tunnel1
    Crypto map tag: Tunnel1-head-0, local addr 52.0.0.26

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (52.0.0.26/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (101.0.0.2/255.255.255.255/47/0)
   current_peer 101.0.0.2 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 267, #pkts encrypt: 267, #pkts digest: 267
    #pkts decaps: 263, #pkts decrypt: 263, #pkts verify: 263
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 52.0.0.26, remote crypto endpt.: 101.0.0.2
     path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/0
     current outbound spi: 0x5614EC94(1444211860)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x39D033A9(969946025)
        transform: esp-des ,
        in use settings ={Transport, }
        conn id: 3, flow_id: SW:3, sibling_flags 80004000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4263722/2052)
        IV size: 8 bytes
        replay detection support: N
        Status: ACTIVE(ACTIVE)
          
     inbound ah sas:
          
     inbound pcp sas:
          
     outbound esp sas:
      spi: 0x5614EC94(1444211860)
        transform: esp-des ,
        in use settings ={Transport, }
        conn id: 4, flow_id: SW:4, sibling_flags 80004000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4263722/2052)
        IV size: 8 bytes
        replay detection support: N
        Status: ACTIVE(ACTIVE)
          
     outbound ah sas:
          
     outbound pcp sas:
```

##### Трасировка между VPC31 и VPC7

```
VPCS> trace 100.1.2.11
trace to 100.1.2.11, 8 hops max, press Ctrl+C to stop
 1   101.1.2.1   0.614 ms  0.423 ms  0.426 ms
 2   192.168.200.1   2.253 ms  1.689 ms  1.647 ms
 3   100.1.0.6   2.034 ms  1.842 ms  1.772 ms
 4   100.1.0.26   2.079 ms  1.862 ms  2.066 ms
 5   *100.1.2.11   2.397 ms (ICMP type:3, code:3, Destination port unreachable)
```
