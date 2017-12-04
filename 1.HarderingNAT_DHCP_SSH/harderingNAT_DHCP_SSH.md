## 1. Hardering SSH servera

Nastavenie SSH servera budeme robiť na počítači [raspberry pi zero](https://www.raspberrypi.org/products/raspberry-pi-zero/) s linuxovou distribúciou [Arch Linux](https://archlinuxarm.org/platforms/armv6/raspberry-pi) nachádzajúcej sa v lokálnej sieti na adrese 192.168.1.115.

SSH server získame nainstalovaním balíka `openssh`. Štandardne je však v distribúcií Arch už nainštalovaný. 

### Konfigurácia



Hlavný konfiguračný súbor je `/etc/ssh/sshd_config`. 

Pre povolenie pripojenia len niektorých užívateľov pridáme riadok:

>AllowUsers    user1 user2

Pre povolenie pripojenia len niektorých skupín pridáme riadok:

>AllowGroups   group1 group2

Peknú uvítaciu správu pri prihlasovaní vieme zobrazit zmenou riadka `#Banner none` na `Banner /etc/issue`, kde `/etc/issue` je súbor peknou uvítacou správou.

Defaultný port 22 na ktorom počúva SSH, môžeme zmeniť riadkom `Port 8181`.

