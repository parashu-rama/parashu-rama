# Лабораторная работа 7
## Реализовать передачу суммарных префиксов через EVPN route-type 5
![Схема сети](net.png "Схема сети")
### Схема eBGP для underlay и overlay
    AS 4200000000 - Spines
    AS 4200000001 - Leaf1
    AS 4200000002 - Leaf2
    AS 4200000003 - Leaf3
### Нумерация VNI
    Yxxxx  - где xxxx - номер Vlan, Y - номер VRF
### IP Адресация
    10.Dn.Sn.X/31, где:
    * Dn – номер ЦОДа,
    * Sn – 0 для vPC secondary ip, 1-100 spine, 101-255 leaf,
    * X – 1-2 для loopback, для p2p по порядку, для vPC secondary ip но номеру домена.
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
          send-community
          send-community extended
          route-map NEXT-HOP-UNCH out
      template peer ULEAFS
        address-family ipv4 unicast
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
          send-community
          send-community extended
          route-map NEXT-HOP-UNCH out
      template peer ULEAFS
        address-family ipv4 unicast
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
    vlan 10
      vn-segment 20010
    vlan 20
      vn-segment 30020
    vlan 210
      vn-segment 20210
    vlan 320
      vn-segment 30320
      
    vrf context 10.2
      vni 20210
      rd auto
      address-family ipv4 unicast
        route-target both auto
        route-target both auto evpn
    vrf context 10.3
      vni 30320
      rd auto
      address-family ipv4 unicast
        route-target both auto
        route-target both auto evpn    
    
    interface Vlan10
      no shutdown
      vrf member 10.2
      no ip redirects
      ip address 10.2.10.1/24
      no ipv6 redirects
      fabric forwarding mode anycast-gateway

    interface Vlan20
      no shutdown
      vrf member 10.3
      no ip redirects
      ip address 10.3.20.1/24
      no ipv6 redirects
      fabric forwarding mode anycast-gateway

    interface Vlan210
      no shutdown
      vrf member 10.2
      no ip redirects
      ip forward
      no ipv6 redirects

    interface Vlan320
      no shutdown
      vrf member 10.3
      no ip redirects
      ip forward
      no ipv6 redirects
      
    interface nve1
      no shutdown
      host-reachability protocol bgp
      source-interface loopback2
      member vni 20010
      member vni 20210 associate-vrf
      member vni 30020
      member vni 30320 associate-vrf
      
    router bgp 4200000001
      router-id 10.0.101.1
      address-family ipv4 unicast
        redistribute direct route-map LOOPBACK
      template peer OSPINE
        remote-as 4200000000
        update-source loopback1
        ebgp-multihop 3
        address-family l2vpn evpn
          send-community
          send-community extended
          route-map NEXT-HOP-UNCH out
      template peer USPINE
        remote-as 4200000000
        address-family ipv4 unicast
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
      vrf 10.2
        address-family ipv4 unicast
      vrf 10.3
        address-family ipv4 unicast
 #### Leaf 2
    vlan 10
      vn-segment 20010
    vlan 20
      vn-segment 30020
    vlan 210
      vn-segment 20210
    vlan 320
      vn-segment 30320  
      
    vrf context 10.2
      vni 20210
      rd auto
      address-family ipv4 unicast
        route-target both auto
        route-target both auto evpn
    vrf context 10.3
      vni 30320
      rd auto
      address-family ipv4 unicast
        route-target both auto
        route-target both auto evpn      

    interface Vlan10
      no shutdown
      vrf member 10.2
      no ip redirects
      ip address 10.2.10.1/24
      no ipv6 redirects
      fabric forwarding mode anycast-gateway

    interface Vlan20
      no shutdown
      vrf member 10.3
      no ip redirects
      ip address 10.3.20.1/24
      no ipv6 redirects
      fabric forwarding mode anycast-gateway

    interface Vlan210
      no shutdown
      vrf member 10.2
      no ip redirects
      ip forward
      no ipv6 redirects

    interface Vlan320
      no shutdown
      vrf member 10.3
      no ip redirects
      ip forward
      no ipv6 redirects
      
    interface nve1
      no shutdown
      host-reachability protocol bgp
      source-interface loopback2
      global mcast-group 239.0.0.1 L2
      member vni 20010
      member vni 20210 associate-vrf
      member vni 30020
      member vni 30320 associate-vrf
  
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
      send-community
      send-community extended
      route-map NEXT-HOP-UNCH out
  template peer USPINE
    remote-as 4200000000
    address-family ipv4 unicast
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
  vrf 10.2
    address-family ipv4 unicast
  vrf 10.3
    address-family ipv4 unicast
 #### Border Leaf 3
    vlan 10
      vn-segment 20010
    vlan 20
      vn-segment 30020
    vlan 210
      vn-segment 20210
    vlan 320
      vn-segment 30320

    interface Vlan10
      no shutdown
      vrf member 10.2
      ip address 10.2.10.1/24
      fabric forwarding mode anycast-gateway

    interface Vlan20
      no shutdown
      vrf member 10.3
      ip address 10.3.20.1/24
      fabric forwarding mode anycast-gateway

    interface Vlan210
      no shutdown
      vrf member 10.2
      ip forward

    interface Vlan320
      no shutdown
      vrf member 10.3
      ip forward

    interface nve1
      no shutdown
      host-reachability protocol bgp
      source-interface loopback2
      global mcast-group 239.0.0.1 L2
      member vni 20010
      member vni 20210 associate-vrf
      member vni 30020
      member vni 30320 associate-vrf
      
    interface Ethernet1/5.110
      encapsulation dot1q 110
      vrf member 10.2
      ip address 10.1.3.1/31
      no shutdown

    interface Ethernet1/5.120
      encapsulation dot1q 120
      vrf member 10.3
      ip address 10.1.3.3/31
      no shutdown      
      
router bgp 4200000003
  router-id 10.1.3.1
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
      send-community
      send-community extended
      route-map NEXT-HOP-UNCH out
  template peer USPINE
    remote-as 4200000000
    address-family ipv4 unicast
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
  vrf 10.2
    address-family ipv4 unicast
      aggregate-address 10.2.0.0/16 summary-only
    neighbor 10.1.3.0
      remote-as 65000.65000
      local-as 65000.10 no-prepend replace-as
      address-family ipv4 unicast
  vrf 10.3
    address-family ipv4 unicast
      aggregate-address 10.3.0.0/16 summary-only
    neighbor 10.1.3.2
      remote-as 65000.65000
      local-as 65000.20 no-prepend replace-as
      address-family ipv4 unicast
#### Edge Router
    interface Ethernet0/0.110
     encapsulation dot1Q 110
     ip address 10.1.3.0 255.255.255.254
    !
    interface Ethernet0/0.120
     encapsulation dot1Q 120
     ip address 10.1.3.2 255.255.255.254
    !
    interface Ethernet0/1
     no ip address
     shutdown
    !
    interface Ethernet0/2
     no ip address
     shutdown
    !
    interface Ethernet0/3
     no ip address
     shutdown
    !
    router bgp 4259905000
     bgp log-neighbor-changes
     neighbor 10.1.3.1 remote-as 4259840010
     neighbor 10.1.3.3 remote-as 4259840020
#### Проверка route-type 5
    Leaf1# sh bgp l2vpn evpn route-type 5 vrf 10.2
    Route Distinguisher: 10.0.101.1:6    (L3VNI 20210)
    BGP routing table entry for [5]:[0]:[0]:[16]:[10.2.0.0]/224, version 3491
    Paths: (1 available, best #1)
    Flags: (0x000002) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not i
    n HW

      Advertised path-id 1
      Path type: external, path is valid, is best path, no labeled nexthop
                 Imported from 10.1.3.1:3:[5]:[0]:[0]:[16]:[10.2.0.0]/224
      Gateway IP: 0.0.0.0
      AS-Path: 4200000000 4200000003 , path sourced external to AS
        10.0.103.2 (metric 0) from 10.0.1.1 (10.0.1.1)
          Origin IGP, MED not set, localpref 100, weight 0
          Aggregated by 10.2.10.1, aggregator AS 4200000003, atomic-aggregate set
          Received label 20210
          Extcommunity: RT:23456:20210 ENCAP:8 Router MAC:5005.0000.1b08

      Path-id 1 not advertised to any peer
    BGP routing table entry for [5]:[0]:[0]:[16]:[10.3.0.0]/224, version 3510
    Paths: (1 available, best #1)
    Flags: (0x000002) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not i
    n HW

      Advertised path-id 1
      Path type: external, path is valid, is best path, no labeled nexthop
                 Imported from 10.1.3.1:3:[5]:[0]:[0]:[16]:[10.3.0.0]/224
      Gateway IP: 0.0.0.0
      AS-Path: 4200000000 4200000003 4259905000 4259840020 , path sourced external t
    o AS
        10.0.103.2 (metric 0) from 10.0.2.1 (10.0.2.1)
          Origin IGP, MED not set, localpref 100, weight 0
          Aggregated by 10.3.20.1, aggregator AS 4200000003, atomic-aggregate set
          Received label 20210
          Extcommunity: RT:23456:20210 ENCAP:8 Router MAC:5005.0000.1b08

      Path-id 1 not advertised to any peer
