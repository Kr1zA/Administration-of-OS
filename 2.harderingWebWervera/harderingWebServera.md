# Hardering Web servera

* pri tvorbe tohoto návodu som použil [raspberry pi zero](https://www.raspberrypi.org/products/raspberry-pi-zero/)
* použitá linuxová distribúcia bola [Arch Linux](https://archlinuxarm.org/platforms/armv6/raspberry-pi)
* pi zero sa nachádzalo v lokálnej sieti na adrese 192.168.1.115 a konfiguroval som cez SSH nastavené v predchádzajúcom návode

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

* hlavný konfiguračný súbor je `/etc/httpd/conf/httpd.conf`, zaujímavé časti:
  * `Listen 80` - nastavenie na akom porte bude Apache počúvať
  * `User http`, `Group http` - pri prvom spusteni httpd je vytvorený používateľ a skupina http
  * `ServerAdmin you@example.com` - emailová adresa zobrazovaná na chybovych stránkach
  * `DocumentRoot "/srv/http"` - umiestnenie webovych stránok
  * `ErrorLog "/var/log/httpd/error_log"` - umiestnenie súboru s logmi chýb
  * `LogLevel warn` - nastavenie levelo logovania

* restart apache servera
> systemctl restart http
