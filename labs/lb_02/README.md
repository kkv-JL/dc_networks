### Цели:

- I:  Настроите OSPF в Underlay сети, для IP связанности между всеми сетевыми устройствами.
- II:  Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
- III: Убедитесь в наличии IP связанности между устройствами в OSFP домене

<img width="1438" height="777" alt="image" src="https://github.com/user-attachments/assets/4773b4ba-90e0-47ec-b6f8-3b360accf91d" />
</br>

#### 1. План OSPF в Underlay сети
##### 1.1 коммутаторы в AREA

|AREA_0|AREA_1
|---|---|
SPINE-SW_01|-----
SPINE-SW_02|SPINE-SW_02
LEAF-SW_01|-----
LEAF-SW_02|-----
LEAF-SW_03|LEAF-SW_03

##### 1.2 p2p ip адреса в AREA

||SPINE-SW_01|SPINE-SW_02|LEAF-SW_01|LEAF-SW_02|LEAF-SW_03|
|---|---|---|---|---|---|
|**AREA_0**|10.99.101.0/31</br>10.99.101.4/31</br>10.99.101.8/31|10.99.101.2/31</br>10.99.101.6/31|10.99.101.1/31<br>10.99.101.3/31|10.99.101.5/31</br>10.99.101.7/31|10.99.101.9/31
|**AREA_1**|-----|10.99.101.10/31|-----|-----|10.99.101.11/31

##### 1.3 Lo0 ip адреса в AREA

||SPINE-SW_01|SPINE-SW_02|LEAF-SW_01|LEAF-SW_02|LEAF-SW_03|
|---|---|---|---|---|---|
|**AREA_0**|10.99.100.1/32|10.99.100.2/32|10.99.100.3/32|10.99.100.4/32|-----
|**AREA_1**|-----|-----|-----|-----|10.99.100.5/32
</br>

#### 2. Настройка Коммутаторов</br>
- SPINE-SW_01
```
interface Ethernet6
   no switchport
   ip address 10.99.101.8/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet7
   no switchport
   ip address 10.99.101.4/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   no switchport
   ip address 10.99.101.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.99.100.1/32
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip ospf area 0.0.0.0
```

- LEAF-SW_01
```
interface Ethernet7
   no switchport
   ip address 10.99.101.3/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   no switchport
   ip address 10.99.101.1/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.99.100.3/32
   ip ospf area 0.0.0.0
```

##### `Cразу после установки соседства, мы можем увидеть обмен пакетами и LSA первого типа`

<img width="922" height="353" alt="Screenshot_15" src="https://github.com/user-attachments/assets/b8c90fa7-1405-4dc2-be12-dcb80d87fd97" /> <br>

<img width="576" height="284" alt="Screenshot_14" src="https://github.com/user-attachments/assets/eaa8302f-a1b9-49e4-9482-b68049160025" /> <br>

- SPINE-SW_02
```
interface Ethernet6
   no switchport
   ip address 10.99.101.10/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
!
interface Ethernet7
   no switchport
   ip address 10.99.101.6/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   no switchport
   ip address 10.99.101.2/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.99.100.2/32
   ip ospf area 0.0.0.0
```
> интерфейсу `Ethernet6` присваиваем `AREA 1`

- LEAF-SW_02
```
interface Ethernet7
   no switchport
   ip address 10.99.101.7/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   no switchport
   ip address 10.99.101.5/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.99.100.4/32
   ip ospf area 0.0.0.0
```

- LEAF-SW_03
```
interface Ethernet7
   no switchport
   ip address 10.99.101.11/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.1
!
interface Ethernet8
   no switchport
   ip address 10.99.101.9/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.99.100.5/32
   ip ospf area 0.0.0.1
```
> интерфейсу `Ethernet7` и `Loopback0` присваиваем `AREA 1`

##### `Через некоторое время, мы можем наблюдать на LEAF-SW_01 обновление информации и LSA третьего типа`

<img width="923" height="375" alt="Screenshot_16" src="https://github.com/user-attachments/assets/67fd3b93-54c8-4dc2-ae2e-b74515f20f00" /> <br>

<img width="592" height="377" alt="Screenshot_17" src="https://github.com/user-attachments/assets/eae74857-113c-40ba-9bbc-e7c135bd3a94" /> <br>

#### 3. Проверка связности
##### 3.1 Проверка доступности между `LEAF-SW_01` и `LEAF-SW_02` (в одной AREA)

```
LEAF-SW1#ping 10.99.100.4 source loopback 0
PING 10.99.100.4 (10.99.100.4) from 10.99.100.3 : 72(100) bytes of data.
80 bytes from 10.99.100.4: icmp_seq=1 ttl=63 time=26.7 ms
80 bytes from 10.99.100.4: icmp_seq=2 ttl=63 time=20.1 ms
80 bytes from 10.99.100.4: icmp_seq=3 ttl=63 time=18.8 ms
80 bytes from 10.99.100.4: icmp_seq=4 ttl=63 time=21.7 ms
80 bytes from 10.99.100.4: icmp_seq=5 ttl=63 time=24.8 ms

--- 10.99.100.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 90ms
rtt min/avg/max/mdev = 18.888/22.480/26.726/2.911 ms, pipe 2, ipg/ewma 22.636/24.649 ms
```

##### 3.2 Проверка доступности между `LEAF-SW_01` и `LEAF-SW_03` (между AREA_0 и AREA_1)
```
LEAF-SW1#ping 10.99.100.5 source loopback 0
PING 10.99.100.5 (10.99.100.5) from 10.99.100.3 : 72(100) bytes of data.
80 bytes from 10.99.100.5: icmp_seq=1 ttl=63 time=35.1 ms
80 bytes from 10.99.100.5: icmp_seq=2 ttl=63 time=35.1 ms
80 bytes from 10.99.100.5: icmp_seq=3 ttl=63 time=28.5 ms
80 bytes from 10.99.100.5: icmp_seq=4 ttl=63 time=19.2 ms
80 bytes from 10.99.100.5: icmp_seq=5 ttl=63 time=19.2 ms

--- 10.99.100.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 91ms
rtt min/avg/max/mdev = 19.274/27.475/35.136/7.109 ms, pipe 3, ipg/ewma 22.877/30.789 ms
```

#### 4. Таблицы маршрутов и ospf database
##### 4.1 Таблица маршрутов на `LEAF-SW_01`
```
LEAF-SW1#show ip ro

VRF: default
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 O        10.99.100.1/32 [110/20] via 10.99.101.0, Ethernet8
 O        10.99.100.2/32 [110/20] via 10.99.101.2, Ethernet7
 C        10.99.100.3/32 is directly connected, Loopback0
 O        10.99.100.4/32 [110/30] via 10.99.101.2, Ethernet7
                                  via 10.99.101.0, Ethernet8
 O IA     10.99.100.5/32 [110/30] via 10.99.101.2, Ethernet7
                                  via 10.99.101.0, Ethernet8
 C        10.99.101.0/31 is directly connected, Ethernet8
 C        10.99.101.2/31 is directly connected, Ethernet7
 O        10.99.101.4/31 [110/20] via 10.99.101.0, Ethernet8
 O        10.99.101.6/31 [110/20] via 10.99.101.2, Ethernet7
 O        10.99.101.8/31 [110/20] via 10.99.101.0, Ethernet8
 O IA     10.99.101.10/31 [110/20] via 10.99.101.2, Ethernet7
```
> Как можем наблюдать, маршруты до `Lo0 10.99.100.5/32` и `p2p 10.99.101.10/31` в `inter area`

##### 4.2 Таблица маршрутов на `LEAF-SW_03`
```
LEAF-SW3#show ip ro

VRF: default
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 O        10.99.100.1/32 [110/20] via 10.99.101.8, Ethernet8
 O        10.99.100.2/32 [110/40] via 10.99.101.8, Ethernet8
 O        10.99.100.3/32 [110/30] via 10.99.101.8, Ethernet8
 O        10.99.100.4/32 [110/30] via 10.99.101.8, Ethernet8
 C        10.99.100.5/32 is directly connected, Loopback0
 O        10.99.101.0/31 [110/20] via 10.99.101.8, Ethernet8
 O        10.99.101.2/31 [110/30] via 10.99.101.8, Ethernet8
 O        10.99.101.4/31 [110/20] via 10.99.101.8, Ethernet8
 O        10.99.101.6/31 [110/30] via 10.99.101.8, Ethernet8
 C        10.99.101.8/31 is directly connected, Ethernet8
 C        10.99.101.10/31 is directly connected, Ethernet7
```
> Видим, что все маршруты в рамках одной AREA, так как у нас есть p2p между `SPINE-SW_01` и `LEAF-SW_03` - <br> `10.99.101.8 <---> 10.99.101.9`, а он, в свою очередь, в `AREA_0` <br>
> Но стоит только на `SPINE-SW_01` имитировать обрыв
```
SPINE-SW1#conf t
SPINE-SW1(config)#int e6
SPINE-SW1(config-if-Et6)#shutdown
```
> Как произойдет перестроение маршрутов
```
LEAF-SW3#show ip ro

VRF: default
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 O IA     10.99.100.1/32 [110/40] via 10.99.101.10, Ethernet7
 O IA     10.99.100.2/32 [110/20] via 10.99.101.10, Ethernet7
 O IA     10.99.100.3/32 [110/30] via 10.99.101.10, Ethernet7
 O IA     10.99.100.4/32 [110/30] via 10.99.101.10, Ethernet7
 C        10.99.100.5/32 is directly connected, Loopback0
 O IA     10.99.101.0/31 [110/30] via 10.99.101.10, Ethernet7
 O IA     10.99.101.2/31 [110/20] via 10.99.101.10, Ethernet7
 O IA     10.99.101.4/31 [110/30] via 10.99.101.10, Ethernet7
 O IA     10.99.101.6/31 [110/20] via 10.99.101.10, Ethernet7
 C        10.99.101.10/31 is directly connected, Ethernet7
```
> И на данный момент все маршруты имеют статус `inter area`

##### 4.3 ospf database на `LEAF-SW_01`
```
LEAF-SW1#show ip ospf database 

            OSPF Router with ID(10.99.100.3) (Instance ID 1) (VRF default)


                 Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age         Seq#         Checksum Link count
10.99.100.4     10.99.100.4     255         0x80000011   0xd755   5
10.99.100.2     10.99.100.2     224         0x80000012   0x2112   5
10.99.100.1     10.99.100.1     397         0x8000001a   0xf880   7
10.99.100.5     10.99.100.5     398         0x8000000f   0x1caf   2
10.99.100.3     10.99.100.3     995         0x80000013   0xaf8e   5

                 Summary Link States (Area 0.0.0.0)

Link ID         ADV Router      Age         Seq#         Checksum
10.99.100.5     10.99.100.5     398         0x8000000b   0xd9a1  
10.99.100.5     10.99.100.2     44          0x8000000b   0x5024  
10.99.101.10    10.99.100.5     398         0x8000000b   0x96df  
10.99.101.10    10.99.100.2     224         0x8000000b   0xa8d0
```

##### 4.3 ospf database на `LEAF-SW_03`
```
LEAF-SW3#show ip ospf database 

            OSPF Router with ID(10.99.100.5) (Instance ID 1) (VRF default)


                 Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age         Seq#         Checksum Link count
10.99.100.2     10.99.100.2     297         0x80000012   0x2112   5
10.99.100.4     10.99.100.4     326         0x80000011   0xd755   5
10.99.100.3     10.99.100.3     1068        0x80000013   0xaf8e   5
10.99.100.1     10.99.100.1     468         0x8000001a   0xf880   7
10.99.100.5     10.99.100.5     467         0x8000000f   0x1caf   2

                 Summary Link States (Area 0.0.0.0)

Link ID         ADV Router      Age         Seq#         Checksum
10.99.101.10    10.99.100.2     297         0x8000000b   0xa8d0  
10.99.100.5     10.99.100.5     467         0x8000000b   0xd9a1  
10.99.100.5     10.99.100.2     117         0x8000000b   0x5024  
10.99.101.10    10.99.100.5     467         0x8000000b   0x96df  

                 Router Link States (Area 0.0.0.1)

Link ID         ADV Router      Age         Seq#         Checksum Link count
10.99.100.2     10.99.100.2     115         0x8000000d   0xb21a   2
10.99.100.5     10.99.100.5     479         0x8000000e   0x4f87   3

                 Summary Link States (Area 0.0.0.1)

Link ID         ADV Router      Age         Seq#         Checksum
10.99.100.4     10.99.100.2     295         0x8000000b   0x5a1b  
10.99.100.1     10.99.100.2     295         0x8000000b   0xdc91  
10.99.100.2     10.99.100.2     295         0x8000000b   0xa77   
10.99.100.4     10.99.100.5     459         0x80000001   0xc0b1  
10.99.100.2     10.99.100.5     459         0x80000001   0x3931  
10.99.100.3     10.99.100.2     295         0x8000000b   0x6412  
10.99.100.1     10.99.100.5     459         0x80000001   0x7a05  
10.99.100.3     10.99.100.5     459         0x80000001   0xcaa8  
10.99.101.2     10.99.100.2     295         0x8000000b   0xf888  
10.99.101.4     10.99.100.2     295         0x8000000b   0x492c  
10.99.101.8     10.99.100.5     479         0x80000001   0xbec3  
10.99.101.0     10.99.100.5     459         0x80000001   0x730d  
10.99.101.0     10.99.100.2     295         0x8000000b   0x7108  
10.99.101.8     10.99.100.2     475         0x80000001   0x99d7  
10.99.101.4     10.99.100.5     459         0x80000001   0x4b31  
10.99.101.6     10.99.100.2     295         0x8000000b   0xd0ac  
10.99.101.2     10.99.100.5     459         0x80000001   0xc3b0  
10.99.101.6     10.99.100.5     459         0x80000001   0x9bd4
```
