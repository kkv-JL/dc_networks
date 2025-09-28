### Цели:

- I:  Настроить BGP peering между Leaf и Spine в AF l2vpn evpn.
- II:  Настроить связанность между клиентами в первой зоне и убедитесь в её наличии.

<img width="1192" height="823" alt="image" src="https://github.com/user-attachments/assets/36f05d71-ed18-44bd-a9a8-2bc81da63f1a" />

`Исходные данные:` </br>
Underlay-сеть:  OSPF </br>
Overlay-сеть:  iBGP </br>
Vlans для конечных хостов: vl_10 и vl_20

#### 1. Настройка Коммутаторов </br>

##### 1.1 Настройка SPINE_1 и SPINE_2 идентична

```
router bgp 65001
   neighbor 10.99.100.3 remote-as 65001
   neighbor 10.99.100.3 update-source Loopback0
   neighbor 10.99.100.3 route-reflector-client
   neighbor 10.99.100.3 send-community extended
   neighbor 10.99.100.4 remote-as 65001
   neighbor 10.99.100.4 update-source Loopback0
   neighbor 10.99.100.4 route-reflector-client
   neighbor 10.99.100.4 send-community extended
   neighbor 10.99.100.5 remote-as 65001
   neighbor 10.99.100.5 update-source Loopback0
   neighbor 10.99.100.5 route-reflector-client
   neighbor 10.99.100.5 send-community extended
   !
   address-family evpn
      neighbor 10.99.100.3 activate
      neighbor 10.99.100.4 activate
      neighbor 10.99.100.5 activate
```
> В данной конфигурации настройка BGP соседства с LEAF_1, LEAF_2 и LEAF_3, важно указывать параметр `send-community extended` для того, чтобы наши route target передавались. </br>
> В `address-family evpn` активируем наших соседей

##### 1.2 Настройка LEAF_1

> Настраиваем наш Vlan и порт, куда подключен конечный хост
```
vlan 10
   name vl10
!
interface Ethernet1
   switchport access vlan 10
```

> Включаем/настраиваем интерфейс Vxlan1
```
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 1010
```
`задаем source-interface - Lo0` </br>
`настраиваем mapping что vlan 10 это vni 1010`

> Настравиваем BGP
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
   address-family evpn
      neighbor 10.99.100.1 activate
      neighbor 10.99.100.2 activate
```
`Так же не забываем о настройке send-community extended`
> `vlan 10` выполняем mapping - назначаем ему route-target `65001:1010` </br>
> Указываем параметр `redistribute learned` - для того, чтобы mac-адреса, которые будут появляться в vlan_10, которые изучили, рассылались по evpn-bgp


##### 1.3 Настройка LEAF_2

> Настройка симетрична; только для Vlan_20
```
vlan 20
   name vl20
!
interface Ethernet1
   switchport access vlan 20
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 20 vni 1020
!
router bgp 65001
   neighbor 10.99.100.1 remote-as 65001
   neighbor 10.99.100.1 update-source Loopback0
   neighbor 10.99.100.1 send-community extended
   neighbor 10.99.100.2 remote-as 65001
   neighbor 10.99.100.2 update-source Loopback0
   neighbor 10.99.100.2 send-community extended
   !
   vlan 20
      rd auto
      route-target both 65001:1020
      redistribute learned
   !
   address-family evpn
      neighbor 10.99.100.1 activate
      neighbor 10.99.100.2 activate
```


##### 1.4 Настройка LEAF_3

> тут для двух Vlan и двух конечных хостов
```
vlan 10
   name vl10
!
vlan 20
   name vl20
!
management cvx
   no shutdown
!
interface Ethernet1
   switchport access vlan 10
!
interface Ethernet2
   switchport access vlan 20
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 1010
   vxlan vlan 20 vni 1020
!
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
   !
   address-family evpn
      neighbor 10.99.100.1 activate
      neighbor 10.99.100.2 activate
```

#### 2. Проверка связности </br>

Проверим связность между VPC_host_1 и VPC_host_2
```
VPCS> ping 192.168.0.3

84 bytes from 192.168.0.3 icmp_seq=1 ttl=64 time=42.088 ms
84 bytes from 192.168.0.3 icmp_seq=2 ttl=64 time=59.933 ms
84 bytes from 192.168.0.3 icmp_seq=3 ttl=64 time=41.127 ms
84 bytes from 192.168.0.3 icmp_seq=4 ttl=64 time=58.860 ms
84 bytes from 192.168.0.3 icmp_seq=5 ttl=64 time=38.277 ms
```

> связность есть и можем наблюдать следующее </br>

<img width="911" height="154" alt="image" src="https://github.com/user-attachments/assets/4056ddb9-0bdc-4b16-a242-8ec18b07281e" />

`ARP запрос`

<img width="709" height="407" alt="image" src="https://github.com/user-attachments/assets/b27ab667-5c3c-40e6-a7d3-a26cdb120456" />

`ICMP`

<img width="760" height="630" alt="image" src="https://github.com/user-attachments/assets/0e801c96-79cc-46a6-9174-bf91821a1055" />

Можно увидеть, и VNI (1010), и адреса наших Lo evpn src-10.99.100.3(LEAF_1), dst-10.99.100.5(LEAF_3)

#### 3. Таблицы маршрутов BGP EVPN </br>

##### SPINE_1
```
SPINE-SW1#show bgp evpn summary 
BGP summary information for VRF default
Router identifier 10.99.100.1, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor    V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.99.100.3 4 65001            209       216    0    0 02:47:51 Estab   1      1
  10.99.100.4 4 65001            207       214    0    0 02:47:52 Estab   1      1
  10.99.100.5 4 65001            209       212    0    0 02:47:51 Estab   2      2
```

##### LEAF_1
```
LEAF-SW1#show bgp evpn summary 
BGP summary information for VRF default
Router identifier 10.99.100.3, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor    V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.99.100.1 4 65001            218       211    0    0 02:49:44 Estab   3      3
  10.99.100.2 4 65001            217       211    0    0 02:50:33 Estab   3      3
```

##### таблица MAC адресов LEAF_1
```
LEAF-SW1#show mac address-table 
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    0050.7966.6806    DYNAMIC     Et1        1       0:01:03 ago
  10    0050.7966.6808    DYNAMIC     Vx1        1       0:01:03 ago
Total Mac Addresses for this criterion: 2

          Multicast Mac Address Table
------------------------------------------------------------------
```
