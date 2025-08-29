Цели:

I:    Собрать топологию CLOS

II:   Распределить адресное пространство для Underlay сети

III:  Зафиксировать в документации план работ, адресное пространство, схему сети, настройки

<img width="1221" height="760" alt="Screenshot_13" src="https://github.com/user-attachments/assets/86b73445-bfe7-450a-aebd-a55928394dc2" />

1 Адресное пространство (Underlay)

1.1 Loopback’и (/32)

| Устройство | Loopback |
|-----------:|:---------|
| SPINE_1    | 10.99.100.1/32 |
| SPINE_2    | 10.99.100.2/32 |
| LEAF_1     | 10.99.100.3/32 |
| LEAF_2     | 10.99.100.4/32 |
| LEAF_3     | 10.99.100.5/32 |


1.2 P2P /31 (Spine↔Leaf)

| Линк | Spine адрес (/31) | Leaf адрес (/31) |
|:----:|:------------------|:-----------------|
| S1—L1 | 10.99.101.0/31 | 10.99.101.1/31 |
| S2—L1 | 10.99.101.2/31 | 10.99.101.3/31 |
| S1—L2 | 10.99.101.4/31 | 10.99.101.5/31 |
| S2—L2 | 10.99.101.6/31 | 10.99.101.7/31 |
| S1—L3 | 10.99.101.8/31 | 10.99.101.9/31 |
| S2—L3 | 10.99.101.10/31 | 10.99.101.11/31 |


2 Настройки

2.1 SPINEs

**SPINE-SW1**

```
interface Ethernet6
   no switchport
   ip address 10.99.101.8/31
!
interface Ethernet7
   no switchport
   ip address 10.99.101.4/31
!
interface Ethernet8
   no switchport
   ip address 10.99.101.0/31
!
interface Loopback0
   ip address 10.99.100.1/32
!
```

**SPINE-SW2**

```
interface Ethernet6
   no switchport
   ip address 10.99.101.10/31
!
interface Ethernet7
   no switchport
   ip address 10.99.101.6/31
!
interface Ethernet8
   no switchport
   ip address 10.99.101.2/31
!
interface Loopback0
   ip address 10.99.100.2/32
!
```

2.2 LEAFs

**LEAF-SW1**

```
interface Ethernet7
   no switchport
   ip address 10.99.101.3/31
!
interface Ethernet8
   no switchport
   ip address 10.99.101.1/31
!
interface Loopback0
   ip address 10.99.100.3/32
!
```

**LEAF-SW2**

```
interface Ethernet7
   no switchport
   ip address 10.99.101.7/31
!
interface Ethernet8
   no switchport
   ip address 10.99.101.5/31
!
interface Loopback0
   ip address 10.99.100.4/32
!
```

**LEAF-SW3**

```
interface Ethernet7
   no switchport
   ip address 10.99.101.11/31
!
interface Ethernet8
   no switchport
   ip address 10.99.101.9/31
!
interface Loopback0
   ip address 10.99.100.5/32
!
```
