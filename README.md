# Jarkom-Modul-3-F11-2022

### Kelompok F11

| **No** | **Nama** | **NRP** | 
| ------------- | ------------- | --------- |
| 1 | Ryo Hilmi Ridho  | 5025201192 | 
| 2 | Moh. Ilham Fakhri Zamzami | 5025201275 |
| 3 | Putu Andhika Pratama | 5025201132 |

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

### Soal 1
Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)

### Jawaban

buat file baru `nano /etc/squid/acl.conf` lalu masukkan

```
acl AVAILABLE_WORKING_1 time MTWHF 00:00-08:00
acl AVAILABLE_WORKING_2 time MTWHF 17:00-24:00
acl AVAILABLE_WORKING_3 time SA 00:00-24:00
```

Kemudian edit file `/etc/squid/squid.conf`

```
include /etc/squid/acl.conf

http_port 8080
visible_hostname Berlint

http_access allow AVAILABLE_WORKING_1
http_access allow AVAILABLE_WORKING_2
http_access allow AVAILABLE_WORKING_3
http_access deny all
```