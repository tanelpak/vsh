## Remote DB
Käesolevas dokumentatsioonis on sammud, et installida wordpress ühes masinas ja DB teises masinas  
Kõik käsud läbi viidud root kasutajana  
Veebiserveri IP on 10.0.2.4 ja db IP on 10.0.2.5
### MYSQL masin
Installin masinas uuendused ja mysql serveri
```
apt-get update
apt-get install mysql-server
mysql_install_db
mysql_secure_installation
```
Selle käigus määrame ka parooli. Parooliks on 'qwerty'
Lubame MYSQL remote accessi. Selleks:
```
nano /etc/mysql/my.cnf
```
Ja selles failis teeme järgmised muudatused [mysqld] lahtris: 
```
bind-address        = 10.0.2.5
```
IP on database masina IP  
Seejärel restart mysqli
```
service mysql restart
```
Järgmisena logime mysqli sisse ja teeme database ja kasutajad
```
mysql -u root -p
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'qwerty';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
CREATE USER 'wordpressuser'@'10.0.2.4' IDENTIFIED BY 'qwerty';
GRANT SELECT,DELETE,INSERT,UPDATE ON wordpress.* TO 'wordpressuser'@'10.0.2.4';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'10.0.2.4';
FLUSH PRIVILEGES;
exit
```
Installime veebiserverisse mysql kliendi
```
apt-get update
apt-get install mysql-client
```
Seejärel proovime läbi veebiserveri masina MYSQLi sisse logida. Kui see töötab, siis on hea!
```
mysql -u wordpressuser -h 10.0.2.5 -p
```
### Veebiserveri seadistamine
Installime nginx-i
```
apt-get install nginx php5-fpm php5-mysql
```
PHP seadistamine
```
nano /etc/php5/fpm/php.ini
```
Seal muudame ühe rea (tuleb comment ära võtta):
```
cgi.fix_pathinfo=0
```
Järgmisena teeme muudatused php confis:
```
nano /etc/php5/fpm/pool.d/www.conf
```
Muudame read, et ei kuulaks porti, vaid unix domain socket
```
listen = /var/run/php5-fpm.sock
```
Service restart
```
service php5-fpm restart
```
### Nginx-i conf
Teeme defaultis koopia
```
cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example.com
```
Avame koopia
```
nano /etc/nginx/sites-available/example.com
```
Selles failis teeme järgmised muudatused :
```
server {
    listen 80;
    root /var/www/example.com;
    index index.php index.hmtl index.htm;
    server_name example.com;
    location / {
        try_files $uri $uri/ /index.php?q=$uri&$args;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/www;
    }
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
    }
}
```
Seejärel kustutame defaulti, lingime uue ja teeme restarti:
```
rm /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
service nginx restart
```
### WORDPRESSI install
Liigume root kausta ja tõmbame vajalikud failid, seejärel pakime need lahti:
```
cd ~
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
```
Kopeerime confi ja teeme conf failis vajalikud muudatused
```
cp ~/wordpress/wp-config-sample.php ~/wordpress/wp-config.php
nano ~/wordpress/wp-config.php  
```
```
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');

/** MySQL hostname */
define('DB_HOST', '10.0.2.5');
```
Loome uue kausta ja kopeerime vajalikud failid:
```
mkdir -p /var/www/example.com
cp -r ~/wordpress/* /var/www/example.com
```
Määrame õigused:
```
cd /var/www/example.com
chown -R www-data:www-data *
usermod -a -G www-data your_user
chmod -R g+rw /var/www/example.com
```
Seejärel saame läbi kliendi masina (eelnevalt tehtud vsh_tanel_deb) avada Wordpressi lehe (kasutab IP-d) ja ongi valmis!
