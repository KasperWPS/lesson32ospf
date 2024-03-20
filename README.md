# Домашнее задание № 21 по теме: "Статическая и динамическая маршрутизация, OSPF". К курсу Administrator Linux. Professional

## Задание

- Поднять три виртуалки
- Объединить их разными vlan
  - настроить OSPF между машинами на базе Quagga;
  - изобразить ассиметричный роутинг;
  - сделать один из линков "дорогим", но что бы при этом роутинг был симметричным.

## Выполнение

![Network topology](https://github.com/KasperWPS/lesson33ospf/blob/main/topology21.svg)

config.json (конфигурация стенда)
```json
[
  {
    "name": "router1",
    "cpus": 1,
    "gui": false,
    "box": "generic/debian11",
    "private_network":
    [
      { "ip": "10.0.10.1",     "adapter": 2, "netmask": "255.255.255.252", "virtualbox__intnet": "r1-r2" },
      { "ip": "10.0.12.1",     "adapter": 3, "netmask": "255.255.255.252", "virtualbox__intnet": "r1-r3" },
      { "ip": "192.168.10.1",  "adapter": 4, "netmask": "255.255.255.0",   "virtualbox__intnet": "net1"  },
      { "ip": "192.168.56.10", "adapter": 5, "netmask": "255.255.255.0" }
    ],
    "memory": "640",
    "no_share": true
  },
  {
    "name": "router2",
    "cpus": 1,
    "gui": false,
    "box": "generic/debian11",
    "private_network":
    [
      { "ip": "10.0.10.2",     "adapter": 2, "netmask": "255.255.255.252", "virtualbox__intnet": "r1-r2" },
      { "ip": "10.0.11.2",     "adapter": 3, "netmask": "255.255.255.252", "virtualbox__intnet": "r2-r3" },
      { "ip": "192.168.20.1",  "adapter": 4, "netmask": "255.255.255.0",   "virtualbox__intnet": "net2"  },
      { "ip": "192.168.56.11", "adapter": 5, "netmask": "255.255.255.0" }
    ],
    "memory": 640,
    "no_share": true
  },
  {
    "name": "router3",
    "cpus": 1,
    "gui": false,
    "box": "generic/debian11",
    "private_network":
    [
      { "ip": "10.0.11.1",     "adapter": 2, "netmask": "255.255.255.252", "virtualbox__intnet": "r2-r3" },
      { "ip": "10.0.12.2",     "adapter": 3, "netmask": "255.255.255.252", "virtualbox__intnet": "r1-r3" },
      { "ip": "192.168.30.1",  "adapter": 4, "netmask": "255.255.255.0",   "virtualbox__intnet": "net3"  },
      { "ip": "192.168.56.12", "adapter": 5, "netmask": "255.255.255.0" }
    ],
    "memory": "640",
    "no_share": true
  }
]
```

### Проверка стенда

```bash
vagrant ssh router1 -c 'ip a | grep inet '
```

```
    inet 127.0.0.1/8 scope host lo
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
    inet 10.0.10.1/30 brd 10.0.10.3 scope global eth1
    inet 10.0.12.1/30 brd 10.0.12.3 scope global eth2
    inet 192.168.10.1/24 brd 192.168.10.255 scope global eth3
    inet 192.168.56.10/24 brd 192.168.56.255 scope global eth4
```

```bash
vagrant ssh router1 -c 'ip r'
```
```
default via 10.0.2.2 dev eth0 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15
10.0.10.0/30 dev eth1 proto kernel scope link src 10.0.10.1
10.0.11.0/30 nhid 36 proto ospf metric 20
        nexthop via 10.0.10.2 dev eth1 weight 1
        nexthop via 10.0.12.2 dev eth2 weight 1
10.0.12.0/30 dev eth2 proto kernel scope link src 10.0.12.1
192.168.10.0/24 dev eth3 proto kernel scope link src 192.168.10.1
192.168.20.0/24 nhid 32 via 10.0.10.2 dev eth1 proto ospf metric 20
192.168.30.0/24 nhid 37 via 10.0.12.2 dev eth2 proto ospf metric 20
192.168.56.0/24 dev eth4 proto kernel scope link src 192.168.56.10
```

```bash
vagrant ssh router2 -c 'ip a | grep inet '
```
```
    inet 127.0.0.1/8 scope host lo
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
    inet 10.0.10.2/30 brd 10.0.10.3 scope global eth1
    inet 10.0.11.2/30 brd 10.0.11.3 scope global eth2
    inet 192.168.20.1/24 brd 192.168.20.255 scope global eth3
    inet 192.168.56.11/24 brd 192.168.56.255 scope global eth4
```

```bash
vagrant ssh router2 -c 'ip r'
```
```
default via 10.0.2.2 dev eth0 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15
10.0.10.0/30 dev eth1 proto kernel scope link src 10.0.10.2
10.0.11.0/30 dev eth2 proto kernel scope link src 10.0.11.2
10.0.12.0/30 nhid 34 proto ospf metric 20
        nexthop via 10.0.10.1 dev eth1 weight 1
        nexthop via 10.0.11.1 dev eth2 weight 1
192.168.10.0/24 nhid 35 via 10.0.10.1 dev eth1 proto ospf metric 20
192.168.20.0/24 dev eth3 proto kernel scope link src 192.168.20.1
192.168.30.0/24 nhid 36 via 10.0.11.1 dev eth2 proto ospf metric 20
192.168.56.0/24 dev eth4 proto kernel scope link src 192.168.56.11
```

```bash
vagrant ssh router3 -c 'ip a | grep inet '
```
```
    inet 127.0.0.1/8 scope host lo
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
    inet 10.0.11.1/30 brd 10.0.11.3 scope global eth1
    inet 10.0.12.2/30 brd 10.0.12.3 scope global eth2
    inet 192.168.30.1/24 brd 192.168.30.255 scope global eth3
    inet 192.168.56.12/24 brd 192.168.56.255 scope global eth4
```

```bash
vagrant ssh router3 -c 'ip r'
```
```
default via 10.0.2.2 dev eth0 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15
10.0.10.0/30 nhid 34 proto ospf metric 20
        nexthop via 10.0.11.2 dev eth1 weight 1
        nexthop via 10.0.12.1 dev eth2 weight 1
10.0.11.0/30 dev eth1 proto kernel scope link src 10.0.11.1
10.0.12.0/30 dev eth2 proto kernel scope link src 10.0.12.2
192.168.10.0/24 nhid 36 via 10.0.12.1 dev eth2 proto ospf metric 20
192.168.20.0/24 nhid 35 via 10.0.11.2 dev eth1 proto ospf metric 20
192.168.30.0/24 dev eth3 proto kernel scope link src 192.168.30.1
192.168.56.0/24 dev eth4 proto kernel scope link src 192.168.56.12
```

```bash
vagrant ssh router3 -c 'ping -c1 192.168.30.1'
```
```
PING 192.168.30.1 (192.168.30.1) 56(84) bytes of data.
64 bytes from 192.168.30.1: icmp_seq=1 ttl=64 time=0.025 ms
```

```bash
vagrant ssh router3 -c 'traceroute 192.168.30.1'
```
```
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets
 1  192.168.30.1 (192.168.30.1)  0.013 ms  0.003 ms  0.003 ms
```

- Отключить интерфейс на router1 и проверить перестроение маршрута

```bash
vagrant ssh router1 -c 'sudo ifconfig eth2 down'
```

```bash
vagrant ssh router1 -c 'traceroute 192.168.30.1'
```
```
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets
 1  10.0.10.2 (10.0.10.2)  0.509 ms  0.492 ms  0.469 ms
 2  192.168.30.1 (192.168.30.1)  81.026 ms  81.019 ms  81.010 ms
```

```bash
vagrant ssh router1 -c 'sudo ifconfig eth2 up'
```

```bash
vagrant ssh router1 -c 'traceroute 192.168.30.1'
```
```
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets
 1  192.168.30.1 (192.168.30.1)  0.282 ms  0.242 ms  0.229 ms
```

### Настройка ассиметричного роутинга

```bash
vagrant ssh router1 -c 'sudo vtysh'
```

```
Hello, this is FRRouting (version 9.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

router1# conf t
router1(config)# int eth1
router1(config-if)# ip
router1(config-if)# ip ospf cost 1000
router1(config-if)# exit
router1(config)# exit
router1# show ip route ospf 
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O   10.0.10.0/30 [110/300] via 10.0.12.2, eth2, weight 1, 00:00:32
O>* 10.0.11.0/30 [110/200] via 10.0.12.2, eth2, weight 1, 00:00:32
O   10.0.12.0/30 [110/100] is directly connected, eth2, weight 1, 00:12:59
O   192.168.10.0/24 [110/100] is directly connected, eth3, weight 1, 02:01:59
O>* 192.168.20.0/24 [110/300] via 10.0.12.2, eth2, weight 1, 00:00:32
O>* 192.168.30.0/24 [110/200] via 10.0.12.2, eth2, weight 1, 00:12:59
router1# exit
```

```bash
vagrant ssh router2 -c 'sudo vtysh'
```

```
Hello, this is FRRouting (version 9.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

router2# show ip route ospf
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O   10.0.10.0/30 [110/100] is directly connected, eth1, weight 1, 02:05:27
O   10.0.11.0/30 [110/100] is directly connected, eth2, weight 1, 02:05:27
O>* 10.0.12.0/30 [110/200] via 10.0.10.1, eth1, weight 1, 00:16:22
  *                        via 10.0.11.1, eth2, weight 1, 00:16:22
O>* 192.168.10.0/24 [110/200] via 10.0.10.1, eth1, weight 1, 02:04:42
O   192.168.20.0/24 [110/100] is directly connected, eth3, weight 1, 02:05:27
O>* 192.168.30.0/24 [110/200] via 10.0.11.1, eth2, weight 1, 02:04:42
router2# exit
```

```bash
vagrant ssh router1 -c 'ping -I 192.168.10.1 192.168.20.1'
```
- Интерфейс eth2 только получает пакеты icmp
```bash
vagrant ssh router2 -c 'sudo tcpdump -i eth2'
```
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth2, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21:20:12.915643 IP 192.168.10.1 > 192.168.20.1: ICMP echo request, id 3822, seq 59, length 64
21:20:13.917457 IP 192.168.10.1 > 192.168.20.1: ICMP echo request, id 3822, seq 60, length 64
```

```bash
vagrant ssh router2 -c 'sudo tcpdump -ni eth1'
```
- Интерфейс eth1 только отправляет icmp пакеты на адрес 192.168.10.1
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth1, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21:23:45.217018 IP 192.168.20.1 > 192.168.10.1: ICMP echo reply, id 6507, seq 1, length 64
21:23:46.227569 IP 192.168.20.1 > 192.168.10.1: ICMP echo reply, id 6507, seq 2, length 64
```

### Настройка симметричной маршрутизации

- Изменить стоимость маршрута от интерфейса eth1

```bash
vagrant ssh router2 -c 'sudo vtysh'
```
```
Hello, this is FRRouting (version 9.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

router2# conf t
router2(config)# int eth1
router2(config-if)# ip ospf cost 1000
router2(config-if)# exit
router2(config)# exit
router2# show ip route ospf
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O   10.0.10.0/30 [110/1000] is directly connected, eth1, weight 1, 00:00:24
O   10.0.11.0/30 [110/100] is directly connected, eth2, weight 1, 02:26:24
O>* 10.0.12.0/30 [110/200] via 10.0.11.1, eth2, weight 1, 00:00:24
O>* 192.168.10.0/24 [110/300] via 10.0.11.1, eth2, weight 1, 00:00:24
O   192.168.20.0/24 [110/100] is directly connected, eth3, weight 1, 02:26:24
O>* 192.168.30.0/24 [110/200] via 10.0.11.1, eth2, weight 1, 02:25:39
router2# exit
```

```bash
vagrant ssh router1 -c 'ping -I 192.168.10.1 192.168.20.1'
```

```bash
vagrant ssh router2 -c 'sudo tcpdump -i eth2'
```
- Трафик между маршрутизаторами ходит симметрично
```
21:34:45.875539 IP 192.168.10.1 > 192.168.20.1: ICMP echo request, id 8725, seq 29, length 64
21:34:45.875579 IP 192.168.20.1 > 192.168.10.1: ICMP echo reply, id 8725, seq 29, length 64
```







