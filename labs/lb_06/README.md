### Цели:

- I:  Настроить каждого клиента в своем VNI.
- II:  Настроить маршрутизацию между клиентами. </br>

       без VRF
       с использованием VRF

  <img width="1169" height="798" alt="image" src="https://github.com/user-attachments/assets/a5676110-3b92-467f-9f46-b5ce3ec3fdae" />

  ##### `без VRF`
  #### 1. Настройка Коммутаторов </br>
  ##### 1.1 Назначаем и настраиваем порты для конечных пользователей

Vlan_10
|host_name|ip address|gw
|---|---|---|
VPC_host_1|192.168.0.2|192.168.0.1
VPC_host_2|192.168.0.3|192.168.0.1

Vlan_20
|host_name|ip address|gw
|---|---|---|
VPC_host_3|192.168.1.2|192.168.1.1
VPC_host_4|192.168.1.3|192.168.1.1
</br>

На LEAF_1
```
vlan 10
   name vl10
!
vlan 20
   name vl20
!
interface Ethernet1
   switchport access vlan 10
```
</br>

На LEAF_2
```
vlan 10
   name vl10
!
vlan 20
   name vl20
!
interface Ethernet1
   switchport access vlan 20
```
</br>

На LEAF_3
```
vlan 10
   name vl10
!
vlan 20
   name vl20
!
interface Ethernet1
   switchport access vlan 10
!
interface Ethernet2
   switchport access vlan 20
```

  ##### 1.2 Настраиваем маршрутизацию между Vlan_10 и Vlan_20

> Настраиваем шлюзы Vlan на всех LEAF коммутаторах
```
interface Vlan10
   ip address 192.168.0.1/24
!
interface Vlan20
   ip address 192.168.1.1/24
```

> Настраиваем интерфейс Vxlan
```
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 1010
   vxlan vlan 20 vni 1020
```

> Добавляем настройки для vlan mapping в bgp
```
router bgp 65001
   neighbor 10.99.100.1 remote-as 65001
   neighbor 10.99.100.1 update-source Loopback0
   neighbor 10.99.100.1 send-community extended
   neighbor 10.99.100.2 remote-as 65001
   neighbor 10.99.100.2 update-source Loopback0
   neighbor 10.99.100.2 send-community extended
   !
   vlan 10
      rd auto
      route-target both 65001:1010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65001:1020
      redistribute learned
```

`Желательно так же настроить виртуальный mac-адрес`
```
ip virtual-router mac-address 00:00:11:11:00:00 (пример)
```
  ##### 1.3 Проверка связности

Проверяем между 192.168.0.2 и 192.168.1.2
```
VPCS> ping 192.168.1.2

84 bytes from 192.168.1.2 icmp_seq=1 ttl=63 time=768.693 ms
84 bytes from 192.168.1.2 icmp_seq=2 ttl=63 time=46.553 ms
84 bytes from 192.168.1.2 icmp_seq=3 ttl=63 time=61.443 ms
84 bytes from 192.168.1.2 icmp_seq=4 ttl=63 time=136.983 ms
84 bytes from 192.168.1.2 icmp_seq=5 ttl=63 time=53.562 ms
```
Связность есть:
<img width="897" height="291" alt="image" src="https://github.com/user-attachments/assets/a74381b7-c545-4b98-b37f-088c9078e265" />

Сначало ARP запрос, видим broadcast, VNI 1020

<img width="907" height="717" alt="image" src="https://github.com/user-attachments/assets/881bce41-6c2b-4fc7-a0cd-a647d9acc198" />

И сами пакеты ICMP request и reply

<img width="902" height="607" alt="image" src="https://github.com/user-attachments/assets/89248043-7a92-4475-bf72-d013a973dec6" />

<br></br>


##### `с использованием VRF`
  #### 2. Настройка Коммутаторов </br>

##### LEAF_1
Необходимо включить/настроить VRF
```
!
vrf instance VRF1
!
```
> если необходимо включаем ip routing - `ip routing vrf VRF1`

Добавляем VRF в Vxlan интерфейс
```
interface Vxlan1
   vxlan vrf VRF1 vni 9999
```

Добавляем VRF в BGP router
```
router bgp 65001
   vrf VRF1
      rd 10.99.100.3:9999
      route-target import evpn 65001:9999
      route-target export evpn 65001:9999
```
> rd 10.99.100.3:9999 - для удобства `10.99.100.3` указываем ip Lo коммутатора на котором выполняем настройку </br>
> route-target import evpn 65001:9999 - `evpn` обязательная настройка; `65001` указываем AS нашего BGP

Настраиваем Vlan интерфейс
```
interface Vlan10
   vrf VRF1
   ip address virtual 192.168.0.1/24
```

##### LEAF_2
> Симетричная настройка, только помним, что у нас на этом коммутаторе Vlan_20 и Lo - 10.99.100.4
```
vrf instance VRF1
!
interface Vxlan1
   vxlan vrf VRF1 vni 9999
!
router bgp 65001
   vrf VRF1
      rd 10.99.100.4:9999
      route-target import evpn 65001:9999
      route-target export evpn 65001:9999
!
interface Vlan20
   vrf VRF1
   ip address virtual 192.168.1.1/24
```

##### LEAF_3
> На этом коммутаторе 2 хоста в разных Vlan, поэтому делаем настройку Vlan интерфейса и для Vlan_10, и для Vlan_20
```
vrf instance VRF1
!
interface Vxlan1
   vxlan vrf VRF1 vni 9999
!
router bgp 65001
   vrf VRF1
      rd 10.99.100.5:9999
      route-target import evpn 65001:9999
      route-target export evpn 65001:9999
!
interface Vlan10
   vrf VRF1
   ip address virtual 192.168.0.1/24
!
interface Vlan20
   vrf VRF1
   ip address virtual 192.168.1.1/24
```
###### TIPS
> если необходимо "пинговать" шлюзы других сетей, в данном случае с `192.168.0.2 --> 192.168.1.1` и с `192.168.1.2 --> 192.168.0.1`, то необходимо настраивать Vlan интерфейсы на всех LEAF коммутаторах
```
interface Vlan10
   vrf VRF1
   ip address virtual 192.168.0.1/24
!
interface Vlan20
   vrf VRF1
   ip address virtual 192.168.1.1/24
```

##### 2.1 Проверка связности
Проверяем между 192.168.0.2 и 192.168.1.2
```
VPCS> sh              

NAME   IP/MASK              GATEWAY                             GATEWAY
VPCS1  192.168.0.2/24       192.168.0.1
       fe80::250:79ff:fe66:6806/64

VPCS> ping 192.168.1.2

84 bytes from 192.168.1.2 icmp_seq=1 ttl=62 time=54.604 ms
84 bytes from 192.168.1.2 icmp_seq=2 ttl=62 time=48.991 ms
84 bytes from 192.168.1.2 icmp_seq=3 ttl=62 time=206.381 ms
84 bytes from 192.168.1.2 icmp_seq=4 ttl=62 time=42.665 ms
84 bytes from 192.168.1.2 icmp_seq=5 ttl=62 time=47.548 ms
```
Связность есть:

<img width="876" height="139" alt="image" src="https://github.com/user-attachments/assets/257ed038-b2c8-48a7-8e17-d2ec626ff995" />

Можем увидеть VRF vni 9999

<img width="840" height="322" alt="image" src="https://github.com/user-attachments/assets/1a0e7ce6-17f3-4f67-a8f5-3c8a1de3f531" />


##### 3 Проверка связности при "обрыве"

Для проверки имитируем обрыв линка LEAF_1_eth8 <--> SPINE_1_eth8
```
LEAF-SW1(config)#interface ethernet 8
LEAF-SW1(config-if-Et8)#shut
```

На хосте VPC_host_1 включили непрерывный ping
```
VPCS> sh                 

NAME   IP/MASK              GATEWAY                             GATEWAY
VPCS1  192.168.0.2/24       192.168.0.1
       fe80::250:79ff:fe66:6806/64

VPCS> ping 192.168.1.2 -t

84 bytes from 192.168.1.2 icmp_seq=1 ttl=62 time=109.253 ms
84 bytes from 192.168.1.2 icmp_seq=2 ttl=62 time=42.168 ms
84 bytes from 192.168.1.2 icmp_seq=3 ttl=62 time=42.809 ms
84 bytes from 192.168.1.2 icmp_seq=4 ttl=62 time=125.846 ms
84 bytes from 192.168.1.2 icmp_seq=5 ttl=62 time=51.899 ms
192.168.1.2 icmp_seq=6 timeout
84 bytes from 192.168.1.2 icmp_seq=7 ttl=62 time=49.455 ms
84 bytes from 192.168.1.2 icmp_seq=8 ttl=62 time=57.996 ms
84 bytes from 192.168.1.2 icmp_seq=9 ttl=62 time=66.820 ms
84 bytes from 192.168.1.2 icmp_seq=10 ttl=62 time=40.685 ms
84 bytes from 192.168.1.2 icmp_seq=11 ttl=62 time=41.516 ms
```
> Как видим перестроение трафика прошло успешно, потеряли только один пакет
