# Домашнее задание №8
## VxLAN. Routing

## Цель:
- ### Реализовать маршрутизацию между "клиентами" через EVPN route-type 5

## Выполнение
### Схема сети
![alt text](images/lab08_routing.png)

### Распределение идентификаторов
|  Клиент |  Subnet  | IP  | Leaf  | Порт подключения  | VLAN  | VRF | VNI |
| :------------: | :------------: | :------------: | :------------: | :------------: | :------------: |:------------: |:------------: |
| Клиент 1 | 192.168.20.2/24  | 192.168.20.1  |  2 | Eht1  |  20 |VRF2040|12040
| Клиент 2 | 192.168.40.2/24 | 192.168.40.2  |  3 | Eht2   |  40 |VRF2040|12040
| Клиент 3 | 192.168.50.2/24 | 192.168.50.2  |  3 | Eht1   |  50 |VRF50|10050

### План работ
Доработаем [лабораторную работу №7](/Homework/Lab07/lab07.md) для возможности установки EVPN route-type 5 в таблицы маршрутизации VxLAN VRF. 

### Шаги для выполнения работы
1. Проверить достижимость loopback'ов с каждого Leaf до каждого Spine и других Leaf. Все должно работать с уже настроенным Underlay.
2. Настроить Overlay, L2VXLAN, L3VXLAN, Multihoming для клиента.
3. Подключить к Leaf-3 маршрутизатор BR. Настроить BGPv4-пиринг между Leaf-3 и BR для каждого клиентского VRF.
4. Настроить BR, перетекание маршрутов, принятых от Leaf-3.
5. Разрешить проблему с повторяющимся номером AS в AS_PATH атрибуте.
6. Проверить связность inter-VRF, убедиться, что трафик между VRF ходит через BR.
7. Дополнительно, для одного из клиентов, реализуем выход из фабрики через BR к внешней сети 8.8.8.8/32 и получение маршрута по умолчанию 0.0.0.0/0 как пример подключения к внешним ресурсам за firewall.

Доработаем лабораторную работу №7 для возможности установки EVPN route-type 5 в таблицы маршрутизации VxLAN VRF. 

VxLAN фабрика реализована на eBGP для организации маршрутизации между VRF через BR роутер необходимо разрешить проблему с повторяющимся номером AS в AS_PATH атрибуте.
 Для этого: 
 - можно использовать подмену AS 
 "neighbor 192.168.40.254 local-as "подменная_AS_для_клиента_в_VRF2040" no-prepend replace" 
 в таком случае BR установит BGP соседство с подменной_AS и в AS_PATH вместо дефолтной AS  стыковочного leaf будет фигурировать она.
 - можно на BR с помощью route-map в анонсируемых соседу маршрутах удалять данные из атрибута AS_PATH  "set as-path match all replacement none" 
 в таком случае BR будет анонсировать клиентские маршруты от своего имени




#### Конфигурация underlay соответствует [OSPF underlay из lab02](/Homework/Lab02/lab02.md)

### Конфигурация оборудования
#### spine-1
```
!
service routing protocols model multi-agent
!
interface Ethernet1
   description --- Leaf-01 ---
   mtu 9214
   no switchport
   ip address 10.11.101.2/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description --- Leaf-02 ---
   mtu 9214
   no switchport
   ip address 10.11.101.4/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description --- Leaf-03 ---
   mtu 9214
   no switchport
   ip address 10.11.101.6/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   description --- Routing ---
   ip address 10.11.0.101/32
!
ip routing
!
ip prefix-list PL_OSPF_OUT seq 10 permit 10.11.0.101/32
!
mpls ip
!
route-map RM_OSPF_OUT permit 1
   match ip address prefix-list PL_OSPF_OUT
!
router bgp 65001
   router-id 10.11.0.101
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 64
   neighbor L_OVERLAY peer group
   neighbor L_OVERLAY next-hop-unchanged
   neighbor L_OVERLAY update-source Loopback1
   neighbor L_OVERLAY bfd
   neighbor L_OVERLAY ebgp-multihop 3
   neighbor L_OVERLAY send-community extended
   neighbor 10.11.101.0 peer group L_OVERLAY
   neighbor 10.11.101.0 remote-as 65101
   neighbor 10.11.102.0 peer group L_OVERLAY
   neighbor 10.11.102.0 remote-as 65102
   neighbor 10.11.103.0 peer group L_OVERLAY
   neighbor 10.11.103.0 remote-as 65103
   !
   address-family evpn
      neighbor L_OVERLAY activate
!
router ospf 1
   router-id 10.11.0.101
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   redistribute connected route-map RM_OSPF_OUT
   max-lsa 12000
!
```
#### spine-2
```
!
service routing protocols model multi-agent
!
!
interface Ethernet1
   description --- Leaf-01 ---
   mtu 9214
   no switchport
   ip address 10.11.102.2/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description --- Leaf-02 ---
   mtu 9214
   no switchport
   ip address 10.11.102.4/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description --- Leaf-03 ---
   mtu 9214
   no switchport
   ip address 10.11.102.6/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   description --- For Routing ---
   ip address 10.11.0.102/32
!
ip routing
!
ip prefix-list PL_OSPF_OUT seq 10 permit 10.11.0.102/32
!
mpls ip
!
route-map RM_OSPF_OUT permit 1
   match ip address prefix-list PL_OSPF_OUT
!
router bgp 65001
   router-id 10.11.0.102
   maximum-paths 4 ecmp 64
   neighbor LEAF_OVERLAY peer group
   neighbor L_OVERLAY peer group
   neighbor L_OVERLAY next-hop-unchanged
   neighbor L_OVERLAY update-source Loopback1
   neighbor L_OVERLAY bfd
   neighbor L_OVERLAY ebgp-multihop 3
   neighbor L_OVERLAY send-community extended
   neighbor 10.11.101.0 peer group L_OVERLAY
   neighbor 10.11.101.0 remote-as 65101
   neighbor 10.11.102.0 peer group L_OVERLAY
   neighbor 10.11.102.0 remote-as 65102
   neighbor 10.11.103.0 peer group L_OVERLAY
   neighbor 10.11.103.0 remote-as 65103
   !
   address-family evpn
      neighbor L_OVERLAY activate
!
router ospf 1
   router-id 10.11.0.102
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   redistribute connected route-map RM_OSPF_OUT
   max-lsa 12000
!
 ``` 

#### leaf-1
```
service routing protocols model multi-agent
!
vlan 10,20
!
vrf instance VRF2040
!
interface Port-Channel1
   switchport trunk allowed vlan 10,20
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:1111:1111:1111:0000
      route-target import 11:11:11:11:11:11
   lacp system-id 1111.1111.1111
!
interface Ethernet1
   switchport access vlan 10
!
interface Ethernet2
   no switchport
   channel-group 1 mode active
!
interface Ethernet3
   switchport access vlan 10
!
!
interface Ethernet7
   description --- Spine-01 ---
   mtu 9214
   no switchport
   ip address 10.11.101.3/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   description --- Spine-02 ---
   mtu 9214
   no switchport
   ip address 10.11.102.3/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   description --- For Routing ---
   ip address 10.11.101.0/32
!
interface Vlan20
   vrf VRF2040
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10-20 vni 10010-10020
   vxlan vrf VRF2040 vni 12040
!
ip virtual-router mac-address aa:bb:cc:dd:ee:ff
!
ip routing
ip routing vrf VRF2040
!
ip prefix-list PL_OSPF_OUT seq 10 permit 10.11.101.0/32
!
mpls ip
!
route-map RM_OSPF_OUT permit 1
   match ip address prefix-list PL_OSPF_OUT
!
router bgp 65101
   router-id 10.11.101.0
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 64
   neighbor SP_OVERLAY peer group
   neighbor SP_OVERLAY update-source Loopback1
   neighbor SP_OVERLAY bfd
   neighbor SP_OVERLAY ebgp-multihop 3
   neighbor SP_OVERLAY send-community extended
   neighbor 10.11.0.101 peer group SP_OVERLAY
   neighbor 10.11.0.101 remote-as 65001
   neighbor 10.11.0.102 peer group SP_OVERLAY
   neighbor 10.11.0.102 remote-as 65001
   redistribute connected
   !
   vlan 10
      rd 10.11.101.0:10010
      route-target both 3:10010
      redistribute learned
   !
   address-family evpn
      neighbor SP_OVERLAY activate
   !
   vrf VRF2040
      rd 10.11.101.0:12040
      route-target import evpn 1:12040
      route-target export evpn 1:12040
      redistribute connected
!
router ospf 1
   router-id 10.11.101.0
   passive-interface default
   no passive-interface Ethernet7
   no passive-interface Ethernet8
   redistribute connected route-map RM_OSPF_OUT
   max-lsa 12000
!
```
#### leaf-2
```
!
service routing protocols model multi-agent
!
vlan 10,20,30,40
!
vrf instance VRF2040
!
interface Port-Channel1
   switchport trunk allowed vlan 10,20
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:1111:1111:1111:0000
      route-target import 11:11:11:11:11:11
   lacp system-id 1111.1111.1111
!
interface Ethernet1
   switchport access vlan 20
!
interface Ethernet2
   channel-group 1 mode active
!
!
interface Ethernet7
   description --- Spine-01 ---
   mtu 9214
   no switchport
   ip address 10.11.101.5/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   description --- Spine-02 ---
   mtu 9214
   no switchport
   ip address 10.11.102.5/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   description --- For Routing ---
   ip address 10.11.102.0/32
!
interface Vlan1
!
interface Vlan20
   vrf VRF2040
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10-20 vni 10010-10020
   vxlan vrf VRF2040 vni 12040
!
ip virtual-router mac-address aa:bb:cc:dd:ee:ff
!
ip routing
ip routing vrf VRF2040
!
ip prefix-list PL_OSPF_OUT seq 10 permit 10.11.102.0/32
!
route-map RM_OSPF_OUT permit 1
   match ip address prefix-list PL_OSPF_OUT
!
router bgp 65102
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 64
   neighbor SP_OVERLAY peer group
   neighbor SP_OVERLAY update-source Loopback1
   neighbor SP_OVERLAY bfd
   neighbor SP_OVERLAY ebgp-multihop 3
   neighbor SP_OVERLAY send-community
   neighbor 10.11.0.101 peer group SP_OVERLAY
   neighbor 10.11.0.101 remote-as 65001
   neighbor 10.11.0.102 peer group SP_OVERLAY
   neighbor 10.11.0.102 remote-as 65001
   !
   vlan 10
      rd 10.11.102.0:10010
      route-target both 3:10010
      redistribute learned
   !
   address-family evpn
      neighbor SP_OVERLAY activate
   !
   vrf VRF2040
      rd 10.11.102.0:12040
      route-target import evpn 1:12040
      route-target export evpn 1:12040
      redistribute connected
!
router ospf 1
   router-id 10.11.102.0
   passive-interface default
   no passive-interface Ethernet7
   no passive-interface Ethernet8
   redistribute connected route-map RM_OSPF_OUT
   max-lsa 12000
!
```
#### leaf-3
```
!
service routing protocols model multi-agent
!
!
vrf instance VRF2040
!
vrf instance VRF50
!
interface Ethernet1
   switchport access vlan 50
!
interface Ethernet2
   switchport access vlan 40
!
interface Ethernet4
   switchport trunk allowed vlan 40,50
   switchport mode trunk
!
!
interface Ethernet7
   description --- Spine-01 ---
   mtu 9214
   no switchport
   ip address 10.11.101.7/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   description --- Spine-02 ---
   mtu 9214
   no switchport
   ip address 10.11.102.7/31
   bfd interval 200 min-rx 200 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet9
!
interface Loopback1
   description --- For Routing ---
   ip address 10.11.103.0/32
!
interface Loopback5
   vrf VRF50
   ip address 5.5.5.5/32
!
interface Vlan40
   vrf VRF2040
   ip address 192.168.40.1/24
!
interface Vlan50
   vrf VRF50
   ip address 192.168.50.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10-20 vni 10010-10020
   vxlan vrf VRF2040 vni 12040
   vxlan vrf VRF50 vni 10050
!
ip routing
ip routing vrf VRF2040
ip routing vrf VRF50
!
ip prefix-list PL_OSPF_OUT seq 10 permit 10.11.103.0/32
!
mpls ip
!
route-map RM_OSPF_OUT permit 1
   match ip address prefix-list PL_OSPF_OUT
!
router bgp 65103
   router-id 10.11.103.0
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 64
   neighbor SPINE_OVERLAY peer group
   neighbor SP_OVERLAY peer group
   neighbor SP_OVERLAY update-source Loopback1
   neighbor SP_OVERLAY bfd
   neighbor SP_OVERLAY ebgp-multihop 3
   neighbor SP_OVERLAY send-community extended
   neighbor 10.11.0.101 peer group SP_OVERLAY
   neighbor 10.11.0.101 remote-as 65001
   neighbor 10.11.0.102 peer group SP_OVERLAY
   neighbor 10.11.0.102 remote-as 65001
   redistribute connected
   !
   vlan 10
      rd 10.11.103.0:10010
      route-target both 3:10010
      redistribute learned
   !
   address-family evpn
      neighbor SP_OVERLAY activate
   !
   vrf VRF2040
      rd 10.11.103.0:12040
      route-target import evpn 1:12040
      route-target export evpn 1:12040
      router-id 192.168.40.1
      neighbor 192.168.40.254 remote-as 65450
      neighbor 192.168.40.254 local-as 62040 no-prepend replace-as fallback
      neighbor 192.168.40.254 bfd
      redistribute connected
      !
      address-family ipv4
         neighbor 192.168.40.254 activate
   !
   vrf VRF50
      rd 10.11.103.0:10050
      route-target import evpn 1:10050
      route-target export evpn 1:10050
      router-id 192.168.50.1
      neighbor 192.168.50.254 remote-as 65450
      neighbor 192.168.50.254 bfd
      redistribute connected
      !
      address-family ipv4
         neighbor 192.168.50.254 activate
!
router ospf 1
   router-id 10.11.103.0
   passive-interface default
   no passive-interface Ethernet7
   no passive-interface Ethernet8
   redistribute connected route-map RM_OSPF_OUT
   max-lsa 12000
!
end
```
#### BR
```
!
vlan 40,50
!
interface Ethernet1
   no switchport
   ip address 10.10.10.2/30
!
interface Ethernet4
   switchport trunk allowed vlan 40,50
   switchport mode trunk
!
interface Loopback0
   ip address 10.11.255.0/32
!
interface Vlan40
   ip address 192.168.40.254/24
   bfd interval 200 min-rx 200 multiplier 3
!
interface Vlan50
   ip address 192.168.50.254/24
   bfd interval 200 min-rx 200 multiplier 3
!
ip routing
!
ip route 8.8.8.8/32 10.10.10.1
!
route-map SINGLE_AS_OUT permit 10
   set as-path match all replacement none
!
router bgp 65450
   neighbor 192.168.40.1 remote-as 65103
   neighbor 192.168.40.1 bfd
   neighbor 192.168.40.1 route-map SINGLE_AS_OUT out
   neighbor 192.168.50.1 remote-as 65103
   neighbor 192.168.50.1 bfd
   neighbor 192.168.50.1 route-map SINGLE_AS_OUT out
   neighbor 192.168.50.1 default-originate always
   redistribute connected
   redistribute bgp leaked
   !
   address-family ipv4
      neighbor 192.168.40.1 activate
      neighbor 192.168.50.1 activate
!
 ```

#### way_out
```
!
interface Ethernet1
   no switchport
   ip address 10.10.10.1/30
!
interface Loopback8
   ip address 8.8.8.8/32
!
ip routing
!
ip route 192.168.0.0/16 10.10.10.2
```



#### Проверка работы схемы
```
Leaf-03#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.11.103.0, local AS number 65103
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.11.103.0:10050 ip-prefix 0.0.0.0/0
                                 -                     -       100     0       65450 ?
 * >      RD: 10.11.103.0:10050 ip-prefix 5.5.5.5/32
                                 -                     -       -       0       i
 * >      RD: 10.11.103.0:12040 ip-prefix 5.5.5.5/32
                                 -                     -       100     0       65450 i
 * >      RD: 10.11.103.0:10050 ip-prefix 10.10.10.0/30
                                 -                     -       100     0       65450 i
 * >      RD: 10.11.103.0:12040 ip-prefix 10.10.10.0/30
                                 -                     -       100     0       65450 i
 * >      RD: 10.11.103.0:10050 ip-prefix 10.11.255.0/32
                                 -                     -       100     0       65450 i
 * >      RD: 10.11.103.0:12040 ip-prefix 10.11.255.0/32
                                 -                     -       100     0       65450 i
 * >Ec    RD: 10.11.102.0:12040 ip-prefix 192.168.20.0/24
                                 10.11.102.0           -       100     0       65001 65102 i
 *  ec    RD: 10.11.102.0:12040 ip-prefix 192.168.20.0/24
                                 10.11.102.0           -       100     0       65001 65102 i
 * >      RD: 10.11.103.0:10050 ip-prefix 192.168.20.0/24
                                 -                     -       100     0       65450 i
 * >      RD: 10.11.103.0:10050 ip-prefix 192.168.40.0/24
                                 -                     -       100     0       65450 i
 * >      RD: 10.11.103.0:12040 ip-prefix 192.168.40.0/24
                                 -                     -       -       0       i
 *        RD: 10.11.103.0:12040 ip-prefix 192.168.40.0/24
                                 -                     -       100     0       65450 i
 * >      RD: 10.11.103.0:10050 ip-prefix 192.168.50.0/24
                                 -                     -       -       0       i
 *        RD: 10.11.103.0:10050 ip-prefix 192.168.50.0/24
                                 -                     -       100     0       65450 i
 * >      RD: 10.11.103.0:12040 ip-prefix 192.168.50.0/24
                                 -                     -       100     0       65450 i
Leaf-03#
```

```
Leaf-03#show bgp evpn route-type ip-prefix 192.168.50.0/24
BGP routing table information for VRF default
Router identifier 10.11.103.0, local AS number 65103
BGP routing table entry for ip-prefix 192.168.50.0/24, Route Distinguisher: 10.11.103.0:10050
 Paths: 2 available
  Local
    - from - (0.0.0.0)
      Origin IGP, metric -, localpref -, weight 0, tag 0, valid, local, best, redistributed (Connected)
      Extended Community: Route-Target-AS:1:10050 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:42:47:8b:8b:f5
      VNI: 10050
  65450
    - from - (0.0.0.0)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external
      Extended Community: Route-Target-AS:1:10050 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:42:47:8b:8b:f5
      VNI: 10050
BGP routing table entry for ip-prefix 192.168.50.0/24, Route Distinguisher: 10.11.103.0:12040
 Paths: 1 available
  65450
    - from - (0.0.0.0)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:1:12040 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:42:47:8b:8b:f5
      VNI: 12040
Leaf-03#
```
```
Leaf-03#show bgp evpn route-type ip-prefix 0.0.0.0/0
BGP routing table information for VRF default
Router identifier 10.11.103.0, local AS number 65103
BGP routing table entry for ip-prefix 0.0.0.0/0, Route Distinguisher: 10.11.103.0:10050
 Paths: 1 available
  65450
    - from - (0.0.0.0)
      Origin INCOMPLETE, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:1:10050 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:42:47:8b:8b:f5
      VNI: 10050
Leaf-03#
```

```
Leaf-03#sh ip route vrf VRF2040

VRF: VRF2040
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

Gateway of last resort is not set

 B E      5.5.5.5/32 [200/0] via 192.168.40.254, Vlan40
 B E      10.10.10.0/30 [200/0] via 192.168.40.254, Vlan40
 B E      10.11.255.0/32 [200/0] via 192.168.40.254, Vlan40
 B E      192.168.20.0/24 [200/0] via VTEP 10.11.102.0 VNI 12040 router-mac 50:60:f5:41:61:d8 local-interface Vxlan1
 C        192.168.40.0/24 is directly connected, Vlan40
 B E      192.168.50.0/24 [200/0] via 192.168.40.254, Vlan40

Leaf-03#
Leaf-03#show ip route vrf VRF50

VRF: VRF50
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

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via 192.168.50.254, Vlan50

 C        5.5.5.5/32 is directly connected, Loopback5
 B E      10.10.10.0/30 [200/0] via 192.168.50.254, Vlan50
 B E      10.11.255.0/32 [200/0] via 192.168.50.254, Vlan50
 B E      192.168.20.0/24 [200/0] via 192.168.50.254, Vlan50
 B E      192.168.40.0/24 [200/0] via 192.168.50.254, Vlan50
 C        192.168.50.0/24 is directly connected, Vlan50

Leaf-03#
```
#### Проверка связи между клиентами
```
VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 192.168.50.2/24
GATEWAY     : 192.168.50.1
DNS         :
MAC         : 00:50:79:66:68:4e
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> ping 192.168.20.2

84 bytes from 192.168.20.2 icmp_seq=1 ttl=61 time=63.272 ms
84 bytes from 192.168.20.2 icmp_seq=2 ttl=61 time=54.268 ms
84 bytes from 192.168.20.2 icmp_seq=3 ttl=61 time=49.266 ms
84 bytes from 192.168.20.2 icmp_seq=4 ttl=61 time=48.956 ms
84 bytes from 192.168.20.2 icmp_seq=5 ttl=61 time=62.142 ms

VPCS> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=63 time=62.085 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=63 time=26.998 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=63 time=17.888 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=63 time=18.242 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=63 time=16.585 ms

VPCS>

VPCS> ping 192.168.40.2

84 bytes from 192.168.40.2 icmp_seq=1 ttl=62 time=101.710 ms
84 bytes from 192.168.40.2 icmp_seq=2 ttl=62 time=37.445 ms
84 bytes from 192.168.40.2 icmp_seq=3 ttl=62 time=39.897 ms
84 bytes from 192.168.40.2 icmp_seq=4 ttl=62 time=30.674 ms
84 bytes from 192.168.40.2 icmp_seq=5 ttl=62 time=28.128 ms

VPCS>

VPCS> trace 192.168.20.2
trace to 192.168.20.2, 8 hops max, press Ctrl+C to stop
 1   192.168.50.1   15.352 ms  16.071 ms  4.702 ms
 2   192.168.50.254   21.966 ms  33.796 ms  44.275 ms
 3   192.168.40.1   40.631 ms  21.647 ms  25.313 ms
 4   192.168.20.1   48.810 ms  62.720 ms  48.707 ms
 5   *192.168.20.2   84.559 ms (ICMP type:3, code:3, Destination port unreachable)

VPCS>
```
