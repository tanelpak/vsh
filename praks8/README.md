## Töö
Allpool kirjeldatakse, kuidas paigaldata ühte serverisse mitu veebilehte. Selleks tehti kolm masinat : 
1) DNS server
2) WEB server , kus töötab wordpress ning dokuwiki
3) Klient testimiseks  
Samuti on olemas pildid töötava dokuwiki ning wordpressi kohta (/vsh/praks8/ vastavalt dokuwiki.png ning wordpress.png). Pildile on kaasatud ka vastava lehe apache conf ning dnsi tanel.loc conf

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
## IP (interfaces)
Kasutati järgmist õpetust : https://linuxconfig.org/how-to-configure-static-ip-address-on-ubuntu-18-04-bionic-beaver-linux  
Määrame masintale staatilised IP-d  
Selleks viisime masinate /etc/network/interfaces failis läbi järgmised muudatused :
### Apache masin
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto ens160
iface ens160 inet static
        address 192.168.16.56
        netmask 255.255.240.0
        network 192.160.16.0
        broadcast 192.168.31.255
        gateway 192.168.16.1
        nameserver 192.168.16.19
```
### DNS masin
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto ens160
iface ens160 inet static
        address 192.168.16.19
        netmask 255.255.240.0
        network 192.168.16.0
        broadcast 192.168.31.255
        gateway 192.168.16.1
        dns-nameservers 192.168.16.19

```
## DNS seadistamine
DNS-i seadistamiseks kasutati järgmist juhendit : https://www.ostechnix.com/install-and-configure-dns-server-ubuntu-16-04-lts/  
Esmalt installime DNS masinale bind9 : 

```
sudo apt-get install bind9 bind9utils bind9-doc  
sudo nano /etc/bind/named.conf.options  
```
Selles failis võtame commenti ära järgmistelt ridatelt 
```
forwarders {
 8.8.8.8;
 };
```
Bind9-le restart
```
sudo systemctl restart bind9
```
Et kontrollida, kas bind9 töötab, sisestame käsu :
```
dig -x 127.0.0.1
```
Väljund peaks olema järgmine : 
```
; <<>> DiG 9.10.3-P4-Ubuntu <<>> -x 127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22769
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;1.0.0.127.in-addr.arpa. IN PTR

;; ANSWER SECTION:
1.0.0.127.in-addr.arpa. 604800 IN PTR localhost.

;; AUTHORITY SECTION:
127.in-addr.arpa. 604800 IN NS localhost.

;; ADDITIONAL SECTION:
localhost. 604800 IN A 127.0.0.1
localhost. 604800 IN AAAA ::1

;; Query time: 0 msec
;; SERVER: 192.168.1.200#53(192.168.1.200)
;; WHEN: Tue Aug 23 15:53:59 IST 2016
;; MSG SIZE rcvd: 132
```
#### DNSi conf
Esmalt installime vajalikud paketid : 
```
sudo apt-get install bind9 bind9utils bind9-doc
```
Teeme bindi confis muudatused :
```
sudo nano /etc/bind/named.conf
```
.
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```
Määrame forward ja reverse zone failid : 
```
sudo nano /etc/bind/named.conf.local
```
.
```
zone "tanel.loc" {
        type master;
        file "/etc/bind/tanel.loc";
        allow-transfer { 192.168.1.201; };
        also-notify { 192.168.1.201; };
 };
zone "1.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/rev.tanel.loc";
        allow-transfer { 192.168.1.201; };
        also-notify { 192.168.1.201; };
 };
 ```
 Loome failid, mille asukoha eelmises käsus määrasime : 
 ```
 sudo nano /etc/bind/tanel.loc
 ```
 .
 ```
  GNU nano 2.5.3                            File: /etc/bind/tanel.loc                                                               

$TTL 86400
@       IN      SOA     VSH-DNS.tanel.loc. root.tanel.loc (
                2011071001      ;Serial
                3600            ;Refresh
                1800            ;Rerty
                604800          ;Expire
                86400           ;Minimum TTL
)
@       IN      NS      VSH-DNS
@       IN      A       192.168.16.19
vsh-dns IN      A       192.168.16.19
vsh-apache      IN      A       192.168.16.52
wp      IN      A       192.168.16.56
wiki    IN      A       192.168.16.56
```
Järgmisena teeme reverse zone : 
```
$TTL 86400
@       IN      SOA     VSH-DNS.tanel.loc. root.tanel.loc (
                2011071001      ;Serial
                3600            ;Refresh
                1800            ;Rerty
                604800          ;Expire
                86400           ;Minimum TTL
)
@       IN      NS      VSH-DNS
@       IN      A       192.168.16.19
vsh-dns IN      A       192.168.16.19
56      IN      PTR     tanel.loc
```
Määrame õigused ja omaniku : 
```
sudo chmod -R 755 /etc/bind  
sudo chown -R bind:bind /etc/bind
```
Kontrollime confi : 
```
sudo named-checkconf /etc/bind/named.conf  
sudo named-checkconf /etc/bind/named.conf.local  
```
Samuti saame kontrollida confi faili : 
```
sudo named-checkzone tanel.loc /etc/bind/tanel.loc
```
Määrame interfacis DNS serveri lisades vastava rea:
```
sudo nano /etc/network/interfaces
```
.
```
dns-nameservers 192.168.16.19
```
Bind9 restart : 
```
sudo systemctl restart bind9
```
#### Testime DNS-i
```
dig tanel.loc
```
Väljund : 
```
; <<>> DiG 9.10.3-P4-Ubuntu <<>> tanel.loc
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61441
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;tanel.loc.                     IN      A

;; ANSWER SECTION:
tanel.loc.              86400   IN      A       192.168.16.19

;; AUTHORITY SECTION:
tanel.loc.              86400   IN      NS      VSH-DNS.tanel.loc.

;; ADDITIONAL SECTION:
vsh-dns.tanel.loc.      86400   IN      A       192.168.16.19

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Thu Apr 25 11:16:29 EEST 2019
;; MSG SIZE  rcvd: 100
```
#### Määrame kliendile DNS-i
Selleks lähme apache masinasse ja teeme interfaces failis vastava kirje
```
sudo nano /etc/network/interfaces
```
Sisu peaks olema järgnev : 
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto ens160
iface ens160 inet static
        address 192.168.16.56
        netmask 255.255.240.0
        network 192.160.16.0
        broadcast 192.168.31.255
        gateway 192.168.16.1
        nameserver 192.168.16.19
```
## Wordpressi paigaldus
Kuidas paigaldada wordpressi, näeb VSH praks 5 kaustast.
## Klient :
Samuti peame määrama kliendi masinas, millist DNSi kasutada. Selle kohta on /praks8 kaustas pilt "win-interfaces"
## Dokuwiki paigaldus
Dokuwiki paigaldamisel kasutati järgmist õpetust :  
https://www.vultr.com/docs/how-to-install-dokuwiki-on-ubuntu-16-04-lts  
Paigaldamine algas php pakettide paigaldamisega : 
```
sudo apt install -y php7.0 php7.0-cli php7.0-fpm php7.0-gd php7.0-xml php7.0-zip
```
Apache confis "sites-available" copisime eelenvalt tehtud "wordpress.confi" samasse kasuta ning nimetasime ümber "dokuwiki.conf"  
Dokuwiki confis muutsime kirjed, mis sisaldasid wordpressi ümber dokuwikiks (asukoht, servername wiki.tanel.loc)  
Seejärel tegime dokuwiki jaoks kausta, tõmbasime dokuwiki alla, pakkisime lahti.
```
sudo mkdir -p /var/www/dokuwiki
cd /var/www/dokuwiki
```
```
wget https://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz
```
```
tar xvf dokuwiki-stable.tgz
```
```
sudo chown -R www-data:www-data /var/www/dokuwiki
```
Dokuwiki töötamist kontrollime läbi kliendi http://wiki.tanel.loc

  
    
## SSL Ühenduse loomine

Järgnevalt lisame ssl sertifikaadid, et ühendus oleks turvaline.

### Sertifikaadid
Esmalt peame looma asukohad, kuhu me sertifikaadid paigutame. 
```
mkdir -p /etc/apache2/ssl/wp.tanel.loc
mkdir -p /etc/apache2/ssl/wiki.tanel.loc
```
Lubame SSL moodule ning siis apachele restart
```
sudo a2enmod ssl
sudo service apache2 restart
```
Loome sertifikaadid eelnevalt loodud kohtadesse. Selleks tuleb mõlemale (nii wordpress ehk wp ja dokuwikile ehk wiki) folderile teha eraldi cert.
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/wp.tanel.loc/apache-selfsigned.key –out /etc/apache2/ssl/wp.tanel.loc/apache-selfsigned.crt  
```
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/wiki.tanel.loc/apache-selfsigned.key –out /etc/apache2/ssl/wiki.tanel.loc/apache-selfsigned.crt  
```
See käivitab väljundi, kus on vaja määrata põhiandmed. Common Name kohale määrame vastava sertifikaadi aadressi, nt esimese puhul wp.tanel.loc
```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:EE
State or Province Name (full name) [Some-State]:Eesti
Locality Name (eg, city) []:Tartu
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Awesome Inc
Organizational Unit Name (eg, section) []:Dept of Merriment
Common Name (e.g. server FQDN or YOUR name) []:wp.tanel.loc                
Email Address []:webmaster@awesomeinc.com
```
### Virtual Hosts
Järgmiseks peame looma hostid, kus määrame ära veebilehe aadressi ja certi asukoha. See tuleb teha taas vastavalt mõlemale lehele eraldi  
Faili peavad lõppema .conf lõuga, muidu ei tunne apache neid hiljem ära
```
sudo nano /etc/apache2/sites-available/wiki.conf
sudo nano /etc/apache2/sites-available/wp.conf
```
Failide sisu on allpool kirjeldatud. DocumentRoot peab olema leheküle folder, Servername vastavalt lehe aadress ja SSLCertificateFile ja KeyFile peavad olema eelnevalt loodud sertifikaatide asukohad!
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName example.com
        DocumentRoot /var/www

</VirtualHost>


<IfModule mod_ssl.c>
<VirtualHost *:443>

        ServerAdmin webmaster@localhost
        ServerName example.com
        DocumentRoot /var/www

        #   SSL Engine Switch:
        #   Enable/Disable SSL for this virtual host.
        SSLEngine on

        #   A self-signed (snakeoil) certificate can be created by installing
        #   the ssl-cert package. See
        #   /usr/share/doc/apache2.2-common/README.Debian.gz for more info.
        #   If both key and certificate are stored in the same file, only the
        #   SSLCertificateFile directive is needed.
        SSLCertificateFile /etc/apache2/ssl/example.com/apache.crt
        SSLCertificateKeyFile /etc/apache2/ssl/example.com/apache.key
</VirtualHost>

</IfModule>
```

### Port
Järgmisena teeme ports.conf failis ühe muudatuse, täpsemalt lisame allpool toodud rea 
```
sudo nano /etc/apache2/ports.conf 
```
```
NameVirtualHost *:443
```
Siis aktiveerimine hostid eelnevalt loodud confi failide abil
```
sudo a2ensite wp.conf
sudo a2ensite wiki.conf
```
Lõpetuseks apachele restart 
```
sudo service apache2 restart
```
Seejärel saame läbi kliendi testida tulemust. Vastavasisulised pildid on praks8 kasutas nimedega wp.tanel.loc_https ja wiki.tanel.loc_https
