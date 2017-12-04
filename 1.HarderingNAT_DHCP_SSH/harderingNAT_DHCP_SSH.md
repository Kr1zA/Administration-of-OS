## 1. Hardering SSH servera

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
