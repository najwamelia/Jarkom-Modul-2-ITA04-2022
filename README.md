# Jarkom-Modul-2-ITA04-2022
Nama Anggota | NRP
------------------- | --------------		
Nida'ul Faizah | 5027201064
Kevin Oktoaria | 5027201046
Najwa Amelia Qorry 'Aina | 5027201001

## Soal 1
WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua node terhubung pada router Ostania, sehingga dapat mengakses internet.

#### Jawab
Membuat Topologi pada GNS3 sebagai berikut:
![Foto](./img/1a.png)

Dengan konfigurasi setiap node:
##### Konfigurasi Ostania
```
auto eth1
iface eth1 inet static
	address 192.212.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.212.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.212.3.1
	netmask 255.255.255.0
```

##### Konfigurasi SSS
```
auto eth0
iface eth0 inet static
	address 192.212.2.2
	netmask 255.255.255.0
	gateway 192.212.2.1
```

##### Konfigurasi Garden
```
auto eth0
iface eth0 inet static
	address 192.212.2.3
	netmask 255.255.255.0
	gateway 192.212.2.1
```

##### Konfigurasi WISE
```
auto eth0
iface eth0 inet static
	address 192.212.1.2
	netmask 255.255.255.0
    gateway 192.212.1.1
```

##### Konfigurasi Berlint
```
auto eth0
iface eth0 inet static
	address 192.212.3.2
	netmask 255.255.255.0
	gateway 192.212.3.1
```

##### Konfigurasi Eden
```
auto eth0
iface eth0 inet static
	address 192.212.3.3
	netmask 255.255.255.0
	gateway 192.212.3.1
```
Dalam script bash ostania.sh kami  memasukkan command 
```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.212.0.0/16
cat /etc/resolv.conf
```
Selanjutnya, di script node lainnya emasukkan command
```
echo "nameserver 192.168.122.1" > /etc/resolv.conf

##### Testing
Berhasil terhubung dengan internet
![Foto](./img/1b.png)

## Soal 2
Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise.

#### Jawab
Melakukan konfigurasi pada file `/etc/bind/named.conf.local`
##### Script wise.sh
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update -y
apt-get install bind9 -y
echo "
zone \"wise.ita04.com\" {
	type master;
	file \"/etc/bind/wise/wise.ita04.com\";
};
"> /etc/bind/named.conf.local

mkdir -p /etc/bind/wise
```

##### Testing
Berhasil terhubung dengan internet
![Foto](./img/2a.png)
