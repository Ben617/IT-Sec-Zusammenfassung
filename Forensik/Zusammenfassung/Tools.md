# Tools

- `dd`  
  > Erstellt bitweise Kopien von Datenträgern, Partitionen oder Dateien. Es kann zur Erzeugung eines RAW-Abbilds verwendet werden.

  **Befehlsaufbau:**

  ```bash
  dd if=<Eingabe> of=<Ausgabe> [Optionen]
  ```

  - `if`: Input File – Quelldatenträger oder Quelldatei
  - `of`: Output File – Zieldatei oder Zieldatenträger
  - `bs`: Größe der zu kopierenden Blöcke
  - `skip`: Anzahl der zu übersprigenden Blöcke
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
