# NTFS

> NTFS (New Technology File System) ist das standardmäßig verwendete Dateisystem moderner Windows-Systeme. Es verwaltet Dateien und Verzeichnisse hauptsächlich als Datensätze innerhalb der Master File Table. Eine Datei wird dabei nicht nur als zusammenhängender Datenbereich betrachtet, sondern als Sammlung verschiedener Attribute.

Für die folgenden Untersuchungen wird zunächst die Partitionstabelle des Datenträgerabbilds angezeigt:

```bash
mmls EDF.dd
```

Die Nummer und der Startsektor der NTFS-Partition werden anschließend in Variablen eingetragen:

```bash
PARTITION=2
STARTSEKTOR=2048
```

> Die Werte müssen an die Ausgabe von `mmls` angepasst werden.

Die Partition kann in eine separate Datei kopiert werden:

```bash
mmcat EDF.dd "$PARTITION" > partition.dd
```

Grundlegende Informationen über das NTFS-Dateisystem liefert:

```bash
fsstat partition.dd
```

Alternativ kann das Dateisystem direkt im vollständigen Datenträgerabbild untersucht werden:

```bash
fsstat -o "$STARTSEKTOR" EDF.dd
```

**[FOTO EINFÜGEN: Ausgabe von `mmls` mit markierter NTFS-Partition]**

**[FOTO EINFÜGEN: Ausgabe von `fsstat` mit den zentralen NTFS-Strukturen]**

## Grundlagen

### Cluster und Logical Cluster Numbers

NTFS fasst mehrere Sektoren zu einem Cluster zusammen. Ein Cluster stellt die kleinste Speichereinheit dar, die einer nicht-resident gespeicherten Datei zugewiesen werden kann.

Die Clustergröße lässt sich mit `fsstat` bestimmen:

```bash
fsstat partition.dd
```

In der Ausgabe werden unter anderem die Sektorgröße und die Clustergröße angezeigt. Die Clustergröße ergibt sich aus:

```text
Clustergröße = Bytes pro Sektor × Sektoren pro Cluster
```

NTFS adressiert Cluster über sogenannte **Logical Cluster Numbers**, kurz LCN. Die Zählung beginnt am Anfang des Dateisystems mit LCN 0.

Die Byteposition eines Clusters lässt sich folgendermaßen berechnen:

```text
Byteposition = LCN × Clustergröße
```

Beispiel:

```bash
LCN=1000
CLUSTERGROESSE=4096
BYTEOFFSET=$((LCN * CLUSTERGROESSE))

echo "$BYTEOFFSET"
```

Ein Cluster kann mit `dd` anhand seines Byteoffsets ausgelesen werden:

```bash
dd if=partition.dd of=cluster.bin \
   bs=1 skip="$BYTEOFFSET" count="$CLUSTERGROESSE" status=none
```

Alternativ kann ein von `istat` angezeigter Dateisystemblock mit `blkcat` ausgegeben werden:

```bash
blkcat partition.dd BLOCKNUMMER > block.bin
```

Der Inhalt wird anschließend als Hexdump dargestellt:

```bash
xxd -g 1 block.bin
```

**[FOTO EINFÜGEN: `fsstat`-Ausgabe mit markierter Sektor- und Clustergröße]**

**[FOTO EINFÜGEN: Hexdump eines ausgewählten Clusters]**

### Dateien als Sammlung von Attributen

NTFS speichert jede Datei und jedes Verzeichnis als Datensatz in der Master File Table. Ein solcher MFT-Datensatz enthält mehrere Attribute.

Typische Attribute sind:

| Attribut | Typkennung | Bedeutung |
|---|---:|---|
| `$STANDARD_INFORMATION` | `0x10` | Zeitstempel, Flags und grundlegende Metadaten |
| `$ATTRIBUTE_LIST` | `0x20` | Verweise auf zusätzliche MFT-Datensätze |
| `$FILE_NAME` | `0x30` | Dateiname, übergeordnetes Verzeichnis und weitere Zeitstempel |
| `$OBJECT_ID` | `0x40` | optionale Objekt-ID |
| `$SECURITY_DESCRIPTOR` | `0x50` | ältere Form von Sicherheitsinformationen |
| `$DATA` | `0x80` | Dateiinhalt oder Datenstrom |
| `$INDEX_ROOT` | `0x90` | residenter Verzeichnisindex |
| `$INDEX_ALLOCATION` | `0xA0` | nicht-residente Teile eines Verzeichnisindexes |
| `$BITMAP` | `0xB0` | Belegungsinformationen eines Indexes |

Die Dateien eines NTFS-Dateisystems können mit `fls` aufgelistet werden:

```bash
fls -r -p partition.dd
```

Eine ausführliche Ausgabe mit Metadatenadressen und Zeitstempeln erhält man mit:

```bash
fls -l -r -p partition.dd
```

Die Attribute eines bestimmten MFT-Eintrags können mit `istat` angezeigt werden:

```bash
istat partition.dd MFT_NUMMER
```

Beispiel:

```bash
istat partition.dd 42
```

Die Ausgabe zeigt unter anderem, welche Attribute resident oder nicht-resident gespeichert sind und welche Cluster eine Datei belegt.

**[FOTO EINFÜGEN: `istat`-Ausgabe einer Datei mit mehreren NTFS-Attributen]**

## Zentrale Strukturen

### Master File Table (`$MFT`)

Die Master File Table ist die zentrale Datenbank eines NTFS-Dateisystems. Jede Datei und jedes Verzeichnis besitzt mindestens einen Eintrag in der MFT. Auch die internen Systemdateien von NTFS werden dort verwaltet.

Die Position und Größe der MFT werden mit `fsstat` angezeigt:

```bash
fsstat partition.dd
```

Die Systemdatei `$MFT` besitzt normalerweise die MFT-Nummer 0. Ihre Metadaten können daher folgendermaßen untersucht werden:

```bash
istat partition.dd 0
```

Die vollständige MFT kann mit `icat` extrahiert werden:

```bash
icat partition.dd 0 > MFT.bin
```

Anschließend werden Größe und Hashwert dokumentiert:

```bash
ls -lh MFT.bin
sha256sum MFT.bin
```

Der Beginn der MFT wird als Hexdump angezeigt:

```bash
xxd -g 1 -l 1024 MFT.bin
```

Ein MFT-Datensatz ist häufig 1024 Byte groß. Die tatsächliche Größe muss jedoch mit `fsstat` geprüft werden.

Ein bestimmter MFT-Datensatz kann anhand seiner Nummer extrahiert werden:

```bash
MFT_NUMMER=42
MFT_RECORD_GROESSE=1024

dd if=MFT.bin of=mft_record_42.bin \
   bs="$MFT_RECORD_GROESSE" \
   skip="$MFT_NUMMER" count=1 status=none
```

Der Datensatz wird danach untersucht:

```bash
xxd -g 1 mft_record_42.bin
```

Ein gültiger MFT-Datensatz beginnt normalerweise mit der Signatur:

```text
46 49 4c 45
```

Diese Bytefolge entspricht dem ASCII-Text `FILE`.

**[FOTO EINFÜGEN: `fsstat`-Ausgabe mit Lage und Größe der MFT]**

**[FOTO EINFÜGEN: Hexdump eines MFT-Datensatzes mit markierter FILE-Signatur]**

### MFT Mirror (`$MFTMirr`)

Die Datei `$MFTMirr` enthält eine Sicherungskopie der ersten wichtigen MFT-Datensätze. Dadurch können zentrale Dateisysteminformationen auch dann rekonstruiert werden, wenn der Anfang der ursprünglichen MFT beschädigt wurde.

`$MFTMirr` besitzt normalerweise die MFT-Nummer 1:

```bash
istat partition.dd 1
```

Die Datei kann mit `icat` extrahiert werden:

```bash
icat partition.dd 1 > MFTMirr.bin
```

Anschließend wird ihr Inhalt betrachtet:

```bash
xxd -g 1 -l 4096 MFTMirr.bin
```

Die ersten Datensätze der MFT und der MFT-Spiegelung können verglichen werden:

```bash
dd if=MFT.bin of=MFT_anfang.bin \
   bs=1 count="$(stat -c %s MFTMirr.bin)" status=none
```

```bash
cmp MFT_anfang.bin MFTMirr.bin
```

Abweichungen werden mit folgendem Befehl angezeigt:

```bash
cmp -l MFT_anfang.bin MFTMirr.bin | head
```

**[FOTO EINFÜGEN: Vergleich zwischen dem Anfang der MFT und `$MFTMirr`]**

### Volume Bitmap (`$Bitmap`)

Die Systemdatei `$Bitmap` beschreibt, welche Cluster des NTFS-Dateisystems belegt und welche frei sind. Jedes Bit repräsentiert einen Cluster:

- Bitwert `1`: Cluster ist belegt.
- Bitwert `0`: Cluster ist frei.

`$Bitmap` besitzt normalerweise die MFT-Nummer 6:

```bash
istat partition.dd 6
```

Die Bitmap kann extrahiert werden:

```bash
icat partition.dd 6 > Bitmap.bin
```

Der Anfang wird als Binär- und Hexdarstellung untersucht:

```bash
xxd -g 1 -l 512 Bitmap.bin
```

Ein Byte wie `0x0F` entspricht binär:

```text
00001111
```

Da die Bits innerhalb eines Bytes vom niederwertigsten zum höherwertigsten Bit ausgewertet werden, sind in diesem Beispiel die ersten vier dargestellten Cluster belegt und die folgenden vier frei.

Die Binärdarstellung der ersten Bytes kann beispielsweise mit `xxd` und `bc` oder einem geeigneten Skript weiter ausgewertet werden. Für eine erste Sichtprüfung genügt der Hexdump.

**[FOTO EINFÜGEN: `istat`-Ausgabe von `$Bitmap`]**

**[FOTO EINFÜGEN: Hexdump der Volume Bitmap mit markierten belegten und freien Bits]**

### Bootsektor (`$Boot`)

Der NTFS-Bootsektor befindet sich am Anfang der Partition. Er enthält den BIOS Parameter Block, Angaben zur Clustergröße, die Position der MFT, die Position von `$MFTMirr` und die Größe eines MFT-Datensatzes.

Der Bootsektor wird aus der Partition kopiert:

```bash
dd if=partition.dd of=ntfs_bootsektor.bin \
   bs=512 count=1 status=none
```

Alternativ kann er direkt aus dem vollständigen Datenträgerabbild extrahiert werden:

```bash
dd if=EDF.dd of=ntfs_bootsektor.bin \
   bs=512 skip="$STARTSEKTOR" count=1 status=none
```

Der Hexdump wird mit folgendem Befehl erzeugt:

```bash
xxd -g 1 ntfs_bootsektor.bin
```

Wichtige Felder des NTFS-Bootsektors sind:

| Offset | Größe | Inhalt |
|---:|---:|---|
| `0x03` | 8 Byte | OEM-Kennung, normalerweise `NTFS    ` |
| `0x0B` | 2 Byte | Bytes pro Sektor |
| `0x0D` | 1 Byte | Sektoren pro Cluster |
| `0x28` | 8 Byte | Gesamtzahl der Sektoren |
| `0x30` | 8 Byte | LCN des ersten MFT-Clusters |
| `0x38` | 8 Byte | LCN des ersten `$MFTMirr`-Clusters |
| `0x40` | 1 Byte | Größe eines MFT-Datensatzes |
| `0x44` | 1 Byte | Größe eines Indexdatensatzes |
| `0x48` | 8 Byte | Volume-Seriennummer |
| `0x1FE` | 2 Byte | Bootsektorsignatur `55 AA` |

Einzelne Bereiche können gezielt angezeigt werden:

```bash
xxd -g 1 -s 3 -l 8 ntfs_bootsektor.bin
```

```bash
xxd -g 1 -s 48 -l 16 ntfs_bootsektor.bin
```

```bash
xxd -g 1 -s 510 -l 2 ntfs_bootsektor.bin
```

**[FOTO EINFÜGEN: Hexdump des NTFS-Bootsektors mit OEM-Kennung, MFT-LCN und Signatur]**

### `$STANDARD_INFORMATION` und `$FILE_NAME`

Das Attribut `$STANDARD_INFORMATION` enthält grundlegende Metadaten einer Datei. Dazu gehören Zeitstempel, Dateiattribute und Verweise auf Sicherheitsinformationen.

Das Attribut `$FILE_NAME` enthält unter anderem:

- den Dateinamen,
- einen Verweis auf das übergeordnete Verzeichnis,
- eine weitere Gruppe von Zeitstempeln,
- die zugewiesene und tatsächliche Dateigröße.

Die Attribute werden mit `istat` angezeigt:

```bash
istat partition.dd MFT_NUMMER
```

Im Hexdump eines MFT-Datensatzes können die Attributkennungen gesucht werden:

```bash
xxd -g 1 mft_record_42.bin | less
```

Die Typkennung `0x10` erscheint wegen Little Endian als:

```text
10 00 00 00
```

Die Typkennung `0x30` erscheint als:

```text
30 00 00 00
```

Das Ende der Attributliste wird durch folgenden Wert markiert:

```text
ff ff ff ff
```

**[FOTO EINFÜGEN: Hexdump eines MFT-Datensatzes mit markiertem `$STANDARD_INFORMATION`-Attribut]**

**[FOTO EINFÜGEN: Hexdump desselben Datensatzes mit markiertem `$FILE_NAME`-Attribut]**

### Indexstrukturen

NTFS verwaltet den Inhalt eines Verzeichnisses als Index. Kleine Verzeichnisse können vollständig im residenten Attribut `$INDEX_ROOT` gespeichert werden. Bei größeren Verzeichnissen werden zusätzliche Indexdatensätze über `$INDEX_ALLOCATION` ausgelagert.

Die Einträge eines Verzeichnisses werden mit `fls` angezeigt:

```bash
fls partition.dd MFT_NUMMER_DES_VERZEICHNISSES
```

Die Attribute des Verzeichnisses werden mit `istat` untersucht:

```bash
istat partition.dd MFT_NUMMER_DES_VERZEICHNISSES
```

Im zugehörigen MFT-Datensatz sind folgende Typkennungen relevant:

```text
90 00 00 00    $INDEX_ROOT
a0 00 00 00    $INDEX_ALLOCATION
b0 00 00 00    $BITMAP
```

Nicht-residente Indexblöcke beginnen typischerweise mit der Signatur:

```text
49 4e 44 58
```

Diese entspricht dem ASCII-Text `INDX`.

Nach solchen Signaturen kann in extrahierten Bereichen gesucht werden:

```bash
LC_ALL=C grep -aob 'INDX' partition.dd | head
```

Ein gefundener Bereich kann anschließend untersucht werden:

```bash
TREFFER_OFFSET=123456

dd if=partition.dd of=index_fragment.bin \
   bs=1 skip="$TREFFER_OFFSET" count=4096 status=none
```

```bash
xxd -g 1 index_fragment.bin
```

**[FOTO EINFÜGEN: `istat`-Ausgabe eines Verzeichnisses mit `$INDEX_ROOT` und `$INDEX_ALLOCATION`]**

**[FOTO EINFÜGEN: Hexdump eines INDX-Datensatzes]**

## Metadaten und Zeitstempel

### MACB-Zeitstempel

In der forensischen Analyse werden Zeitstempel häufig mit der Abkürzung MACB beschrieben:

| Buchstabe | Bedeutung |
|---|---|
| M | Modified – Änderung des Dateiinhalts |
| A | Accessed – letzter Zugriff |
| C | Changed – Änderung des MFT-Eintrags |
| B | Birth – Erstellung der Datei |

Die Zeitstempel einer Datei werden mit `istat` angezeigt:

```bash
istat partition.dd MFT_NUMMER
```

Eine zeitlich sortierbare Bodyfile-Ausgabe kann mit `fls` erzeugt werden:

```bash
fls -r -m / partition.dd > ntfs.body
```

Aus dieser Datei kann mit `mactime` eine Zeitleiste erzeugt werden:

```bash
mactime -b ntfs.body > timeline.txt
```

Mit festgelegter Zeitzone lautet der Befehl beispielsweise:

```bash
mactime -z Europe/Berlin \
   -b ntfs.body > timeline_berlin.txt
```

Die ersten Einträge werden angezeigt:

```bash
head -n 20 timeline_berlin.txt
```

NTFS speichert Zeitstempel als Anzahl von 100-Nanosekunden-Intervallen seit dem 1. Januar 1601 UTC. Die praktisch nutzbare Genauigkeit hängt jedoch von der Anwendung und dem Betriebssystem ab, das den Zeitstempel geschrieben hat.

**[FOTO EINFÜGEN: `istat`-Ausgabe mit MACB-Zeitstempeln]**

**[FOTO EINFÜGEN: Ausschnitt aus der mit `mactime` erzeugten Zeitleiste]**

### Unterschiedliche Zeitstempel in `$STANDARD_INFORMATION` und `$FILE_NAME`

NTFS speichert Zeitstempel sowohl in `$STANDARD_INFORMATION` als auch in `$FILE_NAME`. Die beiden Attributgruppen müssen nicht identisch sein.

`$STANDARD_INFORMATION` wird bei regulären Dateioperationen normalerweise häufiger aktualisiert. Die Zeitstempel in `$FILE_NAME` werden insbesondere bei Änderungen am Verzeichniseintrag aktualisiert und können ältere Werte enthalten.

Beide Gruppen können mit `istat` verglichen werden:

```bash
istat partition.dd MFT_NUMMER
```

Zusätzlich kann der MFT-Datensatz im Hexdump untersucht werden:

```bash
MFT_NUMMER=42
MFT_RECORD_GROESSE=1024

dd if=MFT.bin of=mft_record_42.bin \
   bs="$MFT_RECORD_GROESSE" \
   skip="$MFT_NUMMER" count=1 status=none
```

```bash
xxd -g 1 mft_record_42.bin
```

Abweichende Zeitstempel können forensisch relevant sein. Sie sind jedoch nicht automatisch ein Beweis für Manipulation, da sie auch durch normale Dateioperationen entstehen können.

**[FOTO EINFÜGEN: Vergleich der Zeitstempel aus `$STANDARD_INFORMATION` und `$FILE_NAME`]**

### Eigentümer und Zugriffsrechte

NTFS unterstützt ein umfangreiches Sicherheitsmodell. Dazu gehören:

- Eigentümer,
- Sicherheitskennungen,
- Benutzer- und Gruppenrechte,
- Access Control Lists,
- Vererbungsregeln,
- Überwachungsinformationen.

Aktuelle NTFS-Versionen speichern gemeinsam verwendete Sicherheitsdeskriptoren zentral in der Systemdatei `$Secure`. Dateien verweisen über ihre Sicherheits-ID auf den jeweiligen Datensatz.

`$Secure` besitzt normalerweise die MFT-Nummer 9:

```bash
istat partition.dd 9
```

Die Datei kann zur weiteren Untersuchung extrahiert werden:

```bash
icat partition.dd 9 > Secure.bin
```

Der Anfang wird als Hexdump angezeigt:

```bash
xxd -g 1 -l 1024 Secure.bin
```

Sleuth Kit zeigt mit `istat` zwar die Security-ID einer Datei an, löst jedoch nicht in jedem Fall sämtliche Windows-Berechtigungen benutzerfreundlich auf. Für eine vollständige Interpretation sind gegebenenfalls zusätzliche NTFS- oder Windows-forensische Werkzeuge erforderlich.

**[FOTO EINFÜGEN: Security-ID einer Datei in der `istat`-Ausgabe]**

### Alternate Data Streams

NTFS erlaubt mehrere benannte `$DATA`-Attribute innerhalb einer Datei. Solche zusätzlichen Datenströme werden als **Alternate Data Streams**, kurz ADS, bezeichnet.

Ein ADS ist unter Windows beispielsweise über folgende Schreibweise erreichbar:

```text
datei.txt:versteckt
```

Die Attribute einer verdächtigen Datei werden mit `istat` untersucht:

```bash
istat partition.dd MFT_NUMMER
```

`fls` kann zusätzliche Attribute beziehungsweise Datenströme mit eigenen Attributadressen anzeigen:

```bash
fls -r -p partition.dd
```

Ein bestimmter Datenstrom kann über seine vollständige Attributadresse extrahiert werden. Eine solche Adresse kann beispielsweise folgendermaßen aussehen:

```text
42-128-2
```

Dabei bezeichnet `42` den MFT-Datensatz, `128` den dezimalen Attributtyp von `$DATA` und `2` die Attribut-ID.

Der Stream wird mit `icat` extrahiert:

```bash
icat partition.dd 42-128-2 > ads.bin
```

Danach wird der Inhalt untersucht:

```bash
file ads.bin
xxd -g 1 -l 512 ads.bin
strings -a -t x ads.bin
sha256sum ads.bin
```

**[FOTO EINFÜGEN: `istat`-Ausgabe mit einem benannten `$DATA`-Attribut]**

**[FOTO EINFÜGEN: Hexdump des extrahierten Alternate Data Streams]**

## Gelöschte Dateien

### Als frei markierte MFT-Einträge

Beim Löschen einer Datei wird der zugehörige MFT-Datensatz als nicht mehr verwendet markiert. Der Datensatz und seine Attribute können trotzdem noch vorhanden sein, bis der Eintrag wiederverwendet wird.

Gelöschte Dateien werden mit `fls` angezeigt:

```bash
fls -d -r -p partition.dd
```

Eine ausführliche Ausgabe erhält man mit:

```bash
fls -d -l -r -p partition.dd
```

Ein gelöschter Eintrag wird mit `istat` untersucht:

```bash
istat partition.dd MFT_NUMMER
```

Im Header eines MFT-Datensatzes befindet sich bei Offset `0x16` ein Zwei-Byte-Flags-Feld:

| Wert | Bedeutung |
|---:|---|
| `0x0000` | nicht verwendet |
| `0x0001` | verwendete Datei |
| `0x0002` | nicht verwendetes Verzeichnis |
| `0x0003` | verwendetes Verzeichnis |

Das Flags-Feld eines extrahierten MFT-Datensatzes kann gezielt angezeigt werden:

```bash
xxd -g 1 -s 22 -l 2 mft_record_42.bin
```

**[FOTO EINFÜGEN: Gelöschte Datei in der Ausgabe von `fls -d`]**

**[FOTO EINFÜGEN: MFT-Flags des gelöschten Eintrags im Hexdump]**

### Wiederverwendung von MFT-Einträgen

Freigegebene MFT-Datensätze können später für neue Dateien verwendet werden. NTFS verwaltet deshalb für jeden MFT-Datensatz eine Sequenznummer.

Ein Dateiverweis besteht intern aus:

- der Nummer des MFT-Datensatzes,
- der Sequenznummer dieses Datensatzes.

Wird ein Eintrag erneut verwendet, wird seine Sequenznummer erhöht. Alte Verweise können dadurch als veraltet erkannt werden.

Die Sequenznummer steht im MFT-Header bei Offset `0x10`:

```bash
xxd -g 1 -s 16 -l 2 mft_record_42.bin
```

Nach der Wiederverwendung können einzelne Attribute oder Datenreste des früheren Eintrags im ungenutzten Bereich des Datensatzes verbleiben. Diese Reste sind jedoch nicht mehr zuverlässig der ursprünglichen Datei zuzuordnen.

**[FOTO EINFÜGEN: Sequenznummer eines MFT-Datensatzes im Hexdump]**

### Wiederherstellung residenter und nicht-residenter Daten

Kleine Dateien können ihren gesamten Inhalt resident im `$DATA`-Attribut des MFT-Datensatzes speichern. In diesem Fall ist der Inhalt möglicherweise noch vorhanden, obwohl die Datei gelöscht wurde.

Ein Wiederherstellungsversuch erfolgt mit:

```bash
icat -r partition.dd MFT_NUMMER > wiederhergestellt.bin
```

Die Datei wird anschließend geprüft:

```bash
file wiederhergestellt.bin
xxd -g 1 -l 512 wiederhergestellt.bin
sha256sum wiederhergestellt.bin
```

Bei residenten Daten befindet sich der Dateiinhalt direkt im MFT-Datensatz. Im Attributheader steht bei Offset `0x08` des jeweiligen Attributs ein Kennzeichen:

- `0x00`: resident,
- `0x01`: nicht-resident.

Nicht-residente Dateien speichern ihren Inhalt außerhalb der MFT. Das `$DATA`-Attribut enthält dann sogenannte Data Runs, die auf die belegten Clusterbereiche verweisen.

Die Data Runs werden von `istat` interpretiert angezeigt:

```bash
istat partition.dd MFT_NUMMER
```

Sind MFT-Eintrag und Data Runs noch vorhanden und wurden die Cluster nicht überschrieben, kann `icat -r` die Datei häufig rekonstruieren. Wurden die Cluster bereits neu belegt, kann das Ergebnis unvollständig oder mit Daten anderer Dateien vermischt sein.

**[FOTO EINFÜGEN: Residentes `$DATA`-Attribut im Hexdump eines MFT-Datensatzes]**

**[FOTO EINFÜGEN: `istat`-Ausgabe einer nicht-residenten Datei mit Clusterbereichen]**

## Nicht zugewiesener Speicher und Slack Space

### Freie Cluster

Freie Cluster sind in `$Bitmap` mit dem Bitwert null gekennzeichnet. Sie können Reste gelöschter Dateien enthalten, solange sie nicht überschrieben wurden.

Mit `blkstat` wird geprüft, ob ein Dateisystemblock zugewiesen ist:

```bash
blkstat partition.dd BLOCKNUMMER
```

Ein einzelner Block wird mit `blkcat` extrahiert:

```bash
blkcat partition.dd BLOCKNUMMER > block.bin
```

```bash
xxd -g 1 block.bin
```

Der gesamte nicht zugewiesene Speicher kann mit `blkls` extrahiert werden:

```bash
blkls partition.dd > nicht_zugewiesen.bin
```

Bei Verwendung des vollständigen Datenträgerabbilds gilt:

```bash
blkls -o "$STARTSEKTOR" EDF.dd > nicht_zugewiesen.bin
```

Die Ausgabedatei wird dokumentiert und untersucht:

```bash
ls -lh nicht_zugewiesen.bin
sha256sum nicht_zugewiesen.bin
xxd -g 1 -l 512 nicht_zugewiesen.bin
strings -a -t x nicht_zugewiesen.bin | less
```

Da `blkls` die freien Bereiche in einer neuen Datei zusammenführt, stimmen die Offsets in `nicht_zugewiesen.bin` nicht unmittelbar mit den Offsets des ursprünglichen Datenträgerabbilds überein.

**[FOTO EINFÜGEN: Hexdump eines freien NTFS-Clusters mit vorhandenen Datenresten]**

### File Slack

Wenn die logische Dateigröße kein exaktes Vielfaches der Clustergröße ist, bleibt im letzten zugewiesenen Cluster ein ungenutzter Bereich. Dieser Bereich wird als File Slack bezeichnet.

Die ungefähre Slack-Größe berechnet sich aus:

```text
Slack-Größe =
(Clustergröße − Dateigröße modulo Clustergröße)
modulo Clustergröße
```

In der Shell kann die Berechnung so erfolgen:

```bash
DATEIGROESSE=6000
CLUSTERGROESSE=4096

SLACK_GROESSE=$(
    (CLUSTERGROESSE - DATEIGROESSE % CLUSTERGROESSE)
    % CLUSTERGROESSE
)

echo "$SLACK_GROESSE"
```

Dateigröße und belegte Cluster werden mit `istat` bestimmt:

```bash
istat partition.dd MFT_NUMMER
```

Der letzte belegte Cluster beziehungsweise Block wird anschließend vollständig ausgelesen:

```bash
blkcat partition.dd LETZTER_BLOCK > letzter_block.bin
```

Der Beginn des Slack-Bereichs innerhalb des letzten Clusters wird berechnet:

```bash
REST=$((DATEIGROESSE % CLUSTERGROESSE))
```

Der Slack wird extrahiert:

```bash
dd if=letzter_block.bin of=file_slack.bin \
   bs=1 skip="$REST" status=none
```

Die Untersuchung erfolgt mit:

```bash
xxd -g 1 file_slack.bin
strings -a -t x file_slack.bin
sha256sum file_slack.bin
```

Der Slack Space des gesamten Dateisystems kann mit `blkls` ausgegeben werden:

```bash
blkls -s partition.dd > gesamter_slack.bin
```

```bash
xxd -g 1 -l 512 gesamter_slack.bin
strings -a -t x gesamter_slack.bin | less
```

**[FOTO EINFÜGEN: Letzter Cluster einer Datei mit markiertem logischen Dateiende und Slack Space]**

### MFT Slack

Ein MFT-Datensatz besitzt eine feste Größe, wird aber nicht immer vollständig von Attributen ausgefüllt. Der Bereich zwischen dem Ende der verwendeten Attribute und dem Ende des Datensatzes wird als MFT Slack bezeichnet.

Im MFT-Header stehen:

| Offset | Größe | Inhalt |
|---:|---:|---|
| `0x18` | 4 Byte | tatsächlich verwendete Größe des Datensatzes |
| `0x1C` | 4 Byte | zugewiesene Größe des Datensatzes |

Die Werte werden im Little-Endian-Format gespeichert. Sie können im Hexdump angezeigt werden:

```bash
xxd -g 1 -s 24 -l 8 mft_record_42.bin
```

Die verwendete Größe kann auf einem Little-Endian-System mit `od` ausgegeben werden:

```bash
od -An -tu4 -j 24 -N 4 mft_record_42.bin
```

Die zugewiesene Größe wird folgendermaßen ausgegeben:

```bash
od -An -tu4 -j 28 -N 4 mft_record_42.bin
```

Nach Übernahme der Werte kann der MFT Slack extrahiert werden:

```bash
VERWENDETE_GROESSE=712
ZUGEWIESENE_GROESSE=1024
SLACK_LAENGE=$((ZUGEWIESENE_GROESSE - VERWENDETE_GROESSE))
```

```bash
dd if=mft_record_42.bin of=mft_slack.bin \
   bs=1 skip="$VERWENDETE_GROESSE" \
   count="$SLACK_LAENGE" status=none
```

Der MFT Slack wird anschließend untersucht:

```bash
xxd -g 1 mft_slack.bin
strings -a -t x mft_slack.bin
```

Darin können Reste früherer Attribute, Dateinamen oder residenter Daten vorhanden sein. Solche Fragmente müssen vorsichtig bewertet werden, da sie nicht zwingend zum aktuellen Inhalt des MFT-Datensatzes gehören.

**[FOTO EINFÜGEN: MFT-Header mit verwendeter und zugewiesener Datensatzgröße]**

**[FOTO EINFÜGEN: Hexdump des extrahierten MFT Slack]**

### File Carving

Beim File Carving werden Dateien anhand charakteristischer Signaturen und Strukturen gesucht. Verzeichniseinträge, Dateinamen und MFT-Metadaten werden dafür nicht zwingend benötigt.

Als Eingabe kann der nicht zugewiesene Speicher verwendet werden:

```bash
blkls partition.dd > nicht_zugewiesen.bin
```

Vor der weiteren Verarbeitung wird der Hashwert dokumentiert:

```bash
sha256sum nicht_zugewiesen.bin
```

Typische Dateisignaturen sind:

| Dateityp | Header |
|---|---|
| JPEG | `FF D8 FF` |
| PNG | `89 50 4E 47 0D 0A 1A 0A` |
| PDF | `25 50 44 46` |
| ZIP, DOCX oder XLSX | `50 4B 03 04` |
| GIF | `47 49 46 38` |

Nach einem JPEG-Header kann binär gesucht werden:

```bash
LC_ALL=C grep -aob $'\xFF\xD8\xFF' \
   nicht_zugewiesen.bin
```

Nach einem PDF-Header wird folgendermaßen gesucht:

```bash
LC_ALL=C grep -aob '%PDF' nicht_zugewiesen.bin
```

Ein Bereich an einem gefundenen Offset kann extrahiert werden:

```bash
TREFFER_OFFSET=123456
LAENGE=1048576

dd if=nicht_zugewiesen.bin of=fragment.bin \
   bs=1 skip="$TREFFER_OFFSET" \
   count="$LAENGE" status=none
```

Anschließend wird das Fragment geprüft:

```bash
file fragment.bin
xxd -g 1 -l 512 fragment.bin
strings -a -t x fragment.bin
```

Ein automatisiertes Carving kann beispielsweise mit Foremost durchgeführt werden:

```bash
foremost -i nicht_zugewiesen.bin \
   -o carving_foremost
```

Die Suche kann auf ausgewählte Dateitypen beschränkt werden:

```bash
foremost -t jpg,png,pdf,zip \
   -i nicht_zugewiesen.bin \
   -o carving_auswahl
```

Die gefundenen Dateien werden danach klassifiziert:

```bash
find carving_auswahl -type f -exec file {} \;
```

Zusätzlich werden Hashwerte erzeugt:

```bash
find carving_auswahl -type f -exec sha256sum {} \;
```

Eine gefundene Datei sollte auch anhand ihrer internen Struktur überprüft werden:

```bash
xxd -g 1 -l 512 carving_auswahl/jpg/00000001.jpg
```

```bash
tail -c 32 carving_auswahl/jpg/00000001.jpg |
xxd -g 1
```

File Carving besitzt mehrere Einschränkungen:

- Ursprüngliche Dateinamen und Pfade gehen meist verloren.
- MFT-Zeitstempel und Berechtigungen fehlen.
- Fragmentierte Dateien lassen sich häufig nicht vollständig rekonstruieren.
- Überschriebene Cluster führen zu beschädigten Ergebnissen.
- Dateisignaturen können Fehlalarme verursachen.
- Ein gefundener Header beweist nicht, dass die Datei vollständig erhalten ist.

Aus diesem Grund müssen gecarvte Dateien immer inhaltlich und strukturell validiert werden.

**[FOTO EINFÜGEN: Suche nach einer Dateisignatur im nicht zugewiesenen Speicher]**

**[FOTO EINFÜGEN: Ergebnis des File Carvings mit den gefundenen Dateitypen]**

**[FOTO EINFÜGEN: Hexdump vom Anfang und Ende einer wiederhergestellten Datei]**


## Journaling

NTFS verwendet Journaling-Mechanismen, um die Konsistenz des Dateisystems zu gewährleisten und Änderungen am Dateisystem zu protokollieren. Für die forensische Untersuchung sind insbesondere die Systemdateien `$LogFile` und `$UsnJrnl` relevant.

Die beiden Strukturen erfüllen unterschiedliche Aufgaben:

| Struktur | Hauptaufgabe |
|---|---|
| `$LogFile` | Protokollierung von NTFS-Transaktionen zur Wiederherstellung eines konsistenten Dateisystemzustands |
| `$UsnJrnl` | Protokollierung von Änderungen an Dateien und Verzeichnissen |

`$LogFile` ist primär ein Transaktionsjournal. `$UsnJrnl` ist dagegen ein Änderungsjournal, das unter anderem das Erstellen, Löschen, Umbenennen und Überschreiben von Dateien dokumentieren kann.

Die NTFS-Systemdateien lassen sich zunächst mit `fls` anzeigen:

```bash
fls -r -p partition.dd
```

Eine gezielte Suche erfolgt mit:

```bash
fls -r -p partition.dd |
grep -E '\$LogFile|\$UsnJrnl'
```

Bei der Untersuchung des vollständigen Datenträgerabbilds muss der Startsektor der NTFS-Partition angegeben werden:

```bash
fls -o "$STARTSEKTOR" -r -p EDF.dd |
grep -E '\$LogFile|\$UsnJrnl'
```

**[FOTO EINFÜGEN: Ausgabe von `fls` mit markiertem `$LogFile` und `$UsnJrnl`]**

### `$LogFile`

`$LogFile` ist das Transaktionsjournal von NTFS. Es protokolliert interne Änderungen an den Metadaten des Dateisystems. Nach einem Systemabsturz oder einer unerwarteten Unterbrechung kann NTFS diese Informationen verwenden, um angefangene Operationen rückgängig zu machen oder abzuschließen.

Typische protokollierte Vorgänge betreffen:

- Änderungen an MFT-Datensätzen,
- Änderungen an Verzeichnisindizes,
- Änderungen an Attributen,
- Zuweisung oder Freigabe von Clustern,
- Änderungen an NTFS-Metadaten.

`$LogFile` enthält nicht zwingend den vollständigen Inhalt einer geänderten Datei. Es protokolliert vor allem Transaktionen an Dateisystemstrukturen.

#### Metadaten von `$LogFile`

`$LogFile` besitzt normalerweise die MFT-Nummer 2. Die tatsächliche Adresse sollte dennoch mit `fls` überprüft werden:

```bash
fls -r -p partition.dd |
grep -F '$LogFile'
```

Die Metadaten werden mit `istat` angezeigt:

```bash
istat partition.dd 2
```

Die Ausgabe enthält unter anderem:

- die Größe von `$LogFile`,
- die enthaltenen Attribute,
- die belegten Clusterbereiche,
- den residenten oder nicht-residenten Speicherstatus,
- die Zeitstempel des MFT-Eintrags.

**[FOTO EINFÜGEN: `istat`-Ausgabe von `$LogFile`]**

#### Extraktion von `$LogFile`

Die Systemdatei kann mit `icat` extrahiert werden:

```bash
icat partition.dd 2 > LogFile.bin
```

Wird in der `fls`-Ausgabe eine vollständige Attributadresse angezeigt, kann diese ebenfalls verwendet werden. Ein Beispiel wäre:

```bash
icat partition.dd 2-128-1 > LogFile.bin
```

Die konkrete Attribut-ID kann abweichen und muss aus der eigenen `fls`- oder `istat`-Ausgabe übernommen werden.

Anschließend werden Größe und Hashwert dokumentiert:

```bash
ls -lh LogFile.bin
sha256sum LogFile.bin
```

Der Beginn der Datei wird als Hexdump dargestellt:

```bash
xxd -g 1 -l 4096 LogFile.bin
```

Zusätzlich kann nach lesbaren Zeichenfolgen gesucht werden:

```bash
strings -a -t x LogFile.bin | less
```

#### `$LogFile` im Hexdump

`$LogFile` ist in Seiten organisiert. Wichtige Seitensignaturen sind:

| Signatur | Bedeutung |
|---|---|
| `RSTR` | Restart Page |
| `RCRD` | Record Page |

Der Beginn einer Restart Page erscheint im Hexdump als:

```text
52 53 54 52
```

Dies entspricht dem ASCII-Text `RSTR`.

Eine Record Page beginnt typischerweise mit:

```text
52 43 52 44
```

Dies entspricht dem ASCII-Text `RCRD`.

Nach beiden Signaturen kann direkt in der Binärdatei gesucht werden:

```bash
LC_ALL=C grep -aob 'RSTR' LogFile.bin
```

```bash
LC_ALL=C grep -aob 'RCRD' LogFile.bin | head
```

Die Ausgabe enthält den dezimalen Byteoffset des jeweiligen Treffers. Ein gefundener Bereich kann anschließend separat extrahiert werden:

```bash
TREFFER_OFFSET=8192
SEITENGROESSE=4096

dd if=LogFile.bin of=logfile_page.bin \
   bs=1 skip="$TREFFER_OFFSET" \
   count="$SEITENGROESSE" status=none
```

Danach wird die Seite als Hexdump dargestellt:

```bash
xxd -g 1 logfile_page.bin
```

Alternativ kann direkt ab dem gefundenen Offset gelesen werden:

```bash
xxd -g 1 -s "$TREFFER_OFFSET" \
   -l "$SEITENGROESSE" LogFile.bin
```

**[FOTO EINFÜGEN: Hexdump einer Restart Page mit markierter `RSTR`-Signatur]**

**[FOTO EINFÜGEN: Hexdump einer Record Page mit markierter `RCRD`-Signatur]**

#### Forensische Bedeutung von `$LogFile`

Die Einträge in `$LogFile` können Hinweise auf kürzlich durchgeführte Dateisystemoperationen enthalten. Dazu gehören möglicherweise:

- frühere Zustände von MFT-Datensätzen,
- Änderungen an Dateinamen,
- Änderungen an Verzeichnisstrukturen,
- Zuweisung oder Freigabe von Clustern,
- Erstellen oder Löschen von Dateien,
- Verschieben oder Umbenennen von Dateien.

Da `$LogFile` eine begrenzte Größe besitzt, wird es zyklisch überschrieben. Ältere Transaktionen werden deshalb nach und nach durch neue Einträge ersetzt.

Die interne Struktur ist komplex. Ein einfacher Hexdump zeigt zwar Seitensignaturen und einzelne Datenfragmente, reicht für eine vollständige Interpretation der Transaktionen jedoch nicht aus. Dafür wird in der Regel ein spezieller NTFS-LogFile-Parser benötigt.

Trotzdem können bereits mit `strings` mögliche Dateinamen oder Pfadfragmente gefunden werden:

```bash
strings -a -el -t x LogFile.bin | less
```

Die Option `-el` sucht nach Little-Endian-UTF-16-Zeichenfolgen, wie sie bei NTFS-Dateinamen üblich sind.

Eine gezielte Suche nach einem bekannten Namen erfolgt beispielsweise mit:

```bash
strings -a -el -t x LogFile.bin |
grep -i "Dateiname"
```

Ein gefundener Text beweist allein noch nicht, dass die betreffende Datei zu einem bestimmten Zeitpunkt erstellt, verändert oder gelöscht wurde. Der Fund muss im Kontext des zugehörigen Logeintrags und anderer Dateisystemartefakte bewertet werden.

**[FOTO EINFÜGEN: In `$LogFile` gefundener UTF-16-Dateiname oder Pfadbestandteil]**

### `$UsnJrnl`

Das Update Sequence Number Journal, kurz USN Journal, protokolliert Änderungen an Dateien und Verzeichnissen eines NTFS-Volumes. Es wird unter anderem von Windows-Diensten, Sicherungssoftware und Suchindizierungsprogrammen verwendet.

Im Gegensatz zu `$LogFile` beschreibt `$UsnJrnl` die Art einer Änderung, enthält aber normalerweise nicht den vollständigen früheren oder neuen Dateiinhalt.

Mögliche protokollierte Ereignisse sind:

- Datei erstellt,
- Datei gelöscht,
- Datei umbenannt,
- Daten überschrieben,
- Daten erweitert oder gekürzt,
- Attribute geändert,
- Sicherheitsinformationen geändert,
- Verzeichnis erstellt oder entfernt,
- Datenstrom hinzugefügt oder entfernt.

#### Position innerhalb von `$Extend`

`$UsnJrnl` befindet sich normalerweise im Systemverzeichnis `$Extend`. Es besitzt mindestens zwei benannte Datenströme:

| Datenstrom | Bedeutung |
|---|---|
| `$J` | eigentliche Folge der USN-Datensätze |
| `$Max` | Konfiguration und Verwaltungsinformationen des Journals |

Die betreffenden Einträge werden mit `fls` gesucht:

```bash
fls -r -p partition.dd |
grep -F '$UsnJrnl'
```

Die Ausgabe kann beispielsweise mehrere Attributadressen für denselben MFT-Datensatz enthalten. Die genaue MFT-Nummer und die Attribut-IDs hängen vom untersuchten Dateisystem ab und dürfen nicht pauschal angenommen werden.

Die Metadaten des gefundenen MFT-Eintrags werden anschließend untersucht:

```bash
istat partition.dd MFT_NUMMER
```

`istat` zeigt die benannten `$DATA`-Attribute `$J` und `$Max` sowie deren Attribut-IDs an.

**[FOTO EINFÜGEN: `fls`-Ausgabe mit `$Extend/$UsnJrnl:$J` und `$Extend/$UsnJrnl:$Max`]**

**[FOTO EINFÜGEN: `istat`-Ausgabe des `$UsnJrnl`-MFT-Datensatzes]**

#### Extraktion des `$J`-Datenstroms

Zur Extraktion wird die vollständige Attributadresse aus der `fls`- oder `istat`-Ausgabe verwendet.

Beispiel:

```bash
icat partition.dd MFT-128-ATTRIBUTID > UsnJrnl_J.bin
```

Eine konkrete Adresse könnte beispielsweise so aussehen:

```bash
icat partition.dd 42-128-3 > UsnJrnl_J.bin
```

> `42-128-3` ist nur ein Beispiel. Die tatsächliche Adresse muss aus der Ausgabe des untersuchten Dateisystems übernommen werden.

Da `$J` als spärlich belegter, sogenannter Sparse-Datenstrom gespeichert sein kann, kann seine logische Größe deutlich größer als der tatsächlich belegte Speicher sein. Die Extraktion kann deshalb eine große Ausgabedatei erzeugen.

Größe und Hashwert werden dokumentiert:

```bash
ls -lh UsnJrnl_J.bin
sha256sum UsnJrnl_J.bin
```

Der Anfang wird mit `xxd` untersucht:

```bash
xxd -g 1 -l 4096 UsnJrnl_J.bin
```

Große Nullbereiche sind bei einem Sparse-Datenstrom nicht ungewöhnlich. Der Beginn der tatsächlichen USN-Datensätze kann deshalb weiter hinten liegen.

Nach UTF-16-Dateinamen kann mit `strings` gesucht werden:

```bash
strings -a -el -t x UsnJrnl_J.bin | less
```

Eine gezielte Suche erfolgt mit:

```bash
strings -a -el -t x UsnJrnl_J.bin |
grep -i "Dateiname"
```

**[FOTO EINFÜGEN: Hexdump des `$J`-Datenstroms mit einem USN-Datensatz]**

#### Extraktion des `$Max`-Datenstroms

Auch der `$Max`-Datenstrom wird anhand seiner vollständigen Attributadresse extrahiert:

```bash
icat partition.dd MFT-128-ATTRIBUTID > UsnJrnl_Max.bin
```

Beispiel:

```bash
icat partition.dd 42-128-4 > UsnJrnl_Max.bin
```

Der Inhalt wird anschließend untersucht:

```bash
ls -lh UsnJrnl_Max.bin
sha256sum UsnJrnl_Max.bin
xxd -g 1 UsnJrnl_Max.bin
```

`$Max` enthält Verwaltungsinformationen wie die maximal vorgesehene Journalgröße und die Größe, um die das Journal bei einer Bereinigung reduziert werden kann.

**[FOTO EINFÜGEN: Hexdump des `$Max`-Datenstroms]**

#### Aufbau eines USN-Datensatzes

Der `$J`-Datenstrom besteht aus einer Folge variabel großer USN-Datensätze. Der genaue Aufbau hängt von der verwendeten Version des Datensatzformats ab.

Ein verbreiteter USN-Record der Version 2 enthält unter anderem:

| Relativer Offset | Größe | Inhalt |
|---:|---:|---|
| `0x00` | 4 Byte | Länge des Datensatzes |
| `0x04` | 2 Byte | Major-Version |
| `0x06` | 2 Byte | Minor-Version |
| `0x08` | 8 Byte | Dateireferenznummer |
| `0x10` | 8 Byte | Dateireferenz des übergeordneten Verzeichnisses |
| `0x18` | 8 Byte | USN-Wert |
| `0x20` | 8 Byte | Zeitstempel |
| `0x28` | 4 Byte | Änderungsgrund |
| `0x2C` | 4 Byte | Quellinformationen |
| `0x30` | 4 Byte | Security-ID |
| `0x34` | 4 Byte | Dateiattribute |
| `0x38` | 2 Byte | Länge des Dateinamens |
| `0x3A` | 2 Byte | Offset des Dateinamens |
| ab Namensoffset | variabel | Dateiname in UTF-16LE |

Der erste Vier-Byte-Wert gibt die Gesamtlänge des jeweiligen Datensatzes an. Dadurch kann der Beginn des nächsten Records bestimmt werden.

Ein verdächtiger Datensatz kann zunächst als Ausschnitt extrahiert werden:

```bash
TREFFER_OFFSET=123456
RECORD_LAENGE=128

dd if=UsnJrnl_J.bin of=usn_record.bin \
   bs=1 skip="$TREFFER_OFFSET" \
   count="$RECORD_LAENGE" status=none
```

Die Rohdaten werden anschließend angezeigt:

```bash
xxd -g 1 usn_record.bin
```

Der enthaltene UTF-16LE-Dateiname kann mit folgendem Befehl sichtbar gemacht werden:

```bash
strings -a -el -t x usn_record.bin
```

**[FOTO EINFÜGEN: Hexdump eines USN-Records mit markierter Länge, Dateireferenz, Zeitstempel und Dateiname]**

#### Dateireferenznummern

Ein USN-Datensatz enthält die Dateireferenznummer der betroffenen Datei und die Dateireferenznummer des übergeordneten Verzeichnisses.

Eine NTFS-Dateireferenz setzt sich aus folgenden Bestandteilen zusammen:

- MFT-Datensatznummer,
- Sequenznummer des MFT-Datensatzes.

Dadurch kann ein USN-Ereignis mit einem MFT-Eintrag verbunden werden. Die Zuordnung kann jedoch fehlschlagen, wenn der MFT-Eintrag inzwischen wiederverwendet wurde.

Die ermittelte MFT-Nummer kann mit `istat` geprüft werden:

```bash
istat partition.dd MFT_NUMMER
```

Zusätzlich kann nach dem Eintrag in der Dateiliste gesucht werden:

```bash
fls -r -p partition.dd |
grep 'MFT_NUMMER'
```

Bei gelöschten Einträgen wird folgende Suche verwendet:

```bash
fls -d -r -p partition.dd |
grep 'MFT_NUMMER'
```

**[FOTO EINFÜGEN: Zuordnung eines USN-Ereignisses zu einem MFT-Datensatz]**

#### Änderungsgründe im USN Journal

Das Reason-Feld eines USN-Datensatzes ist eine Bitmaske. Mehrere Änderungsgründe können daher gleichzeitig gesetzt sein.

Häufige Werte sind:

| Wert | Bedeutung |
|---:|---|
| `0x00000001` | Daten überschrieben |
| `0x00000002` | Daten erweitert |
| `0x00000004` | Daten gekürzt |
| `0x00000100` | Datei oder Verzeichnis erstellt |
| `0x00000200` | Datei oder Verzeichnis gelöscht |
| `0x00001000` | Datei umbenannt, alter Name |
| `0x00002000` | Datei umbenannt, neuer Name |
| `0x00008000` | grundlegende Dateiinformationen geändert |
| `0x00010000` | Hardlink geändert |
| `0x00020000` | Komprimierung geändert |
| `0x00040000` | Verschlüsselung geändert |
| `0x00080000` | Objekt-ID geändert |
| `0x00100000` | Reparse Point geändert |
| `0x00200000` | benannter Datenstrom geändert |
| `0x80000000` | Vorgang geschlossen |

Da es sich um eine Bitmaske handelt, kann ein gespeicherter Wert aus mehreren dieser Werte zusammengesetzt sein. Ein Datensatz mit gesetztem Lösch-Bit zeigt, dass ein Löschereignis protokolliert wurde. Er bedeutet jedoch nicht automatisch, dass der Dateiinhalt noch wiederherstellbar ist.

#### Zeitstempel im USN Journal

USN-Records enthalten einen Windows-`FILETIME`-Zeitstempel. Dieser speichert die Anzahl der 100-Nanosekunden-Intervalle seit dem 1. Januar 1601 UTC.

Der Zeitstempel dokumentiert den Zeitpunkt des Journalereignisses. Er ist von den MACB-Zeitstempeln im MFT-Datensatz zu unterscheiden.

Dadurch können mehrere unterschiedliche Zeitangaben zu derselben Datei vorliegen:

- Zeitstempel in `$STANDARD_INFORMATION`,
- Zeitstempel in `$FILE_NAME`,
- Zeitstempel einzelner USN-Ereignisse,
- Transaktionsinformationen aus `$LogFile`.

Diese Werte beschreiben unterschiedliche Vorgänge und müssen getrennt interpretiert werden.

**[FOTO EINFÜGEN: Vergleich zwischen einem USN-Zeitstempel und den MFT-Zeitstempeln derselben Datei]**

### Gemeinsame Auswertung von `$LogFile` und `$UsnJrnl`

Die größte forensische Aussagekraft entsteht durch die gemeinsame Auswertung mehrerer NTFS-Artefakte:

| Artefakt | Mögliche Information |
|---|---|
| `$MFT` | aktueller oder gelöschter Dateieintrag, Attribute und Clusterzuordnung |
| `$LogFile` | interne NTFS-Transaktionen und frühere Metadatenzustände |
| `$UsnJrnl` | Art und Zeitpunkt einer Dateiänderung |
| Verzeichnisindex | Dateinamen und Verzeichniszuordnung |
| nicht zugewiesener Speicher | mögliche Reste des Dateiinhalts |

Ein typischer Untersuchungsablauf ist:

1. Verdächtige oder gelöschte Dateien mit `fls` identifizieren.
2. Den MFT-Datensatz mit `istat` untersuchen.
3. `$UsnJrnl:$J` nach Dateinamen und Dateireferenz durchsuchen.
4. `$LogFile` auf passende Dateinamen und Transaktionen prüfen.
5. MFT-, USN- und LogFile-Zeitstempel miteinander vergleichen.
6. Nicht zugewiesenen Speicher nach möglichen Dateiinhalten durchsuchen.
7. Alle extrahierten Artefakte mit Hashwerten dokumentieren.

Die wichtigsten Dateien werden beispielsweise folgendermaßen gehasht:

```bash
sha256sum \
   MFT.bin \
   LogFile.bin \
   UsnJrnl_J.bin \
   UsnJrnl_Max.bin
```

Bei der Interpretation sind folgende Einschränkungen zu berücksichtigen:

- Beide Journale werden zyklisch überschrieben.
- Nicht jede Dateioperation hinterlässt dauerhaft einen auswertbaren Eintrag.
- `$LogFile` enthält vor allem interne NTFS-Transaktionen.
- `$UsnJrnl` enthält Änderungsinformationen, aber normalerweise keine Dateiinhalte.
- MFT-Datensätze können nach einer Löschung wiederverwendet werden.
- Ein Dateiname kann mehrfach im Journal vorkommen.
- Zeitstempel verschiedener NTFS-Strukturen haben unterschiedliche Bedeutungen.
- Ein isolierter Journaleintrag sollte durch weitere Artefakte bestätigt werden.

**[FOTO EINFÜGEN: Gegenüberstellung der zugehörigen Einträge aus `$MFT`, `$LogFile` und `$UsnJrnl`]**
    
## Timeline-Erstellung
- Kombination von $MFT, $LogFile und $UsnJrnl
- Erkennung von Erstellung, Änderung, Umbenennung und Löschung
