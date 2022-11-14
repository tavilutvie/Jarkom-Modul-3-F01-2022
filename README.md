# Jarkom-Modul-3-F01-2022

Anggota:
1. Eldenabih Tavirazin Lutvie (5025201213)
2. Nouval Bachrezi (502520...)
3. Marsa A R (...)

## Konfigurasi
### Ostania
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
address [IP PREFIX].1.1
netmask 255.255.255.0

auto eth2
iface eth2 inet static
address [IP PREFIX].2.1
netmask 255.255.255.0

auto eth3
iface eth3 inet static
address [IP PREFIX].3.1
netmask 255.255.255.0
```

### SSS & Garden
```
auto eth0
iface eth0 inet dhcp
```

### WISE
```
auto eth0
iface eth0 inet static
  address [IP PREFIX].2.2
  netmask 255.255.255.0
  gateway [IP PREFIX].2.1
```

### BERLINT
```
auto eth0
iface eth0 inet static
  address [IP PREFIX].2.3
  netmask 255.255.255.0
  gateway [IP PREFIX].2.1
```

### WESTALIS 
```
auto eth0
iface eth0 inet static
  address [IP PREFIX].2.4
  netmask 255.255.255.0
  gateway [IP PREFIX].2.1
```

### EDEN, NEWSTON CASTLE, & KEMONO PARK
```
auto eth0
iface eth0 inet dhcp
```

### Ostania
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.29.0.0/16
```
### Node Lain
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```

## Soal 1
Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server (1)
<br>

Command pada terminal nya adalah sebagai berikut.
<br>

### Wise
Install bind9 pada wise.
```
apt-get update
apt-get install bind9 -y
service bind9 start
```

### Westalis
Install isc-dhcp-server pada westalis.
```
apt-get update
apt-get install isc-dhcp-server -y
```
### Berlint
Install squid pada berlint.
```
apt-get update
apt-get install squid -y
service squid restart
```

## Soal 2
Ostania sebagai DHCP Relay (2)
### Ostania
Install isc-dhcp-relay -y pada ostania.
```
apt-get update
apt-get install isc-dhcp-relay -y

echo '
SERVERS = "10.29.2.4"
INTERFACES = "eth1 eth2 eth3"
' > /etc/default/isc-dhcp-relay

service isc-dhcp-relay restart
```

## Soal 3
Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155 (3)
### Westalis
Isi /etc/default/isc-dhcp-server dan /etc/dhcp/dhcpd.conf seperti berikut.
```
echo -e '
INTERFACES="eth0"
' > /etc/default/isc-dhcp-server

echo -e '
subnet 10.29.1.0 netmask 255.255.255.0 {
    range 10.29.1.50 10.29.1.88;
    range 10.29.1.120 10.29.1.155;
    option routers 10.29.1.1;
    option broadcast-address 10.29.1.255;
    option domain-name-servers 10.29.3.2;
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 10.29.2.0 netmask 255.255.255.0 {
    option routers 10.29.2.1;
}
' >> /etc/dhcp/dhcpd.conf

service isc-dhcp-server restart
```
Restart node client pada switch 1 terlebih dahulu dan masukkan ip a.
![image](https://user-images.githubusercontent.com/85897222/201658544-ccd9e606-e5d3-40bd-b3d8-faf7995fa87b.png)

## Soal 4
Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85 (4)
### Westalis
Tambahkan /etc/dhcp/dhcpd.conf seperti berikut.
```
subnet 10.29.3.0 netmask 255.255.255.0 {
    range 10.29.3.10 10.29.3.30;
    range 10.29.3.60 10.29.3.85;
    option routers 10.29.3.1;
    option broadcast-address 10.29.3.255;
    option domain-name-servers 10.29.2.2;
    default-lease-time 600;
    max-lease-time 6900;
}

service isc-dhcp-server restart
```
Restart node client pada switch 3 terlebih dahulu dan masukkan ip a.
![image](https://user-images.githubusercontent.com/85897222/201658653-b17eaca0-3a35-495a-84d5-ef3729668a44.png)


## Soal 5
Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut. (5)
### Wise
```
echo -e '
options {
    directory "/var/cache/bind";
    forwarders {
        192.168.122.1;
    };
    allow-query{any;};
    auth-nxdomain no;
    listen-on-v6 { any; };
};
' > /etc/bind/named.conf.options
service bind9 restart
```
Mencoba ping google dengan client pada switch 1.
![image](https://user-images.githubusercontent.com/85897222/201659647-0194f8ac-b7e4-4f2e-9d96-2cc9dcc8fe82.png)


Mencoba ping google dengan client pada switch 3.
![image](https://user-images.githubusercontent.com/85897222/201659380-d6527597-b204-4d60-b305-86c14b27348c.png)

## Soal 6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit. (6)

### Westalis
Ganti default-lease-time dan max-lease-time seperti berikut. <br>
Switch 1:
```
...
default-lease-time 300;
max-lease-time 6900;
...
```
![image](https://user-images.githubusercontent.com/85897222/201660358-981b590d-5ef2-4485-ac42-75c1f4a4568e.png)

Switch 2:
```
...
default-lease-time 600;
max-lease-time 6900;
...
```
![image](https://user-images.githubusercontent.com/85897222/201660409-bf950652-2eea-4b0d-a8aa-c240c71511e0.png)




## Kendala
Sering terjadi error karena terdapat sedikit typo.
