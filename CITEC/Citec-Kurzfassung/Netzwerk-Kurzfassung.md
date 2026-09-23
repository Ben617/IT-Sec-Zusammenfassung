# Netzwerk Kurzfassung

## Netzwerkarten
Netzwerkarten ermöglichen Geräten die Kommunikation und unterscheiden sich beispielsweise durch ihre Reichweite, Übertragungsart sowie die Verwendung von MAC- und IP-Adressen.
## Netzwerkkomponenten
## Netzwerkmodelle
- OSI-Modell

- TCP/IP-Modell

## MAC-Adressen
Eine MAC-Adresse ist die eindeutige Hardwareadresse einer Netzwerkschnittstelle, die zur Identifikation innerhalb eines lokalen Netzwerks dient.
## IP-Adressen
- v4
- v6

## Subnetze und Subnetzmasken

Ein Subnetz teilt ein großes Netzwerk in kleinere Teilnetze auf, während die Subnetzmaske bestimmt, welcher Teil einer IP-Adresse das Netzwerk und welcher Teil das Gerät kennzeichnet.

Beispiel:

```text
IP-Adresse:     192.168.1.25
Subnetzmaske:   255.255.255.0
CIDR-Schreibweise: /24
Netzwerkadresse: 192.168.1.0
Hostbereich:     192.168.1.1 – 192.168.1.254
Broadcastadresse: 192.168.1.255
```

Netzwerkkonfiguration unter Windows anzeigen:

```powershell
ipconfig
```

Netzwerkkonfiguration unter Linux anzeigen:

```bash
ip address
```


## Ports und Protokolle

Protokolle legen die Regeln für die Kommunikation zwischen Geräten fest, während Ports die Daten dem richtigen Programm oder Netzwerkdienst zuordnen.

* `20/21` – FTP
* `22` – SSH und SFTP
* `25` – SMTP
* `53` – DNS
* `67/68` – DHCP
* `80` – HTTP
* `110` – POP3
* `123` – NTP
* `143` – IMAP
* `443` – HTTPS
* `445` – SMB
* `3389` – RDP

Offene Ports unter Linux anzeigen:

```bash
ss -tulpen
```

Offene Ports unter Windows anzeigen:

```powershell
netstat -ano
```


## Standardgateway
Das Standardgateway ist normalerweise der Router, an den ein Gerät Daten sendet, wenn sich das Ziel außerhalb des eigenen Netzwerks befindet.

## Ports und Protokolle

## Standardgateway ermitteln

Unter Windows:

```powershell
ipconfig
```

Das Standardgateway steht bei **Standardgateway**. 
Alternativ mit PowerShell:

```powershell
Get-NetRoute -DestinationPrefix "0.0.0.0/0"
```

Unter Linux:

```bash
ip route
```

Die Adresse hinter `default via` ist das Standardgateway.





---
## Wichtige Netzwerkprotokolle

### Ethernet

### ARP

### ICMP

### TCP und UDP

### DHCP

### DNS

### HTTP und HTTPS

### SSH

### FTP und SFTP

### SMTP, IMAP und POP3



---
## Switching

### Switch und MAC-Adresstabelle

### VLAN

### Trunking

### Spanning Tree Protocol



---
## Routing

### Router und Routingtabellen

### Statisches Routing

### Dynamisches Routing

### NAT und PAT



---
## WLAN

### WLAN-Standards

### Frequenzbereiche und Kanäle

### WPA2 und WPA3



---
## Netzwerksicherheit

### Firewall

### VPN

### Proxyserver

### Netzwerksegmentierung

### Zugriffskontrolle



---
## Netzwerkdienste

### DNS-Server

### DHCP-Server

### Webserver

### Datei- und Druckserver



---
## Netzwerkbefehle unter Linux

## Netzwerkbefehle unter Windows

## Netzwerküberwachung

## Fehlersuche

## Häufige Netzwerkprobleme

## Beispiel einer Netzwerkkonfiguration
