![image](https://github.com/user-attachments/assets/c0abedce-ea13-4b25-81c3-3bbe72496b17)### Цель:

Настроить маршрутизацию в рамках Overlay между клиентами


###Задание: 

Настроите каждого клиента в своем VNI
Настроите маршрутизацию между клиентами.
Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств

### Схема: 

![](https://github.com/Hardliner991/Otus-network-engineer/blob/main/Lab06/lab6topo.png)

### Конфиги устройств: 


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
vlan 10,20,228
!
vrf definition GAGA
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
   description to_vpc6
   switchport access vlan 228
!
interface Ethernet4
   description to_vpc10
   switchport access vlan 228
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
interface Loopback100
   description Lo100
   ip address 192.100.0.1/32
!
interface Management1
!
interface Vlan228
   vrf forwarding GAGA
   ip address virtual 10.10.12.254/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 66610
   vxlan vlan 20 vni 66620
   vxlan vlan 228 vni 10228
   vxlan vrf GAGA vni 999
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf GAGA
!
ip prefix-list LOOPBACK
   seq 10 permit 192.1.0.1/32
   seq 20 permit 192.1.0.0/24 le 32
!
ip prefix-list LOOPBACK_LEAF
   seq 10 permit 192.1.0.0/24 le 32
   seq 20 permit 10.10.12.0/24
!
ip prefix-list PL_LOOP
   seq 10 permit 192.1.0.1/32
   seq 20 permit 192.100.0.1/32
!
route-map RM_CONN permit 10
   match ip address prefix-list LOOPBACK_LEAF
!
peer-filter EVPN
   10 match as-range 65000-65003 result accept
!
router bgp 65001
   router-id 192.1.0.1
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
   vlan 228
      rd 10228:10228
      route-target both 228:228
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   vrf GAGA
      rd 65001:999
      route-target import evpn 999:999
      route-target export evpn 999:999
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
vlan 10,20,228,777
!
vrf definition GAGA
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
   switchport access vlan 228
!
interface Ethernet4
   description to_vpc7
   switchport access vlan 777
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
interface Loopback100
   description Lo100
   ip address 192.100.0.2/32
!
interface Management1
!
interface Vlan228
   vrf forwarding GAGA
   ip address virtual 10.10.12.253/24
!
interface Vlan777
   ip address virtual 10.10.77.254/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 66610
   vxlan vlan 20 vni 66620
   vxlan vlan 228 vni 10228
   vxlan vlan 777 vni 10777
   vxlan vrf GAGA vni 999
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
no ip routing vrf VRF_VLAN10
no ip routing vrf VRF_VLAN20
ip routing vrf GAGA
!
ip prefix-list PL_LOOP
   seq 10 permit 192.1.0.2/32
   seq 20 permit 192.100.0.2/32
   seq 30 permit 10.10.77.0/24
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
   vlan 228
      rd 10228:10228
      route-target both 228:228
      redistribute learned
   !
   vlan 777
      rd 65002:10777
      route-target both 777:777
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   vrf GAGA
      rd 65002:999
      route-target import evpn 999:999
      route-target export evpn 999:999
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
vlan 10,20,25
!
vrf definition GAGA
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
   switchport access vlan 25
   no switchport
   bfd interval 100 min_rx 100 multiplier 3
!
interface Ethernet4
   switchport access vlan 25
!
interface Ethernet5
   switchport access vlan 25
!
interface Ethernet6
   description to_vpc8
   switchport access vlan 25
!
interface Ethernet7
   description to_vpc7
   switchport access vlan 25
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
   vrf forwarding GAGA
   ip address virtual 10.10.77.253/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 25 vni 10025
   vxlan vrf GAGA vni 999
   vxlan learn-restrict any
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

### Вывод: 
![](https://github.com/Hardliner991/Otus-network-engineer/blob/main/Lab06/Leaf1.png)


 
