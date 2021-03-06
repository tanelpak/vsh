## PHP7 ja PHP5 install
Kõik käsud läbi viidud root kasutajana.  
Esmalt viime läbi masina uuendused
```
apt-get update
apt-get upgrade
```
Lubame PHP PPA
```
apt install ca-certificates apt-transport-https 
wget -q https://packages.sury.org/php/apt.gpg -O- | sudo apt-key add -
echo "deb https://packages.sury.org/php/ jessie main" | tee /etc/apt/sources.list.d/php.list
```
Installime php7.2
```
apt update
apt install php7.2
```
Seejärel lisame vajalikud moodulid
```
apt install php7.2-cli php7.2-common php7.2-curl php7.2-gd php7.2-json php7.2-mbstring php7.2-mysql php7.2-xml
```
## MYSQL paigaldus
Liigu /tmp/ kausta ja tõmba alla mysql conf
```
cd tmp/
wget https://dev.mysql.com/get/mysql-apt-config_0.8.9-1_all.deb
```
Käivitame mysql confi
```
dpkg -i mysql-apt-config_0.8.9-1_all.deb
```
MYSQL Server & Cluster versiooni muudame mysql-5.6  
Teeme uuendused
```
apt-get update
```
Installime serveri
```
apt-get install mysql-community-server
```
Kui failid alla tõmmatud, saame kohe määrata mysql root parooli.  
Parooliks määratud "qwerty"
## PHPMYADMIN
Phpmyadmini kasutamiseks teeme uuendused ja installime phpmyadmini
```
apt-get update
apt-get upgrade
apt-get install phpmyadmin
```
phpmyadmini installi käigus määrame sisse logimiseks parool "qwerty"  
Kui see tehtud, logime läbi graafilise liidese sisse aadressil
```
http://172.23.13.41/phpmyadmin/
```
Sisse logimisel kasutame parool "qwerty" ja kasutajanime "root"  
Töötavast phpmyadminist on pilt githubis /vsh/praks3/Töötavphpmyadmin.png
## PHP töö kontroll ja skript
PHP töö kontrollimiseks kirjutasime /var/www/html kausta faili nimega "test.php"  
Faili sisu
```
<?php phpinfo(); ?>
```
Kaustas /vsh/praks3 on pilt väljundist graafilises liideses nimega phpinfo.png
## PHP tugi
Et võimaldada tugi, tegin kõigepealt vastavasse kohta tugi.php faili
```
nano /var/www/public_html/tugi.php
```
Faili sisu :
```
<html>
 <head>
  <title>PHP tugi</title>
 </head>
 <body>
 <?php echo '<p>Tugi</p>'; ?> 
 </body>
</html>
```
Seejärel tee apachele restart
```
restart service apache2
```
Seejärel muutisn apache2 confis commentiga ära kõik usermodule read  
Faili asukoht :
```
/etc/apache2/mods-enabled/php7.2.conf
```
Seejärel apachele restart
```
service apache2 restart
```
Pildid väljundist on /vsh/praks3 kaustas nimedega tugi1.png ja tugi2.png
