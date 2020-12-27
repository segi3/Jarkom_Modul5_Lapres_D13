## Praktikum Modul 4
Kelompok D13
- 05111840000094 Rafi Nizar Abiyyi
- 05111740000192 Faishal Abiyyudzakir

<br>

> A. Membuat topologi jaringan sesuai dengan rancangan yang diberikan Bibah

```c
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &
uml_switch -unix switch4 > /dev/null < /dev/null &
uml_switch -unix switch5 > /dev/null < /dev/null &
uml_switch -unix switch6 > /dev/null < /dev/null &

# Router
# NID TUNTAP 10.151.78.56/30    10.151.79.112/29
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.78.57 eth1=daemon,,,switch3 eth2=daemon,,,switch4 mem=96M &
xterm -T BATU -e linux ubd0=BATU,jarkom umid=BATU eth0=daemon,,,switch4 eth1=daemon,,,switch5 eth2=daemon,,,switch2 mem=96M &
xterm -T KEDIRI -e linux ubd0=KEDIRI,jarkom umid=KEDIRI eth0=daemon,,,switch1 eth1=daemon,,,switch6 eth2=daemon,,,switch3 mem=96M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=128M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &

# Klien 
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch5 mem=96M &
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch6 mem=96M &

# webserver
xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=daemon,,,switch1 mem=128M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch1 mem=128M &
```

<br>

> B. Bibah meminta kalian untuk membuat topologi tersebut menggunakan teknik CIDR atau VLSM.

Subnetting menggunakan VLSM

![](/img/B-1.PNG)
![](/img/B-3.PNG)
![](/img/B-2.PNG)

<br>

> C. Melakukan routing agar setiap perangkat pada jaringan tersebut dapat terhubung.

```c
// surabaya
route add -net 192.168.3.0 netmask 255.255.255.0 gw 192.168.0.6
route add -net 192.168.0.8 netmask 255.255.225.248 gw 192.168.0.6
route add -net 192.168.2.0 netmask 255.255.255.0 gw 192.168.0.2
route add -net 10.151.79.112 netmask 255.255.225.248 gw 192.168.0.2

// kediri
route add -net 0.0.0.0 netmask 0.0.0.0 gw 192.168.0.5

// batu
route add -net 0.0.0.0 netmask 0.0.0.0 gw 192.168.0.1
```

<br>

> D. memberikan ip pada subnet SIDOARJO dan GRESIK secara dinamis menggunakan bantuan DHCP SERVER dan DHCP RELAY

DHCP SERVER - MOJOKERTO
```c
subnet 192.168.2.0 netmask 255.255.255.0 {
    range 192.168.2.2 192.168.2.254;
    option routers 192.168.2.1;
    option broadcast-address 192.168.2.255;
    option domain-name-servers 10.151.79.114;
    default-lease-time 600;
    max-lease-time 7200;
}

subnet 192.168.3.0 netmask 255.255.255.0 {
    range 192.168.3.2 192.168.3.254;
    option routers 192.168.3.1;
    option broadcast-address 192.168.3.255;
    option domain-name-servers 10.151.79.114;
    default-lease-time 600;
    max-lease-time 7200;
}
```

DHCP RELAY - BATU
```c
interface: eth0 eth1
server: 10.151.79.115 
```

DHCP RELAY - SURABAYA
```c
interface: eth1 eth2
server: 10.151.79.115 
```

DHCP RELAY - KEDIRI
```c
interface: eth0 eth1 eth2
server: 10.151.79.115 
```

> 1. Mengkonfigurasi SURABAYA menggunakan iptables, namun Bibah tidak ingin kalian menggunakan MASQUERADE.

UML SURABAYA
```c
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o eth0 -j SNAT --to-source 10.151.78.58
```

> 2. mendrop semua akses SSH dari luar Topologi (UML) Kalian pada server yang memiliki ip DMZ (DHCP dan DNS SERVER) pada SURABAYA demi menjaga keamanan.

UML SURABAYA
```c
iptables -N VAR2
iptables -A VAR2 -j LOG --log-prefix 'DROPPED PACKET FROM PORT 22 =>' --log-level 6
iptables -A VAR2 -j DROP

iptables -A FORWARD -i eth0 -p tcp -d 10.151.79.112/29 --dport 22 -j VAR2
```

> 3. Bibah meminta kalian untuk membatasi DHCP dan DNS server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari mana saja menggunakan iptables pada masing masing server, selebihnya akan di DROP.

UML MALANG
```c
iptables -N VAR3
iptables -A VAR3 -j LOG --log-prefix 'DROPPED INCOMING ICMP =>' --log-level 6
iptables -A VAR3 -j DROP

iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j VAR3
```

UML MOJOKERTO
```c
iptables -N VAR3
iptables -A VAR3 -j LOG --log-prefix 'DROPPED INCOMING ICMP =>' --log-level 6
iptables -A VAR3 -j DROP

iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j VAR3
```

> 4. Akses dari subnet SIDOARJO hanya diperbolehkan pada pukul 07.00 - 17.00 pada hari Senin sampai Jumat. Selain itu paket akan di REJECT.

UML MALANG
```c
iptables -N VAR4
iptables -A VAR4 -j LOG --log-prefix 'DROPPED PACKET =>' --log-level 6
iptables -A VAR4 -j REJECT
iptables -A INPUT -s 192.168.2.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT

iptables -A INPUT -s 192.168.2.0/24 -j VAR4
```

> 5. Akses dari subnet GRESIK hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap harinya. Selain itu paket akan di REJECT.

UML MALANG
```c
iptables -N VAR5
iptables -A VAR5 -j LOG --log-prefix 'DROPPED PACKET =>' --log-level 6
iptables -A VAR5 -j REJECT
iptables -A INPUT -s 192.168.3.0/24 -m time --timestart 07:00 --timestop 17:00 -j VAR5
```

> 6. Bibah ingin SURABAYA disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada PROBOLINGGO port 80 dan MADIUN port 80.

UML SURABAYA
```c
iptables -t nat -A PREROUTING -p tcp -d 10.151.79.114 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.0.10:80
iptables -t nat -A PREROUTING -p tcp -d 10.151.79.114 -j DNAT --to-destination 192.168.0.11:80
```

> 7. Bibah ingin agar semua paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap UML yang memiliki aturan drop.

Logging digabung dalam jawaban setiap soal

```c
iptables -N VAR
iptables -A VAR -j LOG --log-prefix 'DROPPED PACKET =>' --log-level 6
iptables -A VAR -j DROP
...
```