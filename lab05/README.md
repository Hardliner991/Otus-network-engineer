### VxLAN. L2 VNI

### Цель:

Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами

### Схема: 

![](https://github.com/Hardliner991/Otus-network-engineer/blob/main/lab04/ebgp%20topology.png)

### Конфиг устройств: 

```
! Command: show running-config
! device: spine-1 (vEOS, EOS-4.21.1.1F)
!
! boot system flash:/vEOS-lab.swi
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname spine-1
!
spanning-tree mode mstp
!
no aaa root
!
interface Ethernet1
   description to-leaf-1
   no switchport
   ip address 192.4.1.0/31
   bfd interval 100 min_rx 100 multiplier 3
!
interface Ethernet2
   description to-leaf-2
   no switchport
   ip address 192.4.1.3/31
   bfd interval 100 min_rx 100 multiplier 3
!
interface Ethernet3
   description to-leaf-3
   no switchport
   ip address 192.4.1.4/31
   bfd interval 100 min_rx 100 multiplier 3
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
ip prefix-list LOOPBACK seq 10 permit 192.1.1.0/32
ip prefix-list LOOPBACK seq 20 permit 192.1.0.0/24 le 32
!
route-map RM_CONN permit 10
   match ip address prefix-list LOOPBACK
!
peer-filter EVPN
   10 match as-range 65001-65003 result accept
!
peer-filter LEAF
   10 match as-range 65000-65003 result accept
!
router bgp 65000
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 192.1.0.0/23 peer-group EVPN peer-filter EVPN
   bgp listen range 192.4.0.0/20 peer-group LEAF peer-filter LEAF
   neighbor EVPN peer-group
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor EVPN maximum-routes 12000
   neighbor LEAF peer-group
   neighbor LEAF send-community
   neighbor LEAF maximum-routes 1000
   neighbor 192.1.0.1 peer-group EVPN
   neighbor 192.1.0.1 remote-as 65001
   neighbor 192.1.0.2 peer-group EVPN
   neighbor 192.1.0.2 remote-as 65002
   neighbor 192.1.0.3 peer-group EVPN
   neighbor 192.1.0.3 remote-as 65003
   neighbor 192.4.1.1 peer-group LEAF
   neighbor 192.4.1.1 remote-as 65001
   neighbor 192.4.1.2 remote-as 65002
   neighbor 192.4.1.2 maximum-routes 12000
   neighbor 192.4.1.5 remote-as 65003
   neighbor 192.4.1.5 maximum-routes 12000
   redistribute connected route-map RM_CONN
   !
   vlan 10
      rd 65001:66610
      route-target both 10:66610
   !
   vlan 20
      rd 65001:66620
      route-target both 20:66620
   !
   address-family evpn
      neighbor EVPN activate
!
end
```

```
! Command: show running-config
! device: spine-2 (vEOS, EOS-4.21.1.1F)
!
! boot system flash:/vEOS-lab.swi
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname spine-2
!
spanning-tree mode mstp
!
no aaa root
!
interface Ethernet1
   description to-leaf-1
   no switchport
   ip address 192.4.2.2/31
   bfd interval 100 min_rx 100 multiplier 3
!
interface Ethernet2
   description to-leaf-2
   no switchport
   ip address 192.4.2.8/31
   bfd interval 100 min_rx 100 multiplier 3
!
interface Ethernet3
   description to-leaf-3
   no switchport
   ip address 192.4.1.7/31
   bfd interval 100 min_rx 100 multiplier 3
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
peer-filter LEAF
   10 match as-range 65001-65003 result accept
!
router bgp 65000
   router-id 192.1.2.0
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 192.1.0.0/24 peer-group EVPN peer-filter EVPN
   bgp listen range 192.4.0.0/20 peer-group LEAF peer-filter LEAF
   neighbor EVPN peer-group
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor EVPN maximum-routes 12000
   neighbor LEAF peer-group
   neighbor LEAF send-community
   neighbor LEAF maximum-routes 1000
   neighbor 192.1.0.1 peer-group EVPN
   neighbor 192.1.0.1 remote-as 65001
   neighbor 192.1.0.2 peer-group EVPN
   neighbor 192.1.0.2 remote-as 65002
   neighbor 192.1.0.3 peer-group EVPN
   neighbor 192.1.0.3 remote-as 65003
   neighbor 192.4.1.1 peer-group LEAF
   no neighbor 192.4.1.1 remote-as
   neighbor 192.4.1.6 remote-as 65003
   neighbor 192.4.1.6 maximum-routes 12000
   neighbor 192.4.2.3 remote-as 65001
   neighbor 192.4.2.3 maximum-routes 12000
   neighbor 192.4.2.9 remote-as 65002
   neighbor 192.4.2.9 maximum-routes 12000
   redistribute connected route-map RM_CONN
   !
   address-family evpn
      neighbor EVPN activate
   !
   address-family ipv4
      neighbor LEAF activate
!
end

```
```
! Command: show running-config
! device: leaf1 (vEOS, EOS-4.21.1.1F)
!
! boot system flash:/vEOS-lab.swi
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname leaf1
!
spanning-tree mode mstp
!
no aaa root
!
vlan 10,20
!
interface Ethernet1
   description to-spine-1
   no switchport
   ip address 192.4.1.1/31
   bfd interval 100 min_rx 100 multiplier 3
!
interface Ethernet2
   description to-spine-2
   no switchport
   ip address 192.4.2.3/31
   bfd interval 100 min_rx 100 multiplier 3
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
interface Loopback1
   ip address 192.1.0.1/32
!
interface Management1
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 66610
   vxlan vlan 20 vni 66620
!
ip routing
!
ip prefix-list LOOPBACK seq 10 permit 192.1.0.1/32
ip prefix-list LOOPBACK seq 20 permit 192.1.0.0/24 le 32
ip prefix-list LOOPBACK_LEAF seq 10 permit 192.1.0.0/24 le 32
!
route-map RM_CONN permit 10
   match ip address prefix-list LOOPBACK_LEAF
!
peer-filter EVPN
   10 match as-range 65000-65003 result accept
!
router bgp 65001
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 192.1.0.0/22 peer-group EVPN peer-filter EVPN
   neighbor EVPN peer-group
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor EVPN maximum-routes 12000
   neighbor SPINE peer-group
   neighbor SPINE remote-as 65000
   neighbor SPINE allowas-in 1
   neighbor SPINE send-community
   neighbor SPINE maximum-routes 1000
   neighbor 192.1.1.0 peer-group EVPN
   neighbor 192.1.1.0 remote-as 65000
   neighbor 192.1.2.0 peer-group EVPN
   neighbor 192.1.2.0 remote-as 65000
   neighbor 192.4.1.0 peer-group SPINE
   neighbor 192.4.1.0 remote-as 65000
   neighbor 192.4.1.0 maximum-routes 12000
   neighbor 192.4.2.2 peer-group SPINE
   neighbor 192.4.2.2 remote-as 65000
   neighbor 192.4.2.2 maximum-routes 12000
   redistribute connected route-map RM_CONN
   !
   vlan 10
      rd 65001:66610
      route-target both 10:66610
      redistribute learned
   !
   vlan 20
      rd 65001:66620
      route-target both 20:66620
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
!
end
```
```
! Command: show running-config
! device: leaf2 (vEOS, EOS-4.21.1.1F)
!
! boot system flash:/vEOS-lab.swi
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname leaf2
!
spanning-tree mode mstp
!
no aaa root
!
vlan 10,20
!
vrf definition VRF_VLAN10
   rd 65002:66610
!
vrf definition VRF_VLAN20
   rd 65002:66620
!
interface Ethernet1
   description to-spine-1
   no switchport
   ip address 192.4.1.2/31
   bfd interval 100 min_rx 100 multiplier 3
!
interface Ethernet2
   description to-spine-2
   no switchport
   ip address 192.4.2.9/31
!
interface Ethernet3
   description to_vpc11
   switchport access vlan 10
!
interface Ethernet4
   description to_vpc7
   switchport access vlan 20
!
interface Ethernet5
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
interface Management1
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 66610
   vxlan vlan 20 vni 66620
   vxlan learn-restrict any
!
ip routing
no ip routing vrf VRF_VLAN10
no ip routing vrf VRF_VLAN20
!
ip prefix-list PL_LOOP
   seq 10 permit 192.1.0.2/32
   seq 20 permit 192.100.0.2/32
!
route-map RM_CONN permit 10
   match ip address prefix-list PL_LOOP
!
router bgp 65002
   router-id 192.1.0.2
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 192.1.0.0/24 peer-group EVPN peer-filter EVPN
   neighbor EVPN peer-group
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback2
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor EVPN maximum-routes 12000
   neighbor SPINE peer-group
   neighbor SPINE remote-as 65000
   neighbor SPINE allowas-in 1
   neighbor SPINE send-community
   neighbor SPINE maximum-routes 1000
   neighbor 192.1.1.0 peer-group EVPN
   neighbor 192.1.1.0 remote-as 65000
   neighbor 192.1.2.0 peer-group EVPN
   neighbor 192.1.2.0 remote-as 65000
   neighbor 192.4.1.3 remote-as 65000
   neighbor 192.4.1.3 maximum-routes 12000
   neighbor 192.4.2.8 remote-as 65000
   neighbor 192.4.2.8 maximum-routes 12000
   redistribute connected route-map RM_CONN
   !
   vlan 10
      rd 65002:66610
      route-target both 10:66610
      redistribute learned
   !
   vlan 20
      rd 65002:66620
      route-target both 20:66620
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
!
end
```
```
! Command: show running-config
! device: leaf-3 (vEOS, EOS-4.21.1.1F)
!
! boot system flash:/vEOS-lab.swi
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname leaf-3
!
spanning-tree mode mstp
!
no aaa root
!
vlan 10,20
!
interface Ethernet1
   description to-spine-1
   no switchport
   ip address 192.4.1.5/31
   bfd interval 100 min_rx 100 multiplier 3
!
interface Ethernet2
   description to-spine-2
   no switchport
   ip address 192.4.1.6/31
   bfd interval 100 min_rx 100 multiplier 3
!
interface Ethernet3
   description to_vpc11
   switchport access vlan 10
   no switchport
   ip address 192.4.2.6/31
   bfd interval 100 min_rx 100 multiplier 3
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
interface Loopback2
   ip address 192.1.0.3/32
!
interface Management1
!
ip routing
!
ip prefix-list PL_LOOP
   seq 10 permit 192.1.0.3/32
!
route-map RM_CONN permit 10
   match ip address prefix-list PL_LOOP
!
router bgp 65003
   router-id 192.1.0.3
   timers bgp 3 9
   bgp listen range 192.1.0.0/24 peer-group EVPN peer-filter EVPN
   neighbor EVPN peer-group
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback2
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor EVPN maximum-routes 12000
   neighbor SPINE peer-group
   neighbor SPINE remote-as 65000
   neighbor SPINE allowas-in 1
   neighbor SPINE send-community
   neighbor SPINE maximum-routes 1000
   neighbor 192.1.1.0 peer-group EVPN
   neighbor 192.1.1.0 remote-as 65000
   neighbor 192.1.2.0 peer-group EVPN
   neighbor 192.1.2.0 remote-as 65000
   neighbor 192.4.1.4 remote-as 65000
   neighbor 192.4.1.4 maximum-routes 12000
   neighbor 192.4.1.7 remote-as 65000
   neighbor 192.4.1.7 maximum-routes 12000
   redistribute connected route-map RM_CONN
   !
   address-family evpn
      neighbor EVPN activate
!
end
``` 
### Вывод:

![](https://github.com/Hardliner991/Otus-network-engineer/blob/main/lab05/evpn05.png)




