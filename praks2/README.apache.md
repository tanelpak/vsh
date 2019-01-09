# Apache2 paigaldus
Kõigepealt uuendused masinas
```
apt-get update && apt-get upgrade
```
Seejärel installime apache2
```
apt-get install apache2
```
Peale seda sai muudetud fail 
```
/var/www/html/index.html
```

Kui see oli valmis siis sai lubatud UserDir moodul käsuga: a2enmod userdir

Ning siis sai taaskäivitatud apache2 käsuga: service apache2 restart Siis sai tehtud tavakasutaja kausta kaust public_html ja sinna fail index.html
