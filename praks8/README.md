## Masinad

#### dns - 
Linux, Ubuntu, ilma graafikat, standart system utilities ja ssh  
1 CPU  
2 GB RAM  
16 GB HDD  
#### apache - 
Linux, Ubuntu, ilma graafikat, standart system utilities ja ssh  
db - Linux, Ubuntu  
ilma graafikat  
standart system utilities ja ssh 
#### client -  
WIN10, testimiseks  
4 CPU  
16 GB RAM   
100 GB HDD  
  
## Apache, PHP, MySQL 
Kasutati järgmist õpetust : https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04  
Esmalt, et alustada tööd, paigaldame "apache" masinale apache, MySQLi ja PHP.  
Apache paigaldamine :  
```
sudo apt-get update  
sudo apt-get install apache2  
```
MySQLi paigaldamine, parooliks määrasime "qwerty":
```
sudo apt-get install mysql-server  
mysql_secure_installation
```
PHP paigaldamine :  
```
sudo apt-get install php libapache2-mod-php php-mcrypt php-mysql
```
Seejärel tegime apachele restarti ja veendusime, et apache töötab
```
sudo systemctl restart apache2  
sudo systemctl status apache2
```
Kui kõik tehti õigesti, siis viimase käsu käivitades saime järgmine väljundi :  '
```
● apache2.service - LSB: Apache2 web server
   Loaded: loaded (/etc/init.d/apache2; bad; vendor preset: enabled)
  Drop-In: /lib/systemd/system/apache2.service.d
           └─apache2-systemd.conf
   Active: active (running) since Wed 2016-04-13 14:28:43 EDT; 45s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 13581 ExecStop=/etc/init.d/apache2 stop (code=exited, status=0/SUCCESS)
  Process: 13605 ExecStart=/etc/init.d/apache2 start (code=exited, status=0/SUCCESS)
    Tasks: 6 (limit: 512)
   CGroup: /system.slice/apache2.service
           ├─13623 /usr/sbin/apache2 -k start
           ├─13626 /usr/sbin/apache2 -k start
           ├─13627 /usr/sbin/apache2 -k start
           ├─13628 /usr/sbin/apache2 -k start
           ├─13629 /usr/sbin/apache2 -k start
           └─13630 /usr/sbin/apache2 -k start
```

