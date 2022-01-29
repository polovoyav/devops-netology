# 03-sysadmin-07-net2  
  
## 1)    
#### Linux:  
>vagrant@sandbox:\~$ ip -c -br link  
>lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>  
>eth0             UP             08:00:27:73:60:cf <BROADCAST,MULTICAST,UP,LOWER_UP>  
  
>vagrant@sandbox:\~$ ip -c -br addr  
>lo               UNKNOWN        127.0.0.1/8 ::1/128  
>eth0             UP             10.0.2.15/24 fe80::a00:27ff:fe73:60cf/64  
  
#### Windows:  
>C:\Users\andrew>netsh interface show interface  
>  
>Состояние адм.  Состояние     Тип              Имя интерфейса  
>\---------------------------------------------------------------------  
>Разрешен       Отключен       Выделенный       Ethernet 4  
>Разрешен       Подключен      Выделенный       VirtualBox Host-Only Network  
>Разрешен       Подключен      Выделенный       Беспроводная сеть 2  
  
>C:\Users\andrew>netsh interface ip show address | findstr "IP"  
>    IP-адрес                           192.168.56.1  
>    IP-адрес                           192.168.1.2  
>    IP-адрес                           127.0.0.1  
  
## 2)  
#### Протокол LLDP, пакет lldpd, команды lldpctl и lldpcli.  
  
>vagrant@sandbox:\~$ lldpctl -v  
>1.0.4  
>  
>vagrant@sandbox:\~$ lldpcli show interfaces  
>\-------------------------------------------------------------------------------  
>LLDP interfaces:  
>\-------------------------------------------------------------------------------  
>Interface:    eth0, via: unknown, Time: 0 day, 00:10:22  
>  Chassis:  
>    ChassisID:    mac 08:00:27:73:60:cf  
>    SysName:      sandbox  
>    SysDescr:     Ubuntu 20.04.2 LTS Linux 5.4.0-80-generic #90-Ubuntu SMP Fri Jul 9 22:49:44 UTC 2021 x86_64  
>    MgmtIP:       10.0.2.15  
>    MgmtIP:       fe80::a00:27ff:fe73:60cf  
>    Capability:   Bridge, off  
>    Capability:   Router, off  
>    Capability:   Wlan, off  
>    Capability:   Station, on  
>  Port:  
>    PortID:       mac 08:00:27:73:60:cf  
>    PortDescr:    eth0  
>  TTL:          120  
>\-------------------------------------------------------------------------------  
  
#### Соседей нет:  
  
>vagrant@sandbox:~$ lldpcli show neigh  
>\-------------------------------------------------------------------------------  
>LLDP neighbors:  
>\-------------------------------------------------------------------------------  
  
## 3)  
#### Технология VLAN, пакет vlan.  
  
>vagrant@sandbox:\~$ sudo apt install vlan  
>vagrant@sandbox:\~$ sudo modprobe 8021q  
>  
>vagrant@sandbox:\~$ sudo nano /etc/network/interfaces  
>>\# interfaces(5) file used by ifup(8) and ifdown(8)  
>>\# Include files from /etc/network/interfaces.d:  
>>source-directory /etc/network/interfaces.d  
>>  
>>auto eth0.5  
>>iface eth0.5 inet static  
>>address 192.168.5.1  
>>netmask 255.255.255.0  
  
>vagrant@sandbox:\~$ sudo ifup eth0.5  
>vagrant@sandbox:\~$ ip -c -br link  
>lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>  
>eth0             UP             08:00:27:73:60:cf <BROADCAST,MULTICAST,UP,LOWER_UP>  
>eth0.5@eth0      UP             08:00:27:73:60:cf <BROADCAST,MULTICAST,UP,LOWER_UP>  
>  
>vagrant@sandbox:~$ sudo cat /proc/net/vlan/config  
>VLAN Dev name    | VLAN ID  
>Name-Type: VLAN_NAME_TYPE_RAW_PLUS_VID_NO_PAD  
>eth0.5         | 5  | eth0  
  
## 4)  
#### Типы агрегации интерфейсов (bonding) в Linux:  
  
Статический (настройка вручную) и динамический (настройка с помощью LACP - Link Aggregation Control Protocol).  
  
#### Опции балансировки нагрузки:  
  
* mode=0 (balance-rr). Используется по умолчанию, балансировка нагрузки и отказоустойчивость. Сетевые пакеты отправляются по интерфейсам последовательно, от первого доступного к последнему и сначала.  
* mode=1 (active-backup). Один интерфейс в связке активен, остальные в ожидающем. Если активный падает, один из ожидающих становится активным.  
* mode=2 (balance-xor). Передача пакетов распределяется между интерфейсами по формуле ((MAC src) XOR (MAC dest)) % N интерфейсов. балансировка нагрузки и отказоустойчивость.  
* mode=3 (broadcast). Одновременная передача во все объединенные интерфейсы, обеспечивает отказоустойчивость.  
* mode=4 (802.3ad). Динамическое объединение портов. Увеличение пропускной способности входящего/исходящего трафика. Необходима поддержка и настройка коммутаторов.  
* mode=5 (balance-tlb). Адаптивная балансировка нагрузки. Активный интерфейс принимает входящий трафик, исходящий трафик распределяется на каждый интерфей в зависимости от  загрузки.  
* mode=6 (balance-alb). Адаптивная балансировка нагрузки. Исходящий и входящий трафик распределяются между интерфейсами, но интерфейсы должны уметь изменять MAC.  
  
#### Пример конфига:  
  
>vagrant@sandbox:\~$ sudo apt-get install ifenslave  
>vagrant@sandbox:\~$ sudo modprobe bonding  
>  
>vagrant@sandbox:\~$ sudo nano /etc/network/interfaces  
>>\# interfaces(5) file used by ifup(8) and ifdown(8)  
>>\# Include files from /etc/network/interfaces.d:  
>>source-directory /etc/network/interfaces.d  
>>  
>>auto eth0.5  
>>iface eth0.5 inet static  
>>address 192.168.5.1  
>>netmask 255.255.255.0  
>>  
>>auto bond0  
>>iface bond0 inet static  
>>    address 192.168.5.10  
>>    netmask 255.255.255.0  
>>    network 192.168.5.0  
>>    gateway 192.168.5.254  
>>        slaves eth0 eth0.5  
>>        bond_mode 0  
>>        bond-miimon 100  
>>        bond_downdelay 200  
>>        bound_updelay 200  
  
>vagrant@sandbox:\~$ sudo systemctl restart networking.service  
>vagrant@sandbox:\~$ ip -c -br addr  
>lo               UNKNOWN        127.0.0.1/8 ::1/128  
>eth0             UP             10.0.2.15/24 fe80::a00:27ff:fe73:60cf/64  
>eth0.5@eth0      UP             192.168.5.1/24 fe80::a00:27ff:fe73:60cf/64  
>bond0            DOWN           192.168.5.10/24  
  
>vagrant@sandbox:\~$ cat /proc/net/bonding/bond0  
>Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)  
>Bonding Mode: load balancing (round-robin)  
>MII Status: down  
>MII Polling Interval (ms): 100  
>Up Delay (ms): 0  
>Down Delay (ms): 200  
>Peer Notification Delay (ms): 0  
  
## 5)  
#### В /29 сети 8 IP адресов, 6 хостов.  
  
>vagrant@sandbox:\~$ ipcalc 10.0.0.0/29  
>Address:   10.0.0.0             00001010.00000000.00000000.00000 000  
>Netmask:   255.255.255.248 = 29 11111111.11111111.11111111.11111 000  
>Wildcard:  0.0.0.7              00000000.00000000.00000000.00000 111  
>=>  
>Network:   10.0.0.0/29          00001010.00000000.00000000.00000 000  
>HostMin:   10.0.0.1             00001010.00000000.00000000.00000 001  
>HostMax:   10.0.0.6             00001010.00000000.00000000.00000 110  
>Broadcast: 10.0.0.7             00001010.00000000.00000000.00000 111  
>Hosts/Net: 6                     Class A, Private Internet  
  
#### Можно получить 32 подсети, примеры для 10.10.10.0/24.  
  
>vagrant@sandbox:\~$ ipcalc 10.10.10.0/255.255.255.0 255.255.255.248  
>Address:   10.10.10.0           00001010.00001010.00001010. 00000000  
>Netmask:   255.255.255.0 = 24   11111111.11111111.11111111. 00000000  
>Wildcard:  0.0.0.255            00000000.00000000.00000000. 11111111  
>=>  
>Network:   10.10.10.0/24        00001010.00001010.00001010. 00000000  
>HostMin:   10.10.10.1           00001010.00001010.00001010. 00000001  
>HostMax:   10.10.10.254         00001010.00001010.00001010. 11111110  
>Broadcast: 10.10.10.255         00001010.00001010.00001010. 11111111  
>Hosts/Net: 254                   Class A, Private Internet  
  
...  
  
>Subnets after transition from /24 to /29  
>  
> 1.  
>Network:   10.10.10.0/29        00001010.00001010.00001010.00000 000  
>HostMin:   10.10.10.1           00001010.00001010.00001010.00000 001  
>HostMax:   10.10.10.6           00001010.00001010.00001010.00000 110  
>Broadcast: 10.10.10.7           00001010.00001010.00001010.00000 111  
>Hosts/Net: 6                     Class A, Private Internet  
>  
> 2.  
>Network:   10.10.10.8/29        00001010.00001010.00001010.00001 000  
>HostMin:   10.10.10.9           00001010.00001010.00001010.00001 001  
>HostMax:   10.10.10.14          00001010.00001010.00001010.00001 110  
>Broadcast: 10.10.10.15          00001010.00001010.00001010.00001 111  
>Hosts/Net: 6                     Class A, Private Internet  
>  
> 3.  
>Network:   10.10.10.16/29       00001010.00001010.00001010.00010 000  
>HostMin:   10.10.10.17          00001010.00001010.00001010.00010 001  
>HostMax:   10.10.10.22          00001010.00001010.00001010.00010 110  
>Broadcast: 10.10.10.23          00001010.00001010.00001010.00010 111  
>Hosts/Net: 6                     Class A, Private Internet  
  
...  
  
>Subnets:   32  
>Hosts:     192  
  
## 6)  
#### Можно взять Carrier-Grade NAT 100.64.0.0/10. Для 40-50 хостов внутри подсети нужна маска 255.255.255.192.  
  
>vagrant@sandbox:\~$ ipcalc 100.64.0.0/255.192.0.0 --split 40 50  
>Address:   100.64.0.0           01100100.01 000000.00000000.00000000  
>Netmask:   255.192.0.0 = 10     11111111.11 000000.00000000.00000000  
>Wildcard:  0.63.255.255         00000000.00 111111.11111111.11111111  
>=>  
>Network:   100.64.0.0/10        01100100.01 000000.00000000.00000000  
>HostMin:   100.64.0.1           01100100.01 000000.00000000.00000001  
>HostMax:   100.127.255.254      01100100.01 111111.11111111.11111110  
>Broadcast: 100.127.255.255      01100100.01 111111.11111111.11111111  
>Hosts/Net: 4194302               Class A  
>  
>1. Requested size: 40 hosts  
>Netmask:   255.255.255.192 = 26 11111111.11111111.11111111.11 000000  
>Network:   100.64.0.0/26        01100100.01000000.00000000.00 000000  
>HostMin:   100.64.0.1           01100100.01000000.00000000.00 000001  
>HostMax:   100.64.0.62          01100100.01000000.00000000.00 111110  
>Broadcast: 100.64.0.63          01100100.01000000.00000000.00 111111  
>Hosts/Net: 62                    Class A  
>  
>2. Requested size: 50 hosts  
>Netmask:   255.255.255.192 = 26 11111111.11111111.11111111.11 000000  
>Network:   100.64.0.64/26       01100100.01000000.00000000.01 000000  
>HostMin:   100.64.0.65          01100100.01000000.00000000.01 000001  
>HostMax:   100.64.0.126         01100100.01000000.00000000.01 111110  
>Broadcast: 100.64.0.127         01100100.01000000.00000000.01 111111  
>Hosts/Net: 62                    Class A  
  
## 7)  
#### Linux:  
>vagrant@sandbox:~$ arp -vn  
>Address                  HWtype  HWaddress           Flags Mask            Iface  
>10.0.2.3                 ether   52:54:00:12:35:03   C                     eth0  
>10.0.2.2                 ether   52:54:00:12:35:02   C                     eth0  
  
#### Windows:  
>C:\Users\andrew>arp -a  
>  
>Интерфейс: 192.168.56.1 --- 0xb  
>  адрес в Интернете      Физический адрес      Тип  
>  192.168.56.255        ff-ff-ff-ff-ff-ff     статический  
>  224.0.0.22            01-00-5e-00-00-16     статический  
>  224.0.0.251           01-00-5e-00-00-fb     статический  
>  224.0.0.252           01-00-5e-00-00-fc     статический  
>  239.255.255.250       01-00-5e-7f-ff-fa     статический  
>  
>Интерфейс: 192.168.1.2 --- 0xd  
>  адрес в Интернете      Физический адрес      Тип  
>  192.168.1.1           2c-e4-12-67-75-dd     динамический  
>  192.168.1.4           00-0e-c6-a8-01-75     динамический  
>  192.168.1.255         ff-ff-ff-ff-ff-ff     статический  
>  224.0.0.22            01-00-5e-00-00-16     статический  
>  224.0.0.251           01-00-5e-00-00-fb     статический  
>  224.0.0.252           01-00-5e-00-00-fc     статический  
>  239.255.255.250       01-00-5e-7f-ff-fa     статический  
>  255.255.255.255       ff-ff-ff-ff-ff-ff     статический  
  
#### Добавим хост 10.0.2.20, чтоб потом удалить:  
  
>vagrant@sandbox:\~$ sudo arp -n -s 10.0.2.20 52:54:00:12:35:01  
>vagrant@sandbox:\~$ arp -vn  
>Address                  HWtype  HWaddress           Flags Mask            Iface  
>10.0.2.3                 ether   52:54:00:12:35:03   C                     eth0  
>10.0.2.2                 ether   52:54:00:12:35:02   C                     eth0  
>10.0.2.20                ether   52:54:00:12:35:01   CM                    eth0  
  
#### Удаляем хост 10.0.2.20:  
  
>vagrant@sandbox:\~$ sudo arp -d 10.0.2.20  
>vagrant@sandbox:\~$ arp -vn  
>Address                  HWtype  HWaddress           Flags Mask            Iface  
>10.0.2.3                 ether   52:54:00:12:35:03   C                     eth0  
>10.0.2.2                 ether   52:54:00:12:35:02   C                     eth0  
  
#### Очистить ARP кеш полностью:  
  
>vagrant@sandbox:\~$ sudo ip -s -s neigh flush all  
>10.0.2.3 dev eth0 lladdr 52:54:00:12:35:03 used 409/404/387 probes 1 STALE  
>10.0.2.2 dev eth0 lladdr 52:54:00:12:35:02 ref 1 used 24/0/24 probes 1 REACHABLE  
>  
>*** Round 1, deleting 2 entries ***  
>*** Flush is complete after 1 round ***  
>  
>vagrant@sandbox:\~$ arp -vn  
>Address                  HWtype  HWaddress           Flags Mask            Iface  
>10.0.2.2                 ether   52:54:00:12:35:02   C                     eth0  
>Entries: 1      Skipped: 0      Found: 1  
  
## 8)  

