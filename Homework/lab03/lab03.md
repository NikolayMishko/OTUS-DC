# Домашнее задание №3
## Underlay. ISIS

## Цель:
- ### Настроить ISIS для Underlay сети

## Выполнение
### Схема сети
![alt text](images/lab02_ISIS.png)


### План работ
- #### настройка процесса ISIS
	- В net зашифрован ip адрес интерфейса loopback 1
	 - Настройка аутентификации
	  - Включение процесса ISIS
- #### настройка интерфейсов

    - Тип сети = point-to-point (для Ethernet интерфейсов)
    - включение BFD для ISIS
    - ISIS timers = defaults
 - #### проверка связности

### Конфигурация 
```
!
interface Ethernet1
   description --- Leaf-01 ---
   no switchport
   ip address 10.11.101.2/31
   isis enable TST
   isis bfd
!
interface Ethernet2
   description --- Leaf-02 ---
   no switchport
   ip address 10.11.101.4/31
   isis enable TST
   isis bfd
!
interface Ethernet3
   description --- Leaf-03 ---
   no switchport
   ip address 10.11.101.6/31
   isis enable TST
   isis bfd
!
interface Loopback1
   description --- Routing ---
   ip address 10.11.0.101/32
   isis enable TST
!
ip routing
no ip routing vrf MGMT
!
router isis TST
   net 49.0001.0100.1100.0101.00
   authentication mode md5
   authentication key 7 zQSKLK8ImCE=
   !
   address-family ipv4 unicast
!
```

- spine-2
```
!
interface Ethernet1
   description --- Leaf-01 ---
   no switchport
   ip address 10.11.102.2/31
   isis enable TST
   isis bfd
!
interface Ethernet2
   description --- Leaf-02 ---
   no switchport
   ip address 10.11.102.4/31
   isis enable TST
   isis bfd
!
interface Ethernet3
   description --- Leaf-03 ---
   no switchport
   ip address 10.11.102.6/31
   isis enable TST
   isis bfd
!
interface Loopback1
   description --- For Routing ---
   ip address 10.11.0.102/32
   isis enable TST
!
!
ip routing
no ip routing vrf MGMT
!
router isis TST
   net 49.0001.0100.1100.0102.00
   authentication mode md5
   authentication key 7 abqdVoorX9QBPXMJ2bwNpA==
   !
   address-family ipv4 unicast
!
```
- leaf-1
```
!
interface Ethernet7
   description --- Spine-01 ---
   no switchport
   ip address 10.11.101.3/31
   isis enable TST
   isis bfd
!
interface Ethernet8
   description --- Spine-02 ---
   no switchport
   ip address 10.11.102.3/31
   isis enable TST
   isis bfd
!
interface Loopback1
   description --- For Routing ---
   ip address 10.11.101.0/32
   isis enable TST
!
ip routing
no ip routing vrf MGMT
!
router isis TST
   net 49.0001.0100.1110.1000.00
   authentication mode md5
   authentication key 7 abqdVoorX9QBPXMJ2bwNpA==
   !
   address-family ipv4 unicast
!
end
```
- leaf-2
```
!
interface Ethernet7
   description --- Spine-01 ---
   no switchport
   ip address 10.11.101.5/31
   isis enable TST
   isis bfd
!
interface Ethernet8
   description --- Spine-02 ---
   no switchport
   ip address 10.11.102.5/31
   isis enable TST
   isis bfd
!
interface Loopback1
   description --- For Routing ---
   ip address 10.11.102.0/32
   isis enable TST
!
ip routing
no ip routing vrf MGMT
!
router isis TST
   net 49.0001.0100.1110.2000.00
   authentication mode md5
   authentication key 7 abqdVoorX9QBPXMJ2bwNpA==
   !
   address-family ipv4 unicast
!
```
- leaf-3
```
!
interface Ethernet7
   description --- Spine-01 ---
   no switchport
   ip address 10.11.101.7/31
   isis enable TST
   isis bfd
!
interface Ethernet8
   description --- Spine-02 ---
   no switchport
   ip address 10.11.102.7/31
   isis enable TST
   isis bfd
!
interface Loopback1
   description --- For Routing ---
   ip address 10.11.103.0/32
   isis enable TST
!
ip routing
no ip routing vrf MGMT
!
router isis TST
   net 49.0001.0100.1110.3000.00
   authentication mode md5
   authentication key 7 abqdVoorX9QBPXMJ2bwNpA==
   !
   address-family ipv4 unicast
!
```

### Проверка связанности устройств по протоколу ISIS

#### ISIS-соседства
- spine-1
```
Spine-01#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
TST       default  Leaf-01          L1   Ethernet1          50:2b:b6:54:9:27  UP    6           Leaf-01.13
TST       default  Leaf-01          L2   Ethernet1          50:2b:b6:54:9:27  UP    7           Leaf-01.13
TST       default  Leaf-02          L1   Ethernet2          50:f9:ee:c:9f:59  UP    8           Leaf-02.14
TST       default  Leaf-02          L2   Ethernet2          50:f9:ee:c:9f:59  UP    7           Leaf-02.14
TST       default  Leaf-03          L1   Ethernet3          50:c1:19:7b:cd:55 UP    8           Leaf-03.13
TST       default  Leaf-03          L2   Ethernet3          50:c1:19:7b:cd:55 UP    7           Leaf-03.13
Spine-01#
```

#### ISIS таблица маршрутов
```
Spine-01#show ip route isis

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 I L1     10.11.0.102/32 [115/30] via 10.11.101.3, Ethernet1
                                  via 10.11.101.5, Ethernet2
                                  via 10.11.101.7, Ethernet3
 I L1     10.11.101.0/32 [115/20] via 10.11.101.3, Ethernet1
 I L1     10.11.102.0/32 [115/20] via 10.11.101.5, Ethernet2
 I L1     10.11.102.2/31 [115/20] via 10.11.101.3, Ethernet1
 I L1     10.11.102.4/31 [115/20] via 10.11.101.5, Ethernet2
 I L1     10.11.102.6/31 [115/20] via 10.11.101.7, Ethernet3
 I L1     10.11.103.0/32 [115/20] via 10.11.101.7, Ethernet3
```

#### Проверка связности Loopback 
 ```
 Spine-01#ping 10.11.0.102
PING 10.11.0.102 (10.11.0.102) 72(100) bytes of data.
80 bytes from 10.11.0.102: icmp_seq=1 ttl=63 time=11.7 ms
80 bytes from 10.11.0.102: icmp_seq=2 ttl=63 time=12.5 ms
80 bytes from 10.11.0.102: icmp_seq=3 ttl=63 time=11.3 ms
80 bytes from 10.11.0.102: icmp_seq=4 ttl=63 time=14.8 ms
80 bytes from 10.11.0.102: icmp_seq=5 ttl=63 time=16.1 ms
--- 10.11.0.102 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 55ms
rtt min/avg/max/mdev = 11.387/13.317/16.136/1.844 ms, pipe 2, ipg/ewma 13.905/12.664 ms

Spine-01#ping 10.11.101.0
PING 10.11.101.0 (10.11.101.0) 72(100) bytes of data.
80 bytes from 10.11.101.0: icmp_seq=1 ttl=64 time=4.75 ms
80 bytes from 10.11.101.0: icmp_seq=2 ttl=64 time=6.45 ms
80 bytes from 10.11.101.0: icmp_seq=3 ttl=64 time=4.40 ms
80 bytes from 10.11.101.0: icmp_seq=4 ttl=64 time=4.84 ms
80 bytes from 10.11.101.0: icmp_seq=5 ttl=64 time=6.65 ms
--- 10.11.101.0 ping statistics ---
[Kpackets transmitted, 5 received, 0% packet loss, time 33ms
ping: 10.: Name or service not known.654/0.941 ms, ipg/ewma 8.316/5.109 ms

Spine-01#ping 10.11.102.0
PING 10.11.102.0 (10.11.102.0) 72(100) bytes of data.
80 bytes from 10.11.102.0: icmp_seq=1 ttl=64 time=5.79 ms
80 bytes from 10.11.102.0: icmp_seq=2 ttl=64 time=7.45 ms
80 bytes from 10.11.102.0: icmp_seq=3 ttl=64 time=4.52 ms
80 bytes from 10.11.102.0: icmp_seq=4 ttl=64 time=8.61 ms
80 bytes from 10.11.102.0: icmp_seq=5 ttl=64 time=5.87 ms
--- 10.11.102.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 39ms
rtt min/avg/max/mdev = 4.520/6.450/8.610/1.424 ms, ipg/ewma 9.965/6.130 ms

Spine-01#ping 10.11.103.0
PING 10.11.103.0 (10.11.103.0) 72(100) bytes of data.
80 bytes from 10.11.103.0: icmp_seq=1 ttl=64 time=6.35 ms
80 bytes from 10.11.103.0: icmp_seq=2 ttl=64 time=4.80 ms
80 bytes from 10.11.103.0: icmp_seq=3 ttl=64 time=4.50 ms
80 bytes from 10.11.103.0: icmp_seq=4 ttl=64 time=5.70 ms
80 bytes from 10.11.103.0: icmp_seq=5 ttl=64 time=4.10 ms
--- 10.11.103.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 39ms
rtt min/avg/max/mdev = 4.106/5.094/6.356/0.823 ms, ipg/ewma 9.909/5.696 ms
Spine-01#
```
