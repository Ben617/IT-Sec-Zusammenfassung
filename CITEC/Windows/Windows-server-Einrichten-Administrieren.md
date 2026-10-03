# Windows Server einrichten und administrieren

## 1. Testumgebung vorbereiten
- Netzwerkplan und IP-Adressen festlegen
- Virtuelle Maschinen für Server und Windows-Client anlegen

## 2. Windows Server installieren

### 2.1 Edition und Installationsart auswählen

Starte die virtuelle Maschine oder den Server vom Windows-Server-Installationsmedium.

1. Wähle Sprache, Uhrzeitformat und Tastaturlayout.
2. Starte die Installation.
3. Wähle die Edition und die Installationsart.
4. Wähle die benutzerdefinierte Installation.
5. Wähle den vorgesehenen Datenträger und starte die Installation.

Für eine Testumgebung eignet sich **Windows Server Standard mit Desktopdarstellung**.

| Auswahl | Beschreibung |
|---|---|
| Standard | Für grundlegende Serverrollen wie Active Directory und DNS ausreichend |
| Datacenter | Zusätzliche Funktionen und umfangreichere Virtualisierungsrechte |
| Desktopdarstellung | Grafische Oberfläche und PowerShell |
| Server Core | Ohne vollständige Desktopoberfläche; Verwaltung über Konsole und Remote-Werkzeuge |

> **Hinweis:** Edition und Installationsart sind unterschiedliche Entscheidungen. Sowohl Standard als auch Datacenter sind mit Desktopdarstellung oder als Server Core verfügbar.

Öffne nach der Installation **Windows PowerShell als Administrator** und prüfe das installierte Betriebssystem:

```powershell
Get-ComputerInfo |
    Select-Object WindowsProductName, OsName, OsVersion
```

### 2.2 Administrator-Kennwort festlegen und lokales Konto verwenden

Beim ersten Start legst du ein sicheres Kennwort für das integrierte Konto `Administrator` fest.

Der Server gehört zunächst keiner Domäne an. Die erste Anmeldung erfolgt mit dem lokalen Administrator:

```text
.\Administrator
```

Der Punkt `.` steht für den lokalen Computer.

Aktuell angemeldetes Konto anzeigen:

```powershell
whoami
```

Lokale Benutzerkonten anzeigen:

```powershell
Get-LocalUser
```

Das Kennwort des lokalen Administrators kannst du später folgendermaßen ändern:

```powershell
$Kennwort = Read-Host "Neues Administrator-Kennwort" -AsSecureString

Set-LocalUser -Name "Administrator" -Password $Kennwort
```

Das Kennwort wird verdeckt eingegeben und nicht als Klartext im Befehl hinterlegt.

#### Optional: Zusätzliches lokales Administratorkonto erstellen

Auf einem eigenständigen Server oder Mitgliedsserver kann ein zusätzliches lokales Konto für Wartungsarbeiten verwendet werden:

```powershell
$Kennwort = Read-Host "Kennwort für das Wartungskonto" -AsSecureString

New-LocalUser `
    -Name "wartung" `
    -Password $Kennwort `
    -Description "Lokales Konto für Wartungsarbeiten"
```

Füge das Konto der lokalen Administratorengruppe hinzu:

```powershell
$AdminGruppe = Get-LocalGroup -SID "S-1-5-32-544"

Add-LocalGroupMember `
    -Group $AdminGruppe `
    -Member "$env:COMPUTERNAME\wartung"
```

Die SID `S-1-5-32-544` bezeichnet die lokale Administratorengruppe unabhängig von der Windows-Anzeigesprache.

Gruppenmitgliedschaft prüfen:

```powershell
Get-LocalGroupMember -Group $AdminGruppe
```

Die Anmeldung mit diesem Konto erfolgt über:

```text
.\wartung
```

> **Wichtig:** Nach der Hochstufung zum Domänencontroller stehen normale lokale Benutzerkonten im regulären Betrieb nicht mehr zur Verfügung. Der Domänencontroller wird dann mit Domänenkonten administriert. Ein zusätzliches lokales Konto ist daher kein dauerhaftes Wartungskonto für einen Domänencontroller.

### 2.3 Updates installieren

Installiere vor der Einrichtung weiterer Serverrollen die verfügbaren Windows-Updates.

#### Mit Desktopdarstellung

Öffne Windows Update:

```powershell
Start-Process "ms-settings:windowsupdate"
```

1. Wähle **Nach Updates suchen**.
2. Installiere die verfügbaren Updates.
3. Starte den Server neu, wenn dies erforderlich ist.

Ein Neustart kann über PowerShell ausgelöst werden:

```powershell
Restart-Computer
```

> Speichere vorher offene Arbeiten. Der Befehl startet den Computer neu.

#### Mit Server Core

Starte das Konfigurationswerkzeug:

```powershell
SConfig
```

Wähle **Option 6**, um Updates zu suchen und zu installieren. Folge anschließend den angezeigten Auswahlmöglichkeiten.

#### Installation prüfen

Zeige die zuletzt installierten Hotfixes an:

```powershell
Get-HotFix |
    Sort-Object InstalledOn -Descending |
    Select-Object -First 10 HotFixID, InstalledOn
```

Prüfe nach einem Neustart erneut auf ausstehende Updates. Die Hotfix-Liste allein bestätigt nicht, dass das System vollständig aktualisiert ist.

## 3. Grundeinstellungen vornehmen

Die folgenden Befehle verwenden diese Beispielkonfiguration:

| Einstellung | Beispielwert |
|---|---|
| Servername | `SRV-DC01` |
| Netzwerkadapter | `Ethernet` |
| IPv4-Adresse | `192.168.10.10` |
| Präfixlänge | `24` |
| Subnetzmaske | `255.255.255.0` |
| Standardgateway | `192.168.10.1` |
| Späterer interner DNS-Server | `192.168.10.10` |

Passe die Werte an deine Umgebung an. Die Serveradresse muss frei sein und darf nicht anderweitig durch DHCP vergeben werden.

Führe alle Konfigurationsbefehle in **Windows PowerShell als Administrator** aus.

### 3.1 Server umbenennen

Ein eindeutiger Servername erleichtert die Verwaltung. Im Beispiel bezeichnet `SRV-DC01` den vorgesehenen ersten Domänencontroller.

Aktuellen Computernamen anzeigen:

```powershell
hostname
```

Server umbenennen und anschließend neu starten:

```powershell
Rename-Computer -NewName "SRV-DC01" -Restart
```

Melde dich nach dem Neustart erneut mit dem lokalen Konto an:

```text
.\Administrator
```

Alternativ kannst du den Computernamen ausdrücklich angeben:

```text
SRV-DC01\Administrator
```

Neuen Namen überprüfen:

```powershell
hostname
```

Erwartete Ausgabe:

```text
SRV-DC01
```

> Lege den endgültigen Servernamen vor der Einrichtung von Active Directory fest.

### 3.2 Statische IP-Adresse und DNS konfigurieren

Ein Server soll unter einer gleichbleibenden IP-Adresse erreichbar sein.

#### Netzwerkadapter prüfen

Zeige zunächst die vorhandenen Netzwerkadapter an:

```powershell
Get-NetAdapter
```

Prüfe die aktuelle Netzwerkkonfiguration:

```powershell
Get-NetIPConfiguration
```

Übernimm den tatsächlichen Adapternamen in die folgenden Befehle. Im Beispiel heißt der Adapter `Ethernet`.

> Führe Netzwerkänderungen direkt über die Server- oder VM-Konsole aus. Eine bestehende Remoteverbindung kann dabei abbrechen.

#### Statische IPv4-Adresse vergeben

Die folgenden Befehle setzen eine bisher per DHCP konfigurierte Netzwerkkarte voraus:

```powershell
$Adapter = "Ethernet"

New-NetIPAddress `
    -InterfaceAlias $Adapter `
    -IPAddress "192.168.10.10" `
    -PrefixLength 24 `
    -DefaultGateway "192.168.10.1"
```

Bedeutung der Parameter:

| Parameter | Bedeutung |
|---|---|
| `InterfaceAlias` | Name des Netzwerkadapters |
| `IPAddress` | Statische IP-Adresse des Servers |
| `PrefixLength` | Größe des Netzwerks; `24` entspricht `255.255.255.0` |
| `DefaultGateway` | Router für die Kommunikation mit anderen Netzen |

DHCP wird beim Anlegen der statischen IPv4-Adresse automatisch deaktiviert.

In einem isolierten Testnetz ohne Router lässt du den Parameter `-DefaultGateway` weg.

> **Hinweis:** Ist bereits eine statische Adresse eingerichtet, müssen vorhandene Adressen und Routen zuerst geprüft und gezielt angepasst werden. Führe `New-NetIPAddress` nicht mehrfach unverändert aus.

#### DNS-Server festlegen

Der richtige DNS-Server hängt vom geplanten Einsatz ab:

- **Vorhandene Domäne:** Verwende den internen DNS-Server dieser Domäne.
- **Erster Domänencontroller einer neuen Domäne:** Verwende unmittelbar vor der AD-/DNS-Installation die eigene Serveradresse.
- **Noch keine DNS-Rolle eingerichtet:** Behalte für Updates und Downloads zunächst einen erreichbaren DNS-Server bei.

Für den geplanten ersten Domänencontroller setzt du:

```powershell
Set-DnsClientServerAddress `
    -InterfaceAlias "Ethernet" `
    -ServerAddresses "192.168.10.10"
```

> Unter `192.168.10.10` funktioniert die Namensauflösung erst, sobald der DNS-Dienst eingerichtet wurde. Installiere benötigte Updates daher vorher.

#### Konfiguration überprüfen

```powershell
Get-NetIPConfiguration

Get-DnsClientServerAddress `
    -InterfaceAlias "Ethernet" `
    -AddressFamily IPv4
```

Alternativ kannst du die vollständige Konfiguration anzeigen:

```powershell
ipconfig /all
```

Wenn ein Gateway vorhanden ist, prüfe seine Erreichbarkeit:

```powershell
Test-Connection -ComputerName "192.168.10.1" -Count 4
```

Eine fehlende Antwort bedeutet nicht zwingend, dass die Verbindung gestört ist: Eine Firewall kann Ping-Anfragen blockieren.

### 3.3 Zeitzone prüfen

Die korrekte Uhrzeit ist unter anderem für Protokolle und spätere Domänenanmeldungen wichtig.

Aktuelles Datum und aktuelle Uhrzeit anzeigen:

```powershell
Get-Date
```

Eingestellte Zeitzone anzeigen:

```powershell
Get-TimeZone
```

Zeitzone für Deutschland setzen:

```powershell
Set-TimeZone -Id "W. Europe Standard Time"
```

Ergebnis überprüfen:

```powershell
Get-TimeZone

Get-Date
```

Die Umstellung zwischen Sommer- und Winterzeit erfolgt automatisch.

> **Unterschied:** Die Zeitzone bestimmt die angezeigte Ortszeit. Die Zeitsynchronisation sorgt dafür, dass die Systemuhr korrekt läuft.

Status und Quelle der Zeitsynchronisation anzeigen:

```powershell
w32tm /query /status

w32tm /query /source
```

### 3.4 Remoteverwaltung einrichten

PowerShell-Remoting ermöglicht die Administration des Servers von einem anderen Computer aus. Dafür wird der Dienst **Windows Remote Management (WinRM)** verwendet.

#### PowerShell-Remoting aktivieren

```powershell
Enable-PSRemoting -Force
```

Der Befehl konfiguriert WinRM, die PowerShell-Sitzungsendpunkte und die erforderlichen Firewallregeln. Auf Windows Server ist Remoting häufig bereits aktiviert.

#### Funktion lokal prüfen

Status des Dienstes anzeigen:

```powershell
Get-Service WinRM
```

Der Dienst sollte den Status `Running` besitzen.

Lokalen WinRM-Endpunkt testen:

```powershell
Test-WSMan localhost
```

Bei erfolgreicher Einrichtung werden Informationen zum WinRM-Endpunkt angezeigt.

#### Später: Remoteverbindung innerhalb der Domäne aufbauen

Das folgende Beispiel setzt voraus, dass die Domäne `ad.example.test` mit dem Kurznamen `LAB` bereits eingerichtet ist und der Verwaltungscomputer ihr angehört.

Führe auf dem Verwaltungscomputer aus:

```powershell
Test-WSMan -ComputerName "SRV-DC01.ad.example.test"
```

Öffne anschließend eine Sitzung mit einem berechtigten Domänenkonto:

```powershell
Enter-PSSession `
    -ComputerName "SRV-DC01.ad.example.test" `
    -Credential (Get-Credential "LAB\Administrator")
```

Prüfe innerhalb der Sitzung, auf welchem Computer und unter welchem Konto du arbeitest:

```powershell
hostname

whoami
```

Remote-Sitzung beenden:

```powershell
Exit-PSSession
```

> **Lokales Konto:** Vor der Domäneneinrichtung ist für Remotezugriffe mit lokalen Konten eine zusätzliche Authentifizierungs- und Verbindungskonfiguration erforderlich, beispielsweise WinRM über HTTPS. Verwende für die beschriebenen ersten Einrichtungsschritte die lokale Server- oder VM-Konsole.

> **Abgrenzung:** PowerShell-Remoting stellt eine Befehlszeilensitzung bereit. Grafischer Remotedesktopzugriff über RDP muss separat eingerichtet werden.

## 4. Active Directory und DNS einrichten
- Serverrollen installieren
- Neue Domäne erstellen
- DNS-Auflösung überprüfen

## 5. Benutzer und Computer verwalten
- Organisationseinheiten (OUs) erstellen
- Benutzer und Gruppen anlegen
- Windows-Client in die Domäne aufnehmen

## 6. DHCP einrichten
- Adressbereich und Ausschlüsse definieren
- Gateway und DNS-Server verteilen
- Adressvergabe am Client testen

## 7. Dateifreigaben und Berechtigungen einrichten
- Ordner freigeben
- Freigabe- und NTFS-Berechtigungen vergeben
- Zugriff mit einem Testbenutzer prüfen

## 8. Gruppenrichtlinien einsetzen
- GPO erstellen und verknüpfen
- Beispiel: Netzlaufwerk verbinden
- Anwendung der Richtlinie überprüfen

## 9. Server warten und absichern
- Ereignisanzeige und Dienste prüfen
- Updates verwalten
- Firewall und administrative Zugriffe konfigurieren

## 10. Sicherung und Wiederherstellung
- Sicherung einrichten
- Wiederherstellung einer Datei testen
- Vorgehen bei Ausfällen dokumentieren








