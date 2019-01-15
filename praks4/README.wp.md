## SSL sertifikaadi lisamine
Eesmärgiks on lisada HTTPS protokoll  
Kõik käsud läbi viidud root kasutajana  
Esmalt update ja uuendame openssl-i
```
apt-get update
apt-get upgrade openssl
```
Kuna apache2 on juba eelnevalt masinas installitud, siis seda enam vaja teha ei ole. Kui seda ei oleks, siis esmalt apache2 installida   
Kui apache2 peal, siis lubame Apache SSL mooduli ja lubame default lehe
```
a2enmod ssl
a2ensite default-ssl
```
Seejärel apachele restart
```
service apache2 reload
```
Loome SSL certi  
Esmalt teeme selle jaoks vastava kausta
```
mkdir /etc/apache2/ssl
```
Loome uue sertifikaadi ja sellele vastavata key  
Days määrab kaua cert aktiivne on
Keyout määrab kuhu luuakse key  
Out määrab kuhu luuake  cert
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
```
Seejärel tuleb määrata asukoht jms. Nende määramise koht tuleb ette automaatselt.  
Näide:
```
Interactive
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
——-
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:New York
Locality Name (eg, city) []:NYC
Organization Name (eg, company) [Internet Widgits Pty Ltd]:DigitalOcean
Organizational Unit Name (eg, section) []:SSL Certificate Test
Common Name (e.g. server FQDN or YOUR name) []:example.com               
Email Address []:test@example.com
```
Määrame õigused certi ja key kaitsemiseks
```
chmod 600 /etc/apache2/ssl/*
```
Muudame ssl confi 
```
nano /etc/apache2/sites-enabled/default-ssl.conf
```
Failis viime läbi järgmised muudatused
1. Määrame serveri nime real, millele eelneb <VirtualHost _default_:443>. Serveri nime võib asendada ka IP-aadressiga.  
Kuna meie IP on 172.23.13.41, siis määrame selle serveri nimeks.
``` 
ServerAdmin webmaster@localhost
ServerName 172.23.13.41:443
```
2. Määrame certi ja key asukoha
```
SSLCertificateFile /etc/apache2/ssl/apache.crt
SSLCertificateKeyFile /etc/apache2/ssl/apache.key
 ```
 Tulemus peaks välja nähgema selline : 
```
<IfModule mod_ssl.c>
    <VirtualHost _default_:443>
        ServerAdmin webmaster@localhost
        ServerName 172.23.13.41:443
        DocumentRoot /var/www/html

        . . .
        SSLEngine on

        . . .

        SSLCertificateFile /etc/apache2/ssl/apache.crt
        SSLCertificateKeyFile /etc/apache2/ssl/apache.key
```
Teeme apache restart
```
service apache2 reload
```
Seejärel testime. Testimise osast on /vsh/praks4/ kaustas pilt nimega HTTPS, mis näitab, milline on tulemus.
