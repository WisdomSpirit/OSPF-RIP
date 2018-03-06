# OSPF-RIP

## Настроим роутеры и интерфейсы на них:

###R1AR0:
R1AR0>enable
R1AR0#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1AR0(config)#int fa 0/0
R1AR0(config-if)#ip addr 10.2.2.1 255.255.255.0
R1AR0(config-if)#no shu
R1AR0(config-if)#no shutdown 
R1AR0(config-if)#exit
R1AR0(config)#int se 2/0
R1AR0(config-if)#ip addr 10.1.1.1 255.255.255.0
R1AR0(config-if)#no shu
R1AR0(config-if)#no shutdown 
R1AR0(config-if)#end
R1AR0#wr memo
Building configuration...
[OK]

###R2AR0:
R2AR0>enable
R2AR0#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2AR0(config)#int fa 0/0
R2AR0(config-if)#ip addr 10.2.2.2 255.255.255.0
R2AR0(config-if)#no shut
R2AR0(config-if)#exit
R2AR0(config)#int se 2/0
R2AR0(config-if)#ip addr 10.3.3.1 255.255.255.0
R2AR0(config-if)#no shut
R2AR0(config-if)#exit
R2AR0(config)#int loo
R2AR0(config)#int loopback 0
R2AR0(config-if)#ip addr 10.29.0.1 255.255.255.255
R2AR0(config-if)#no shut
R2AR0(config-if)#exit
R2AR0(config)#int loop 1
R2AR0(config-if)#ip addr 10.29.1.1 255.255.255.255
R2AR0(config-if)#no shut
R2AR0(config-if)#exit
R2AR0(config)#int loop 2
R2AR0(config-if)#ip addr 10.29.2.1 255.255.255.255
R2AR0(config-if)#no shut
R2AR0(config-if)#exit
R2AR0(config)#int loop 3
R2AR0(config-if)#ip addr 10.29.3.1 255.255.255.255
R2AR0(config-if)#no shut
R2AR0(config-if)#end
R2AR0#wr memo
Building configuration...
[OK]

###ABR:
ABR>enable
ABR#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
ABR(config)#int se 2/0
ABR(config-if)#ip addr 10.1.1.2 255.255.255.0
ABR(config-if)#no shut
ABR(config-if)#exit
ABR(config)#int se 3/0
ABR(config-if)#ip addr 192.168.1.1 255.255.255.0
ABR(config-if)#no shut
ABR(config-if)#end
ABR#wr memo
Building configuration...
[OK]

## Запустим OSPF:

###R1AR0:
R1AR0>enable
R1AR0#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1AR0(config)#rou ospf 1
R1AR0(config-router)#rou 1.1.1.1
R1AR0(config-router)#net 10.1.1.0 0.0.0.255 area 0
R1AR0(config-router)#net 10.2.2.0 0.0.0.255 area 0
R1AR0(config-router)#end
R1AR0#wr memo
Building configuration...
[OK]

###R2AR0:
R2AR0>enable
R2AR0#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2AR0(config)#rou ospf 1
R2AR0(config-router)#rou 2.2.2.2
R2AR0(config-router)#net 10.2.2.0 0.0.0.255 area 0
R2AR0(config-router)#net 10.29.0.0 0.0.255.255 area 0
R2AR0(config-router)#end
R2AR0#wr memo
Building configuration...
[OK]

###ABR:
ABR>enable
ABR#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
ABR(config)#rou ospf 1
ABR(config-router)#rou 1.1.2.2
ABR(config-router)#net 10.0.0.0 0.255.255.255 area 0
ABR(config-router)#end
ABR#wr memo
Building configuration...
[OK]

##Выполним команду show ip route на R1AR0:
R1AR0#show ip route
Codes: C - connected, S - static, I - IGRP, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
C       10.1.1.0/24 is directly connected, Serial2/0
C       10.2.2.0/24 is directly connected, FastEthernet0/0
O       10.29.0.1/32 [110/2] via 10.2.2.2, 00:48:21, FastEthernet0/0
O       10.29.1.1/32 [110/2] via 10.2.2.2, 00:48:11, FastEthernet0/0
O       10.29.2.1/32 [110/2] via 10.2.2.2, 00:48:11, FastEthernet0/0
O       10.29.3.1/32 [110/2] via 10.2.2.2, 00:48:11, FastEthernet0/0
O IA 192.168.1.0/24 [110/128] via 10.1.1.2, 00:27:26, Serial2/0
O IA 192.168.2.0/24 [110/192] via 10.1.1.2, 00:27:16, Serial2/0

Как интерпретировать «10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks»?
*Маршрутизатор знает о 6 подсетях с таким префиксом, и под него подходят маски 2 сетей*

## Выполним команду show ip ospf database router на а R1AR0:
R1AR0#show ip ospf database router

            OSPF Router with ID (1.1.1.1) (Process ID 1)

                Router Link States (Area 0)

  LS age: 298
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 2.2.2.2
  Advertising Router: 2.2.2.2
  LS Seq Number: 80000011
  Checksum: 0xa287
  Length: 84
  Number of Links: 5

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 10.2.2.2
     (Link Data) Router Interface address: 10.2.2.2
      Number of TOS metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.29.0.1
     (Link Data) Network Mask: 255.255.255.255
      Number of TOS metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.29.1.1
     (Link Data) Network Mask: 255.255.255.255
      Number of TOS metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.29.2.1
     (Link Data) Network Mask: 255.255.255.255
      Number of TOS metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.29.3.1
     (Link Data) Network Mask: 255.255.255.255
      Number of TOS metrics: 0
       TOS 0 Metrics: 1

  LS age: 293
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 1.1.1.1
  Advertising Router: 1.1.1.1
  LS Seq Number: 8000000c
  Checksum: 0xbba9
  Length: 60
  Number of Links: 3

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 10.2.2.2
     (Link Data) Router Interface address: 10.2.2.1
      Number of TOS metrics: 0
       TOS 0 Metrics: 1

    Link connected to: another Router (point-to-point)
     (Link ID) Neighboring Router ID: 1.1.2.2
     (Link Data) Router Interface address: 10.1.1.1
      Number of TOS metrics: 0
       TOS 0 Metrics: 64

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.1.1.0
     (Link Data) Network Mask: 255.255.255.0
      Number of TOS metrics: 0
       TOS 0 Metrics: 64

  LS age: 293
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 1.1.2.2
  Advertising Router: 1.1.2.2
  LS Seq Number: 80000008
  Checksum: 0x7a1a
  Length: 48
  Area Border Router
  Number of Links: 2

    Link connected to: another Router (point-to-point)
     (Link ID) Neighboring Router ID: 1.1.1.1
     (Link Data) Router Interface address: 10.1.1.2
      Number of TOS metrics: 0
       TOS 0 Metrics: 64

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.1.1.0
     (Link Data) Network Mask: 255.255.255.0
      Number of TOS metrics: 0
       TOS 0 Metrics: 64

Какой тип этого LSA? - *Router LSA*
Сколько Router LSA хранится на хранится на R1AR0? Почему именно столько? - *3, так как 2 соседа + сам*
А сколько Router LSA хранится на других маршрутизаторах этой области? - *по 2*
Кто генерирует Router LSA? - *Маршрутизатор*
Какую информацию содержат Router LSA? - *Router LSA содержат информацию о соседях внутри области*
В каких пределах распространяются маршрутизаторами Router LSA (в пределах сети, области, автономной системы и т. п.)? - *Области*

##Выполним команду show ip ospf database network на R1AR0:
R1AR0#show ip ospf database network

            OSPF Router with ID (1.1.1.1) (Process ID 1)

                Net Link States (Area 0)

  Routing Bit Set on this LSA
  LS age: 115
  Options: (No TOS-capability, DC)
  LS Type: Network Links
  Link State ID: 10.2.2.2  (address of Designated Router)
  Advertising Router: 2.2.2.2
  LS Seq Number: 80000004
  Checksum: 0x4ca6
  Length: 32
  Network Mask: /24
        Attached Router: 1.1.1.1
        Attached Router: 2.2.2.2
 Какой тип этого LSA? - *Network LSA*
 Сколько Network LSA хранится на хранится на R1AR0? Почему именно столько? - *1,так как выделен 1 Designated Router 10.2.2.2*
 А сколько Network LSA хранится на других маршрутизаторах этой области? - *1*
 Что такое Designated Router? Зачем он нужен? - *Управляющий роутер.Занимается рассылкой LSA в сети*
 Кто генерирует Network LSA? - *Designated Router*
 Какую информацию содержат Network LSA? - *О местонахождении Designated Router*
 В каких пре- делах распространяются маршрутизаторами Network LSA? - *В пределах области*
 
## Настроим маршрутизаторы в области 1:

###R1AR1:
R1AR1>enable
R1AR1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1AR1(config)#int se 2/0
R1AR1(config-if)#ip addr 192.168.1.2 255.255.255.0
R1AR1(config-if)#no shut
R1AR1(config-if)#exit
R1AR1(config)#int se 3/0
R1AR1(config-if)#ip addr 192.168.2.1 255.255.255.0
R1AR1(config-if)#no shut
R1AR1(config-if)#end
R1AR1#wr memo
Building configuration...
[OK]

###ASBR:
ASBR>enable
ASBR#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
ASBR(config)#int se 2/0
ASBR(config-if)#ip addr 192.168.2.2 255.255.255.0
ASBR(config-if)#no shut
ASBR(config-if)#end
ASBR#wr memo
Building configuration...
[OK]

##Настроим OSPF в области 1:
###R1AR1:
###ASBR:
ASBR#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
ASBR(config)#rou ospf 1
ASBR(config-router)#rou 4.4.4.4
ASBR(config-router)#net 192.168.2.0 0.0.0.255 area 1
ASBR(config-router)#end
ASBR#wr memo
Building configuration...
[OK]
###ABR:
