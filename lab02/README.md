# OSPF

#### Цель:
Настроить OSPF.
Разделить сеть на зоны.
Настроить фильтрацию между зонами.

Описание/Пошаговая инструкция выполнения домашнего задания:
Маршрутизаторы BBR1-BBR4 находятся в зоне 0 - backbone.
Маршрутизаторы ACX6-ACX9 находятся в зоне 1.

Использовал образы Juniper VMX, аутентификацию не настраивал

Адресное пространство: 

|Device|Port|IPv4      |   ||  Link           |Comment           |
|------|----|-----------|----------------------------------------|----|------------------|------------------|
|BBR1    |    |10.192.2.1  |                   | | |Management (lo0) |
|      |ge-0.0.0|192.168.100.17 |  ||BBR1 ge-0.0.0 - ge-0.0.1 BBR2 |Connectivity      |
|      |ge-0.0.1|192.168.100.21 |  ||BBR1 ge-0.0.1 - ge-0.0.1 BBR3 |Connectivity      |
|BBR2    |    |10.192.2.2  |     ||          |Management (lo0) |
|      |ge-0.0.1|192.168.100.18 |  ||BBR2 ge-0.0.1 - ge-0.0.0 BBR1 |Connectivity      |
|      |ge-0.0.2|192.168.100.29 |  ||BBR2 ge-0.0.2 - ge-0.0.2 BBR4 |Connectivity      |
|BBR3    |     |10.192.2.3 |                ||     |Management (lo0) |
|      |ge-0.0.1|192.168.100.22 |  ||BBR3 ge-0.0.1 - ge-0.0.1 BBR1 |Connectivity      |
|      |ge-0.0.2|192.168.100.25 |  ||BBR3 ge-0.0.2 - ge-0.0.3 BBR4 |Connectivity      |
|      |ge-0.0.0|192.168.100.33 |  ||BBR3 ge-0.0.3 - ge-0.0.1 ACX9 |Connectivity      |
|BBR4    |    |10.192.2.4 |               ||      |Management (lo0) |
|      |ge-0.0.2|192.168.100.22 |  ||BBR4 ge-0.0.2 - ge-0.0.2 BBR2 |Connectivity      |
|      |ge-0.0.3|192.168.100.26 |  ||BBR4 ge-0.0.3 - ge-0.0.2 BBR3 |Connectivity      |
|      |ge-0.0.1|192.168.100.50 |  ||BBR4 ge-0.0.1 - ge-0.0.0 ACX6 |Connectivity      |
|ACX9    |    |10.192.2.5 |               ||      |Management (lo0) |
|      |ge-0.0.1|192.168.100.34 |  ||ACX9 ge-0.0.1 - ge-0.0.0 BBR3 |Connectivity      |
|      |ge-0.0.0|192.168.100.37|  ||ACX9 ge-0.0.0 - ge-0.0.1 ACX8 |Connectivity      |
|ACX8    |    |10.192.2.6 |               ||      |Management (lo0) |
|      |ge-0.0.1|192.168.100.38 |  ||ACX8 ge-0.0.1 - ge-0.0.0 ACX7 |Connectivity      |
|      |ge-0.0.0|192.168.100.41 |  ||ACX8 ge-0.0.0 - ge-0.0.1 ACX9 |Connectivity      |
|ACX7    |    |10.192.2.7 |               ||      |Management (lo0) |
|      |ge-0.0.1|192.168.100.42 |  ||ACX7 ge-0.0.1 - ge-0.0.0 ACX6 |Connectivity      |
|      |ge-0.0.0|192.168.100.45|  ||ACX7 ge-0.0.0 - ge-0.0.1 ACX8 |Connectivity      |
|ACX6    |    |10.192.2.8 |               ||      |Management (lo0) |


### Конфигурация оборудования: 

```
BBR1

deactivate system syslog user *
set system root-authentication plain-text-password
set system host-name BBR1

set interfaces lo0 unit 0 family inet address 10.192.2.1/32

set interfaces ge-0/0/0 description TO_BBR2_ge-0/0/1
set interfaces ge-0/0/0 enable
set interfaces ge-0/0/0 vlan-tagging
set interfaces ge-0/0/0 mtu 2000
set interfaces ge-0/0/0 link-mode full-duplex
set interfaces ge-0/0/0 encapsulation vlan-ccc
set interfaces ge-0/0/0 unit 0 description TO_BBR2
set interfaces ge-0/0/0 unit 0 vlan-id 0
set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.17/30

set interfaces ge-0/0/1 description TO_BBR3_ge-0/0/1
set interfaces ge-0/0/1 enable
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 mtu 2000
set interfaces ge-0/0/1 link-mode full-duplex
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 0 description TO_BBR3
set interfaces ge-0/0/1 unit 0 vlan-id 0
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.21/30

set protocols ospf area 0.0.0.0 interface lo0.0 passive
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/1.0
set protocols ospf area 0.0.0.0 stub no-summaries 
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0 interface-type p2p

set protocols ospf area 0.0.0.0 stub no-summaries 
set protocols ospf area 0.0.0.0 interface ge-0/0/1.0 interface-type p2p

set interfaces ge-0/0/0 description to_BBR_2
set interfaces ge-0/0/0 unit 0 family inet address 10.10.10.12/31
set interfaces ge-0/0/0 unit 0 family mpls

set interfaces ge-0/0/1 description to_BBR_3
set interfaces ge-0/0/1 unit 0 family inet address 10.10.10.13/31
set interfaces ge-0/0/1 unit 0 family mpls


BBR2

deactivate system syslog user *
set system root-authentication plain-text-password
set system host-name BBR2

set interfaces lo0 unit 0 family inet address 10.192.2.2/32
set interfaces lo0 unit 0 family inet address 169.254.18.204/32

set interfaces ge-0/0/1 description TO_BBR1_ge-0/0/0
set interfaces ge-0/0/1 enable
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 mtu 2000
set interfaces ge-0/0/1 link-mode full-duplex
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 0 description TO_BBR1
set interfaces ge-0/0/1 unit 0 vlan-id 0
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.18/30

set interfaces ge-0/0/2 description TO_BBR4_ge-0/0/2
set interfaces ge-0/0/2 enable
set interfaces ge-0/0/2 vlan-tagging
set interfaces ge-0/0/2 mtu 2000
set interfaces ge-0/0/2 link-mode full-duplex
set interfaces ge-0/0/2 encapsulation vlan-ccc
set interfaces ge-0/0/2 unit 0 description TO_BBR2
set interfaces ge-0/0/2 unit 0 vlan-id 0
set interfaces ge-0/0/2 unit 0 family inet address 192.168.100.29/30

set protocols ospf area 0.0.0.0 interface lo0.0 passive
set protocols ospf area 0.0.0.0 interface ge-0/0/1.0
set protocols ospf area 0.0.0.0 interface ge-0/0/2.0
set protocols ospf area 0.0.0.0 stub no-summaries 
set protocols ospf area 0.0.0.0 interface ge-0/0/1.0 interface-type p2p

set protocols ospf area 0.0.0.0 stub no-summaries 
set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 interface-type p2p


R3

deactivate system syslog user *
set system root-authentication plain-text-password
set system host-name BBR3

set interfaces lo0 unit 0 family inet address 10.192.2.3/32
set interfaces lo0 unit 0 family inet address 169.254.18.205/32

set protocols ospf area 0.0.0.0 interface lo0.0 passive
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0 interface-type p2p
set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 interface-type p2p
set protocols ospf area 0.0.0.1 interface ge-0/0/0.0 interface-type p2p
set protocols ospf area 0.0.0.1 interface ge-0/0/0.0
set protocols ospf area 0.0.0.1 stub no-summaries
set protocols ospf area 0.0.0.0 interface ge-0/0/1.0
set protocols ospf area 0.0.0.0 interface ge-0/0/2.0
set protocols ospf area 0.0.0.0 stub no-summaries 

set interfaces ge-0/0/1 description TO_BBR1_ge-0/0/1
set interfaces ge-0/0/1 enable
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 mtu 2000
set interfaces ge-0/0/1 link-mode full-duplex
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 0 description TO_BBR1
set interfaces ge-0/0/1 unit 0 vlan-id 0
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.22/30

set interfaces ge-0/0/2 description TO_BBR4_ge-0/0/3
set interfaces ge-0/0/2 enable
set interfaces ge-0/0/2 vlan-tagging
set interfaces ge-0/0/2 mtu 2000
set interfaces ge-0/0/2 link-mode full-duplex
set interfaces ge-0/0/2 encapsulation vlan-ccc
set interfaces ge-0/0/2 unit 0 description TO_BBR4
set interfaces ge-0/0/2 unit 0 vlan-id 0
set interfaces ge-0/0/2 unit 0 family inet address 192.168.100.25/30

set interfaces ge-0/0/0 description TO_ACX_9_ge-0/0/1
set interfaces ge-0/0/0 enable
set interfaces ge-0/0/0 vlan-tagging
set interfaces ge-0/0/0 mtu 2000
set interfaces ge-0/0/0 link-mode full-duplex
set interfaces ge-0/0/0 encapsulation vlan-ccc
set interfaces ge-0/0/0 unit 0 description TO_ACX_9
set interfaces ge-0/0/0 unit 0 vlan-id 0
set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.33/30


R4

deactivate system syslog user *
set system root-authentication plain-text-password
set system host-name BBR4

set interfaces lo0 unit 0 family inet address 10.192.2.4/32
set interfaces lo0 unit 0 family inet address 169.254.18.206/32

set protocols ospf area 0.0.0.0 interface lo0.0 passive
set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 interface-type p2p
set protocols ospf area 0.0.0.0 interface ge-0/0/3.0 interface-type p2p
set protocols ospf area 0.0.0.1 interface ge-0/0/1.0 interface-type p2p
set protocols ospf area 0.0.0.1 interface ge-0/0/1.0
set protocols ospf area 0.0.0.1 stub no-summaries 
set protocols ospf area 0.0.0.0 interface ge-0/0/2.0
set protocols ospf area 0.0.0.0 interface ge-0/0/3.0
set protocols ospf area 0.0.0.0 stub no-summaries 

set interfaces ge-0/0/2 description TO_BBR2_ge-0/0/1
set interfaces ge-0/0/2 enable
set interfaces ge-0/0/2 vlan-tagging
set interfaces ge-0/0/2 mtu 2000
set interfaces ge-0/0/2 link-mode full-duplex
set interfaces ge-0/0/2 encapsulation vlan-ccc
set interfaces ge-0/0/2 unit 0 description TO_BBR2
set interfaces ge-0/0/2 unit 0 vlan-id 0
set interfaces ge-0/0/2 unit 0 family inet address 192.168.100.22/30

set interfaces ge-0/0/3 description TO_BBR3_ge-0/0/2
set interfaces ge-0/0/3 enable
set interfaces ge-0/0/3 vlan-tagging
set interfaces ge-0/0/3 mtu 2000
set interfaces ge-0/0/3 link-mode full-duplex
set interfaces ge-0/0/3 encapsulation vlan-ccc
set interfaces ge-0/0/3 unit 0 description TO_BBR3
set interfaces ge-0/0/3 unit 0 vlan-id 0
set interfaces ge-0/0/3 unit 0 family inet address 192.168.100.26/30

set interfaces ge-0/0/1 description TO_ACX_6_ge-0/0/0
set interfaces ge-0/0/1 enable
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 mtu 2000
set interfaces ge-0/0/1 link-mode full-duplex
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 0 description TO_ACX6
set interfaces ge-0/0/1 unit 0 vlan-id 0
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.50/30


ACX9

deactivate system syslog user *
set system root-authentication plain-text-password
set system host-name ACX9

set interfaces lo0 unit 0 family inet address 10.192.2.5/32
set interfaces lo0 unit 0 family inet address 169.254.18.207/32

set interfaces ge-0/0/1 description TO_BBR3_ge-0/0/0
set interfaces ge-0/0/1 enable
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 mtu 2000
set interfaces ge-0/0/1 link-mode full-duplex
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 0 description TO_BBR3
set interfaces ge-0/0/1 unit 0 vlan-id 0
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.34/30

set interfaces ge-0/0/0 description TO_ACX8_ge-0/0/1
set interfaces ge-0/0/0 enable
set interfaces ge-0/0/0 vlan-tagging
set interfaces ge-0/0/0 mtu 2000
set interfaces ge-0/0/0 link-mode full-duplex
set interfaces ge-0/0/0 encapsulation vlan-ccc
set interfaces ge-0/0/0 unit 0 description TO_ACX8
set interfaces ge-0/0/0 unit 0 vlan-id 0
set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.37/30

set interface lo0 unit 0 family inet address 169.254.18.207
set protocols ospf area 0.0.0.1 stub no-summaries 
set protocols ospf area 0.0.0.1 interface ge-0/0/1.0 interface-type p2p
set protocols ospf area 0.0.0.1 interface ge-0/0/0.0 interface-type p2p
set protocols ospf area 0.0.0.1 interface lo0.0 passive


ACX8

deactivate system syslog user *
set system root-authentication plain-text-password
set system host-name ACX8

set interfaces lo0 unit 0 family inet address 10.192.2.6/32
set interfaces lo0 unit 0 family inet address 169.254.18.208/32

set interfaces ge-0/0/1 description TO_ACX9_ge-0/0/0
set interfaces ge-0/0/1 enable
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 mtu 2000
set interfaces ge-0/0/1 link-mode full-duplex
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 0 description TO_ACX9
set interfaces ge-0/0/1 unit 0 vlan-id 0
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.38/30

set interfaces ge-0/0/0 description TO_ACX7_ge-0/0/1
set interfaces ge-0/0/0 enable
set interfaces ge-0/0/0 vlan-tagging
set interfaces ge-0/0/0 mtu 2000
set interfaces ge-0/0/0 link-mode full-duplex
set interfaces ge-0/0/0 encapsulation vlan-ccc
set interfaces ge-0/0/0 unit 0 description TO_ACX7
set interfaces ge-0/0/0 unit 0 vlan-id 0
set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.41/30

set interface lo0 unit 0 family inet address 169.254.18.207
set protocols ospf area 0.0.0.1 stub no-summaries 
set protocols ospf area 0.0.0.1 interface ge-0/0/1.0 interface-type p2p
set protocols ospf area 0.0.0.1 interface ge-0/0/0.0 interface-type p2p
set protocols ospf area 0.0.0.1 interface lo0.0 passive


ACX7

deactivate system syslog user *
set system root-authentication plain-text-password
set system host-name ACX7

set interfaces lo0 unit 0 family inet address 10.192.2.7/32
set interfaces lo0 unit 0 family inet address 169.254.18.209/32

set interfaces ge-0/0/1 description TO_ACX8_ge-0/0/0
set interfaces ge-0/0/1 enable
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 mtu 2000
set interfaces ge-0/0/1 link-mode full-duplex
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 0 description TO_ACX8
set interfaces ge-0/0/1 unit 0 vlan-id 0
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.42/30

set interfaces ge-0/0/0 description TO_ACX6_ge-0/0/1
set interfaces ge-0/0/0 enable
set interfaces ge-0/0/0 vlan-tagging
set interfaces ge-0/0/0 mtu 2000
set interfaces ge-0/0/0 link-mode full-duplex
set interfaces ge-0/0/0 encapsulation vlan-ccc
set interfaces ge-0/0/0 unit 0 description TO_ACX6
set interfaces ge-0/0/0 unit 0 vlan-id 0
set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.45/30

set interface lo0 unit 0 family inet address 169.254.18.209
set protocols ospf area 0.0.0.1 stub no-summaries 
set protocols ospf area 0.0.0.1 interface ge-0/0/1.0 interface-type p2p
set protocols ospf area 0.0.0.1 interface ge-0/0/0.0 interface-type p2p
set protocols ospf area 0.0.0.1 interface lo0.0 passive


ACX6

deactivate system syslog user *
set system root-authentication plain-text-password
set system host-name ACX6

set interfaces lo0 unit 0 family inet address 10.192.2.8/32
set interfaces lo0 unit 0 family inet address 169.254.18.210/32

set interfaces ge-0/0/1 description TO_ACX7_ge-0/0/0
set interfaces ge-0/0/1 enable
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 mtu 2000
set interfaces ge-0/0/1 link-mode full-duplex
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 0 description TO_ACX7
set interfaces ge-0/0/1 unit 0 vlan-id 0
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.46/30

set interfaces ge-0/0/0 description TO_BBR4_ge-0/0/1
set interfaces ge-0/0/0 enable
set interfaces ge-0/0/0 vlan-tagging
set interfaces ge-0/0/0 mtu 2000
set interfaces ge-0/0/0 link-mode full-duplex
set interfaces ge-0/0/0 encapsulation vlan-ccc
set interfaces ge-0/0/0 unit 0 description TO_BBR4
set interfaces ge-0/0/0 unit 0 vlan-id 0
set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.49/30

set interface lo0 unit 0 family inet address 169.254.18.210
set protocols ospf area 0.0.0.1 stub no-summaries 
set protocols ospf area 0.0.0.1 interface ge-0/0/1.0 interface-type p2p
set protocols ospf area 0.0.0.1 interface ge-0/0/0.0 interface-type p2p
set protocols ospf area 0.0.0.1 interface lo0.0 passive

```




|      |ge-0.0.1|192.168.100.46 |  ||ACX6 ge-0.0.1 - ge-0.0.0 ACX6 |Connectivity      |
|      |ge-0.0.0|192.168.100.49 |  ||ACX6 ge-0.0.0 - ge-0.0.1 ACX8 |Connectivity      |


