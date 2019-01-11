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
apt install php7.2-cli php7.2-common php7.2-curl php7.2-gd php7.2-json php7.2-mbstring php7.2-xml
```
