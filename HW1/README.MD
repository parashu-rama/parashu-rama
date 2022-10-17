# Домашнее задание 1
## Проектирование адресного пространства
![Схема сети](hw1.drawio.png "Схема сети")
### IP plan
p2p - /31 
loopback - /32

IP = 10.Dn.Sn.X/31, где:
Dn – номер ЦОДа,
Sn – номер spine,
X – значение по порядку,
Dn для DC1 = 0 – 7, где 0 – loopback1
1 – loopback2
2 – p2p links
3 – reserved 4-7 – services
