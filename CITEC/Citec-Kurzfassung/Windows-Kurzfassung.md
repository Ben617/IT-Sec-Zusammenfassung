# Windows – Kurzzusammenfassung

## Grundlagen

* Windows ist ein proprietäres Betriebssystem von Microsoft.
* Wichtige Varianten sind Windows 10, Windows 11 und Windows Server.
* Die grafische Oberfläche wird durch Eingabeaufforderung, PowerShell und Windows Terminal ergänzt.
* Systemeinstellungen befinden sich in der Einstellungen-App und teilweise in der klassischen Systemsteuerung.

## Installation

* Installation über einen bootfähigen USB-Stick oder ein Installationsmedium.
* Auswahl von Sprache, Edition, Datenträger und Partition.
* Aktivierung erfolgt über einen Product Key oder eine digitale Lizenz.
* Nach der Installation müssen Updates und Gerätetreiber eingerichtet werden.

## Dateisystem

* Windows verwendet normalerweise das Dateisystem `NTFS`.

* Laufwerke werden mit Buchstaben wie `C:`, `D:` oder `E:` bezeichnet.

* Benutzerdateien liegen normalerweise unter:

  ```powershell
  C:\Users\BENUTZERNAME
  ```

* Wichtige Verzeichnisse:

  ```powershell
  C:\Windows
  C:\Program Files
  C:\Program Files (x86)
  C:\Users
  ```

## Benutzer

* Benutzer anzeigen:

  ```powershell
  Get-LocalUser
  ```

* Benutzer erstellen:

  ```powershell
  New-LocalUser -Name "BENUTZERNAME"
  ```

* Benutzer löschen:

  ```powershell
  Remove-LocalUser -Name "BENUTZERNAME"
  ```

* Benutzer aktivieren oder deaktivieren:

  ```powershell
  Enable-LocalUser -Name "BENUTZERNAME"
  Disable-LocalUser -Name "BENUTZERNAME"
  ```

## Gruppen

* Lokale Gruppen anzeigen:

  ```powershell
  Get-LocalGroup
  ```

* Gruppe erstellen:

  ```powershell
  New-LocalGroup -Name "GRUPPENNAME"
  ```

* Benutzer einer Gruppe hinzufügen:

  ```powershell
  Add-LocalGroupMember -Group "GRUPPENNAME" -Member "BENUTZERNAME"
  ```

* Benutzer aus einer Gruppe entfernen:

  ```powershell
  Remove-LocalGroupMember -Group "GRUPPENNAME" -Member "BENUTZERNAME"
  ```

## Rechte

* NTFS-Rechte regeln den Zugriff auf Dateien und Ordner.

* Wichtige Rechte sind Lesen, Schreiben, Ändern und Vollzugriff.

* Rechte anzeigen:

  ```powershell
  Get-Acl "C:\PFAD"
  ```

* Rechte können über **Eigenschaften → Sicherheit** bearbeitet werden.

* Administratorrechte werden durch die Benutzerkontensteuerung, kurz UAC, geschützt.

## Programme

* Programme können über den Microsoft Store, Installationsdateien oder den Paketmanager `winget` installiert werden.

* Programm suchen:

  ```powershell
  winget search PROGRAMMNAME
  ```

* Programm installieren:

  ```powershell
  winget install PROGRAMMNAME
  ```

* Programm aktualisieren:

  ```powershell
  winget upgrade PROGRAMMNAME
  ```

* Alle Programme aktualisieren:

  ```powershell
  winget upgrade --all
  ```

* Programm deinstallieren:

  ```powershell
  winget uninstall PROGRAMMNAME
  ```

## Prozesse

* Prozesse anzeigen:

  ```powershell
  Get-Process
  ```

* Prozess beenden:

  ```powershell
  Stop-Process -Name "PROZESSNAME"
  ```

* Prozesse können auch mit dem Task-Manager verwaltet werden:

  ```powershell
  taskmgr
  ```

## Dienste

* Dienste anzeigen:

  ```powershell
  Get-Service
  ```

* Dienst starten:

  ```powershell
  Start-Service -Name "DIENSTNAME"
  ```

* Dienst stoppen:

  ```powershell
  Stop-Service -Name "DIENSTNAME"
  ```

* Dienste können außerdem über `services.msc` verwaltet werden.

## Netzwerk

* Netzwerkkonfiguration anzeigen:

  ```powershell
  ipconfig /all
  ```

* Verbindung testen:

  ```powershell
  ping HOSTNAME
  ```

* Route zu einem Ziel anzeigen:

  ```powershell
  tracert HOSTNAME
  ```

* DNS-Auflösung testen:

  ```powershell
  nslookup HOSTNAME
  ```

* Aktive Netzwerkverbindungen anzeigen:

  ```powershell
  netstat -ano
  ```

* DNS-Zwischenspeicher leeren:

  ```powershell
  ipconfig /flushdns
  ```

## Sicherheit

* Microsoft Defender schützt vor Schadsoftware.
* Die Windows-Firewall kontrolliert ein- und ausgehende Verbindungen.
* BitLocker verschlüsselt Laufwerke.
* UAC verhindert unbemerkte Änderungen mit Administratorrechten.
* Updates sollten regelmäßig installiert werden.
* Für Benutzer sollten möglichst keine dauerhaften Administratorrechte verwendet werden.

## Updates

* Windows Update verteilt Sicherheits-, Funktions- und Treiberupdates.

* Einstellungen öffnen:

  ```powershell
  start ms-settings:windowsupdate
  ```

* Nach größeren Updates können Neustarts erforderlich sein.

* Vor Funktionsupdates sollte eine Datensicherung erstellt werden.

## Ereignisanzeige und Protokolle

* Die Ereignisanzeige protokolliert System-, Sicherheits- und Programmereignisse.

* Ereignisanzeige öffnen:

  ```powershell
  eventvwr.msc
  ```

* Ereignisse mit PowerShell anzeigen:

  ```powershell
  Get-WinEvent -LogName System
  ```

## Datensicherung

* Wichtige Dateien sollten auf einem externen Datenträger oder in einem sicheren Cloudspeicher gespeichert werden.
* Windows bietet Dateiversionsverlauf, Wiederherstellungspunkte und Systemabbilder.
* Wiederherstellungsoptionen befinden sich in der Einstellungen-App.

## Domäne und Active Directory

* Eine Arbeitsgruppe verbindet Rechner ohne zentrale Verwaltung.
* Eine Domäne ermöglicht die zentrale Verwaltung von Benutzern, Computern und Richtlinien.
* Active Directory speichert Benutzer, Gruppen und Computer.
* Gruppenrichtlinien steuern Sicherheitseinstellungen und Systemkonfigurationen.

## Häufige Probleme

* Fehlende oder fehlerhafte Gerätetreiber

* Fehlgeschlagene Windows-Updates

* Netzwerk- und DNS-Probleme

* Beschädigte Systemdateien

* Zu wenig Speicherplatz

* Rechte- und Anmeldeprobleme

* Systemdateien überprüfen:

  ```powershell
  sfc /scannow
  ```

* Windows-Abbild reparieren:

  ```powershell
  DISM /Online /Cleanup-Image /RestoreHealth
  ```

* Datenträger überprüfen:

  ```powershell
  chkdsk C: /scan
  ```
