# Datenträgerforensik


## Datenträgeraufbau
- Sektoren und Blöcke
- physische und logische Blockadressierung
- LBA
- Advanced Format (4Kn, 512e)


## Partitionierungsschemata

### MBR/DOS
- Partitionstabelle
- primäre und erweiterte Partitionen
- Bootcode
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