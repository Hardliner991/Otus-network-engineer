### VXLAN. Multihoming.

### Цель

Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming

### Задание: 

1. Подключите клиентов 2-я линками к различным Leaf
2. Настроите агрегированный канал со стороны клиента
3. Настроите multihoming для работы в Overlay сети. Если используете Cisco NXOS - vPC, если иной вендор - то ESI LAG (либо MC-LAG с поддержкой VXLAN)

### Схема:













###Конфиг: 


```
! device: spine-1 (vEOS-lab, EOS-4.28.0F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname spine-1
!
spanning-tree mode mstp
!
interface Ethernet1
   description to-leaf-1
   no switchport
   ip address 192.4.1.6/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet2
   description to-leaf-2
   no switchport
   ip address 192.4.1.2/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet3
   description to-leaf-3
   no switchport
   ip address 192.4.1.4/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback1
   ip address 192.1.1.0/32
!
interface Management1
!
ip routing
!
ip prefix-list PL_LOOP
   seq 10 permit 192.1.1.0/32
   seq 20 permit 192.100.0.0/24
!
route-map RM_CONN permit 10
   match ip address prefix-list PL_LOOP
!
peer-filter EVPN
   10 match as-range 65001-65003 result accept
!
peer-filter LEAF
   10 match as-range 65001-65003 result accept
!
router bgp 65000
   router-id 192.1.1.0
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 192.1.0.0/16 peer-group EVPN peer-filter EVPN
   bgp listen range 192.4.0.0/22 peer-group LEAF peer-filter LEAF
   neighbor EVPN peer group
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN route-reflector-client
   neighbor EVPN send-community extended
   neighbor EVPN maximum-routes 12000
   neighbor LEAF peer group
   neighbor LEAF bfd
   neighbor LEAF rib-in pre-policy retain all
   neighbor LEAF send-community
   neighbor LEAF maximum-routes 1000
   redistribute connected route-map RM_CONN
   !
   vlan 100
      route-target both 100:100
   !
   vlan 200
      route-target both 200:200
   !
   address-family evpn
      neighbor EVPN activate
   !
   vrf GAGA
      route-target import evpn 999:999
      route-target export evpn 999:999
!
end
```
```
! Command: show running-config
! device: spine-2 (vEOS-lab, EOS-4.28.0F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname spine-2
!
spanning-tree mode mstp
!
interface Ethernet1
   description to-leaf-1
   no switchport
   ip address 192.4.2.3/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet2
   description to-leaf-2
   no switchport
   ip address 192.4.1.5/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet3
   description to leaf-3
   no switchport
   ip address 192.4.1.7/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback1
   ip address 192.1.2.0/32
!
interface Management1
!
ip routing
!
ip prefix-list PL_LOOP
   seq 10 permit 192.1.2.0/32
!
route-map RM_CONN permit 10
   match ip address prefix-list PL_LOOP
!
peer-filter EVPN
   10 match as-range 65001-65003 result accept
!
peer-filter LEAF
   10 match as-range 65001-65003 result accept
!
router bgp 65000
   router-id 192.1.2.0
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 192.1.0.0/16 peer-group EVPN peer-filter EVPN
   bgp listen range 192.4.0.0/22 peer-group LEAF peer-filter LEAF
   neighbor EVPN peer group
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor EVPN maximum-routes 12000
   neighbor LEAF peer group
   neighbor LEAF bfd
   neighbor LEAF rib-in pre-policy retain all
   neighbor LEAF send-community
   neighbor LEAF maximum-routes 1000
   redistribute connected route-map RM_CONN
   !
   vlan 100
      route-target both 100:100
   !
   vlan 200
      route-target both 200:200
   !
   address-family evpn
      neighbor EVPN activate
   !
   vrf GAGA
      route-target import evpn 999:999
      route-target export evpn 999:999
!
end

```
```
! Command: show running-config
! device: leaf1 (vEOS-lab, EOS-4.28.0F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname leaf1
!
spanning-tree mode mstp
no spanning-tree vlan-id 4094
!
vlan 100,200
!
vlan 4094
   trunk group mlag
!
vrf instance GAGA
!
interface Port-Channel1
   description peerlink
   switchport mode trunk
   switchport trunk group mlag
!
interface Port-Channel2
   description to-srv-mlag
   switchport trunk allowed vlan 100,200
   switchport mode trunk
   mlag 2
!
interface Ethernet1
   description to-spine-1
   no switchport
   ip address 192.4.1.7/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet2
   description to-spine-2
   no switchport
   ip address 192.4.2.2/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet3
   description to-leaf2
   channel-group 1 mode active
!
interface Ethernet4
   description to-leaf2
   channel-group 1 mode active
!
interface Ethernet5
   description to-SRV
   channel-group 2 mode active
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback1
   ip address 192.1.0.1/32
!
interface Loopback100
   description lo100
   ip address 192.100.0.1/32
!
interface Management1
!
interface Vlan100
   vrf GAGA
   ip address virtual 10.10.2.254/24
!
interface Vlan200
   vrf GAGA
   ip address virtual 10.10.3.253/24
!
interface Vlan4094
   ip address 10.10.100.0/31
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 100 vni 100100
   vxlan vlan 200 vni 100200
   vxlan vrf GAGA vni 999
   vxlan virtual-vtep local-interface Loopback100
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf GAGA
!
ip prefix-list PL_LOOP
   seq 10 permit 192.1.0.1/32
   seq 20 permit 192.100.0.1/32
!
mlag configuration
   domain-id mlag1
   local-interface Vlan4094
   peer-address 10.10.100.1
   peer-link Port-Channel1
!
route-map RM_CONN permit 10
   match ip address prefix-list PL_LOOP
!
peer-filter EVPN
   10 match as-range 65000-65003 result accept
!
router bgp 65001
   router-id 192.1.0.1
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 192.1.0.0/22 peer-group EVPN peer-filter EVPN
   neighbor EVPN peer group
   neighbor EVPN remote-as 65000
   no neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE allowas-in 1
   neighbor SPINE rib-in pre-policy retain all
   neighbor SPINE send-community
   neighbor SPINE maximum-routes 1000
   neighbor 192.1.1.0 peer group EVPN
   neighbor 192.1.1.0 remote-as 65000
   neighbor 192.1.2.0 peer group EVPN
   neighbor 192.1.2.0 remote-as 65000
   neighbor 192.4.1.6 peer group SPINE
   neighbor 192.4.2.2 peer group SPINE
   no neighbor 192.4.2.2 remote-as
   neighbor 192.4.2.2 maximum-routes 256000 warning-limit 80 percent
   neighbor 192.4.2.3 peer group SPINE
   redistribute connected route-map RM_CONN
   !
   vlan 100
      rd 65001:100100
      route-target both 100:100
      redistribute learned
   !
   vlan 200
      rd 65001:100200
      route-target both 200:200
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   vrf GAGA
      rd 65001:999
      route-target import evpn 999:999
      route-target export evpn 999:999
      redistribute connected
!
end

```

```
! Command: show running-config
! device: leaf2 (vEOS-lab, EOS-4.27.3F)! Command: show running-config
! device: leaf2 (vEOS-lab, EOS-4.28.0F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname leaf2
!
spanning-tree mode mstp
no spanning-tree vlan-id 4094
!
vlan 100,200
!
vlan 4094
   trunk group mlag
!
vrf instance GAGA
!
interface Port-Channel1
   description peerlink
   switchport mode trunk
   switchport trunk group mlag
!
interface Port-Channel2
   description to-SRV
   switchport trunk allowed vlan 100,200
   switchport mode trunk
   mlag 2
!
interface Ethernet1
   description to-spine-1
   no switchport
   ip address 192.4.1.3/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet2
   description to-spine-2
   no switchport
   ip address 192.4.1.4/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet3
   description to-leaf1
   channel-group 1 mode active
!
interface Ethernet4
   description to-leaf1
   channel-group 1 mode active
!
interface Ethernet5
   description to-SRV
   channel-group 2 mode active
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback2
   ip address 192.1.0.2/32
!
interface Loopback100
   description Lo100
   ip address 192.100.0.2/32
!
interface Management1
!
interface Vlan100
   vrf GAGA
   ip address virtual 10.10.2.253/24
!
interface Vlan200
   ip address virtual 10.10.3.254/24
!
interface Vlan4094
   ip address 10.10.100.1/31
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 100 vni 100100
   vxlan vlan 200 vni 100200
   vxlan vrf GAGA vni 999
   vxlan virtual-vtep local-interface Loopback100
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
ip routing vrf GAGA
!
ip prefix-list PL_LOOP
   seq 10 permit 192.1.0.2/32
   seq 20 permit 192.100.0.2/32
!
mlag configuration
   domain-id mlag1
   local-interface Vlan4094
   peer-address 10.10.100.0
   peer-link Port-Channel1
!
route-map RM_CONN permit 10
   match ip address prefix-list PL_LOOP
!
router bgp 65002
   router-id 192.1.0.2
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65000
   no neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback2
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE allowas-in 1
   neighbor SPINE rib-in pre-policy retain all
   neighbor SPINE send-community
   neighbor SPINE maximum-routes 1000
   neighbor 192.1.1.0 peer group EVPN
   neighbor 192.1.1.0 remote-as 65000
   neighbor 192.1.2.0 peer group EVPN
   neighbor 192.1.2.0 remote-as 65000
   neighbor 192.4.1.2 peer group SPINE
   neighbor 192.4.1.5 peer group SPINE
   redistribute connected route-map RM_CONN
   !
   vlan 100
      rd 65002:100100
      route-target both 100:100
      redistribute learned
   !
   vlan 200
      rd 65002:100200
      route-target both 200:200
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   vrf GAGA
      rd 65002:999
      route-target import evpn 999:999
      route-target export evpn 999:999
      redistribute connected
!
end

```
```
! Command: show running-config
! device: leaf-3 (vEOS-lab, EOS-4.28.0F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname leaf-3
!
spanning-tree mode mstp
!
vlan 10,20,25,100,200
!
vrf instance GAGA
!
interface Ethernet1
   description to-spine-1
   no switchport
   ip address 192.4.1.5/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet2
   description to-spine-2
   no switchport
   ip address 192.4.1.6/31
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet3
   description to_vpc11
   switchport access vlan 25
   no switchport
   bfd interval 100 min-rx 100 multiplier 3
!
interface Ethernet4
   switchport access vlan 25
!
interface Ethernet5
   switchport access vlan 25
!
interface Ethernet6
   description to-vpc
   switchport access vlan 100
!
interface Ethernet7
   description to-vpc
   switchport access vlan 200
!
interface Ethernet8
!
interface Loopback2
   ip address 192.1.0.3/32
!
interface Loopback100
   description Lo100
   ip address 192.100.0.3/32
!
interface Management1
!
interface Vlan25
   vrf GAGA
   ip address virtual 10.10.77.253/24
!
interface Vlan100
   description to-vpc
   vrf GAGA
   ip address virtual 10.10.2.252/24
!
interface Vlan200
   vrf GAGA
   ip address virtual 10.10.3.252/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 25 vni 10025
   vxlan vlan 100 vni 100100
   vxlan vlan 200 vni 100200
   vxlan vrf GAGA vni 999
   vxlan virtual-vtep local-interface Loopback100
!
ip virtual-router mac-address 00:00:00:00:00:03
!
ip routing
ip routing vrf GAGA
!
ip prefix-list PL_LOOP
   seq 10 permit 192.1.0.3/32
   seq 20 permit 10.10.77.0/24
!
route-map RM_CONN permit 10
   match ip address prefix-list PL_LOOP
!
router bgp 65003
   router-id 192.1.0.3
   timers bgp 3 9
   bgp listen range 192.1.0.0/24 peer-group EVPN peer-filter EVPN
   neighbor EVPN peer group
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback2
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor EVPN maximum-routes 12000
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE allowas-in 1
   neighbor SPINE send-community
   neighbor SPINE maximum-routes 1000
   neighbor 192.1.1.0 peer group EVPN
   neighbor 192.1.1.0 remote-as 65000
   neighbor 192.1.2.0 peer group EVPN
   neighbor 192.1.2.0 remote-as 65000
   neighbor 192.4.1.4 peer group SPINE
   neighbor 192.4.1.7 peer group SPINE
   neighbor 192.4.1.7 maximum-routes 12000
   redistribute connected route-map RM_CONN
   !
   vlan 100
      rd 65003:100100
      route-target both 100:100
      redistribute learned
   !
   vlan 200
      rd 65003:100200
      route-target both 200:200
      redistribute learned
   !
   vlan 25
      rd 65003:100025
      route-target both 25:25
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   vrf GAGA
      rd 65003:999
      route-target import evpn 999:999
      route-target export evpn 999:999
!
end
```
```
! Command: show running-config
! device: localhost (vEOS-lab, EOS-4.28.0F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
spanning-tree mode mstp
!
vlan 100,200
!
interface Port-Channel2
   switchport trunk allowed vlan 100,200
   switchport mode trunk
!
interface Ethernet1
   channel-group 2 mode active
!
interface Ethernet2
   channel-group 2 mode active
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Management1
!
interface Vlan100
   ip address 10.10.2.10/24
!
interface Vlan200
   ip address 10.10.3.10/24
!
ip routing
!
end

```















