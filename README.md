# Jarkom-Modul-2-ITA04-2022
Nama Anggota | NRP
------------------- | --------------		
Nida'ul Faizah | 5027201064
Kevin Oktoaria | 5027201046
Najwa Amelia Qorry 'Aina | 5027201001

## Soal 1
Twilight (〈黄昏 (たそがれ) 〉, <Tasogare>) adalah seorang mata-mata yang berasal dari negara Westalis. Demi menjaga perdamaian antara Westalis dengan Ostania, Twilight dengan nama samaran Loid Forger (ロイド・フォージャー, Roido Fōjā) di bawah organisasi WISE menjalankan operasinya di negara Ostania dengan cara melakukan spionase, sabotase, penyadapan dan kemungkinan pembunuhan. Berikut adalah peta dari negara Ostania:

![Foto](./img/soal1.PNG)

WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua node terhubung pada router Ostania, sehingga dapat mengakses internet.

### Jawab
Pertama-tama kita perlu membuat Topologi sesuai soal, Topologi yang telah kami buat adalah sebagai berikut :

![Foto](./img/1a.PNG)

Setelah itu kita perlu melakukan konfigurasi pada setiap node, berikut adalah konfigurasi setiap node :

#### Konfigurasi Ostania
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

#### Konfigurasi SSS
```
auto eth0
iface eth0 inet static
	address 192.212.2.2
	netmask 255.255.255.0
	gateway 192.212.2.1
```

#### Konfigurasi Garden
```
auto eth0
iface eth0 inet static
	address 192.212.2.3
	netmask 255.255.255.0
	gateway 192.212.2.1
```

#### Konfigurasi WISE
```
auto eth0
iface eth0 inet static
	address 192.212.1.2
	netmask 255.255.255.0
    gateway 192.212.1.1
```

#### Konfigurasi Berlint
```
auto eth0
iface eth0 inet static
	address 192.212.3.2
	netmask 255.255.255.0
	gateway 192.212.3.1
```

#### Konfigurasi Eden
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
```

#### Testing
Untuk melakukan testing, kita dapat menjalankan command `PING google.com`
![Foto](./img/1b.PNG)

## Soal 2
Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder WISE.

### Jawab
Pertama melakukan update package lists dan install aplikasi bind9 pada WISE. Kemudian, membuat konfigurasi domain `wise.ita04.com` pada file `/etc/bind/named.conf.local` dan buat folder wise pada /etc/bind dengan command sebagai berikut:
#### Script wise.sh
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
Langkah selanjutnya, membuat konfigurasi domain menjadi www.ita04.com lalu membuat CNAME www untuk wise.ita04.com dengan cara melakukan konfigurasi pada file `/etc/bind/wise/wise.ita04.com`, command sebagai berikut:
#### Script wise.sh
```
echo "
;
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA     wise.ita04.com. root.wise.ita04.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      wise.ita04.com.
@               IN      A       192.212.1.2
www             IN      CNAME   wise.ita04.com.
"> /etc/bind/wise/wise.ita04.com
```

#### Testing
Berhasil mencoba ping ke `wise.ita04.com` dan `www.wise.ita04.com` serta mengecek CNAME dari `www.wise.ita04.com`
![Foto](./img/2a.PNG)

## Soal 3
Setelah itu ia juga ingin membuat subdomain eden.wise.yyy.com dengan alias www.eden.wise.yyy.com yang diatur DNS-nya di WISE dan mengarah ke Eden.

### Jawab
Dilakukan dengan menambahkan konfigurasi pada file `/etc/bind/wise/wise.ita04.com`, command sebagai berikut:
#### Script wise.sh
```
echo "
;
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA     wise.ita04.com. root.wise.ita04.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      wise.ita04.com.
@               IN      A       192.212.1.2
www             IN      CNAME   wise.ita04.com.
eden            IN      A       192.212.3.3
www.eden        IN      CNAME   eden.wise.ita04.com.
"> /etc/bind/wise/wise.ita04.com
```

#### Testing
Me-restart service bind9 dan kemudian mencoba ping serta mengecek IPv4 address dari `eden.wise.ita04.com`
![Foto](./img/3a.PNG)

Mencoba ping `www.eden.wise.ita04.com` serta cek alias dari `www.eden.wise.ita04.com`
![Foto](./img/3b.PNG)

## Soal 4
Buat reverse domain untuk domain utama.

### Jawab
Pertama, menambahkan konfigurasi pada file `/etc/bind/named.conf.local`, command sebagai berikut:
#### Script wise.sh
```
zone \"1.212.192.in-addr.arpa\" {
	type master;
	file \"/etc/bind/wise/1.212.192.in-addr.arpa\";
};
```
Kemudian ditambahkan juga konfigurasi yang ada pada file `/etc/bind/wise/1.212.192.in-addr.arpa` seperti berikut:
```
echo "
;
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA     wise.ita04.com. root.wise.ita04.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
1.212.192.in-addr.arpa. IN      NS      wise.ita04.com.
2                       IN      PTR     wise.ita04.com.
" > /etc/bind/wise/1.212.192.in-addr.arpa
```

#### Testing
Mengecek host yang ditunjuk dari reverse domain utama
![Foto](./img/4a.PNG)

## Soal 5
Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama 

### Jawab
Untuk menjadikan Berlint sebagai DNS Slave, kami melakukan konfigurasi pada `/etc/bind/named.conf.local` menjadi sebagai berikut:
#### Pada Wise
```
zone \"wise.ita04.com\" {
	type master;
	notify yes;
	also-notify { 192.212.3.2; };
	allow-transfer { 192.212.3.2; };
	file \"/etc/bind/wise/wise.ita04.com\";
};

zone \"1.212.192.in-addr.arpa\" {
	type master;
	file \"/etc/bind/wise/1.212.192.in-addr.arpa\";
};
```
#### Pada Berlint
Melakukan `apt-get update` dan menginstall bind9 dengan cara `apt-get install bind9 -y` dikarenakan Berlint akan dijadikan DNS Slave

Adapun konfigurasi pada file `/etc/bind/named.conf.local` sebagai berikut: 
```
zone \"wise.ita04.com\" {
    type slave;
    masters { 192.212.1.2; };
    file \"/var/lib/bind/wise.ita04.com\";
};
```
Setelah itu lakukan restart sevice bind9 dengan `service bind9 restart`

#### Testing
Pertama, kami memberhentikan service bind9 pada WISE menggunakan `service bind9 stop` kemudian melakukan ping pada SSS
![Foto](./img/5a.PNG)

## Soal 6
Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu operation.wise.yyy.com dengan alias www.operation.wise.yyy.com yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation 

### Jawab
Pertama kita harus ke WISE dahulu dimana kita akan mengkonfigurasikan wise.ita04.com di `/etc/bind/wise/wise.ita04.com`

#### Pada WISE
```
$TTL    604800
@       IN      SOA     wise.ita04.com. root.wise.ita04.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      wise.ita04.com.
@               IN      A       192.212.1.2
www             IN      CNAME   wise.ita04.com.
eden            IN      A       192.212.3.3
www.eden        IN      CNAME   eden.wise.ita04.com.
ns1             IN      A       192.212.3.2
operation       IN      NS      ns1
```
Selain itu kami juga menambahkan konfigurasi `allow-query{any;};` pada `/etc/bind/named.conf.options` seperti pada berikut ini:
```
options {
        directory \"/var/cache/bind\";
        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113
        // If your ISP provided one or more IP addresses for stable 
        // nameservers, you probably want to use them as forwarders.  
        // Uncomment the following block, and insert the addresses replacing 
        // the all-0's placeholder.
        // forwarders {
        //      0.0.0.0;
        // };
        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        //dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

#### Pada Berlint
Kami juga menambahkan zone pada file `/etc/bind/named.conf.local` pada Berlint untuk mendelegasikan `operation.wise.ita04.com`
```
zone \"wise.ita04.com\" {
    type slave;
    masters { 192.212.1.2; };
    file \"/var/lib/bind/wise.ita04.com\";
};
zone \"operation.wise.ita04.com\" {
        type master;
        file \"/etc/bind/operation/operation.wise.ita04.com\";
```

Terakhir, kami membuat direktori baru yaitu `/etc/bind/operation/operation.wise.ita04.com` dan mengedit konfigurasinya agar terlihat seperti berikut:
```
$TTL    604800
@       IN      SOA     operation.wise.ita04.com. root.operation.wise.ita04.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      operation.wise.ita04.com.
@               IN      A       192.212.3.3
www             IN      CNAME   operation.wise.ita04.com.
```

#### Testing
Di SSSS, kami melakukan ping pada `operation.wise.ita04.com` dan juga `wwww.operation.wise.ita04.com`
![Foto](./img/6a.PNG)
![Foto](./img/6b.PNG)

## Soal 7
Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses `strix.operation.wise.yyy.com` dengan alias `www.strix.operation.wise.yyy.com` yang mengarah ke Eden

### Jawab
Pada soal ini kami hanya perlu menambahkan konfigurasi pada file `/etc/bind/operation/operation.wise.ita04.com` sehingga terlihat seperti berikut ini:

#### Pada Berlint
```
$TTL    604800
@       IN      SOA     operation.wise.ita04.com. root.operation.wise.ita04.com$
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      operation.wise.ita04.com.
@               IN      A       192.212.3.3
www             IN      CNAME   operation.wise.ita04.com.
strix           IN      A       192.212.3.3
www.strix       IN      CNAME   strix.operation.wise.ita04.com.
```

#### Testing
Pada SSS, kami melakukan ping seperti berikut ini:
![Foto](./img/7a.PNG)


## Soal 8
Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.wise.yyy.com. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com.

### Jawab
Pertama melakukan install apache, php, openssl, dsb pada Eden:
```
apt-get install apache2 -y
service apache2 start
apt-get install php -y
apt-get install libapache2-mod-php7.0 -y
apt-get install ca-certificates openssl -y
apt-get install apache2-utils -y
```
Serta melakukan update package dan instal lynx pada client SSS dan Garden.
```
apt-get update
apt-get install dnsutils -y
apt-get install lynx -y
``` 
Kemudian, membuat directory `wise.ita04.com` pada file `var/www/` dan dilakukan konfigurasi dari subdomain di `/etc/apache2/sites-available/wise.ita04.com.conf` sehingga DocumentRoot dari subdomain www.wise.yyy.com terletak di `/var/www/wise.ita04.com` sebagai berikut:
```
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/wise.ita04.com
    ServerName wise.ita04.com
    ServerAlias www.wise.ita04.com
```
#### Testing
Mencoba lynx wise.ita04.com
![Foto](./img/8a.PNG)
![Foto](./img/8b.PNG)


## Soal 9
Setelah itu, Loid juga membutuhkan agar url www.wise.yyy.com/index.php/home dapat menjadi www.wise.yyy.com/home.

### Jawab
Dilakukan dengan menambahkan sintaks berikut pada `/etc/apache2/sites-available/wise.ita04.com.conf` sehingga dari yang sebelumnya `/index.php/home` bisa langsung direct menjadi `/home`:
#### Script eden.sh
```
Alias "/home" "/var/www/wise.ita04.com/index.php/home"
```

#### Testing
Mengakses url `www.wise.ita04.com/home` dengan lynx
![Foto](./img/9a.PNG)
![Foto](./img/9b.PNG)

## Soal 10
Setelah itu, pada subdomain www.eden.wise.yyy.com, Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada `/var/www/eden.wise.yyy.com`.

### Jawab
Melakukan konfigurasi pada file `/etc/apache2/sites-available/wise.ita04.com.conf` seperti berikut:
#### Script eden.sh
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/eden.wise.ita04.com
        ServerName eden.wise.ita04.com
        ServerAlias www.eden.wise.ita04.com

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Selanjutnya membuat directory `eden.wise.ita04.com` pada /var/www/ dan mengundul file zip dari google drive resource serta menyalinnya. Kemudian dengan command `a2ensite` mengaktifkan virtualhost:
#### Script eden.sh
```
mkdir /var/www/eden.wise.ita01.com
wget -c "https://drive.google.com/uc?export=download&id=1S0XhL9ViYN7TyCj2W66BNEXQD2AAAw2e"
unzip /root/eden.wise.zip
cp -r /root/eden.wise/. /var/www/eden.wise.ita01.com
a2ensite eden.wise.ita01.com
a2enmod rewrite
service apache2 restart
```

#### Testing
Berhasil mencoba lynx `eden.wise.ita04.com`
![Foto](./img/10a.PNG)
![Foto](./img/10b.PNG)


## Soal 11
Akan tetapi, pada folder /public, Loid ingin hanya dapat melakukan directory listing saja.

### Jawab
Dalam Eden dilakukan konfigurasi di `/etc/apache2/sites-available/eden.wise.ita04.com.conf` dengan command sebagai berikut:

#### Script eden.sh
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/eden.wise.ita04.com
        ServerName eden.wise.ita04.com
        ServerAlias www.eden.wise.ita04.com

        <Directory /var/www/eden.wise.ita04.com/public>
                Options +Indexes
        </Directory>

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/wise.ita04.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>
```

#### Testing
Berhasil melakukan lynx `www.eden.wise.ita04.com/public`
![Foto](./img/11a.PNG)
![Foto](./img/11b.PNG)



## Soal 12
Tidak hanya itu, Loid juga ingin menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache.

### Jawab
Dalam Eden ditambahkan ErrorDocument yang mengarah ke file `/error/404.html` pada konfigurasi di `/etc/apache2/sites-available/eden.wise.ita04.com.conf` dengan command sebagai berikut:

#### Script eden.sh
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/eden.wise.ita04.com
        ServerName eden.wise.ita04.com
        ServerAlias www.eden.wise.ita04.com

        ErrorDocument 404 /error/404.html
        ErrorDocument 500 /error/404.html
        ErrorDocument 502 /error/404.html
        ErrorDocument 503 /error/404.html
        ErrorDocument 504 /error/404.html

        <Directory /var/www/eden.wise.ita04.com/public>
                Options +Indexes
        </Directory>

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/wise.ita04.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>
```

#### Testing
Berhasil mencoba lynx `eden.wise.ita04.com/hahaha`
![Foto](./img/12a.PNG)
![Foto](./img/12b.PNG)


## Soal 13
Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.eden.wise.yyy.com/public/js menjadi www.eden.wise.yyy.com/js.

### Jawab
Dalam Eden dilakukan tambahan konfigurasi alias di `/etc/apache2/sites-available/eden.wise.ita04.com.conf` dengan command sebagai berikut:
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/eden.wise.ita04.com
        ServerName eden.wise.ita04.com
        ServerAlias www.eden.wise.ita04.com

        ErrorDocument 404 /error/404.html
        ErrorDocument 500 /error/404.html
        ErrorDocument 502 /error/404.html
        ErrorDocument 503 /error/404.html
        ErrorDocument 504 /error/404.html

        <Directory /var/www/eden.wise.ita04.com/public>
                Options +Indexes
        </Directory>

        Alias \"/js\" \"/var/www/eden.wise.ita04.com/public/js\"

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/wise.ita04.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>
```

#### Testing
Berhasil mencoba lynx `www.eden.wise.ita04.com/js`
![Foto](./img/13a.PNG)
![Foto](./img/13b.PNG)
	

## Soal 14
Loid meminta agar www.strix.operation.wise.yyy.com hanya bisa diakses dengan port 15000 dan port 15500.

### Jawab
Dalam Eden dilakukan tambahan konfigurasi pada port 15000 dan 15500 di `/etc/apache2/sites-available/eden.wise.ita04.com.conf` dengan command sebagai berikut:

```
<VirtualHost *:15000>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/strix.operation.wise.ita04.com
        ServerName strix.operation.wise.ita04.com
        ServerAlias www.strix.operation.wise.ita04.com


        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

<VirtualHost *:15500>        
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/strix.operation.wise.ita04.com
        ServerName strix.operation.wise.ita04.com
        ServerAlias www.strix.operation.wise.ita04.com
        

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Langkah selanjutnya, mengaktifkan dengan a2ensite, merestart apache2, dan membuat direktori baru:

```
a2ensite strix.operation.wise.ita04.com
service apache2 restart
mkdir /var/www/strix.operation.wise.ita04.com
```

#### Testing
Mencoba lynx `strix.operation.wise.ita04.com:1500` dan `strix.operation.wise.ita04.com:15500`
![Foto](./img/14a.PNG)
![Foto](./img/14b.PNG)


## Soal 15
Dengan autentikasi username Twilight dan password opStrix dan file di /var/www/strix.operation.wise.yyy.

### Jawab
jawaban
```
this is where you write the syntax
```

#### Testing


## Soal 16
Setiap kali mengakses IP Eden akan dialihkan secara otomatis ke www.wise.yyy.com.

### Jawab
Menambahkan konfigurasi redirect 301 pada `/etc/apache2/sites-available/000-default.conf` dengan sintaks sebagai berikut:
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        Redirect 301 / http://www.wise.ita04.com


        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

#### Testing
Berhasil mencoba lynx `192.212.1.2`
![Foto](./img/16a.PNG)


## Soal 17
Karena website www.eden.wise.yyy.com semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring “eden” akan diarahkan menuju eden.png. Bantulah Agent Twilight dan Organisasi WISE menjaga perdamaian!.

### Jawab
Menambahkan konfigurasi rewriterule di .htaxxess pada direktori `/var/www/eden.wise.ita04.com/.htaccess` dengan sintaks sebagai berikut:
```
ErrorDocument 404 /error/404.html
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^public/images/.*eden.*$ public/images/eden.png [NC,L]
```

#### Testing
![Foto](./img/17a.PNG)