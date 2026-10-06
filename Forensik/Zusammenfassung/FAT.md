# FAT

    > Typischer Einsatz: USB-Sticks, Speicherkarten und ältere Systeme
    
## Grundbegriffe
    - FAT12, FAT16 und FAT32
    - Cluster und Clusterketten
## Zentrale Strukturen:
Zunächst wird die Partitionstabelle des Datenträgerabbilds untersucht. Dadurch lässt sich feststellen, unter welcher Partitionsnummer die FAT-Partition geführt wird und bei welchem Sektor sie beginnt.

```bash
mmls EDF.dd
```
![]()

Die gewünschte Partitionsnummer und ihr Startsektor werden anschließend in Variablen eingetragen:

![]()

```bash
PARTITION=2
STARTSEKTOR=2048
```
Die ausgewählte Partition kann mit `mmcat` aus dem Datenträgerabbild extrahiert werden:

```bash
mmcat EDF.dd "$PARTITION" > partition.dd
```

Anschließend zeigt `fsstat` die zentralen Strukturen des enthaltenen Dateisystems an. Dazu gehören unter anderem die Position des Bootsektors, die FAT-Bereiche, die Anzahl der FAT-Kopien, das Root-Verzeichnis und der Datenbereich.

```bash
fsstat partition.dd
```

![]()





- Bootsektor
Der Bootsektor befindet sich am Anfang der FAT-Partition. Er enthält neben Startcode und Dateisysteminformationen auch den BIOS Parameter Block. Um den Bootsektor aus der bereits extrahierten Partition zu kopieren, werden die ersten 512 Byte ausgelesen:

```bash
dd if=partition.dd of=bootsektor.bin bs=512 count=1 status=none
```

todo

- BIOS Parameter Block (BPB)
Der BIOS Parameter Block beginnt bei Byte 11 beziehungsweise am Offset `0x0B` des Bootsektors. Er enthält die Parameter, aus denen die Lage und Größe der übrigen Dateisystembereiche berechnet werden. Dazu gehören unter anderem die Sektorgröße, die Anzahl reservierter Sektoren, die Anzahl der FAT-Kopien und die Größe einer FAT.

Der relevante Bereich kann mit `xxd` hervorgehoben ausgegeben werden:
```bash
mmcat EDF.dd partition
```
![]()


```bash
xxd -g 1 -s 11 -l 79 bootsektor.bin
```
![]()


Eine leichter lesbare Interpretation der im BPB gespeicherten Werte liefert:

```bash
fsstat partition.dd
```
![]()

todo auswertung




        
- File Allocation Table
        
Die File Allocation Table verwaltet die Belegung und Verkettung der Cluster. Ihre Position und Größe lassen sich der Ausgabe von `fsstat` entnehmen:

```bash
fsstat partition.dd
```

![]()


In der Ausgabe werden beispielsweise die Sektorbereiche der ersten und zweiten FAT angezeigt. Der Anfangssektor und die Größe der ersten FAT werden anschließend übernommen:

```bash
FAT1_START=32
FAT_GROESSE=123
```

> Die Zahlen sind Beispiele und müssen durch die Werte aus der eigenen `fsstat`-Ausgabe ersetzt werden.

Die erste FAT kann dann aus der Partition kopiert werden:

```bash
dd if=partition.dd of=fat1.bin bs=512 skip="$FAT1_START" count="$FAT_GROESSE" status=none
```

Zur ersten Kontrolle wird der Anfang der FAT hexadezimal ausgegeben:

```bash
xxd -g 1 -l 512 fat1.bin
```

![]()


Todo Auswertung



- FAT-Kopien
        
FAT-Dateisysteme besitzen üblicherweise zwei Kopien der File Allocation Table. Die zweite FAT liegt normalerweise unmittelbar hinter der ersten. Ihr Startsektor ergibt sich daher aus:

```text
Start FAT 2 = Start FAT 1 + Größe einer FAT
```

Der Startsektor der zweiten FAT kann in der Shell berechnet werden:

```bash
FAT2_START=$((FAT1_START + FAT_GROESSE))
```

Anschließend wird auch die zweite FAT extrahiert:

```bash
dd if=partition.dd of=fat2.bin bs=512 skip="$FAT2_START" count="$FAT_GROESSE" status=none
```

Mit `cmp` lässt sich überprüfen, ob beide FAT-Kopien identisch sind:
todo sollte er gelcih sein?
```bash
cmp fat1.bin fat2.bin
```

Sind die Dateien identisch, erzeugt `cmp` keine Ausgabe. Abweichende Bytes können mit folgendem Befehl angezeigt werden:

```bash
cmp -l fat1.bin fat2.bin | head
```

Alternativ können Prüfsummen gebildet werden:

```bash
sha256sum fat1.bin fat2.bin
```



- Verzeichniseinträge

Ein FAT-Verzeichnis besteht aus einer Folge von jeweils **32 Byte großen Verzeichniseinträgen**. Ein Eintrag enthält unter anderem den Dateinamen, Dateiattribute, Zeitstempel, Startcluster und die Dateigröße.

Zunächst können die Verzeichniseinträge mit `fls` aufgelistet werden:

```bash
fls partition.dd
```
![]()


Eine rekursive Ausgabe aller Verzeichnisse und Dateien erfolgt mit:

```bash
fls -r -p partition.dd
```
![]()

Die ausführliche Ausgabe mit Zeitstempeln und Metadatenadressen erhält man mit:

```bash
fls -l -r -p partition.dd
```

![]()

Vor jedem Eintrag zeigt `fls` eine Metadatenadresse an. Diese Adresse kann anschließend an `istat` übergeben werden:

```bash
istat partition.dd METADATENADRESSE
```

![]()

todo Auswertung

`istat` zeigt unter anderem:

- den Dateityp,
- die Dateigröße,
- den Startcluster,
- die belegten Cluster beziehungsweise Sektoren,
- den Erstellungszeitpunkt,
- den letzten Schreibzugriff,
- den letzten Lesezugriff,
- den Zustand des Eintrags.
        
- Verzeichniseinträge im Hexdump

Um die ursprünglichen 32-Byte-Einträge zu untersuchen, muss der Datenbereich des jeweiligen Verzeichnisses ausgelesen werden. Zunächst wird die Metadatenadresse des Verzeichnisses mit `fls` bestimmt. Danach zeigt `istat`, in welchen Sektoren beziehungsweise Clustern das Verzeichnis gespeichert ist:

```bash
istat partition.dd VERZEICHNISADRESSE
```

Für das Root-Verzeichnis wird häufig die Metadatenadresse `2` verwendet:

```bash
istat partition.dd 2
```

Die genaue Adresse sollte jedoch immer anhand der eigenen `fls`-Ausgabe überprüft werden.

Ein durch `istat` angezeigter Sektor kann mit `blkcat` ausgelesen werden:

```bash
blkcat partition.dd SEKTORNUMMER > verzeichnis.bin
```

Falls das Verzeichnis mehrere zusammenhängende Sektoren belegt, kann deren Anzahl zusätzlich angegeben werden:

```bash
blkcat partition.dd STARTSEKTOR ANZAHL > verzeichnis.bin
```

Danach wird der Verzeichnisbereich als Hexdump dargestellt:

```bash
xxd -g 1 -c 32 verzeichnis.bin
```

Die Option `-c 32` stellt 32 Byte pro Zeile dar. Dadurch entspricht eine Zeile genau der Größe eines FAT-Verzeichniseintrags, sofern der Eintrag am Zeilenanfang beginnt.

Ein einzelner Eintrag kann beispielsweise so extrahiert werden:

```bash
dd if=verzeichnis.bin of=eintrag.bin \
   bs=1 skip=$((NUMMER * 32)) count=32 status=none
```

Dabei beginnt die Nummerierung mit `NUMMER=0`. Der extrahierte Eintrag kann anschließend untersucht werden:

```bash
xxd -g 1 -c 32 eintrag.bin
```

Die wichtigsten Felder eines normalen FAT-Verzeichniseintrags sind:

| Offset | Größe | Inhalt |
|---:|---:|---|
| `0x00` | 8 Byte | kurzer Dateiname |
| `0x08` | 3 Byte | Dateiendung |
| `0x0B` | 1 Byte | Dateiattribute |
| `0x0E` | 2 Byte | Erstellungszeit |
| `0x10` | 2 Byte | Erstellungsdatum |
| `0x16` | 2 Byte | letzte Schreibzeit |
| `0x18` | 2 Byte | letztes Schreibdatum |
| `0x14` | 2 Byte | obere 16 Bit des Startclusters, nur FAT32 |
| `0x1A` | 2 Byte | untere 16 Bit des Startclusters |
| `0x1C` | 4 Byte | Dateigröße |

Der Startcluster einer FAT32-Datei wird aus zwei Feldern zusammengesetzt:

```text
Startcluster = (oberer Teil << 16) | unterer Teil
```

Besondere Werte im ersten Byte eines Verzeichniseintrags sind:

- `0x00`: Dieser und alle folgenden Einträge sind unbenutzt.
- `0xE5`: Der Eintrag wurde gelöscht.
- `0x2E`: Der Eintrag bezeichnet `.` oder `..`.
- Attribut `0x0F`: Der Eintrag gehört zu einem langen Dateinamen.
        
        
- Long File Names (LFN)

Das ursprüngliche FAT-Format unterstützt im normalen Verzeichniseintrag lediglich Dateinamen im **8.3-Format**. Ein Dateiname besteht dort aus höchstens acht Zeichen für den Namen und drei Zeichen für die Dateiendung.

Längere Dateinamen werden durch zusätzliche **Long-File-Name-Einträge** gespeichert. Auch ein LFN-Eintrag ist 32 Byte groß. Er steht unmittelbar vor dem zugehörigen normalen 8.3-Verzeichniseintrag.

Die Anzeige mit `fls` rekonstruiert in der Regel bereits den vollständigen langen Dateinamen:

```bash
fls -l -r -p partition.dd
```

Anschließend kann der betreffende Eintrag mit `istat` untersucht werden:

```bash
istat partition.dd METADATENADRESSE
```

```bash
istat partition.dd 42
```

`istat` zeigt hauptsächlich die zusammengefassten Metadaten der Datei. Die einzelnen LFN-Einträge werden am deutlichsten im Hexdump des übergeordneten Verzeichnisses sichtbar:

```bash
istat partition.dd VERZEICHNISADRESSE
```

Der dort genannte Sektor wird ausgelesen:

```bash
blkcat partition.dd SEKTORNUMMER > verzeichnis.bin
```

Danach wird der Inhalt mit 32 Byte pro Zeile dargestellt:

```bash
xxd -g 1 -c 32 verzeichnis.bin
```

LFN-Einträge lassen sich am Attributwert `0x0F` erkennen. Das Attribut steht bei Offset `0x0B` innerhalb des Eintrags.

Die Struktur eines LFN-Eintrags lautet:

| Offset | Größe | Inhalt |
|---:|---:|---|
| `0x00` | 1 Byte | Sequenznummer |
| `0x01` | 10 Byte | erste fünf UTF-16-Zeichen |
| `0x0B` | 1 Byte | Attribut `0x0F` |
| `0x0C` | 1 Byte | Typ, normalerweise `0x00` |
| `0x0D` | 1 Byte | Prüfsumme des 8.3-Namens |
| `0x0E` | 12 Byte | weitere sechs UTF-16-Zeichen |
| `0x1A` | 2 Byte | immer `0x0000` |
| `0x1C` | 4 Byte | letzte zwei UTF-16-Zeichen |

Ein LFN-Eintrag speichert somit höchstens **13 UTF-16-Zeichen**. Für längere Namen werden mehrere LFN-Einträge verwendet.

Im Hexdump erscheinen einfache lateinische Zeichen beispielsweise folgendermaßen:

```text
44 00 6F 00 6B 00 75 00 6D 00
```

Diese Bytefolge entspricht in UTF-16LE dem Text:

```text
Dokum
```

Die Reihenfolge der LFN-Einträge ist rückwärts: Der Eintrag mit dem letzten Teil des Namens steht zuerst. Das Bit `0x40` in der Sequenznummer kennzeichnet den ersten physischen Eintrag beziehungsweise den letzten Teil des vollständigen Namens.

Die zugehörigen LFN-Einträge können im Hexdump gesucht werden:

```bash
xxd -g 1 -c 32 verzeichnis.bin | less
```

Alternativ kann nach dem Attributbyte `0x0F` gesucht werden:

```bash
xxd -g 1 -c 32 verzeichnis.bin | grep -i "0f"
```

Diese Suche dient nur als erste Orientierung, da der Wert `0x0F` auch an anderen Positionen vorkommen kann. Entscheidend ist, dass er sich am Offset `0x0B` eines 32-Byte-Eintrags befindet.


- FAT32: FSInfo-Sektor


Bei FAT32 existiert zusätzlich ein sogenannter **FSInfo-Sektor**. Er enthält Verwaltungsinformationen über den Datenbereich. Dazu gehören insbesondere:

- die zuletzt bekannte Anzahl freier Cluster,
- ein Hinweis auf den nächsten möglicherweise freien Cluster.

Die Nummer des FSInfo-Sektors steht im BIOS Parameter Block bei Offset `0x30`. Das Feld ist zwei Byte groß und wird im Little-Endian-Format gespeichert.

Der betreffende BPB-Wert kann aus dem Bootsektor ausgelesen werden:

```bash
xxd -g 1 -s 48 -l 2 bootsektor.bin
```

Typischerweise enthält das Feld den Wert:

```text
01 00
```

Wegen der Little-Endian-Reihenfolge entspricht dies dem Sektor `1`. Der FSInfo-Sektor befindet sich dann unmittelbar hinter dem Bootsektor.

Der Wert kann auch dezimal ausgegeben werden:

```bash
od -An -tu2 -j 48 -N 2 bootsektor.bin
```

Die Position des FSInfo-Sektors wird außerdem von `fsstat` interpretiert angezeigt:

```bash
fsstat partition.dd
```

Der FSInfo-Sektor kann anhand des ermittelten Wertes extrahiert werden:

```bash
FSINFO_SEKTOR=1
```

```bash
dd if=partition.dd of=fsinfo.bin \
   bs=512 skip="$FSINFO_SEKTOR" count=1 status=none
```

Alternativ kann `blkcat` verwendet werden:

```bash
blkcat partition.dd "$FSINFO_SEKTOR" > fsinfo.bin
```

Anschließend wird der vollständige Sektor als Hexdump dargestellt:

```bash
xxd -g 1 fsinfo.bin
```

Die wichtigsten Felder des FSInfo-Sektors sind:

| Offset | Größe | Bedeutung |
|---:|---:|---|
| `0x000` | 4 Byte | erste Signatur |
| `0x004` | 480 Byte | reservierter Bereich |
| `0x1E4` | 4 Byte | zweite Signatur |
| `0x1E8` | 4 Byte | zuletzt bekannte Anzahl freier Cluster |
| `0x1EC` | 4 Byte | Hinweis auf den nächsten freien Cluster |
| `0x1F0` | 12 Byte | reservierter Bereich |
| `0x1FC` | 4 Byte | Abschlusssignatur |

Die charakteristischen Signaturen erscheinen im Hexdump in Little-Endian-Darstellung:

```text
Offset 0x000: 52 52 61 41
Offset 0x1E4: 72 72 41 61
Offset 0x1FC: 00 00 55 AA
```

Die erste Signatur entspricht dem Wert `0x41615252`, die zweite dem Wert `0x61417272`.

Gezielt können die letzten relevanten 32 Byte angezeigt werden:

```bash
xxd -g 1 -s 484 -l 28 fsinfo.bin
```

Die Anzahl der freien Cluster steht bei Offset 488 beziehungsweise `0x1E8`:

```bash
od -An -tu4 -j 488 -N 4 fsinfo.bin
```

Der Hinweis auf den nächsten freien Cluster steht bei Offset 492 beziehungsweise `0x1EC`:

```bash
od -An -tu4 -j 492 -N 4 fsinfo.bin
```

Der Wert `0xFFFFFFFF` bedeutet bei beiden Feldern, dass der jeweilige Wert nicht bekannt ist. Außerdem handelt es sich beim freien Clusterzähler lediglich um einen zwischengespeicherten Hinweis. Der Wert kann veraltet sein und sollte bei einer forensischen Untersuchung nicht ungeprüft als korrekt angesehen werden.




## Metadaten und Zeitstempel

FAT speichert die Metadaten einer Datei direkt in ihrem 32 Byte großen Verzeichniseintrag. Dazu gehören der Dateiname, die Dateiattribute, der Startcluster, die Dateigröße sowie mehrere Zeit- und Datumsangaben.

Die vorhandenen Metadaten können zunächst mit `fls` aufgelistet werden:

```bash
fls -l -r -p partition.dd
```

Bei der Untersuchung des vollständigen Datenträgerabbilds muss der Startsektor der Partition angegeben werden:

```bash
fls -o "$STARTSEKTOR" -l -r -p EDF.dd
```

Vor jedem Eintrag zeigt `fls` eine Metadatenadresse an. Mit dieser Adresse können die Informationen einer einzelnen Datei ausführlicher dargestellt werden:

```bash
istat partition.dd METADATENADRESSE
```

Beispiel:

```bash
istat partition.dd 42
```

**[FOTO EINFÜGEN: Ausgabe von `fls` mit den Zeitstempeln einer ausgewählten Datei]**

**[FOTO EINFÜGEN: `istat`-Ausgabe derselben Datei]**

### Erstellungs-, Änderungs- und Zugriffszeit

Je nach FAT-Variante kann ein Verzeichniseintrag folgende Zeitangaben enthalten:

- Erstellungsdatum und Erstellungszeit
- Datum und Uhrzeit der letzten Änderung
- Datum des letzten Zugriffs

Der letzte Zugriff wird nur als Datum gespeichert. Eine Uhrzeit für den letzten Zugriff ist im FAT-Verzeichniseintrag nicht vorgesehen.

Die relevanten Felder eines regulären Verzeichniseintrags befinden sich an folgenden Positionen:

| Offset | Größe | Inhalt |
|---:|---:|---|
| `0x0D` | 1 Byte | Erstellungszeit in Zehntelsekunden |
| `0x0E` | 2 Byte | Erstellungszeit |
| `0x10` | 2 Byte | Erstellungsdatum |
| `0x12` | 2 Byte | Datum des letzten Zugriffs |
| `0x16` | 2 Byte | Uhrzeit der letzten Änderung |
| `0x18` | 2 Byte | Datum der letzten Änderung |

Um die Zeitstempel im Hexdump zu betrachten, wird zunächst der Sektor des übergeordneten Verzeichnisses ermittelt:

```bash
istat partition.dd VERZEICHNISADRESSE
```

Der durch `istat` angezeigte Sektor wird anschließend ausgelesen:

```bash
blkcat partition.dd SEKTORNUMMER > verzeichnis.bin
```

Die Darstellung mit 32 Byte pro Zeile erleichtert die Zuordnung zu den Verzeichniseinträgen:

```bash
xxd -g 1 -c 32 verzeichnis.bin
```

Ein bestimmter Verzeichniseintrag kann separat extrahiert werden:

```bash
dd if=verzeichnis.bin of=eintrag.bin \
   bs=1 skip=$((NUMMER * 32)) count=32 status=none
```

```bash
xxd -g 1 -c 32 eintrag.bin
```

Die Zeit- und Datumsfelder werden im Little-Endian-Format gespeichert. Ein Zwei-Byte-Wert muss daher entsprechend interpretiert werden.

Das FAT-Datumsformat besteht aus folgenden Bitfeldern:

| Bits | Bedeutung |
|---|---|
| 0–4 | Tag, Werte 1 bis 31 |
| 5–8 | Monat, Werte 1 bis 12 |
| 9–15 | Jahr seit 1980 |

Die Jahreszahl wird damit folgendermaßen berechnet:

```text
Jahr = 1980 + gespeicherter Jahreswert
```

Das FAT-Zeitformat besteht aus:

| Bits | Bedeutung |
|---|---|
| 0–4 | Sekunden geteilt durch 2 |
| 5–10 | Minuten |
| 11–15 | Stunden |

Die tatsächliche Sekundenzahl ergibt sich aus:

```text
Sekunden = gespeicherter Sekundenwert × 2
```

**[FOTO EINFÜGEN: Hexdump eines Verzeichniseintrags mit markierten Zeit- und Datumsfeldern]**

### Begrenzte Zeitstempelauflösung

Die zeitliche Auflösung von FAT ist begrenzt. Insbesondere die Änderungszeit kann Sekunden nur in Schritten von zwei Sekunden speichern. Ein Zeitwert wie `14:32:15` kann daher nicht exakt als reguläre FAT-Änderungszeit abgelegt werden.

Die Zeitstempel besitzen typischerweise folgende Auflösung:

| Zeitstempel | Auflösung |
|---|---|
| Erstellungszeit | nominell bis zu 10 Millisekunden über ein Zusatzfeld |
| Änderungszeit | 2 Sekunden |
| letzter Zugriff | 1 Tag |

Das Zusatzfeld der Erstellungszeit enthält einen Wert von 0 bis 199. Es ergänzt das normale Zwei-Sekunden-Zeitfeld und kann auch die ungerade Sekunde darstellen. Die genaue Behandlung dieses Feldes hängt jedoch vom Betriebssystem und vom verwendeten FAT-Treiber ab.

FAT speichert außerdem grundsätzlich keine Zeitzone. Die Zeitangaben werden üblicherweise als lokale Zeit abgelegt. Bei einer forensischen Auswertung muss deshalb die Zeitzone des untersuchten Systems berücksichtigt werden.

Unterschiede zwischen der Anzeige von `fls` und `istat` können durch eine explizite Zeitzone überprüft werden:

```bash
TZ=Europe/Berlin fls -l -r -p partition.dd
```

```bash
TZ=Europe/Berlin istat partition.dd METADATENADRESSE
```

**[FOTO EINFÜGEN: Vergleich der Zeitstempelanzeige mit und ohne gesetzte Zeitzone]**

### Keine Benutzer- und Berechtigungsinformationen

FAT besitzt kein mit Unix-Dateisystemen vergleichbares Rechte- und Eigentümermodell. In einem FAT-Verzeichniseintrag werden daher keine Benutzer-ID, Gruppen-ID oder detaillierten Zugriffsrechte gespeichert.

Insbesondere fehlen:

- Benutzer-ID und Gruppen-ID,
- Eigentümer einer Datei,
- Lese-, Schreib- und Ausführungsrechte für Benutzer und Gruppen,
- Access Control Lists,
- Sicherheitskennungen.

Stattdessen besitzt FAT lediglich ein Byte mit einfachen Dateiattributen. Dieses steht bei Offset `0x0B` des Verzeichniseintrags.

| Bitwert | Attribut |
|---:|---|
| `0x01` | schreibgeschützt |
| `0x02` | versteckt |
| `0x04` | Systemdatei |
| `0x08` | Datenträgerbezeichnung |
| `0x10` | Verzeichnis |
| `0x20` | Archiv |
| `0x0F` | Long-File-Name-Eintrag |

Das Attributbyte kann in einem extrahierten Verzeichniseintrag gezielt angezeigt werden:

```bash
xxd -g 1 -s 11 -l 1 eintrag.bin
```

Ein gesetztes Schreibschutzattribut ist nicht mit einer wirksamen Benutzerberechtigung moderner Dateisysteme gleichzusetzen. Es stellt lediglich eine einfache Kennzeichnung dar und kann leicht geändert oder ignoriert werden.

**[FOTO EINFÜGEN: Attributbyte eines Verzeichniseintrags im Hexdump]**

## Gelöschte Dateien

Beim Löschen einer Datei entfernt FAT normalerweise weder sofort den vollständigen Verzeichniseintrag noch den Dateiinhalt. Stattdessen wird der Eintrag als gelöscht gekennzeichnet und die zugehörigen Cluster werden in der FAT als frei markiert.

Solange die betreffenden Datenbereiche nicht überschrieben wurden, kann eine teilweise oder vollständige Wiederherstellung möglich sein.

Gelöschte Einträge können mit `fls` angezeigt werden:

```bash
fls -d -r -p partition.dd
```

Dabei beschränkt `-d` die Ausgabe auf gelöschte Einträge. Eine ausführliche Ausgabe erhält man mit:

```bash
fls -d -l -r -p partition.dd
```

Gelöschte und noch vorhandene Einträge können gemeinsam angezeigt werden:

```bash
fls -a -r -p partition.dd
```

Bei Verwendung des vollständigen Datenträgerabbilds wird wieder der Partitionsoffset benötigt:

```bash
fls -o "$STARTSEKTOR" -d -l -r -p EDF.dd
```

**[FOTO EINFÜGEN: Mit `fls -d` aufgelistete gelöschte Dateien]**

### Kennzeichnung gelöschter Verzeichniseinträge

Ein gelöschter Verzeichniseintrag wird im ersten Byte mit dem Wert `0xE5` gekennzeichnet. Die übrigen Metadaten des Eintrags können zunächst erhalten bleiben.

Der Verzeichnisbereich kann wie zuvor ausgelesen werden:

```bash
istat partition.dd VERZEICHNISADRESSE
```

```bash
blkcat partition.dd SEKTORNUMMER > verzeichnis.bin
```

Danach wird nach `0xE5` am Anfang eines 32-Byte-Eintrags gesucht:

```bash
xxd -g 1 -c 32 verzeichnis.bin
```

Ein gelöschter Eintrag könnte beispielsweise folgendermaßen beginnen:

```text
e5 41 54 45 49 20 20 20 54 58 54 ...
```

Das erste Byte war ursprünglich das erste Zeichen des kurzen 8.3-Dateinamens. Nach der Löschung ist dieses Zeichen nicht mehr unmittelbar bekannt.

Wichtig ist die Unterscheidung zwischen:

- `0xE5`: Eintrag wurde gelöscht.
- `0x00`: Dieser und alle nachfolgenden Einträge wurden noch nie verwendet beziehungsweise markieren das logische Verzeichnisende.

**[FOTO EINFÜGEN: Gelöschter Verzeichniseintrag mit `0xE5` im ersten Byte]**

### Verlust des ersten Zeichens im Dateinamen

Bei der Löschung wird das erste Byte des kurzen 8.3-Dateinamens durch `0xE5` ersetzt. Dadurch geht das ursprüngliche erste Zeichen verloren.

Aus einem ursprünglichen Namen wie:

```text
DATEI.TXT
```

wird im gelöschten Verzeichniseintrag beispielsweise:

```text
?ATEI.TXT
```

Die restlichen Zeichen, die Dateiendung, die Dateigröße, die Zeitstempel und der Startcluster können weiterhin im Eintrag vorhanden sein.

`fls` kennzeichnet gelöschte Dateien und ersetzt das verlorene Zeichen in der Darstellung häufig sinngemäß:

```bash
fls -d -l -r -p partition.dd
```

Die Metadaten des gelöschten Eintrags werden mit `istat` untersucht:

```bash
istat partition.dd METADATENADRESSE
```

Der Dateiinhalt kann mit `icat` anhand der Metadatenadresse wiederhergestellt werden:

```bash
icat partition.dd METADATENADRESSE > wiederhergestellt.bin
```

Für gelöschte Dateien kann zusätzlich die Recovery-Option `-r` verwendet werden:

```bash
icat -r partition.dd METADATENADRESSE > wiederhergestellt.bin
```

Das Ergebnis sollte anschließend geprüft werden:

```bash
file wiederhergestellt.bin
```

```bash
xxd -g 1 -l 512 wiederhergestellt.bin
```

```bash
sha256sum wiederhergestellt.bin
```

**[FOTO EINFÜGEN: Gelöschter Dateiname in `fls`, zugehöriger Eintrag in `istat` und Anfang des wiederhergestellten Inhalts im Hexdump]**

### Rekonstruktion von Clusterketten

Eine Datei kann über mehrere Cluster verteilt gespeichert sein. Bei einer vorhandenen Datei enthält die FAT für jeden Cluster einen Verweis auf den nachfolgenden Cluster. Der letzte Cluster wird durch eine End-of-Chain-Markierung gekennzeichnet.

Beim Löschen einer Datei werden die Einträge ihrer Clusterkette üblicherweise als frei markiert. Der Startcluster kann noch im Verzeichniseintrag vorhanden sein, die Verknüpfung der weiteren Cluster geht jedoch häufig verloren.

Zunächst wird der frühere Startcluster mit `istat` ermittelt:

```bash
istat partition.dd METADATENADRESSE
```

Zusätzlich kann der Verzeichniseintrag im Hexdump untersucht werden. Bei FAT32 setzt sich der Startcluster aus zwei Feldern zusammen:

| Offset | Größe | Inhalt |
|---:|---:|---|
| `0x14` | 2 Byte | obere 16 Bit des Startclusters |
| `0x1A` | 2 Byte | untere 16 Bit des Startclusters |
| `0x1C` | 4 Byte | Dateigröße |

Bei FAT12 und FAT16 werden nur die unteren 16 Bit bei Offset `0x1A` verwendet.

Die betreffenden Bytes können aus dem Eintrag ausgelesen werden:

```bash
xxd -g 1 -s 20 -l 12 eintrag.bin
```

Ist die frühere Clusterkette noch rekonstruierbar, kann `icat` versuchen, die Datei wiederherzustellen:

```bash
icat -r partition.dd METADATENADRESSE > wiederhergestellt.bin
```

Problematisch ist eine fragmentierte Datei. Wurden die FAT-Einträge bereits gelöscht, ist nur der erste Cluster sicher bekannt. Die weiteren Cluster müssen dann aus Dateigröße, Dateityp, Inhalt und nicht überschriebenen Datenbereichen rekonstruiert werden.

Die Clustergröße wird mit `fsstat` bestimmt:

```bash
fsstat partition.dd
```

Die Anzahl der für eine Datei benötigten Cluster lässt sich näherungsweise berechnen:

```text
Clusteranzahl = aufrunden(Dateigröße / Clustergröße)
```

In der Shell kann die Berechnung so durchgeführt werden:

```bash
DATEIGROESSE=10000
CLUSTERGROESSE=4096
CLUSTERANZAHL=$(((DATEIGROESSE + CLUSTERGROESSE - 1) / CLUSTERGROESSE))

echo "$CLUSTERANZAHL"
```

Falls die Datei zusammenhängend gespeichert war, können der Startcluster und die unmittelbar folgenden Cluster ausgelesen werden. Die Umrechnung eines Clusters in einen Dateisystemsektor lautet:

```text
Sektor = Beginn des Datenbereichs
       + (Clusternummer − 2) × Sektoren pro Cluster
```

Beispiel:

```bash
STARTCLUSTER=125
DATENBEREICH_START=1000
SEKTOREN_PRO_CLUSTER=8

DATEISEKTOR=$((DATENBEREICH_START + (STARTCLUSTER - 2) * SEKTOREN_PRO_CLUSTER))
```

Die benötigten Cluster können anschließend extrahiert werden:

```bash
blkcat partition.dd "$DATEISEKTOR" \
   $((CLUSTERANZAHL * SEKTOREN_PRO_CLUSTER)) > rekonstruiert.bin
```

Da der letzte Cluster mehr Daten enthalten kann, als zur Datei gehören, wird das Ergebnis auf die ursprüngliche Dateigröße gekürzt:

```bash
truncate -s "$DATEIGROESSE" rekonstruiert.bin
```

Die rekonstruierte Datei wird danach geprüft:

```bash
file rekonstruiert.bin
```

```bash
xxd -g 1 -l 512 rekonstruiert.bin
```

Diese Vorgehensweise funktioniert nur zuverlässig, wenn die Datei nicht fragmentiert war und ihre ehemaligen Cluster noch nicht überschrieben wurden. Bei einer fragmentierten gelöschten Datei reichen Startcluster und Dateigröße allein nicht aus, um die ursprüngliche Clusterkette eindeutig zu bestimmen.

**[FOTO EINFÜGEN: `istat`-Ausgabe mit Startcluster und Dateigröße]**

**[FOTO EINFÜGEN: Hexdump des gelöschten Verzeichniseintrags mit markiertem Startcluster]**

**[FOTO EINFÜGEN: Vergleich zwischen dem Hexdump der rekonstruierten Datei und dem erwarteten Dateiformat]**


## Nicht zugewiesener Speicher und Slack Space

Nicht zugewiesener Speicher umfasst Dateisystembereiche, die gegenwärtig keiner aktiven Datei zugeordnet sind. Bei FAT handelt es sich dabei hauptsächlich um Cluster, deren Einträge in der File Allocation Table als frei markiert sind.

Solche Bereiche können trotzdem Daten früherer Dateien enthalten. Beim Löschen einer Datei werden die Datencluster normalerweise nicht sofort überschrieben. Das Dateisystem markiert sie lediglich als verfügbar. Zusätzlich können sich im Slack Space einer vorhandenen Datei Reste älterer Inhalte befinden.

Vor der Untersuchung werden zunächst die Dateisystemparameter angezeigt:

```bash
fsstat partition.dd
```

Bei der Arbeit mit dem vollständigen Datenträgerabbild lautet der Befehl:

```bash
fsstat -o "$STARTSEKTOR" EDF.dd
```

Wichtige Werte sind:

- Bytes pro Sektor,
- Sektoren pro Cluster,
- Clustergröße,
- Beginn des Datenbereichs,
- Bereich der gültigen Cluster.

**[FOTO EINFÜGEN: `fsstat`-Ausgabe mit markierter Clustergröße und Lage des Datenbereichs]**

### Freie Cluster

Ein freier Cluster wird in der FAT durch einen Eintrag mit dem Wert null gekennzeichnet. Er ist keiner aktiven Datei zugewiesen, kann jedoch noch Daten einer gelöschten Datei enthalten.

Mit `blkstat` kann der Zustand eines bestimmten Dateisystemblocks beziehungsweise Sektors untersucht werden:

```bash
blkstat partition.dd BLOCKNUMMER
```

Beispiel:

```bash
blkstat partition.dd 2500
```

Die Ausgabe zeigt unter anderem, ob der Block zugewiesen oder nicht zugewiesen ist.

Bei Verwendung des vollständigen Datenträgerabbilds wird der Partitionsoffset ergänzt:

```bash
blkstat -o "$STARTSEKTOR" EDF.dd BLOCKNUMMER
```

Der Inhalt eines einzelnen Blocks kann anschließend mit `blkcat` ausgelesen werden:

```bash
blkcat partition.dd BLOCKNUMMER > block.bin
```

Der Block wird danach als Hexdump dargestellt:

```bash
xxd -g 1 block.bin
```

Für eine kompakte Vorschau reichen beispielsweise die ersten 512 Byte:

```bash
xxd -g 1 -l 512 block.bin
```

Zusätzlich können lesbare Zeichenfolgen gesucht werden:

```bash
strings -a -t x block.bin
```

Dabei zeigt `-t x` den jeweiligen Offset hexadezimal an.

Alle nicht zugewiesenen Dateisystemblöcke können mit `blkls` zusammengeführt und extrahiert werden:

```bash
blkls partition.dd > nicht_zugewiesen.bin
```

Direkt aus dem vollständigen Datenträgerabbild erfolgt die Extraktion mit:

```bash
blkls -o "$STARTSEKTOR" EDF.dd > nicht_zugewiesen.bin
```

Die Größe und der Hashwert des extrahierten Bereichs werden anschließend dokumentiert:

```bash
ls -lh nicht_zugewiesen.bin
sha256sum nicht_zugewiesen.bin
```

Die ersten Bytes können mit `xxd` betrachtet werden:

```bash
xxd -g 1 -l 512 nicht_zugewiesen.bin
```

Nach möglicherweise erhaltenen Texten kann mit `strings` gesucht werden:

```bash
strings -a -t x nicht_zugewiesen.bin | less
```

Eine gezielte Suche nach Begriffen ist ebenfalls möglich:

```bash
strings -a -t x nicht_zugewiesen.bin |
grep -i "Suchbegriff"
```

Zu beachten ist, dass `blkls` die freien Blöcke hintereinander in die Ausgabedatei schreibt. Ein Offset in `nicht_zugewiesen.bin` entspricht deshalb nicht automatisch demselben Offset im ursprünglichen Datenträgerabbild.

**[FOTO EINFÜGEN: `blkstat`-Ausgabe eines freien Blocks]**

**[FOTO EINFÜGEN: Hexdump eines freien Clusters mit noch vorhandenen Datenresten]**

### File Slack

Die logische Dateigröße entspricht nur selten exakt einem Vielfachen der Clustergröße. Der letzte Cluster einer Datei kann daher teilweise ungenutzt bleiben. Dieser ungenutzte Bereich wird als **File Slack** bezeichnet.

Beispiel:

```text
Dateigröße:     6.000 Byte
Clustergröße:   4.096 Byte
belegter Platz: 8.192 Byte
File Slack:     2.192 Byte
```

Die Größe des File Slack wird folgendermaßen berechnet:

```text
File Slack =
(Clustergröße − Dateigröße modulo Clustergröße)
modulo Clustergröße
```

In der Shell kann die Berechnung so durchgeführt werden:

```bash
DATEIGROESSE=6000
CLUSTERGROESSE=4096

SLACK_GROESSE=$(
    (CLUSTERGROESSE - DATEIGROESSE % CLUSTERGROESSE)
    % CLUSTERGROESSE
)

echo "$SLACK_GROESSE"
```

Zunächst wird die Metadatenadresse der zu untersuchenden Datei mit `fls` bestimmt:

```bash
fls -l -r -p partition.dd
```

Danach werden Dateigröße und belegte Blöcke mit `istat` angezeigt:

```bash
istat partition.dd METADATENADRESSE
```

Beispiel:

```bash
istat partition.dd 42
```

Der Inhalt der Datei kann mit `icat` extrahiert werden:

```bash
icat partition.dd 42 > datei_logisch.bin
```

`icat` gibt normalerweise nur den logischen Dateiinhalt bis zur gespeicherten Dateigröße aus. Der Slack Space ist darin nicht enthalten. Für seine Untersuchung muss der letzte durch `istat` angezeigte Block vollständig ausgelesen werden:

```bash
blkcat partition.dd LETZTER_BLOCK > letzter_block.bin
```

An welcher Position der Slack im letzten Cluster beginnt, ergibt sich aus:

```bash
REST=$((DATEIGROESSE % CLUSTERGROESSE))
```

Der Slack-Bereich wird extrahiert:

```bash
dd if=letzter_block.bin of=file_slack.bin \
   bs=1 skip="$REST" status=none
```

Falls die Datei mehrere Sektoren pro Cluster verwendet, müssen alle zum letzten Cluster gehörenden Sektoren mit `blkcat` ausgelesen werden. Die genaue Zuordnung ergibt sich aus `istat` und `fsstat`.

Der extrahierte Slack Space wird anschließend untersucht:

```bash
xxd -g 1 file_slack.bin
```

```bash
strings -a -t x file_slack.bin
```

Der gesamte Slack Space des Dateisystems kann außerdem mit `blkls` extrahiert werden:

```bash
blkls -s partition.dd > gesamter_slack.bin
```

Für das vollständige Datenträgerabbild gilt:

```bash
blkls -o "$STARTSEKTOR" -s EDF.dd > gesamter_slack.bin
```

Anschließend können Größe, Hashwert und Inhalt dokumentiert werden:

```bash
ls -lh gesamter_slack.bin
sha256sum gesamter_slack.bin
xxd -g 1 -l 512 gesamter_slack.bin
strings -a -t x gesamter_slack.bin | less
```

File Slack kann Reste zuvor gespeicherter Inhalte enthalten. Er kann aber auch vollständig mit Nullbytes gefüllt sein. Das hängt vom Betriebssystem, vom Treiber, vom Speichermedium und von vorherigen Schreibvorgängen ab.

**[FOTO EINFÜGEN: `istat`-Ausgabe mit Dateigröße und belegten Blöcken]**

**[FOTO EINFÜGEN: Hexdump des letzten Clusters mit markiertem Dateiende und anschließendem Slack Space]**

### Dateifragmente

Dateifragmente sind unvollständige Teile früherer oder beschädigter Dateien. Sie können sich in freien Clustern, im File Slack oder in teilweise überschriebenen Clusterketten befinden.

Zunächst kann der nicht zugewiesene Speicher nach lesbaren Zeichenfolgen durchsucht werden:

```bash
strings -a -t x nicht_zugewiesen.bin | less
```

Eine Suche nach bestimmten Textfragmenten erfolgt beispielsweise mit:

```bash
strings -a -t x nicht_zugewiesen.bin |
grep -i -C 3 "Suchbegriff"
```

Die Option `-C 3` zeigt zusätzlich drei Zeilen vor und nach jedem Treffer an.

Binäre Dateifragmente können anhand charakteristischer Dateisignaturen erkannt werden. Beispiele dafür sind:

| Dateityp | Typische Signatur |
|---|---|
| JPEG | `FF D8 FF` |
| PNG | `89 50 4E 47 0D 0A 1A 0A` |
| PDF | `25 50 44 46` |
| ZIP/DOCX/XLSX | `50 4B 03 04` |
| GIF | `47 49 46 38` |

Mit `xxd` und `grep` kann beispielsweise nach dem Beginn einer JPEG-Datei gesucht werden:

```bash
xxd -p nicht_zugewiesen.bin |
tr -d '\n' |
grep -bo "ffd8ff"
```

Die angezeigte Position bezieht sich in diesem Fall auf die hexadezimale Textdarstellung. Da jedes Byte durch zwei Hexzeichen dargestellt wird, wird der gefundene Wert durch zwei geteilt, um den Byte-Offset zu erhalten.

Eine praktischere binäre Suche kann mit `grep` erfolgen:

```bash
LC_ALL=C grep -aob $'\xFF\xD8\xFF' nicht_zugewiesen.bin
```

Nach einem PDF-Header wird folgendermaßen gesucht:

```bash
LC_ALL=C grep -aob '%PDF' nicht_zugewiesen.bin
```

Nach einem ZIP-Header kann so gesucht werden:

```bash
LC_ALL=C grep -aob $'\x50\x4B\x03\x04' nicht_zugewiesen.bin
```

Ein Bereich um einen gefundenen Byte-Offset kann anschließend extrahiert werden:

```bash
TREFFER_OFFSET=123456
LAENGE=4096

dd if=nicht_zugewiesen.bin of=fragment.bin \
   bs=1 skip="$TREFFER_OFFSET" count="$LAENGE" status=none
```

Das Fragment wird danach überprüft:

```bash
file fragment.bin
xxd -g 1 -l 512 fragment.bin
strings -a -t x fragment.bin
```

Ein gefundener Datei-Header beweist noch nicht, dass eine vollständige Datei vorliegt. Das Ende kann fehlen, die Daten können überschrieben worden sein oder die Datei kann ursprünglich fragmentiert gespeichert gewesen sein.

**[FOTO EINFÜGEN: Suche nach einer Dateisignatur im nicht zugewiesenen Speicher]**

**[FOTO EINFÜGEN: Hexdump eines gefundenen Dateifragments mit markierter Dateisignatur]**

### File Carving

Beim File Carving werden Dateien anhand ihres Inhalts und typischer Dateisignaturen rekonstruiert. Im Gegensatz zur Wiederherstellung mit `fls`, `istat` und `icat` benötigt Carving keinen erhaltenen Verzeichniseintrag.

Als Eingabe kann entweder das gesamte Partitionsabbild oder der zuvor mit `blkls` extrahierte nicht zugewiesene Speicher verwendet werden. Für die gezielte Untersuchung gelöschter Daten bietet sich zunächst der nicht zugewiesene Speicher an:

```bash
blkls partition.dd > nicht_zugewiesen.bin
```

Vor dem Carving wird der Hashwert dokumentiert:

```bash
sha256sum nicht_zugewiesen.bin
```

#### Carving mit Foremost

Mit `foremost` kann nach bekannten Dateisignaturen gesucht werden:

```bash
foremost -i nicht_zugewiesen.bin -o carving_foremost
```

Die Suche kann auf bestimmte Dateitypen begrenzt werden:

```bash
foremost -t jpg,png,pdf,zip \
   -i nicht_zugewiesen.bin \
   -o carving_foremost_auswahl
```

Foremost legt die gefundenen Dateien nach Dateitypen sortiert im Ausgabeordner ab. Zusätzlich wird normalerweise eine Protokolldatei erzeugt.

Die Ergebnisse können aufgelistet werden:

```bash
find carving_foremost_auswahl -type f -print
```

Die Dateitypen werden überprüft:

```bash
find carving_foremost_auswahl -type f \
   -exec file {} \;
```

#### Carving mit PhotoRec

Alternativ kann PhotoRec verwendet werden:

```bash
photorec nicht_zugewiesen.bin
```

PhotoRec arbeitet menügesteuert. Im Dialog können die zu suchenden Dateitypen und das Zielverzeichnis ausgewählt werden.

Soll direkt die gesamte Partition untersucht werden, kann PhotoRec mit der extrahierten Partition gestartet werden:

```bash
photorec partition.dd
```

#### Auswertung der gefundenen Dateien

Die Ergebnisse des Carvings sollten anschließend überprüft werden. Zunächst werden Hashwerte erzeugt:

```bash
find carving_foremost_auswahl -type f \
   -exec sha256sum {} \;
```

Einzelne Dateien können mit `file` klassifiziert werden:

```bash
file carving_foremost_auswahl/jpg/00000001.jpg
```

Der Anfang einer gefundenen Datei wird als Hexdump dargestellt:

```bash
xxd -g 1 -l 512 \
   carving_foremost_auswahl/jpg/00000001.jpg
```

Bei einem JPEG sollten am Anfang beispielsweise die Bytes `FF D8 FF` zu erkennen sein. Das Dateiende enthält bei einem vollständigen JPEG üblicherweise `FF D9`:

```bash
tail -c 32 carving_foremost_auswahl/jpg/00000001.jpg |
xxd -g 1
```

Mit einem Integritätstest kann geprüft werden, ob die interne Struktur einer gefundenen Datei plausibel ist. Beispiele sind:

```bash
pdfinfo gefundene_datei.pdf
```

```bash
unzip -t gefundene_datei.zip
```

```bash
identify gefundene_datei.jpg
```

File Carving besitzt mehrere Einschränkungen:

- Ursprüngliche Dateinamen und Pfade gehen normalerweise verloren.
- Zeitstempel und andere Verzeichnismetadaten fehlen.
- Fragmentierte Dateien können häufig nicht vollständig rekonstruiert werden.
- Header-Signaturen können zu Fehlalarmen führen.
- Bereits überschriebene Bereiche bleiben unwiederbringlich beschädigt.
- Mehrere Fragmente können irrtümlich zu einer Datei zusammengesetzt werden.

Deshalb müssen gecarvte Dateien stets inhaltlich und strukturell validiert werden.

**[FOTO EINFÜGEN: Ausgabe des Carving-Werkzeugs mit Anzahl und Typen der gefundenen Dateien]**

**[FOTO EINFÜGEN: Hexdump von Anfang und Ende einer wiederhergestellten Datei]**

**[FOTO EINFÜGEN: Prüfung einer gecarvten Datei mit `file` und einem formatspezifischen Prüfwerkzeug]**


## Timeline-Erstellung
- Auswertung der Verzeichniseinträge
- eingeschränkte Aussagekraft der Zeitstempel
