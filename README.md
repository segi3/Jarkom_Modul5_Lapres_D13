## Praktikum Modul 4
Kelompok D13
- 05111840000094 Rafi Nizar Abiyyi
- 05111740000192 Faishal Abiyyudzakir

<br>

> No 1

di surabaya
```c
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o eth0 -j SNAT --to-source 10.151.78.58
```

> No 2

di surabaya
```c
iptables -N VAR2
iptables -A VAR2 -j LOG --log-prefix 'DROPPED PACKET FROM PORT 22 =>' --log-level 6
iptables -A VAR2 -j DROP

iptables -A FORWARD -i eth0 -p tcp -d 10.151.79.112/29 --dport 22 -j VAR2
```

> No 3

di malang dan mojokerto
```c
iptables -N VAR3
iptables -A VAR3 -j LOG --log-prefix 'DROPPED PASSING ICMP =>' --log-level 6
iptables -A VAR3 -j DROP

iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j VAR3
```

> No 4

di malang
```c
iptables -N VAR4
iptables -A VAR4 -j LOG --log-prefix 'DROPPED PACKET =>' --log-level 6
iptables -A VAR4 -j REJECT
iptables -A INPUT -s 192.168.2.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT

iptables -A INPUT -s 192.168.2.0/24 -j VAR4
```

> No 5

di malang
```c
iptables -N VAR5
iptables -A VAR5 -j LOG --log-prefix 'DROPPED PACKET =>' --log-level 6
iptables -A VAR5 -j REJECT
iptables -A INPUT -s 192.168.3.0/24 -m time --timestart 07:00 --timestop 17:00 -j VAR5
```

> No 6

di surabaya
```c
iptables -t nat -A PREROUTING -p tcp -d 10.151.79.114 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.0.10:80
iptables -t nat -A PREROUTING -p tcp -d 10.151.79.114 -j DNAT --to-destination 192.168.0.11:80
```

> No 7