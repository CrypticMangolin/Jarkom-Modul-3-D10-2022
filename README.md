# Jarkom-Modul-3-D10-2022
Repositori untuk Laporan Resmi praktikum mata kuliah Jarkom 2022 Modul 2: DNS dan WebServer

## Anggota Kelompok:
|Nama     |NRP    |
|----------|-------|
|Hasna Lathifah Purbaningdyah|5025201108|
|Anggito Anju Hartawan Manalu|5025201216|
|Hidayatullah|5025201031|

# Nomor 1
Membuat topologi sesuai gambar pada soal shift

<img width="664" alt="image" src="https://user-images.githubusercontent.com/80145586/201475924-fa784b73-1778-480b-9b9c-d3ba6f299efb.png">

IP Address dari masing-masing node adalah:

    - Ostania   : 10.20.1.1	| DHCP Relay | Router 
    - WISE      : 10.20.2.2	| DNS Server
    - Berlint   : 10.20.3.2	| Proxy Server
    - Westalia  : 10.20.3.3	| DHCP Server
    - SSS           : DHCP	| Client Subnet 1
    - Garden        : DHCP	| Client Subnet 1
    - Eden          : DHCP	| Client Subnet 3
    - NewstonCastle : DHCP	| Client Subnet 3
    - KemonoPark    : DHCP	| Client Subnet 3

## Isi File konfigurasi `no1.sh`

### Pada node Ostania
```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.20.0.0/16
cat /etc/resolv.conf
```

### Pada semua node kecuali Ostania
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```

### Pada node WISE sebagai DNS server
Menginstall bind
```
apt-get update
apt-get install bind9 -y
```
### Pada node Berlint sebagai Proxy server
Menginstall squid
```
apt-get update
apt-get install squid -y
```
### Pada node Westalis sebagai DHCP server
Menginstall isc-dhcp-server
```
apt-get update
apt-get install isc-dhcp-server -y
```

# Nomor 2
Mengatur Ostania sebagai DHCP relay
## Isi file `no2.sh` pada ostania
```
apt-get update

apt-get install isc-dhcp-relay 

echo 'net.ipv4.ip_forward=1'>/etc/sysctl.conf

service isc-dhcp-relay restart
```
## Isi file `/etc/default/isc-dhcp-relay`
```
# Defaults for isc-dhcp-relay initscript
# sourced by /etc/init.d/isc-dhcp-relay
# installed at /etc/default/isc-dhcp-relay by the maintainer scripts

#
# This is a POSIX shell fragment
#

# What servers should the DHCP relay forward requests to?
SERVERS="10.20.2.4"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth2 eth3"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```

# Nomor 3
Client yang melalui Switch1 mendapatkan range IP dari `10.20.1.50` - `10.20.1.88` dan `10.20.1.120` - `10.20.1.155`

Mengedit file `/etc/dhcp/dhcpd.conf`pada node Westalis
```
subnet 10.20.2.0 netmask 255.255.255.0 {
}

subnet 10.20.1.0 netmask 255.255.255.0 {
    range 10.20.1.50 10.20.1.88;
    range 10.20.1.120 10.20.1.155;
    option routers 10.20.1.1;
    option broadcast-address 10.20.1.255;
    option domain-name-servers 10.20.2.2;
    default-lease-time 600;
    max-lease-time 6900;
}
```
Mengedit line berikut pada file `/etc/default/isc-dhcp-server`
```
INTERFACES=`eth0`
```
Mengedit file `/etc/network/interfaces` pada semua node client
```
auto eth0
iface eth0 inet dhcp
```
## Hasil testing
### node SSS
<img width="500" alt="image" src="https://user-images.githubusercontent.com/80145586/201476012-136e71ab-531b-487e-bf0c-06ce9fe8205b.png">

### Node Garden
<img width="500" alt="image" src="https://user-images.githubusercontent.com/80145586/201476029-f890ea51-2ad8-42d9-8660-10b3afb80ab1.png">

# Nomor 4
Client yang melalui Switch3 mendapatkan range IP dari `10.20.3.10` - `10.20.3.30` dan `10.20.3.60` - `10.20.3.85`

Pada file `/etc/dhcp/dhcpd.conf` di node Westalis ditambahkan
```
subnet 10.20.3.0 netmask 255.255.255.0 {
    range 10.20.3.10 10.20.3.30;
    range 10.20.3.60 10.20.3.85;
    option routers 10.20.3.1;
    option broadcast-address 10.20.3.255;
    option domain-name-servers 10.20.2.2;
}
```
## Hasil testing
### node Eden
<img width="500" alt="image" src="https://user-images.githubusercontent.com/80145586/201476089-dc557f9c-7e7c-41af-a879-e2051ef604fe.png">

### node NewstonCastle
<img width="500" alt="image" src="https://user-images.githubusercontent.com/80145586/201476045-381ea187-2cd2-4253-9e1a-2053a523845b.png">

### node KemonoPark
<img width="500" alt="image" src="https://user-images.githubusercontent.com/80145586/201476060-8fbcfb18-b8de-41f9-b6c7-095ceb1c78e1.png">



# Nomor 5
Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut

## Pada node WISE
Mengedit line berikut pada file ‘/etc/bind/named.conf.options`
```
forwarders {
                192.168.122.1;
};

//dnssec-validation auto;
allow-query{any;};
```
## hasil testing
### sebelum
<img width="300" alt="image" src="https://user-images.githubusercontent.com/80145586/201476147-47a9a3f9-03dd-4f0a-9aa2-f06035c6b084.png">

### setelah
<img width="500" alt="image" src="https://user-images.githubusercontent.com/80145586/201476160-d6698957-8725-42e1-8a29-49b0fa5b3c4e.png">


# Nomor 6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit

Mengedit file ‘/etc/dhcp/dhcpd.conf` pada node Westalis
```
subnet 10.20.2.0 netmask 255.255.255.0 {
}

subnet 10.20.1.0 netmask 255.255.255.0 {
    range 10.20.1.50 10.20.1.88;
    range 10.20.1.120 10.20.1.155;
    option routers 10.20.1.1;
    option broadcast-address 10.20.1.255;
    option domain-name-servers 10.20.2.2;
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 10.20.3.0 netmask 255.255.255.0 {
    range 10.20.3.10 10.20.3.30;
    range 10.20.3.60 10.20.3.85;
    option routers 10.20.3.1;
    option broadcast-address 10.20.3.255;
    option domain-name-servers 10.20.2.2;
    default-lease-time 600;
    max-lease-time 6900;
}
```

## Hasil testing
### Node Garden (Switch 1)
<img width="510" alt="image" src="https://user-images.githubusercontent.com/80145586/201476170-479de0d1-7586-48a5-84ef-458767fca38b.png">

### Node KemonoPark (Switch 3)
<img width="510" alt="image" src="https://user-images.githubusercontent.com/80145586/201476187-c50aeb51-c827-41d1-85a3-702c93d2f38d.png">


# Nomor 7
Node Eden mendapatkan alamat IP tetap yaitu `10.20.3.13`
## Pada node Westalis
Mengedit file `/etc/dhcp/dhcpd.conf`
```
subnet 10.20.2.0 netmask 255.255.255.0 {
}

subnet 10.20.1.0 netmask 255.255.255.0 {
    range 10.20.1.50 10.20.1.88;
    range 10.20.1.120 10.20.1.155;
    option routers 10.20.1.1;
    option broadcast-address 10.20.1.255;
    option domain-name-servers 10.20.2.2;
    default-lease-time 600;
    max-lease-time 6900;
}

subnet 10.20.3.0 netmask 255.255.255.0 {
    range 10.20.3.10 10.20.3.30;
    range 10.20.3.60 10.20.3.85;
    option routers 10.20.3.1;
    option broadcast-address 10.20.3.255;
    option domain-name-servers 10.20.2.2;
    default-lease-time 600;
    max-lease-time 6900;
}

host Eden {
    hardware ethernet 5a:b7:3a:2b:e8:be;
    fixed-address 10.20.3.13;
}
```
## Pada node Eden
Mengedit file `/etc/network/interfaces`
```
auto eth0
iface eth0 inet dhcp

hwaddress ether 5a:b7:3a:2b:e8:be
```
## Hasil testing
### Pada node Eden
<img width="500" alt="image" src="https://user-images.githubusercontent.com/80145586/201476225-73cc973e-9232-45ba-9955-833d0046c07e.png">



