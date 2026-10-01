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

Für Active Directory wird ein unterstütztes **Windows Server-Betriebssystem** benötigt. Der Server sollte über ausreichend Prozessorleistung, Arbeitsspeicher und Speicherplatz verfügen. Außerdem ist eine funktionierende Netzwerkverbindung erforderlich.

#### Serverinstallation

Zunächst wird **Windows Server** installiert und grundlegend eingerichtet. Dazu gehören beispielsweise die Vergabe eines eindeutigen Servernamens, die Installation von Updates und die Konfiguration der Netzwerkeinstellungen.

#### Statische IP-Konfiguration

Ein Domänencontroller sollte eine **statische IP-Adresse** besitzen. Dadurch ist sichergestellt, dass der Server für andere Geräte dauerhaft unter derselben Adresse erreichbar ist.

#### Installation und Konfiguration von DNS

**DNS** ist ein wichtiger Bestandteil von Active Directory. Der DNS-Server sorgt dafür, dass Domänencontroller und andere Geräte über ihre Namen gefunden werden können. Bei der Einrichtung einer neuen AD-Domäne kann die DNS-Serverrolle direkt mitinstalliert werden.

### 4.2 Aufbau der Domäne

#### Installation der AD-Domänendienste (AD DS)

Die Serverrolle **Active Directory Domain Services (AD DS)** wird über den Server-Manager oder mit PowerShell installiert. Sie stellt die grundlegenden Funktionen von Active Directory bereit.

#### Heraufstufen eines Servers zum Domänencontroller

Nach der Installation von AD DS wird der Server zum **Domänencontroller (DC)** heraufgestuft. Dabei werden die benötigten Active-Directory-Komponenten eingerichtet.

#### Erstellen einer neuen Domäne

Bei einer neuen Umgebung kann eine **neue Gesamtstruktur mit einer neuen Domäne** erstellt werden. Dabei werden unter anderem der Domänenname und wichtige Einstellungen für Active Directory festgelegt.

#### Hinzufügen weiterer Domänencontroller

Für eine höhere **Ausfallsicherheit und Verfügbarkeit** können weitere Domänencontroller zur bestehenden Domäne hinzugefügt werden. Die Active-Directory-Daten werden anschließend automatisch zwischen den Domänencontrollern repliziert.

## 5. Verwaltung von Active Directory

### 5.1 Benutzer- und Gruppenverwaltung

-   Benutzerkonten erstellen und verwalten
-   Gruppenarten:
    -   Sicherheitsgruppen
    -   Verteilergruppen
-   Gruppenrichtlinienbasierte Berechtigungen
-   Delegation von Verwaltungsaufgaben

### 5.2 Computerverwaltung

-   Computerobjekte verwalten
-   Domänenbeitritt
-   Richtlinien für Clients
-   Inventarisierung

## 6. Gruppenrichtlinien (GPO)

-   Grundlagen von Group Policy Objects
-   Aufbau und Vererbung
-   GPOs erstellen und verknüpfen
-   Sicherheitsrichtlinien
-   Softwareverteilung
-   Benutzer- und Computerkonfiguration
-   Troubleshooting von Gruppenrichtlinien

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
