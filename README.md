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
TAP интересен, если необходимо иметь устройства в одном подсети на обоих концах VPN без дополнительной маршрутизации, но при этом возрастут расходы в виде дополнительных заголовкой. 

2. 
