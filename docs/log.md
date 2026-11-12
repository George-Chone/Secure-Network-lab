## Tag 1: Firewall und erstes Segment (Oktober)

Ich habe OPNsense in einer neuen VirtualBox-VM installiert (FreeBSD 64-bit, 4 GB RAM, 2 CPUs, 16 GB Disk). Die VM hat vier Netzwerkadapter: einmal NAT für das Internet und drei interne Netze für Clients, Server und DMZ. Zugewiesen sind em0 = WAN, em1 = LAN (Clients), em2 = Server, em3 = DMZ.

Das LAN hat 10.0.10.1/24 mit DHCP von .100 bis .200. Meine Kali-VM hängt in diesem Netz und hat 10.0.10.200 bekommen. Der Ping auf die Firewall läuft ohne Paketverlust.

![Kali: IP-Adresse und Ping auf die Firewall](/docs/Github-screenshots/day1-ip-und-ping.png)

**Probleme**
- Der Installer wollte mehr RAM (ca. 3 GB). Ich habe die VM von 2 auf 4 GB erhöht.
- Nach der Installation musste ich die ISO aus dem virtuellen Laufwerk entfernen, sonst startet wieder der Installer.
- Bei der Interface-Zuweisung habe ich zu schnell bestätigt und sie wiederholt. Übernommen wird erst nach der letzten Abfrage.

## Tag 2: Server-Segment und erste Firewall-Regeln (10./11. November)

Der Setup-Wizard hat Hostname und DNS nicht gespeichert. Ich habe beides unter System → Settings → General von Hand gesetzt (fw-lab01, lab.local). Danach habe ich die zwei übrigen Interfaces konfiguriert: OPT1 als Server-Segment (10.0.20.1/24) und OPT2 als DMZ (10.0.30.1/24). Beide Adressen sind statisch, damit die Firewall für die Geräte im Segment immer unter derselben Adresse als Gateway erreichbar ist.

![Hostname der Firewall](/docs/Github-screenshots/day2-hostname.png)
![Interfaces auf dem Dashboard](/docs/Github-screenshots/day-2-interface.png)

Dann habe ich einen Ubuntu-Server (srv-linux01) installiert, aktualisiert und mit fester Adresse (10.0.20.10) ins Server-Segment gehängt. Die Netzwerkkonfiguration ist eine YAML-Datei, bei der die Einrückung zählt. Mit `netplan try` habe ich sie getestet, bevor ich sie übernommen habe.

**Default deny:** Der erste Ping vom Server zur Firewall wurde geblockt, weil ein neues Interface keine Freigaben hat. Erst eine eigene Regel (nur ICMP Echo, nur an die Firewall-Adresse) hat ihn durchgelassen.

![Blockierter Ping](/docs/Github-screenshots/day2-default-deny-blocked.png)
![Freigegebener Ping](/docs/Github-screenshots/day2-ping-allowed.png)

**Zugriff vom Client:** Im LAN erlaubte die Standardregel zuerst alles, Kali kam also überall hin. Ich habe eine Regel angelegt, die nur SSH (Port 22) von den Clients zu 10.0.20.10 erlaubt, und die Allow-All-Regel für IPv4 deaktiviert. SSH funktioniert weiter, ein Ping zum selben Server wird jetzt geblockt.

![Firewall-Log: SSH-Regel greift](/docs/Github-screenshots/day2-ssh-rule-pass.png)

## Tag 3: DNS, SSH-Härtung und DMZ (12. November)

**DNS:** Nach dem Abschalten der Allow-All-Regel konnte Kali keine Namen mehr auflösen. Ich habe eine Regel ergänzt, die DNS (Port 53) nur an die Firewall erlaubt. Alles andere ins Internet bleibt geblockt.

![DNS-Abfrage über die Firewall](/docs/Github-screenshots/day2-dns-works.png)

**SSH-Härtung auf srv-linux01:** Ich habe auf Kali ein ed25519-Schlüsselpaar erzeugt und den öffentlichen Schlüssel mit `ssh-copy-id` auf den Server kopiert. Danach habe ich Passwort-Login und Root-Login abgeschaltet, aber erst, nachdem der Schlüssel-Login funktioniert hat.

Eine Änderung in `/etc/ssh/sshd_config` allein hätte nicht gereicht, weil `50-cloud-init.conf` `PasswordAuthentication yes` setzt und zuerst gelesen wird. Deshalb habe ich `10-hardening.conf` angelegt. Die Dateien werden alphabetisch gelesen, mein Wert wird also zuerst gefunden. Getestet habe ich in einer zweiten Sitzung, damit ich mich nicht aussperre. Ein Login mit Passwort wird jetzt mit `Permission denied (publickey)` abgelehnt.

![Passwort-Login wird abgelehnt](/docs/Github-screenshots/day2-password-denied.png)

**DMZ:** Für den Webserver habe ich meinen Ubuntu-Server geklont und `dmz-web01` genannt, mit neuen MAC-Adressen, damit es keinen Konflikt gibt. Hostname und Netplan habe ich angepasst (10.0.30.10/24, Gateway 10.0.30.1) und die Netzwerkkarte auf `net-dmz` gestellt. Wie schon im Server-Segment wurde der erste Ping geblockt, bis eine Regel nur für ICMP Echo an die Firewall-Adresse ihn erlaubt hat.

![Ping aus der DMZ wird erlaubt](/docs/Github-screenshots/day3-dmz-ping-allowed.png)

Zum Installieren von nginx hatte die DMZ kurz Internet. Dafür habe ich drei temporäre Regeln angelegt: DNS zur Firewall sowie HTTP und HTTPS ins Internet, bei den Web-Regeln mit dem Ziel "alles außer 10.0.0.0/8", damit die DMZ auch währenddessen nicht an meine internen Netze kommt. Danach habe ich alle drei wieder deaktiviert, aber nicht gelöscht, damit ich sie für spätere Updates wieder einschalten kann. Aus der DMZ gibt es keinen Weg ins Internet mehr.

![nginx läuft auf dmz-web01](/docs/Github-screenshots/day3-nginx-running2.png)
![Temporäre Regeln sind aus](/docs/Github-screenshots/day3-temp-rules-off.png)

**Als Nächstes:** eine Regel, damit die Clients die Webseite auf dmz-web01 aufrufen dürfen, und ein Test, dass die DMZ keine Verbindung ins Server- oder Clients-Netz aufbauen kann.
