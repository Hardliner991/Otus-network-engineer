# IS-IS

#### Цель:
Настроить IS-IS .

### Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

Настроите ISIS в Underlay сети, для IP связанности между всеми сетевыми устройствами.
Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
Убедитесь в наличии IP связанности между устройствами в ISIS домене

Использовал образы Juniper VMX, аутентификацию не настраивал

### IS-IS (Intermediate System to Intermediate System)

Как и OSPF это Link-State на основе Дейкстры.
Это тоже IGP протокол.
В крупных провайдерах любят IS-IS так как он **хорошо интегрируется с MPLS**.

OSPF - работает поверх IP.
**IS-IS - работает поверх Ethernet v.1**, из-за этого сложно использовать его для построения VPN.
Так как VPN - это по сути инкапсуляция «IP внутрь IP», а тут IP нет. Зато нет лишних заголовков.

Скорость сходимости у IS-IS выше, поэтому в больших сетях может быть лучше.
Из коробки **может работать с 200-250 устройствами**, в то время когда OSPF для 50-70 устройств.
IS-IS более масштабируем - в него проще добавить например метку для MPLS.

IS-IS часто работает с BGP и можно дожидаться поднятия BGP.

**Уровни IS-IS**

- L1 - внутри одной зоны
- L2 - между несколькими зонами. L2 можно сравнить с магистралью.
Можно сделать только L2 взаимодействие, но это неоптимально.

### Адресное пространство: 

Таблицы IP адресов

|Device|Port|IPv4      |   ||  Link           |Comment           |
|------|----|-----------|----------------------------------------|----|------------------|------------------|
|BBR1    |    |10.192.2.1  |                   | | |Management (lo0) |
|      |ge-0.0.0|192.168.100.17 |  ||BBR1 ge-0.0.0 - ge-0.0.0 BBR2 |Connectivity      |
|      |ge-0.0.1|192.168.100.21 |  ||BBR1 ge-0.0.1 - ge-0.0.1 BBR3 |Connectivity      |
|BBR2    |    |10.192.2.2  |       ||            |Management (lo0) |
|      |ge-0.0.0|192.168.100.18 |  ||BBR2 ge-0.0.0 - ge-0.0.0 BBR1 |Connectivity      |
|      |ge-0.0.2|192.168.100.29 |  ||BBR2 ge-0.0.2 - ge-0.0.2 BBR4 |Connectivity      |
|BBR3    |     |10.192.2.3 |       ||            |Management (lo0) |
|      |ge-0.0.1|192.168.100.22 |  ||BBR3 ge-0.0.1 - ge-0.0.1 BBR1 |Connectivity      |
|      |ge-0.0.0|192.168.100.25 |  ||BBR3 ge-0.0.0 - ge-0.0.3 BBR4 |Connectivity      |
|      |ge-0.0.2|192.168.100.33 |  ||BBR3 ge-0.0.2 - ge-0.0.0 SW1  |Connectivity      |
|BBR4    |    |10.192.2.4 |        ||             |Management (lo0)|
|      |ge-0.0.2|192.168.100.50 |  ||BBR4 ge-0.0.2 - ge-0.0.50 SW2 |Connectivity      |
|      |ge-0.0.0|192.168.100.26 |  ||BBR4 ge-0.0.0 - ge-0.0.0 BBR3 |Connectivity      |
|      |ge-0.0.1|192.168.100.50 |  ||BBR4 ge-0.0.1 - ge-0.0.1 BBR2 |Connectivity      |
|SW1    |    |10.192.2.5 |         ||            |Management (lo0) |
|SW2    |    |10.192.2.6 |         ||            |Management (lo0) |
|VPC1   |    |10.192.2.7 |         ||            |Management (lo0) |
|      |eth0|192.168.100.42 |  ||ACX7 ge-0.0.1 - ge-0.0.0 ACX6 |Connectivity      |
|VPC2   |    |10.192.2.7 |         ||            |Management (lo0) |
|      |eth0|192.168.100.42 |  ||ACX7 ge-0.0.1 - ge-0.0.0 ACX6 |Connectivity      |

### Конфигурация на оборудовании следующая: 

```
BBR1

deactivate system syslog user *
set system root-authentication plain-text-password
set system host-name BBR1

set interfaces lo0 unit 0 family inet address 10.192.2.1/32
set interfaces lo0 unit 0 family inet address 169.254.18.203/32

set interfaces ge-0/0/0 description TO_BBR2_ge-0/0/0
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


set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.17/30
set interfaces ge-0/0/0 unit 0 family iso
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.21/30
set interfaces ge-0/0/1 unit 0 family iso
set protocols isis interface ge-0/0/1.0 level 2 enable
set protocols isis interface ge-0/0/0.0 level 2 enable
set protocols isis interface lo0.0 level 2 enable
set protocols isis interface lo0.0
set interfaces lo0 unit 0 family inet address 10.0.0.1/32
set interfaces lo0 unit 0 family iso address 49.0001.1000.0000.0001.00


BBR2

deactivate system syslog user *
set system root-authentication plain-text-password
set system host-name BBR2

set interfaces lo0 unit 0 family inet address 10.192.2.2/32
set interfaces lo0 unit 0 family inet address 169.254.18.204/32

set interfaces ge-0/0/0 description TO_BBR1_ge-0/0/0
set interfaces ge-0/0/0 enable
set interfaces ge-0/0/0 vlan-tagging
set interfaces ge-0/0/0 mtu 2000
set interfaces ge-0/0/0 link-mode full-duplex
set interfaces ge-0/0/0 encapsulation vlan-ccc
set interfaces ge-0/0/0 unit 0 description TO_BBR1
set interfaces ge-0/0/0 unit 0 vlan-id 0
set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.18/30

set interfaces ge-0/0/1 description TO_BBR4_ge-0/0/2
set interfaces ge-0/0/1 enable
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 mtu 2000
set interfaces ge-0/0/1 link-mode full-duplex
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 0 description TO_BBR2
set interfaces ge-0/0/1 unit 0 vlan-id 0
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.29/30

set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.18/30
set interfaces ge-0/0/0 unit 0 family iso
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.29/30
set interfaces ge-0/0/1 unit 0 family iso
set protocols isis interface ge-0/0/0.0 level 2 enable
set protocols isis interface ge-0/0/1.0 level 2 enable
set protocols isis interface lo0.0 level 2 enable
set protocols isis interface lo0.0
set interfaces lo0 unit 0 family inet address 10.0.0.2/32
set interfaces lo0 unit 0 family iso address 49.0001.1000.0000.0002.00

BBR3

deactivate system syslog user *
set system root-authentication plain-text-password
set system host-name BBR3

set interfaces lo0 unit 0 family inet address 10.192.2.3/32
set interfaces lo0 unit 0 family inet address 169.254.18.205/32

set interfaces ge-0/0/1 description TO_BBR1_ge-0/0/1
set interfaces ge-0/0/1 enable
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 mtu 2000
set interfaces ge-0/0/1 link-mode full-duplex
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 0 description TO_BBR1
set interfaces ge-0/0/1 unit 0 vlan-id 0
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.22/30

set interfaces ge-0/0/0 description TO_BBR4_ge-0/0/3
set interfaces ge-0/0/0 enable
set interfaces ge-0/0/0 vlan-tagging
set interfaces ge-0/0/0 mtu 2000
set interfaces ge-0/0/0 link-mode full-duplex
set interfaces ge-0/0/0 encapsulation vlan-ccc
set interfaces ge-0/0/0 unit 0 description TO_BBR4
set interfaces ge-0/0/0 unit 0 vlan-id 0
set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.25/30

set interfaces ge-0/0/2 description TO_SW1_ge-0/0/0
set interfaces ge-0/0/2 enable
set interfaces ge-0/0/2 vlan-tagging
set interfaces ge-0/0/2 mtu 2000
set interfaces ge-0/0/2 link-mode full-duplex
set interfaces ge-0/0/2 encapsulation vlan-ccc
set interfaces ge-0/0/2 unit 0 description TO_SW1
set interfaces ge-0/0/2 unit 0 vlan-id 0
set interfaces ge-0/0/2 unit 0 family inet address 192.168.100.33/30

set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.25/30
set interfaces ge-0/0/0 unit 0 family iso
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.22/30
set interfaces ge-0/0/1 unit 0 family iso
set interfaces ge-0/0/2 unit 0 family inet address 192.168.100.33/30
set interfaces ge-0/0/2 unit 0 family iso
set protocols isis interface ge-0/0/0.0 level 2 enable
set protocols isis interface ge-0/0/1.0 level 2 enable
set protocols isis interface ge-0/0/2.0 level 2 enable
set protocols isis interface lo0.0 level 2 enable
set protocols isis interface lo0.0
set interfaces lo0 unit 0 family inet address 10.0.0.3/32
set interfaces lo0 unit 0 family iso address 49.0001.1000.0000.0003.00

BBR4

deactivate system syslog user *
set system root-authentication plain-text-password
set system host-name BBR4

set interfaces lo0 unit 0 family inet address 10.192.2.4/32
set interfaces lo0 unit 0 family inet address 169.254.18.206/32

set interfaces ge-0/0/0 description TO_BBR3_ge-0/0/0
set interfaces ge-0/0/0 enable
set interfaces ge-0/0/0 vlan-tagging
set interfaces ge-0/0/0 mtu 2000
set interfaces ge-0/0/0 link-mode full-duplex
set interfaces ge-0/0/0 encapsulation vlan-ccc
set interfaces ge-0/0/0 unit 0 description TO_BBR3
set interfaces ge-0/0/0 unit 0 vlan-id 0
set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.26/30

set interfaces ge-0/0/1 description TO_BBR2_ge-0/0/1
set interfaces ge-0/0/1 enable
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 mtu 2000
set interfaces ge-0/0/1 link-mode full-duplex
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 0 description TO_BBR2
set interfaces ge-0/0/1 unit 0 vlan-id 0
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.30/30

set interfaces ge-0/0/2 description TO_SW2_ge-0/0/2
set interfaces ge-0/0/2 enable
set interfaces ge-0/0/2 vlan-tagging
set interfaces ge-0/0/2 mtu 2000
set interfaces ge-0/0/2 link-mode full-duplex
set interfaces ge-0/0/2 encapsulation vlan-ccc
set interfaces ge-0/0/2 unit 0 description TO_SW2
set interfaces ge-0/0/2 unit 0 vlan-id 0
set interfaces ge-0/0/2 unit 0 family inet address 192.168.100.50/30

set interfaces ge-0/0/0 unit 0 family inet address 192.168.100.26/30
set interfaces ge-0/0/0 unit 0 family iso
set interfaces ge-0/0/1 unit 0 family inet address 192.168.100.30/30
set interfaces ge-0/0/1 unit 0 family iso
set interfaces ge-0/0/2 unit 0 family inet address 192.168.100.50/30
set interfaces ge-0/0/2 unit 0 family iso
set protocols isis interface ge-0/0/0.0 level 2 enable
set protocols isis interface ge-0/0/1.0 level 2 enable
set protocols isis interface ge-0/0/2.0 level 2 enable
set protocols isis interface lo0.0 level 2 enable
set protocols isis interface lo0.0
set interfaces lo0 unit 0 family inet address 10.0.0.4/32
set interfaces lo0 unit 0 family iso address 49.0001.1000.0000.0004.00

```





|      |ge-0.0.0|192.168.100.45|  ||ACX7 ge-0.0.0 - ge-0.0.1 ACX8 |Connectivity      |
|VPC    |    |10.192.2.8 |               ||      |Management (lo0) |
|      |ge-0.0.1|192.168.100.46 |  ||ACX6 ge-0.0.1 - ge-0.0.0 ACX6 |Connectivity      |
|      |ge-0.0.0|192.168.100.49 |  ||ACX6 ge-0.0.0 - ge-0.0.1 ACX8 |Connectivity      |
