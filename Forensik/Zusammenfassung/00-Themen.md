# Forensik

## Datenerfassung und Beweissicherung

## [Datenträgerforensik](Datenträgerforensik.md)
- Datenträgeraufbau
    - Sektoren und Blöcke
    - physische und logische Blockadressierung
    - LBA
    - Advanced Format (4Kn, 512e)
- Partitionierungsschemata
    - MBR/DOS
    - GPT
- Verborgene beziehungsweise nicht zugewiesene Bereiche
    - HPA - Host Protected Area
    - DCO - Device Configuration Overlay
    - nicht zugewiesener Speicher
    - Partition Gaps
    - SSD Over-Provisioning
    - Over-Provisioning bei SSDs
- Verschlüsselung:
    - BitLocker
    - LUKS
    - FileVault
    - VeraCrypt
    - Self-Encrypting Drives



## Dateisystemforensik

- [FAT](FAT.md)

    > Typischer Einsatz: USB-Sticks, Speicherkarten und ältere Systeme
    
- [NTFS](NTFS.md)

    > Typisches Dateisystem von Windows
    
    
- [EXT4](EXT4.md)

    > Typisches Dateisystem von Linux-Systemen
    

## Betriebssystemforensik

- [Windows](Windows-Forensik.md)

- [Linux](Linux-Forensik.md)

## [Dateiforensik](Datei-Forensik.md)

- Dateityperkennung
  - Dateiendung
  - Dateisignatur beziehungsweise Magic Bytes
  - MIME-Type
  - Abweichung zwischen Endung und tatsächlichem Format
- Dokumentanalyse
  - PDF
  - Microsoft-Office-Dokumente
  - Archive
  - eingebettete Dateien und Makros
- Anwendungsmetadaten
  - Autor und Organisation
  - Erstellungsprogramm
  - Bearbeitungsdauer und Revisionsnummer
  - EXIF-Daten von Bildern
  - GPS- und Kamerainformationen
- Inhaltsanalyse
  - Zeichenketten
  - Skripte und Makros
  - URLs und E-Mail-Adressen
  - eingebettete beziehungsweise verschlüsselte Inhalte
- Integrität und Manipulation
  - Hashwerte
  - digitale Signaturen
  - beschädigte Dateistrukturen
  - manipulierte Metadaten
  - Steganografie
- Dateirekonstruktion
  - File Carving
  - fragmentierte Dateien
  - Reparatur beschädigter Dateien



## Hauptspeicherforensik

### Grundlagen und Datenerfassung

- Inhalte des Hauptspeichers
  - laufende Prozesse und Threads
  - geladene Bibliotheken und Kernelmodule
  - Netzwerkverbindungen
  - offene Dateien und Handles
  - Befehle und Benutzereingaben
  - temporär entschlüsselte Daten
- Speicherabbilder
  - vollständiger Memory Dump
  - Crash Dump
  - Hibernation-Datei
  - virtuelle Maschinensnapshots
- Forensische Sicherung
  - Live-Erfassung
  - Hashwerte und Integritätsprüfung
  - Dokumentation des Systemzustands
  - Veränderungen durch das Erfassungswerkzeug
- Analysewerkzeuge
  - Volatility
  - MemProcFS
  - betriebsspezifische Erfassungswerkzeuge

### [Windows](HS-Windows-Forensik.md)

### [Linux](HS-Linux-Forensik.md)

### Gemeinsame Auswertung

- Rekonstruktion des Systemzustands
- Erstellung einer Prozess-Timeline
- Zuordnung von Netzwerkverbindungen zu Prozessen
- Extraktion verdächtiger Prozesse und Speicherbereiche
- Vergleich von Listen- und Scanverfahren
- Erkennung von Prozessmanipulation und Code-Injection
- Korrelation mit Datenträger- und Betriebssystemartefakten


## [Tools](Tools.md)
- `dd`  
- `sha256sum`  
- `xxd`  
- `file`  
- `strings`  
- The Sleuth Kit  
- Autopsy  
- Volatility 3  
- ExifTool  
- `fdisk`  
- `parted`  





Optional:

## Multimedia-Forensik

## IoT- und Embedded-Forensik

## Mobile Forensik

## Cloud-Forensik

## Malware-Forensik

## Log-, Ereignis- und Timeline-Analyse

## Netzwerkforensik

## Anwendungs- und Kommunikationsforensik

# Optinonale Tools

- dc3dd und dcfldd
- Guymager
- PhotoRec
- Scalpel