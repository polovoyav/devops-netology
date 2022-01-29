# 03-sysadmin-08-net3    
    
## 1)    
>vagrant@sandbox:\~$ wget -qO- eth0.me  
>79.126.61.2  
>  
>route-views>show ip route 79.126.61.2 255.255.255.255  
>% Subnet not in table  
  
>route-views>show ip route 79.126.61.2  
>Routing entry for 79.126.0.0/18  
>  Known via "bgp 6447", distance 20, metric 0  
>  Tag 2497, type external  
>  Last update from 202.232.0.2 1d13h ago  
>  Routing Descriptor Blocks:  
>  \* 202.232.0.2, from 202.232.0.2, 1d13h ago  
>      Route metric is 0, traffic share count is 1  
>      AS Hops 2  
>      Route tag 2497  
>      MPLS label: none  
>	    
>route-views>show ip bgp 79.126.61.2 255.255.255.255  
>% Network not in table  
>route-views>show ip bgp 79.126.61.2  
>BGP routing table entry for 79.126.0.0/18, version 281622839  
>Paths: (23 available, best #18, table default)  
>  Not advertised to any peer  
>  Refresh Epoch 1  
>  701 1273 12389  
>    137.39.3.55 from 137.39.3.55 (137.39.3.55)  
>      Origin IGP, localpref 100, valid, external  
>      path 7FE13908BBC8 RPKI State valid  
>      rx pathid: 0, tx pathid: 0  
>  Refresh Epoch 1  
>  20912 3257 1273 12389  
>    212.66.96.126 from 212.66.96.126 (212.66.96.126)  
>      Origin IGP, localpref 100, valid, external  
>      Community: 3257:8070 3257:30352 3257:50001 3257:53900 3257:53902 20912:65004  
>      path 7FE18D73D788 RPKI State valid  
>      rx pathid: 0, tx pathid: 0  
  
...  
  
## 2)  
>vagrant@sandbox:\~$ sudo modprobe dummy  
>vagrant@sandbox:\~$ lsmod | grep dummy  
>dummy                  16384  0  
  
>vagrant@sandbox:\~$ sudo su  
>root@sandbox:/home/vagrant# echo "options dummy numdummies=2" > /etc/modprobe.d/dummy.conf  
  
>root@sandbox:/home/vagrant# nano /etc/network/interfaces  
>>\# interfaces(5) file used by ifup(8) and ifdown(8)  
>>\# Include files from /etc/network/interfaces.d:  
>>source-directory /etc/network/interfaces.d  
>>auto dummy0  
>>iface dummy0 inet static  
>>address 10.2.2.2/32  
>>pre-up ip link add dummy0 type dummy  
>>post-down ip link del dummy0  
  
>root@sandbox:/home/vagrant# ifup dummy0  
>root@sandbox:/home/vagrant# ip -c -br a  
>lo               UNKNOWN        127.0.0.1/8 ::1/128  
>eth0             UP             10.0.2.15/24 fe80::a00:27ff:fe73:60cf/64  
>dummy0           UNKNOWN        10.2.2.2/32 fe80::58e4:bff:fe7a:295c/64  
  
>root@sandbox:/home/vagrant# ip route add 172.16.10.0/24 dev dummy0  
>root@sandbox:/home/vagrant# ip route add 10.0.0.0/8 dev dummy0  
>root@sandbox:/home/vagrant# ip route add 192.168.250.0/24 dev dummy0  
  
>root@sandbox:/home/vagrant# ip route show  
>default via 10.0.2.2 dev eth0 proto dhcp src 10.0.2.15 metric 100  
>10.0.0.0/8 dev dummy0 scope link  
>10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15  
>10.0.2.2 dev eth0 proto dhcp scope link src 10.0.2.15 metric 100  
>172.16.10.0/24 dev dummy0 scope link  
>192.168.250.0/24 dev dummy0 scope link  
  
## 3)    
>vagrant@sandbox:\~$ sudo ss -ltpn  
>State             Recv-Q            Send-Q                         Local Address:Port                          Peer Address:Port            Process  
>LISTEN            0                 4096                           127.0.0.53%lo:53                                 0.0.0.0:*                users:(("systemd-resolve",pid=558,fd=13))  
>LISTEN            0                 128                                  0.0.0.0:22                                 0.0.0.0:*                users:(("sshd",pid=662,fd=3))  
>LISTEN            0                 4096                               127.0.0.1:8125                               0.0.0.0:*                users:(("netdata",pid=651,fd=46))  
>LISTEN            0                 4096                                 0.0.0.0:19999                              0.0.0.0:*                users:(("netdata",pid=651,fd=4))  
>LISTEN            0                 4096                                 0.0.0.0:111                                0.0.0.0:*                users:(("rpcbind",pid=557,fd=4),("systemd",pid=1,fd=35))  
>LISTEN            0                 128                                     [::]:22                                    [::]:*                users:(("sshd",pid=662,fd=4))  
>LISTEN            0                 4096                                   [::1]:8125                                  [::]:*                users:(("netdata",pid=651,fd=35))  
>LISTEN            0                 4096                                       *:9100                                     *:*                users:(("node_exporter",pid=652,fd=3))  
>LISTEN            0                 4096                                    [::]:111                                   [::]:*                users:(("rpcbind",pid=557,fd=6),("systemd",pid=1,fd=37))  
  
Видим, что порт 53 слушает служба резолвера dns, порт 22 слушает служба sshd (протокол SSH), порт 9100 приложение node_exporter.  
  
## 4)    
>vagrant@sandbox:\~$ sudo ss -lupn  
>State             Recv-Q            Send-Q                          Local Address:Port                         Peer Address:Port            Process  
>UNCONN            0                 0                                   127.0.0.1:8125                              0.0.0.0:*                users:(("netdata",pid=651,fd=24))  
>UNCONN            0                 0                               127.0.0.53%lo:53                                0.0.0.0:*                users:(("systemd-resolve",pid=558,fd=12))  
>UNCONN            0                 0                              10.0.2.15%eth0:68                                0.0.0.0:*                users:(("systemd-network",pid=383,fd=19))  
>UNCONN            0                 0                                     0.0.0.0:111                               0.0.0.0:*                users:(("rpcbind",pid=557,fd=5),("systemd",pid=1,fd=36))  
>UNCONN            0                 0                                       [::1]:8125                                 [::]:*                users:(("netdata",pid=651,fd=23))  
>UNCONN            0                 0                                        [::]:111                                  [::]:*                users:(("rpcbind",pid=557,fd=7),("systemd",pid=1,fd=38))  
  
Видим, что порт 53 слушает служба резолвера dns, порт 68 использует протокол DHCP, порт 8125 приложение мониторинга netdata.  
  
## 5)    
L3 диаграмму сети приложил.  
  
## 6)    
  
## 7)    
  
## 8)  
