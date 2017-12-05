
## 1. Hardering NAT servera

Konfigurovať NAT server budeme na Debiane 9 nainstalovanom vo Virtualboxe. Nainštaluejme si 2 virtuálne stroje, kde jeden bude slúžiť ako brána/NAT server a druhý ako klient na ktorom budeme testovať, či nastavená konfigurácia na NAT serveri funguje. 

Na NAT serveri nastavime v nastaveniach virtuálky 2 sieťové adaptéry (jeden potrebujeme na dotiahnutie internetu zvonku - WAN a jeden pre lokálnu siet LAN). Jeden bude nastavený na NAT aby sme dostali IP addresu z voknu z DHCP. Druhý bude nastaveny na Internal Network teda pre nás LAN. Na druhom klientskom stroji nastavíme jednu sieťovku na Internal Network.

Po zapnutí oboch strojov si možeme príkazom `ip a` poziriet dostupné sieťové adaptéry. Na serveri uvidíme 3 adaptéry, v mojom prípade: lo - loopback, enp0s3 ktorý má pridelenú adresu od DHCP, teda je to WAN a enp0s8, ktorý má status DOWN, ktorý bude náš adaptér do lokálnej siete. Na klientovi uvidíme 2 adaptéry, v mojom prípade: lo - loopback a enp0s3, ktorý je tiež DOWN.

Jednotlivé sieťové rozhrania vieme zapínať/vypínať príkazom `ip link set dev ethX up/down` je sieťové rozhranie, napríklad eth0s8.

Teraz potrebujeme nastaviť sietové rozhrania na serveri. Konfiguračný súbor je `/etc/network/interfaces`. Upravíme konfigurák tak, aby vyzeral nasledovne (v komentároch vysvetlenia):

```
source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
auto enp0s3  #pridame, aby sa interface zaplo po reštartovaní sieťových rozhraní
iface enp0s3 inet dhcp

# LAN interface
auto enp0s8 #pridame, aby sa interface zaplo po reštartovaní sieťových rozhraní
iface enp0s8 inet static #nastavime aby bola staticka ip a v dalších riadkoch nastavime parametre
	address 192.168.1.1
	netmask 255.255.255.0
	network 192.168.1.0
	broadcast 192.168.1.255
```

Teraz potrebujeme reštartovať sieťové rozhrania príkazom:

> systemctl restart networking

Týmto máme nastavené rozhranie pre LAN na bráne.

Ak chceme vedieť s klientským počítačom komunikovať s bránou potrebujeme tiež upraviť konfiguračný súbor `/etc/network/interfaces` aby vyzeral nasledovne:

```
source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
auto enp0s3  #pridame, aby sa interface zaplo po reštartovaní sieťových rozhraní
iface enp0s3 inet static #nastavime aby bola staticka ip a v dalších riadkoch nastavime parametre
address 192.168.1.2
netmask 255.255.255.0
gateway 192.168.1.1
broadcast 192.168.1.255
```

Teraz už vieme komunikovať medzi týmito dvoma strojmi, čo vieme skontrolovať napriklad ningnutím. Problém ale nastane, ak chceme pristupovať na internet. Preto potrebujeme nastaviť na bráne smerovanie. To vieme dočastne (do najbližsieho reštartu) urobiť príkazom `sysctl -w net.ipv4.ip_forward=1`


## 2. Hardering DHCP servera



## 3. Hardering SSH servera

Nastavenie SSH servera budeme robiť na počítači [raspberry pi zero](https://www.raspberrypi.org/products/raspberry-pi-zero/) s linuxovou distribúciou [Arch Linux](https://archlinuxarm.org/platforms/armv6/raspberry-pi) nachádzajúcej sa v lokálnej sieti na adrese 192.168.1.115.

SSH server získame nainstalovaním balíka `openssh`. Štandardne je však v distribúcií Arch už nainštalovaný. 

### Konfigurácia

Hlavný konfiguračný súbor je `/etc/ssh/sshd_config`, doležité riadky:
  * `AllowUsers    user1 user2`, povolenie pripojenia len niektorých užívateľov
  * `AllowGroups   group1 group2`, povolenie pripojenia len niektorých skupín
  * `Banner /etc/issue`, pekná uvítacia správa uložená v `/etc/issue`, ktorá sa zobrazí pri prihlasovaní 
  * `Port 8181`, zmena defaultného portu z 22 na 8181
  * `PasswordAuthentication no`, zakázanie prihlasovania heslom (nutnosť mať nakopírovaný verejný kľúč v súbore ~/.ssh/authorized_keys)
  * `PermitRootLogin no` zmenou z `#PermitRootLogin prohibit-password` zakážeme prihlásenie ako root
  
SSH vieme zapnúť/vypnúť/reštartovať pomocou príkazu `sudo systemctl enable/disable/restart sshd`.
