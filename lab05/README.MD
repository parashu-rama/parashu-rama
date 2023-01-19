# Лабораторная работа 5
## Overlay на основе VxLAN EVPN для L3 связанности между клиентами
![Схема сети](net.png "Схема сети")
### Схема eBGP для underlay и overlay
    AS 4200000000 - Spines
    AS 4200000001 - Leaf1
    AS 4200000002 - Leaf2
    AS 4200000003 - Leaf3
### Нумерация VNI
    1xxxx  - где xxxx - номер Vlan
    900001 - vrf для L3 связанности
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
    nv overlay evpn
    feature bgp
    feature fabric forwarding
    feature interface-vlan
    feature vn-segment-vlan-based
    feature nv overlay
    fabric forwarding anycast-gateway-mac 0000.2222.3333
    
    vlan 10
      vn-segment 10010 
    vlan 101
      vn-segment 900001
      
    route-map HOST-SVI permit 10
      match interface Vlan10 
    route-map LOOPBACK permit 10
      match interface loopback1 loopback2 
    route-map NEXT-HOP-UNCH permit 10
      set ip next-hop unchanged

    vrf context vxlan-900001
      vni 900001
      rd auto
      
    interface Vlan10
      no shutdown
      vrf member vxlan-900001
      ip address 10.2.10.1/24
      fabric forwarding mode anycast-gateway
      
    interface Vlan101
      no shutdown
      vrf member vxlan-900001
      no ip redirects
      ip forward
      ipv6 address use-link-local-only
      no ipv6 redirects
      
    interface nve1
      no shutdown
      host-reachability protocol bgp
      source-interface loopback2
      member vni 10010
      member vni 900001 associate-vrf
      
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
      vrf vxlan-900001
        address-family ipv4 unicast
          redistribute direct route-map HOST-SVI
 #### Leaf 2
    nv overlay evpn
    feature bgp
    feature fabric forwarding
    feature interface-vlan
    feature vn-segment-vlan-based
    feature nv overlay
    fabric forwarding anycast-gateway-mac 0000.2222.3333
    
    vlan 20
      vn-segment 10020
    vlan 101
      vn-segment 900001
      
    route-map HOST-SVI permit 10
      match interface Vlan20
    route-map LOOPBACK permit 10
      match interface loopback1 loopback2
    route-map NEXT-HOP-UNCH permit 10
      set ip next-hop unchanged
      
    vrf context vxlan-900001
      vni 900001
      rd auto
      
    interface Vlan20
      no shutdown
      vrf member vxlan-900001
      ip address 10.2.20.1/24
      fabric forwarding mode anycast-gateway
      
    interface Vlan101
      no shutdown
      vrf member vxlan-900001
      no ip redirects
      ip forward
      ipv6 address use-link-local-only
      no ipv6 redirects
      
    interface nve1
      no shutdown
      host-reachability protocol bgp
      source-interface loopback2
      member vni 10020
      member vni 900001 associate-vrf
      
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
      vrf vxlan-900001
        address-family ipv4 unicast
          redistribute direct route-map HOST-SVI
 #### Leaf 3
    nv overlay evpn
    feature bgp
    feature fabric forwarding
    feature interface-vlan
    feature vn-segment-vlan-based
    feature nv overlay
    fabric forwarding anycast-gateway-mac 0000.2222.3333
    
    vlan 10
      vn-segment 10010
    vlan 20
      vn-segment 10020
    vlan 101
      vn-segment 900001

    route-map HOST-SVI permit 10
      match interface Vlan10 Vlan20
    route-map LOOPBACK permit 10
      match interface loopback1 loopback2
    route-map NEXT-HOP-UNCH permit 10
      set ip next-hop unchanged

    vrf context vxlan-900001
      vni 900001
      rd auto

    interface Vlan10
      no shutdown
      vrf member vxlan-900001
      ip address 10.2.10.1/24
      fabric forwarding mode anycast-gateway

    interface Vlan20
      no shutdown
      vrf member vxlan-900001
      ip address 10.2.20.1/24
      fabric forwarding mode anycast-gateway

    interface Vlan101
      no shutdown
      vrf member vxlan-900001
      no ip redirects
      ip forward
      ipv6 address use-link-local-only
      no ipv6 redirects
      
    interface nve1
      no shutdown
      host-reachability protocol bgp
      source-interface loopback2
      member vni 10010
      member vni 10020
      member vni 900001 associate-vrf
      
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
      vrf vxlan-900001
        address-family ipv4 unicast
          redistribute direct route-map HOST-SVI
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
          
