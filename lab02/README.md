# OSPF

#### Цель:
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
|      |ge-0.0.1|192.168.100.46 |  ||ACX6 ge-0.0.1 - ge-0.0.0 ACX6 |Connectivity      |
|      |ge-0.0.0|192.168.100.49 |  ||ACX6 ge-0.0.0 - ge-0.0.1 ACX8 |Connectivity      |


