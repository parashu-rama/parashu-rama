# Лабораторная работа 3
## eBGP underlay
![Схема сети](net.png "Схема сети")
### Схема eBGP
AS 65000 - Spines<br>
AS 65001 - Leaf1<br>
AS 65002 - Leaf2<br>
AS 65003 - Leaf3
### Конфигурация
#### Spine 1
    feature bgp
    router bgp 65000
      router-id 10.0.1.1
      bestpath as-path multipath-relax
      address-family ipv4 unicast
        maximum-paths 64
      neighbor 10.1.1.1
        remote-as 65001
        timers 3 9
        address-family ipv4 unicast
      neighbor 10.1.1.3
        remote-as 65002
        timers 3 9
        address-family ipv4 unicast
      neighbor 10.1.1.5
        remote-as 65003
        timers 3 9
        address-family ipv4 unicast
#### Spine 2
    feature bgp
    router bgp 65000
      router-id 10.0.2.1
      bestpath as-path multipath-relax
      address-family ipv4 unicast
        maximum-paths 64
      neighbor 10.1.2.1
        remote-as 65001
        timers 3 9
        address-family ipv4 unicast
      neighbor 10.1.2.3
        remote-as 65002
        timers 3 9
        address-family ipv4 unicast
      neighbor 10.1.2.5
        remote-as 65003
        timers 3 9
        address-family ipv4 unicast
#### Leaf 1
    feature isis
    interface Ethernet1/1
      isis circuit-type level-1
      ip router isis underlay
    interface Ethernet1/2
      isis circuit-type level-1
      ip router isis underlay
    router isis underlay
      net 49.0001.0001.0000.0001.00
      is-type level-1 
#### Leaf 2
    feature isis
    interface Ethernet1/1
      isis circuit-type level-1
      ip router isis underlay
    interface Ethernet1/2
      isis circuit-type level-1
      ip router isis underlay
    router isis underlay
      net 49.0001.0001.0000.0002.00
      is-type level-1 
#### Leaf 3
    feature isis
    interface Ethernet1/1
      isis circuit-type level-1
      ip router isis underlay
    interface Ethernet1/2
      isis circuit-type level-1
      ip router isis underlay
    router isis underlay
      net 49.0001.0001.0000.0003.00
      is-type level-1 
