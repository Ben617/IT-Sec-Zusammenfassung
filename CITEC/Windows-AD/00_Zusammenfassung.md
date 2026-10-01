# Windows Active Directory -- Gliederung

## 1. Einführung in Active Directory (AD)

-   Zweck und Einsatzbereiche von Active Directory
-   Historie und Entwicklung
-   Abgrenzung zu lokalen Benutzerverwaltungen
-   Grundbegriffe:
    -   Domäne
    -   Gesamtstruktur (Forest)
    -   Domänenbaum
    -   Organisationseinheit (OU)
    -   Domänencontroller (DC)
    -   Globaler Katalog
    -   LDAP und Kerberos

## 2. Grundlagen der Windows-Domäne

### 2.1 Architektur

-   Aufbau einer AD-Umgebung
-   Rollen und Funktionen von Domänencontrollern
-   Replikation zwischen Domänencontrollern
-   FSMO-Rollen:
    -   Schema Master
    -   Domain Naming Master
    -   RID Master
    -   PDC Emulator
    -   Infrastructure Master

### 2.2 Netzwerkgrundlagen

-   DNS als Grundlage von Active Directory
-   DHCP-Integration
-   Zeitdienst und Kerberos
-   Netzwerkports und Firewall-Anforderungen

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

-   Windows-Server-Anforderungen
-   Serverinstallation
-   Statische IP-Konfiguration
-   Installation und Konfiguration von DNS

### 4.2 Aufbau der Domäne

-   Installation der AD-Domänendienste (AD DS)
-   Heraufstufen eines Servers zum Domänencontroller
-   Erstellen einer neuen Domäne
-   Hinzufügen weiterer Domänencontroller

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
