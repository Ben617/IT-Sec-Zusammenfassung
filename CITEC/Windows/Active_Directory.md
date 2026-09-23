# Active Directory – theoretische Grundlagen

## Lernziele

In diesem Kapitel:

- Zweck von Active Directory,
- lokale Benutzerverwaltung und Domänenverwaltung unterscheiden,
- die wichtigsten Bestandteile von Active Directory Domain Services einordnen,
- Domänen, Gesamtstrukturen, Organisationseinheiten und Objekte unterscheiden,
- den grundsätzlichen Ablauf einer Domänenanmeldung beschreiben,
- Benutzer, Gruppen und Gruppenrichtlinien fachlich einordnen,
- die Bedeutung von DNS, Kerberos, LDAP und Domain Controllern erklären.

## 1. Was ist Active Directory?

**Active Directory (AD)** ist eine Verzeichnisdienst-Technologie von Microsoft. Sie dient dazu, Identitäten, Computer und weitere Ressourcen eines Netzwerks zentral zu verwalten.

Statt Benutzerkonten auf jedem Computer einzeln anzulegen, können Organisationen Konten zentral speichern und verwalten. Benutzer melden sich dadurch mit einem Domänenkonto an unterschiedlichen freigegebenen Computern und Diensten an.

Active Directory ist kein einzelnes Programm und keine einfache Benutzerliste. Es besteht aus mehreren Rollen und Diensten. Der wichtigste Dienst für klassische Windows-Domänen ist **Active Directory Domain Services (AD DS)**.

AD DS übernimmt insbesondere:

- die zentrale Speicherung von Benutzern, Gruppen und Computern,
- die Authentifizierung von Benutzern und Computern,
- die Vergabe und Prüfung von Zugriffsrechten,
- die strukturierte Verwaltung von Objekten,
- die Verteilung zentraler Einstellungen über Gruppenrichtlinien,
- die Bereitstellung von Informationen für andere Netzwerkdienste.

> **Merksatz:** Active Directory beantwortet vor allem die Fragen: Wer oder was existiert im Netzwerk, wie wird seine Identität geprüft und auf welche Ressourcen darf es zugreifen?

## 2. Lokale Verwaltung und Domänenverwaltung

### Lokale Benutzerverwaltung

Ein lokales Benutzerkonto existiert nur auf einem bestimmten Windows-Computer. Die zugehörigen Kontodaten und Gruppenmitgliedschaften werden in dessen lokaler Sicherheitsdatenbank gespeichert.

Beispiel:

```text
PC-01\anna
```

Dieses Konto gehört zu `PC-01`. Ein gleichnamiges Konto auf `PC-02` wäre ein anderes Konto mit einer eigenen Sicherheitskennung.

### Domänenverwaltung

Ein Domänenkonto wird zentral in Active Directory gespeichert. Es kann – abhängig von den Berechtigungen – an verschiedenen Domänencomputern und für unterschiedliche Netzwerkdienste verwendet werden.

Beispiel:

```text
FIRMA\anna
anna@firma.example
```

| Lokales Konto | Domänenkonto |
|---|---|
| Gilt grundsätzlich auf einem Computer | Gilt innerhalb der Domäneninfrastruktur |
| Wird lokal verwaltet | Wird zentral in AD verwaltet |
| Lokale Anmeldung möglich | Anmeldung und Zugriff auf zentrale Ressourcen möglich |
| Lokale Gruppen und Richtlinien | Zentrale Gruppen und Gruppenrichtlinien |

Auch ein Domänencomputer besitzt weiterhin lokale Konten. Domänenverwaltung ersetzt die lokale Verwaltung daher nicht vollständig.

## 3. Zentrale Begriffe und Strukturen

### Objekt

Ein **Objekt** ist ein verwalteter Eintrag im Verzeichnis. Typische Objektarten sind:

- Benutzer,
- Gruppen,
- Computer,
- Drucker,
- Kontakte,
- Organisationseinheiten.

Jedes Objekt besitzt **Attribute**. Bei einem Benutzer können dies beispielsweise Anzeigename, Benutzername, E-Mail-Adresse und Gruppenmitgliedschaften sein.

### Schema

Das **Schema** legt fest, welche Objektklassen und Attribute im Verzeichnis existieren dürfen. Es definiert also die grundlegende Datenstruktur von Active Directory.

Schemaänderungen betreffen die gesamte Gesamtstruktur und müssen deshalb besonders sorgfältig geplant werden.

### Domäne

Eine **Domäne** ist eine logische Verwaltungs- und Sicherheitsgrenze innerhalb von AD DS. Sie enthält unter anderem Benutzer, Gruppen, Computer und Richtlinien.

Eine Domäne besitzt einen DNS-Namen, zum Beispiel:

```text
ad.firma.example
```

Alle Objekte einer Domäne teilen sich eine gemeinsame Verzeichnisdatenbank und können zentral verwaltet werden.

### Organisationseinheit

Eine **Organisationseinheit**, kurz **OU** (*Organizational Unit*), strukturiert Objekte innerhalb einer Domäne.

Beispiel:

```text
ad.firma.example
├── Benutzer
│   ├── Verwaltung
│   └── Vertrieb
└── Computer
    ├── Arbeitsplätze
    └── Notebooks
```

OUs werden vor allem verwendet, um:

- Verwaltungsaufgaben zu delegieren,
- Gruppenrichtlinien gezielt zuzuweisen,
- Objekte übersichtlich zu organisieren.

Eine OU ist keine Sicherheitsgruppe. Die Ablage eines Benutzers in einer OU gewährt ihm allein keine Zugriffsrechte.

### Strukturbehälter

Active Directory enthält neben OUs auch vordefinierte Container, beispielsweise `Users` und `Computers`. Ein Container kann Objekte aufnehmen, bietet aber nicht alle Verwaltungsfunktionen einer OU. Insbesondere lassen sich Gruppenrichtlinien nicht direkt mit jedem Standardcontainer verknüpfen.

### Baum

Ein **Domänenbaum** besteht aus einer oder mehreren Domänen mit einem zusammenhängenden DNS-Namensraum.

```text
firma.example
└── berlin.firma.example
```

### Gesamtstruktur

Die **Gesamtstruktur**, englisch **Forest**, ist die oberste logische Struktur von AD DS. Alle Domänen eines Forests teilen sich unter anderem:

- ein gemeinsames Schema,
- eine gemeinsame Konfiguration,
- einen globalen Katalog,
- automatisch aufgebaute Vertrauensstellungen innerhalb des Forests.

Ein Forest kann mehrere Domänenbäume enthalten. Er bildet die wichtigste Sicherheits- und Verwaltungsgrenze von Active Directory.

## 4. Domain Controller

Ein **Domain Controller (DC)** ist ein Server, auf dem AD DS ausgeführt wird. Er speichert eine Kopie der Verzeichnisdatenbank und stellt zentrale Domänendienste bereit.

Zu seinen Aufgaben gehören:

- Benutzer und Computer zu authentifizieren,
- Verzeichnisabfragen zu beantworten,
- Änderungen am Verzeichnis zu speichern,
- Gruppenrichtlinien bereitzustellen,
- Verzeichnisänderungen mit anderen Domain Controllern zu replizieren.

Produktive Domänen sollten mehrere Domain Controller besitzen. Fällt ein DC aus, können andere DCs weiterhin Anmeldungen und Verzeichnisdienste bereitstellen.

### Replikation

Mehrere Domain Controller halten ihre Verzeichnisdaten durch **Replikation** weitgehend auf demselben Stand. Änderungen können grundsätzlich auf verschiedenen beschreibbaren DCs vorgenommen und anschließend verteilt werden. Dieses Prinzip wird als **Multi-Master-Replikation** bezeichnet.

Bestimmte seltene Aufgaben dürfen dennoch nur von festgelegten DCs ausgeführt werden. Diese besonderen Zuständigkeiten heißen **FSMO-Rollen** (*Flexible Single Master Operations*).

Zu ihnen gehören:

- Schema-Master,
- Domänennamen-Master,
- RID-Master,
- PDC-Emulator,
- Infrastruktur-Master.

Für die Grundlagen genügt: AD arbeitet überwiegend mit mehreren gleichberechtigten DCs, einige Sonderaufgaben besitzen jedoch einen eindeutigen Rolleninhaber.

### Globaler Katalog

Der **globale Katalog** enthält eine Teilmenge der Attribute aller Objekte des Forests. Dadurch können Objekte domänenübergreifend gesucht und bestimmte Anmeldevorgänge unterstützt werden.

## 5. Active Directory und DNS

AD DS ist eng mit dem **Domain Name System (DNS)** verbunden. DNS löst nicht nur Namen in IP-Adressen auf, sondern hilft Clients auch dabei, passende Domain Controller und Dienste zu finden.

Dafür werden spezielle DNS-Einträge verwendet, insbesondere **SRV-Records**. Sie verweisen auf Server, die bestimmte Dienste anbieten.

Vereinfacht läuft die Suche so ab:

1. Ein Client fragt DNS nach einem zuständigen Domänendienst.
2. DNS liefert einen oder mehrere passende Domain Controller.
3. Der Client kontaktiert einen DC und beginnt die Authentifizierung oder Verzeichnisabfrage.

Ein fehlerhaft konfigurierter DNS-Client kann deshalb trotz erreichbarem Netzwerk zu Anmelde-, Gruppenrichtlinien- und Domänenbeitrittsproblemen führen.

> **Merksatz:** Ohne korrekt funktionierendes DNS funktioniert eine Active-Directory-Domäne nicht zuverlässig.

## 6. Authentifizierung und Autorisierung

Die beiden Begriffe beschreiben unterschiedliche Vorgänge:

- **Authentifizierung:** Prüfung der Identität – „Wer bist du?“
- **Autorisierung:** Prüfung einer Berechtigung – „Was darfst du?“

### Kerberos

**Kerberos** ist das bevorzugte Authentifizierungsprotokoll in modernen Windows-Domänen. Nach erfolgreicher Anmeldung erhält der Benutzer ein zeitlich begrenztes Ticket. Weitere Tickets ermöglichen den Zugriff auf Dienste, ohne das Kennwort bei jedem Zugriff erneut zu übertragen.

Vereinfacht:

1. Der Benutzer meldet sich an.
2. Der Domain Controller prüft die Identität.
3. Der Benutzer erhält ein Ticket Granting Ticket (TGT).
4. Für einen gewünschten Dienst wird ein Dienstticket angefordert.
5. Der Dienst prüft das Ticket und gewährt bei ausreichender Berechtigung Zugriff.

Kerberos ist auf eine annähernd übereinstimmende Uhrzeit der beteiligten Systeme angewiesen. Größere Zeitabweichungen können daher die Anmeldung verhindern.

### NTLM

**NTLM** ist ein älteres Authentifizierungsverfahren. Es wird aus Kompatibilitätsgründen noch in bestimmten Situationen verwendet, sollte aber, wo möglich, durch Kerberos ersetzt werden.

### LDAP

**LDAP** (*Lightweight Directory Access Protocol*) dient dazu, Verzeichnisinformationen abzufragen und zu verändern. LDAP ist nicht dasselbe wie Active Directory: AD DS ist der Verzeichnisdienst, LDAP eines der Protokolle für den Zugriff darauf.

### Zugriffstoken und SID

Nach einer erfolgreichen Anmeldung erzeugt Windows ein **Zugriffstoken**. Es enthält unter anderem:

- die Sicherheitskennung des Benutzers,
- die Sicherheitskennungen seiner Gruppen,
- bestimmte Benutzerrechte.

Eine **SID** (*Security Identifier*) ist eine eindeutige Sicherheitskennung. Windows vergibt Berechtigungen intern an SIDs und nicht unmittelbar an sichtbare Kontonamen. Wird ein Konto gelöscht und mit demselben Namen neu angelegt, besitzt es eine neue SID und erhält nicht automatisch die alten Berechtigungen.

## 7. Benutzer, Computer und Gruppen

### Benutzerkonten

Ein Domänenbenutzer repräsentiert normalerweise eine Person oder ein technisches Konto. Das Objekt speichert Identitäts- und Verwaltungsinformationen.

### Computerkonten

Auch ein Computer, der Mitglied der Domäne ist, besitzt ein eigenes Konto. Dieses Computerkonto ermöglicht eine Vertrauensbeziehung zwischen Computer und Domäne.

Beim **Domänenbeitritt** wird der Computer in die Domäne aufgenommen. Danach können sich berechtigte Domänenbenutzer anmelden und zentrale Richtlinien können angewendet werden.

### Gruppen

Gruppen fassen Benutzer, Computer oder andere Gruppen zusammen. Berechtigungen sollten möglichst Gruppen und nicht einzelnen Benutzern zugewiesen werden.

Man unterscheidet insbesondere:

- **Sicherheitsgruppen:** zur Vergabe von Berechtigungen,
- **Verteilergruppen:** hauptsächlich für E-Mail-Verteilung, nicht für Zugriffsrechte.

Sicherheitsgruppen besitzen außerdem unterschiedliche Gültigkeitsbereiche:

| Gruppenbereich | Grundidee |
|---|---|
| Domänenlokal | Berechtigungen auf Ressourcen einer Domäne zuweisen |
| Global | Konten mit ähnlicher Funktion innerhalb einer Domäne zusammenfassen |
| Universal | Domänenübergreifende Gruppierung im Forest |

Ein verbreitetes Modell für die Berechtigungsvergabe lautet **AGDLP**:

```text
Accounts → Global Groups → Domain Local Groups → Permissions
```

Beispiel:

```text
Benutzerin Anna
→ globale Gruppe GG_Vertrieb
→ domänenlokale Gruppe DL_Vertriebsordner_Lesen
→ NTFS-Berechtigung Lesen
```

Dieses Modell trennt organisatorische Zugehörigkeit von der konkreten Ressourcenberechtigung.

## 8. Gruppenrichtlinien

**Gruppenrichtlinien**, englisch **Group Policy**, ermöglichen die zentrale Konfiguration von Benutzern und Computern.

Eine Sammlung von Einstellungen heißt **Group Policy Object (GPO)**. GPOs können unter anderem mit folgenden Bereichen verknüpft werden:

- Sites,
- Domänen,
- Organisationseinheiten.

Typische Einsatzgebiete sind:

- Kennwort- und Sicherheitsrichtlinien,
- Firewall-Einstellungen,
- Desktop- und Systemeinstellungen,
- Softwarebereitstellung,
- Anmelde- und Startskripte,
- Einschränkung bestimmter Funktionen.

Die grundsätzliche Verarbeitungsreihenfolge wird häufig mit **LSDOU** beschrieben:

```text
Local → Site → Domain → Organizational Unit
```

Später angewendete Einstellungen können frühere Einstellungen überschreiben. Zusätzlich beeinflussen Vererbung, Priorität, Sicherheitsfilter und WMI-Filter das tatsächliche Ergebnis.

Gruppenrichtlinien bestehen grundsätzlich aus zwei Bereichen:

- **Computerkonfiguration:** gilt für Computer unabhängig vom angemeldeten Benutzer,
- **Benutzerkonfiguration:** gilt für Benutzer unabhängig vom verwendeten Computer, sofern keine Sonderregeln greifen.

## 9. Sites und Subnetze

Eine **AD-Site** bildet die physische beziehungsweise netzwerktechnische Struktur einer Organisation ab. Sie besteht typischerweise aus gut miteinander verbundenen IP-Subnetzen.

Sites helfen dabei:

- Clients einen nahegelegenen Domain Controller zuzuordnen,
- Replikation zwischen Standorten zu steuern,
- unnötigen Datenverkehr über langsame Verbindungen zu reduzieren.

Domänen und OUs bilden vor allem die logische Verwaltungsstruktur ab; Sites bilden die Netzwerktopologie ab.

## 10. Vertrauensstellungen

Eine **Vertrauensstellung** ermöglicht es, Identitäten aus einer Domäne in einer anderen Domäne zu authentifizieren. Vertrauen bedeutet jedoch nicht automatisch, dass Zugriff auf Ressourcen erlaubt ist. Die eigentliche Autorisierung erfolgt weiterhin über Berechtigungen.

Innerhalb eines Forests entstehen viele Vertrauensstellungen automatisch. Zwischen getrennten Forests oder bestimmten Domänen können Vertrauensstellungen gezielt eingerichtet werden.

Wichtige Eigenschaften sind:

- **einseitig oder beidseitig**,
- **transitiv oder nicht transitiv**.

## 11. Weitere Active-Directory-Dienste

Der Begriff Active Directory bezeichnet eine Produktfamilie. Dazu gehören unter anderem:

| Dienst | Zweck |
|---|---|
| Active Directory Domain Services (AD DS) | Klassische Domänen-, Identitäts- und Verzeichnisdienste |
| Active Directory Certificate Services (AD CS) | Ausstellung und Verwaltung digitaler Zertifikate |
| Active Directory Federation Services (AD FS) | Verbundidentitäten und föderierte Anmeldung |
| Active Directory Lightweight Directory Services (AD LDS) | LDAP-Verzeichnis ohne klassische Windows-Domäne |
| Active Directory Rights Management Services (AD RMS) | Schutz und Nutzungssteuerung von Informationen |

**Microsoft Entra ID** hieß früher Azure Active Directory. Trotz des früheren Namens ist es nicht einfach ein Domain Controller in der Cloud und kein vollständiger Ersatz für AD DS. Es ist ein cloudbasierter Identitäts- und Zugriffsverwaltungsdienst mit einem anderen Architekturmodell.

## 12. Der Anmeldevorgang – stark vereinfacht

Bei einer interaktiven Domänenanmeldung geschieht vereinfacht Folgendes:

1. Der Benutzer gibt seine Anmeldedaten ein.
2. Der Computer ermittelt über DNS einen Domain Controller.
3. Der Domain Controller authentifiziert das Konto, bevorzugt mit Kerberos.
4. Windows erstellt ein Zugriffstoken mit Benutzer- und Gruppen-SIDs.
5. Benutzer- und Computerrichtlinien werden verarbeitet.
6. Windows lädt das Benutzerprofil und startet die Sitzung.
7. Beim Zugriff auf eine Ressource vergleicht Windows das Token mit deren Berechtigungen.

Kann kein Domain Controller erreicht werden, ist auf einem bereits verwendeten Computer unter Umständen eine Anmeldung mit zwischengespeicherten Domäneninformationen möglich. Zentrale Dienste und aktuelle Richtlinien sind dann jedoch möglicherweise nicht verfügbar.

## 13. Sicherheit und bewährte Grundprinzipien

- Benutzer arbeiten mit möglichst wenigen Rechten.
- Administrative und normale Benutzerkonten werden getrennt.
- Berechtigungen werden nach Möglichkeit an Gruppen vergeben.
- Gruppenmitgliedschaften und privilegierte Konten werden regelmäßig geprüft.
- Mehrere Domain Controller erhöhen die Verfügbarkeit.
- DNS und Zeitsynchronisierung werden zuverlässig betrieben.
- Domain Controller werden besonders geschützt und zeitnah aktualisiert.
- Verzeichnisdaten und Systemzustand werden gesichert.
- Änderungen an Schema, Gesamtstruktur und privilegierten Gruppen werden kontrolliert durchgeführt.
- Alte Protokolle und unnötige Dienste werden soweit möglich deaktiviert.

## 14. Typische Missverständnisse

### „Eine Domäne ist nur ein DNS-Name.“

Nein. Sie verwendet zwar einen DNS-Namen, ist aber zusätzlich eine Verwaltungs-, Verzeichnis- und Sicherheitsstruktur.

### „Eine OU ist eine Gruppe.“

Nein. Eine OU strukturiert Objekte, ermöglicht Delegation und dient als Ziel für GPOs. Eine Sicherheitsgruppe bündelt Identitäten für Berechtigungen.

### „Wer authentifiziert ist, darf auf alle Domänenressourcen zugreifen.“

Nein. Authentifizierung bestätigt nur die Identität. Berechtigungen entscheiden anschließend über den Zugriff.

### „Active Directory und LDAP sind dasselbe.“

Nein. AD DS ist ein Verzeichnisdienst; LDAP ist ein Protokoll, mit dem auf Verzeichnisdaten zugegriffen werden kann.

### „Microsoft Entra ID ist Active Directory in der Cloud.“

Das ist zu stark vereinfacht. Entra ID und AD DS verwalten Identitäten, verwenden aber unterschiedliche Konzepte und Protokolle.

### „Ein zweiter Domain Controller ist automatisch ein vollständiges Backup.“

Nein. Replikation erhöht die Verfügbarkeit, überträgt aber auch viele fehlerhafte oder unerwünschte Änderungen. Eine geeignete Datensicherung bleibt notwendig.

## 15. Erste Orientierung bei Störungen

Bei AD-Problemen sollte die Diagnose zunächst systematisch erfolgen:

1. Ist der Client mit dem richtigen Netzwerk verbunden?
2. Hat er eine gültige IP-Konfiguration?
3. Verwendet er den vorgesehenen internen DNS-Server?
4. Kann der Domänenname aufgelöst werden?
5. Ist ein Domain Controller erreichbar?
6. Stimmen Datum, Uhrzeit und Zeitzone ausreichend überein?
7. Ist das Benutzer- oder Computerkonto aktiv und nicht gesperrt?
8. Betrifft der Fehler nur ein Konto, einen Computer oder viele Systeme?
9. Gibt es relevante Einträge in der Ereignisanzeige?
10. Wurde kürzlich eine Richtlinie, Berechtigung oder Netzwerkkonfiguration geändert?

Nützliche Windows-Werkzeuge für spätere Übungen sind unter anderem:

```powershell
ipconfig /all
nslookup ad.firma.example
whoami
whoami /groups
gpresult /r
```

Die Befehle werden in einem späteren Praxiskapitel detailliert behandelt.

## 16. Zusammenfassung

- AD DS verwaltet Identitäten und Ressourcen einer Windows-Domäne zentral.
- Domain Controller speichern das Verzeichnis und stellen Anmelde- und Verzeichnisdienste bereit.
- DNS ist entscheidend, damit Clients Domänendienste finden.
- Kerberos dient bevorzugt der Authentifizierung; LDAP ermöglicht Verzeichniszugriffe.
- Domänen enthalten Objekte; OUs strukturieren diese und dienen der Delegation sowie der GPO-Zuweisung.
- Gruppen vereinfachen die Vergabe von Berechtigungen.
- Ein Forest ist die oberste logische AD-DS-Struktur.
- Authentifizierung bestätigt eine Identität; Autorisierung entscheidet über den Zugriff.
- Gruppenrichtlinien konfigurieren Benutzer und Computer zentral.
- Replikation erhöht die Verfügbarkeit, ersetzt aber keine Datensicherung.

## 17. Kontrollfragen

1. Welchen Vorteil bietet ein Domänenkonto gegenüber einem lokalen Konto?
2. Welche Hauptaufgaben übernimmt ein Domain Controller?
3. Warum ist DNS für Active Directory besonders wichtig?
4. Worin unterscheiden sich Authentifizierung und Autorisierung?
5. Was ist der Unterschied zwischen einer OU und einer Sicherheitsgruppe?
6. Welche Funktion besitzt eine SID?
7. Welche Aufgaben haben Kerberos und LDAP?
8. Was unterscheidet Domäne, Domänenbaum und Gesamtstruktur?
9. Warum sollten Berechtigungen bevorzugt über Gruppen vergeben werden?
10. Was beschreibt das Modell AGDLP?
11. Wofür werden Gruppenrichtlinien eingesetzt?
12. Warum ersetzt ein zusätzlicher Domain Controller kein Backup?

## 18. Begriffsübersicht

| Begriff | Kurzbeschreibung |
|---|---|
| AD | Microsoft-Technologie für Verzeichnis- und Identitätsdienste |
| AD DS | Dienst für klassische Windows-Domänen |
| Domain Controller | Server, der AD DS bereitstellt |
| Domäne | Logische Verwaltungs- und Sicherheitsstruktur |
| Forest | Oberste logische AD-DS-Struktur |
| OU | Container für Organisation, Delegation und GPO-Zuweisung |
| Objekt | Verwalteter Verzeichniseintrag |
| Attribut | Eigenschaft eines Objekts |
| Schema | Definition erlaubter Objektklassen und Attribute |
| SID | Eindeutige Sicherheitskennung |
| DNS | Namensauflösung und Auffinden von AD-Diensten |
| LDAP | Protokoll für Verzeichniszugriffe |
| Kerberos | Bevorzugtes ticketbasiertes Authentifizierungsprotokoll |
| GPO | Sammlung zentraler Benutzer- oder Computereinstellungen |
| Replikation | Abgleich der Verzeichnisdaten zwischen Domain Controllern |
| Globaler Katalog | Domänenübergreifend durchsuchbare Teilmenge der Forest-Daten |
| Site | Abbildung der physischen Netzwerktopologie |
| Vertrauensstellung | Ermöglicht domänenübergreifende Authentifizierung |
