# Forensik

## Datenerfassung und Beweissicherung

## Datenträgerforensik
- Datenträgeraufbau
    - Sektoren und Blöcke
    - physische und logische Blockadressierung
    - LBA
    - Advanced Format (4Kn, 512e)
- Partitionierungsschemata
    - MBR/DOS
        - Partitionstabelle
        - primäre und erweiterte Partitionen
        - Bootcode
    - GPT
        - Protective MBR
        - primärer und sekundärer GPT-Header
        - Partitionseinträge und Prüfsummen
        - Microsoft Reserved Partition (MSR)
        - EFI-Systempartition (ESP)
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

- FAT 

    > Typischer Einsatz: USB-Sticks, Speicherkarten und ältere Systeme
    
    - Grundbegriffe
        - FAT12, FAT16 und FAT32
        - Cluster und Clusterketten
    - Zentrale Strukturen:
        - Bootsektor
        - BIOS Parameter Block (BPB)
        - File Allocation Table
        - FAT-Kopien
        - Verzeichniseinträge
        - Long File Names (LFN)
        - FAT32: FSInfo-Sektor
    - Metadaten und Zeitstempel
        - Erstellungs-, Änderungs- und Zugriffszeit
        - begrenzte Zeitstempelauflösung
        - keine Benutzer- und Berechtigungsinformationen
    - Gelöschte Dateien
        - Kennzeichnung gelöschter Verzeichniseinträge
        - Verlust des ersten Zeichens im Dateinamen
        - Rekonstruktion von Clusterketten
    - Nicht zugewiesener Speicher und Slack Space
        - freie Cluster
        - File Slack
        - Dateifragmente
        - File Carving
    - Timeline-Erstellung
        - Auswertung der Verzeichniseinträge
        - eingeschränkte Aussagekraft der Zeitstempel
- NTFS 

    > Typisches Dateisystem von Windows
    
    -Grundlagen
        - Cluster und Logical Cluster Numbers
        - Dateien als Sammlung von Attributen
    - Zentrale Strukturen:
        - Master File Table ($MFT)
        - MFT Mirror ($MFTMirr)
        - Volume Bitmap ($Bitmap)
        - Bootsektor ($Boot)
        - Attribute wie $STANDARD_INFORMATION und $FILE_NAME
        - Indexstrukturen ($INDEX_ROOT, $INDEX_ALLOCATION)
    - Metadaten und Zeitstempel
        - MACB-Zeitstempel
        - unterschiedliche Zeitstempel in $STANDARD_INFORMATION und $FILE_NAME
        - Eigentümer und Zugriffsrechte
        - Alternate Data Streams (ADS)
    - Gelöschte Dateien
        - als frei markierte MFT-Einträge
        - Wiederverwendung von MFT-Einträgen
        - Wiederherstellung residenter und nicht-residenter Daten
    - Nicht zugewiesener Speicher und Slack Space
        - freie Cluster
        - File Slack
        - MFT Slack
        - File Carving
    - Journaling
        - $LogFile
        - $UsnJrnl
    - Timeline-Erstellung
        - Kombination von $MFT, $LogFile und $UsnJrnl
        - Erkennung von Erstellung, Änderung, Umbenennung und Löschung

- EXT4

    > Typisches Dateisystem von Linux-Systemen
    
    -Grundlagen
        - Blöcke und Blockgruppen
        - Inodes
    - Zentrale Strukturen:
        - Superblock
        - Group Descriptor Table
        - Inode-Tabellen
        - Block- und Inode-Bitmaps
        - Extents
        - Verzeichniseinträge
    - Metadaten und Zeitstempel
        - Inode-Metadaten
        - Berechtigungen, Eigentümer und Dateimodus
        - mtime, atime, ctime und crtime
        - Extended Attributes
    - Gelöschte Dateien
        - Freigabe von Inodes und Blöcken
        - entfernte Verzeichniseinträge
        - Rekonstruktion über Inodes und Journal
    - Nicht zugewiesener Speicher und Slack Space
        - freie Blöcke
        - Block Slack
        - File Carving
    - Journaling
        - JBD2-Journal
        - Journalmodi
        - Rekonstruktion früherer Metadaten
    - Timeline-Erstellung
        - Kombination aus Inodes, Verzeichniseinträgen und Journal



## Betriebssystemforensik

- [Windows](Windows-Forensik.md)

- [Linux](Linux-Forensik.md)

## Dateiforensik

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

### Windows

- Systemidentifikation
  - Windows-Version und Build
  - Kernel und Symbolinformationen
  - Systemzeit und Startzeitpunkt
- Prozesse und Threads
  - aktive und beendete Prozesse
  - Prozesshierarchie
  - Kommandozeilenargumente
  - versteckte Prozesse
  - verdächtige Threads
- Geladene Komponenten
  - DLLs
  - Kernelmodule und Treiber
  - nicht verknüpfte oder versteckte Module
- Speicherbereiche
  - Virtual Address Descriptors (`VAD`)
  - ausführbare Speicherbereiche
  - Speicherberechtigungen
  - Process Hollowing
  - DLL- und Code-Injection
- Handles und Objekte
  - offene Dateien
  - Registry-Schlüssel
  - Mutexe
  - Pipes
  - Prozesse und Tokens
- Netzwerkaktivitäten
  - aktive und geschlossene Verbindungen
  - offene Ports
  - Sockets
  - Zuordnung zu Prozessen
- Benutzer- und Anmeldedaten
  - angemeldete Benutzer
  - Sitzungen
  - Zugriffstokens
  - Anmeldeartefakte
  - gegebenenfalls im Speicher vorhandene Zugangsdaten
- Malware- und Rootkit-Erkennung
  - versteckte Prozesse
  - manipulierte Kernelstrukturen
  - injizierter Code
  - verdächtige Treiber
  - API-Hooking
- Weitere Artefakte
  - Konsolen- und Befehlsverläufe
  - Zwischenablage
  - Registry-Fragmente
  - im Speicher befindliche Dateien

### Linux

- Systemidentifikation
  - Distribution und Kernel-Version
  - Kernel-Symbole
  - Systemzeit und Startzeitpunkt
- Prozesse und Threads
  - aktive und beendete Prozesse
  - Prozesshierarchie
  - Kommandozeilenargumente
  - Umgebungsvariablen
  - versteckte Prozesse
- Geladene Komponenten
  - Shared Libraries
  - Kernelmodule
  - nicht verknüpfte oder versteckte Module
- Speicherbereiche
  - virtuelle Speicherbereiche (`VMAs`)
  - Heap und Stack
  - ausführbare Speicherbereiche
  - Speicherberechtigungen
  - Code-Injection
- Dateisystem- und Kernelobjekte
  - offene Dateien und Dateideskriptoren
  - gemountete Dateisysteme
  - Inode- und Dentry-Cache
  - im Speicher befindliche Dateiinhalte
- Netzwerkaktivitäten
  - aktive Verbindungen
  - offene Ports
  - Sockets
  - Zuordnung zu Prozessen
  - Netzwerk-Namensräume
- Benutzeraktivitäten
  - aktive Sitzungen
  - Shell-Prozesse
  - Befehlszeilen
  - Umgebungsvariablen
  - SSH-bezogene Artefakte
- Malware- und Rootkit-Erkennung
  - versteckte Prozesse
  - verdächtige Kernelmodule
  - System-Call-Hooking
  - manipulierte Kernelstrukturen
  - injizierter Code
- Container und Virtualisierung
  - Containerprozesse
  - Namespaces
  - Control Groups (`cgroups`)
  - Zuordnung von Prozessen zu Containern

### Gemeinsame Auswertung

- Rekonstruktion des Systemzustands
- Erstellung einer Prozess-Timeline
- Zuordnung von Netzwerkverbindungen zu Prozessen
- Extraktion verdächtiger Prozesse und Speicherbereiche
- Vergleich von Listen- und Scanverfahren
- Erkennung von Prozessmanipulation und Code-Injection
- Korrelation mit Datenträger- und Betriebssystemartefakten









Optional:

## Multimedia-Forensik

## IoT- und Embedded-Forensik

## Mobile Forensik

## Cloud-Forensik

## Malware-Forensik

## Log-, Ereignis- und Timeline-Analyse

## Netzwerkforensik

## Anwendungs- und Kommunikationsforensik