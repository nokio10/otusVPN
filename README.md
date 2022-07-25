# otusVPN



## Задание 

1. Между двумя виртуалками поднять vpn в режимах
    tun;
    tap; Прочуствовать разницу.
2. Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на виртуалку. 
3*. Самостоятельно изучить, поднять ocserv и подключиться с хоста к виртуалке Формат сдачи ДЗ - vagrant + ansible

## Решение 

1. После развертывания стенда командой ```vagrant up``` получаю настроенный vpn в режиме tap. Проверяю работу. 

```
[root@client ~]# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  4] local 10.10.10.2 port 38304 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec  52.7 MBytes  88.4 Mbits/sec   41   1.03 MBytes
[  4]   5.00-10.00  sec  54.6 MBytes  91.7 Mbits/sec    3    979 KBytes
[  4]  10.00-15.00  sec  55.9 MBytes  93.8 Mbits/sec    0   1.07 MBytes
[  4]  15.00-20.00  sec  55.9 MBytes  93.7 Mbits/sec    0   1.10 MBytes
[  4]  20.00-25.00  sec  50.8 MBytes  85.3 Mbits/sec   10   1.02 MBytes
[  4]  25.00-30.00  sec  55.9 MBytes  93.8 Mbits/sec    0   1.25 MBytes
[  4]  30.00-35.00  sec  53.3 MBytes  89.4 Mbits/sec    2   1.05 MBytes
[  4]  35.00-40.00  sec  55.9 MBytes  93.7 Mbits/sec    0   1.07 MBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-40.00  sec   435 MBytes  91.2 Mbits/sec   56             sender
[  4]   0.00-40.00  sec   433 MBytes  90.8 Mbits/sec                  receiver

iperf Done.
```

Редактирую ```/etc/openvpn/server.conf``` на сервере и клиента, заменяю первую строку на ```dev tun```, перезапускаю сервис ```systemctl restart openvpn@server```
Проверяю работу

```
[root@client ~]# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  4] local 10.10.10.2 port 38288 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec  52.6 MBytes  88.2 Mbits/sec   21    640 KBytes
[  4]   5.00-10.01  sec  55.7 MBytes  93.4 Mbits/sec    2    555 KBytes
[  4]  10.01-15.01  sec  56.3 MBytes  94.5 Mbits/sec    0    597 KBytes
[  4]  15.01-20.00  sec  54.3 MBytes  91.2 Mbits/sec    2    606 KBytes
[  4]  20.00-25.00  sec  54.6 MBytes  91.6 Mbits/sec  152    521 KBytes
[  4]  25.00-30.01  sec  54.8 MBytes  91.8 Mbits/sec    0    577 KBytes
[  4]  30.01-35.01  sec  56.1 MBytes  94.1 Mbits/sec    6    603 KBytes
[  4]  35.01-40.00  sec  56.1 MBytes  94.2 Mbits/sec    2    538 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-40.00  sec   440 MBytes  92.4 Mbits/sec  185             sender
[  4]   0.00-40.00  sec   439 MBytes  92.0 Mbits/sec                  receiver

iperf Done.
```

Скорость соединения в обоих режимах получилась примерно одинаковой в рамках одной машины. 
TAP работает на канальной уровне модели OSI, а TUN на сетевом. 
TAP интересен, если необходимо иметь устройства в одном подсети на обоих концах VPN без дополнительной маршрутизации, но при этом возрастут расходы в виде дополнительных заголовков. 

2. Проверяю работу RAS на базе OpenVPN на сервере ras. Сервер настраивается с помощью ansible, никаких дополнительных действий не требуется. 

```
root@ubuntu:~/otusVPN/ansible# openvpn --config ./client.conf
Tue Jul 26 02:08:01 2022 OpenVPN 2.4.7 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on Mar 22 2022
Tue Jul 26 02:08:01 2022 library versions: OpenSSL 1.1.1f  31 Mar 2020, LZO 2.10
Tue Jul 26 02:08:01 2022 WARNING: No server certificate verification method has been enabled.  See http://openvpn.net/howto.html#mitm for more info.
Tue Jul 26 02:08:01 2022 TCP/UDP: Preserving recently used remote address: [AF_INET]192.168.10.21:1207
Tue Jul 26 02:08:01 2022 Socket Buffers: R=[212992->212992] S=[212992->212992]
Tue Jul 26 02:08:01 2022 UDP link local (bound): [AF_INET][undef]:1194
Tue Jul 26 02:08:01 2022 UDP link remote: [AF_INET]192.168.10.21:1207
Tue Jul 26 02:08:01 2022 TLS: Initial packet from [AF_INET]192.168.10.21:1207, sid=ce1b38c7 ea95d709
Tue Jul 26 02:08:01 2022 VERIFY OK: depth=1, CN=rasvpn
Tue Jul 26 02:08:01 2022 VERIFY OK: depth=0, CN=rasvpn
Tue Jul 26 02:08:01 2022 Control Channel: TLSv1.2, cipher TLSv1.2 ECDHE-RSA-AES256-GCM-SHA384, 2048 bit RSA
Tue Jul 26 02:08:01 2022 [rasvpn] Peer Connection Initiated with [AF_INET]192.168.10.21:1207
Tue Jul 26 02:08:02 2022 SENT CONTROL [rasvpn]: 'PUSH_REQUEST' (status=1)
Tue Jul 26 02:08:02 2022 PUSH: Received control message: 'PUSH_REPLY,route 10.10.10.0 255.255.255.0,topology net30,ping 10,ping-restart 120,ifconfig 10.10.10.6 10.10.10.5,peer-id 0,cipher AES-256-GCM'
Tue Jul 26 02:08:02 2022 OPTIONS IMPORT: timers and/or timeouts modified
Tue Jul 26 02:08:02 2022 OPTIONS IMPORT: --ifconfig/up options modified
Tue Jul 26 02:08:02 2022 OPTIONS IMPORT: route options modified
Tue Jul 26 02:08:02 2022 OPTIONS IMPORT: peer-id set
Tue Jul 26 02:08:02 2022 OPTIONS IMPORT: adjusting link_mtu to 1625
Tue Jul 26 02:08:02 2022 OPTIONS IMPORT: data channel crypto options modified
Tue Jul 26 02:08:02 2022 Data Channel: using negotiated cipher 'AES-256-GCM'
Tue Jul 26 02:08:02 2022 Outgoing Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Tue Jul 26 02:08:02 2022 Incoming Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Tue Jul 26 02:08:02 2022 ROUTE_GATEWAY 192.168.1.1/255.255.255.0 IFACE=wlp3s0 HWADDR=cc:af:78:00:fd:7e
Tue Jul 26 02:08:02 2022 TUN/TAP device tun0 opened
Tue Jul 26 02:08:02 2022 TUN/TAP TX queue length set to 100
Tue Jul 26 02:08:02 2022 /sbin/ip link set dev tun0 up mtu 1500
Tue Jul 26 02:08:02 2022 /sbin/ip addr add dev tun0 local 10.10.10.6 peer 10.10.10.5
Tue Jul 26 02:08:02 2022 /sbin/ip route add 192.168.10.0/24 via 10.10.10.5
RTNETLINK answers: File exists
Tue Jul 26 02:08:02 2022 ERROR: Linux route add command failed: external program exited with error status: 2
Tue Jul 26 02:08:02 2022 /sbin/ip route add 10.10.10.0/24 via 10.10.10.5
Tue Jul 26 02:08:02 2022 WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this
Tue Jul 26 02:08:02 2022 Initialization Sequence Completed



root@ubuntu:~/otusVPN# ping -c 4 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=1.39 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=1.57 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=1.66 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=1.14 ms

--- 10.10.10.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
root@ubuntu:~/otusVPN# ip r
default via 192.168.1.1 dev wlp3s0 proto dhcp metric 600
10.10.10.0/24 via 10.10.10.5 dev tun0
10.10.10.5 dev tun0 proto kernel scope link src 10.10.10.6
```
