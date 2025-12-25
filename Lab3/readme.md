
# Лабораторная работа №3

## Underlay. IS-IS

### Цель:
Настроить IS_IS для Underlay сети.



### Выполнение

Для выполнения данной работы используем топологию сети и IPv4 адресацию, разработанную в [лабораторной работе №1](https://github.com/i-gershuni/OTUS-DC-NET-Design-Labs/tree/82b7ce8b1be000731163ed32d370006d2b370917/Lab1).
Кроме того, чтобы немного развлечься, предположим, что у нас в планах предстоит мигрирация на IPv6, поэтому кроме IPv4 Underlay мы хотим развернуть так же IPv6 Underlay сеть в режиме Dual-Stack.


Схема сети, используемая в данной работе, представлена на рисунке ниже.

![Схема сети, используемая в данной лабораторной работе](./img/topology_lab3.png)


### IPv4 адресация, используемаяв данной работе


IPv4 адресация для устройств на стенде приведена в таблицах ниже.

#### Подсети, выделенные для P2P интерфейсов:

| P2P |	L1 | L2 | L3 |
|---|----|---|---|
| **S1** | 10.22.32.0/31 | 10.22.32.2/31 | 10.22.32.4/31 |
| **S2** | 10.22.32.64/31 | 10.22.32.66/31 | 10.22.32.68/31 |

#### Адреса Loopback интерфейсов:

|  Spine |	S1 | S2 |
|-------------|---------------|---------------|
| loopback | 10.22.36.1/32 | 10.22.36.2/32 |

|  Leaf |	L1 | L2 | L3 |
|-------------|---------------|---------------|------------|
| loopback |	10.22.37.1/32 | 10.22.37.2/32 | 10.22.37.3/32 |

#### Адреса интерфейсов в сторону клиентских подсетей:

| If\Sw | L1 | L2 | L3 |
|---|--|--|--|
| **Ethernet 7** | | | 172.22.3.1/24 |
| **Ethernet 8** | 172.22.1.1/24 | 172.22.2.1/24 | 172.22.4.1/24 |

#### Настройки IP на клиентских устройствах:

| Client | IP Addr | Def GW |
|---|---|---|
| **C1** | 172.22.1.11/24 | 172.22.1.1 |
| **C2** | 172.22.2.22/24 | 172.22.2.1 |
| **C3** | 172.22.3.33/24 | 172.22.3.1 |
| **C4** | 172.22.4.44/24 | 172.22.4.1 |


Для IPv6 адресного плана, чтобы долго не думать, возмем за основу адресацию IPv4, просто переложив адреса IPv4 в последние 4 октета адреса IPv6  в диапазоне ULA (Unique Local Address) (fc00::/7) 

### IPv6 адресация, используемая в данной работе

#### Подсети, выделенные для P2P интерфейсов:

| P2P |	L1 | L2 | L3 |
|---|----|---|---|
| **S1** | fc00::a16:2000/127 | fc00::a16:2002/127 | fc00::a16:2004/127 |
| **S2** | fc00::a16:2040/127 | fc00::a16:2042/127 | fc00::a16:2044/127 |

#### Адреса Loopback интерфейсов:

|  Spine |	S1 | S2 |
|-------------|---------------|---------------|
| loopback | fc00::a16:2401/128 | fc00::a16:2402/128 |

|  Leaf |	L1 | L2 | L3 |
|-------------|---------------|---------------|------------|
| loopback |	fc00::a16:2501/128 | fc00::a16:2502/128 | fc00::a16:2503/128 |

#### Адреса интерфейсов в сторону клиентских подсетей:

| If\Sw | L1 | L2 | L3 |
|---|--|--|--|
| **Ethernet 7** | | | fc00::ac16:301/120 |
| **Ethernet 8** | fc00::ac16:101/120 | fc00::ac16:101/120 | fc00::ac16:401/120 |

#### Настройки IP на клиентских устройствах:

| Client | IP Addr | Def GW |
|---|---|---|
| **C1** | fc00::ac16:111/120 | fc00::ac16:101 |
| **C2** | fc00::ac16:222/120 | fc00::ac16:201 |
| **C3** | fc00::ac16:333/120 | fc00::ac16:301 |
| **C4** | fc00::ac16:444/120 | fc00::ac16:401 |



### Выполняем настройки на коммутаторах:

#### Настраиваем IS-IS маршрутизатор

- на всех коммутаторах настраиваем экземпляр IS-IS маршрутзатора с именем **Underlay**
- NET (Network Entity Title) формируем по принципу 49.{код_ЦОД}.{номер_pod}.{1 для spine, 2 для leaf}.{Номер_коммутатора}.00  
Пусть код нашего ЦОД будет 22, такм образом NET на наших коммутаторах будут следующие:

| SW | NET |
|---|---|
| S1 | 49.0022.0001.0001.0001.00 |
| S2 | 49.0022.0001.0001.0002.00 |
| L1 | 49.0022.0001.0002.0001.00 |
| L2 | 49.0022.0001.0002.0002.00 |
| L3 | 49.0022.0001.0002.0003.00 |

- Поскольку у нас планируется только один pod, то тип маршрутизатора и все линки делаем L1;
- Для поддержки IPv4 добавляем address-family ipv4;
- Для поддержки IPv6 добавляем address-family ipv6.

#### Настраиваем интерфейсы

- включаем на коммутаторе маршрутизацию IPv6;
- на всех интерфейсах настраивает IPv6 адрес в дополнение к уже существующему IPv4;
- включаем на интерфейсе протокол IS-IS для instance Underlay;
- настраиваем тип интерфейса L1 и тип сети point-to-point;
- включаем BFD и настраиваем рекомендованные значения таймеров;
- настраиваем аутентификацию;
- Loopback и интерфейсы в сторону клиентов настраиваем как passive.

### Итоговые настройки коммутаторов:

#### Настройки коммутатора S1:
```
hostname S1
!
interface Ethernet1
   description Leaf1_Et1
   no switchport
   ip address 10.22.32.0/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 address fc00::a16:2000/127
   isis enable Underlay
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication mode sha key-id 1
   isis authentication key-id 1 algorithm sha-256 key 7 NSFjnA0bJybM4espbUU41w==
!
interface Ethernet2
   description Leaf2_Et1
   no switchport
   ip address 10.22.32.2/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 address fc00::a16:2002/127
   isis enable Underlay
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication mode sha key-id 1
   isis authentication key-id 1 algorithm sha-256 key 7 NSFjnA0bJybM4espbUU41w==
!
interface Ethernet3
   description Leaf3_Et1
   no switchport
   ip address 10.22.32.4/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 address fc00::a16:2004/127
   isis enable Underlay
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication mode sha key-id 1
   isis authentication key-id 1 algorithm sha-256 key 7 NSFjnA0bJybM4espbUU41w==
!
interface Loopback1
   ip address 10.22.36.1/32
   ipv6 address fc00::a16:2401/128
   isis enable Underlay
   isis passive
!
ip routing
no ip icmp redirect
!
ipv6 unicast-routing
!
router isis Underlay
   net 49.0022.0001.0001.0001.00
   is-type level-1
   log-adjacency-changes
   !
   address-family ipv4 unicast
   !
   address-family ipv6 unicast
!
end
```

#### Настройки коммутатора S2:
```
hostname S2
!
interface Ethernet1
   description Leaf1_Et2
   no switchport
   ip address 10.22.32.64/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 address fc00::a16:2040/127
   isis enable Underlay
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication mode sha key-id 1
   isis authentication key-id 1 algorithm sha-256 key 7 NSFjnA0bJybM4espbUU41w==
!
interface Ethernet2
   description Leaf2_Et2
   no switchport
   ip address 10.22.32.66/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 address fc00::a16:2042/127
   isis enable Underlay
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication mode sha key-id 1
   isis authentication key-id 1 algorithm sha-256 key 7 NSFjnA0bJybM4espbUU41w==
!
interface Ethernet3
   description Leaf3_Et2
   no switchport
   ip address 10.22.32.68/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 address fc00::a16:2044/127
   isis enable Underlay
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication mode sha key-id 1
   isis authentication key-id 1 algorithm sha-256 key 7 NSFjnA0bJybM4espbUU41w==
!
interface Loopback1
   ip address 10.22.36.2/32
   ipv6 address fc00::a16:2402/128
   isis enable Underlay
   isis passive
!
ip routing
no ip icmp redirect
!
ipv6 unicast-routing
!
router isis Underlay
   net 49.0022.0001.0001.0002.00
   is-type level-1
   log-adjacency-changes
   !
   address-family ipv4 unicast
   !
   address-family ipv6 unicast
!
end
```

#### Настройки коммутатора L1:
```
hostname L1
!
interface Ethernet1
   description Spoke1_Et1
   no switchport
   ip address 10.22.32.1/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 address fc00::a16:2001/127
   isis enable Underlay
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication mode sha key-id 1
   isis authentication key-id 1 algorithm sha-256 key 7 NSFjnA0bJybM4espbUU41w==
!
interface Ethernet2
   description Spoke2_Et1
   no switchport
   ip address 10.22.32.65/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 address fc00::a16:2040/127
   isis enable Underlay
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication mode sha key-id 1
   isis authentication key-id 1 algorithm sha-256 key 7 NSFjnA0bJybM4espbUU41w==
!
interface Ethernet8
   description ClientSubNet1
   no switchport
   ip address 172.22.1.1/24
   ipv6 address fc00::ac16:101/120
   isis enable Underlay
   isis passive
!
interface Loopback1
   ip address 10.22.37.1/32
   ipv6 address fc00::a16:2501/128
   isis enable Underlay
   isis passive
!
ip routing
no ip icmp redirect
!
ipv6 unicast-routing
!
router isis Underlay
   net 49.0022.0001.0002.0001.00
   is-type level-1
   log-adjacency-changes
   !
   address-family ipv4 unicast
   !
   address-family ipv6 unicast
!
end
```

#### Настройки коммутатора L2:
```
hostname L2
!
interface Ethernet1
   description Spine1_Et2
   no switchport
   ip address 10.22.32.3/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 address fc00::a16:2003/127
   isis enable Underlay
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication mode sha key-id 1
   isis authentication key-id 1 algorithm sha-256 key 7 NSFjnA0bJybM4espbUU41w==
!
interface Ethernet2
   description Spine2_Et2
   no switchport
   ip address 10.22.32.67/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 address fc00::a16:2043/127
   isis enable Underlay
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication mode sha key-id 1
   isis authentication key-id 1 algorithm sha-256 key 7 NSFjnA0bJybM4espbUU41w==
!
interface Ethernet8
   description ClientSubNet2
   no switchport
   ip address 172.22.2.1/24
   ipv6 address fc00::ac16:201/120
   isis enable Underlay
   isis passive
!
interface Loopback1
   ip address 10.22.37.2/32
   ipv6 address fc00::a16:2502/128
   isis enable Underlay
   isis passive
!
ip routing
no ip icmp redirect
!
ipv6 unicast-routing
!
router isis Underlay
   net 49.0022.0001.0002.0002.00
   is-type level-1
   log-adjacency-changes
   !
   address-family ipv4 unicast
   !
   address-family ipv6 unicast
!
end
```

#### Настройки коммутатора L3:
```
hostname L3
!
interface Ethernet1
   description Spine1_Et3
   no switchport
   ip address 10.22.32.5/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 address fc00::a16:2005/127
   isis enable Underlay
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication mode sha key-id 1
   isis authentication key-id 1 algorithm sha-256 key 7 NSFjnA0bJybM4espbUU41w==
!
interface Ethernet2
   description Spine2_Et3
   no switchport
   ip address 10.22.32.69/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 address fc00::a16:2045/127
   isis enable Underlay
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication mode sha key-id 1
   isis authentication key-id 1 algorithm sha-256 key 7 NSFjnA0bJybM4espbUU41w==
!
interface Ethernet7
   description ClientSubNet3
   no switchport
   ip address 172.22.3.1/24
   ipv6 address fc00::ac16:301/120
   isis enable Underlay
   isis passive
!
interface Ethernet8
   description ClientSubNet4
   no switchport
   ip address 172.22.4.1/24
   ipv6 address fc00::ac16:401/120
   isis enable Underlay
   isis passive
!
interface Loopback1
   ip address 10.22.37.3/32
   ipv6 address fc00::a16:2503/128
   isis enable Underlay
   isis passive
!
ip routing
no ip icmp redirect
!
ipv6 unicast-routing
!
router isis Underlay
   net 49.0022.0001.0002.0003.00
   is-type level-1
   log-adjacency-changes
   !
   address-family ipv4 unicast
   !
   address-family ipv6 unicast
!
end
```

