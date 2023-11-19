# Jarkom-Modul-3-B01-2023

| Name           | NRP        | Kelas     |
| ---            | ---        | ----------|
| Rr. Diajeng Alfisyahrinnisa Anandha | 5025211147 | Jaringan Komputer (B) |


# Laporan grimiore: https://drive.google.com/file/d/1NS1KcXgIiTNqHYVVXjZS9QiiYdGzXPUk/view?usp=sharing 

# Soal 0 dan 1
Register domain berupa riegel.canyon.yyy.com untuk worker Laravel dan granz.channel.yyy.com untuk worker PHP mengarah pada worker yang memiliki IP [prefix IP].x.1. Lakukan konfig yang sesuai

## Jawaban Soal 0 dan 1

### Kita tetapkan konfig IP pada masing-masing node:

1. Aura

    ```
    auto eth0
    iface eth0 inet dhcp
    up iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.9.0.0/16

    auto eth1
    iface eth1 inet static
        address 10.9.1.33
        netmask 255.255.255.0

    auto eth2
    iface eth2 inet static
        address 10.9.2.33
        netmask 255.255.255.0

    auto eth3
    iface eth3 inet static
        address 10.9.3.33
        netmask 255.255.255.0

    auto eth4
    iface eth4 inet static
        address 10.9.4.33
        netmask 255.255.255.0

    ```
2. Himmel

    ```
    auto eth0
    iface eth0 inet static
        address 10.9.1.2
        netmask 255.255.255.0
        gateway 10.9.1.33
    ```

3. Heiter

    ```
    auto eth0
    iface eth0 inet static
        address 10.9.1.1
        netmask 255.255.255.0
        gateway 10.9.1.33
    ```

4. Denken
    ```
    auto eth0
    iface eth0 inet static
        address 10.9.2.2
        netmask 255.255.255.0
        gateway 10.9.2.33
    ```

5. Eisen

    ```
    auto eth0
    iface eth0 inet static
        address 10.9.2.1
        netmask 255.255.255.0
        gateway 10.9.2.33

    ```

6. Node sein, stark, frieren, flamme, fern, revoite, richter, lawine, linie, lugner memiliki konfig yang sama yaitu

    ```
    auto eth0
    iface eth0 inet dhcp
    ```

### Untuk membuat dns, maka kita lakukan konfig di node heiter. Konfig bashnya adalah:

```
#!/bin/bash

apt-get update
apt-get install bind9 -y

echo 'zone "canyon.b01.com" {
    type master;
    file "/etc/bind/jarkom/canyon.b01.com";
};

zone "channel.b01.com" {
    type master;
    file "/etc/bind/jarkom/channel.b01.com";
};' > /etc/bind/named.conf.local

mkdir -p /etc/bind/jarkom
cp /etc/bind/db.local /etc/bind/jarkom/canyon.b01.com
cp /etc/bind/db.local /etc/bind/jarkom/channel.b01.com

echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     canyon.b01.com. root.canyon.b01.com. (
                        2023111301    ; Serial
                        604800        ; Refresh
                        86400         ; Retry
                        2419200       ; Expire
                        604800 )      ; Negative Cache TTL
;
@               IN      NS      canyon.b01.com.
@               IN      A       10.9.2.1 ; IP Eisen LB
www             IN      CNAME   canyon.b01.com.
riegel          IN      A       10.9.4.1 ; IP Frieren' > /etc/bind/jarkom/canyon.b01.com

echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     channel.b01.com. root.channel.b01.com. (
                        2023111301    ; Serial
                        604800        ; Refresh
                        86400         ; Retry
                        2419200       ; Expire
                        604800 )      ; Negative Cache TTL
;
@               IN      NS      channel.b01.com.
@               IN      A       10.9.2.1 ; IP Eisen LB
www             IN      CNAME   channel.b01.com.
granz           IN      A       10.9.3.1 ; IP Lawine' > /etc/bind/jarkom/channel.b01.com

echo '
options {
        directory "/var/cache/bind";
        forwarders {
                192.168.122.1;
        };
        // dnssec-validation auto;
        allow-query{any;};
        auth-nxdomain no; # conform to RFC1035
        listen-on-v6 { any; };
}; ' >/etc/bind/named.conf.options

service bind9 restart
service bind9 restart
```

> Disini, kita arahkan domain utama yaitu `canyon.b01.com` ke IP dari Eisen yang menjadi loadbalancer. Lalu, kita arahkan subdomain `riegel.canyon.b01.com` ke IP dari worker Frieren yang memiliki IP [prefix IP].x.1.

> Kita juga arahkan domain utama yaitu `channel.b01.com` ke IP dari Eisen yang menjadi loadbalancer. Lalu, kita juga arahkan subdomain `granz.channel.b01.com` ke IP dari worker Lawine yang memiliki IP [prefix IP].x.1.

> Di `named.conf.options`, kita tambahkan forwarders menuju IP dari google yaitu `192.168.122.1` agar dapat terhubung dengan internet

## Kendala

Tidak ada kendala dalam pengerjaan

# Soal 2, 3, 4, 5 (konfig dhcp)

## Jawaban 

### a. Kita lakukan konfig dhcp untuk subnet dengan ketentuan bahwa ketika melalui switch 3 maka mendapatkan range IP dari [prefix IP].3.16 - [prefix IP].3.32 dan [prefix IP].3.64 - [prefix IP].3.80. 

### b. Kita juga lakukan konfig dhcp untuk subnet dengan ketentuan bahwa ketika melalui switch 4 maka mendapatkan range IP dari [prefix IP].4.12 - [prefix IP].4.20 dan [prefix IP].4.160 - [prefix IP].4.168

### c. Kita juga pastikan bahwa client mendapatkan dns dari heiter dan dapat terhubung dengan internet melalui DNS tersebut

### d. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch3 selama 3 menit sedangkan pada client yang melalui Switch4 selama 12 menit. Dengan waktu maksimal dialokasikan untuk peminjaman alamat IP selama 96 menit

### Maka, kita setup dhcp server di node himmel

```
#!/bin/bash

apt-get update
apt-get install isc-dhcp-server -y

echo 'subnet 10.9.1.0 netmask 255.255.255.0 {
}

subnet 10.9.2.0 netmask 255.255.255.0 {
}

subnet 10.9.3.0 netmask 255.255.255.0 {
  range 10.9.3.16 10.9.3.32;
  range 10.9.3.64 10.9.3.80;
  option routers 10.9.3.33;
  option broadcast-address 10.9.3.255;
  option domain-name-servers 10.9.1.1;
  default-lease-time 180;
  max-lease-time 5760;
}

subnet 10.9.4.0 netmask 255.255.255.0 {
  range 10.9.4.12 10.9.4.20;
  range 10.9.4.160 10.9.4.168;
  option routers 10.9.4.33;
  option broadcast-address 10.9.4.255;
  option domain-name-servers 10.9.1.1;
  default-lease-time 720;
  max-lease-time 5760;
}' > /etc/dhcp/dhcpd.conf

echo 'INTERFACESv4="eth0"' > /etc/default/isc-dhcp-server

service isc-dhcp-server restart

service isc-dhcp-server status
```

### Setelah itu, kita lakukan konfig dhcp relay di node aura 

1. Pertama-tama kita install dulu isc-dhcp-relay 
2. Lalu kita masukkan dhcp server yang sesuai, yaitu menuju node himmel:
    <br>![1](./assets/1.jpg)</br>
3. Lalu, kita tentukan interfaces yang sesuai yaitu eth1 eth2 eth3 eth4 
    <br>![2](./assets/2.jpg)</br>
4. Hasil konfig dhcp relay kalau sudah selesai adalah:
    <br>![3](./assets/3.jpg)</br>
5. Lalu, kita setup `net.ipv4.ip_forward=1` ke `/etc/sysctl.conf` dan kita lakukan restart
    
    <br>![4](./assets/4.jpg)</br>

### Setelah itu, kita restart semua client dan worker. Lalu, kita lihat hasil dari salah satu worker jika dhcp berhasil:

<br>![5](./assets/5.jpg)</br>

### Lalu, karena ip dari worker adalah static, sedangkan dia mendapatkan ip dari dhcp, maka kita harus terapkan fixed address pada masing-masing worker dan client Richter

1. Untuk node frieren, kita tetapkan dhcp client:
    <br>![6](./assets/6.jpg)</br>

2. Untuk node flamme, kita tetapkan dhcp client:
    <br>![7](./assets/7.jpg)</br>

3. Untuk node fern, kita tetapkan dhcp client:
    <br>![8](./assets/8.jpg)</br>

4. Untuk node lawine, kita tetapkan dhcp client:
    <br>![9](./assets/9.jpg)</br>

5. Untuk node linie, kita tetapkan dhcp client:
    <br>![10](./assets/10.jpg)</br>

6. Untuk node lugner, kita tetapkan dhcp client:
    <br>![11](./assets/11.jpg)</br>

7. Untuk node richter, kita tetapkan dhcp client:
    <br>![11b](./assets/11b.jpg)</br>

### Setelah menetapkan dhcp client, kita lakukan konfig fixed address pada node himmel

```
#!/bin/bash

apt-get update 
apt-get install isc-dhcp-server -y

echo 'subnet 10.9.1.0 netmask 255.255.255.0 {
}

subnet 10.9.2.0 netmask 255.255.255.0 {
}

subnet 10.9.3.0 netmask 255.255.255.0 {
  range 10.9.3.16 10.9.3.32;
  range 10.9.3.64 10.9.3.80;
  option routers 10.9.3.33;
  option broadcast-address 10.9.3.255;
  option domain-name-servers 10.9.1.1;
  default-lease-time 180;
  max-lease-time 5760;
}

subnet 10.9.4.0 netmask 255.255.255.0 {
  range 10.9.4.12 10.9.4.20;
  range 10.9.4.160 10.9.4.168;
  option routers 10.9.4.33;
  option broadcast-address 10.9.4.255;
  option domain-name-servers 10.9.1.1;
  default-lease-time 720;
  max-lease-time 5760;
}

host Frieren{
  hardware ethernet aa:63:df:11:85:d2;
  fixed-address 10.9.4.1;
}

host Flamme{
  hardware ethernet 4a:14:12:0a:eb:e3;
  fixed-address 10.9.4.2;
}

host Fern{
  hardware ethernet ca:ed:fc:03:80:b2;
  fixed-address 10.9.4.3;
}

host Lawine{
  hardware ethernet 22:29:40:af:6d:ff;
  fixed-address 10.9.3.1;
}

host Linie{
  hardware ethernet 3a:60:9b:7d:bb:32;
  fixed-address 10.9.3.2;
}

host Lugner{
  hardware ethernet 2e:f8:c6:93:d4:b2;
  fixed-address 10.9.3.3;
}

host Richter{
  hardware ethernet d2:b4:bd:b4:c9:f1;
  fixed-address 10.9.3.69;
}' > /etc/dhcp/dhcpd.conf

service isc-dhcp-server restart

service isc-dhcp-server status
```

Lalu restart salah satu node, hasilnya adalah

<br>![12](./assets/12.jpg)</br>

## Kendala

Tidak ada kendala dalam pengerjaan

# Soal 6,7,8,9 (deploy index.php pada worker lawine, linie, lugner)

## Jawab

### a. Kita terapkan halaman granz pada masing-masing worker dengan kode:

```
#!/bin/bash

apt-get update 
cp -R /var/lib/dpkg /var/lib/dpkg.backup
rm /var/lib/dpkg/info/format
dpkg --clear-avail
dpkg --configure -a
fsck
apt-get install --reinstall perl debconf
apt install php7.3 php7.3-fpm -y
apt-get install nginx -y
apt-get install git -y
cd /var/www

git clone https://github.com/Rr-Diajeng/granz.channel.b01.com.git

echo 'server {
        listen 80 default_server;

        root /var/www/granz.channel.b01.com;

        index index.php;

        server_name _;

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
        }

        location / {
                try_files $uri $uri/ =404;
        }
}' > /etc/nginx/sites-available/granz 

ln -s /etc/nginx/sites-available/granz /etc/nginx/sites-enabled/granz

unlink /etc/nginx/sites-enabled/default

service php7.3-fpm start
service php7.3-fpm restart 
service nginx restart
```

1. Hasil deploy pada Lawine
    <br>![13](./assets/13.jpg)</br>

2. Hasil deploy pada Linie
    <br>![14](./assets/14.jpg)</br>

3. Hasil deploy pada Lugner
    <br>![15](./assets/15.jpg)</br>


### b. lalu kita setup algoritma weighted round robin load balancing agar dapat mengatur worker lawine, linie, dan lugner

1. kita setup di node eisen yang merupakan load balancer

    ```
    #!/bin/bash

    apt-get install nginx -y

    echo '
    upstream worker-granz{
    server 10.9.3.1 weight=640; #IP Lawine
    server 10.9.3.2 weight=200; #IP Linie
    server 10.9.3.3 weight=25; #IP Lugner
    }

    server {
        listen 80;
        server_name channel.b01.com;

                location / {
                        proxy_pass http://worker-granz;
                        proxy_set_header    X-Real-IP $remote_addr;
                        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                        proxy_set_header    Host $http_host;
                }

        error_log /var/log/nginx/lb_error.log;
        access_log /var/log/nginx/lb_access.log;
    }

    ' > /etc/nginx/sites-available/granz 


    ln -s /etc/nginx/sites-available/granz /etc/nginx/sites-enabled/granz
    unlink /etc/nginx/sites-enabled/granz-rr
    unlink /etc/nginx/sites-enabled/default

    service nginx restart
    service nginx restart
    ```

2. Hasil konfig tersebut dengan testing terdapat 1000 request dan 100 req/second adalah
    <br>![16](./assets/16.jpg)</br>

### c. lalu kita setup algoritma round robin load balancing agar dapat mengatur worker lawine, linie, dan lugner

di node eisen
```
echo '
upstream worker-granz-rr{
   server 10.9.3.1; #IP Lawine
   server 10.9.3.2; #IP Linie
   server 10.9.3.3; #IP Lugner
}

server {
    listen 80;
    server_name channel.b01.com;

            location / {
                    proxy_pass http://worker-granz-rr;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

    error_log /var/log/nginx/lb_error.log;
    access_log /var/log/nginx/lb_access.log;
}

' > /etc/nginx/sites-available/granz-rr


ln -s /etc/nginx/sites-available/granz-rr /etc/nginx/sites-enabled/granz-rr

unlink /etc/nginx/sites-enabled/granz

service nginx restart
service nginx restart
```

hasil uji testing dengan 200 request dan 10 req/second adalah

<br>![17](./assets/17.jpg)</br>

### d. lalu kita setup algoritma weighted round robin load balancing agar dapat mengatur worker lawine, linie, dan lugner

di node eisen
```
#!/bin/bash

apt-get install nginx -y

echo '
upstream worker-granz{
server 10.9.3.1 weight=640; #IP Lawine
server 10.9.3.2 weight=200; #IP Linie
server 10.9.3.3 weight=25; #IP Lugner
}

server {
    listen 80;
    server_name channel.b01.com;

            location / {
                    proxy_pass http://worker-granz;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

    error_log /var/log/nginx/lb_error.log;
    access_log /var/log/nginx/lb_access.log;
}

' > /etc/nginx/sites-available/granz 


ln -s /etc/nginx/sites-available/granz /etc/nginx/sites-enabled/granz
unlink /etc/nginx/sites-enabled/granz-rr
unlink /etc/nginx/sites-enabled/default

service nginx restart
service nginx restart
```

hasil uji testing dengan 200 request dan 10 req/second adalah

<br>![18](./assets/18.jpg)</br>

### e. lalu kita setup algoritma least connection load balancing agar dapat mengatur worker lawine, linie, dan lugner

di node eisen 
```
echo '
upstream worker-granz-lc{
    least_conn;
    server 10.9.3.1; #IP Lawine
    server 10.9.3.2; #IP Linie
    server 10.9.3.3; #IP Lugner
}

server {
    listen 80;
    server_name channel.b01.com;

            location / {
                    proxy_pass http://worker-granz-lc;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

    error_log /var/log/nginx/lb_error.log;
    access_log /var/log/nginx/lb_access.log;
}

' > /etc/nginx/sites-available/granz-lc


ln -s /etc/nginx/sites-available/granz-lc /etc/nginx/sites-enabled/granz-lc

unlink /etc/nginx/sites-enabled/granz

service nginx restart
service nginx restart
```

hasil uji testing dengan 200 request dan 10 req/second adalah

<br>![19](./assets/19.jpg)</br>

### f. lalu kita setup algoritma ip hash load balancing agar dapat mengatur worker lawine, linie, dan lugner

di node eisen
```
echo '
upstream worker-granz-ih{
    ip_hash;
    server 10.9.3.1; #IP Lawine
    server 10.9.3.2; #IP Linie
    server 10.9.3.3; #IP Lugner
}

server {
    listen 80;
    server_name channel.b01.com;

            location / {
                    proxy_pass http://worker-granz-ih;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

    error_log /var/log/nginx/lb_error.log;
    access_log /var/log/nginx/lb_access.log;
}

' > /etc/nginx/sites-available/granz-ih


ln -s /etc/nginx/sites-available/granz-ih /etc/nginx/sites-enabled/granz-ih

unlink /etc/nginx/sites-enabled/granz-lc

service nginx restart
service nginx restart
```

hasil uji testing dengan 200 request dan 10 req/second adalah

<br>![20](./assets/20.jpg)</br>

### g. lalu kita setup algoritma generic hash load balancing agar dapat mengatur worker lawine, linie, dan lugner

di node eisen
```
echo '
upstream worker-granz-gh{
    hash $request_uri consistent;
    server 10.9.3.1; #IP Lawine
    server 10.9.3.2; #IP Linie
    server 10.9.3.3; #IP Lugner
}

server {
    listen 80;
    server_name channel.b01.com;

            location / {
                    proxy_pass http://worker-granz-gh;
                    proxy_set_header    X-Real-IP $remote_addr;
                    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header    Host $http_host;
            }

    error_log /var/log/nginx/lb_error.log;
    access_log /var/log/nginx/lb_access.log;
}

' > /etc/nginx/sites-available/granz-gh


ln -s /etc/nginx/sites-available/granz-gh /etc/nginx/sites-enabled/granz-gh

unlink /etc/nginx/sites-enabled/granz-ih

service nginx restart
service nginx restart
```

hasil uji testing dengan 200 request dan 10 req/second adalah

<br>![21](./assets/21.jpg)</br>

### h. Dengan algoritma round robin, kita testing 100 request dan 10 req/second dengan ketentuan:

1. Untuk 3 worker:
    <br>![22](./assets/22.jpg)</br>

2. Untuk 2 worker:
    <br>![23](./assets/23.jpg)</br>

3. Untuk 1 worker:
    <br>![24](./assets/24.jpg)</br>

## Kendala

Tidak ada kendala dalam pengerjaan

# Soal 10 dan 11
Kita tambahkan konfig loadbalancer dengan kombinasi username: “netics” dan password: “ajkyyy”, dengan yyy merupakan kode kelompok. Terakhir simpan file “htpasswd” nya di /etc/nginx/rahasisakita/ dan setiap request yang mengandung /its akan mengarah ke halaman https://www.its.ac.id 

## Jawab

### Kita buat konfigurasi autentikasi pada node eisen yang merupakan load balancer

```
apt-get install apache2-utils -y

mkdir -p /etc/nginx/rahasiakita
htpasswd -c /etc/nginx/rahasiakita/.htpasswd netics

echo '
upstream worker-granz-its{
   server 10.9.3.1; #IP Lawine
   server 10.9.3.2; #IP Linie
   server 10.9.3.3; #IP Lugner
}

server {
    listen 80;
    server_name channel.b01.com;

            location / {
                proxy_pass http://worker-granz-its;
                proxy_set_header    X-Real-IP $remote_addr;
                proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header    Host $http_host;
                auth_basic "Administrators Area";
                auth_basic_user_file /etc/nginx/rahasiakita/.htpasswd;
            }

            location /its/{
                auth_basic "Administrators Area";
                auth_basic_user_file /etc/nginx/rahasiakita/.htpasswd;
                proxy_pass http://www.its.ac.id;
            }

    error_log /var/log/nginx/lbits_error.log;
    access_log /var/log/nginx/lbits_access.log;
}

' > /etc/nginx/sites-available/granz-its


ln -s /etc/nginx/sites-available/granz-its /etc/nginx/sites-enabled/granz-its

unlink /etc/nginx/sites-enabled/granz-gh

service nginx restart
service nginx restart
```

Hasil:

<br>![25](./assets/25.jpg)</br>

## Kendala

Tidak ada kendala dalam pengerjaan

# Soal no. 12

## Jawab 

### Loadbalancer yang sudah kita setup tadi hanya boleh diakses oleh client dengan IP [Prefix IP].3.69, [Prefix IP].3.70, [Prefix IP].4.167, dan [Prefix IP].4.168

di node eisen:
```
mkdir -p /etc/nginx/rahasiakita
htpasswd -c /etc/nginx/rahasiakita/.htpasswd netics

echo '
upstream worker-granz-its{
   server 10.9.3.1; #IP Lawine
   server 10.9.3.2; #IP Linie
   server 10.9.3.3; #IP Lugner
}

server {
    listen 80;
    server_name channel.b01.com;

            location / {
                allow 10.9.3.69;
                allow 10.9.3.70;
                allow 10.9.4.167;
                allow 10.9.4.168;
                deny all;
                proxy_pass http://worker-granz-its;
                proxy_set_header    X-Real-IP $remote_addr;
                proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header    Host $http_host;
                auth_basic "Administrators Area";
                auth_basic_user_file /etc/nginx/rahasiakita/.htpasswd;
            }

            location /its/{
                allow 10.9.3.69;
                allow 10.9.3.70;
                allow 10.9.4.167;
                allow 10.9.4.168;
                deny all;
                auth_basic "Administrators Area";
                auth_basic_user_file /etc/nginx/rahasiakita/.htpasswd;
                proxy_pass http://www.its.ac.id;
            }

    error_log /var/log/nginx/lbits_error.log;
    access_log /var/log/nginx/lbits_access.log;
}

' > /etc/nginx/sites-available/granz-its


ln -s /etc/nginx/sites-available/granz-its /etc/nginx/sites-enabled/granz-its

service nginx restart
service nginx restart
```

Hasil jika dibuka di ip [Prefix IP].3.69:

<br>![26](./assets/26.jpg)</br>

Hasil jika dibuka di ip yang tidak memiliki akses:

<br>![27](./assets/27.jpg)</br>

## Kendala

Tidak ada kendala dalam pengerjaan

# Soal no. 13 dan 14

Kita atur config untuk laravel pada domain canyon.b01.com. Lalu, frieren, flamme, fern memiliki riegel cannyon dan instalasi dilakukan dengan PHP8.0 dan composer

## Jawab 

### a. Kita setup database dulu di node denken

```
echo 'nameserver 192.168.122.1' > /etc/resolv.conf
apt-get update
apt-get install dialog -y && apt-get install libterm-readline-perl-perl -y
apt-get install nano -y
apt-get install iputils-ping -y
apt-get install bash -y
apt-get install iproute2 -y
apt-get install iptables -y


apt-get install mariadb-server -y
service mariadb start
```


Lalu, kita setup agar dapat diakses di semua worker

- Setup pada `/etc/mysql/my.cnf'
    ```
    [mysqld]
    skip-networking=0
    skip-bind-address

    ```
- Setup pada `/etc/mysql/mariadb.conf.d/50-server.cnf`
    ```
    bind-address            = 0.0.0.0
    ```

- Lalu `service mariadb restart`

Setelah itu, kita buat user dan database baru pada root dengan cara

```
mysql -u root -p

CREATE USER 'b01'@'%' IDENTIFIED BY 'b01';
CREATE USER 'b01'@'localhost' IDENTIFIED BY 'b01';
CREATE DATABASE db_b01;
GRANT ALL PRIVILEGES ON *.* TO 'b01'@'%';
GRANT ALL PRIVILEGES ON *.* TO 'b01'@'localhost';
FLUSH PRIVILEGES;
```

Lalu, kita uji untuk akses database melalui worker frieren:

<br>![28](./assets/28.jpg)</br>

Lalu, kita uji untuk akses database melalui worker flamme:

<br>![29](./assets/29.jpg)</br>

Lalu, kita uji untuk akses database melalui worker fern:

<br>![30](./assets/30.jpg)</br>

### b. Kita setup composer pada node frieren, flamme, dan fern lalu setup php artisan migrate ke node frieren

1. Node frieren
    - Setup kode bash seperti ini:
        ```
        apt-get update
        apt-get install -y lsb-release ca-certificates apt-transport-https software-properties-common gnupg2
        curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
        sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'

        apt-get update
        apt-get install php8.0-mbstring php8.0-xml php8.0-cli php8.0-common php8.0-intl php8.0-opcache php8.0-readline php8.0-mysql php8.0-fpm php8.0-curl unzip wget -y
        apt-get install nginx -y
        apt-get install --reinstall tzdata

        wget https://getcomposer.org/download/2.0.13/composer.phar
        chmod +x composer.phar
        mv composer.phar /usr/bin/composer

        apt-get install git -y
        cd /var/www 
        git clone https://github.com/martuafernando/laravel-praktikum-jarkom.git
        cd laravel-praktikum-jarkom
        composer install
        composer update 

        cd /var/www/laravel-praktikum-jarkom
        chmod -R 777 public
        chmod -R 777 storage

        cp .env.example .env 
        ```
    - Lalu, kita konfig env agar seperti ini:
        <br>![31](./assets/31.jpg)</br>
    
    - Lalu setup php artisan seperti ini:

        ```
        php artisan migrate:fresh
        php artisan db:seed --class=AiringsTableSeeder
        php artisan jwt:secret
        php artisan key:generate
        ```

2. Node flamme
    - Setup kode bash seperti ini:
        ```
        apt-get update
        apt-get install -y lsb-release ca-certificates apt-transport-https software-properties-common gnupg2
        curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
        sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'

        apt-get update
        apt-get install php8.0-mbstring php8.0-xml php8.0-cli php8.0-common php8.0-intl php8.0-opcache php8.0-readline php8.0-mysql php8.0-fpm php8.0-curl unzip wget -y
        apt-get install nginx -y
        apt-get install --reinstall tzdata

        wget https://getcomposer.org/download/2.0.13/composer.phar
        chmod +x composer.phar
        mv composer.phar /usr/bin/composer

        apt-get install git -y
        cd /var/www 
        git clone https://github.com/martuafernando/laravel-praktikum-jarkom.git
        cd laravel-praktikum-jarkom
        composer install
        composer update 

        cd /var/www/laravel-praktikum-jarkom
        chmod -R 777 public
        chmod -R 777 storage

        cp .env.example .env 
        ```
    - Lalu, kita konfig env agar seperti ini:
        <br>![31](./assets/31.jpg)</br>
    
    - Lalu setup php artisan seperti ini:

        ```
        php artisan jwt:secret
        php artisan key:generate
        ```

3. Node fern

    - Setup kode bash seperti ini:
        ```
        apt-get update
        apt-get install -y lsb-release ca-certificates apt-transport-https software-properties-common gnupg2
        curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
        sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'

        apt-get update
        apt-get install php8.0-mbstring php8.0-xml php8.0-cli php8.0-common php8.0-intl php8.0-opcache php8.0-readline php8.0-mysql php8.0-fpm php8.0-curl unzip wget -y
        apt-get install nginx -y
        apt-get install --reinstall tzdata

        wget https://getcomposer.org/download/2.0.13/composer.phar
        chmod +x composer.phar
        mv composer.phar /usr/bin/composer

        apt-get install git -y
        cd /var/www 
        git clone https://github.com/martuafernando/laravel-praktikum-jarkom.git
        cd laravel-praktikum-jarkom
        composer install
        composer update 

        cd /var/www/laravel-praktikum-jarkom
        chmod -R 777 public
        chmod -R 777 storage

        cp .env.example .env 
        ```
    - Lalu, kita konfig env agar seperti ini:
        <br>![31](./assets/31.jpg)</br>
    
    - Lalu setup php artisan seperti ini:

        ```
        php artisan jwt:secret
        php artisan key:generate
        ```

### b. Kita setup halaman dinamis pada node frieren, flamme, dan fern 

1. Node Frieren
    - Setup kode bash untuk halaman dinamis:

        ```
        #!/bin/bash

        echo 'server {
                listen 8001 default_server;

                root /var/www/laravel-praktikum-jarkom/public;

                index index.php;

                server_name _;

                location / {
                    try_files $uri $uri/ /index.php?$query_string;
                }

                location ~ \.php$ {
                        include snippets/fastcgi-php.conf;
                        fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
                }

                location ~ /\.ht {
                        deny all;
                }

                error_log /var/log/nginx/riegel.log;
                access_log /var/log/nginx/riegel.log;

        }' > /etc/nginx/sites-available/riegel 

        ln -s /etc/nginx/sites-available/riegel /etc/nginx/sites-enabled/riegel 

        unlink /etc/nginx/sites-enabled/default

        chown -R www-data.www-data /var/www/laravel-praktikum-jarkom/storage


        service php8.0-fpm start
        service php8.0-fpm restart 
        service nginx restart
        ```
    - Lalu kita coba uji untuk melakukan register dengan cara:
        ```curl -X POST "http://localhost:8001/api/auth/register" -d "username=b01&password=jarkomb01"```

        <br>![32](./assets/32.jpg)</br>

2. Node Flamme
    - Setup kode bash untuk halaman dinamis:

        ```
        #!/bin/bash

        echo 'server {
                listen 8002 default_server;

                root /var/www/laravel-praktikum-jarkom/public;

                index index.php;

                server_name _;

                location / {
                    try_files $uri $uri/ /index.php?$query_string;
                }

                location ~ \.php$ {
                        include snippets/fastcgi-php.conf;
                        fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
                }

                location ~ /\.ht {
                        deny all;
                }

                error_log /var/log/nginx/riegel.log;
                access_log /var/log/nginx/riegel.log;

        }' > /etc/nginx/sites-available/riegel 

        ln -s /etc/nginx/sites-available/riegel /etc/nginx/sites-enabled/riegel 

        unlink /etc/nginx/sites-enabled/default

        chown -R www-data.www-data /var/www/laravel-praktikum-jarkom/storage

        service php8.0-fpm start
        service php8.0-fpm restart 
        service nginx restart
        ```
    
    - Lalu kita coba uji untuk melakukan register dengan cara:
        ```curl -X POST "http://localhost:8002/api/auth/register" -d "username=b001&password=jarkomb001"```

        <br>![33](./assets/33.jpg)</br>

3. Node Fern
    - Setup kode bash untuk halaman dinamis:

        ```
        !/bin/bash

        echo 'server {
                listen 8003 default_server;

                root /var/www/laravel-praktikum-jarkom/public;

                index index.php;

                server_name _;

                location / {
                    try_files $uri $uri/ /index.php?$query_string;
                }

                location ~ \.php$ {
                        include snippets/fastcgi-php.conf;
                        fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
                }

                location ~ /\.ht {
                        deny all;
                }

                error_log /var/log/nginx/riegel.log;
                access_log /var/log/nginx/riegel.log;

        }' > /etc/nginx/sites-available/riegel 

        ln -s /etc/nginx/sites-available/riegel /etc/nginx/sites-enabled/riegel 

        unlink /etc/nginx/sites-enabled/default

        chown -R www-data.www-data /var/www/laravel-praktikum-jarkom/storage

        service php8.0-fpm start
        service php8.0-fpm restart 
        service nginx restart
        ```

    - Lalu kita coba uji untuk melakukan register dengan cara:
        ```curl -X POST "http://localhost:8003/api/auth/register" -d "username=b0001&password=jarkomb0001"```

        <br>![34](./assets/34.jpg)</br>

Hasil database yang terbentuk setelah kita melakukan register:

<br>![35](./assets/35.jpg)</br>

4. Lalu, kita setup loadbalancing untuk laravel pada node eisen
    ```
    #!/bin/bash

    echo 'upstream worker-riegel {
        server 10.9.4.1:8001; #IP Frieren 
        server 10.9.4.2:8002; #IP Flamme
        server 10.9.4.3:8003; #IP Fern
    }

    server {
        listen 80 default_server;
        server_name canyon.b01.com;

        location / {
            proxy_pass http://worker-riegel;
            proxy_set_header    X-Real-IP $remote_addr;
            proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header    Host $http_host;
        }

        error_log /var/log/nginx/lb_error.log;
        access_log /var/log/nginx/lb_access.log;
    }' > /etc/nginx/sites-available/riegel

    ln -s /etc/nginx/sites-available/riegel /etc/nginx/sites-enabled/riegel

    service nginx restart
    service nginx restart
    ```

## Kendala

Tidak ada kendala dalam pengerjaan
# Soal no. 15, 16, dan 17

## Jawab

Pertama-tama, kita buat file `loginregister.json` pada client

```
{
  "username": "yuan",
  "password": "donghae"
}
```

Kita lakukan testing sebanyak 100 request dengan 10 req/second pada 

1. POST /auth/register
    ```
    ab -n 100 -c 10 -p ~/loginregister.json -T application/json http://canyon.b01.com/api/auth/register/
    ```

    Hasilnya adalah
    
    <br>![36](./assets/36.jpg)</br>


2. POST /auth/login

    ```
    ab -n 100 -c 10 -p ~/loginregister.json -T application/json http://canyon.b01.com/api/auth/login/
    ```

    Hasilnya adalah
    
    <br>![37](./assets/37.jpg)</br>

3. GET /me
    ```
    ab -n 100 -c 10 -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vY2FueW9uLmIwMS5jb20vYXBpL2F1dGgvbG9naW4iLCJpYXQiOjE3MDAzNzkxNDksImV4cCI6MTcwMDM4Mjc0OSwibmJmIjoxNzAwMzc5MTQ5LCJqdGkiOiJXamF6b2JTUXczcDJUdGV2Iiwic3ViIjoiNCIsInBydiI6IjIzYmQ1Yzg5NDlmNjAwYWRiMzllNzAxYzQwMDg3MmRiN2E1OTc2ZjcifQ.1lU4VSGYLDn5EY_P8LAkoEM6Xc4hmpgYQvL-KurQ4gY" -r -k "http://canyon.b01.com/api/me"
    ```

    Hasilnya adalah
    
    <br>![38](./assets/38.jpg)</br>


## Kendala

Tidak ada kendala dalam pengerjaan

# Soal no 18

Implementasikan proxy bind pada eisen untuk mengaitkan IP dari frieren, flamme, dan fern

## Jawab

### Kita atur proxy bind pada loadbalancing di node eisen
- Kode bash:

    ```
    #!/bin/bash

    echo 'upstream worker-riegel-proxy {
        server 10.9.4.1:8001; #IP Frieren 
        server 10.9.4.2:8002; #IP Flamme
        server 10.9.4.3:8003; #IP Fern
    }

    server {
        listen 80 default_server;
        server_name canyon.b01.com;

        location / {
            proxy_pass http://worker-riegel-proxy;
        }

        location /frieren/{
            proxy_bind 10.9.2.1;
            proxy_pass http://10.9.4.1:8001/index.php;
        }

        location /flamme/{
            proxy_bind 10.9.2.1;
            proxy_pass http://10.9.4.2:8002/index.php;
        }

        location /fern/{
            proxy_bind 10.9.2.1;
            proxy_pass http://10.9.4.3:8003/index.php;
        }

        error_log /var/log/nginx/lb_error.log;
        access_log /var/log/nginx/lb_access.log;
    }' > /etc/nginx/sites-available/riegel-proxy

    ln -s /etc/nginx/sites-available/riegel-proxy /etc/nginx/sites-enabled/riegel-proxy

    unlink /etc/nginx/sites-enabled/riegel

    service nginx restart
    service nginx restart
    ```

## Kendala

Tidak ada kendala dalam pengerjaan

# Soal no 19

Untuk meningkatkan performa dari Worker, coba implementasikan PHP-FPM pada Frieren, Flamme, dan Fern. Untuk testing kinerja naikkan 
- pm.max_children
- pm.start_servers
- pm.min_spare_servers
- pm.max_spare_servers
sebanyak tiga percobaan dan lakukan testing sebanyak 100 request dengan 10 request/second kemudian berikan hasil analisisnya pada Grimoire

## Jawab

### Percobaan pertama

- Kita tetapkan:  
    pm = dynamic  
    pm.max_children = 40  
    pm.start_servers = 10  
    pm.min_spare_servers = 5  
    pm.max_spare_servers = 30  
    pm.process_idle_timeout = 1000s  

1. Setup di node frieren

    ```
    #!/bin/bash

    echo '[frieren_site]
    user = frieren_user
    group = frieren_user
    listen = /var/run/php/php8.0-fpm-frieren-site.sock
    listen.owner = www-data
    listen.group = www-data
    php_admin_value[disable_functions] = exec,passthru,shell_exec,system
    php_admin_flag[allow_url_fopen] = off

    ; Choose how the process manager will control the number of child processes.

    pm = dynamic
    pm.max_children = 40
    pm.start_servers = 10
    pm.min_spare_servers = 5
    pm.max_spare_servers = 30
    pm.process_idle_timeout = 1000s' > /etc/php/8.0/fpm/pool.d/frieren_site.conf


    groupadd frieren_user
    useradd -g frieren_user frieren_user

    service php8.0-fpm restart

    echo 'server {
            listen 8001;

            root /var/www/laravel-praktikum-jarkom/public;

            index index.php;

            server_name _;

            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }

            location ~ \.php$ {
                    include snippets/fastcgi-php.conf;
                    fastcgi_pass unix:/var/run/php/php8.0-fpm-frieren-site.sock;
            }

            location ~ /\.ht {
                    deny all;
            }

            error_log /var/log/nginx/riegel.log;
            access_log /var/log/nginx/riegel.log;

    }' > /etc/nginx/sites-available/riegel

    service php8.0-fpm restart
    service nginx restart
    ```

2. Setup di node flamme

    ```
    #!/bin/bash

    echo '[flamme_site]
    user = flamme_user
    group = flamme_user
    listen = /var/run/php/php8.0-fpm-flamme-site.sock
    listen.owner = www-data
    listen.group = www-data
    php_admin_value[disable_functions] = exec,passthru,shell_exec,system
    php_admin_flag[allow_url_fopen] = off

    ; Choose how the process manager will control the number of child processes.

    pm = dynamic
    pm.max_children = 40
    pm.start_servers = 10
    pm.min_spare_servers = 5
    pm.max_spare_servers = 30
    pm.process_idle_timeout = 1000s' > /etc/php/8.0/fpm/pool.d/flamme_site.conf


    groupadd flamme_user
    useradd -g flamme_user flamme_user

    service php8.0-fpm restart

    echo 'server {
            listen 8002 default_server;

            root /var/www/laravel-praktikum-jarkom/public;

            index index.php;

            server_name _;

            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }

            location ~ \.php$ {
                    include snippets/fastcgi-php.conf;
                    fastcgi_pass unix:/var/run/php/php8.0-fpm-flamme-site.sock;
            }

            location ~ /\.ht {
                    deny all;
            }

            error_log /var/log/nginx/riegel.log;
            access_log /var/log/nginx/riegel.log;

    }' > /etc/nginx/sites-available/riegel

    service nginx restart
    ```

3. Setup di node fern

    ```
    #!/bin/bash

    echo '[fern_site]
    user = fern_user
    group = fern_user
    listen = /var/run/php/php8.0-fpm-fern-site.sock
    listen.owner = www-data
    listen.group = www-data
    php_admin_value[disable_functions] = exec,passthru,shell_exec,system
    php_admin_flag[allow_url_fopen] = off

    ; Choose how the process manager will control the number of child processes.

    pm = dynamic
    pm.max_children = 40
    pm.start_servers = 10
    pm.min_spare_servers = 5
    pm.max_spare_servers = 30
    pm.process_idle_timeout = 1000s' > /etc/php/8.0/fpm/pool.d/fern_site.conf


    groupadd fern_user
    useradd -g fern_user fern_user

    service php8.0-fpm restart

    echo 'server {
            listen 8003 default_server;

            root /var/www/laravel-praktikum-jarkom/public;

            index index.php;

            server_name _;

            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }

            location ~ \.php$ {
                    include snippets/fastcgi-php.conf;
                    fastcgi_pass unix:/var/run/php/php8.0-fpm-fern-site.sock;
            }

            location ~ /\.ht {
                    deny all;
            }

            error_log /var/log/nginx/riegel.log;
            access_log /var/log/nginx/riegel.log;

    }' > /etc/nginx/sites-available/riegel 

    service nginx restart
    ```

4. Hasil ab 
    ```
    ab -n 100 -c 10 http://canyon.b01.com/
    ```
    <br>![39](./assets/39.jpg)</br>


### Percobaan kedua

- Kita tetapkan:  
    pm = dynamic  
    pm.max_children = 35
    pm.start_servers = 5  
    pm.min_spare_servers = 3  
    pm.max_spare_servers = 10  
    pm.process_idle_timeout = 100s  

1. Setup di node frieren

    ```
    #!/bin/bash

    echo '[frieren_site]
    user = frieren_user
    group = frieren_user
    listen = /var/run/php/php8.0-fpm-frieren-site.sock
    listen.owner = www-data
    listen.group = www-data
    php_admin_value[disable_functions] = exec,passthru,shell_exec,system
    php_admin_flag[allow_url_fopen] = off

    ; Choose how the process manager will control the number of child processes.

    pm = dynamic
    pm.max_children = 35
    pm.start_servers = 5
    pm.min_spare_servers = 3
    pm.max_spare_servers = 10
    pm.process_idle_timeout = 100s' > /etc/php/8.0/fpm/pool.d/frieren_site.conf


    groupadd frieren_user
    useradd -g frieren_user frieren_user

    service php8.0-fpm restart

    echo 'server {
            listen 8001 default_server;

            root /var/www/laravel-praktikum-jarkom/public;

            index index.php;

            server_name _;

            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }

            location ~ \.php$ {
                    include snippets/fastcgi-php.conf;
                    fastcgi_pass unix:/var/run/php/php8.0-fpm-frieren-site.sock;
            }

            location ~ /\.ht {
                    deny all;
            }

            error_log /var/log/nginx/riegel.log;
            access_log /var/log/nginx/riegel.log;

    }' > /etc/nginx/sites-available/riegel

    service nginx restart
    ```

2. Setup di node flamme

    ```
    #!/bin/bash

    echo '[flamme_site]
    user = flamme_user
    group = flamme_user
    listen = /var/run/php/php8.0-fpm-flamme-site.sock
    listen.owner = www-data
    listen.group = www-data
    php_admin_value[disable_functions] = exec,passthru,shell_exec,system
    php_admin_flag[allow_url_fopen] = off

    ; Choose how the process manager will control the number of child processes.

    pm = dynamic
    pm.max_children = 35
    pm.start_servers = 5
    pm.min_spare_servers = 3
    pm.max_spare_servers = 10
    pm.process_idle_timeout = 100s' > /etc/php/8.0/fpm/pool.d/flamme_site.conf


    groupadd flamme_user
    useradd -g flamme_user flamme_user

    service php8.0-fpm restart

    echo 'server {
            listen 8002 default_server;

            root /var/www/laravel-praktikum-jarkom/public;

            index index.php;

            server_name _;

            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }

            location ~ \.php$ {
                    include snippets/fastcgi-php.conf;
                    fastcgi_pass unix:/var/run/php/php8.0-fpm-flamme-site.sock;
            }

            location ~ /\.ht {
                    deny all;
            }

            error_log /var/log/nginx/riegel.log;
            access_log /var/log/nginx/riegel.log;

    }' > /etc/nginx/sites-available/riegel

    service nginx restart
    ```
3. Setup di node fern

    ```
    #!/bin/bash

    echo '[fern_site]
    user = fern_user
    group = fern_user
    listen = /var/run/php/php8.0-fpm-fern-site.sock
    listen.owner = www-data
    listen.group = www-data
    php_admin_value[disable_functions] = exec,passthru,shell_exec,system
    php_admin_flag[allow_url_fopen] = off

    ; Choose how the process manager will control the number of child processes.

    pm = dynamic
    pm.max_children = 35
    pm.start_servers = 5
    pm.min_spare_servers = 3
    pm.max_spare_servers = 10
    pm.process_idle_timeout = 100s' > /etc/php/8.0/fpm/pool.d/fern_site.conf


    groupadd fern_user
    useradd -g fern_user fern_user

    service php8.0-fpm restart

    echo 'server {
            listen 8003 default_server;

            root /var/www/laravel-praktikum-jarkom/public;

            index index.php;

            server_name _;

            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }

            location ~ \.php$ {
                    include snippets/fastcgi-php.conf;
                    fastcgi_pass unix:/var/run/php/php8.0-fpm-fern-site.sock;
            }

            location ~ /\.ht {
                    deny all;
            }

            error_log /var/log/nginx/riegel.log;
            access_log /var/log/nginx/riegel.log;

    }' > /etc/nginx/sites-available/riegel 

    service nginx restart
    ```

4. Hasil ab 
    ```
    ab -n 100 -c 10 http://canyon.b01.com/
    ```
    <br>![40](./assets/40.jpg)</br>


### Percobaan ketiga

- Kita tetapkan:  
    pm = dynamic  
    pm.max_children = 75  
    pm.start_servers = 10  
    pm.min_spare_servers = 5  
    pm.max_spare_servers = 20  
    pm.process_idle_timeout = 100s    

1. Setup di node frieren

    ```
    #!/bin/bash

    echo '[frieren_site]
    user = frieren_user
    group = frieren_user
    listen = /var/run/php/php8.0-fpm-frieren-site.sock
    listen.owner = www-data
    listen.group = www-data
    php_admin_value[disable_functions] = exec,passthru,shell_exec,system
    php_admin_flag[allow_url_fopen] = off

    ; Choose how the process manager will control the number of child processes.

    pm = dynamic
    pm.max_children = 75
    pm.start_servers = 10
    pm.min_spare_servers = 5
    pm.max_spare_servers = 20
    pm.process_idle_timeout = 100s' > /etc/php/8.0/fpm/pool.d/frieren_site.conf


    groupadd frieren_user
    useradd -g frieren_user frieren_user

    service php8.0-fpm restart

    echo 'server {
            listen 8001 default_server;

            root /var/www/laravel-praktikum-jarkom/public;

            index index.php;

            server_name _;

            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }

            location ~ \.php$ {
                    include snippets/fastcgi-php.conf;
                    fastcgi_pass unix:/var/run/php/php8.0-fpm-frieren-site.sock;
            }

            location ~ /\.ht {
                    deny all;
            }

            error_log /var/log/nginx/riegel.log;
            access_log /var/log/nginx/riegel.log;

    }' > /etc/nginx/sites-available/riegel

    service nginx restart
    ```

2. Setup di node flamme

    ```
    #!/bin/bash

    echo '[flamme_site]
    user = flamme_user
    group = flamme_user
    listen = /var/run/php/php8.0-fpm-flamme-site.sock
    listen.owner = www-data
    listen.group = www-data
    php_admin_value[disable_functions] = exec,passthru,shell_exec,system
    php_admin_flag[allow_url_fopen] = off

    ; Choose how the process manager will control the number of child processes.

    pm = dynamic
    pm.max_children = 75
    pm.start_servers = 10
    pm.min_spare_servers = 5
    pm.max_spare_servers = 50
    pm.process_idle_timeout = 100s' > /etc/php/8.0/fpm/pool.d/flamme_site.conf


    groupadd flamme_user
    useradd -g flamme_user flamme_user

    service php8.0-fpm restart

    echo 'server {
            listen 8002 default_server;

            root /var/www/laravel-praktikum-jarkom/public;

            index index.php;

            server_name _;

            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }

            location ~ \.php$ {
                    include snippets/fastcgi-php.conf;
                    fastcgi_pass unix:/var/run/php/php8.0-fpm-flamme-site.sock;
            }

            location ~ /\.ht {
                    deny all;
            }

            error_log /var/log/nginx/riegel.log;
            access_log /var/log/nginx/riegel.log;

    }' > /etc/nginx/sites-available/riegel

    service nginx restart
    ```

3. Setup di node fern

    ```
    #!/bin/bash

    echo '[fern_site]
    user = fern_user
    group = fern_user
    listen = /var/run/php/php8.0-fpm-fern-site.sock
    listen.owner = www-data
    listen.group = www-data
    php_admin_value[disable_functions] = exec,passthru,shell_exec,system
    php_admin_flag[allow_url_fopen] = off

    ; Choose how the process manager will control the number of child processes.

    pm = dynamic
    pm.max_children = 75
    pm.start_servers = 10
    pm.min_spare_servers = 5
    pm.max_spare_servers = 20
    pm.process_idle_timeout = 100s' > /etc/php/8.0/fpm/pool.d/fern_site.conf


    groupadd fern_user
    useradd -g fern_user fern_user

    service php8.0-fpm restart

    echo 'server {
        listen 8003 default_server;

        root /var/www/laravel-praktikum-jarkom/public;

        index index.php;

        server_name _;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php8.0-fpm-fern-site.sock;
        }

        location ~ /\.ht {
                deny all;
        }

        error_log /var/log/nginx/riegel.log;
        access_log /var/log/nginx/riegel.log;

    }' > /etc/nginx/sites-available/riegel 

    service nginx restart
    ```

4. Hasil ab 
    ```
    ab -n 100 -c 10 http://canyon.b01.com/
    ```
    <br>![41](./assets/41.jpg)</br>


## Kendala

Tidak ada kendala dalam pengerjaan

# Soal no 20

Nampaknya hanya menggunakan PHP-FPM tidak cukup untuk meningkatkan performa dari worker maka implementasikan Least-Conn pada Eisen. Untuk testing kinerja dari worker tersebut dilakukan sebanyak 100 request dengan 10 request/second.

## Jawab

### Tambahkan algoritma least connnection loadbalancer pada node eisen

Dengan masih menggunakan ketentuan phpfpm yaitu:

pm = dynamic  
pm.max_children = 75  
pm.start_servers = 10  
pm.min_spare_servers = 5  
pm.max_spare_servers = 20  
pm.process_idle_timeout = 100s  

Di node eisen:
```
echo 'upstream worker-riegel-proxy {
    least_conn;
    server 10.9.4.1:8001; #IP Frieren 
    server 10.9.4.2:8002; #IP Flamme
    server 10.9.4.3:8003; #IP Fern
}

server {
    listen 80 default_server;
    server_name canyon.b01.com;

    location / {
        proxy_pass http://worker-riegel-proxy;
    }

    location /frieren/{
        proxy_bind 10.9.2.1;
        proxy_pass http://10.9.4.1:8001/index.php;
    }

    location /flamme/{
        proxy_bind 10.9.2.1;
        proxy_pass http://10.9.4.2:8002/index.php;
    }

    location /fern/{
        proxy_bind 10.9.2.1;
        proxy_pass http://10.9.4.3:8003/index.php;
    }

    error_log /var/log/nginx/lb_error.log;
    access_log /var/log/nginx/lb_access.log;
}' > /etc/nginx/sites-available/riegel-lc-proxy

ln -s /etc/nginx/sites-available/riegel-lc-proxy /etc/nginx/sites-enabled/riegel-lc-proxy

unlink /etc/nginx/sites-enabled/riegel-proxy

service nginx restart
service nginx restart
```

Hasil ab:
```
ab -n 100 -c 10 -g lcphpfpm.data http://canyon.b01.com/
```

<br>![42](./assets/42.jpg)</br>

## Kendala

Tidak ada kendala dalam pengerjaan
