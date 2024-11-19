# vpc & eigrp

### Топология
![топология](top.png "топология")

### Конфигурация
![Текущая топология](eve-ng_topology.png "Текущая топология")
Настройка vpc для N9k. Внизу коммутатор Ариста, т.к. память на компьютере не безграничная
влючаем фичи для настройки и начинаем делать vpc peer-link. Он нужен для проверки, что нексусы живые, друг друга видят

```
feature vpc   
feature lacp

interface mgmt0 
 ip address 10.0.0.1/30
 no shut 
```
Делаем vpc домен, указываем с каких адресов проверять доступность хостов. priority минимальный будет мастер, в нашем случае N9k1 мастер. Проверяем
```
vpc domain 1
  role priority 20
  peer-keepalive destination 10.0.0.2 source 10.0.0.1 vrf management

N9k1# ping 10.0.0.2 source 10.0.0.1 vrf management
PING 10.0.0.2 (10.0.0.2) from 10.0.0.1: 56 data bytes
64 bytes from 10.0.0.2: icmp_seq=0 ttl=254 time=3.506 ms
64 bytes from 10.0.0.2: icmp_seq=1 ttl=254 time=2.768 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=254 time=2.179 ms
64 bytes from 10.0.0.2: icmp_seq=3 ttl=254 time=3.218 ms
64 bytes from 10.0.0.2: icmp_seq=4 ttl=254 time=2.683 ms

--- 10.0.0.2 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 2.179/2.87/3.506 ms
```
Делаем сам vpc, что-то наподобие стека или vss линка для 6500
```
interface ethernet 1/1-2
 channel-group 4096 mode active
 no shutdown
 
interface port-channel 4096
 no shutdown 
 switchport
 switchport mode trunk 
 vpc peer-link
```

Создаем тестовый влан и проверяем
```
vlan 10 
 name test 
 
interface eth1/3
 channel-group 10 mode active

interface port-channel 10
 switchport 
 switchport mode trunk 
 switchport trunk allowed vlan 10
 vpc 4096


feature interface-vlan

interface Vlan10
  no shutdown
  ip address 192.168.10.252/24

N9k1# ping 192.168.10.1
PING 192.168.10.1 (192.168.10.1): 56 data bytes
64 bytes from 192.168.10.1: icmp_seq=0 ttl=63 time=21.459 ms
64 bytes from 192.168.10.1: icmp_seq=1 ttl=63 time=10.942 ms
64 bytes from 192.168.10.1: icmp_seq=2 ttl=63 time=11.115 ms
64 bytes from 192.168.10.1: icmp_seq=3 ttl=63 time=12.199 ms
64 bytes from 192.168.10.1: icmp_seq=4 ttl=63 time=13.965 ms
```
Для N9k2 аналогичный конфиг
```
feature vpc   
feature lacp 
interface mgmt0
 ip address 10.0.0.2/30
 no shut 
 
vpc domain 1
  role priority 30
  peer-keepalive destination 10.0.0.1 source 10.0.0.2 vrf management
  
  
 interface ethernet 1/1-2
 channel-group 4096 mode active
 no shutdown
 
interface port-channel 4096
 no shutdown 
 switchport
 switchport mode trunk 
 vpc peer-link

vlan 10 
 name test 
 
interface eth1/3
 channel-group 10 mode active

interface port-channel 10
 switchport 
 switchport mode trunk 
 switchport trunk allowed vlan 10
 vpc 4096  
```
