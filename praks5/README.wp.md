# Wordpressi paigaldus
## Wordpressi install
Dokumentatsioon käsitleb wordpressi installi  
Kõik käsud läbi viidud root kasutajana  
Esmalt liigume kausta, kuhu tahame wordpressi tõmmata ja laeme selle alla
```
cd /var/www/html/
wget /http://wordpress.org/latest.zip
```
Installime unzipi, et tõm,matud fail lahti pakkida
```
aptitude install unzip
```
Pakime wordpressi lahti
```
unzip -q latest.zip
```
Määrame kausta õigused
```
chown -R www-data:www-data /var/www/html/wordpress
chmod -R 755 /var/www/html/wordpress
```
Loome kausta, kuhu failid laadida
```
mkdir -p /var/www/html/wordpress/wp-content/uploads
```
Määrame õigused, et server saaks õigused read and write
```
chown -R www-data:www-data /var/www/html/wordpress/wp-content/uploads
```
## MYSQL Database loomine
Logime sisse MYSQLi
```
mysql -u root -p
```
Loome andmebaasi ja kasutaja
```
CREATE DATABASE wordpress_sample;
CREATE USER wp_user@localhost IDENTIFIED BY 'wp_password';
GRANT ALL PRIVILEGES ON wordpress_sample.* TO wp_user@localhost;
FLUSH PRIVILEGES;
```
Väljume MYSQList
```
exit
```
Seejärel avame kliendi masina ja logime wordpressi sisse. Selleks kasutame enda ip-d
```
https://172.23.13.41/wordpress
```
Sisse logimiseks kasutame eelenvalt loodud andmebaasi ja selle kasutajat. Kui sisse logitud, vajutame "Run installation" ja määrame lehele nime jms. Lõpuks viisime lehel sisse ka mõed muudatused, valmis lehest pilt kasutas /vsh/praks5/ ja pildi nimeks "wordpress"
