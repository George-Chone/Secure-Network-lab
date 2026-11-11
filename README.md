# Secure Network Lab

Segmentiertes Firmennetzwerk in VirtualBox: eine OPNsense-Firewall trennt Clients, Server und DMZ. Ich baue es, um Netzwerksegmentierung, Firewall-Regeln, Systemhärtung und Monitoring praktisch zu lernen. Mein Ziel ist eine Ausbildung zum Fachinformatiker für Systemintegration mit Schwerpunkt IT-Sicherheit.

## Aufbau

| Segment | Zweck | Netz | Firewall-Adresse |
|---|---|---|---|
| WAN | Internetzugang (VirtualBox NAT) | 10.0.2.0/24 | 10.0.2.15 |
| Clients | Arbeitsplätze (Kali Linux als Testrechner) | 10.0.10.0/24 | 10.0.10.1 |
| Server | interne Server | 10.0.20.0/24 | 10.0.20.1 |
| DMZ | extern erreichbare Systeme | 10.0.30.0/24 | 10.0.30.1 |

## Geplante Schwerpunkte

- Firewall-Regeln nach dem Prinzip "alles verboten, nur Nötiges erlaubt"
- Gehärteter Linux-Server (SSH-Schlüssel, kein Root-Login, Updates)
- Monitoring mit Wazuh und ein nachgestellter Angriff aus dem Clients-Netz
- Einordnung nach BSI IT-Grundschutz
