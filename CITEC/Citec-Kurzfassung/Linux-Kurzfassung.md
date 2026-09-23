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
##
##



## Quellen:
- https://wiki.ubuntuusers.de/Benutzer_und_Gruppen/