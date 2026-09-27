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

- [FAT](FAT.md)

    > Typischer Einsatz: USB-Sticks, Speicherkarten und ältere Systeme
    
- [NTFS](NTFS.md)

    > Typisches Dateisystem von Windows
    
    
- [EXT4](EXT4.md)

    > Typisches Dateisystem von Linux-Systemen
    

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


## Tools

- `dd`  
  > Erstellt bitweise Kopien von Datenträgern, Partitionen oder Dateien. Es kann zur Erzeugung eines RAW-Abbilds verwendet werden.

  **Befehlsaufbau:**

  ```bash
  dd if=<Eingabe> of=<Ausgabe> [Optionen]
  ```

  - `if`: Input File – Quelldatenträger oder Quelldatei
  - `of`: Output File – Zieldatei oder Zieldatenträger
  - `bs`: Größe der zu kopierenden Blöcke
  - `conv=noerror`: Kopiervorgang bei Lesefehlern fortsetzen
  - `conv=sync`: fehlerhafte Blöcke mit Nullen auffüllen
  - `status=progress`: Fortschritt anzeigen

  **Datenträgerabbild erstellen:**

  ```bash
  sudo dd if=/dev/sdX of=beweismittel.dd bs=4M conv=noerror,sync status=progress
  ```

  **Abbild zurück auf einen Datenträger schreiben:**

  ```bash
  sudo dd if=beweismittel.dd of=/dev/sdX bs=4M status=progress
  ```

  > **Achtung:** Eine Verwechslung von `if` und `of` kann Daten überschreiben. Der Quelldatenträger sollte über einen Write-Blocker schreibgeschützt sein.

- `sha256sum`  
  > Berechnet SHA-256-Hashwerte zur Überprüfung der Integrität eines Beweismittels.

  **Hashwert berechnen:**

  ```bash
  sha256sum beweismittel.dd
  ```

  **Hashwert speichern:**

  ```bash
  sha256sum beweismittel.dd > beweismittel.dd.sha256
  ```

  **Gespeicherten Hashwert prüfen:**

  ```bash
  sha256sum --check beweismittel.dd.sha256
  ```

- `xxd`  
  > Stellt binäre Daten hexadezimal dar. Dadurch können Sektoren, Dateiköpfe und Signaturen untersucht werden.

  **Erste 512 Bytes anzeigen:**

  ```bash
  xxd -l 512 beweismittel.dd
  ```

  **Daten ab einem bestimmten Offset anzeigen:**

  ```bash
  xxd -s 1024 -l 256 beweismittel.dd
  ```

  **Hexdump einer Datei erstellen:**

  ```bash
  xxd datei.bin > datei.hex
  ```

- `file`  
  > Bestimmt den tatsächlichen Dateityp anhand des Inhalts und charakteristischer Dateisignaturen.

  **Dateityp bestimmen:**

  ```bash
  file unbekannte_datei
  ```

  **Mehrere Dateien untersuchen:**

  ```bash
  file *
  ```

  **MIME-Type anzeigen:**

  ```bash
  file --mime-type unbekannte_datei
  ```

- `strings`  
  > Extrahiert lesbare Zeichenketten aus Dateien oder Datenträgerabbildern.

  **Zeichenketten anzeigen:**

  ```bash
  strings beweismittel.dd
  ```

  **Mindestlänge festlegen:**

  ```bash
  strings -n 8 beweismittel.dd
  ```

  **Offsets der Fundstellen anzeigen:**

  ```bash
  strings -t d beweismittel.dd
  ```

  **Nach einem Begriff suchen:**

  ```bash
  strings beweismittel.dd | grep -i "password"
  ```

- The Sleuth Kit  
  > Sammlung von Kommandozeilenwerkzeugen zur Analyse von Partitionen, Dateisystemen, Verzeichnissen und gelöschten Dateien.

  **Partitionstabelle anzeigen:**

  ```bash
  mmls beweismittel.dd
  ```

  **Informationen zum Dateisystem anzeigen:**

  ```bash
  fsstat -o 2048 beweismittel.dd
  ```

  **Dateien und Verzeichnisse auflisten:**

  ```bash
  fls -r -o 2048 beweismittel.dd
  ```

  **Auch gelöschte Einträge anzeigen:**

  ```bash
  fls -r -d -o 2048 beweismittel.dd
  ```

  **Datei anhand ihrer Metadatenadresse extrahieren:**

  ```bash
  icat -o 2048 beweismittel.dd 128 > extrahierte_datei
  ```

  **Hinweis:** Der Wert `2048` ist ein Beispiel für den Startsektor der Partition. Der tatsächliche Wert kann mit `mmls` ermittelt werden.

- Autopsy  
  > Grafische Plattform zur Untersuchung von Datenträgerabbildern und Dateisystemen. Autopsy verwendet Funktionen des Sleuth Kit.

  **Autopsy unter Linux starten:**

  ```bash
  autopsy
  ```

  Anschließend wird je nach Installation eine lokale Weboberfläche geöffnet. In der Desktop-Version erfolgt die Analyse über die grafische Benutzeroberfläche:

  1. neuen Fall anlegen,
  2. Datenquelle hinzufügen,
  3. Datenträgerabbild auswählen,
  4. Analysemodule festlegen,
  5. Ergebnisse auswerten.

- Volatility 3  
  > Framework zur forensischen Analyse von Hauptspeicherabbildern.

  **Allgemeiner Befehlsaufbau:**

  ```bash
  python3 vol.py -f <Speicherabbild> <Plugin>
  ```

  **Windows-Systeminformationen:**

  ```bash
  python3 vol.py -f memory.raw windows.info
  ```

  **Windows-Prozesse anzeigen:**

  ```bash
  python3 vol.py -f memory.raw windows.pslist
  ```

  **Nach versteckten Prozessen suchen:**

  ```bash
  python3 vol.py -f memory.raw windows.psscan
  ```

  **Netzwerkverbindungen untersuchen:**

  ```bash
  python3 vol.py -f memory.raw windows.netscan
  ```

  **Nach verdächtigen Speicherbereichen suchen:**

  ```bash
  python3 vol.py -f memory.raw windows.malfind
  ```

  Abhängig von der Installation kann der Aufruf auch so lauten:

  ```bash
  vol -f memory.raw windows.pslist
  ```

- ExifTool  
  > Liest Metadaten aus Bildern, Dokumenten und zahlreichen weiteren Dateiformaten aus.

  **Alle erkannten Metadaten anzeigen:**

  ```bash
  exiftool bild.jpg
  ```

  **Bestimmte Metadaten anzeigen:**

  ```bash
  exiftool -FileName -FileType -CreateDate -ModifyDate bild.jpg
  ```

  **GPS-Informationen anzeigen:**

  ```bash
  exiftool -GPSLatitude -GPSLongitude bild.jpg
  ```

  **Metadaten mehrerer Dateien untersuchen:**

  ```bash
  exiftool untersuchungsordner/
  ```

- `fdisk`  
  > Zeigt Partitionstabellen, Startsektoren, Partitionsgrößen und Partitionstypen an.

  **Datenträger auflisten:**

  ```bash
  sudo fdisk -l
  ```

  **Partitionstabelle eines Abbilds anzeigen:**

  ```bash
  fdisk -l beweismittel.dd
  ```

  **Sektoren als Einheit verwenden:**

  ```bash
  fdisk -l -u sectors beweismittel.dd
  ```

- `parted`  
  > Untersucht MBR- und GPT-Partitionstabellen sowie nicht zugewiesene Bereiche.

  **Partitionstabelle anzeigen:**

  ```bash
  parted beweismittel.dd unit s print
  ```

  **Freie Bereiche anzeigen:**

  ```bash
  parted beweismittel.dd unit s print free
  ```

  **Maschinenlesbare Ausgabe erzeugen:**

  ```bash
  parted -s -m beweismittel.dd unit s print
  ```

> Bei forensischen Untersuchungen sollten die Originaldatenträger schreibgeschützt und möglichst nur deren Abbilder analysiert werden. Insbesondere `dd`, `fdisk` und `parted` können bei einem falschen Aufruf Daten verändern oder überschreiben.






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