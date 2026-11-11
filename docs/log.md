## Tag 1: Firewall und erstes Segment (okt)

**Was ich gemacht habe**
- OPNsense in einer neuen VirtualBox-VM installiert (FreeBSD 64-bit, 4 GB RAM, 2 CPUs, 16 GB Disk) mit vier Netzwerkadaptern: einmal NAT (Internet) und drei interne Netze (Clients, Server, DMZ).
- Interfaces zugewiesen: em0 = WAN, em1 = LAN (Clients), em2 = Server, em3 = DMZ.
- LAN auf 10.0.10.1/24 gesetzt, DHCP für 10.0.10.100 bis 10.0.10.200 aktiviert.
- Kali-VM ins Clients-Netz gehängt. Sie bekam 10.0.10.200 von der Firewall, der Ping auf 10.0.10.1 lief mit 0% packet loss (Screenshot: `screenshots/day1-ip-und-ping.png`).
- Über Kali per HTTPS in die OPNsense-Weboberfläche eingeloggt und den Setup-Wizard gestartet.

**Probleme und Lösungen**
- Bei der Interface-Zuweisung habe ich zu schnell bestätigt und sie wiederholt. Es wird erst nach der letzten Abfrage übernommen.

**Was ich gelernt habe**
- Warum man ein Netz in Segmente teilt und die Firewall dazwischen setzt.
- Was DHCP macht
- Warum private Adressen (10.x.x.x) in vielen Netzen gleichzeitig vorkommen können (NAT).

   ![IP-Adresse und Ping](Github/day1-ip-und-ping.png)





## Tag 2: Server- und DMZ-Segment, erste Firewall-Regeln (10./11. November)

Ich habe die zwei übrigen Interfaces der Firewall konfiguriert: OPT1 als Server-Segment (10.0.20.1/24) und OPT2 als DMZ (10.0.30.1/24). Beide Adressen sind statisch, damit die Firewall für die Geräte im Segment immer unter derselben Adresse als Gateway erreichbar ist.

![Interfaces auf dem Dashboard](/docs/Github/day-2-interface.png)
![Hostname der Firewall](/docs/Github/day2-hostname.png)

Danach habe ich einen Ubuntu-Server (srv-linux01) installiert, aktualisiert und mit einer festen Adresse (10.0.20.10) ins Server-Segment gehängt. Die Netzwerkkonfiguration (Netplan) ist eine YAML-Datei, bei der die Einrückung entscheidet. Mit `netplan try` habe ich sie getestet, bevor ich sie übernommen habe.

**Default deny in der Praxis:** Der erste Ping vom Server zur Firewall wurde blockiert, weil ein neues Interface keine Freigaben hat. Erst nach einer eigenen Regel (nur ICMP Echo Request, nur zur Firewall-Adresse) kamen Antworten.

![Blockierter Ping](/docs/Github/day2-default-deny-blocked.png)
![Freigegebener Ping](/docs/Github/day2-ping-allowed.png)

**Zugriff vom Client:** Kali konnte den Server zuerst über die Standardregel "alles erlaubt" im LAN erreichen. Ich habe eine Regel ergänzt, die nur SSH (Port 22) von den Clients zu 10.0.20.10 erlaubt, und die Allow-All-Regel deaktiviert. Danach funktioniert SSH weiter, der Ping zum selben Server wird geblockt.

![SSH-Login von Kali auf den Server](/docs/Github/day2-ssh-login.png)

**Bekannte Lücke:** Kali kann jetzt keine DNS-Anfragen mehr an die Firewall stellen, weil dafür noch keine Regel existiert.
