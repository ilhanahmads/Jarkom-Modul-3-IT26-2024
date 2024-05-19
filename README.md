# Jarkom-Modul-3-IT26-2024
Laporan Resmi pengerjaan soal shift modul 3 Praktikum Jaringan Komputer 2024 Kelompok IT26

## Anggota Kelompok
1. Ilhan Ahmad Syafa (5027221040)
2. George David Neborre (5027221043)

## Nomor 0 - 5
Buatlah topologi seperti berikut: \
<a href="https://ibb.co.com/j5t3FSw"><img src="https://i.ibb.co.com/H2ZGQjP/Screenshot-2024-05-18-001525.png" alt="Screenshot-2024-05-18-001525" border="0"></a> \
Kemudian setup konfig untuk setiap node seperti berikut:
```
//Arakis
auto eth0
iface eth0 inet dhcp
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.246.0.0/16

auto eth1
iface eth1 inet static
	address 192.246.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.246.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.246.3.1
	netmask 255.255.255.0

auto eth4
iface eth4 inet static
	address 192.246.4.1
	netmask 255.255.255.0

//Selain Client
//x menyesuaikan topologi eth tiap switch
//y menyesuaikan urutan node pada tiap switch

auto eth0
iface eth0 inet static
	address 192.246.x.y
	netmask 255.255.255.0
  gateway 192.246.x.1

//Client Dmitri
auto eth0
iface eth0 inet dhcp

//Client Paul
auto eth0
iface eth0 inet dhcp

```
Konfigurasi `/root/.bashrc` Arakis
```
Apt-get update
Apt-get install isc-dhcp-relay -y

```
Konfigurasi script arakis.sh
```
apt-get update
apt-get install isc-dhcp-relay -y
service isc-dhcp-relay start

relay="SERVERS=\"192.246.3.2\" 
INTERFACES=\"eth1 eth2 eth3 eth4\"
OPTIONS=\"\"
"
echo "$relay" > /etc/default/isc-dhcp-relay
echo net.ipv4.ip_forward=1 > /etc/sysctl.conf
service isc-dhcp-relay restart
```
Konfigurasi `/root/.bashrc` Irulan
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install bind9 -y
```
Buat script `.sh ` pada Irulan dan jalankan
```
apt-get update
apt-get install bind9 -y

echo 'zone "atreides.it26.com" {
        type master;
        file "/etc/bind/jarkom/atreides.it26.com";
};

zone "harkonen.it26.com" {
        type master;
        file "/etc/bind/jarkom/harkonen.it26.com";
};' > /etc/bind/named.conf.local

mkdir /etc/bind/jarkom

echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     atreides.it26.com. root.atreides.it26.com. (
                        2024051601      ; Serial
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        604800 )        ; Negative Cache TTL
;
@               IN      NS      atreides.it26.com.
@               IN      A       192.246.2.3 ; IP Leto' > /etc/bind/jarkom/atreides.it26.com

echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     harkonen.it26.com.  harkonen.it26.com.  (
                        2024051601      ; Serial
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        604800 )        ; Negative Cache TTL
;
@               IN      NS      harkonen.it26.com.
@               IN      A       192.246.1.3 ; IP Vladimir' > /etc/bind/jarkom/harkonen.it26.com

echo 'options {
        directory "/var/cache/bind";

        forwarders {
                192.168.122.1;
        };

        // dnssec-validation auto;
        allow-query{any;};
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
}; ' >/etc/bind/named.conf.options

service bind9 restart
```
Konfigurasi `/root/.bashrc` Mohiam
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install isc-dhcp-server -y
dhcpd --version

echo INTERFACES="eth0" > /etc/default/isc-dhcp-server

```
Buat konfigurasi `.sh` pada Mohiam dan jalankan
```
apt-get update
apt-get install isc-dhcp-server -y

interfaces="INTERFACESv4=\"eth0\"
INTERFACESv6=\"\"
"
echo "$interfaces" > /etc/default/isc-dhcp-server

subnet="option domain-name \"example.org\";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

ddns-update-style-none;

subnet 192.246.1.0 netmask 255.255.255.0 {
    range 192.246.1.14 192.246.1.28;
    range 192.246.1.49 192.246.1.70;
    option routers 192.246.1.1;
    option broadcast-address 192.246.1.255;
    option domain-name-servers 192.246.3.3;
    default-lease-time 300;
    max-lease-time 5220;
}


subnet 192.246.2.0 netmask 255.255.255.0 {
    range 192.246.2.15 192.246.2.25;
    range 192.246.2.200 192.246.2.210;
    option routers 192.246.2.1;
    option broadcast-address 192.246.2.255;
    option domain-name-servers 192.246.3.3;
    default-lease-time 1200;
    max-lease-time 5220;
}

subnet 192.246.3.0 netmask 255.255.255.0 {
}

subnet 192.246.4.0 netmask 255.255.255.0 {
}

"
echo "$subnet" > /etc/dhcp/dhcpd.conf

service isc-dhcp-server restart

apt-get update
apt-get install isc-dhcp-server -y

interfaces="INTERFACESv4=\"eth0\"
INTERFACESv6=\"\"
"
echo "$interfaces" > /etc/default/isc-dhcp-server

subnet="option domain-name \"example.org\";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

ddns-update-style-none;

subnet 192.246.1.0 netmask 255.255.255.0 {
    range 192.246.1.14 192.246.1.28;
    range 192.246.1.49 192.246.1.70;
    option routers 192.246.1.1;
    option broadcast-address 192.246.1.255;
    option domain-name-servers 192.246.3.3;
    default-lease-time 300;
    max-lease-time 5220;
}

subnet 192.246.2.0 netmask 255.255.255.0 {
    range 192.246.2.15 192.246.2.25;
    range 192.246.2.200 192.246.2.210;
    option routers 192.246.2.1;
    option broadcast-address 192.246.2.255;
    option domain-name-servers 192.246.3.3;
    default-lease-time 1200;
    max-lease-time 5220;
}

subnet 192.246.3.0 netmask 255.255.255.0 {
}

subnet 192.246.4.0 netmask 255.255.255.0 {
}

host Revolte {
    hardware ethernet aa:ce:c1:04:40:60;
    fixed-address 192.246.2.203;
}
"
echo "$subnet" > /etc/dhcp/dhcpd.conf

service isc-dhcp-server restart
```
Jika pada salah satu node client sudah bisa melakukan `ping`, maka konfigurasi seharusnya sudah berhasil \
<a href="https://ibb.co.com/qr4CkRH"><img src="https://i.ibb.co.com/crpwDNW/Screenshot-2024-05-18-045315.png" alt="Screenshot-2024-05-18-045315" border="0"></a>
