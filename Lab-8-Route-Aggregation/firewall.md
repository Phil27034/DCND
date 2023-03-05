show run
Building configuration...

Current configuration : 1550 bytes
!
! Last configuration change at 20:25:47 UTC Sun Mar 5 2023
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname firewall
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!


!
!
!
!
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
redundancy
!
!
! 
!
!
!
!
!
!
!
!
!
!
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.0
!
interface Loopback1
 ip address 1.1.2.1 255.255.255.0
!
interface Loopback2
 ip address 1.1.3.1 255.255.255.0
!
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.1
!
interface Ethernet0/0.2
 encapsulation dot1Q 2
 ip address 192.168.5.2 255.255.255.252
!
interface Ethernet0/0.3
 encapsulation dot1Q 3
 ip address 192.168.6.2 255.255.255.252
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
router bgp 64800
 bgp log-neighbor-changes
 network 1.1.0.0 mask 255.255.0.0
 network 1.1.1.0 mask 255.255.255.0
 network 1.1.2.0 mask 255.255.255.0
 network 1.1.3.0 mask 255.255.255.0
 aggregate-address 1.1.0.0 255.255.0.0 as-set summary-only
 neighbor 192.168.5.1 remote-as 64705
 neighbor 192.168.6.1 remote-as 64704
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
!
!
!
control-plane
!
!
!
!
!
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input all
!
!
end

firewall#