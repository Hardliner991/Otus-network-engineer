OSPF
Цель:
Настроить OSPF.
Разделить сеть на зоны.
Настроить фильтрацию между зонами.

Описание/Пошаговая инструкция выполнения домашнего задания:
Маршрутизаторы BBR1-BBR4 находятся в зоне 0 - backbone.
Маршрутизаторы ACX6-ACX9 находятся в зоне 1.



Адресное пространство: 

|Device|Port|IPv4      |   ||  Link           |Comment           |
|------|----|-----------|----------------------------------------|----|------------------|------------------|
|BBR1    |    |10.192.2.1  |                   | | |Management (lo0) |
|      |ge-0.0.0|10.10.0.10 |  ||BBR1 ge-0.0.0 - ge-0.0.1 BBR2 |Connectivity      |
|      |ge-0.0.1|10.10.0.12 |  ||BBR1 e1/1 - e1/1 BBR3 |Connectivity      |
|BBR2    |    |10.192.2.2  |     ||          |Management (lo0) |
|      |ge-0.0.1|10.10.0.14 |  ||R2 e1/0 - e1/0 R1 |Connectivity      |
|      |ge-0.0.2|10.10.0.15 |  ||R2 e1/1 - e1/1 R4 |                  |
|BBR3    |     |10.192.2.3 |                ||     |Management (lo0) |
|      |ge-0.0.1|10.10.0.16 |  ||R3 e1/0 - e1/0 R4 |Connectivity      |
|      |ge-0.0.2|100.0.0.13 |  ||R3 e1/1 - e1/1 R1 |Connectivity      |
|      |ge-0.0.1|100.0.0.13 |  ||R3 e1/1 - e1/1 R1 |Connectivity      |
|BBR4    |    |10.192.2.4 |               ||      |Management (lo0) |
|      |ge-0.0.2|10.10.0.17 |  ||R4 e1/0 - e1/0 R3 |Connectivity      |
|      |ge-0.0.3|172.16.0.15|  ||R4 e1/1 - e1/1 R2 |Connectivity      |
|      |ge-0.0.1|100.0.0.13 |  ||R3 e1/1 - e1/1 R1 |Connectivity      |
|ACX9    |    |10.192.2.5 |               ||      |Management (lo0) |
|      |ge-0.0.2|10.10.0.17 |  ||R4 e1/0 - e1/0 R3 |Connectivity      |
|      |ge-0.0.3|172.16.0.15|  ||R4 e1/1 - e1/1 R2 |Connectivity      |
|ACX8    |    |10.192.2.6 |               ||      |Management (lo0) |
|      |ge-0.0.2|10.10.0.17 |  ||R4 e1/0 - e1/0 R3 |Connectivity      |
|      |ge-0.0.3|172.16.0.15|  ||R4 e1/1 - e1/1 R2 |Connectivity      |
|ACX7    |    |10.192.2.7 |               ||      |Management (lo0) |
|      |ge-0.0.2|10.10.0.17 |  ||R4 e1/0 - e1/0 R3 |Connectivity      |
|      |ge-0.0.3|172.16.0.15|  ||R4 e1/1 - e1/1 R2 |Connectivity      |
|ACX6    |    |10.192.2.8 |               ||      |Management (lo0) |
|      |ge-0.0.2|10.10.0.17 |  ||R4 e1/0 - e1/0 R3 |Connectivity      |
|      |ge-0.0.3|172.16.0.15|  ||R4 e1/1 - e1/1 R2 |Connectivity      |
|SW1   |    |10.10.0.26 |                  ||   |Management (VLAN) |
|SW2   |    |10.10.0.25 |                 ||    |Management (VLAN) |
|VPC1  |eth0|100.1.1.70 |   ||VPC1 eth0 - e1/1 SW1|Connectivity   |
|VPC2  |eth0|100.1.2.65 |   ||VPC2 eth0 - e1/1 SW2|Connectivity   |

