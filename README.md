# Modul 1 Jarkom IT14
* Moh. Sulthan Arief Rahmatullah
* Fransiskus Benyamin

## No 1
Yudhistira akan digunakan sebagai DNS Master, Werkudara sebagai DNS Slave, Arjuna merupakan Load Balancer yang terdiri dari beberapa Web Server yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Buatlah topologi dengan pembagian sebagai berikut. Folder topologi dapat diakses pada drive berikut 

[![ggg.png](https://i.postimg.cc/k4WFDBF8/ggg.png)](https://postimg.cc/FdsJBF6s)
[![pandu.png](https://i.postimg.cc/jd2wFDwX/pandu.png)](https://postimg.cc/fJ4R3Tvt)
[![yudhis.png](https://i.postimg.cc/KjzzwWzz/yudhis.png)](https://postimg.cc/jwptJZy0)
[![Werkudara.png](https://i.postimg.cc/qM6BKy69/Werkudara.png)](https://postimg.cc/7fDy8CVV)
[![arjuna.png](https://i.postimg.cc/nLpnNWSJ/arjuna.png)](https://postimg.cc/5XKcCsM7)
[![Sadewa.png](https://i.postimg.cc/7YzDh4p4/Sadewa.png)](https://postimg.cc/gnG1tCzt)
[![nakula.png](https://i.postimg.cc/gJrbWbvf/nakula.png)](https://postimg.cc/BLWVTy7g)
[![prabu.png](https://i.postimg.cc/wBjj7PXk/prabu.png)](https://postimg.cc/pm3HSs19)
[![Abimanyu.png](https://i.postimg.cc/J4C7bt30/Abimanyu.png)](https://postimg.cc/Z0jhmbMz)
[![wisageni.png](https://i.postimg.cc/rw88PbPL/wisageni.png)](https://postimg.cc/dLxP7WFW)

## Menyelesaikan DNS dulu dari no2-8
Karena soal no2-8 berkaitan dengan DNS bind9 maka saya buat script untuk me setup dulu semuanya nama scriptnya `dns_master.sh`

```
#!/bin/bash

apt install -y bind9 bind9utils bind9-doc dnsutils
rm -f /var/cache/bind/*
# Konfigurasi di Server Yudhistira

echo "Memperbaiki konfigurasi DNS untuk Yudhistira..."

cat > /etc/bind/named.conf.local <<EOL
zone "it14.com" {
    type master;
    file "/etc/bind/db.it14";
    allow-transfer { 192.240.1.5; };
};

zone "3.240.192.in-addr.arpa" {
    type master;
    file "/etc/bind/3.240.192.in-addr.arpa";
};
EOL

cat > /etc/bind/db.it14 <<EOL
\$TTL    604800
@       IN      SOA     it14.com. admin.it14.com. (
                              3         
                         604800       
                          86400        
                        2419200         
                         604800 )       
@               IN      NS      Yudhistira.it14.com.
@               IN      NS      Werkudara.it14.com.
Yudhistira      IN      A       192.240.1.4
Werkudara       IN      A       192.240.1.5
arjuna          IN      A       192.240.2.2
www.arjuna      IN      A       192.240.2.2
abimanyu        IN      A       192.240.3.3
www.abimanyu    IN      A       192.240.3.3
parikesit.abimanyu  IN  A       192.240.3.3
www.parikesit.abimanyu IN A     192.240.3.3
baratayuda.abimanyu     IN      NS      Werkudara.it14.com.
EOL

# Konfigurasi zona terbalik
cat > /etc/bind/3.240.192.in-addr.arpa <<EOL
; BIND reverse data file for broadcast zone
\$TTL    604800
@       IN      SOA     abimanyu.it14.com. root.abimanyu.it14.com. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      abimanyu.it14.com.
3.240.192.in-addr.arpa          IN      NS      abimanyu.it14.com.
4               IN      PTR     abimanyu.it14.com.
EOL

# Restart BIND9
/etc/init.d/bind9 restart

echo "Perbaikan konfigurasi DNS untuk Yudhistira selesai."
```


* `cat > /etc/bind/named.conf.local <<EOL ... EOL:` Ini adalah perintah untuk membuat atau mengedit file konfigurasi DNS lokal yang disebut "named.conf.local". Dalam file ini, dua zona DNS didefinisikan: "it14.com" dan "3.240.192.in-addr.arpa". Zona "it14.com" adalah zona master, sedangkan zona "3.240.192.in-addr.arpa" adalah zona reverse untuk mencocokkan alamat IP dengan nama domain.
* `cat > /etc/bind/db.it14 <<EOL ... EOL:` Ini adalah perintah untuk membuat atau mengedit file zona DNS yang disebut "db.it14". Dalam file ini, informasi tentang nama-nama domain dan alamat IP mereka didefinisikan. Ini mencakup definisi NS (Name Server) dan record A (alamat IP) untuk beberapa nama domain, seperti "Yudhistira.it14.com", "Werkudara.it14.com", dan lainnya.
* `cat > /etc/bind/3.240.192.in-addr.arpa <<EOL ... EOL:` Ini adalah perintah untuk membuat atau mengedit file zona reverse DNS yang disebut "3.240.192.in-addr.arpa". File ini digunakan untuk mencocokkan alamat IP dengan nama domain. Dalam file ini, ada definisi PTR (Pointer) yang menghubungkan alamat IP "192.240.1.4" dengan nama domain "abimanyu.it14.com".
*  skrip ini juga mungkin telah mengonfigurasi server DNS untuk mengizinkan transfer zona dari alamat IP "192.240.1.5".


## Web server No2 dan 3
Karena nomer 2 dan 3 sama sama deploy maka script nya sama

```
#!/bin/bash

echo -e "\nnameserver 192.240.1.4\nnameserver 192.240.1.5\nnameserver 192.168.122.1" > /etc/resolv.conf

# Instal paket yang diperlukan
apt-get update
apt-get install -y software-properties-common unzip
# add-apt-repository ppa:ondrej/php
apt-get install -y nginx

# Instal PHP
apt-get install -y php7.0 php7.0-fpm php7.0-cli php7.0-common php7.0-json php7.0-opcache php7.0-mysql php7.0-mbstring php7.0-mcrypt php7.0-zip

service php7.0-fpm start
service php7.0-fpm restart
# Tautan Google Drive (tanpa perlu konfirmasi)
google_drive_link="https://drive.usercontent.google.com/uc?id=17tAM_XDKYWDvF-JJix1x7txvTBEax7vX&export=download"

# Nama file yang akan disimpan
output_file="arjuna.zip"

# Unduh file dari Google Drive
curl -L -o "$output_file" "$google_drive_link"

# Mengekstrak file (jika perlu)
unzip "$output_file"

# Ganti nama direktori arjuna.yyy.com menjadi arjuna.it14.com
mv arjuna.yyy.com arjuna.it14.com

# Pindahkan direktori ke /var/www
mv arjuna.it14.com /var/www/html

# Buat file konfigurasi arjuna.it14.com
echo 'server {
    listen 80;
    listen [::]:80;

    root /var/www/html/arjuna.it14.com/;
    index index.php index.html index.htm;

    server_name arjuna.it14.com www.arjuna.it14.com;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
' > /etc/nginx/sites-available/default

# rm /etc/nginx/sites-enabled/default
# ln -s /etc/nginx/sites-available/arjuna.it14.com /etc/nginx/sites-enabled/

# Restart Nginx
service nginx restart
# Mengecek konfigurasi Nginx
nginx -t

```

Konfigurasi Nginx yang diberikan mengatur server web untuk mendengarkan permintaan HTTP di port 80. Akar situs webnya terletak di direktori `/var/www/html/arjuna.it14.com/` dan akan mencoba file indeks "index.php", "index.html", atau "index.htm" jika ada. Server ini akan menangani permintaan untuk nama domain "arjuna.it14.com" dan "www.arjuna.it14.com". Selain itu, konfigurasi ini mengarahkan permintaan yang berakhir dengan ekstensi ".php" ke server PHP-FPM menggunakan soket UNIX `/var/run/php/php7.0-fpm.sock`. Konfigurasi juga melarang akses ke file ".ht" yang tersembunyi.

lakukan juga kepada abimanyu dengan mengubah server name yang sesuai.

setelah setup no2-8 lakukan pengetesan terhadap semua domain dengan cara ping, dan outputnya akan mengarah ke IP yang sesuai. dan lakukan curl juga untuk arjua dan abimanyu agar daapat  mengecek websitenya berhasil di deploy atau tidak.

## no 9 - 10
Pada nomer ini kita diminta untuk men deploy yang berasal dari resource arjuna ke semua worker
Berikut script untuk workernya

```
echo -e "\nnameserver 192.240.1.4\nnameserver 192.240.1.5\nnameserver 192.168.122.1" > /etc/resolv.conf

# Instal paket yang diperlukan
apt-get update
apt-get install -y software-properties-common unzip
# add-apt-repository ppa:ondrej/php
 apt-get install -y nginx

# Instal PHP
apt-get install -y php7.0 php7.0-fpm php7.0-cli php7.0-common php7.0-json php7.0-opcache php7.0-mysql php7.0-mbstring php7.0-mcrypt php7.0-zip

service php7.0-fpm start
service php7.0-fpm restart
# Tautan Google Drive (tanpa perlu konfirmasi)
google_drive_link="https://drive.usercontent.google.com/uc?id=17tAM_XDKYWDvF-JJix1x7txvTBEax7vX&export=download"

# Nama file yang akan disimpan
output_file="arjuna.zip"

# Unduh file dari Google Drive
curl -L -o "$output_file" "$google_drive_link"

# Mengekstrak file (jika perlu)
unzip "$output_file"

# Ganti nama direktori arjuna.yyy.com menjadi arjuna.it14.com
mv arjuna.yyy.com arjuna.it14.com

# Pindahkan direktori ke /var/www
mv arjuna.it14.com /var/www/html


# Buat konfigurasi Nginx untuk Prabakusuma
echo 'server {
    listen 8002;
    root /var/www/html/arjuna.it14.com/;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
' > /etc/nginx/sites-available/default

# Restart Nginx
service nginx restart

# Mengecek konfigurasi Nginx
nginx -t

```
Konfigurasi Nginx yang diberikan adalah untuk server dengan nama domain "Prabakusuma.it14.com" yang akan mendengarkan permintaan di port 8002. Direktori root situs webnya berada di "/var/www/html/arjuna.it14.com/", dan server akan mencoba file indeks seperti "index.php", "index.html", atau "index.htm" jika tersedia. Selain itu, konfigurasi ini mengatur server untuk mengarahkan permintaan ke file PHP jika ditemukan, menggunakan soket UNIX "/var/run/php/php7.0-fpm.sock". Seluruh konfigurasi ini ditempatkan dalam file konfigurasi default Nginx di direktori "/etc/nginx/sites-available/default".

Untuk worker yang lainnya konfigurasinya sama hanya saja beda port sesuai soalnya Prabakusuma:8001 Abimanyu:8002 Wisanggeni:8003

dan untuk load balancernya yaitu arjuna

```
#!/bin/bash

echo -e "\nnameserver 192.240.1.4\nnameserver 192.240.1.5\nnameserver 192.168.122.1" > /etc/resolv.conf

# Instal paket yang diperlukan
apt-get update
apt-get install -y software-properties-common unzip
# add-apt-repository ppa:ondrej/php
apt-get install -y nginx

# Instal PHP
apt-get install -y php7.0 php7.0-fpm php7.0-cli php7.0-common php7.0-json php7.0-opcache php7.0-mysql php7.0-mbstring php7.0-mcrypt php7.0-zip

service php7.0-fpm start
service php7.0-fpm restart

# Buat konfigurasi Load Balancer dengan Nginx untuk Arjuna
echo 'upstream backend_servers {
    server 192.240.3.2:8001; # Prabakusuma
    server 192.240.3.3:8002; # Abimanyu
    server 192.240.3.4:8003; # Wisanggeni
}

server {
    listen 80;
    listen [::]:80;

    server_name arjuna.it14.com www.arjuna.it14.com;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location ~ /\.ht {
        deny all;
    }
}
' > /etc/nginx/sites-available/default

# Restart Nginx
service nginx restart

# Mengecek konfigurasi Nginx
nginx -t

```

Konfigurasi Nginx yang diberikan adalah untuk membuat load balancer yang akan mengarahkan permintaan ke tiga server backend dengan alamat IP yang berbeda (192.240.3.2, 192.240.3.3, dan 192.240.3.4) pada port masing-masing (8001, 8002, dan 8003). Load balancer ini akan mendengarkan permintaan di port 80 dan akan menangani permintaan untuk nama domain "arjuna.it14.com" dan "www.arjuna.it14.com". Setiap permintaan akan diarahkan ke salah satu server backend dalam grup "backend_servers" menggunakan metode load balancing yang sesuai (biasanya round-robin). Konfigurasi ini juga melampirkan header seperti "Host," "X-Real-IP," dan "X-Forwarded-For" saat meneruskan permintaan ke server backend. Selain itu, file konfigurasi ini disimpan dalam file konfigurasi default Nginx di direktori "/etc/nginx/sites-available/default".

Setelah itu lakukan test dengan me `curl arjuna.it14.com` maka respon yang didapatkan akan berbeda beda dari 3 workernya


## no 11-17
Untuk no 11-17 semua setupnya ada di abimanyu jadi di satukan untuk scriptnya

```
#!/bin/bash

# Update repositori
apt-get update

# Instalasi Apache
apt-get install -y apache2

# Instalasi modul PHP untuk Apache
apt-get install -y libapache2-mod-php7.0 unzip curl

apt-get install -y apache2-utils



# Buat direktori untuk situs web Abimanyu
mkdir -p /var/www/abimanyu.it14
mkdir /var/www/error
echo "<h1>404 Not Found</h1><p>INI NOUTFOUND WOI.</p>" > /var/www/error/404.html
echo "<h1>403 Forbidden</h1><p>INI FORBIDDENNNNNNN.</p>" > /var/www/error/403.html

google_drive_link="https://drive.usercontent.google.com/download?id=1a4V23hwK9S7hQEDEcv9FL14UkkrHc-Zc&export=download&authuser=0&confirm=t&uuid=a0a1d5d3-1b88-41f3-a805-57dc7883a1f0&at=APZUnTVJ1TucA9LeM7rwt43EgR2J:1697073963527"

# Nama file yang akan disimpan
output_file="abimanyu.zip"

# Unduh file dari Google Drive
curl -L -o "$output_file" "$google_drive_link"

# Mengekstrak file (jika perlu)
unzip "$output_file"
mv abimanyu.yyy.com/* /var/www/abimanyu.it14

# Buat file konfigurasi virtual host untuk Abimanyu
echo "
<VirtualHost *:80>
    ServerAdmin webmaster@abimanyu.it14.com
    ServerName abimanyu.it14.com
    ServerAlias www.abimanyu.it14.com
    DocumentRoot /var/www/abimanyu.it14
    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined
    ErrorDocument 404 /error/404.html
    ErrorDocument 403 /error/403.html

    <Directory /var/www/abimanyu.it14>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    <Directory /var/www/error>
        Options -Indexes
        Require all granted
    </Directory>
</VirtualHost>
" | tee /etc/apache2/sites-available/abimanyu.it14.conf

# Aktifkan situs dan modul yang diperlukan
a2ensite abimanyu.it14.conf
a2enmod rewrite

# Restart Apache
service apache2 restart

# Buat file .htaccess dengan aturan rewrite
echo "
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ /index.php/$1 [L]
" > /var/www/abimanyu.it14/.htaccess

# Tambahkan entri di file hosts (jika Anda melakukan ini di mesin lokal atau jika tidak memiliki DNS yang terkonfigurasi)
# echo "127.0.0.1 abimanyu.it14.com" | tee -a /etc/hosts

# Buat direktori untuk situs web Parikesit
mkdir -p /var/www/parikesit.abimanyu.it14

# Gantilah ini dengan tautan Google Drive yang benar untuk resource Parikesit
parikesit_google_drive_link="https://drive.usercontent.google.com/download?id=1LdbYntiYVF_NVNgJis1GLCLPEGyIOreS&export=download&authuser=0&confirm=t&uuid=ca0a64db-915b-48ec-83d5-33c7a535cfb5&at=APZUnTUgx7sZLt9wUe5NppQoAv87:1697108475851"

# Nama file yang akan disimpan untuk Parikesit
parikesit_output_file="parikesit.zip"

# Unduh file dari Google Drive untuk Parikesit
curl -L -o "$parikesit_output_file" "$parikesit_google_drive_link"

# Mengekstrak file untuk Parikesit
unzip "$parikesit_output_file"
# Sesuaikan ini jika struktur direktori dalam zip berbeda
mv parikesit.abimanyu.yyy.com/* /var/www/parikesit.abimanyu.it14




echo "
<VirtualHost *:80>
    ServerAdmin webmaster@parikesit.abimanyu.it14.com
    ServerName parikesit.abimanyu.it14.com
    ServerAlias www.parikesit.abimanyu.it14.com
    DocumentRoot /var/www/parikesit.abimanyu.it14
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    ErrorDocument 404 /error/404.html
    ErrorDocument 403 /error/403.html

    <Directory /var/www/parikesit.abimanyu.it14>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    <Directory /var/www/parikesit.abimanyu.it14/public>
        Options +Indexes
    </Directory>

    Alias "/js" "/var/www/parikesit.abimanyu.it14/public/js"

    <Directory "/var/www/parikesit.abimanyu.it14/public/js">
        Options +Indexes
        AllowOverride All
        Require all granted
    </Directory>

    <Directory /var/www/parikesit.abimanyu.it14/secret>
        Options -Indexes
        Require all denied
    </Directory>

    <Directory /var/www/error>
        Options -Indexes
        Require all granted
    </Directory>
</VirtualHost>
" | tee /etc/apache2/sites-available/parikesit.abimanyu.it14.conf

mkdir /var/www/parikesit.abimanyu.it14/secret


# Aktifkan situs Parikesit
a2ensite parikesit.abimanyu.it14.conf

# Restart Apache
service apache2 restart

echo "Setup untuk Parikesit dengan hak akses khusus selesai!"


# Buat direktori untuk situs web rjp.baratayuda.abimanyu
mkdir -p /var/www/rjp.baratayuda.abimanyu.it14.com

# Gantilah ini dengan tautan Google Drive yang benar untuk rjp.baratayuda.abimanyu
baratayuda_google_drive_link="https://drive.google.com/uc?export=download&id=1pPSP7yIR05JhSFG67RVzgkb-VcW9vQO6"

# Nama file yang akan disimpan untuk rjp.baratayuda.abimanyu
baratayuda_output_file="baratayuda.zip"

# Unduh file dari Google Drive untuk rjp.baratayuda.abimanyu
curl -L -o "$baratayuda_output_file" "$baratayuda_google_drive_link"

# Mengekstrak file untuk rjp.baratayuda.abimanyu
unzip "$baratayuda_output_file"
mv rjp.baratayuda.abimanyu.yyy.com/* /var/www/rjp.baratayuda.abimanyu.it14.com/

# Update the Apache VirtualHost configuration for rjp.baratayuda.abimanyu
echo "
<VirtualHost *:14000 *:14400>
    ServerAdmin webmaster@rjp.baratayuda.abimanyu.it14.com
    ServerName rjp.baratayuda.abimanyu.it14.com
    ServerAlias www.rjp.baratayuda.abimanyu.it14.com
    DocumentRoot /var/www/rjp.baratayuda.abimanyu.it14.com/
    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined

    <Directory /var/www/rjp.baratayuda.abimanyu.it14.com/>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
        AuthType Basic
        AuthName "Restricted Content"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
    </Directory>
</VirtualHost>
" | tee /etc/apache2/sites-available/rjp.baratayuda.abimanyu.it14.com.conf

# Create the Apache password file with the specified username and password
htpasswd -b /etc/apache2/.htpasswd Wayang baratayudayyyit14

# Set proper permissions for the .htpasswd file
chmod 640 /etc/apache2/.htpasswd

# Add the custom ports to /etc/apache2/ports.conf
echo "Listen 14000" >> /etc/apache2/ports.conf
echo "Listen 14400" >> /etc/apache2/ports.conf

# Enable the rjp.baratayuda.abimanyu site
a2ensite rjp.baratayuda.abimanyu.it14.com.conf

# Restart Apache
service apache2 restart

echo "Setup for rjp.baratayuda.abimanyu with custom ports and authentication is complete!"

```

Konfigurasi pertama adalah untuk membuat sebuah virtual host pada server web Apache yang akan menangani permintaan untuk nama domain "abimanyu.it14.com" dan "www.abimanyu.it14.com". Virtual host ini akan mengarahkan permintaan ke direktori "/var/www/abimanyu.it14" sebagai root situs webnya. Selain itu, konfigurasi ini mencakup pengaturan log, pengaturan kustom untuk mengatasi kesalahan 404 dan 403, serta pengaturan izin yang memungkinkan akses ke direktori yang diperlukan. Hasil konfigurasi ini disimpan dalam file konfigurasi virtual host Apache yang bernama "abimanyu.it14.conf" di direktori "/etc/apache2/sites-available/".

Selanjutnya melakukan beberapa tindakan konfigurasi untuk situs web dengan nama domain "parikesit.abimanyu.it14.com". Pertama, dalam blok pertama, sebuah aturan mod_rewrite didefinisikan dalam file .htaccess yang ada di direktori situs web. Aturan ini memeriksa apakah permintaan yang masuk tidak sesuai dengan file yang ada (RewriteCond %{REQUEST_FILENAME} !-f) atau direktori yang ada (RewriteCond %{REQUEST_FILENAME} !-d) dan, jika tidak, akan meminta ulang permintaan ke /index.php/$1.

Kemudian, dalam blok kedua, skrip membuat direktori untuk situs web "parikesit.abimanyu.it14.com" dan mengunduh file dari Google Drive menggunakan curl, mengekstraknya, dan mengatur konfigurasi Apache untuk situs web tersebut. Ini mencakup pengaturan ServerAdmin, DocumentRoot, log file, serta aturan-aturan terkait direktori yang menentukan izin dan pengaturan akses untuk direktori dan file tertentu dalam situs web. Selanjutnya, Alias dibuat untuk direktori "/js", yang akan mengarahkan permintaan ke direktori yang sesuai dalam situs web. Seluruh konfigurasi ini disimpan dalam file .conf yang akan digunakan oleh Apache untuk mengatur situs web "parikesit.abimanyu.it14.com".

