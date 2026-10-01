# Windows Active Directory -- Gliederung

## 1. Einführung in Active Directory (AD)

### Zweck und Einsatzbereiche

**Active Directory (AD)** ist ein Verzeichnisdienst von Microsoft zur zentralen Verwaltung von **Benutzern, Computern, Gruppen und Zugriffsrechten** in einem Netzwerk. Es wird hauptsächlich in Unternehmen eingesetzt, um IT-Systeme einfacher und sicherer zu verwalten.

### Abgrenzung zur lokalen Benutzerverwaltung

Bei einer **lokalen Benutzerverwaltung** werden Benutzerkonten auf jedem Computer einzeln angelegt und verwaltet. Mit Active Directory werden Benutzerkonten dagegen **zentral auf Servern verwaltet**. Dadurch können sich Benutzer beispielsweise mit demselben Konto an verschiedenen Computern einer Domäne anmelden.

### Grundbegriffe

- **Domäne:** Zentrale Verwaltungseinheit, in der Benutzer, Computer und andere Ressourcen zusammengefasst werden.
- **Gesamtstruktur (Forest):** Oberste Struktur eines Active Directory, die eine oder mehrere Domänen enthalten kann.
- **Domänenbaum:** Hierarchische Anordnung mehrerer zusammengehöriger Domänen.
- **Organisationseinheit (OU):** Dient zur übersichtlichen Strukturierung von Benutzern, Computern und Gruppen innerhalb einer Domäne.
- **Domänencontroller (DC):** Server, der Active Directory bereitstellt und unter anderem Benutzeranmeldungen überprüft.
- **Globaler Katalog:** Enthält Informationen über Objekte im gesamten Forest und ermöglicht eine schnelle Suche.
- **LDAP:** Protokoll zum Abfragen und Verwalten von Informationen in einem Verzeichnisdienst.
- **Kerberos:** Authentifizierungsprotokoll, das eine sichere Anmeldung und den Zugriff auf Netzwerkressourcen ermöglicht.

## 2. Grundlagen der Windows-Domäne

### 2.1 Architektur

#### Aufbau einer AD-Umgebung
Eine Active-Directory-Umgebung besteht aus einer oder mehreren **Domänen** mit Benutzern, Computern, Gruppen und Servern. Die zentrale Verwaltung erfolgt über **Domänencontroller (DCs)**.

#### Rollen und Funktionen von Domänencontrollern
Domänencontroller speichern die Active-Directory-Datenbank und übernehmen wichtige Aufgaben wie die **Anmeldung von Benutzern, Prüfung von Berechtigungen und Bereitstellung von Gruppenrichtlinien**.

#### Replikation zwischen Domänencontrollern
Gibt es mehrere Domänencontroller, werden Änderungen zwischen ihnen **automatisch repliziert**. Dadurch besitzen die DCs weitgehend denselben aktuellen Datenbestand und die Ausfallsicherheit wird erhöht.

#### FSMO-Rollen
Bestimmte Aufgaben werden durch fünf spezielle **FSMO-Rollen** übernommen:

- **Schema Master:** Verwaltet Änderungen am AD-Schema.
- **Domain Naming Master:** Verwaltet das Hinzufügen und Entfernen von Domänen im Forest.
- **RID Master:** Vergibt ID-Bereiche, die zur Erstellung eindeutiger Sicherheitskennungen benötigt werden.
- **PDC Emulator:** Übernimmt wichtige Aufgaben bei Kennwortänderungen, Zeitabgleich und Kompatibilität.
- **Infrastructure Master:** Verwaltet domänenübergreifende Objektverweise.

### 2.2 Netzwerkgrundlagen

#### DNS als Grundlage von Active Directory
**DNS (Domain Name System)** ist für Active Directory besonders wichtig. Es ermöglicht die Namensauflösung und hilft Clients dabei, Domänencontroller und andere AD-Dienste im Netzwerk zu finden.

#### DHCP-Integration
**DHCP** kann Clients automatisch mit Netzwerkeinstellungen wie **IP-Adresse, Standardgateway und DNS-Server** versorgen. Dadurch wird die Einbindung von Geräten in die Domäne erleichtert.

#### Zeitdienst und Kerberos
Für die **Kerberos-Authentifizierung** müssen die Systemzeiten der Geräte ausreichend synchron sein. Active Directory verwendet deshalb einen hierarchischen Windows-Zeitdienst, wobei der **PDC-Emulator der Forest-Root-Domäne** eine zentrale Rolle spielt.

#### Netzwerkports und Firewall-Anforderungen
Active Directory benötigt verschiedene Netzwerkports. Dazu gehören beispielsweise **DNS (53), Kerberos (88), LDAP (389), SMB (445) und LDAPS (636)**. Firewalls müssen die jeweils benötigten Verbindungen zwischen Clients, Servern und Domänencontrollern zulassen.

## 3. Planung einer Active-Directory-Umgebung

-   Anforderungsanalyse
-   Domänen- und Forest-Design
-   Namenskonventionen
-   OU-Struktur planen
-   Benutzer- und Gruppenmodell
-   Sicherheitsanforderungen
-   Backup- und Wiederherstellungsstrategie


## 4. Installation und Einrichtung

### 4.1 Vorbereitung

#### Windows-Server-Anforderungen

Für Active Directory wird ein unterstütztes **Windows Server-Betriebssystem** benötigt. Der Server sollte über ausreichend Prozessorleistung, Arbeitsspeicher und Speicherplatz verfügen und mit aktuellen Updates versorgt sein.

Systeminformationen können mit folgendem Befehl geprüft werden:

```powershell
Get-ComputerInfo
```

#### Serverinstallation

Nach der Installation von Windows Server sollte zunächst ein eindeutiger **Servername** vergeben werden.

Aktuellen Computernamen anzeigen:

```powershell
hostname
```

Server umbenennen:

```powershell
Rename-Computer -NewName "DC01" -Restart
```

Nach dem Neustart trägt der Server den Namen `DC01`.

#### Statische IP-Konfiguration

Ein Domänencontroller sollte eine **statische IP-Adresse** besitzen, damit er dauerhaft unter derselben Adresse erreichbar ist.

Netzwerkadapter anzeigen:

```powershell
Get-NetAdapter
```

Vorhandene IP-Konfiguration anzeigen:

```powershell
Get-NetIPConfiguration
```

Beispiel für die Vergabe einer statischen IP-Adresse:

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" `
-IPAddress 192.168.1.10 `
-PrefixLength 24 `
-DefaultGateway 192.168.1.1
```

DNS-Server festlegen:

```powershell
Set-DnsClientServerAddress `
-InterfaceAlias "Ethernet" `
-ServerAddresses 192.168.1.10
```

Die verwendeten IP-Adressen müssen an das eigene Netzwerk angepasst werden.

#### Installation und Konfiguration von DNS

**DNS** ist ein wichtiger Bestandteil von Active Directory. Es ermöglicht unter anderem das Auffinden von Domänencontrollern.

DNS-Serverrolle installieren:

```powershell
Install-WindowsFeature DNS -IncludeManagementTools
```

Installation überprüfen:

```powershell
Get-WindowsFeature DNS
```

DNS-Konfiguration testen:

```powershell
nslookup
```

### 4.2 Aufbau der Domäne

#### Installation der AD-Domänendienste (AD DS)

Die Rolle **Active Directory Domain Services (AD DS)** stellt die grundlegenden Funktionen von Active Directory bereit.

Installation über PowerShell:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

Installation überprüfen:

```powershell
Get-WindowsFeature AD-Domain-Services
```

#### Heraufstufen eines Servers zum Domänencontroller

Nach der Installation von AD DS muss der Server zum **Domänencontroller (DC)** heraufgestuft werden.

Für eine komplett neue Umgebung wird dafür eine neue Gesamtstruktur erstellt.

Beispiel:

```powershell
Install-ADDSForest `
-DomainName "firma.local" `
-DomainNetbiosName "FIRMA" `
-InstallDNS
```

Während der Einrichtung wird unter anderem nach dem Kennwort für den **Directory Services Restore Mode (DSRM)** gefragt.

Nach erfolgreicher Installation wird der Server normalerweise automatisch neu gestartet.

#### Erstellen einer neuen Domäne

Mit `Install-ADDSForest` wird gleichzeitig die erste Domäne einer neuen Gesamtstruktur erstellt.

Beispiel:

```powershell
Install-ADDSForest `
-DomainName "firma.local" `
-DomainNetbiosName "FIRMA" `
-InstallDNS
```

Nach dem Neustart kann die Domäne überprüft werden:

```powershell
Get-ADDomain
```

Informationen über die Gesamtstruktur anzeigen:

```powershell
Get-ADForest
```

Vorhandene Domänencontroller anzeigen:

```powershell
Get-ADDomainController -Filter *
```

#### Hinzufügen weiterer Domänencontroller

Für eine höhere **Ausfallsicherheit und Verfügbarkeit** sollten in produktiven Umgebungen mehrere Domänencontroller eingesetzt werden.

Auf dem zusätzlichen Server wird zunächst AD DS installiert:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

Anschließend wird der Server als zusätzlicher Domänencontroller zur vorhandenen Domäne hinzugefügt:

```powershell
Install-ADDSDomainController `
-DomainName "firma.local" `
-InstallDNS
```

Danach kann die Replikation zwischen den Domänencontrollern überprüft werden:

```powershell
repadmin /replsummary
```

Detaillierte Replikationsinformationen können mit folgendem Befehl angezeigt werden:

```powershell
repadmin /showrepl
```

Die Erreichbarkeit eines Domänencontrollers kann außerdem überprüft werden:

```powershell
nltest /dsgetdc:firma.local
```

#### Wichtige Befehle im Überblick

| Aufgabe | Befehl |
|---|---|
| Servername anzeigen | `hostname` |
| Server umbenennen | `Rename-Computer` |
| Netzwerkadapter anzeigen | `Get-NetAdapter` |
| IP-Konfiguration anzeigen | `Get-NetIPConfiguration` |
| Statische IP vergeben | `New-NetIPAddress` |
| DNS-Server festlegen | `Set-DnsClientServerAddress` |
| DNS installieren | `Install-WindowsFeature DNS` |
| AD DS installieren | `Install-WindowsFeature AD-Domain-Services` |
| Neue Gesamtstruktur erstellen | `Install-ADDSForest` |
| Domäne anzeigen | `Get-ADDomain` |
| Forest anzeigen | `Get-ADForest` |
| Domänencontroller anzeigen | `Get-ADDomainController` |
| Weiteren DC hinzufügen | `Install-ADDSDomainController` |
| Replikation prüfen | `repadmin /replsummary` |

## 5. Verwaltung von Active Directory

### 5.1 Benutzer- und Gruppenverwaltung

#### Benutzerkonten erstellen und verwalten

Benutzerkonten werden zentral im Active Directory gespeichert. Sie können über **Active Directory-Benutzer und -Computer (ADUC)** oder über PowerShell erstellt und verwaltet werden.

Neuen Benutzer erstellen:

```powershell
New-ADUser `
-Name "Max Mustermann" `
-GivenName "Max" `
-Surname "Mustermann" `
-SamAccountName "max.mustermann" `
-UserPrincipalName "max.mustermann@firma.local" `
-AccountPassword (Read-Host -AsSecureString "Passwort") `
-Enabled $true
```

Benutzer anzeigen:

```powershell
Get-ADUser -Filter *
```

Bestimmten Benutzer anzeigen:

```powershell
Get-ADUser -Identity "max.mustermann"
```

Benutzer deaktivieren:

```powershell
Disable-ADAccount -Identity "max.mustermann"
```

Benutzer wieder aktivieren:

```powershell
Enable-ADAccount -Identity "max.mustermann"
```

Benutzer löschen:

```powershell
Remove-ADUser -Identity "max.mustermann"
```

#### Gruppenarten

Gruppen erleichtern die Verwaltung von Benutzern und Berechtigungen. Statt Rechte für jeden Benutzer einzeln zu vergeben, können diese einer Gruppe zugewiesen werden.

##### Sicherheitsgruppen

**Sicherheitsgruppen** werden verwendet, um Berechtigungen auf Ressourcen wie Ordner, Dateien oder Drucker zu vergeben.

Sicherheitsgruppe erstellen:

```powershell
New-ADGroup `
-Name "IT-Mitarbeiter" `
-GroupScope Global `
-GroupCategory Security
```

Benutzer einer Gruppe hinzufügen:

```powershell
Add-ADGroupMember `
-Identity "IT-Mitarbeiter" `
-Members "max.mustermann"
```

Mitglieder einer Gruppe anzeigen:

```powershell
Get-ADGroupMember -Identity "IT-Mitarbeiter"
```

##### Verteilergruppen

**Verteilergruppen** werden hauptsächlich für E-Mail-Verteiler verwendet. Sie dienen nicht zur Vergabe von Zugriffsberechtigungen.

Verteilergruppe erstellen:

```powershell
New-ADGroup `
-Name "Marketing-Verteiler" `
-GroupScope Global `
-GroupCategory Distribution
```

#### Gruppenrichtlinienbasierte Berechtigungen

Mit **Gruppenrichtlinien (Group Policy Objects, GPOs)** können Einstellungen für Benutzer und Computer zentral vorgegeben werden.

Vorhandene Gruppenrichtlinien anzeigen:

```powershell
Get-GPO -All
```

Neue Gruppenrichtlinie erstellen:

```powershell
New-GPO -Name "Client-Sicherheitsrichtlinie"
```

Eine GPO kann anschließend beispielsweise mit einer Organisationseinheit verknüpft werden:

```powershell
New-GPLink `
-Name "Client-Sicherheitsrichtlinie" `
-Target "OU=Clients,DC=firma,DC=local"
```

Gruppenrichtlinien auf einem Client aktualisieren:

```cmd
gpupdate /force
```

Angewendete Richtlinien anzeigen:

```cmd
gpresult /r
```

#### Delegation von Verwaltungsaufgaben

Mit der **Delegation** können bestimmte Verwaltungsaufgaben an andere Benutzer übertragen werden, ohne ihnen vollständige Administratorrechte zu geben.

Beispielsweise kann einem Mitarbeiter erlaubt werden:

- Benutzerkonten zu erstellen
- Passwörter zurückzusetzen
- Gruppenmitgliedschaften zu verwalten
- Computerobjekte zu verwalten

Die Delegation wird üblicherweise über **Active Directory-Benutzer und -Computer** durchgeführt:

1. Rechtsklick auf die gewünschte OU
2. **Steuerung delegieren** auswählen
3. Benutzer oder Gruppe auswählen
4. Gewünschte Aufgaben und Berechtigungen festlegen

### 5.2 Computerverwaltung

#### Computerobjekte verwalten

Jeder Computer, der Mitglied einer Domäne ist, besitzt ein eigenes **Computerobjekt im Active Directory**.

Computerobjekte anzeigen:

```powershell
Get-ADComputer -Filter *
```

Bestimmten Computer suchen:

```powershell
Get-ADComputer -Identity "PC01"
```

Computerobjekt erstellen:

```powershell
New-ADComputer `
-Name "PC01" `
-SamAccountName "PC01"
```

Computerobjekt löschen:

```powershell
Remove-ADComputer -Identity "PC01"
```

Computer deaktivieren:

```powershell
Disable-ADAccount -Identity "PC01$"
```

#### Domänenbeitritt

Damit ein Windows-Computer zentral verwaltet werden kann, muss er der Active-Directory-Domäne beitreten.

Vor dem Domänenbeitritt muss der Client den **DNS-Server der AD-Umgebung** verwenden.

Domänenbeitritt über PowerShell:

```powershell
Add-Computer `
-DomainName "firma.local" `
-Credential "FIRMA\Administrator" `
-Restart
```

Nach dem Neustart ist der Computer Mitglied der Domäne.

Die aktuelle Domänenzugehörigkeit kann überprüft werden:

```powershell
Get-CimInstance Win32_ComputerSystem |
Select-Object Name, Domain
```

#### Richtlinien für Clients

Über Gruppenrichtlinien können Einstellungen zentral auf Client-Computer verteilt werden.

Beispiele sind:

- Passwort- und Sicherheitsrichtlinien
- Windows-Firewall-Einstellungen
- Netzlaufwerke
- Drucker
- Desktop-Einstellungen
- Windows-Update-Einstellungen
- Softwareverteilung

Gruppenrichtlinien aktualisieren:

```cmd
gpupdate /force
```

Angewendete Gruppenrichtlinien überprüfen:

```cmd
gpresult /r
```

Ausführlichen Bericht erstellen:

```cmd
gpresult /h C:\gp-report.html
```

#### Inventarisierung

Active Directory kann grundlegende Informationen über die vorhandenen Computerobjekte liefern.

Alle Computer anzeigen:

```powershell
Get-ADComputer -Filter *
```

Computer mit zusätzlichen Informationen anzeigen:

```powershell
Get-ADComputer -Filter * `
-Properties OperatingSystem, IPv4Address |
Select-Object Name, OperatingSystem, IPv4Address
```

Computer nach Betriebssystem filtern:

```powershell
Get-ADComputer `
-Filter 'OperatingSystem -like "*Windows*"' `
-Properties OperatingSystem |
Select-Object Name, OperatingSystem
```

Active Directory selbst ist jedoch **kein vollständiges Inventarisierungssystem**. Für detaillierte Informationen über installierte Software, Hardware oder Gerätezustände werden zusätzliche Verwaltungs- und Inventarisierungslösungen benötigt.

## 6. Gruppenrichtlinien (GPO)

### Grundlagen von Group Policy Objects

**Group Policy Objects (GPOs)** ermöglichen die zentrale Konfiguration von Benutzern und Computern innerhalb einer Active-Directory-Domäne.

Damit können beispielsweise folgende Einstellungen festgelegt werden:

- Passwort- und Sicherheitsrichtlinien
- Windows-Firewall
- Desktop-Einstellungen
- Netzlaufwerke und Drucker
- Softwareverteilung
- Windows-Einstellungen

Vorhandene GPOs anzeigen:

```powershell
Get-GPO -All
```

### Aufbau und Vererbung

GPOs können mit **Standorten, Domänen und Organisationseinheiten (OUs)** verknüpft werden. Einstellungen werden dabei grundsätzlich an untergeordnete Bereiche weitervererbt.

Die typische Reihenfolge der Verarbeitung lautet:

1. Lokale Richtlinie
2. Standort
3. Domäne
4. Organisationseinheit (OU)

Eine OU kann wiederum weitere untergeordnete OUs enthalten. Dadurch können Richtlinien gezielt auf bestimmte Benutzer oder Computer angewendet werden.

Die Vererbung einer OU anzeigen:

```powershell
Get-GPInheritance -Target "OU=Clients,DC=firma,DC=local"
```

### GPOs erstellen und verknüpfen

Eine neue Gruppenrichtlinie kann über die **Gruppenrichtlinienverwaltung (GPMC)** oder PowerShell erstellt werden.

Neue GPO erstellen:

```powershell
New-GPO -Name "Client-Sicherheitsrichtlinie"
```

GPO mit einer OU verknüpfen:

```powershell
New-GPLink `
-Name "Client-Sicherheitsrichtlinie" `
-Target "OU=Clients,DC=firma,DC=local"
```

Alle GPOs anzeigen:

```powershell
Get-GPO -All
```

Eine GPO löschen:

```powershell
Remove-GPO -Name "Client-Sicherheitsrichtlinie"
```

### Sicherheitsrichtlinien

Mit GPOs können zentrale **Sicherheitsrichtlinien** festgelegt werden.

Dazu gehören beispielsweise:

- Kennwortrichtlinien
- Kontosperrungsrichtlinien
- Firewall-Einstellungen
- Benutzerrechte
- Sicherheitseinstellungen
- Einschränkung bestimmter Windows-Funktionen

Die Standard-Kennwortrichtlinie einer Domäne kann beispielsweise angezeigt werden mit:

```powershell
Get-ADDefaultDomainPasswordPolicy
```

### Softwareverteilung

Über Gruppenrichtlinien kann Software zentral an Computer oder Benutzer verteilt werden. Klassisch werden dafür insbesondere **MSI-Pakete** verwendet.

Das Installationspaket sollte über einen Netzwerkpfad erreichbar sein, zum Beispiel:

```text
\\SERVER01\Software\Programm.msi
```

Die Softwareverteilung wird in der Gruppenrichtlinienverwaltung beispielsweise unter folgendem Bereich eingerichtet:

```text
Computerkonfiguration
└── Richtlinien
    └── Softwareeinstellungen
        └── Softwareinstallation
```

Damit kann benötigte Software automatisch auf den entsprechenden Computern installiert werden.

### Benutzer- und Computerkonfiguration

Eine GPO besitzt zwei wichtige Bereiche:

**Computerkonfiguration** enthält Einstellungen, die für Computer gelten. Dazu gehören beispielsweise Firewall-, Sicherheits- oder Systemeinstellungen.

**Benutzerkonfiguration** enthält Einstellungen für Benutzer. Dazu gehören beispielsweise Desktop-Einstellungen, Netzlaufwerke oder Einschränkungen der Benutzeroberfläche.

Gruppenrichtlinien auf einem Computer aktualisieren:

```cmd
gpupdate /force
```

Nur Computerrichtlinien aktualisieren:

```cmd
gpupdate /target:computer /force
```

Nur Benutzerrichtlinien aktualisieren:

```cmd
gpupdate /target:user /force
```

### Troubleshooting von Gruppenrichtlinien

Wenn eine Gruppenrichtlinie nicht angewendet wird, sollten zunächst die Netzwerkverbindung, DNS-Konfiguration, OU-Zuordnung und GPO-Verknüpfungen überprüft werden.

Angewendete Richtlinien anzeigen:

```cmd
gpresult /r
```

Ausführlichen HTML-Bericht erstellen:

```cmd
gpresult /h C:\gp-report.html
```

Resultierende Richtlinien mit PowerShell abrufen:

```powershell
Get-GPResultantSetOfPolicy `
-ReportType Html `
-Path "C:\gpo-report.html"
```

Erreichbarkeit der Domäne prüfen:

```cmd
nltest /dsgetdc:firma.local
```

DNS-Auflösung überprüfen:

```cmd
nslookup firma.local
```

Zusätzlich können Fehler in der **Ereignisanzeige** unter den GroupPolicy-Protokollen überprüft werden.

Typische Ursachen für GPO-Probleme sind:

- Fehlerhafte DNS-Konfiguration
- Client befindet sich in der falschen OU
- GPO wurde nicht korrekt verknüpft
- Sicherheitsfilter verhindern die Anwendung
- Andere GPOs überschreiben Einstellungen
- Keine Verbindung zum Domänencontroller

## 7. Sicherheit in Active Directory

-   Prinzip der geringsten Rechte
-   Passwort- und Kontorichtlinien
-   Administrative Konten absichern
-   Kerberos-Sicherheit
-   Auditing und Überwachung
-   Schutz vor Angriffen:
    -   Pass-the-Hash
    -   Kerberoasting
    -   Credential Theft

## 8. Erweiterte AD-Funktionen

-   Vertrauensstellungen
-   Mehrere Domänen verwalten
-   Read-Only Domain Controller (RODC)
-   Active Directory Federation Services (AD FS)
-   Active Directory Certificate Services (AD CS)

## 9. Administration und Wartung

-   Benutzer- und Gruppenpflege
-   Berechtigungsmanagement
-   Replikationsüberwachung
-   Ereignisprotokolle analysieren
-   Performance-Überwachung
-   Aufräumen alter Objekte

## 10. Backup und Wiederherstellung

-   Backup-Konzepte für Active Directory
-   System State Backup
-   Autoritative und nicht autoritative Wiederherstellung
-   Wiederherstellung eines Domänencontrollers

## 11. Fehlerbehebung (Troubleshooting)

-   DNS-Probleme analysieren
-   Replikationsfehler beheben
-   Anmeldeprobleme untersuchen
-   Gruppenrichtlinien analysieren
-   AD-Datenbankprobleme

## 12. Werkzeuge zur Administration

-   Active Directory-Benutzer und -Computer (ADUC)
-   Active Directory-Verwaltungscenter
-   Gruppenrichtlinienverwaltung (GPMC)
-   PowerShell für Active Directory
-   Windows Admin Center
-   Diagnosewerkzeuge:
    -   dcdiag
    -   repadmin
    -   nltest

## 13. Best Practices

-   Dokumentation der AD-Struktur
-   Regelmäßige Sicherheitsprüfungen
-   Patch-Management
-   Monitoring
-   Least-Privilege-Konzept
-   Notfallplanung

## 14. Praxisprojekte und Übungen

-   Aufbau einer Testdomäne
-   Einrichtung von Benutzern und Gruppen
-   Erstellung einer OU-Struktur
-   Implementierung von Gruppenrichtlinien
-   Backup und Restore testen
-   Fehlerfälle simulieren
