# Benutzer-, Gruppen- und Rechteverwaltung

## Lernziele

In diesem Kapitel:

- lokale, Microsoft- und Domänenkonten unterscheiden,
- Benutzer- und Gruppenkonten fachlich einordnen,
- Administratoren und Standardbenutzer unterscheiden,
- Aufbau und Zweck eines Benutzerprofils erklären,
- NTFS-Berechtigungen lesen und grundlegend planen,
- Vererbung, effektive Berechtigungen und Besitz erklären,
- die Arbeitsweise der Benutzerkontensteuerung beschreiben,
- Authentifizierung und Autorisierung unterscheiden,
- den Zusammenhang zwischen Active Directory, Gruppen und Gruppenrichtlinien erklären.

## 1. Grundprinzip der Benutzer- und Rechteverwaltung

Windows muss bei jedem Zugriff drei grundlegende Fragen beantworten:

1. **Identifikation:** Welches Konto behauptet der Benutzer oder Prozess zu sein?
2. **Authentifizierung:** Kann diese Identität nachgewiesen werden?
3. **Autorisierung:** Welche Aktionen darf die bestätigte Identität ausführen?

Nach einer erfolgreichen Anmeldung erstellt Windows ein **Zugriffstoken**. Dieses enthält unter anderem:

- die Sicherheitskennung des Benutzers,
- die Sicherheitskennungen seiner Gruppen,
- bestimmte Benutzerrechte,
- Informationen zur Integritätsstufe und gegebenenfalls zur Erhöhung durch UAC.

Beim Zugriff auf eine Datei, einen Ordner oder ein anderes geschütztes Objekt vergleicht Windows dieses Token mit dessen Sicherheitsinformationen.

> **Merksatz:** Ein Benutzername ist für Menschen lesbar; Windows trifft Zugriffsentscheidungen hauptsächlich anhand von Sicherheitskennungen und Zugriffstoken.

## 2. Benutzerkonten und lokale Gruppen

### Benutzerkonten

Ein Benutzerkonto repräsentiert eine Identität. Es besitzt unter anderem:

- einen Kontonamen,
- eine eindeutige Sicherheitskennung (**SID**),
- Anmeldeinformationen,
- Gruppenmitgliedschaften,
- ein Benutzerprofil,
- Kontoeinstellungen wie Aktivierungsstatus oder Kennwortoptionen.

Ein Konto kann einer Person, einem technischen Dienst oder einem administrativen Zweck dienen. Für unterschiedliche Aufgaben sollten getrennte Konten verwendet werden.

### Lokale Kontodatenbank

Lokale Benutzer und Gruppen werden auf einem einzelnen Windows-System verwaltet. Ihre Kontoinformationen liegen in der lokalen **Security Accounts Manager**-Datenbank, kurz **SAM**.

Ein lokales Konto wird üblicherweise so angegeben:

```text
COMPUTERNAME\Benutzername
```

Beispiel:

```text
PC-01\anna
```

Das Konto ist grundsätzlich nur auf `PC-01` bekannt. Ein Konto `PC-02\anna` wäre trotz identischen Namens eine andere Identität mit einer anderen SID.

### Lokale Gruppen

Lokale Gruppen fassen Konten zusammen und erleichtern die Rechtevergabe. Statt einer großen Zahl einzelner Benutzer wird eine Gruppe berechtigt und die Mitgliedschaft zentral gepflegt.

Typische integrierte lokale Gruppen sind:

| Gruppe | Grundlegende Bedeutung |
|---|---|
| Administratoren | Weitreichende Verwaltungsrechte auf dem lokalen System |
| Benutzer | Normale Nutzung des Systems ohne vollständige Administration |
| Remotedesktopbenutzer | Erlaubnis zur Remotedesktopanmeldung, sofern weitere Bedingungen erfüllt sind |
| Ereignisprotokollleser | Lesender Zugriff auf Ereignisprotokolle |
| Leistungsprotokollbenutzer | Bestimmte Aufgaben zur Leistungsüberwachung |

Die genaue Wirkung hängt zusätzlich von lokalen Sicherheitsrichtlinien, NTFS-Berechtigungen und anderen Einstellungen ab.

### Konten und Gruppen verwalten

Je nach Windows-Edition und Aufgabe stehen unter anderem folgende Werkzeuge zur Verfügung:

- **Einstellungen** für alltägliche Kontoverwaltung,
- **Systemsteuerung** für klassische Optionen,
- **Lokale Benutzer und Gruppen** (`lusrmgr.msc`),
- **Computerverwaltung** (`compmgmt.msc`),
- **Eingabeaufforderung** mit `net user` und `net localgroup`,
- **PowerShell** mit Cmdlets wie `Get-LocalUser` und `Get-LocalGroup`.

Die MMC-Verwaltung lokaler Benutzer und Gruppen steht nicht in jeder Windows-Edition vollständig zur Verfügung.

## 3. Lokale, Microsoft- und Domänenkonten

Windows unterstützt unterschiedliche Kontotypen. Sie unterscheiden sich vor allem darin, wo die Identität gespeichert wird und wer sie verwaltet.

### Lokales Konto

Ein lokales Konto wird auf einem einzelnen Computer gespeichert und verwaltet.

**Eigenschaften:**

- gilt grundsätzlich nur für diesen Computer,
- funktioniert unabhängig von einem Onlinekonto,
- besitzt ein lokales Kennwort,
- synchronisiert Einstellungen nicht automatisch mit anderen Geräten,
- eignet sich für Einzelplatzsysteme, Testsysteme und spezielle Administrationszwecke.

### Microsoft-Konto

Ein Microsoft-Konto ist eine cloudbasierte Identität für Microsoft-Dienste. Es kann zur Windows-Anmeldung verwendet werden und Dienste oder Einstellungen geräteübergreifend verbinden.

**Eigenschaften:**

- zentrale Onlineidentität,
- Verbindung mit Microsoft-Diensten,
- mögliche Synchronisierung ausgewählter Einstellungen,
- Unterstützung moderner Anmeldeverfahren,
- Wiederherstellungsoptionen über das Onlinekonto.

Das Microsoft-Konto ist nicht dasselbe wie ein klassisches Active-Directory-Domänenkonto.

### Domänenkonto

Ein Domänenkonto wird zentral in **Active Directory Domain Services (AD DS)** verwaltet.

```text
DOMÄNE\Benutzername
Benutzername@domaene.example
```

**Eigenschaften:**

- zentrale Verwaltung durch eine Organisation,
- Anmeldung an berechtigten Domänencomputern,
- Zugriff auf zentrale Ressourcen,
- zentrale Gruppenmitgliedschaften und Sicherheitsrichtlinien,
- Anwendung von Gruppenrichtlinien.

### Microsoft Entra ID-Konto

Organisationen können Geräte auch mit **Microsoft Entra ID** verbinden. Entra ID ist ein cloudbasierter Identitätsdienst und architektonisch nicht mit einer klassischen AD-DS-Domäne gleichzusetzen.

Eine typische Kontoangabe kann so aussehen:

```text
AzureAD\name@firma.example
```

Die konkrete Darstellung hängt vom Anmelde- und Verwaltungskontext ab.

### Vergleich

| Merkmal | Lokales Konto | Microsoft-Konto | AD-Domänenkonto | Entra-ID-Konto |
|---|---|---|---|---|
| Verwaltung | Einzelner PC | Microsoft-Cloud | Organisation in AD DS | Organisation in der Cloud |
| Typischer Einsatz | Einzelplatz, Technik, Notfall | Private Windows-Nutzung | Klassisches Unternehmensnetz | Cloudverwaltete Organisation |
| Zentrale Richtlinien | Nur lokal | Begrenzt | Gruppenrichtlinien | Cloudbasierte Richtlinien und Verwaltung |
| Netzwerkweite Identität | Nein | Für Microsoft-Dienste | Innerhalb der Domäne | Für angebundene Clouddienste |

## 4. Administrator und Standardbenutzer

### Standardbenutzer

Ein Standardbenutzer kann typische Alltagsaufgaben ausführen, beispielsweise:

- eigene Dateien bearbeiten,
- installierte Anwendungen verwenden,
- persönliche Einstellungen verändern,
- zulässige Netzwerkressourcen nutzen.

Systemweite Änderungen sind dagegen eingeschränkt. Dazu zählen häufig die Installation bestimmter Programme, die Änderung sicherheitsrelevanter Einstellungen oder die Verwaltung anderer Konten.

### Administrator

Ein Mitglied der lokalen Gruppe **Administratoren** besitzt grundsätzlich weitreichende Verwaltungsrechte. Dazu gehören unter anderem:

- Software und Treiber installieren,
- Systemeinstellungen verändern,
- lokale Benutzer und Gruppen verwalten,
- Berechtigungen und Besitz anpassen,
- Dienste konfigurieren.

Ein Administratorkonto arbeitet wegen der Benutzerkontensteuerung nicht ständig mit allen möglichen Rechten. Für administrative Aktionen wird eine Erhöhung angefordert.

### Integriertes Administratorkonto

Windows besitzt ein eingebautes Konto namens **Administrator**. Es ist besonders privilegiert und auf modernen Clientinstallationen normalerweise deaktiviert oder speziell geschützt. Es ist nicht mit jedem beliebigen Konto gleichzusetzen, das Mitglied der Gruppe `Administratoren` ist.

### Prinzip der minimalen Rechte

Benutzer und Prozesse sollen nur die Rechte erhalten, die sie für ihre Aufgabe benötigen. Dieses Prinzip heißt **Least Privilege**.

Für administrative Arbeit empfiehlt sich eine Trennung:

- Standardkonto für alltägliche Aufgaben,
- separates Administratorkonto für Verwaltungsaufgaben.

Dadurch sinkt das Risiko, dass Schadsoftware oder Fehlbedienung sofort administrative Rechte ausnutzt.

## 5. Benutzerprofile

Ein Benutzerkonto beschreibt die Identität; das **Benutzerprofil** enthält die persönliche Arbeitsumgebung und benutzerspezifische Daten.

Standardmäßig befinden sich lokale Profile unter:

```text
C:\Users\Benutzername
```

Ein Profil enthält typischerweise:

- Desktop,
- Dokumente, Bilder und Downloads,
- anwendungsspezifische Daten,
- persönliche Einstellungen,
- benutzerspezifische Registry-Einstellungen.

### Wichtige Profilordner

| Ordner | Zweck |
|---|---|
| `Desktop` | Dateien und Verknüpfungen auf dem Desktop |
| `Documents` | persönliche Dokumente |
| `Downloads` | heruntergeladene Dateien |
| `AppData\Roaming` | potenziell übertragbare Anwendungsdaten |
| `AppData\Local` | computerbezogene Anwendungsdaten |
| `AppData\LocalLow` | Daten von Anwendungen mit niedriger Integritätsstufe |

Viele dieser Ordner sind standardmäßig ausgeblendet.

### NTUSER.DAT

Die Datei `NTUSER.DAT` enthält den benutzerspezifischen Registry-Bereich. Bei der Anmeldung wird er unter `HKEY_CURRENT_USER` eingebunden.

### Profilarten

- **Lokales Profil:** liegt auf einem bestimmten Computer.
- **Servergespeichertes Profil:** wird in klassischen Domänenumgebungen zentral gespeichert und beim An- und Abmelden übertragen.
- **Verbindliches Profil:** Änderungen des Benutzers werden nicht dauerhaft gespeichert.
- **Temporäres Profil:** wird geladen, wenn das reguläre Profil nicht verwendet werden kann; Änderungen können nach der Abmeldung verloren gehen.

In modernen Umgebungen werden häufig zusätzlich Ordnerumleitung, OneDrive oder andere Profilcontainer- und Synchronisierungslösungen eingesetzt.

> **Wichtig:** Das Löschen eines Benutzerkontos und das Löschen seines Profilordners sind unterschiedliche Vorgänge.

## 6. Rechte, Berechtigungen und Privilegien

Die Begriffe werden im Alltag oft gleich verwendet, bezeichnen aber unterschiedliche Konzepte.

### Berechtigungen

**Berechtigungen** beziehen sich auf ein bestimmtes Objekt, beispielsweise eine Datei oder einen Ordner.

Beispiele:

- Datei lesen,
- Datei ändern,
- Ordnerinhalt anzeigen,
- Datei löschen.

### Benutzerrechte

**Benutzerrechte** erlauben systemweite Aktionen und werden über Sicherheitsrichtlinien vergeben.

Beispiele:

- lokal anmelden,
- über Remotedesktop anmelden,
- System herunterfahren,
- Dateien und Verzeichnisse sichern,
- Besitz von Objekten übernehmen.

### Privilegien

Windows bildet viele Benutzerrechte intern als **Privilegien** im Zugriffstoken ab. Ein administratives Konto besitzt daher nicht automatisch Zugriff auf jedes Objekt, kann aber gegebenenfalls besondere Rechte einsetzen, um Konfiguration oder Besitz zu ändern.

## 7. NTFS-Berechtigungen

Das Dateisystem **NTFS** speichert für Dateien und Ordner Zugriffssteuerungsinformationen. Diese legen fest, welche Sicherheitsprinzipale welche Aktionen ausführen dürfen.

Als Sicherheitsprinzipal gelten unter anderem:

- Benutzer,
- Gruppen,
- Computer,
- Dienstkonten.

### Standardberechtigungen

Für Ordner werden häufig folgende zusammengefasste Berechtigungen angezeigt:

| Berechtigung | Grundlegende Wirkung |
|---|---|
| Vollzugriff | Lesen, Ändern, Löschen und Berechtigungen verwalten |
| Ändern | Lesen, Schreiben und Löschen |
| Lesen und Ausführen | Inhalte anzeigen und Programme ausführen |
| Ordnerinhalt anzeigen | Ordner durchsuchen und Inhalte auflisten |
| Lesen | Inhalte und Eigenschaften lesen |
| Schreiben | Dateien oder Daten erstellen beziehungsweise verändern |

Diese Einträge bestehen intern aus detaillierteren Einzelberechtigungen.

### DACL und ACE

Die **Discretionary Access Control List (DACL)** eines Objekts enthält einzelne Zugriffssteuerungseinträge, sogenannte **Access Control Entries (ACEs)**.

Ein ACE beschreibt vereinfacht:

- für welche SID er gilt,
- ob er Zugriff erlaubt oder verweigert,
- welche Einzelberechtigungen betroffen sind,
- ob und wie er vererbt wird.

### Zulassen und Verweigern

Windows kombiniert grundsätzlich die Berechtigungen aller relevanten Gruppenmitgliedschaften. Erlaubte Berechtigungen wirken daher kumulativ.

Ein explizites **Verweigern** kann eine entsprechende Erlaubnis überstimmen. Verweigerungseinträge sollten sparsam verwendet werden, da sie die Berechtigungsanalyse komplizierter machen.

### Datei- und Ordnerberechtigungen

Ordnerberechtigungen können auf folgende Elemente wirken:

- nur diesen Ordner,
- diesen Ordner und Unterordner,
- diesen Ordner, Unterordner und Dateien,
- nur Unterordner oder Dateien.

Deshalb reicht es nicht, nur den sichtbaren Berechtigungsnamen zu prüfen. Auch der Geltungsbereich eines Eintrags ist entscheidend.

## 8. Vererbung

Bei der **Vererbung** übernehmen untergeordnete Dateien und Ordner Berechtigungseinträge eines übergeordneten Ordners.

Beispiel:

```text
D:\Abteilungen             Gruppe Mitarbeitende: Lesen
└── Vertrieb               erbt: Lesen
    └── Angebote.docx      erbt: Lesen
```

Vorteile der Vererbung:

- einheitliche Berechtigungsstrukturen,
- weniger einzelne Einträge,
- einfachere Administration,
- geringere Fehleranfälligkeit.

Zusätzlich können Objekte **explizite Berechtigungen** besitzen, die direkt auf ihnen eingetragen wurden.

### Vererbung deaktivieren

Wird die Vererbung deaktiviert, bietet Windows üblicherweise zwei grundlegende Möglichkeiten:

- geerbte Einträge in explizite Einträge umwandeln,
- geerbte Einträge entfernen.

Das unüberlegte Entfernen geerbter Einträge kann Benutzer oder sogar Administratoren aussperren. Änderungen sollten deshalb zuerst geplant und anschließend mit einem Testkonto geprüft werden.

## 9. Effektive Berechtigungen

Die **effektiven Berechtigungen** sind die tatsächlich wirksamen Zugriffsrechte eines Benutzers oder einer Gruppe auf ein konkretes Objekt.

Sie ergeben sich unter anderem aus:

- direkten Benutzerberechtigungen,
- Mitgliedschaften in mehreren Gruppen,
- geerbten Berechtigungen,
- expliziten Berechtigungen,
- Zulassen- und Verweigern-Einträgen,
- dem Geltungsbereich der Einträge,
- gegebenenfalls Freigabeberechtigungen,
- der Art des Zugriffs und bestimmten Sonderrechten.

### Grundregeln

Als erste Orientierung gelten:

1. Erlaubnisse aus mehreren Gruppen werden kombiniert.
2. Explizite Einträge haben typischerweise Vorrang vor geerbten Einträgen.
3. Ein zutreffendes Verweigern hat in der normalen Auswertung meist Vorrang vor einem entsprechenden Zulassen.
4. Der konkrete Geltungsbereich eines Eintrags muss berücksichtigt werden.

Diese Regeln sind eine Vereinfachung. Windows verarbeitet ACEs in einer bestimmten Reihenfolge; schlecht sortierte oder komplexe ACLs können unerwartete Ergebnisse erzeugen.

### Zugriff über eine Netzwerkfreigabe

Bei Netzwerkzugriffen wirken sowohl **Freigabeberechtigungen** als auch **NTFS-Berechtigungen**. Der tatsächlich mögliche Zugriff entspricht der restriktiveren Kombination beider Ebenen.

Beispiel:

| Freigabe | NTFS | Ergebnis über das Netzwerk |
|---|---|---|
| Lesen | Ändern | Lesen |
| Vollzugriff | Lesen | Lesen |
| Ändern | Ändern | Ändern |

Bei lokalem Zugriff auf den Ordner gelten die Freigabeberechtigungen nicht; dort sind die NTFS-Berechtigungen maßgeblich.

## 10. Besitz von Dateien und Verzeichnissen

Jedes NTFS-Objekt besitzt einen **Besitzer**. Der Besitzer kann grundsätzlich die Berechtigungen des Objekts ändern, auch wenn ihm zuvor andere Zugriffe fehlen.

Typischerweise ist der Ersteller eines Objekts dessen Besitzer. Besitzer können aber auch Gruppen oder Systemkonten sein.

Administratoren besitzen nicht automatisch unmittelbaren Zugriff auf jede Datei. Sie verfügen jedoch häufig über das Benutzerrecht, den Besitz zu übernehmen. Nach der Besitzübernahme können sie passende Berechtigungen setzen.

### Sicherheitsrelevanz

Eine Besitzübernahme verändert die Sicherheitsinformationen des Objekts und kann Datenschutz-, Nachweis- oder Anwendungsprobleme verursachen. Sie sollte daher nur erfolgen, wenn sie notwendig und autorisiert ist.

### Besitz und Berechtigung unterscheiden

- **Besitz** bestimmt, wer die Zugriffssteuerung verwalten kann.
- **Berechtigungen** bestimmen, welche konkreten Zugriffe erlaubt sind.

Der Besitzer muss nicht automatisch Leserechte auf den Dateiinhalt besitzen, kann sich aber normalerweise entsprechende Berechtigungen geben.

## 11. Sicherheitskennungen und bekannte Identitäten

Windows identifiziert Konten intern durch **SIDs**. Ein Kontoname ist lediglich eine lesbare Zuordnung.

Beispiele für bekannte Identitäten sind:

- `SYSTEM`,
- `Administratoren`,
- `Benutzer`,
- `Authentifizierte Benutzer`,
- `Jeder`,
- `CREATOR OWNER`.

### Authentifizierte Benutzer

Diese Identität umfasst Konten, die sich erfolgreich authentifiziert haben. Sie ist nicht mit einer einzelnen Gruppe gleichzusetzen, die man manuell pflegt.

### Jeder

`Jeder` ist eine weit gefasste bekannte Identität. Ihre genaue Wirkung hängt von Windows-Version, Zugriffstyp und Konfiguration ab. Sie sollte bei sensiblen Ressourcen bewusst und nicht gedankenlos verwendet werden.

### Verwaiste SID

Wird ein Konto gelöscht, kann dessen SID weiterhin in einer ACL stehen. Windows zeigt dann gegebenenfalls nur noch eine Zeichenfolge wie diese an:

```text
S-1-5-21-…
```

Ein neu angelegtes Konto mit demselben Namen erhält eine neue SID und übernimmt diese Berechtigungen nicht automatisch.

## 12. Benutzerkontensteuerung (UAC)

Die **Benutzerkontensteuerung**, englisch **User Account Control (UAC)**, reduziert die unbemerkte Nutzung administrativer Rechte.

### Arbeitsweise

Meldet sich ein Mitglied der Administratorengruppe an, arbeitet es im Alltag normalerweise mit einem eingeschränkten Zugriffstoken. Fordert eine Aufgabe administrative Rechte, erscheint eine UAC-Abfrage und Windows kann einen erhöhten Prozess starten.

Bei einem Standardbenutzer verlangt die Abfrage in der Regel die Anmeldedaten eines administrativen Kontos.

### Arten von UAC-Abfragen

- **Zustimmungsabfrage:** Ein Administrator bestätigt die Erhöhung.
- **Anmeldeinformationsabfrage:** Ein Standardbenutzer gibt administrative Anmeldedaten ein.

Die Abfrage kann auf dem **sicheren Desktop** erscheinen. Andere Programme können diesen Bereich nicht ohne Weiteres bedienen.

### Was UAC nicht ist

UAC ist:

- ein Mechanismus zur Trennung normal gestarteter und erhöht gestarteter Prozesse,
- ein Mechanismus zur bewussten Freigabe administrativer Aktionen,
- ein Schutz gegen unbeabsichtigte Systemänderungen.

UAC gilt dabei nicht als vollständige Sicherheitsgrenze gegen Schadsoftware, die bereits im Kontext desselben Benutzers ausgeführt wird.

UAC ist kein Ersatz für:

- ein Standardbenutzerkonzept,
- Schadsoftwareschutz,
- Updates,
- sichere Kennwörter oder Mehrfaktorauthentifizierung,
- eine durchdachte Rechtevergabe.

### Virtualisierung

Für ältere Anwendungen kann UAC bestimmte Schreibzugriffe auf geschützte Bereiche in benutzerspezifische Orte umleiten. Diese Kompatibilitätsfunktion ist keine normale Lösung für moderne Anwendungen und gilt nicht uneingeschränkt.

## 13. Authentifizierung und Anmeldeverfahren

### Authentifizierungsfaktoren

Authentifizierungsmerkmale werden häufig in drei Kategorien eingeteilt:

- **Wissen:** etwas, das der Benutzer weiß, etwa ein Kennwort oder eine PIN,
- **Besitz:** etwas, das der Benutzer besitzt, etwa eine Smartcard oder ein Sicherheitsschlüssel,
- **Inhärenz:** ein körperliches Merkmal, etwa Fingerabdruck oder Gesichtserkennung.

Mehrfaktorauthentifizierung kombiniert Faktoren aus unterschiedlichen Kategorien.

### Kennwort

Ein Kennwort ist ein gemeinsames Geheimnis. Es sollte lang, einzigartig und nicht mehrfach verwendet werden. In verwalteten Umgebungen können Kennwortrichtlinien Mindestanforderungen und Sperrregeln festlegen.

### Windows Hello und Windows Hello for Business

Windows Hello ermöglicht die Anmeldung beispielsweise mit PIN, Gesichtserkennung oder Fingerabdruck. Die PIN ist an das Gerät gebunden und nicht einfach ein kürzeres Domänenkennwort.

**Windows Hello for Business** verwendet moderne, schlüsselbasierte Anmeldeverfahren für Organisationen und kann Kennwörter bei der täglichen Anmeldung ersetzen.

### Sicherheitsschlüssel und Smartcards

Hardwaregestützte Verfahren können kryptografische Schlüssel sicher speichern. Der private Schlüssel verlässt das Gerät im Normalfall nicht.

### Kerberos

Kerberos ist das bevorzugte Authentifizierungsprotokoll in klassischen Windows-Domänen. Nach der Anmeldung erhält der Benutzer Tickets für den Zugriff auf Dienste, ohne sein Kennwort bei jedem Zugriff erneut zu übertragen.

### NTLM

NTLM ist ein älteres Challenge-Response-Verfahren. Es wird noch aus Kompatibilitätsgründen verwendet, bietet jedoch weniger moderne Sicherheitsmerkmale als Kerberos und sollte möglichst eingeschränkt werden.

### Interaktive und Netzwerk-Anmeldung

Windows unterscheidet verschiedene Anmeldetypen, beispielsweise:

- interaktive Anmeldung direkt am Computer,
- Remotedesktopanmeldung,
- Netzwerkzugriff auf eine Freigabe,
- Dienstanmeldung,
- Stapelverarbeitung für geplante Aufgaben.

Ein Konto kann für einen Anmeldetyp berechtigt und für einen anderen gesperrt sein.

### Zwischengespeicherte Domänenanmeldung

War ein Domänenbenutzer bereits an einem Computer angemeldet, kann Windows unter bestimmten Bedingungen zwischengespeicherte Informationen für eine Anmeldung ohne erreichbaren Domain Controller verwenden. Dadurch entsteht jedoch keine vollständige Verbindung zur Domäne; aktuelle Richtlinien und Netzwerkressourcen können fehlen.

## 14. Active Directory als zentrale Benutzerverwaltung

**Active Directory Domain Services (AD DS)** speichert Benutzer, Gruppen, Computer und weitere Objekte zentral. Domain Controller authentifizieren Konten und stellen Verzeichnisinformationen bereit.

Eine typische Domänenidentität wird so dargestellt:

```text
FIRMA\anna
anna@firma.example
```

### Organisationseinheiten

**Organisationseinheiten (OUs)** strukturieren Objekte innerhalb einer Domäne. Sie dienen insbesondere:

- der übersichtlichen Organisation,
- der Delegation von Verwaltungsaufgaben,
- der gezielten Zuweisung von Gruppenrichtlinien.

Eine OU ist keine Sicherheitsgruppe und vergibt allein keine Zugriffsrechte.

### Domänengruppen

In Active Directory werden Sicherheitsgruppen typischerweise nach ihrem Gültigkeitsbereich unterschieden:

| Gruppenbereich | Grundidee |
|---|---|
| Global | Konten mit gemeinsamer Funktion zusammenfassen |
| Domänenlokal | Berechtigungen auf Ressourcen einer Domäne erhalten |
| Universal | Mitgliedschaften über Domänengrenzen eines Forests hinweg bündeln |

Ein bewährtes Grundmodell ist **AGDLP**:

```text
Accounts → Global Groups → Domain Local Groups → Permissions
```

Beispiel:

```text
Anna
→ GG_Buchhaltung
→ DL_Rechnungen_Aendern
→ NTFS-Berechtigung „Ändern“
```

Die globale Gruppe beschreibt die Funktion oder Zugehörigkeit. Die domänenlokale Gruppe beschreibt den Zugriff auf eine konkrete Ressource.

## 15. Grundlagen von Gruppenrichtlinien

**Gruppenrichtlinien** konfigurieren Benutzer und Computer in einer Active-Directory-Umgebung zentral. Die Einstellungen werden in **Group Policy Objects (GPOs)** gespeichert.

Typische Anwendungsfälle sind:

- Kennwort- und Kontosperrrichtlinien,
- Zuweisung von Benutzerrechten,
- Windows-Firewall-Einstellungen,
- Sicherheitsoptionen,
- Softwarekonfiguration,
- Desktopvorgaben,
- Anmelde- und Startskripte.

### Benutzer- und Computerkonfiguration

Ein GPO besitzt zwei Hauptbereiche:

- **Computerkonfiguration:** Einstellungen für Computer,
- **Benutzerkonfiguration:** Einstellungen für Benutzer.

### Verknüpfung und Reihenfolge

GPOs können mit Sites, Domänen und OUs verknüpft werden. Die grundlegende Reihenfolge wird als **LSDOU** zusammengefasst:

```text
Local → Site → Domain → Organizational Unit
```

Spätere Einstellungen können frühere überschreiben. Vererbung, Priorität, Sicherheitsfilter und weitere Mechanismen beeinflussen das Endergebnis.

### Berechtigungen und Gruppenrichtlinien unterscheiden

- **NTFS-Berechtigungen** regeln den Zugriff auf konkrete Dateien und Ordner.
- **Benutzerrechte** erlauben bestimmte systemweite Aktionen.
- **Gruppenrichtlinien** verteilen und erzwingen Konfigurationen; sie können unter anderem Benutzerrechte und Sicherheitseinstellungen festlegen.

## 16. Planung einer übersichtlichen Rechtevergabe

Eine wartbare Rechteverwaltung folgt einigen Grundprinzipien:

1. Berechtigungen möglichst an Gruppen statt direkt an Benutzer vergeben.
2. Gruppen nach Funktion und Ressourcenrolle eindeutig benennen.
3. Vererbung nutzen und Sonderfälle begrenzen.
4. Verweigern-Einträge nur gezielt einsetzen.
5. Alltags- und Administratorkonten trennen.
6. Änderungen dokumentieren und mit einem passenden Testkonto prüfen.
7. Nicht mehr benötigte Konten deaktivieren und kontrolliert entfernen.
8. Gruppenmitgliedschaften regelmäßig überprüfen.
9. Besitzübernahmen nur bei begründetem Bedarf durchführen.
10. Rechte nach dem Prinzip der minimalen Berechtigung vergeben.

Ein mögliches Namensschema lautet:

```text
GG_<Abteilung oder Rolle>
DL_<Ressource>_<Berechtigungsstufe>
```

Beispiele:

```text
GG_Vertrieb
DL_Angebote_Lesen
DL_Angebote_Aendern
```

## 17. Erste Troubleshooting-Methoden

Bei einem Zugriffs- oder Anmeldeproblem sollte zunächst das genaue Fehlerbild ermittelt werden.

### Leitfragen

1. Welches Konto wird tatsächlich verwendet?
2. Handelt es sich um ein lokales, Microsoft-, Domänen- oder Entra-ID-Konto?
3. Funktioniert die Anmeldung oder scheitert erst der Zugriff auf eine Ressource?
4. Betrifft der Fehler nur einen Benutzer oder mehrere?
5. Betrifft er nur einen Computer oder mehrere?
6. Ist das Konto aktiv, gesperrt oder abgelaufen?
7. Welche direkten und indirekten Gruppenmitgliedschaften bestehen?
8. Welche Berechtigungen sind explizit und welche geerbt?
9. Gibt es einen Verweigern-Eintrag?
10. Erfolgt der Zugriff lokal oder über eine Netzwerkfreigabe?
11. Wurde kürzlich ein Konto, eine Gruppe, GPO oder ACL geändert?
12. Gibt es passende Meldungen in der Ereignisanzeige?

### Nützliche Befehle

```powershell
whoami
whoami /user
whoami /groups
whoami /priv
net user
net localgroup
icacls C:\Pfad
gpresult /r
```

In PowerShell stehen außerdem beispielsweise diese Cmdlets zur Verfügung:

```powershell
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember -Group "Administratoren"
Get-Acl C:\Pfad
```

### Typische Ursachen

- falscher Kontokontext,
- fehlende oder noch nicht wirksame Gruppenmitgliedschaft,
- unterbrochene Vererbung,
- unerwarteter Verweigern-Eintrag,
- restriktivere Freigabeberechtigung,
- fehlende Erhöhung durch UAC,
- beschädigtes oder temporäres Benutzerprofil,
- nicht erreichbarer Domain Controller,
- fehlerhafte DNS- oder Zeiteinstellungen,
- noch nicht aktualisierte Gruppenrichtlinie.

Nach einer Änderung an Gruppenmitgliedschaften kann eine erneute Anmeldung notwendig sein, damit Windows ein neues Zugriffstoken erstellt.

## 18. Typische Missverständnisse

### „Ein Administrator darf automatisch jede Datei öffnen.“

Nicht zwingend. Auch Administratoren unterliegen ACLs. Sie können jedoch häufig den Besitz übernehmen oder Berechtigungen ändern.

### „UAC macht aus einem Standardbenutzer einen Administrator.“

Nein. Ein Standardbenutzer muss administrative Anmeldedaten bereitstellen. Die erhöhte Aktion läuft dann im Kontext des administrativen Kontos.

### „Ein neuer Benutzer mit demselben Namen ist dasselbe Konto.“

Nein. Das neue Konto erhält eine neue SID.

### „Eine OU vergibt Berechtigungen.“

Nein. Eine OU organisiert Objekte und ermöglicht Delegation sowie GPO-Zuweisung. Ressourcenberechtigungen werden typischerweise über Sicherheitsgruppen vergeben.

### „Freigabeberechtigungen ersetzen NTFS-Berechtigungen.“

Nein. Bei Netzwerkzugriffen werden beide Ebenen berücksichtigt.

### „UAC ist nur eine störende Bestätigung.“

Nein. UAC trennt normale von erhöhten Prozessen und begrenzt die unbemerkte Verwendung administrativer Rechte.

### „Besitz bedeutet automatisch Vollzugriff.“

Nicht unmittelbar. Besitz ermöglicht grundsätzlich die Verwaltung der Berechtigungen; die konkreten Zugriffe werden weiterhin durch die ACL bestimmt.

## 19. Zusammenfassung

- Konten repräsentieren Identitäten; Gruppen vereinfachen die Rechtevergabe.
- Lokale Konten gelten für einen Computer, Domänenkonten werden zentral verwaltet.
- Ein Benutzerprofil enthält die persönliche Arbeitsumgebung, ist aber nicht das Benutzerkonto selbst.
- Windows verwendet SIDs und Zugriffstoken für Sicherheitsentscheidungen.
- NTFS-Berechtigungen werden in ACLs gespeichert und können vererbt werden.
- Effektive Berechtigungen entstehen aus direkten, geerbten und gruppenbasierten Einträgen.
- Bei Netzwerkfreigaben wirken Freigabe- und NTFS-Berechtigungen gemeinsam.
- Der Besitzer eines Objekts kann dessen Berechtigungen grundsätzlich verwalten.
- UAC fordert administrative Erhöhung bewusst an, ersetzt aber kein sicheres Kontenkonzept.
- Active Directory zentralisiert Identitäten; Gruppenrichtlinien zentralisieren Konfigurationen.
- Berechtigungen sollten möglichst über klar benannte Gruppen und nach dem Prinzip der minimalen Rechte vergeben werden.

## 20. Kontrollfragen

1. Was unterscheidet Identifikation, Authentifizierung und Autorisierung?
2. Worin unterscheiden sich ein lokales Konto und ein Domänenkonto?
3. Welche Informationen enthält ein Zugriffstoken?
4. Warum ist die SID wichtiger als der sichtbare Benutzername?
5. Was unterscheidet ein Benutzerkonto von einem Benutzerprofil?
6. Welche Aufgabe besitzt die Datei `NTUSER.DAT`?
7. Was ist der Unterschied zwischen einer Berechtigung und einem Benutzerrecht?
8. Was enthält eine DACL?
9. Wie wirken Gruppenmitgliedschaften auf erlaubte Berechtigungen?
10. Warum sollten Verweigern-Einträge sparsam eingesetzt werden?
11. Was geschieht beim Deaktivieren der Vererbung?
12. Wie wirken Freigabe- und NTFS-Berechtigungen zusammen?
13. Was bedeutet Besitz bei einer Datei?
14. Wie arbeitet UAC bei einem Administrator und bei einem Standardbenutzer?
15. Was beschreibt das Modell AGDLP?
16. Welche Unterschiede bestehen zwischen einer OU und einer Sicherheitsgruppe?
17. Wofür werden Gruppenrichtlinien verwendet?
18. Warum kann nach einer geänderten Gruppenmitgliedschaft eine erneute Anmeldung erforderlich sein?

## 21. Begriffsübersicht

| Begriff | Kurzbeschreibung |
|---|---|
| Benutzerkonto | Identität eines Benutzers oder technischen Zwecks |
| Gruppe | Zusammenfassung von Sicherheitsprinzipalen |
| SID | Eindeutige Windows-Sicherheitskennung |
| SAM | Lokale Datenbank für Konten und Gruppen |
| Zugriffstoken | Sicherheitsinformationen einer angemeldeten Identität |
| Benutzerprofil | Persönliche Daten und Einstellungen eines Benutzers |
| NTFS-Berechtigung | Zugriffserlaubnis für eine Datei oder einen Ordner |
| ACL | Liste von Zugriffs- oder Überwachungseinträgen |
| DACL | ACL, die erlaubte und verweigerte Zugriffe festlegt |
| ACE | Einzelner Eintrag innerhalb einer ACL |
| Vererbung | Übernahme von Berechtigungen übergeordneter Ordner |
| Effektive Berechtigung | Tatsächlich wirksame Berechtigung |
| Besitzer | Identität mit Kontrolle über die Berechtigungsverwaltung eines Objekts |
| UAC | Kontrollierte Erhöhung für administrative Aktionen |
| AD DS | Zentraler Verzeichnisdienst für Windows-Domänen |
| OU | Organisationseinheit für Struktur, Delegation und GPOs |
| GPO | Sammlung zentraler Benutzer- und Computereinstellungen |
| AGDLP | Modell für gruppenbasierte Ressourcenberechtigungen |
