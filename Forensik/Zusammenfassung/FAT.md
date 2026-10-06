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

Eine rekursive Ausgabe aller Verzeichnisse und Dateien erfolgt mit:

```bash
fls -r -p partition.dd
```

Falls direkt mit dem vollständigen Datenträgerabbild gearbeitet wird, muss der Startsektor der Partition angegeben werden:

```bash
fls -o "$STARTSEKTOR" -r -p EDF.dd
```

Die ausführliche Ausgabe mit Zeitstempeln und Metadatenadressen erhält man mit:

```bash
fls -l -r -p partition.dd
```

Vor jedem Eintrag zeigt `fls` eine Metadatenadresse an. Diese Adresse kann anschließend an `istat` übergeben werden:

```bash
istat partition.dd METADATENADRESSE
```

Beispiel:

```bash
istat partition.dd 42
```

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
- Erstellungs-, Änderungs- und Zugriffszeit
- begrenzte Zeitstempelauflösung
- keine Benutzer- und Berechtigungsinformationen
## Gelöschte Dateien
- Kennzeichnung gelöschter Verzeichniseinträge
- Verlust des ersten Zeichens im Dateinamen
- Rekonstruktion von Clusterketten
## Nicht zugewiesener Speicher und Slack Space
- freie Cluster
- File Slack
- Dateifragmente
- File Carving
## Timeline-Erstellung
- Auswertung der Verzeichniseinträge
- eingeschränkte Aussagekraft der Zeitstempel
