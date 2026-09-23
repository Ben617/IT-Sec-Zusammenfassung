# Linux Kurzfassung

## Installation

## Benutzter
- Standartduser wie root (id 0)
- Informationen über Benutzter: 
    ```bash
        id
    ```
- Benutzter hinzufügen und Gruppe hinzufügen:
    ```bash
        sudo adduser BENUTZERNAME 
        sudo adduser BENUTZERNAME --ingroup GRUPPENNAME 
    ```
- Benutzter löschen buw mit Home-Verzeichnis löschen:
    ```bash
        sudo deluser BENUTZERNAME
        sudo deluser --remove-home BENUTZERNAME 
    ```
- Benutzter zu Gruppe hinzufügen:
    ```bash
        sudo usermod -aG GRUPPENNAME BENUTZERNAME 
    ```
- Benutzter von Gruppe entfernen:
    ```bash
        sudo deluser BENUTZERNAME GRUPPENNAME 
    ```
- Standardgruppe eines Benutzers ändern:
    ```bash
        sudo usermod -g GRUPPENNAME BENUTZERNAME  
    ```
- Benutzernamen ändern:
    ```bash
        usermod -l NEUERNAME ALTERNAME 
        usermod -d /home/NEUERNAME -m NEUERNAME 
    ```
- Passwörter ändern:
    ```bash
        passwd 
        sudo passwd BENUTZERNAME 
    ```

## Gruppen
- Jeder Benutzter wird einer Hauptgruppe zugeordnet, kann aber Mitglied mehrerer Gruppen Spezialitäten

- Anzeigen der Benutzer in den Gruppen:
  ```bash
      less /etc/group
  ```

- Eine neue Gruppe erstellen:
  ```bash
      sudo addgroup GRUPPENNAME
  ```

- Einen Benutzer zu einer Gruppe hinzufügen:
  ```bash
      sudo usermod -aG GRUPPENNAME BENUTZERNAME
  ```

- Gruppen eines Benutzers anzeigen:
  ```bash
      groups BENUTZERNAME
  ```

- Einen Benutzer aus einer Gruppe entfernen:
  ```bash
      sudo gpasswd -d BENUTZERNAME GRUPPENNAME
  ```

- Eine Gruppe umbenennen:
  ```bash
      sudo groupmod -n NEUER_GRUPPENNAME ALTER_GRUPPENNAME
  ```

- Eine Gruppe löschen:
  ```bash
      sudo groupdel GRUPPENNAME
  ```
## Rechte
- Rechte einer Datei oder eines Ordners anzeigen:
  ```bash
      ls -l DATEI
  ```

- Besitzer einer Datei oder eines Ordners ändern:
  ```bash
      sudo chown BENUTZERNAME DATEI
  ```

- Besitzer und Gruppe ändern:
  ```bash
      sudo chown BENUTZERNAME:GRUPPENNAME DATEI
  ```

- Gruppe einer Datei oder eines Ordners ändern:
  ```bash
      sudo chgrp GRUPPENNAME DATEI
  ```

- Rechte für Benutzer, Gruppe und andere festlegen:
  ```bash
      chmod 750 DATEI
  ```

- Rechte mit Buchstaben bearbeiten:
  ```bash
      chmod u+rwx,g+rx,o-rwx DATEI
  ```

- Rechte eines Ordners und aller enthaltenen Dateien ändern:
  ```bash
      chmod -R 750 ORDNERNAME
  ```

- Besitzer und Gruppe rekursiv ändern:
  ```bash
      sudo chown -R BENUTZERNAME:GRUPPENNAME ORDNERNAME
  ```

Bedeutung der Buchstaben:

* `u` = Besitzer
* `g` = Gruppe
* `o` = andere Benutzer




## Programme
-apt install ist für die direkte Nutzung im Terminal gedacht. Es bietet eine übersichtlichere Ausgabe, Fortschrittsanzeigen und einfachere Befehle.
-apt-get install ist die ältere, stabilere Schnittstelle. Sie wird häufig in Skripten verwendet, weil sich ihre Ausgabe und ihr Verhalten seltener ändern.
- Paketlisten der Repositorys aktualisieren:
  ```bash
      sudo apt update
  ```
- Programme installieren:
  ```bash
      sudo apt-get install Name
      sudo apt install Name
  ```
- Programme deinstallieren:
  ```bash
      sudo apt-get remove Name
      sudo apt purge PROGRAMMNAME
      sudo apt-get remove Name
  ```
- Nicht mehr benötigte Pakete entfernen:
  ```bash
      sudo apt autoremove
  ```
- Programme updaten:
  ```bash
      sudo apt install --only-upgrade PROGRAMMNAME
  ```
- Abgebrochene Paketinstallationen fertigstellen::
  ```bash
      sudo dpkg --configure -a
  ```
- Nach einem Programm suchen::
  ```bash
      apt search PROGRAMMNAME
  ```
- Informationen zu einem Programm anzeigen:
  ```bash
      apt show PROGRAMMNAME
  ```


## Netzwerk

* Netzwerkschnittstellen und IP-Adressen anzeigen:

  ```bash
      ip address
  ```

* Kurze Übersicht der Netzwerkschnittstellen:

  ```bash
      ip -br address
  ```

* Routingtabelle und Standardgateway anzeigen:

  ```bash
      ip route
  ```

* Verbindung zu einem anderen Gerät testen:

  ```bash
      ping IP-ADRESSE
      ping HOSTNAME
  ```

* Weg der Netzwerkpakete verfolgen:

  ```bash
      traceroute HOSTNAME
  ```

* DNS-Auflösung überprüfen:

  ```bash
      nslookup HOSTNAME
      dig HOSTNAME
  ```

* Offene Ports und Netzwerkverbindungen anzeigen:

  ```bash
      ss -tulpen
  ```

* Netzwerkverbindungen mit NetworkManager anzeigen:

  ```bash
      nmcli connection show
  ```

* Netzwerkverbindung aktivieren:

  ```bash
      sudo nmcli connection up VERBINDUNGSNAME
  ```

* Netzwerkverbindung deaktivieren:

  ```bash
      sudo nmcli connection down VERBINDUNGSNAME
  ```

* Hostname anzeigen:

  ```bash
      hostname
      hostnamectl
  ```

* Hostname ändern:

  ```bash
      sudo hostnamectl set-hostname NEUER_HOSTNAME
  ```

* Wichtige Konfigurationsdateien:

  ```text
      /etc/hosts
      /etc/hostname
      /etc/resolv.conf
  ```

---

## Verzeichnisstruktur

* `/` – Wurzelverzeichnis des gesamten Systems

* `/bin` – wichtige Systembefehle

* `/boot` – Kernel und Dateien für den Systemstart

* `/dev` – Geräte und Datenträger

* `/etc` – systemweite Konfigurationsdateien

* `/home` – persönliche Verzeichnisse der Benutzer

* `/lib` – wichtige Programmbibliotheken

* `/media` – automatisch eingebundene Wechselmedien

* `/mnt` – manuell eingebundene Datenträger

* `/opt` – zusätzliche Programme

* `/proc` – Informationen über Prozesse und Kernel

* `/root` – persönliches Verzeichnis des Root-Benutzers

* `/run` – Laufzeitinformationen seit dem Systemstart

* `/srv` – Daten von Serverdiensten

* `/tmp` – temporäre Dateien

* `/usr` – installierte Programme und gemeinsam verwendete Dateien

* `/var` – veränderliche Daten wie Protokolle und Zwischenspeicher

* Aktuelles Verzeichnis anzeigen:

  ```bash
      pwd
  ```

* Inhalt eines Verzeichnisses anzeigen:

  ```bash
      ls
      ls -la
  ```

* Verzeichnis wechseln:

  ```bash
      cd VERZEICHNIS
  ```

* Neues Verzeichnis erstellen:

  ```bash
      mkdir VERZEICHNISNAME
  ```

* Leeres Verzeichnis löschen:

  ```bash
      rmdir VERZEICHNISNAME
  ```

* Speicherplatz der Dateisysteme anzeigen:

  ```bash
      df -h
  ```

* Größe eines Verzeichnisses anzeigen:

  ```bash
      du -sh VERZEICHNIS
  ```

---

## Shell und Bash

* Die Shell nimmt Befehle entgegen und führt sie aus.

* Bash ist eine häufig verwendete Linux-Shell.

* Verwendete Shell anzeigen:

  ```bash
      echo $SHELL
  ```

* Befehlshistorie anzeigen:

  ```bash
      history
  ```

* Bildschirm leeren:

  ```bash
      clear
  ```

* Handbuch eines Befehls öffnen:

  ```bash
      man BEFEHL
  ```

* Hilfe zu einem Befehl anzeigen:

  ```bash
      BEFEHL --help
  ```

* Ausgabe in eine Datei schreiben:

  ```bash
      BEFEHL > DATEI
  ```

* Ausgabe an eine Datei anhängen:

  ```bash
      BEFEHL >> DATEI
  ```

* Ausgabe eines Befehls an einen anderen Befehl weitergeben:

  ```bash
      BEFEHL1 | BEFEHL2
  ```

* Umgebungsvariablen anzeigen:

  ```bash
      printenv
  ```

* Variable erstellen:

  ```bash
      NAME="WERT"
  ```

* Umgebungsvariable exportieren:

  ```bash
      export NAME="WERT"
  ```

* Einfaches Bash-Skript:

  ```bash
      #!/bin/bash
      echo "Hallo Welt"
  ```

* Skript ausführbar machen und starten:

  ```bash
      chmod +x skript.sh
      ./skript.sh
  ```

---

## Prozesse

* Alle laufenden Prozesse anzeigen:

  ```bash
      ps aux
  ```

* Prozesse dynamisch anzeigen:

  ```bash
      top
  ```

* Einen bestimmten Prozess suchen:

  ```bash
      pgrep PROZESSNAME
  ```

* Prozess anhand seiner PID beenden:

  ```bash
      kill PID
  ```

* Prozess sofort beenden:

  ```bash
      kill -9 PID
  ```

* Prozess anhand seines Namens beenden:

  ```bash
      pkill PROZESSNAME
  ```

* Programm im Hintergrund starten:

  ```bash
      BEFEHL &
  ```

* Hintergrundprozesse der aktuellen Shell anzeigen:

  ```bash
      jobs
  ```

* Hintergrundprozess in den Vordergrund holen:

  ```bash
      fg
  ```

* Prozesspriorität beim Start festlegen:

  ```bash
      nice -n 10 BEFEHL
  ```

* Priorität eines laufenden Prozesses ändern:

  ```bash
      sudo renice PRIORITÄT -p PID
  ```

---

## Dienste

* Linux-Systeme verwenden häufig `systemd` zur Verwaltung von Diensten.

* Status eines Dienstes anzeigen:

  ```bash
      systemctl status DIENSTNAME
  ```

* Dienst starten:

  ```bash
      sudo systemctl start DIENSTNAME
  ```

* Dienst stoppen:

  ```bash
      sudo systemctl stop DIENSTNAME
  ```

* Dienst neu starten:

  ```bash
      sudo systemctl restart DIENSTNAME
  ```

* Konfiguration eines Dienstes neu laden:

  ```bash
      sudo systemctl reload DIENSTNAME
  ```

* Dienst beim Systemstart aktivieren:

  ```bash
      sudo systemctl enable DIENSTNAME
  ```

* Dienst beim Systemstart deaktivieren:

  ```bash
      sudo systemctl disable DIENSTNAME
  ```

* Dienst sofort starten und gleichzeitig aktivieren:

  ```bash
      sudo systemctl enable --now DIENSTNAME
  ```

* Alle aktiven Dienste anzeigen:

  ```bash
      systemctl list-units --type=service --state=running
  ```

* Alle fehlgeschlagenen Dienste anzeigen:

  ```bash
      systemctl --failed
  ```

---

## Protokolle und Fehlersuche

* Systemprotokolle anzeigen:

  ```bash
      journalctl
  ```

* Protokolle des aktuellen Systemstarts anzeigen:

  ```bash
      journalctl -b
  ```

* Protokolle eines bestimmten Dienstes anzeigen:

  ```bash
      journalctl -u DIENSTNAME
  ```

* Neue Protokolle fortlaufend anzeigen:

  ```bash
      journalctl -f
  ```

* Fehlermeldungen des aktuellen Systemstarts anzeigen:

  ```bash
      journalctl -b -p err
  ```

* Kernelmeldungen anzeigen:

  ```bash
      dmesg
  ```

* Letzte Zeilen einer Protokolldatei anzeigen:

  ```bash
      tail -n 50 /var/log/DATEI
  ```

* Änderungen an einer Protokolldatei verfolgen:

  ```bash
      tail -f /var/log/DATEI
  ```

* Nach Fehlern in einer Datei suchen:

  ```bash
      grep -i "error" DATEI
  ```

* CPU- und Arbeitsspeicherauslastung anzeigen:

  ```bash
      top
  ```

* Arbeitsspeicher anzeigen:

  ```bash
      free -h
  ```

* Freien Speicherplatz anzeigen:

  ```bash
      df -h
  ```

* Fehlgeschlagene Dienste überprüfen:

  ```bash
      systemctl --failed
  ```

* Wichtige Protokollverzeichnisse:

  ```text
      /var/log
      /var/log/syslog
      /var/log/auth.log
  ```

---

## Sicherheit

* Aktuellen Benutzer anzeigen:

  ```bash
      whoami
  ```

* Angemeldete Benutzer anzeigen:

  ```bash
      who
  ```

* Befehl mit Administratorrechten ausführen:

  ```bash
      sudo BEFEHL
  ```

* Dateirechte anzeigen:

  ```bash
      ls -l DATEI
  ```

* Dateirechte ändern:

  ```bash
      chmod 750 DATEI
  ```

* Besitzer und Gruppe ändern:

  ```bash
      sudo chown BENUTZERNAME:GRUPPENNAME DATEI
  ```

* System und Programme aktualisieren:

  ```bash
      sudo apt update
      sudo apt upgrade
  ```

* Firewallstatus unter Ubuntu anzeigen:

  ```bash
      sudo ufw status
  ```

* Firewall aktivieren:

  ```bash
      sudo ufw enable
  ```

* SSH-Verbindungen erlauben:

  ```bash
      sudo ufw allow OpenSSH
  ```

* Bestimmten Port erlauben:

  ```bash
      sudo ufw allow PORTNUMMER
  ```

* Letzte Anmeldungen anzeigen:

  ```bash
      last
  ```

* Fehlgeschlagene Anmeldeversuche anzeigen:

  ```bash
      sudo lastb
  ```

* Offene Ports überprüfen:

  ```bash
      sudo ss -tulpen
  ```

* Wichtige Sicherheitsmaßnahmen:

  * regelmäßig Updates installieren
  * starke und unterschiedliche Passwörter verwenden
  * Benutzern nur notwendige Rechte geben
  * nicht dauerhaft als `root` arbeiten
  * unbenötigte Dienste deaktivieren
  * Firewall aktivieren
  * SSH-Zugriff absichern
  * regelmäßige Datensicherungen erstellen



## Quellen:
- https://wiki.ubuntuusers.de/Benutzer_und_Gruppen/