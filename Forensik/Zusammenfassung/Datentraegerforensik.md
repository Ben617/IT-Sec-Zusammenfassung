# Datenträgerforensik


## [Datenträgeraufbau](Datenträgeraufbau.md)

## Partitionierungsschemata

### MBR/DOS
- Partitionstabelle
  - Zunächst einal die Hexdump-Darstellung des Beginns eines Datenträgers mit DOS-Partitionierungsschemata:
    ```bash
    xxd EDF.dd | less
    ```
    Ausgabe:
    ![Ausgabe von XXD des DOS-Partitionierungsschemata](../Forensik-Grafiken/Forensik-DOS-darstellung.png)
    - `Nicht markiert`: Bootcode und Fehlermeldungen
    - `Grün`: Datenträgersignatur in Little Endian => `0xFBB219F8`
    - `Rosa`: reserviert TODO
    - `Gelb und Rot`: 4 Partitionseinträge
    - `Bs`: Windows Signatur `55aa`

    Anschließend mit dd den MBR herauskopieren:
    ```bash
    dd if=EDF.dd of=mbr.dd bs=512 count=1 conv=noerror,sync status=progress
    ```

- primäre und erweiterte Partitionen
  - Danach die primäre Partitionstabelle anzeigen lassen:
  ```bash
  xxd -g 1 -s 0x1BE -l 64 mbr.dd
  ```
  Ausgabe:
  ![abc](../Forensik-Grafiken/Forensik-DOS-Partitionstabelle.png)
  
  - Bedeutung der Einzelnen Spalten in der Partitionstabelle:
  | Relativer Offset | Größe | Bedeutung | Besonderheit |
  |---:|---:|---|---|
  | `+0x00` | 1 Byte | Boot-Indikator | 80 = bootbar |
  | `+0x01` | 3 Byte | CHS-Startadresse | nicht genutzt |
  | `+0x04` | 1 Byte | Partitionstyp | |
  | `+0x05` | 3 Byte | CHS-Endadresse | nicht genutzt |
  | `+0x08` | 4 Byte | Start-LBA | |
  | `+0x0C` | 4 Byte | Anzahl der Sektoren | |
  
    - Partitionstyp:
  | Wert | Bedeutung |
  |---:|---|
  | `00` | leer |
  | `01` | FAT12 |
  | `02` |  |
  | `03` |  |
  | `04` |  |
  | `05` | erweiterte DOS-Partition |
  | `06` |  |
  | `07` | NTFS |
  | `0B` |  |
  | `0F` |  |
  | `82` | Linux Swap |
  | `83` | Linux |
  | `A5` | FreeBSD |
  | `A6` | OpenBSD |

    - Start-LBA:
  Der Wert ist im Little Endian gespeichert, daher lautet der echte Wert `00 00 08 00` und wird zu `0x00000800 = 2048` übersetzt.
  
    - Anzahl der Sektoren:
    Die letzten vier Bytes enthalten die Länge der Partition in Sektoren. Auch dieser Wert ist in Little Endian gespeichert.
    - End-LBA berechnen:
    > End-LBA = Start-LBA + Anzahl der Sektoren − 1

- Bootcode

Der Bootcode befindet sich am Anfang des MBR. Bei einem klassischen 
BIOS-Start sucht er in der Partitionstabelle nach einer als aktiv 
markierten Partition, lädt deren Volume Boot Record und übergibt diesem 
die weitere Ausführung.

Der Bootcode besitzt keine einheitlichen Flags, da es sich um ausführbare 
Maschinenbefehle handelt. Das Boot-Flag einer Partition steht stattdessen 
im ersten Byte ihres Partitionseintrags. Der Wert `0x80` kennzeichnet eine 
aktive Partition, während `0x00` eine inaktive Partition kennzeichnet.

- Ausgabe von mmls:
  ![abc](../Forensik-Grafiken/Forensik-DOS-mmls.png)
   
  Todo

### GPT
- Protective MBR
- primärer und sekundärer GPT-Header
- Partitionseinträge und Prüfsummen
- Microsoft Reserved Partition (MSR)
- EFI-Systempartition (ESP)


## Verborgene beziehungsweise nicht zugewiesene Bereiche
- HPA - Host Protected Area
- DCO - Device Configuration Overlay
- nicht zugewiesener Speicher
- Partition Gaps
- SSD Over-Provisioning
- Over-Provisioning bei SSDs


## Verschlüsselung:
- BitLocker
- LUKS
- FileVault
- VeraCrypt
- Self-Encrypting Drives