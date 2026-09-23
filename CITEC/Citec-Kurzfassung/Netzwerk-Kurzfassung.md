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

IP-Adressen identifizieren Geräte in einem Netzwerk und ermöglichen die Kommunikation zwischen ihnen.

* IPv4:

  IPv4-Adressen bestehen aus 32 Bit und werden als vier Dezimalzahlen dargestellt.

  ```text
  192.168.1.25
  ```

  IPv4 verwendet **ARP (Address Resolution Protocol)**, um zu einer bekannten IPv4-Adresse die entsprechende MAC-Adresse im lokalen Netzwerk zu ermitteln.

  ```bash
  ip neighbor
  ```

* IPv6:

  IPv6-Adressen bestehen aus 128 Bit und werden hexadezimal dargestellt.

  ```text
  2001:db8:1234::25
  ```

  IPv6 verwendet **NDP (Neighbor Discovery Protocol)** zur Ermittlung von MAC-Adressen sowie zur Erkennung von Nachbarn, Routern und Netzwerkparametern.

  ```bash
  ip -6 neighbor
  ```

* IP-Adressen unter Windows anzeigen:

  ```powershell
  ipconfig
  ```

* IP-Adressen unter Linux anzeigen:

  ```bash
  ip address
  ```


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


- Standardgateway ermitteln

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

Ethernet überträgt Daten innerhalb eines lokalen, meist kabelgebundenen Netzwerks und verwendet MAC-Adressen zur Identifikation der Netzwerkschnittstellen.

### ARP

ARP ermittelt im lokalen IPv4-Netzwerk die MAC-Adresse, die zu einer bekannten IPv4-Adresse gehört.

### ICMP

ICMP übermittelt Kontroll- und Fehlermeldungen und wird beispielsweise vom Befehl `ping` zur Überprüfung der Erreichbarkeit verwendet.

```bash
ping IP-ADRESSE
```

### TCP und UDP

TCP baut eine zuverlässige und verbindungsorientierte Verbindung auf, während UDP Daten schneller, aber ohne Empfangsbestätigung überträgt.

* TCP: zuverlässig, verbindungsorientiert und mit Fehlerkontrolle
* UDP: schnell, verbindungslos und ohne Zustellgarantie

### DHCP

DHCP weist Geräten automatisch Netzwerkeinstellungen wie IP-Adresse, Subnetzmaske, Standardgateway und DNS-Server zu.

* UDP-Port `67`: DHCP-Server
* UDP-Port `68`: DHCP-Client

### DNS

DNS übersetzt Domainnamen wie `example.com` in IP-Adressen und verwendet normalerweise Port `53`.

```bash
nslookup example.com
```

### HTTP und HTTPS

HTTP überträgt Webseiten unverschlüsselt über Port `80`, während HTTPS die Verbindung verschlüsselt über Port `443` aufbaut.

### SSH

SSH ermöglicht den verschlüsselten Fernzugriff auf andere Computer und verwendet normalerweise Port `22`.

```bash
ssh BENUTZERNAME@IP-ADRESSE
```

### FTP und SFTP

FTP überträgt Dateien normalerweise unverschlüsselt über die Ports `20` und `21`, während SFTP die Dateiübertragung verschlüsselt über SSH und Port `22` durchführt.

### SMTP, IMAP und POP3

SMTP dient zum Versenden von E-Mails, während IMAP und POP3 zum Abrufen von E-Mails verwendet werden.

* SMTP: Port `25`, `465` oder `587`
* IMAP: Port `143`, verschlüsselt Port `993`
* POP3: Port `110`, verschlüsselt Port `995`



---
## Switching

Switching beschreibt die Weiterleitung von Ethernet-Frames innerhalb eines lokalen Netzwerks anhand von MAC-Adressen.

### Switch und MAC-Adresstabelle

Ein Switch verbindet Geräte innerhalb eines LANs und speichert in seiner MAC-Adresstabelle, welche MAC-Adresse über welchen Anschluss erreichbar ist.

Ist die Zieladresse unbekannt, sendet der Switch den Frame zunächst an alle passenden Anschlüsse, außer an den Eingangsport.

### VLAN

Ein VLAN unterteilt einen physischen Switch in mehrere logisch getrennte Netzwerke, wodurch Sicherheit und Übersichtlichkeit verbessert werden.

Geräte in unterschiedlichen VLANs benötigen einen Router oder Layer-3-Switch, um miteinander zu kommunizieren.

### Trunking

Ein Trunk überträgt den Datenverkehr mehrerer VLANs über eine gemeinsame Verbindung und kennzeichnet die Frames normalerweise nach dem Standard IEEE 802.1Q mit einer VLAN-ID.

Trunks werden häufig zwischen Switches oder zwischen einem Switch und einem Router verwendet.

### Spanning Tree Protocol

Das Spanning Tree Protocol verhindert Schleifen zwischen Switches, indem es redundante Verbindungen erkennt und ausgewählte Verbindungswege vorübergehend blockiert.

Fällt eine aktive Verbindung aus, kann STP eine zuvor blockierte Ersatzverbindung freigeben.




---
## Routing

Routing bezeichnet die Weiterleitung von Datenpaketen zwischen verschiedenen Netzwerken anhand ihrer IP-Adressen.

### Router und Routingtabellen

Ein Router verbindet unterschiedliche Netzwerke und verwendet eine Routingtabelle, um den besten Weg zum Zielnetzwerk zu bestimmen.

Routingtabelle unter Linux anzeigen:

```bash
ip route
```

Routingtabelle unter Windows anzeigen:

```powershell
route print
```

### Statisches Routing

Beim statischen Routing werden Routen manuell eingetragen und ändern sich nicht automatisch, wenn sich das Netzwerk verändert.

Statische Route unter Linux hinzufügen:

```bash
sudo ip route add ZIELNETZ via GATEWAY
```

Beispiel:

```bash
sudo ip route add 192.168.2.0/24 via 192.168.1.1
```

### Dynamisches Routing

Beim dynamischen Routing tauschen Router mithilfe von Routingprotokollen automatisch Informationen über erreichbare Netzwerke aus und passen ihre Routingtabellen an.

Wichtige dynamische Routingprotokolle sind:

* RIP
* OSPF
* IS-IS
* BGP

### NAT und PAT

NAT übersetzt private IP-Adressen in öffentliche IP-Adressen, während PAT zusätzlich Portnummern verwendet, damit mehrere Geräte gleichzeitig dieselbe öffentliche IPv4-Adresse nutzen können.

* NAT: Übersetzung zwischen privaten und öffentlichen IP-Adressen
* PAT: Unterscheidung mehrerer Verbindungen über Portnummern
* Typischer Einsatz: Internetzugang für Geräte in einem privaten Netzwerk




---
## WLAN

### WLAN-Standards

### Frequenzbereiche und Kanäle

### WPA2 und WPA3



---
## Netzwerksicherheit

Netzwerksicherheit umfasst Maßnahmen, die Netzwerke und übertragene Daten vor unbefugtem Zugriff, Manipulation und Ausfällen schützen.

### Firewall

Eine Firewall überwacht den Netzwerkverkehr und erlaubt oder blockiert Verbindungen anhand festgelegter Regeln, beispielsweise nach IP-Adresse, Port und Protokoll.

Firewallstatus unter Linux anzeigen:

```bash
sudo ufw status
```

Firewall unter Linux aktivieren:

```bash
sudo ufw enable
```

### VPN

Ein VPN stellt über ein unsicheres Netzwerk eine verschlüsselte Verbindung her und ermöglicht beispielsweise den geschützten Zugriff auf ein internes Firmennetzwerk.

Häufig verwendete VPN-Techniken sind:

* IPsec
* WireGuard
* OpenVPN

### Proxyserver

Ein Proxyserver nimmt Anfragen stellvertretend für andere Geräte entgegen und kann Zugriffe filtern, Inhalte zwischenspeichern oder die interne IP-Adresse verbergen.

Ein Reverse Proxy nimmt hingegen Anfragen aus dem Netzwerk entgegen und leitet sie an interne Server weiter.

### Netzwerksegmentierung

Bei der Netzwerksegmentierung wird ein großes Netzwerk in kleinere, getrennte Bereiche aufgeteilt, beispielsweise durch Subnetze, VLANs oder Firewalls.

Dadurch werden Netzwerkverkehr und Zugriffsrechte besser kontrolliert und Angriffe können sich schwerer im gesamten Netzwerk ausbreiten.

### Zugriffskontrolle

Die Zugriffskontrolle legt fest, welche Benutzer und Geräte auf bestimmte Netzwerke, Systeme oder Dienste zugreifen dürfen.

Mögliche Maßnahmen sind:

* Benutzername und Passwort
* Mehrfaktor-Authentifizierung
* Zugriffslisten, kurz ACL
* Rollen und Berechtigungen
* Zertifikate
* Port-Security
* Network Access Control, kurz NAC




---
## Netzwerkdienste

Netzwerkdienste stellen anderen Geräten bestimmte Funktionen und Ressourcen über ein Netzwerk zur Verfügung.

### DNS-Server

Ein DNS-Server übersetzt Domain- und Hostnamen in IP-Adressen und ermöglicht dadurch den Zugriff auf Geräte und Webseiten über leicht lesbare Namen.

Beispiel:

```text
example.com → 93.184.216.34
```

DNS-Abfrage durchführen:

```bash
nslookup example.com
```

### DHCP-Server

Ein DHCP-Server vergibt automatisch Netzwerkeinstellungen an Clients, beispielsweise:

* IP-Adresse
* Subnetzmaske
* Standardgateway
* DNS-Server
* Gültigkeitsdauer der Zuweisung

Diese zeitlich begrenzte Zuweisung wird als **Lease** bezeichnet.

### Webserver

Ein Webserver stellt Webseiten und Webanwendungen über HTTP oder HTTPS bereit.

Häufig verwendete Webserver sind:

* Apache HTTP Server
* Nginx
* Microsoft IIS

HTTP verwendet normalerweise Port `80`, HTTPS verwendet Port `443`.

### Datei- und Druckserver

Ein Dateiserver stellt Dateien und Verzeichnisse zentral im Netzwerk bereit, während ein Druckserver Drucker verwaltet und Druckaufträge an sie weiterleitet.

Häufig verwendete Dienste sind:

* SMB für Windows-Netzwerke
* NFS für Linux- und UNIX-Systeme
* CUPS zur Verwaltung von Druckern unter Linux




---
## Netzwerkbefehle unter Linux

* IP-Adressen und Netzwerkschnittstellen anzeigen:

  ```bash
  ip address
  ```

* Kurze Übersicht anzeigen:

  ```bash
  ip -br address
  ```

* Routingtabelle und Standardgateway anzeigen:

  ```bash
  ip route
  ```

* Erreichbarkeit eines Geräts prüfen:

  ```bash
  ping HOSTNAME
  ```

* Route zum Ziel verfolgen:

  ```bash
  traceroute HOSTNAME
  ```

* DNS-Auflösung überprüfen:

  ```bash
  nslookup HOSTNAME
  ```

* Ausführliche DNS-Abfrage durchführen:

  ```bash
  dig HOSTNAME
  ```

* Offene Ports und aktive Verbindungen anzeigen:

  ```bash
  ss -tulpen
  ```

* ARP- und NDP-Nachbartabelle anzeigen:

  ```bash
  ip neighbor
  ```

* Netzwerkverbindungen des NetworkManagers anzeigen:

  ```bash
  nmcli connection show
  ```

* Hostname anzeigen:

  ```bash
  hostnamectl
  ```

* Öffentliche IP-Adresse abfragen:

  ```bash
  curl ifconfig.me
  ```

## Netzwerkbefehle unter Windows

* IP-Konfiguration anzeigen:

  ```powershell
  ipconfig
  ```

* Ausführliche IP-Konfiguration anzeigen:

  ```powershell
  ipconfig /all
  ```

* Erreichbarkeit eines Geräts prüfen:

  ```powershell
  ping HOSTNAME
  ```

* Route zum Ziel verfolgen:

  ```powershell
  tracert HOSTNAME
  ```

* DNS-Auflösung überprüfen:

  ```powershell
  nslookup HOSTNAME
  ```

* Routingtabelle anzeigen:

  ```powershell
  route print
  ```

* Aktive Verbindungen und offene Ports anzeigen:

  ```powershell
  netstat -ano
  ```

* ARP-Tabelle anzeigen:

  ```powershell
  arp -a
  ```

* DNS-Zwischenspeicher anzeigen:

  ```powershell
  ipconfig /displaydns
  ```

* DNS-Zwischenspeicher leeren:

  ```powershell
  ipconfig /flushdns
  ```

* DHCP-Konfiguration erneuern:

  ```powershell
  ipconfig /release
  ipconfig /renew
  ```

* Netzwerkverbindung ausführlich testen:

  ```powershell
  Test-NetConnection HOSTNAME
  ```


## Netzwerküberwachung

Bei der Netzwerküberwachung werden Verbindungen, Datenverkehr, Auslastung und Fehler kontrolliert, um Probleme oder Angriffe frühzeitig zu erkennen.

* Netzwerkschnittstellen und Statistik anzeigen:

  ```bash
  ip -s link
  ```

* Offene Ports und aktive Verbindungen anzeigen:

  ```bash
  ss -tulpen
  ```

* Netzwerkverkehr in Echtzeit untersuchen:

  ```bash
  sudo tcpdump -i SCHNITTSTELLE
  ```

* Datenverkehr grafisch analysieren:

  ```text
  Wireshark
  ```

* Typische Überwachungswerte:

  * Bandbreite und Auslastung
  * Paketverluste
  * Antwortzeiten
  * Fehlerraten
  * Verfügbarkeit von Geräten und Diensten
  * Ungewöhnliche Netzwerkverbindungen

## Fehlersuche

Netzwerkfehler sollten schrittweise von der physischen Verbindung bis zum betroffenen Netzwerkdienst untersucht werden.

1. Kabel, WLAN und Netzwerkschnittstelle prüfen:

   ```bash
   ip link
   ```

2. IP-Adresse und Subnetzmaske prüfen:

   ```bash
   ip address
   ```

3. Standardgateway prüfen:

   ```bash
   ip route
   ```

4. Eigene Netzwerkschnittstelle testen:

   ```bash
   ping 127.0.0.1
   ```

5. Standardgateway testen:

   ```bash
   ping GATEWAY-ADRESSE
   ```

6. Internetverbindung ohne DNS testen:

   ```bash
   ping 1.1.1.1
   ```

7. DNS-Auflösung testen:

   ```bash
   nslookup example.com
   ```

8. Route zum Ziel untersuchen:

   ```bash
   traceroute example.com
   ```

9. Erreichbarkeit eines Ports testen:

   ```bash
   nc -vz HOSTNAME PORT
   ```

## Häufige Netzwerkprobleme

* Keine IP-Adresse:

  Möglicherweise ist der DHCP-Server nicht erreichbar oder die Netzwerkkonfiguration ist fehlerhaft.

* Falsche Subnetzmaske:

  Geräte im gleichen Netzwerk können fälschlicherweise als entfernte Geräte behandelt werden.

* Falsches oder fehlendes Standardgateway:

  Das lokale Netzwerk funktioniert, aber andere Netzwerke oder das Internet sind nicht erreichbar.

* DNS-Fehler:

  IP-Adressen sind erreichbar, aber Domainnamen können nicht aufgelöst werden.

* Doppelte IP-Adresse:

  Zwei Geräte verwenden dieselbe IP-Adresse und verursachen Verbindungsprobleme.

* Firewall blockiert Verbindung:

  Ein benötigter Port oder ein Protokoll wird durch eine Firewallregel blockiert.

* Netzwerkdienst nicht verfügbar:

  Der benötigte Dienst wurde gestoppt, ist falsch konfiguriert oder verwendet einen anderen Port.

* Kabel- oder WLAN-Probleme:

  Defekte Kabel, schwaches WLAN, Störungen oder eine deaktivierte Netzwerkschnittstelle verhindern die Verbindung.

* Falsche VLAN-Zuordnung:

  Das Gerät befindet sich im falschen logischen Netzwerk und kann die gewünschten Systeme nicht erreichen.

## Beispiel einer Netzwerkkonfiguration

Ein Gerät erhält folgende statische IPv4-Konfiguration:

```text
IP-Adresse:       192.168.10.25
Subnetzmaske:     255.255.255.0
CIDR-Präfix:      /24
Netzwerkadresse:  192.168.10.0
Standardgateway:  192.168.10.1
DNS-Server:       192.168.10.1
Alternativer DNS: 1.1.1.1
```

Bei einem `/24`-Netz gilt:

```text
Nutzbarer Bereich:  192.168.10.1 bis 192.168.10.254
Broadcastadresse:   192.168.10.255
```

Temporäre Konfiguration unter Linux:

```bash
sudo ip address add 192.168.10.25/24 dev eth0
sudo ip route add default via 192.168.10.1
```

DNS wird abhängig von der Distribution beispielsweise über NetworkManager oder `systemd-resolved` eingerichtet.

Konfiguration überprüfen:

```bash
ip address
ip route
ping 192.168.10.1
nslookup example.com
```

