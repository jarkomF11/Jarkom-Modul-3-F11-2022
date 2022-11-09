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
