## Проектирование ЦОД с использованием EVPN-VXLAN
### Цель
Создание масштабируемой, отказоустойчивой и гибкой сети в раммках ЦОД, с поддержкой единых L2/L3 доменов.

### Основные компоненты

EVPN (Ethernet VPN)

Control Plane , использующая BGP для обмена маршрутами между коммутаторами (VTEP).
Поддерживает многопоточность , маршрутизацию L2/L3 , интеграцию с VRF и оптимизацию BUM-трафика (Broadcast, Unknown Unicast, Multicast) .

VXLAN (Virtual Extensible LAN)

Туннелирование , позволяющее расширять L2-сети между ЦОДами через IP-инфраструктуру.
Использует VNI (VXLAN Network Identifier) для изоляции трафика между виртуальными сетями.

Spine-Leaf архитектура

Spine-Leaf архитектура — это тип сетевой топологии, используемой в центрах обработки данных (Data Center), которая обеспечивает высокую производительность, отказоустойчивость и масштабируемость. Она была разработана как улучшение по сравнению с традиционной трёхуровневой архитектурой (Core – Distribution – Access), чтобы избежать узких мест при передаче данных между серверами.

ЦОД строится по схеме spine-leaf , где:

Spine — Core коммутаторы, отвечающие за маршрутизацию между leaf-узлами.
Leaf — Border коммутаторы, подключающие серверы и обеспечивающие VXLAN-туннели.

## Топология и логическая схема

![](https://github.com/Hardliner991/Otus-network-engineer/blob/main/Project/Topology.png)

## Ключевые аспекты проектирования

Мультисайтовое соединение через EVPN Type-5
Type-5 маршруты (IP Prefix routes) используются для маршрутизации между ЦОДами.

Настройка Route Reflectors (RR)
В качестве RR используются Spine коммутаторы.

Route Targets (RT) для изоляции трафика
RT определяют, какие маршруты должны быть импортированы/экспортированы между VRF.

Обработка BUM-трафика
Ingress Replication : Каждый Leaf отправляет копии пакетов всем удаленным VTEP.
PIM (Protocol Independent Multicast) : Используется для оптимизации multicast-трафика.
Optimized BUM (OB) : Современный подход, сочетающий ingress replication и PIM.

Отказоустойчивость
Redundant VTEP : Сервер подключается к двум Leaf-коммутаторам (vPC или MLAG).

## Адресное пространство

| hostname | interface |   IP/MASK   | Description |
| :------: | :-------: | :----------: | :---------: |
|  leaf-1  | Loopback1 | 192.2.0.1 /32 |            |
|  leaf-1  |  eth 1/1  | 192.4.1.7 /31 | to-spine-1 |
|  leaf-1  |  eth 1/2  | 192.4.2.2 /31 | to-spine-2 |
|          |          |              |            |
|  leaf-2  | Loopback1 | 192.2.0.2 /32 |            |
|  leaf-2  |  eth 1/1  | 192.4.1.3 /31 | to-spine-1 |
|  leaf-2  |  eth 1/2  | 192.4.1.4 /31 | to-spine-2 |
|          |          |              |            |
|  leaf-3  | Loopback1 | 192.2.0.3 /32 |            |
|  leaf-3  |  eth 1/1  | 192.4.1.5 /31 | to-spine-1 |
|  leaf-3  |  eth 1/2  | 192.4.1.6 /31 | to-spine-2 |
|          |          |              |            |
| spine-1 | Loopback1 | 192.1.1.0/32 |            |
| spine-1 |  eth 1/1  | 192.4.1.6/31 |  to-leaf-1  |
| spine-1 |  eth 1/2  | 192.4.1.2/31 |  to-leaf-2  |
| spine-1 |  eth 1/3  | 192.4.1.4/31 |  to-leaf-3  |
|          |          |              |            |
| spine-2 | Loopback1 | 192.1.2.0/32 |            |
| spine-2 |  eth 1/1  | 192.4.2.3/31 |  to-leaf-1  |
| spine-2 |  eth 1/2  | 192.4.1.5/31 |  to-leaf-2  |
| spine-2 |  eth 1/3  | 192.4.1.7/31 |  to-leaf-3  |

### Сеть Underlay

Использование протокола Border Gateway Protocol (BGP) для маршрутизации на физическом уровне сети, т.е. на уровне underlay-сети , которая обеспечивает подключение всех коммутаторов и маршрутизаторов между собой.
Это особенно актуально в современных архитектурах центров обработки данных (Data Centers), таких как Spine-Leaf , где важно организовать эффективную, масштабируемую и отказоустойчивую инфраструктуру передачи данных.

### Сеть Overlay 

Overlay сеть — это логическая сеть , построенная на уровне программного обеспечения, поверх существующей физической (или underlay) сети.

Обеспечивает абстракцию от физической топологии. В данной лаборатоной работе используется EVPN/VXLAN.
Позволяет создавать виртуальные L2/L3 сети между хостами, даже если они находятся в разных частях дата-центра.
Использует инкапсуляцию пакетов для передачи данных через underlay.

## Проектирование клиентских подключений

На проектируемой сети будут предоставлены услуги L2VPN, L3VPN
Резервирование обеспечивается на основе ESI LAG т.е. когда клиент одновременно включается в два и более leaf свитчей и 
организует LAG со свой стороны.


## Ход работ

### Вывод диагностической информации

Leaf1

```
leaf1(config)#sh ip bgp summary
BGP summary information for VRF default
Router identifier 192.1.0.1, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  192.1.1.0        4  65000            778       781    0    0 00:38:32 Estab   5      4
  192.1.2.0        4  65000            780       781    0    0 00:38:38 Estab   5      4
  192.4.1.6        4  65000            779       780    0    0 00:38:36 Estab   5      5
  192.4.2.3        4  65000            780       783    0    0 00:38:40 Estab   5      5
```
```
leaf1(config-if-Et2)#sh bgp evpn
BGP routing table information for VRF default
Router identifier 192.1.0.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65003:1003333 mac-ip 5000.008c.5cd7
                                 192.100.0.3           -       100     0       65000 65003 i
 *  ec    RD: 65003:1003333 mac-ip 5000.008c.5cd7
                                 192.100.0.3           -       100     0       65000 65003 i
 * >Ec    RD: 65003:1003333 mac-ip 5000.008c.5cd7 33.33.33.50
                                 192.100.0.3           -       100     0       65000 65003 i
 *  ec    RD: 65003:1003333 mac-ip 5000.008c.5cd7 33.33.33.50
                                 192.100.0.3           -       100     0       65000 65003 i
 * >Ec    RD: 65001:1003333 imet 192.100.0.2
                                 192.100.0.2           -       100     0       65000 65002 i
 *  ec    RD: 65001:1003333 imet 192.100.0.2
                                 192.100.0.2           -       100     0       65000 65002 i
 * >Ec    RD: 65002:100100 imet 192.100.0.2
                                 192.100.0.2           -       100     0       65000 65002 i
 *  ec    RD: 65002:100100 imet 192.100.0.2
                                 192.100.0.2           -       100     0       65000 65002 i
 * >Ec    RD: 65002:100200 imet 192.100.0.2
                                 192.100.0.2           -       100     0       65000 65002 i
 *  ec    RD: 65002:100200 imet 192.100.0.2
                                 192.100.0.2           -       100     0       65000 65002 i
 * >Ec    RD: 65003:100100 imet 192.100.0.3
                                 192.100.0.3           -       100     0       65000 65003 i
 *  ec    RD: 65003:100100 imet 192.100.0.3
                                 192.100.0.3           -       100     0       65000 65003 i
 * >Ec    RD: 65003:100200 imet 192.100.0.3
                                 192.100.0.3           -       100     0       65000 65003 i
 *  ec    RD: 65003:100200 imet 192.100.0.3
                                 192.100.0.3           -       100     0       65000 65003 i
 * >Ec    RD: 65003:1003333 imet 192.100.0.3
                                 192.100.0.3           -       100     0       65000 65003 i
 *  ec    RD: 65003:1003333 imet 192.100.0.3
                                 192.100.0.3           -       100     0       65000 65003 i
 * >      RD: 65001:100100 imet 192.100.0.1
                                 -                     -       -       0       i
 * >      RD: 65001:100200 imet 192.100.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 65003:333 ip-prefix 0.0.0.0/0
                                 192.100.0.3           -       100     0       65000 65003 64009 ?
 *  ec    RD: 65003:333 ip-prefix 0.0.0.0/0
                                 192.100.0.3           -       100     0       65000 65003 64009 ?
 * >Ec    RD: 65003:999 ip-prefix 0.0.0.0/0
                                 192.100.0.3           -       100     0       65000 65003 64009 ?
 *  ec    RD: 65003:999 ip-prefix 0.0.0.0/0
                                 192.100.0.3           -       100     0       65000 65003 64009 ?
 * >Ec    RD: 65003:333 ip-prefix 8.8.4.4/32
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 *  ec    RD: 65003:333 ip-prefix 8.8.4.4/32
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 * >Ec    RD: 65003:999 ip-prefix 8.8.4.4/32
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 *  ec    RD: 65003:999 ip-prefix 8.8.4.4/32
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 * >Ec    RD: 65003:333 ip-prefix 8.8.8.8/32
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 *  ec    RD: 65003:333 ip-prefix 8.8.8.8/32
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 * >Ec    RD: 65003:999 ip-prefix 8.8.8.8/32
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 *  ec    RD: 65003:999 ip-prefix 8.8.8.8/32
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 * >      RD: 65001:999 ip-prefix 10.10.2.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 65002:999 ip-prefix 10.10.2.0/24
                                 192.100.0.2           -       100     0       65000 65002 i
 *  ec    RD: 65002:999 ip-prefix 10.10.2.0/24
                                 192.100.0.2           -       100     0       65000 65002 i
 * >Ec    RD: 65003:999 ip-prefix 10.10.2.0/24
                                 192.100.0.3           -       100     0       65000 65003 i
 *  ec    RD: 65003:999 ip-prefix 10.10.2.0/24
                                 192.100.0.3           -       100     0       65000 65003 i
 * >      RD: 65001:999 ip-prefix 10.10.3.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 65002:999 ip-prefix 10.10.3.0/24
                                 192.100.0.2           -       100     0       65000 65002 i
 *  ec    RD: 65002:999 ip-prefix 10.10.3.0/24
                                 192.100.0.2           -       100     0       65000 65002 i
 * >Ec    RD: 65003:999 ip-prefix 10.10.3.0/24
                                 192.100.0.3           -       100     0       65000 65003 i
 *  ec    RD: 65003:999 ip-prefix 10.10.3.0/24
                                 192.100.0.3           -       100     0       65000 65003 i
 * >Ec    RD: 65003:333 ip-prefix 10.200.0.1/32
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 *  ec    RD: 65003:333 ip-prefix 10.200.0.1/32
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 * >Ec    RD: 65003:999 ip-prefix 10.200.0.1/32
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 *  ec    RD: 65003:999 ip-prefix 10.200.0.1/32
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 * >Ec    RD: 65002:333 ip-prefix 33.33.33.0/24
                                 192.100.0.2           -       100     0       65000 65002 i
 *  ec    RD: 65002:333 ip-prefix 33.33.33.0/24
                                 192.100.0.2           -       100     0       65000 65002 i
 * >Ec    RD: 65003:333 ip-prefix 33.33.33.0/24
                                 192.100.0.3           -       100     0       65000 65003 i
 *  ec    RD: 65003:333 ip-prefix 33.33.33.0/24
                                 192.100.0.3           -       100     0       65000 65003 i
 * >Ec    RD: 65003:999 ip-prefix 33.33.33.0/24
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 *  ec    RD: 65003:999 ip-prefix 33.33.33.0/24
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 * >Ec    RD: 65003:333 ip-prefix 66.66.66.0/24
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 *  ec    RD: 65003:333 ip-prefix 66.66.66.0/24
                                 192.100.0.3           -       100     0       65000 65003 64009 i
 * >Ec    RD: 65003:999 ip-prefix 66.66.66.0/24
                                 192.100.0.3           -       100     0       65000 65003 i
 *  ec    RD: 65003:999 ip-prefix 66.66.66.0/24
                                 192.100.0.3           -       100     0       65000 65003 i
```
Состояние MLAG между Leaf1 и Leaf2
```
leaf1(config)#sh mlag detail
MLAG Configuration:
domain-id                          :               mlag1
local-interface                    :            Vlan4094
peer-address                       :         10.10.100.1
peer-link                          :       Port-Channel1
peer-config                        :        inconsistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:00:00:5b:6f:f5
dual-primary detection             :            Disabled
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   0
Active-full                        :                   1

MLAG Detailed Status:
State                           :            secondary
Peer State                      :              primary
State changes                   :                    2
Last state change time          :          0:06:10 ago
Hardware ready                  :                 True
Failover                        :                False
Failover Cause(s)               :              Unknown
Last failover change time       :                never
Secondary from failover         :                 True
Peer MAC address                :    50:00:00:5b:6f:f5
Peer MAC routing supported      :                False
Reload delay                    :          300 seconds
Non-MLAG reload delay           :          300 seconds
Ports errdisabled               :                False
Lacp standby                    :                False
Configured heartbeat interval   :              4000 ms
Effective heartbeat interval    :              4000 ms
Heartbeat timeout               :             60000 ms
Last heartbeat timeout          :                never
Heartbeat timeouts since reboot :                    0
UDP heartbeat alive             :                 True
Heartbeats sent/received        :                95/94
Peer monotonic clock offset     :   513.530470 seconds
Agent should be running         :                 True
P2p mount state changes         :                    1
Fast MAC redirection enabled    :                False

```
VPC 10.10.2.13 ping VPC 10.10.2.11
```
VPCS> ping 10.10.2.11

84 bytes from 10.10.2.11 icmp_seq=1 ttl=64 time=8.988 ms
84 bytes from 10.10.2.11 icmp_seq=2 ttl=64 time=8.087 ms
84 bytes from 10.10.2.11 icmp_seq=3 ttl=64 time=8.520 ms
84 bytes from 10.10.2.11 icmp_seq=4 ttl=64 time=6.713 ms
84 bytes from 10.10.2.11 icmp_seq=5 ttl=64 time=8.846 ms
```
VPC 10.10.2.11 ping VPC 10.10.2.13
```
VPCS> ping 10.10.2.13

84 bytes from 10.10.2.13 icmp_seq=1 ttl=64 time=7.775 ms
84 bytes from 10.10.2.13 icmp_seq=2 ttl=64 time=7.767 ms
84 bytes from 10.10.2.13 icmp_seq=3 ttl=64 time=8.140 ms
84 bytes from 10.10.2.13 icmp_seq=4 ttl=64 time=8.658 ms
84 bytes from 10.10.2.13 icmp_seq=5 ttl=64 time=7.801 ms
```
VPC 10.10.3.13 ping VPC 10.10.3.11
```
VPCS> ping 10.10.3.11

84 bytes from 10.10.3.11 icmp_seq=1 ttl=64 time=10.237 ms
84 bytes from 10.10.3.11 icmp_seq=2 ttl=64 time=10.215 ms
84 bytes from 10.10.3.11 icmp_seq=3 ttl=64 time=9.501 ms
84 bytes from 10.10.3.11 icmp_seq=4 ttl=64 time=9.003 ms
84 bytes from 10.10.3.11 icmp_seq=5 ttl=64 time=9.568 ms
```
VPC 10.10.3.11 ping VPC 10.10.3.13
```
VPCS> ping 10.10.3.13

84 bytes from 10.10.3.13 icmp_seq=1 ttl=64 time=8.853 ms
84 bytes from 10.10.3.13 icmp_seq=2 ttl=64 time=8.311 ms
84 bytes from 10.10.3.13 icmp_seq=3 ttl=64 time=8.894 ms
84 bytes from 10.10.3.13 icmp_seq=4 ttl=64 time=8.480 ms
84 bytes from 10.10.3.13 icmp_seq=5 ttl=64 time=10.564 ms
```

Проверка связности через Lo1 и Lo100

```
spine-1(config)#ping 192.1.0.1
PING 192.1.0.1 (192.1.0.1) 72(100) bytes of data.
80 bytes from 192.1.0.1: icmp_seq=1 ttl=64 time=3.26 ms
80 bytes from 192.1.0.1: icmp_seq=2 ttl=64 time=1.43 ms
80 bytes from 192.1.0.1: icmp_seq=3 ttl=64 time=1.53 ms
80 bytes from 192.1.0.1: icmp_seq=4 ttl=64 time=2.40 ms
80 bytes from 192.1.0.1: icmp_seq=5 ttl=64 time=1.83 ms

--- 192.1.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 13ms
rtt min/avg/max/mdev = 1.435/2.093/3.261/0.675 ms, ipg/ewma 3.366/2.670 ms
spine-1(config)#ping 192.1.0.2
PING 192.1.0.2 (192.1.0.2) 72(100) bytes of data.
80 bytes from 192.1.0.2: icmp_seq=1 ttl=64 time=2.08 ms
80 bytes from 192.1.0.2: icmp_seq=2 ttl=64 time=1.90 ms
80 bytes from 192.1.0.2: icmp_seq=3 ttl=64 time=1.81 ms
80 bytes from 192.1.0.2: icmp_seq=4 ttl=64 time=1.55 ms
80 bytes from 192.1.0.2: icmp_seq=5 ttl=64 time=1.74 ms

--- 192.1.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 11ms
rtt min/avg/max/mdev = 1.555/1.820/2.083/0.180 ms, ipg/ewma 2.910/1.942 ms
spine-1(config)#ping 192.1.0.3
PING 192.1.0.3 (192.1.0.3) 72(100) bytes of data.
80 bytes from 192.1.0.3: icmp_seq=1 ttl=64 time=1.85 ms
80 bytes from 192.1.0.3: icmp_seq=2 ttl=64 time=1.84 ms
80 bytes from 192.1.0.3: icmp_seq=3 ttl=64 time=1.54 ms
80 bytes from 192.1.0.3: icmp_seq=4 ttl=64 time=1.58 ms
80 bytes from 192.1.0.3: icmp_seq=5 ttl=64 time=1.81 ms

--- 192.1.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 12ms
rtt min/avg/max/mdev = 1.544/1.727/1.854/0.138 ms, ipg/ewma 3.161/1.788 ms

```
```
spine-1(config)#ping 192.100.0.1
PING 192.100.0.1 (192.100.0.1) 72(100) bytes of data.
80 bytes from 192.100.0.1: icmp_seq=1 ttl=64 time=1.90 ms
80 bytes from 192.100.0.1: icmp_seq=2 ttl=64 time=1.44 ms
80 bytes from 192.100.0.1: icmp_seq=3 ttl=64 time=1.66 ms
80 bytes from 192.100.0.1: icmp_seq=4 ttl=64 time=2.02 ms
80 bytes from 192.100.0.1: icmp_seq=5 ttl=64 time=2.41 ms

--- 192.100.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 11ms
rtt min/avg/max/mdev = 1.445/1.890/2.410/0.329 ms, ipg/ewma 2.777/1.921 ms
spine-1(config)#ping 192.100.0.2
PING 192.100.0.2 (192.100.0.2) 72(100) bytes of data.
80 bytes from 192.100.0.2: icmp_seq=1 ttl=64 time=2.40 ms
80 bytes from 192.100.0.2: icmp_seq=2 ttl=64 time=1.82 ms
80 bytes from 192.100.0.2: icmp_seq=3 ttl=64 time=1.85 ms
80 bytes from 192.100.0.2: icmp_seq=4 ttl=64 time=1.83 ms
80 bytes from 192.100.0.2: icmp_seq=5 ttl=64 time=1.92 ms

--- 192.100.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 12ms
rtt min/avg/max/mdev = 1.827/1.969/2.405/0.225 ms, ipg/ewma 3.017/2.181 ms
spine-1(config)#ping 192.100.0.3
PING 192.100.0.3 (192.100.0.3) 72(100) bytes of data.
80 bytes from 192.100.0.3: icmp_seq=1 ttl=64 time=2.12 ms
80 bytes from 192.100.0.3: icmp_seq=2 ttl=64 time=1.80 ms
80 bytes from 192.100.0.3: icmp_seq=3 ttl=64 time=1.71 ms
80 bytes from 192.100.0.3: icmp_seq=4 ttl=64 time=1.27 ms
80 bytes from 192.100.0.3: icmp_seq=5 ttl=64 time=1.81 ms

--- 192.100.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 11ms
rtt min/avg/max/mdev = 1.273/1.747/2.126/0.274 ms, ipg/ewma 2.841/1.927 ms
```

## Конфигурация устройств: 

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
   seq 30 permit 192.1.0.1/32
   seq 40 permit 192.1.0.2/32
   seq 50 permit 192.1.0.3/32
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
vlan 100
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
   seq 20 permit 192.100.0.0/24
   seq 30 permit 10.10.2.0/24
   seq 40 permit 10.10.3.0/24
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
      rd 65001:100100
      route-target both 100:100
      redistribute learned
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
system mac-address 00:00:00:00:00:06
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
   description to_vpc
   switchport access vlan 100
!
interface Ethernet7
   description to_vpc
   switchport access vlan 200
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
peer-filter LEAF
   10 match as-range 65000-65003 result accept
!
router bgp 65001
   router-id 192.1.0.1
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 192.1.0.0/16 peer-group EVPN peer-filter EVPN
   bgp listen range 192.4.0.0/22 peer-group LEAF peer-filter LEAF
   neighbor EVPN peer group
   neighbor EVPN remote-as 65000
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor LEAF peer group
   neighbor LEAF remote-as 65000
   neighbor LEAF bfd
   neighbor LEAF allowas-in 1
   neighbor LEAF rib-in pre-policy retain all
   neighbor LEAF send-community
   neighbor LEAF maximum-routes 100
   neighbor 192.1.1.0 peer group EVPN
   neighbor 192.1.1.0 remote-as 65000
   neighbor 192.1.2.0 peer group EVPN
   neighbor 192.1.2.0 remote-as 65000
   neighbor 192.4.1.6 peer group LEAF
   neighbor 192.4.2.3 peer group LEAF
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
system mac-address 00:00:00:00:00:07
!
vlan 100,200,333
!
vlan 4094
   trunk group mlag
!
vrf instance GAGA
!
vrf instance GAGA333
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
   description to_VPC
   switchport access vlan 100
!
interface Ethernet7
   description to_VPC
   switchport access vlan 333
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
   vrf GAGA
   ip address virtual 10.10.3.254/24
!
interface Vlan333
   vrf GAGA333
   ip address 33.33.33.34/24
!
interface Vlan4094
   no autostate
   ip address 10.10.100.1/31
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 100 vni 100100
   vxlan vlan 200 vni 100200
   vxlan vlan 333 vni 3333
   vxlan vrf GAGA vni 999
   vxlan vrf GAGA333 vni 333
   vxlan virtual-vtep local-interface Loopback100
!
ip virtual-router mac-address 00:00:00:00:00:02
!
ip routing
ip routing vrf GAGA
ip routing vrf GAGA333
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
peer-filter LEAF
   10 match as-range 65000-65003 result accept
!
router bgp 65002
   router-id 192.1.0.2
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 192.1.0.0/16 peer-group EVPN peer-filter EVPN
   bgp listen range 192.4.0.0/22 peer-group LEAF peer-filter LEAF
   neighbor EVPN peer group
   neighbor EVPN remote-as 65000
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback2
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor LEAF peer group
   neighbor LEAF remote-as 65000
   neighbor LEAF bfd
   neighbor LEAF allowas-in 1
   neighbor LEAF rib-in pre-policy retain all
   neighbor LEAF send-community
   neighbor LEAF maximum-routes 1000
   neighbor 192.1.1.0 peer group EVPN
   neighbor 192.1.1.0 remote-as 65000
   neighbor 192.1.2.0 peer group EVPN
   neighbor 192.1.2.0 remote-as 65000
   neighbor 192.4.1.2 peer group LEAF
   neighbor 192.4.1.5 peer group LEAF
   neighbor 192.100.0.1 remote-as 65001
   neighbor 192.100.0.1 update-source Loopback100
   neighbor 192.100.0.3 remote-as 65003
   neighbor 192.100.0.3 update-source Loopback100
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
   vlan 333
      rd 65001:1003333
      route-target both 3333:3333
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
   vrf GAGA333
      rd 65002:333
      route-target import evpn 333:333
      route-target export evpn 333:333
      redistribute connected
!
end
```
```
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
vlan 25,100,200,333,1000
!
vrf instance GAGA
!
vrf instance GAGA333
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
   description to_BorderRouter
   switchport trunk allowed vlan 333,1000
   switchport mode trunk
!
interface Loopback1
   ip address 192.1.0.3/32
!
interface Loopback100
   description Lo100
   ip address 192.100.0.3/32
!
interface Management1
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
interface Vlan333
   description to_BorderRouter
   vrf GAGA333
   ip address 33.33.33.33/24
!
interface Vlan1000
   description to_BorderRouter
   vrf GAGA
   ip address 66.66.66.66/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 100 vni 100100
   vxlan vlan 200 vni 100200
   vxlan vlan 333 vni 3333
   vxlan vrf GAGA vni 999
   vxlan vrf GAGA333 vni 333
   vxlan virtual-vtep local-interface Loopback100
!
ip virtual-router mac-address 00:00:00:00:00:03
!
ip routing
ip routing vrf GAGA
ip routing vrf GAGA333
!
ip prefix-list PL_LOOP
   seq 10 permit 192.1.0.3/32
   seq 20 permit 192.100.0.3/32
!
route-map RM_CONN permit 10
   match ip address prefix-list PL_LOOP
!
peer-filter EVPN
   10 match as-range 65000-65003 result accept
!
peer-filter LEAF
   10 match as-range 65000-65003 result accept
!
router bgp 65003
   router-id 192.1.0.3
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 192.1.0.0/16 peer-group EVPN peer-filter EVPN
   bgp listen range 192.4.0.0/22 peer-group LEAF peer-filter LEAF
   neighbor EVPN peer group
   neighbor EVPN remote-as 65000
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor LEAF peer group
   neighbor LEAF remote-as 65000
   neighbor LEAF bfd
   neighbor LEAF allowas-in 1
   neighbor LEAF rib-in pre-policy retain all
   neighbor LEAF send-community
   neighbor LEAF maximum-routes 1000
   neighbor 192.1.1.0 peer group EVPN
   neighbor 192.1.1.0 remote-as 65000
   neighbor 192.1.2.0 peer group EVPN
   neighbor 192.1.2.0 remote-as 65000
   neighbor 192.4.1.4 peer group LEAF
   neighbor 192.4.1.7 peer group LEAF
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
   vlan 333
      rd 65003:1003333
      route-target both 3333:3333
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   vrf GAGA
      rd 65003:999
      route-target import evpn 999:999
      route-target export evpn 999:999
      neighbor 66.66.66.80 remote-as 64009
      redistribute connected
      !
      address-family ipv4
         neighbor 66.66.66.80 activate
   !
   vrf GAGA333
      rd 65003:333
      route-target import evpn 333:333
      route-target export evpn 333:333
      neighbor 33.33.33.50 remote-as 64009
      redistribute connected
      !
      address-family ipv4
         neighbor 33.33.33.50 activate
!
end
```
## Вывод

В ходе выполнения лабораторной работы была спроектирована масштабируемая и отказоустойчивая сеть центра обработки данных (ЦОД) с использованием технологии EVPN-VXLAN . Реализация данной архитектуры позволила создать гибкую сетевую инфраструктуру, поддерживающую как L2 , так и L3 домены в едином логическом пространстве.
Сеть была построена на основе Spine-Leaf топологии , что обеспечило высокую пропускную способность, отсутствие узких мест и равномерное распределение нагрузки. Использование протокола BGP для маршрутизации в underlay-сети обеспечило стабильность, предсказуемое поведение сети и возможность её дальнейшего масштабирования без значительных изменений существующей инфраструктуры.

Таким образом, применение технологии EVPN-VXLAN в проектировании ЦОД позволяет создавать современные, высокопроизводительные, отказоустойчивые и легко управляемые сетевые среды, подходящие как для частных, так и для публичных облаков.



