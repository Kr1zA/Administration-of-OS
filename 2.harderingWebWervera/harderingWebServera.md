# Hardering Web servera

* pri tvorbe tohoto návodu som použil [raspberry pi zero](https://www.raspberrypi.org/products/raspberry-pi-zero/)
* použitá linuxová distribúcia bola [Arch Linux](https://archlinuxarm.org/platforms/armv6/raspberry-pi)
* pi zero sa nachádzalo v lokálnej sieti na adrese 192.168.1.115 a konfiguroval som cez SSH nastavené v predchádzajúcom návode, pri úpravách konfiguračných súborov používam editor vim (môžete používať ľubovoľný editor napr. nano)

## Cieľ:
* Fungujúci Wordpress

## Čo budeme potrebovať:
1. [Apache server](https://httpd.apache.org/)
2. [PHP](http://php.net/)
3. [MySQL](https://mariadb.com/)
4. [Wordpress](https://wordpress.org/)

## 1. Inštálácia a nastavenie Apache servera
* nainštalujme balík apache:

> sudo pacman -S apache

* spustíme apache server príkazom:

> sudo systemctl start httpd

* restart apache servera:

> systemctl restart http

* príkaz pre otestovanie správnosti nastaveni:

> apachectl configtest

Hlavný konfiguračný súbor je `/etc/httpd/conf/httpd.conf`, zaujímavé časti:
  * `Listen 80` - nastavenie na akom porte bude Apache počúvať
  * `User http`, `Group http` - pri prvom spusteni httpd je vytvorený používateľ a skupina http
  * `ServerAdmin you@example.com` - emailová adresa zobrazovaná na chybovych stránkach
  * `DocumentRoot "/srv/http"` - umiestnenie webovych stránok
  * `ErrorLog "/var/log/httpd/error_log"` - umiestnenie súboru s logmi chýb
  * `LogLevel warn` - nastavenie [levelu logovania](https://httpd.apache.org/docs/2.4/mod/core.html#loglevel)
Do hlavného konfiguračného súboru `/etc/httpd/conf/httpd.conf` je možné pridávať ďalšie nastavenia uložene v iných súboroch, ktoré treba zapísať pomocou `Include conf/extra/nazovKonfiguracnehoSuboru.conf` kde `nazovKonfiguracnehoSuboru.conf` je pridávaný konfiguračný súbor
    
### Používateľské priečinky, alebo web pre každého používateľa 
* príklad https://thesis.science.upjs.sk/~rstana/
* takéto niečo vieme vytvoriť odkomentovaním 

>Include conf/extra/httpd-userdir.conf 

v `/etc/httpd/conf/httpd.conf`, čím sa na adrese `webovyserver.com/~menoPouzivatela` zobrazi obsah uložený v zložke `~/public_html` každého používateľa

* pre správne fungovanie musia byť nastavené oprávnenia tak, aby apache vedel čítať zložku `~/public_html`, čo urobíme pomocou:

>chmod o+x ~

>chmod o+x ~/public_html

>chmod -R o+r ~/public_html

* pre aplikovanie zmien reštartujeme apache server

Teraz môžeme vyskúšat funkčnosť apache servera tým, že do webového prehliadača zadáme adresu 192.168.1.115. ak je vsetko v poriadku, malo by nám zobraziť stránku apache servera bežiaceho na pi zere. V prípade problémov môžme skontrolovať nastavenie apache servera príkazom:

> apachectl configtest

## 2. PHP
* najprv nainštalujeme PHP pomocou:

> sudo pacman -S php php-apache

* v `/etc/httpd/conf/httpd.conf` zakomentujeme 

> #LoadModule mpm_event_module modules/mod_mpm_event.so 

a odkomentujeme 

> LoadModule mpm_prefork_module modules/mod_mpm_prefork.so

* Aby sme sfunkčnili PHP pridáme do `/etc/httpd/conf/httpd.conf` na koniec listu `LoadModule` tieto riadky:

>LoadModule php7_module modules/libphp7.so

>AddHandler php7-script .php

a na koniec `Include` listu riadok:

>Include conf/extra/php7_module.conf

* pre aplikovanie zmien reštartujeme apache server

## 3. MySQL

* nainštalujeme balík mariadb (MariaDB je defaultná implementácia MySQL pre Arch Linux): 

> sudo pacman -S mariadb

* pred spustením servisu spustíme nasledujúci príkaz:

> sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql

* spustíme MySQL príkazom:

> sudo systemctl start mariadb

* aby sa MySQL spúšťal pri štarte:

> sudo systemctl enabe mariadb

* nasledujúcim príkazom interaktívne nastavíme zabezpecenie databázy:

> sudo mysql_secure_installation

## 4. Wordpress

Máme nainštalované a základne nastavené všetky potrebné veci pre Wordpress, môžeme ho nainštalovať pikazom `sudo pacman -S wordpress`, čo ale robiť nebudeme. Kôli zložitému nastavovaniu oprávnení a kôli tomu, že WordPress má vlastný manažment aktualizácií ho nainštalujeme ručne.

Zo stránky wordpress.org stiahneme a rozbalíme poslednú verziu WordPressu:

>cd /srv/http
>sudo wget https://wordpress.org/latest.tar.gz
>sudo tar xvzf latest.tar.gz

Potrebujeme vytvoriť konfiguračný súbor pre apache, aby vedel, kde je WordPress nainštalovaný:

* vytvoríme konfiguracný súbor:

>sudo vim /etc/httpd/conf/extra/httpd-wordpress.conf

* vložíme doňho nasledovné:


``` 
Alias / "/srv/http/wordpress/wordpress/"

<Directory "/srv/http/wordpress/wordpress/">
         AllowOverride All
         Options FollowSymlinks
         Require all granted
</Directory>

```
## 5. Bonus Https, SSL...

Pre povolenie SSL odkomenrujeme v '/etc/httpd/conf/httpd.conf' nasledujúce riadky:

>LoadModule ssl_module modules/mod_ssl.so

>LoadModule socache_shmcb_module modules/mod_socache_shmcb.so

>Include conf/extra/httpd-ssl.conf

Ak chceme zabezpečené pripojenie cez https a SSL budeme potrebovať certifikát. Môžeme si podpísať vlastnou certifikačnou autoritou sami ale webový prehliadač bude papuľovať. Necháme si teda certifikát vygenerovať certifikačnou autoritou [Let’s Encrypt](https://letsencrypt.org/). Na to budeme potrebovať oficiálneho klienta s názvom Certbot, ktorý nám vygeneruje certifikát a vŠetko potrebná nastaví. Nainštalujeme ho pomocou:

> sudo pacman -S certboot

Následne ešte povolíme rewrite_module odkomentovaním nasledúceho riadka v '/etc/httpd/conf/httpd.conf' aby certbot nevypísal chybu a mohol presmerovat http na https:

> LoadModule rewrite_module modules/mod_rewrite.so 

Certbot spustíme príkazom:

> sudo certbot --apache

Certbot sa nás spýta pre ktorý web chceme aktivovať HTTPS. Zvolíme číslo a stlačime enter. Následne Certbot získa certifikát a verifikuje nastavenie servera. Potom sa už len spýta, či má presmerovať HTTP na HTTPS a tým je hotovo. Náš web je zabezpečený.

Keďže certifikát má určitú platnosť, niekedy je potrebné ho obnoviť. To mǒžeme urobiť príkazom:

> sudo certbot renew
