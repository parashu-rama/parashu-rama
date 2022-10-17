# Домашнее задание 1
## Проектирование адресного пространства
![Схема сети](hw1.drawio.png "Схема сети")
### IP plan
#### Маски:
* p2p - /31
* loopback - /32
#### IP = 10.Dn.Sn.X/31, где:
* Dn – номер ЦОДа,
* Sn – 1-100 spine, 101-255 leaf
* X – 1-2 для loopback, для p2p по порядку
#### Dn для DC1 = 0 – 7, где 
* 0 - loopback
* 1 – p2p links
* 2-3 - reserved
* 4-7 – services
### Адресация
#### Spine 1
* 10.0.1.1/32 loopback1
* 10.0.1.2/32 loopback2
* 10.1.1.0/31 p2p leaf1
* 10.1.1.2/31 p2p leaf2
* 10.1.1.4/31 p2p leaf3
#### Spine 2
* 10.0.2.1/32 loopback1
* 10.0.2.2/32 loopback2
* 10.1.2.0/31 p2p leaf1
* 10.1.2.2/31 p2p leaf2
* 10.1.2.4/31 p2p leaf3
#### Spine 3
* 10.0.3.1/32 loopback1
* 10.0.3.2/32 loopback2
* 10.1.3.0/31 p2p leaf4
#### Leaf 1
* 10.0.101.1/32 loopback1
* 10.0.101.1/32 loopback2
* 10.1.1.1/31 p2p spine1
* 10.1.2.1/31 p2p spine2
#### Leaf 2
* 10.0.102.1/32 loopback1
* 10.0.102.2/32 loopback2
* 10.1.1.3/31 p2p spine1
* 10.1.2.3/31 p2p spine2
#### Leaf 3
* 10.0.103.1/32 loopback1
* 10.0.103.2/32 loopback2
* 10.1.1.5/31 p2p spine1
* 10.1.2.5/31 p2p spine2
#### Leaf 4
* 10.0.104.1/32 loopback1
* 10.0.104.2/32 loopback2
* 10.1.3.1/31 p2p spine1

10.4.0.0/14 - services
