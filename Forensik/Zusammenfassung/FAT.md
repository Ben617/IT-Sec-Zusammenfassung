- FAT

    > Typischer Einsatz: USB-Sticks, Speicherkarten und ältere Systeme
    
    - Grundbegriffe
        - FAT12, FAT16 und FAT32
        - Cluster und Clusterketten
    - Zentrale Strukturen:
        Zunächst wird die Partitionstabelle des Datenträgerabbilds untersucht. Dadurch lässt sich feststellen, unter welcher Partitionsnummer die FAT-Partition geführt wird und bei welchem Sektor sie beginnt.

        ```bash
        mmls EDF.dd
        ```

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


        # Todo Auswertung



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
        #todo sollte er gelcih sein?
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
