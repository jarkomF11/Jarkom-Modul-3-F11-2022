# Jarkom-Modul-3-F11-2022

### Kelompok F11

| **No** | **Nama** | **NRP** | 
| ------------- | ------------- | --------- |
| 1 | Ryo Hilmi Ridho  | 5025201192 | 
| 2 | Moh. Ilham Fakhri Zamzami | 5025201275 |
| 3 | Putu Andhika Pratama | 5025201132 |

## Soal 1

### Soal
Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server

### Jawaban
Install package pada masing masing node
- Westalis
```
apt-get update
apt-get install isc-dhcp-server -y

service isc-dhcp-server start
```
- Berlint
```
apt-get update
apt-get install squid -y
```
- WISE
```
apt-get update
apt-get install bind9 -y
```

Pada node Westalis, tambahkan config pada `/etc/default/isc-dhcp-server`
```
INTERFACES="eth0"
```

Untuk soal no 3, 4, dan 6 gunakan config `/etc/dhcp/dhcpd.conf` pada node Westalis
```
subnet 10.34.2.0 netmask 255.255.255.0{
    option routers 10.34.2.1;
}

# Switch 1
subnet 10.34.1.0 netmask 255.255.255.0 {
    range 10.34.1.50 10.34.1.88;
    range 10.34.1.120 10.34.1.155;
    option routers 10.34.1.1;
    option broadcast-address 10.34.1.255;
    option domain-name-servers 10.34.2.2;
    default-lease-time 350;
    max-lease-time 6900;
}

# Switch 3
subnet 10.34.3.0 netmask 255.255.255.0 {
    range 10.34.3.10 10.34.3.30;
    range 10.34.3.60 10.34.3.85;
    option routers 10.34.3.1;
    option broadcast-address 10.34.3.255;
    option domain-name-servers 10.34.2.2;
    default-lease-time 700;
    max-lease-time 6900;
}
```

## Soal 2

### Soal
Jadikan Ostania sebagai DHCP Relay

### Jawaban
Install package relay
```
apt-get update
apt-get instal isc-dhcp-relay -y
```
Lalu tambahkan config pada `/etc/default/isc-dhcp-relay`
```
SERVERS="10.34.2.4"
INTERFACES="eth1 eth2 eth3"
```

## Soal 3

### Soal
Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155

### Jawaban
Gunakan option berikut pada `dhcpd.conf`
```
range 10.34.1.50 10.34.1.88;
range 10.34.1.120 10.34.1.155;
```    

## Soal 4

### Soal
Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85

### Jawaban
Gunakan option berikut pada `dhcpd.conf`
```
range 10.34.3.10 10.34.3.30;
range 10.34.3.60 10.34.3.85;
```    

## Soal 5

### Soal
Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut.

### Jawaban
Pada node WISE, tambahkan config untuk `/etc/bind/named.conf.options`
```
forwarders {
      192.168.122.1;
};

allow-query{any;};
```

## Soal 6

### Soal
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit.

### Jawaban
Ubah default-lease-time dan max-lease-time di dalam `dhcpd.conf` menjadi
```
default-lease-time 350; # Switch 1
default-lease-time 700; # Switch 3
max-lease-time 6900;
```


## Soal 7

### Soal
Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13

### Jawaban
Jalankan `ip a` untuk mendapatkan hwaddress Eden

![image](images/7-1.png)

Lalu edit file `dhcpd.conf` pada Westalis tambahkan

![image](images/7-2.png)

Kemudia restart `service isc-dhcp-server restart`

Edit konfigurasi pada node Eden, Hardware addresss perlu di-setting juga di /etc/network/interfaces untuk mencegah bergantinya hwaddress saat project GNS3 dimatikan atau diexport.

![image](images/7-3.png)

Periksa IP Eden dengan melakukan `ip a`

![image](images/7-4.png)


## Proxy Server

### Soal 
SSS, Garden dan Eden digunakan sebagai client Proxy agar pertukaran informasi dapat terjamin keamanannya, juga untuk mencegah kebocoran data. Pada Proxy Server di Berlint, Loid berencana untuk mengatur bagaimana Client dapat mengakses internet. Artinya setiap client harus menggunakan Berlint sebagai HTTP & HTTPS proxy.

### Jawaban
Pada node Berlint edit file konfigurasi squid pada `/etc/squid/squid.conf`

![image](images/8-1.png)

Kemudian restart squid `service squid restart`

Pada node SSS, Garden dan Eden aktifkan proxy menggunakan IP Berlint

`export http_proxy=http://10.34.2.3:8080`

Coba akses halaman web http://its.ac.id

`lynx http://its.ac.id`

### Proxy Server Soal 1
Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)

### Jawaban

buat file baru `nano /etc/squid/acl.conf` lalu masukkan

```
acl NOT_WORKING_1 time MTWHF 00:00-08:00
acl NOT_WORKING_2 time MTWHF 17:00-24:00
acl NOT_WORKING_3 time SA 00:00-24:00
acl WORKING time MTWHF 08:00-17:00
```

Kemudian edit file `/etc/squid/squid.conf`

```
include /etc/squid/acl.conf

http_port 8080
visible_hostname Berlint

http_access deny WORKING
http_access allow all
```

### Proxy Server Soal 2
Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan) 

### Jawaban

edit `nano /etc/squid/acl.conf` lalu masukkan

```
acl NOT_WORKING_1 time MTWHF 00:00-08:00
acl NOT_WORKING_2 time MTWHF 17:00-24:00
acl NOT_WORKING_3 time SA 00:00-24:00
acl WORKING time MTWHF 08:00-17:00

acl LOID dstdomain loid-work.com
acl FRANKY dstdomain franky-work.com
```

Kemudian edit file `/etc/squid/squid.conf`

```
include /etc/squid/acl.conf

http_port 8080
visible_hostname Berlint

http_access allow WORKING LOID
http_access allow WORKING FRANKY
http_access deny WORKING
http_access allow NOT_WORKING_1 !LOID
http_access allow NOT_WORKING_1 !FRANKY
http_access allow NOT_WORKING_2 !LOID
http_access allow NOT_WORKING_2 !FRANKY
http_access allow NOT_WORKING_3 !LOID
http_access allow NOT_WORKING_3 !FRANKY
http_access deny all
```

Kemudian pada node WISE buat domain `loid-work.com` dan `franky-work.com` dengan mengedit file `/etc/bind/named.conf.local`

```
zone "loid-work.com" {
  type master;
  file "/etc/bind/loid-work/loid-work.com";
};
zone "franky-work.com" {
  type master;
  file "/etc/bind/franky-work/franky-work.com";
};
```

buat folder loid dan franky

```
mkdir /etc/bind/loid-work
mkdir /etc/bind/franky-work
```

Kemudian endit file `/etc/bind/loid-work/loid-work.com`

```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     loid-work.com.  root.loid-work.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      loid-work.com.
@       IN      A       10.34.2.2
@       IN      AAAA    ::1
```

Edit juga file `/etc/bind/franky-work/franky-work.com`

```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky-work.com. root.franky-work.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky-work.com.
@       IN      A       10.34.2.2
@       IN      AAAA    ::1
```

Buka file `/etc/apache2/sites-available/loid-work.com.conf` lalu tambahkan

```
ServerAdmin webmaster@localhost
DocumentRoot /var/www/loid-work.com
ServerName loid-work.com
```

Buka file `/etc/apache2/sites-available/franky-work.com.conf` lalu tambahkan

```
ServerAdmin webmaster@localhost
DocumentRoot /var/www/franky-work.com
ServerName franky-work.com
```

Aktifkan domain dengan `a2ensite loid-work.com` dan  `a2ensite franky-work.com`

Buat folder untuk webnya

```
mkdir /var/www/loid-work.com
mkdir /var/www/franky-work.com
```

Kemudian buat file `index.php` lalu masukkan

```
<?php
        phpinfo();
?>
```

Coba jalankan

![image](images/8-2.png)

![image](images/8-2_1.png)

### Proxy Server Soal 3
Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)

### Jawaban
Edit file `/etc/squid/squid.conf` tambahkan

```
acl HTTPS port 443

http_access deny HARI_KERJA HTTPS
http_access deny AVAILABLE_WORKING !HTTPS
http_access deny AVAILABLE_WORKING2 !HTTPS
http_access deny HARI_LIBUR !HTTPS
```

### Proxy Server Soal 4 dan 5
Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)

### Jawaban
Buat file bernama acl-bandwidth.conf di folder squid `nano /etc/squid/acl-bandwidth.conf` lalu masukkan

```
delay_pools 2
delay_class 1 1
delay_class 1 2
delay_access 1 allow all
delay_access 2 allow all
delay_parameters 2 128000/128000 128000/128000
```

Kemudian edit file `/etc/squid/squid.conf`

```
include /etc/squid/acl-bandwidth.conf
```

Coba kecepatan internet dengan speedtest

![image](images/8-4.png)

