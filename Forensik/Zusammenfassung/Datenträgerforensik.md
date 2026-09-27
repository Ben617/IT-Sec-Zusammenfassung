# Datenträgerforensik


## [Datenträgeraufbau](Datenträgeraufbau.md)

## Partitionierungsschemata

### MBR/DOS
- Partitionstabelle
  - Zunächst mit xxd die Sektoren und Blockgröße auslesen.
    ```bash
    Todo
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
  
  Todo

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