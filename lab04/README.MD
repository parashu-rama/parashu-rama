# Лабораторная работа 4
## Overlay на основе VxLAN EVPN для L2 связанности между клиентами
![Схема сети](net.png "Схема сети")
### Схема eBGP для underlay и overlay
    AS 4200000000 - Spines
    AS 4200000001 - Leaf1
    AS 4200000002 - Leaf2
    AS 4200000003 - Leaf3
### Нумерация VNI
    1xxxx - где xxxx - номер Vlan
### Коментарии к настройке
    disable-peer-as-check - не проверяем peer-as т.к. используем одну AS на Spine и разные на Leaf
    allowas-in - разрешаем повторение AS, т.к. используем разные AS на Leaf и Spine
    next-hop unchanged - приоритет directly-connected маршрутов
### Конфигурация
#### Spine 1
    router bgp 4200000000
      router-id 10.0.1.1
      address-family ipv4 unicast
        redistribute direct route-map LOOPBACK
      address-family l2vpn evpn
        nexthop route-map NEXT-HOP-UNCH
        retain route-target all
      template peer OLEAFS
        update-source loopback1
        ebgp-multihop 3
        address-family l2vpn evpn
          disable-peer-as-check
          send-community
          send-community extended
          route-map NEXT-HOP-UNCH out
      template peer ULEAFS
        address-family ipv4 unicast
          allowas-in 3
          disable-peer-as-check
      neighbor 10.0.101.1
        inherit peer OLEAFS
        remote-as 4200000001
      neighbor 10.0.102.1
        inherit peer OLEAFS
        remote-as 4200000002
      neighbor 10.0.103.1
        inherit peer OLEAFS
        remote-as 4200000003
      neighbor 10.1.1.1
        inherit peer ULEAFS
        remote-as 4200000001
        update-source Ethernet1/1
      neighbor 10.1.1.3
        inherit peer ULEAFS
        remote-as 4200000002
        update-source Ethernet1/2
      neighbor 10.1.1.5
        inherit peer ULEAFS
        remote-as 4200000003
        update-source Ethernet1/3
#### Spine 2
    router bgp 4200000000
      router-id 10.0.2.1
      address-family ipv4 unicast
        redistribute direct route-map LOOPBACK
      address-family l2vpn evpn
        nexthop route-map NEXT-HOP-UNCH
        retain route-target all
      template peer OLEAFS
        update-source loopback1
        ebgp-multihop 3
        address-family l2vpn evpn
          disable-peer-as-check
          send-community
          send-community extended
          route-map NEXT-HOP-UNCH out
      template peer ULEAFS
        address-family ipv4 unicast
          allowas-in 3
          disable-peer-as-check
      neighbor 10.0.101.1
        inherit peer OLEAFS
        remote-as 4200000001
      neighbor 10.0.102.1
        inherit peer OLEAFS
        remote-as 4200000002
      neighbor 10.0.103.1
        inherit peer OLEAFS
        remote-as 4200000003
      neighbor 10.1.2.1
        inherit peer ULEAFS
        remote-as 4200000001
        update-source Ethernet1/1
      neighbor 10.1.2.3
        inherit peer ULEAFS
        remote-as 4200000002
        update-source Ethernet1/2
      neighbor 10.1.2.5
        inherit peer ULEAFS
        remote-as 4200000003
        update-source Ethernet1/3
#### Leaf 1
    interface nve1
      no shutdown
      host-reachability protocol bgp
      source-interface loopback2
      member vni 10010
    router bgp 4200000001
      router-id 10.0.101.1
      address-family ipv4 unicast
        redistribute direct route-map LOOPBACK
      template peer OSPINE
        remote-as 4200000000
        update-source loopback1
        ebgp-multihop 3
        address-family l2vpn evpn
          disable-peer-as-check
          send-community
          send-community extended
          route-map NEXT-HOP-UNCH out
      template peer USPINE
        remote-as 4200000000
        address-family ipv4 unicast
          allowas-in 3
          disable-peer-as-check
      neighbor 10.0.1.1
        inherit peer OSPINE
      neighbor 10.0.2.1
        inherit peer OSPINE
      neighbor 10.1.1.0
        inherit peer USPINE
        update-source Ethernet1/1
      neighbor 10.1.2.0
        inherit peer USPINE
        update-source Ethernet1/2  
#### Leaf 2
    interface nve1
      no shutdown
      host-reachability protocol bgp
      source-interface loopback2
      member vni 10020
    router bgp 4200000002
      router-id 10.0.102.1
      address-family ipv4 unicast
        redistribute direct route-map LOOPBACK
      address-family l2vpn evpn
        nexthop route-map NEXT-HOP-UNCH
        retain route-target all
      template peer OSPINE
        remote-as 4200000000
        update-source loopback1
        ebgp-multihop 3
        address-family l2vpn evpn
          disable-peer-as-check
          send-community
          send-community extended
          route-map NEXT-HOP-UNCH out
      template peer USPINE
        remote-as 4200000000
        address-family ipv4 unicast
          allowas-in 3
          disable-peer-as-check
      neighbor 10.0.1.1
        inherit peer OSPINE
      neighbor 10.0.2.1
        inherit peer OSPINE
      neighbor 10.1.1.2
        inherit peer USPINE
        update-source Ethernet1/1
      neighbor 10.1.2.2
        inherit peer USPINE
        update-source Ethernet1/2
#### Leaf 3
    interface nve1
      no shutdown
      host-reachability protocol bgp
      source-interface loopback2
      member vni 10010
      member vni 10020
    router bgp 4200000003
      router-id 10.0.103.1
      address-family ipv4 unicast
        redistribute direct route-map LOOPBACK
      address-family l2vpn evpn
        nexthop route-map NEXT-HOP-UNCH
        retain route-target all
      template peer OSPINE
        remote-as 4200000000
        update-source loopback1
        ebgp-multihop 3
        address-family l2vpn evpn
          disable-peer-as-check
          send-community
          send-community extended
          route-map NEXT-HOP-UNCH out
      template peer USPINE
        remote-as 4200000000
        address-family ipv4 unicast
          allowas-in 3
          disable-peer-as-check
      neighbor 10.0.1.1
        inherit peer OSPINE
      neighbor 10.0.2.1
        inherit peer OSPINE
      neighbor 10.1.1.4
        inherit peer USPINE
        update-source Ethernet1/1
      neighbor 10.1.2.4
        inherit peer USPINE
        update-source Ethernet1/2  
#### Проверка доступности
    Leaf3# show bgp l2vpn evpn
    BGP routing table information for VRF default, address family L2VPN EVPN
    BGP table version is 1200, Local Router ID is 10.0.103.1
    Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
    Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
    njected
    Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
    est2

       Network            Next Hop            Metric     LocPrf     Weight Path
    Route Distinguisher: 10.0.101.1:3
    *>e[5]:[0]:[0]:[24]:[10.2.10.0]/224
                          10.0.0.1                                       0 420000000
    0 4200000001 ?
    * e                   10.0.0.1                                       0 420000000
    0 4200000001 ?
    *>e[5]:[0]:[0]:[24]:[10.2.20.0]/224
                          10.0.0.1                                       0 420000000
    0 4200000001 ?
    * e                   10.0.0.1                                       0 420000000
    0 4200000001 ?

    Route Distinguisher: 10.0.101.1:32777
    *>e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    * e                   10.0.0.1                                       0 420000000
    0 4200000001 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000001 i
    *>e[2]:[0]:[0]:[48]:[0050.7966.6806]:[32]:[10.2.10.11]/272
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    * e                   10.0.0.1                                       0 420000000
    0 4200000001 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[32]:[10.2.10.15]/272
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000001 i

    Route Distinguisher: 10.0.101.1:32787
    *>e[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    * e                   10.0.0.1                                       0 420000000
    0 4200000001 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000001 i
    * e[2]:[0]:[0]:[48]:[0050.7966.6807]:[32]:[10.2.20.12]/272
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000001 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[32]:[10.2.20.15]/272
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000001 i

    Route Distinguisher: 10.0.102.1:3
    * e[5]:[0]:[0]:[24]:[10.2.10.0]/224
                          10.0.0.1                                       0 420000000
    0 4200000002 ?
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 ?
    * e[5]:[0]:[0]:[24]:[10.2.20.0]/224
                          10.0.0.1                                       0 420000000
    0 4200000002 ?
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 ?

    Route Distinguisher: 10.0.102.1:32777
    * e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i
    * e[2]:[0]:[0]:[48]:[0050.7966.6806]:[32]:[10.2.10.11]/272
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[32]:[10.2.10.15]/272
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i

    Route Distinguisher: 10.0.102.1:32787
    * e[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e[2]:[0]:[0]:[48]:[0050.7966.6807]:[32]:[10.2.20.12]/272
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    * e                   10.0.0.1                                       0 420000000
    0 4200000002 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[32]:[10.2.20.15]/272
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i

    Route Distinguisher: 10.0.103.1:32777    (L2VNI 10010)
    * e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000001 i
    *>l[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                          10.0.103.2                        100      32768 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i
    * e[2]:[0]:[0]:[48]:[0050.7966.6806]:[32]:[10.2.10.11]/272
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i
    *>l[2]:[0]:[0]:[48]:[0050.7966.6808]:[32]:[10.2.10.13]/272
                          10.0.103.2                        100      32768 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[32]:[10.2.10.15]/272
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i

    Route Distinguisher: 10.0.103.1:32787    (L2VNI 10020)
    * e[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i
    *>l[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                          10.0.103.2                        100      32768 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000001 i
    * e[2]:[0]:[0]:[48]:[0050.7966.6807]:[32]:[10.2.20.12]/272
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000001 i
    *>l[2]:[0]:[0]:[48]:[0050.7966.6809]:[32]:[10.2.20.14]/272
                          10.0.103.2                        100      32768 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[32]:[10.2.20.15]/272
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000001 i

    Route Distinguisher: 10.0.103.1:3    (L3VNI 900001)
    * e[2]:[0]:[0]:[48]:[0050.7966.6806]:[32]:[10.2.10.11]/272
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i
    * e[2]:[0]:[0]:[48]:[0050.7966.6807]:[32]:[10.2.20.12]/272
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000001 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[32]:[10.2.10.15]/272
                          10.0.0.1                                       0 420000000
    0 4200000001 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000002 i
    * e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[32]:[10.2.20.15]/272
                          10.0.0.1                                       0 420000000
    0 4200000002 i
    *>e                   10.0.0.1                                       0 420000000
    0 4200000001 i
    *>l[5]:[0]:[0]:[24]:[10.2.10.0]/224
                          10.0.103.2               0        100      32768 ?
    * e                   10.0.0.1                                       0 420000000
    0 4200000002 ?
    * e                   10.0.0.1                                       0 420000000
    0 4200000001 ?
    *>l[5]:[0]:[0]:[24]:[10.2.20.0]/224
                          10.0.103.2               0        100      32768 ?
    * e                   10.0.0.1                                       0 420000000
    0 4200000002 ?
    * e                   10.0.0.1                                       0 420000000
    0 4200000001 ?
