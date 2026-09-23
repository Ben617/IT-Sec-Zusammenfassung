# Windows-Sicherheit 

## Lernziele

Nach diesem Kapitel kannst du:

- die wichtigsten Schutzziele der IT-Sicherheit erklÃ¤ren,
- das mehrschichtige Sicherheitsmodell von Windows beschreiben,
- zentrale Windows-Sicherheitsfunktionen einordnen,
- Authentifizierung, Autorisierung und Ãœberwachung unterscheiden,
- Microsoft Defender Antivirus und die Windows-Firewall erklÃ¤ren,
- BitLocker, EFS, TPM und Secure Boot voneinander abgrenzen,
- die Bedeutung von Updates, Backups und Protokollierung erlÃ¤utern,
- erste Sicherheitsprobleme strukturiert untersuchen.

## 1. Grundbegriffe der IT-Sicherheit

### Schutzziele

Die drei klassischen Schutzziele werden hÃ¤ufig als **CIA-Triade** zusammengefasst:

| Schutzziel | Bedeutung | Beispiel |
|---|---|---|
| Vertraulichkeit | Informationen sind nur fÃ¼r Berechtigte zugÃ¤nglich | NTFS-Berechtigungen, BitLocker |
| IntegritÃ¤t | Daten und Systeme werden nicht unbemerkt verÃ¤ndert | Signaturen, Secure Boot |
| VerfÃ¼gbarkeit | Systeme und Daten stehen bei Bedarf zur VerfÃ¼gung | Backups, Redundanz, Wiederherstellung |

Weitere wichtige Ziele sind:

- **AuthentizitÃ¤t:** Eine IdentitÃ¤t oder Nachricht ist echt.
- **Nachvollziehbarkeit:** Aktionen kÃ¶nnen protokolliert und zugeordnet werden.
- **Verbindlichkeit:** Eine ausgefÃ¼hrte Handlung kann nicht glaubhaft abgestritten werden.

SicherheitsmaÃŸnahmen kÃ¶nnen miteinander in Konflikt stehen. Sehr strenge Kontrollen kÃ¶nnen beispielsweise die Bedienbarkeit oder VerfÃ¼gbarkeit beeintrÃ¤chtigen. Sicherheit ist daher eine fortlaufende Risikobetrachtung und kein einmal erreichter Endzustand.

### Bedrohung, Schwachstelle und Risiko

- Eine **Bedrohung** ist etwas, das Schaden verursachen kann, etwa Schadsoftware oder ein Angreifer.
- Eine **Schwachstelle** ist eine ausnutzbare SchwÃ¤che, etwa ungepatchte Software.
- Ein **Risiko** ergibt sich aus der Wahrscheinlichkeit eines Schadens und dessen mÃ¶glichen Auswirkungen.
- Ein **Exploit** ist eine Methode oder Software, mit der eine Schwachstelle ausgenutzt wird.

### Angriff, Sicherheitsereignis und Sicherheitsvorfall

- Ein **Angriff** ist ein gezielter Versuch, eine SchutzmaÃŸnahme zu umgehen oder Schaden zu verursachen.
- Ein **Sicherheitsereignis** ist eine beobachtbare AktivitÃ¤t mit mÃ¶glicher Sicherheitsrelevanz.
- Ein **Sicherheitsvorfall** ist ein Ereignis, das die Sicherheit tatsÃ¤chlich oder wahrscheinlich beeintrÃ¤chtigt und behandelt werden muss.

Nicht jede Warnmeldung ist ein bestÃ¤tigter Vorfall. Umgekehrt kann ein Vorfall aus mehreren zunÃ¤chst unauffÃ¤lligen Ereignissen bestehen.

## 2. Mehrschichtige Sicherheit

Windows-Sicherheit beruht auf mehreren Schutzschichten. Dieses Prinzip heiÃŸt **Defense in Depth**.

Typische Schichten sind:

1. physischer Schutz des GerÃ¤ts,
2. sichere Firmware und vertrauenswÃ¼rdiger Startvorgang,
3. aktuelles Betriebssystem und aktuelle Anwendungen,
4. IdentitÃ¤ts- und Anmeldeschutz,
5. minimale Benutzer- und Prozessrechte,
6. Schutz vor Schadsoftware,
7. Netzwerkfilterung,
8. Daten- und LaufwerksverschlÃ¼sselung,
9. Protokollierung und Ãœberwachung,
10. Datensicherung und Wiederherstellung.

Eine einzelne Funktion kann nicht alle Bedrohungen abwehren. Beispielsweise schÃ¼tzt BitLocker ein ausgeschaltetes, gestohlenes GerÃ¤t, verhindert aber nicht automatisch, dass ein angemeldeter Benutzer eine schÃ¤dliche Datei Ã¶ffnet.

## 3. Windows-Sicherheitsarchitektur

### Kernelmodus und Benutzermodus

Windows trennt grundsÃ¤tzlich zwei AusfÃ¼hrungsbereiche:

- **Benutzermodus:** Anwendungen laufen mit eingeschrÃ¤nktem Zugriff auf das System.
- **Kernelmodus:** Betriebssystemkern und viele Treiber besitzen weitreichenden Hardware- und Speicherzugriff.

Ein Fehler oder Angriff im Kernelmodus kann besonders schwerwiegend sein. Deshalb sind Treibersignaturen, Speicherisolierung und ein sicherer Startvorgang wichtig.

### Sicherheitsprinzipale

Ein **Sicherheitsprinzipal** ist eine IdentitÃ¤t, der Berechtigungen zugewiesen werden kÃ¶nnen. Dazu gehÃ¶ren:

- Benutzer,
- Gruppen,
- Computer,
- Dienstkonten,
- integrierte IdentitÃ¤ten wie `SYSTEM`.

Windows identifiziert Sicherheitsprinzipale intern durch **Security Identifiers (SIDs)**.

### Zugriffstoken

Nach erfolgreicher Anmeldung erzeugt Windows ein Zugriffstoken. Es enthÃ¤lt unter anderem:

- die SID des Benutzers,
- die SIDs seiner Gruppen,
- Benutzerrechte und Privilegien,
- die IntegritÃ¤tsstufe,
- Informationen zur UAC-ErhÃ¶hung.

Prozesse laufen mit einem solchen Token. Beim Zugriff auf ein geschÃ¼tztes Objekt prÃ¼ft Windows, ob das Token die erforderlichen Berechtigungen besitzt.

### Sicherheitsdeskriptoren und ACLs

Ein geschÃ¼tztes Objekt besitzt einen **Sicherheitsdeskriptor**. Dieser enthÃ¤lt insbesondere:

- den Besitzer,
- eine **DACL** fÃ¼r erlaubte und verweigerte Zugriffe,
- gegebenenfalls eine **SACL** fÃ¼r die Ãœberwachung bestimmter Zugriffsversuche.

Die DACL steuert also den Zugriff; die SACL steuert, welche Zugriffsversuche bei aktivierter Ãœberwachungsrichtlinie protokolliert werden.

## 4. Identifikation, Authentifizierung und Autorisierung

Diese Begriffe beschreiben unterschiedliche Schritte:

- **Identifikation:** Ein Benutzer gibt an, welche IdentitÃ¤t er besitzt.
- **Authentifizierung:** Windows prÃ¼ft den IdentitÃ¤tsnachweis.
- **Autorisierung:** Windows entscheidet, welche Aktionen erlaubt sind.
- **Ãœberwachung:** Sicherheitsrelevante Aktionen werden protokolliert.

### Authentifizierungsfaktoren

Faktoren werden hÃ¤ufig in drei Kategorien eingeteilt:

- **Wissen:** Kennwort oder PIN,
- **Besitz:** Smartcard, Smartphone oder SicherheitsschlÃ¼ssel,
- **InhÃ¤renz:** Fingerabdruck oder Gesichtserkennung.

**Mehrfaktorauthentifizierung (MFA)** kombiniert Faktoren aus unterschiedlichen Kategorien. Zwei KennwÃ¶rter wÃ¤ren daher keine echte Zwei-Faktor-Authentifizierung.

### Windows Hello

Windows Hello ermÃ¶glicht die Anmeldung mit PIN oder biometrischen Merkmalen. Die PIN ist an das GerÃ¤t gebunden und nicht lediglich ein kurzes Ersatzkennwort, das unverÃ¤ndert an einen Server Ã¼bertragen wird.

Windows Hello for Business verwendet in Organisationen schlÃ¼ssel- oder zertifikatsbasierte Verfahren und kann die tÃ¤gliche Kennwortanmeldung ersetzen.

### Kerberos und NTLM

In klassischen Windows-DomÃ¤nen ist **Kerberos** das bevorzugte Authentifizierungsprotokoll. Es arbeitet mit zeitlich begrenzten Tickets.

**NTLM** ist ein Ã¤lteres Challenge-Response-Verfahren. Es wird aus KompatibilitÃ¤tsgrÃ¼nden noch verwendet, sollte aber nach MÃ¶glichkeit eingeschrÃ¤nkt werden.

### Kontosperrung

Kontosperrrichtlinien kÃ¶nnen ein Konto nach mehreren fehlerhaften Anmeldeversuchen vorÃ¼bergehend sperren. Das erschwert einfache Kennwortangriffe, kann aber bei ungeeigneter Konfiguration auch zur absichtlichen Sperrung fremder Konten missbraucht werden.

## 5. Konten, Rechte und geringste Berechtigung

### Standard- und Administratorkonten

Standardbenutzer besitzen die fÃ¼r alltÃ¤gliche Aufgaben benÃ¶tigten Rechte. Administratoren kÃ¶nnen systemweite Ã„nderungen durchfÃ¼hren.

Nach dem Prinzip der **minimalen Rechte** sollen Benutzer und Prozesse nur die Rechte erhalten, die ihre Aufgabe erfordert. FÃ¼r Administration und Alltagsarbeit sollten nach MÃ¶glichkeit getrennte Konten verwendet werden.

### Benutzerkontensteuerung

Die **Benutzerkontensteuerung (UAC)** sorgt dafÃ¼r, dass administrative Aktionen bewusst erhÃ¶ht gestartet werden. Mitglieder der Administratorengruppe arbeiten im Alltag normalerweise mit einem eingeschrÃ¤nkten Token.

UAC reduziert unbeabsichtigte Ã„nderungen, ist aber keine vollstÃ¤ndige Sicherheitsgrenze gegen Schadsoftware im selben Benutzerkontext. Sie ersetzt weder Standardbenutzerkonten noch Schadsoftwareschutz.

### Dienstkonten

Dienste sollten nicht mit mehr Rechten als nÃ¶tig ausgefÃ¼hrt werden. Windows stellt dafÃ¼r integrierte Konten und verwaltete Dienstkonten bereit.

Typische integrierte IdentitÃ¤ten sind:

| Konto | Einordnung |
|---|---|
| LocalSystem | Sehr weitreichende lokale Rechte |
| LocalService | Stark eingeschrÃ¤nkte lokale Rechte |
| NetworkService | EingeschrÃ¤nkte lokale Rechte mit NetzwerkidentitÃ¤t des Computers |

Die Auswahl eines zu mÃ¤chtigen Dienstkontos vergrÃ¶ÃŸert den mÃ¶glichen Schaden bei einer kompromittierten Anwendung.

### Least Privilege und Zero Trust

**Least Privilege** begrenzt Rechte auf das notwendige Minimum. **Zero Trust** erweitert diesen Gedanken: Kein Zugriff wird allein wegen Standort oder NetzwerkzugehÃ¶rigkeit automatisch als vertrauenswÃ¼rdig behandelt. IdentitÃ¤t, GerÃ¤tezustand, Kontext und Risiko werden fortlaufend berÃ¼cksichtigt.

## 6. Windows-Sicherheit-App

Die App **Windows-Sicherheit** bÃ¼ndelt Statusanzeigen und Einstellungen verschiedener Schutzfunktionen. Typische Bereiche sind:

- Viren- und Bedrohungsschutz,
- Kontoschutz,
- Firewall- und Netzwerkschutz,
- App- und Browsersteuerung,
- GerÃ¤tesicherheit,
- GerÃ¤teleistung und -integritÃ¤t,
- Familienoptionen.

Die App ist hauptsÃ¤chlich eine Bedien- und StatusoberflÃ¤che. Die dahinterstehenden Schutzkomponenten sind eigenstÃ¤ndige Windows-Dienste und Technologien.

In verwalteten Organisationen kÃ¶nnen Einstellungen zentral festgelegt sein. Eine lokal ausgegraute Option ist daher nicht automatisch ein technischer Fehler.

## 7. Microsoft Defender Antivirus

**Microsoft Defender Antivirus** ist die integrierte Antischadsoftware-Komponente von Windows.

### Schutzmechanismen

Defender verwendet mehrere Erkennungsmethoden:

- bekannte Signaturen,
- heuristische Analyse,
- VerhaltensÃ¼berwachung,
- cloudgestÃ¼tzten Schutz,
- maschinelle Erkennungsmodelle,
- ÃœberprÃ¼fung heruntergeladener oder ausgefÃ¼hrter Dateien.

### Echtzeitschutz

Der Echtzeitschutz Ã¼berwacht Dateien und Prozesse beim Zugriff. Er kann verdÃ¤chtige Inhalte blockieren, bevor sie ausgefÃ¼hrt oder weiterverarbeitet werden.

### Scantypen

- **SchnellÃ¼berprÃ¼fung:** untersucht typische Angriffs- und Autostartbereiche.
- **VollstÃ¤ndige ÃœberprÃ¼fung:** untersucht alle erreichbaren Dateien und Programme.
- **Benutzerdefinierte ÃœberprÃ¼fung:** untersucht ausgewÃ¤hlte Orte.
- **Microsoft Defender Offline:** startet eine ÃœberprÃ¼fung auÃŸerhalb der normalen Windows-Sitzung und kann dadurch bestimmte hartnÃ¤ckige Bedrohungen besser erkennen.

### Sicherheitsinformationen

Erkennungssignaturen und weitere Schutzinformationen mÃ¼ssen regelmÃ¤ÃŸig aktualisiert werden. Ein installierter Virenschutz mit veralteten Informationen bietet einen deutlich geringeren Schutz.

### QuarantÃ¤ne

Erkannte Dateien kÃ¶nnen in die QuarantÃ¤ne verschoben werden. Dort sind sie isoliert und kÃ¶nnen nicht normal ausgefÃ¼hrt werden. Vor einer Wiederherstellung muss sicher geklÃ¤rt sein, dass es sich um eine Falscherkennung handelt.

### AusschlÃ¼sse

AusschlÃ¼sse verhindern die Untersuchung bestimmter Dateien, Ordner, Prozesse oder Dateitypen. Sie kÃ¶nnen Leistungs- oder KompatibilitÃ¤tsprobleme lÃ¶sen, erzeugen aber SchutzlÃ¼cken. AusschlÃ¼sse sollten daher eng begrenzt, begrÃ¼ndet und regelmÃ¤ÃŸig Ã¼berprÃ¼ft werden.

### Manipulationsschutz

Der **Manipulationsschutz** erschwert unautorisierte Ã„nderungen an wichtigen Defender-Einstellungen. Dadurch soll Schadsoftware den Schutz nicht einfach deaktivieren kÃ¶nnen.

### Kontrollierter Ordnerzugriff

Der **kontrollierte Ordnerzugriff** schÃ¼tzt ausgewÃ¤hlte Ordner vor unerlaubten Ã„nderungen durch nicht vertrauenswÃ¼rdige Anwendungen. Er kann insbesondere die Auswirkungen bestimmter Ransomware-Angriffe begrenzen.

## 8. Reputations- und Anwendungsschutz

### Microsoft Defender SmartScreen

SmartScreen bewertet unter anderem Webseiten, Downloads und Anwendungen anhand von Reputation und bekannten Bedrohungen. Eine geringe Reputation bedeutet nicht automatisch Schadsoftware, erhÃ¶ht aber das Risiko und erfordert eine bewusste PrÃ¼fung.

### Potenziell unerwÃ¼nschte Anwendungen

**Potentially Unwanted Applications (PUA)** sind nicht immer klassische Schadsoftware, kÃ¶nnen aber unerwÃ¼nschte Werbung, BÃ¼ndelsoftware oder irrefÃ¼hrendes Verhalten mitbringen. Windows kann ihre Installation und AusfÃ¼hrung blockieren.

### Anwendungssteuerung

Anwendungssteuerung legt fest, welche Programme, Skripte oder Bibliotheken ausgefÃ¼hrt werden dÃ¼rfen. Je nach Windows-Edition und Umgebung kommen unterschiedliche Technologien infrage, beispielsweise AppLocker oder App Control for Business.

**Allowlisting** erlaubt nur ausdrÃ¼cklich freigegebene Software. Es bietet eine starke Kontrolle, erfordert aber sorgfÃ¤ltige Planung und Pflege.

### Exploit-Schutz

Windows enthÃ¤lt SchutzmaÃŸnahmen gegen verbreitete Ausnutzungstechniken. Dazu gehÃ¶ren beispielsweise Speicher- und Ablaufkontrollen. Solche MaÃŸnahmen kÃ¶nnen bekannte Angriffsmuster erschweren, ersetzen aber keine Updates.

## 9. Windows Defender Firewall

Die **Windows Defender Firewall** filtert ein- und ausgehenden Netzwerkverkehr anhand von Regeln.

### Zustandsbehaftete Filterung

Die Firewall arbeitet zustandsbehaftet. Sie kann Antworten auf erlaubte ausgehende Verbindungen erkennen, ohne dafÃ¼r jede RÃ¼ckrichtung einzeln freizugeben.

### Netzwerkprofile

Windows unterscheidet drei grundlegende Firewallprofile:

| Profil | Typischer Einsatz |
|---|---|
| DomÃ¤ne | Netzwerk mit erkannter AD-DomÃ¤ne |
| Privat | VertrauenswÃ¼rdiges privates Netzwerk |
| Ã–ffentlich | Nicht vertrauenswÃ¼rdiges Netzwerk, etwa Ã¶ffentliches WLAN |

Das Ã¶ffentliche Profil sollte restriktiver behandelt werden. Ein Netzwerk sollte nur dann als privat eingestuft werden, wenn es tatsÃ¤chlich vertrauenswÃ¼rdig ist.

### Eingehende und ausgehende Regeln

- **Eingehende Regeln** steuern Verbindungen zum Computer.
- **Ausgehende Regeln** steuern Verbindungen vom Computer zu anderen Zielen.

Regeln kÃ¶nnen unter anderem einschrÃ¤nken nach:

- Programm oder Dienst,
- Protokoll,
- lokalem oder entferntem Port,
- IP-Adresse,
- Netzwerkprofil,
- authentifizierter Verbindung.

### Firewall nicht pauschal deaktivieren

Bei Netzwerkproblemen sollte die Firewall nicht dauerhaft vollstÃ¤ndig deaktiviert werden. Besser ist es, das betroffene Profil, die konkrete Regel und die benÃ¶tigten Ports zu analysieren. Eine gezielte Regel ist nachvollziehbarer und sicherer als eine breite Freigabe.

## 10. Netzwerk- und Protokollsicherheit

### Sichere Protokolle

Wo mÃ¶glich, sollten verschlÃ¼sselte und moderne Protokolle verwendet werden. Beispiele:

- HTTPS statt HTTP,
- SSH statt unverschlÃ¼sselter Fernzugriffe,
- aktuelle SMB-Versionen statt SMBv1,
- moderne TLS-Versionen statt veralteter SSL-/TLS-Verfahren.

### SMB

Das **Server Message Block (SMB)**-Protokoll stellt unter Windows Datei- und Druckfreigaben bereit. SMBv1 ist veraltet und sollte nicht verwendet werden. Moderne SMB-Versionen unterstÃ¼tzen unter anderem Signierung und VerschlÃ¼sselung.

### Remote Desktop

Remote Desktop sollte nur bei Bedarf aktiviert und geschÃ¼tzt werden, beispielsweise durch:

- beschrÃ¤nkte Benutzergruppen,
- Network Level Authentication,
- Firewallregeln,
- sichere Zugangswege wie VPN oder kontrollierte Gateways,
- starke Authentifizierung,
- aktuelle Systeme.

Ein direkt aus dem Internet erreichbarer RDP-Dienst stellt ein erhebliches Risiko dar.

## 11. Laufwerks- und DateiverschlÃ¼sselung

### BitLocker

**BitLocker** verschlÃ¼sselt vollstÃ¤ndige Volumes. Es schÃ¼tzt insbesondere Daten auf verlorenen, gestohlenen oder ausgebauten DatentrÃ¤gern.

BitLocker kann den **Trusted Platform Module (TPM)** verwenden, um SchlÃ¼sselmaterial zu schÃ¼tzen und den Systemzustand beim Start zu prÃ¼fen.

Wichtige Begriffe:

- **WiederherstellungsschlÃ¼ssel:** ermÃ¶glicht den Zugriff, wenn die normale Entsperrung fehlschlÃ¤gt.
- **TPM-Schutz:** bindet die Entsperrung an die Plattform und gemessene StartzustÃ¤nde.
- **TPM mit PIN:** ergÃ¤nzt die GerÃ¤tebindung um einen Wissensfaktor beim Start.

Der WiederherstellungsschlÃ¼ssel muss sicher und getrennt vom GerÃ¤t aufbewahrt werden. Ohne geeigneten SchlÃ¼ssel kann eine erfolgreiche Datenwiederherstellung unmÃ¶glich sein.

### GerÃ¤teverschlÃ¼sselung

Auf unterstÃ¼tzten GerÃ¤ten kann Windows eine vereinfachte Form der GerÃ¤teverschlÃ¼sselung anbieten. Die VerfÃ¼gbarkeit und Verwaltung hÃ¤ngen von Hardware, Edition, Konto und Organisationsrichtlinien ab.

### Encrypting File System

**EFS** verschlÃ¼sselt einzelne Dateien und Ordner auf NTFS-Volumes. Die VerschlÃ¼sselung ist an ein Benutzerzertifikat gebunden.

| BitLocker | EFS |
|---|---|
| VerschlÃ¼sselt ein Volume | VerschlÃ¼sselt einzelne Dateien oder Ordner |
| SchÃ¼tzt besonders bei ausgeschaltetem GerÃ¤t | SchÃ¼tzt Daten vor anderen Konten auf dem laufenden System |
| Entsperrung auf Volumeebene | EntschlÃ¼sselung im Benutzerkontext |
| WiederherstellungsschlÃ¼ssel erforderlich | Zertifikat und privater SchlÃ¼ssel entscheidend |

BitLocker und EFS erfÃ¼llen unterschiedliche Zwecke und kÃ¶nnen sich ergÃ¤nzen. EFS darf nicht ohne geplante Zertifikats- und SchlÃ¼sselwiederherstellung eingesetzt werden.

### VerschlÃ¼sselung schÃ¼tzt nicht vor allem

Ist ein Benutzer angemeldet und das Volume entsperrt, kÃ¶nnen Programme mit ausreichenden Rechten auf entschlÃ¼sselte Daten zugreifen. VerschlÃ¼sselung ersetzt daher weder Berechtigungen noch Schadsoftwareschutz oder Backups.

## 12. TPM, Secure Boot und vertrauenswÃ¼rdiger Start

### Trusted Platform Module

Das **TPM** ist eine Sicherheitskomponente, die kryptografische SchlÃ¼ssel schÃ¼tzen und den Startzustand eines Systems messen kann. Es wird unter anderem von BitLocker und Windows Hello verwendet.

### UEFI Secure Boot

**Secure Boot** Ã¼berprÃ¼ft digitale Signaturen wichtiger Startkomponenten. Dadurch soll verhindert werden, dass nicht vertrauenswÃ¼rdiger Code bereits vor Windows geladen wird.

Secure Boot ist eine UEFI-Funktion und nicht dasselbe wie BitLocker:

- Secure Boot schÃ¼tzt die Vertrauenskette des Startvorgangs.
- BitLocker schÃ¼tzt gespeicherte Daten durch VerschlÃ¼sselung.
- Das TPM kann SchlÃ¼ssel schÃ¼tzen und Messwerte des Startvorgangs speichern.

### Trusted Boot und Early Launch Antimalware

Windows Ã¼berprÃ¼ft beim vertrauenswÃ¼rdigen Start weitere Komponenten. **Early Launch Antimalware (ELAM)** ermÃ¶glicht eine frÃ¼he Bewertung bestimmter Starttreiber, bevor viele andere Komponenten geladen werden.

### Virtualization-Based Security

**Virtualization-Based Security (VBS)** nutzt Hardwarevirtualisierung, um sensible Sicherheitsfunktionen in einer isolierten Umgebung auszufÃ¼hren.

Beispiele sind:

- **Memory Integrity / Hypervisor-Protected Code Integrity (HVCI)** zur Kontrolle von Kernelcode,
- **Credential Guard** zum Schutz bestimmter Anmeldeinformationen.

Die VerfÃ¼gbarkeit hÃ¤ngt von Hardware, Edition und Konfiguration ab. ZusÃ¤tzliche Schutzfunktionen kÃ¶nnen KompatibilitÃ¤ts- oder Leistungsanforderungen mitbringen.

## 13. Updates und Patchmanagement

Sicherheitsupdates schlieÃŸen bekannte Schwachstellen. Ein aktueller Virenschutz kann eine ungepatchte Systemkomponente nicht vollstÃ¤ndig kompensieren.

### Updatearten

Windows unterscheidet unter anderem:

- monatliche kumulative QualitÃ¤ts- und Sicherheitsupdates,
- Funktionsupdates,
- .NET-Updates,
- Defender-Sicherheitsinformationen,
- Treiber- und Firmwareupdates,
- optionale Vorschauupdates.

### Patchmanagement als Prozess

Professionelles Patchmanagement umfasst mehr als das Starten von Windows Update:

1. Systeme und Software inventarisieren.
2. verfÃ¼gbare Updates und Risiken bewerten,
3. kritische Updates priorisieren,
4. Updates in einer geeigneten Testgruppe prÃ¼fen,
5. gestaffelt ausrollen,
6. Installation und Neustarts Ã¼berwachen,
7. Fehler behandeln und den Stand dokumentieren.

Zu langes Aufschieben vergrÃ¶ÃŸert das Angriffsfenster. Ungetestete sofortige Verteilung an alle Systeme kann jedoch die VerfÃ¼gbarkeit gefÃ¤hrden. Der Prozess muss beide Risiken berÃ¼cksichtigen.

### Neustarts

Einige Updates werden erst nach einem Neustart vollstÃ¤ndig wirksam. Ein angezeigtes â€žUpdate installiertâ€œ bedeutet daher nicht immer, dass der Patchvorgang abgeschlossen ist.

## 14. Browser-, Makro- und Skriptsicherheit

Viele Angriffe beginnen mit manipulierten Webseiten, Dokumenten, Archiven oder Skripten.

Grundlegende SchutzmaÃŸnahmen sind:

- Browser und Erweiterungen aktuell halten,
- unbekannte Downloads nicht ungeprÃ¼ft ausfÃ¼hren,
- Makros aus nicht vertrauenswÃ¼rdigen Quellen blockieren,
- Dateinamenerweiterungen sichtbar machen,
- SkriptausfÃ¼hrung einschrÃ¤nken und protokollieren,
- E-Mail-AnhÃ¤nge und Links sorgfÃ¤ltig prÃ¼fen,
- Anwendungssteuerung einsetzen.

### PowerShell-Sicherheit

PowerShell ist ein legitimes Administrationswerkzeug, kann aber auch missbraucht werden. Eine Execution Policy ist vor allem eine SchutzmaÃŸnahme gegen unbeabsichtigte SkriptausfÃ¼hrung und keine vollstÃ¤ndige Sicherheitsgrenze.

In Organisationen sind unter anderem Skriptsignierung, Protokollierung, beschrÃ¤nkte Administration und Anwendungssteuerung wichtig.

## 15. Ereignisprotokollierung und Ãœberwachung

Windows schreibt System-, Anwendungs- und Sicherheitsereignisse in Ereignisprotokolle. Die **Ereignisanzeige** stellt diese Daten dar.

Wichtige Protokolle sind:

- Anwendung,
- Sicherheit,
- Setup,
- System,
- weiterfÃ¼hrende anwendungs- und dienstspezifische Protokolle.

### Ãœberwachungsrichtlinien

Ãœberwachungsrichtlinien legen fest, welche sicherheitsrelevanten Aktionen protokolliert werden. Beispiele:

- erfolgreiche und fehlgeschlagene Anmeldungen,
- Konto- und GruppenÃ¤nderungen,
- Zugriffe auf ausgewÃ¤hlte Objekte,
- RichtlinienÃ¤nderungen,
- Prozessstarts,
- Nutzung privilegierter Rechte.

Eine SACL allein erzeugt nicht zwingend Ereignisse; auch die entsprechende Ãœberwachungsrichtlinie muss aktiv sein.

### Aussagekraft von Protokollen

Ein einzelnes Ereignis beweist nicht immer einen Angriff. Bei der Bewertung sind Kontext, Zeit, betroffener Computer, Konto, Quelladresse und benachbarte Ereignisse wichtig.

Protokolle sollten vor Manipulation geschÃ¼tzt, zeitlich synchronisiert und ausreichend lange aufbewahrt werden. In grÃ¶ÃŸeren Umgebungen werden sie zentral gesammelt und korreliert.

## 16. Datensicherung und Wiederherstellung

Backups sind ein zentraler Bestandteil der Sicherheit. Sie schÃ¼tzen nicht nur vor HardwareausfÃ¤llen, sondern auch vor Fehlbedienung, Ransomware und beschÃ¤digten Updates.

### 3-2-1-Regel

Eine verbreitete Grundregel lautet:

- mindestens **3** Kopien der Daten,
- auf mindestens **2** unterschiedlichen Medientypen,
- davon mindestens **1** Kopie rÃ¤umlich oder logisch getrennt.

Eine zusÃ¤tzliche unverÃ¤nderbare oder offline gehaltene Kopie verbessert den Schutz gegen Ransomware.

### Backup ist nicht Synchronisierung

Bei einer Synchronisierung kÃ¶nnen LÃ¶schungen und VerschlÃ¼sselungen auf andere Speicherorte Ã¼bertragen werden. Ein Backup bewahrt definierte frÃ¼here ZustÃ¤nde und besitzt eine Aufbewahrungsstrategie.

### Wiederherstellung testen

Ein Backup ist erst dann verlÃ¤sslich, wenn seine Wiederherstellung geprÃ¼ft wurde. Dazu gehÃ¶ren:

- Lesbarkeit der Sicherung,
- vorhandene SchlÃ¼ssel und KennwÃ¶rter,
- dokumentierte Wiederherstellungsschritte,
- realistische Wiederherstellungstests.

## 17. Typische Angriffsarten

### Schadsoftware

Der Oberbegriff umfasst unter anderem Viren, WÃ¼rmer, Trojaner, Spyware und Rootkits.

### Ransomware

Ransomware verschlÃ¼sselt oder entwendet Daten und fordert hÃ¤ufig LÃ¶segeld. Schutz erfordert mehrere Ebenen: Updates, minimale Rechte, Anwendungs- und Endpunktschutz, Netzwerksegmentierung sowie getrennte Backups.

### Phishing

Phishing versucht, Benutzer zur Preisgabe von Anmeldedaten oder zur AusfÃ¼hrung schÃ¤dlicher Inhalte zu bewegen. Technische Filter helfen, ersetzen aber keine bewusste PrÃ¼fung.

### Credential Theft

Angreifer versuchen KennwÃ¶rter, Hashes, Tokens oder Tickets zu stehlen. SchutzmaÃŸnahmen sind unter anderem MFA, Credential Guard, getrennte Administratorkonten und die Begrenzung privilegierter Anmeldungen.

### Privilege Escalation

Bei einer **Rechteausweitung** versucht ein Angreifer, hÃ¶here Berechtigungen zu erlangen. Ursachen kÃ¶nnen Schwachstellen, Fehlkonfigurationen oder zu weitreichende Gruppenmitgliedschaften sein.

### Lateral Movement

Nach der Kompromittierung eines Systems versucht ein Angreifer hÃ¤ufig, weitere Systeme zu erreichen. Netzwerksegmentierung, unterschiedliche Administrationskonten und eingeschrÃ¤nkte Fernzugriffe kÃ¶nnen dies erschweren.

## 18. Grundlegende SystemhÃ¤rtung

**HÃ¤rtung** bedeutet, die AngriffsflÃ¤che eines Systems gezielt zu reduzieren.

GrundmaÃŸnahmen sind:

- Betriebssystem, Anwendungen und Firmware aktuell halten,
- unnÃ¶tige Dienste und Funktionen deaktivieren,
- nicht benÃ¶tigte Software entfernen,
- Standardbenutzer fÃ¼r Alltagsaufgaben einsetzen,
- administrative Konten trennen und schÃ¼tzen,
- sichere Anmeldeverfahren und MFA verwenden,
- Firewall aktiviert lassen,
- Defender-Schutzfunktionen aktiv halten,
- alte Protokolle wie SMBv1 deaktivieren,
- BitLocker mit gesichertem WiederherstellungsschlÃ¼ssel verwenden,
- Secure Boot und geeignete Hardware-Schutzfunktionen aktivieren,
- Makros und Skripte kontrollieren,
- Sicherheitsereignisse protokollieren,
- getestete und getrennte Backups vorhalten.

HÃ¤rtungsmaÃŸnahmen sollten dokumentiert und getestet werden. Eine unÃ¼berlegte Ã„nderung kann Anwendungen, Treiber oder Verwaltungswege beeintrÃ¤chtigen.

## 19. Sicherheitsrichtlinien und zentrale Verwaltung

### Lokale Sicherheitsrichtlinie

Die lokale Sicherheitsrichtlinie kann unter anderem festlegen:

- Kennwort- und Kontosperrregeln,
- Benutzerrechte,
- Ãœberwachungsrichtlinien,
- Sicherheitsoptionen.

### Gruppenrichtlinien

In einer Active-Directory-Umgebung verteilen Gruppenrichtlinien Sicherheitseinstellungen zentral. DomÃ¤nenrichtlinien kÃ¶nnen lokale Einstellungen Ã¼berschreiben.

### Sicherheitsbaselines

Eine **Sicherheitsbaseline** definiert einen geprÃ¼ften Ausgangszustand fÃ¼r Systeme. Sie hilft dabei, Einstellungen einheitlich und nachvollziehbar umzusetzen.

Eine Baseline muss zur Umgebung passen. Maximale EinschrÃ¤nkung ist nicht automatisch die beste Konfiguration, wenn notwendige Funktionen dadurch ausfallen.

### Mobile GerÃ¤teverwaltung

Cloudverwaltete GerÃ¤te kÃ¶nnen Sicherheitsvorgaben Ã¼ber Mobile Device Management erhalten. Dabei werden Richtlinien nicht zwingend Ã¼ber klassische Gruppenrichtlinien verteilt.

## 20. Vorgehen bei einem Sicherheitsverdacht

Bei einem mÃ¶glichen Sicherheitsvorfall sollten Beweise nicht durch unÃ¼berlegte Ã„nderungen zerstÃ¶rt werden. Das konkrete Vorgehen hÃ¤ngt von Organisation, Bedeutung des Systems und Vorfallsart ab.

Eine erste Orientierung:

1. Beobachtung, Zeitpunkt und betroffene Systeme dokumentieren.
2. Dringlichkeit und mÃ¶gliche Auswirkungen einschÃ¤tzen.
3. Bei aktivem Angriff das System nach festgelegtem Verfahren isolieren.
4. Nicht wahllos Dateien lÃ¶schen oder Protokolle bereinigen.
5. Ereignisprotokolle, Defender-Verlauf und relevante Warnungen sichern.
6. Konten, Prozesse, Netzwerkverbindungen und Autostarts untersuchen.
7. ZustÃ¤ndige Administratoren oder das Sicherheitsteam informieren.
8. Ursache beseitigen, Systeme wiederherstellen und Zugangsdaten absichern.
9. Nach dem Vorfall Kontrollen verbessern und Erkenntnisse dokumentieren.

Das Ausschalten eines Systems kann flÃ¼chtige Informationen aus dem Arbeitsspeicher vernichten. Das Weiterlaufenlassen kann dagegen weiteren Schaden ermÃ¶glichen. Diese Entscheidung sollte bei wichtigen Systemen nach dem Incident-Response-Verfahren der Organisation getroffen werden.

## 21. Erste Sicherheitsdiagnose

### Leitfragen

1. Was wurde beobachtet und wann begann es?
2. Betrifft es einen Benutzer, einen Prozess oder mehrere Systeme?
3. Welche Ã„nderung ging dem Problem voraus?
4. Sind Windows, Defender und Anwendungen aktuell?
5. Zeigt Windows-Sicherheit Warnungen oder deaktivierte Schutzfunktionen?
6. Ist das richtige Firewallprofil aktiv?
7. Gibt es unbekannte Prozesse, Dienste, Aufgaben oder AutostarteintrÃ¤ge?
8. Sind ungewÃ¶hnliche Anmeldungen oder GruppenÃ¤nderungen protokolliert?
9. Wurden Schutzfunktionen oder AusschlÃ¼sse verÃ¤ndert?
10. Existiert ein aktuelles und getestetes Backup?

### NÃ¼tzliche integrierte Werkzeuge

- Windows-Sicherheit,
- Task-Manager,
- Ressourcenmonitor,
- Ereignisanzeige,
- ZuverlÃ¤ssigkeitsverlauf,
- Windows Defender Firewall mit erweiterter Sicherheit,
- Aufgabenplanung,
- Diensteverwaltung,
- Autoruns und Process Explorer aus den Microsoft Sysinternals-Werkzeugen.

### Beispielbefehle

```powershell
whoami /all
Get-MpComputerStatus
Get-MpThreatDetection
Get-NetFirewallProfile
Get-NetFirewallRule -Enabled True
Get-BitLockerVolume
Get-Tpm
Get-WinEvent -LogName System -MaxEvents 20
```

Die Ausgabe muss im Kontext interpretiert werden. Ein laufender Prozess, eine offene Verbindung oder ein fehlgeschlagener Anmeldeversuch ist nicht automatisch schÃ¤dlich.

## 22. Typische MissverstÃ¤ndnisse

### â€žEin Virenscanner macht das System sicher.â€œ

Nein. Er ist nur eine Schutzschicht. Updates, Rechteverwaltung, Firewall, sichere Anmeldung, VerschlÃ¼sselung und Backups bleiben notwendig.

### â€žEine Firewall blockiert alle Angriffe.â€œ

Nein. Erlaubter Datenverkehr, Benutzeraktionen, Anwendungen und Angriffe innerhalb bereits zugelassener Verbindungen bleiben mÃ¶gliche Risiken.

### â€žBitLocker schÃ¼tzt vor Schadsoftware.â€œ

BitLocker schÃ¼tzt gespeicherte Daten insbesondere bei ausgeschaltetem oder gesperrtem GerÃ¤t. Auf einem entsperrten System sind die Daten fÃ¼r berechtigte Prozesse verfÃ¼gbar.

### â€žSecure Boot und TPM sind dasselbe.â€œ

Nein. Secure Boot Ã¼berprÃ¼ft Startkomponenten; das TPM schÃ¼tzt SchlÃ¼ssel und speichert Messwerte. Beide Funktionen kÃ¶nnen zusammenarbeiten.

### â€žUAC ist ein vollstÃ¤ndiger Schutz vor Schadsoftware.â€œ

Nein. UAC kontrolliert die ErhÃ¶hung administrativer Rechte, verhindert aber nicht jede schÃ¤dliche Aktion im Benutzerkontext.

### â€žWenn keine Warnung erscheint, ist das System sauber.â€œ

Nein. Schutzprogramme erkennen nicht jede Bedrohung. Fehlende Warnungen sind kein Beweis fÃ¼r ein kompromissfreies System.

### â€žSynchronisierte Daten sind automatisch ein Backup.â€œ

Nein. UnerwÃ¼nschte Ã„nderungen kÃ¶nnen mitsynchronisiert werden. Ein Backup benÃ¶tigt Versionierung, Aufbewahrung und eine getestete Wiederherstellung.

### â€žMehr Sicherheitsfunktionen sind immer besser.â€œ

Nicht ungeprÃ¼ft. SchutzmaÃŸnahmen mÃ¼ssen zur Hardware, Software und Nutzung passen. Falsch geplante Einstellungen kÃ¶nnen VerfÃ¼gbarkeit und Administration beeintrÃ¤chtigen.

## 23. Zusammenfassung

- Windows-Sicherheit besteht aus mehreren aufeinander abgestimmten Schutzschichten.
- Vertraulichkeit, IntegritÃ¤t und VerfÃ¼gbarkeit sind zentrale Schutzziele.
- Zugriffstoken, SIDs und ACLs bilden eine Grundlage des Windows-Sicherheitsmodells.
- Authentifizierung bestÃ¤tigt eine IdentitÃ¤t; Autorisierung entscheidet Ã¼ber Zugriffe.
- Standardbenutzer, UAC und minimale Rechte begrenzen mÃ¶gliche SchÃ¤den.
- Defender Antivirus, SmartScreen und Anwendungssteuerung schÃ¼tzen auf unterschiedliche Weise vor schÃ¤dlichen Inhalten.
- Die Windows-Firewall filtert Netzwerkverkehr abhÃ¤ngig von Regeln und Profilen.
- BitLocker, EFS, TPM und Secure Boot erfÃ¼llen unterschiedliche Sicherheitsaufgaben.
- Updates schlieÃŸen Schwachstellen; Backups ermÃ¶glichen die Wiederherstellung.
- Protokollierung schafft Nachvollziehbarkeit, muss aber im Kontext ausgewertet werden.
- HÃ¤rtung reduziert die AngriffsflÃ¤che und muss geplant sowie getestet werden.
- Bei einem Sicherheitsverdacht sind Dokumentation, kontrollierte Isolation und Beweiserhalt wichtig.

## 24. Kontrollfragen

1. Welche drei klassischen Schutzziele umfasst die CIA-Triade?
2. Worin unterscheiden sich Bedrohung, Schwachstelle und Risiko?
3. Was bedeutet Defense in Depth?
4. Welche Informationen enthÃ¤lt ein Zugriffstoken?
5. Worin unterscheiden sich DACL und SACL?
6. Was unterscheidet Authentifizierung von Autorisierung?
7. Warum sind zwei KennwÃ¶rter keine echte Zwei-Faktor-Authentifizierung?
8. Welche Aufgabe besitzt UAC?
9. Welche Schutzmechanismen verwendet Microsoft Defender Antivirus?
10. Welches Risiko verursachen zu weit gefasste Defender-AusschlÃ¼sse?
11. Worin unterscheiden sich die Firewallprofile DomÃ¤ne, Privat und Ã–ffentlich?
12. Warum sollte die Firewall bei einer StÃ¶rung nicht pauschal deaktiviert werden?
13. Worin unterscheiden sich BitLocker und EFS?
14. Welche Aufgaben Ã¼bernehmen TPM und Secure Boot?
15. Warum ersetzen Exploit-SchutzmaÃŸnahmen keine Updates?
16. Was unterscheidet Synchronisierung von einem Backup?
17. Was bedeutet SystemhÃ¤rtung?
18. Warum ist ein einzelnes Protokollereignis nicht automatisch ein Angriffsbeweis?
19. Welche Risiken entstehen durch zu weitreichende administrative Konten?
20. Warum kann das sofortige Ausschalten eines verdÃ¤chtigen Systems problematisch sein?

## 25. BegriffsÃ¼bersicht

| Begriff | Kurzbeschreibung |
|---|---|
| CIA-Triade | Vertraulichkeit, IntegritÃ¤t und VerfÃ¼gbarkeit |
| Defense in Depth | Schutz durch mehrere unabhÃ¤ngige Ebenen |
| Sicherheitsprinzipal | IdentitÃ¤t, der Rechte zugewiesen werden kÃ¶nnen |
| SID | Eindeutige Sicherheitskennung |
| Zugriffstoken | Sicherheitskontext eines Benutzers oder Prozesses |
| DACL | Liste erlaubter und verweigerter Zugriffe |
| SACL | Liste zu Ã¼berwachender Zugriffsversuche |
| UAC | Kontrollierte ErhÃ¶hung administrativer Prozesse |
| MFA | Authentifizierung mit mehreren Faktorarten |
| Defender Antivirus | Integrierter Schadsoftwareschutz |
| SmartScreen | Reputationsbasierter Web- und Anwendungsschutz |
| PUA | Potenziell unerwÃ¼nschte Anwendung |
| Firewall | Filterung von Netzwerkverkehr anhand von Regeln |
| BitLocker | VerschlÃ¼sselung vollstÃ¤ndiger Volumes |
| EFS | Dateibasierte NTFS-VerschlÃ¼sselung |
| TPM | HardwaregestÃ¼tzter Schutz kryptografischer Werte |
| Secure Boot | SignaturprÃ¼fung wichtiger Startkomponenten |
| VBS | Isolierung von Sicherheitsfunktionen durch Virtualisierung |
| HVCI | HypervisorgeschÃ¼tzte Kontrolle von Kernelcode |
| Patchmanagement | Geplanter Prozess zur Bewertung und Verteilung von Updates |
| HÃ¤rtung | Gezielte Reduzierung der AngriffsflÃ¤che |
| Sicherheitsbaseline | Definierter, geprÃ¼fter Ausgangszustand |
| Incident Response | Geordnetes Vorgehen bei SicherheitsvorfÃ¤llen |
