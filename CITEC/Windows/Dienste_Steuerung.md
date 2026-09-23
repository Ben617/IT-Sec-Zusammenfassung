# Windows-Dienste und Aufgabensteuerung – theoretische Grundlagen

## Lernziele

Nach diesem Kapitel kannst du:

- Anwendungen, Prozesse, Dienste und geplante Aufgaben unterscheiden,
- die Rolle des Service Control Managers erklären,
- Starttypen und Dienstzustände einordnen,
- Dienstkonten und deren Sicherheitsbedeutung beschreiben,
- Abhängigkeiten und Wiederherstellungsaktionen erklären,
- Aufbau und Ausführung geplanter Aufgaben verstehen,
- wichtige Verwaltungswerkzeuge und Befehle einordnen,
- typische Dienst- und Aufgabenfehler systematisch untersuchen.

## 1. Programme, Prozesse, Dienste und Aufgaben

Die Begriffe Programm, Prozess, Dienst und Aufgabe beschreiben unterschiedliche Dinge.

### Programm

Ein **Programm** ist ausführbarer Code auf einem Datenträger, beispielsweise eine EXE-Datei. Solange das Programm nicht gestartet wurde, ist es kein laufender Prozess.

### Prozess

Ein **Prozess** ist eine laufende Instanz eines Programms. Er besitzt unter anderem:

- eine Prozess-ID (PID),
- einen eigenen virtuellen Adressraum,
- mindestens einen Thread,
- ein Zugriffstoken,
- geöffnete Handles,
- zugewiesene Systemressourcen.

Ein Programm kann gleichzeitig in mehreren Prozessen laufen. Umgekehrt kann ein Prozess mehrere Aufgaben oder Komponenten bereitstellen.

### Anwendung

Eine **Anwendung** ist Software, mit der ein Benutzer typischerweise interagiert. Sie kann aus einem oder mehreren Prozessen bestehen und muss nicht zwingend eine sichtbare Oberfläche besitzen.

### Dienst

Ein **Windows-Dienst** ist eine vom Betriebssystem verwaltete Hintergrundkomponente. Dienste können unabhängig von einer interaktiven Benutzeranmeldung starten und laufen.

Typische Aufgaben von Diensten sind:

- Netzwerkfunktionen bereitstellen,
- Druckaufträge verwalten,
- Updates suchen und installieren,
- Ereignisse protokollieren,
- Sicherheitsfunktionen ausführen,
- Datenbanken oder Serveranwendungen betreiben.

### Geplante Aufgabe

Eine **geplante Aufgabe** führt ein Programm oder Skript aus, wenn ein definierter Auslöser eintritt. Sie läuft nicht zwingend dauerhaft.

Mögliche Auslöser sind:

- eine bestimmte Uhrzeit,
- die Anmeldung eines Benutzers,
- der Systemstart,
- ein bestimmtes Ereignis,
- Leerlauf des Computers,
- Herstellung einer Netzwerkverbindung.

### Vergleich

| Element | Start | Typische Laufzeit | Benutzeroberfläche |
|---|---|---|---|
| Anwendung | meist durch Benutzer | nach Bedarf | häufig vorhanden |
| Prozess | durch Benutzer, System oder anderen Prozess | beliebig | nicht zwingend |
| Dienst | durch Dienststeuerung | häufig dauerhaft | normalerweise keine direkte Oberfläche |
| Geplante Aufgabe | durch Auslöser oder manuell | häufig zeitlich begrenzt | normalerweise keine |

> **Merksatz:** Ein Dienst ist eine verwaltete Hintergrundfunktion, ein Prozess ist ihre laufende technische Instanz und eine geplante Aufgabe ist eine auslösergesteuerte Ausführungsdefinition.

## 2. Architektur der Windows-Dienste

### Service Control Manager

Der **Service Control Manager (SCM)** ist die zentrale Verwaltungsinstanz für Windows-Dienste. Er startet während des Systemstarts und verwaltet die installierten Dienste.

Zu seinen Aufgaben gehören:

- Dienstkonfigurationen lesen,
- Dienste starten und stoppen,
- Steuerbefehle an Dienste senden,
- Startreihenfolge und Abhängigkeiten berücksichtigen,
- Dienststatus an Verwaltungsprogramme melden,
- Wiederherstellungsaktionen auslösen.

Die Dienstkonfiguration wird hauptsächlich in der Registry unter folgendem Bereich gespeichert:

```text
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services
```

Änderungen sollten normalerweise über vorgesehene Verwaltungswerkzeuge und nicht direkt in der Registry erfolgen.

### Dienstprogramm und Dienstprozess

Ein Dienst ist zunächst eine registrierte Konfiguration. Erst wenn er gestartet wird, läuft sein ausführbarer Code in einem Prozess.

Es gibt zwei grundlegende Varianten:

- ein Dienst besitzt einen eigenen Prozess,
- mehrere Dienste teilen sich einen Hostprozess.

### Service Host

Viele Windows-Dienste werden durch `svchost.exe` ausgeführt. Der **Service Host** lädt Dienste, die häufig als DLL implementiert sind.

Mehrere `svchost.exe`-Prozesse sind normal. Windows trennt Dienste je nach Version, Ressourcen und Sicherheitsanforderungen in unterschiedliche Hostprozesse oder Gruppen.

Ein Prozessname allein zeigt daher nicht immer, welcher Dienst darin läuft.

### Dienstname und Anzeigename

Jeder Dienst besitzt mindestens:

- einen **Dienstnamen**, den Befehlszeilenwerkzeuge und das System verwenden,
- einen **Anzeigenamen**, der in grafischen Werkzeugen lesbar dargestellt wird.

Beide Namen können voneinander abweichen.

Beispiel:

```text
Anzeigename: Windows-Ereignisprotokoll
Dienstname:  EventLog
```

## 3. Dienstzustände

Ein Dienst kann verschiedene Zustände besitzen:

| Zustand | Bedeutung |
|---|---|
| Wird ausgeführt | Dienst ist aktiv |
| Beendet | Dienst läuft nicht |
| Wird gestartet | Startvorgang läuft |
| Wird beendet | Beendigung läuft |
| Angehalten | Verarbeitung wurde vorübergehend pausiert |
| Wird fortgesetzt | Dienst wechselt aus dem Pausenzustand |

Nicht jeder Dienst unterstützt Pausieren und Fortsetzen.

### Statusübergänge

Ein Dienst reagiert auf unterstützte Steuerbefehle. Dazu gehören je nach Implementierung:

- Starten,
- Stoppen,
- Pausieren,
- Fortsetzen,
- benutzerdefinierte Steuerbefehle.

Ein Neustart ist technisch meist eine Kombination aus Stoppen und anschließendem Starten.

### Ausstehender Zustand

Bleibt ein Dienst lange in „Wird gestartet“ oder „Wird beendet“, wartet er möglicherweise auf:

- eine Abhängigkeit,
- eine Netzwerkantwort,
- eine Datei oder Datenbank,
- einen Treiber,
- die Freigabe einer Ressource,
- das Ende eines internen Vorgangs.

Das sofortige Beenden des zugehörigen Prozesses kann Datenverlust oder Folgefehler verursachen und sollte nicht der erste Diagnoseschritt sein.

## 4. Starttypen

Der Starttyp legt fest, unter welchen Bedingungen ein Dienst gestartet wird.

### Automatisch

Der Dienst wird während des Systemstarts automatisch gestartet.

Typische Verwendung:

- grundlegende Systemfunktionen,
- dauerhaft benötigte Serverdienste,
- Sicherheits- oder Verwaltungsdienste.

### Automatisch (Verzögerter Start)

Der Dienst startet automatisch, aber zeitlich nach den unmittelbar benötigten automatischen Diensten. Dadurch kann der frühe Systemstart entlastet werden.

Ein verzögerter Start ist keine genaue Zeitplanung und ersetzt keine geplante Aufgabe.

### Manuell

Der Dienst startet bei Bedarf, durch einen Benutzer, eine Anwendung, einen anderen Dienst oder einen Systemmechanismus.

„Manuell“ bedeutet daher nicht, dass ausschließlich ein Mensch den Dienst starten kann.

### Manuell mit Auslöserstart

Viele moderne Dienste werden ereignisgesteuert gestartet oder beendet. Auslöser können beispielsweise sein:

- Geräteankunft,
- Netzwerkverfügbarkeit,
- Firewallereignisse,
- Domänenbeitritt,
- benutzerdefinierte Systemereignisse.

Die Dienste-Konsole stellt diese Unterscheidung nicht immer vollständig dar.

### Deaktiviert

Ein deaktivierter Dienst kann nicht normal gestartet werden, bevor sein Starttyp geändert wurde.

Das Deaktivieren unbekannter Windows-Dienste kann Anmeldung, Netzwerk, Updates, Sicherheit oder Wiederherstellung beeinträchtigen. Es sollte nur nach Prüfung von Zweck und Abhängigkeiten erfolgen.

### Starttyp und Status unterscheiden

Der Starttyp beschreibt die Startregel, nicht den aktuellen Zustand.

Beispiele:

- Ein automatischer Dienst kann wegen eines Fehlers beendet sein.
- Ein manueller Dienst kann aktuell ausgeführt werden.
- Ein deaktivierter Dienst ist normalerweise beendet, doch Status und Konfiguration bleiben verschiedene Eigenschaften.

## 5. Dienstkonten

Jeder Dienst läuft in einem Sicherheitskontext. Das verwendete Konto bestimmt, auf welche lokalen und entfernten Ressourcen der Dienst zugreifen kann.

### LocalSystem

`LocalSystem` besitzt sehr weitreichende Rechte auf dem lokalen Computer. Im Netzwerk tritt es typischerweise mit der Identität des Computerkontos auf.

Es sollte nur eingesetzt werden, wenn diese Rechte tatsächlich erforderlich sind.

### LocalService

`LocalService` besitzt stark eingeschränkte lokale Rechte. Im Netzwerk verwendet es eine anonyme beziehungsweise stark begrenzte Identität.

### NetworkService

`NetworkService` besitzt eingeschränkte lokale Rechte, kann im Netzwerk jedoch typischerweise mit der Identität des Computerkontos auftreten.

### Virtuelle Dienstkonten

Virtuelle Konten isolieren Dienste voneinander, ohne dass ein eigenes Kennwort manuell verwaltet werden muss.

Sie erscheinen beispielsweise in dieser Form:

```text
NT SERVICE\Dienstname
```

### Domänenkonten

Ein Dienst kann mit einem normalen Domänenkonto ausgeführt werden, wenn er auf Domänenressourcen zugreifen muss. Dabei entstehen jedoch Verwaltungsrisiken:

- Kennwörter müssen geschützt und regelmäßig geändert werden,
- abgelaufene oder geänderte Kennwörter können den Dienststart verhindern,
- interaktive Anmeldung sollte möglichst unterbunden werden,
- das Konto darf nur die notwendigen Rechte besitzen.

### Verwaltete Dienstkonten

In Active-Directory-Umgebungen können **Managed Service Accounts** beziehungsweise **Group Managed Service Accounts (gMSA)** die Kennwortverwaltung automatisieren.

gMSAs eignen sich für unterstützte Dienste auf einem oder mehreren Servern und reduzieren Risiken manuell verwalteter Dienstkennwörter.

### Anmelden als Dienst

Ein Konto benötigt das Benutzerrecht **Anmelden als Dienst**, damit der Service Control Manager einen Dienst in diesem Kontext starten kann. Sicherheitsrichtlinien oder Gruppenrichtlinien können dieses Recht vergeben oder entziehen.

## 6. Dienstberechtigungen und Sicherheit

Dienste besitzen eigene Zugriffssteuerungslisten. Diese bestimmen beispielsweise, wer einen Dienst:

- abfragen,
- starten,
- stoppen,
- pausieren,
- konfigurieren,
- löschen darf.

### Prinzip der minimalen Rechte

Ein Dienst sollte:

- mit dem am wenigsten privilegierten geeigneten Konto laufen,
- nur auf notwendige Dateien, Registrybereiche und Netzwerkziele zugreifen,
- keine interaktive Anmeldung benötigen,
- nur kontrolliert konfigurierbar sein.

### Unsichere Dienstkonfigurationen

Mögliche Schwachstellen sind:

- beschreibbare Dienstprogramme oder Programmordner,
- unsichere Berechtigungen auf der Dienstkonfiguration,
- ungeschützte Dienstkennwörter,
- übermäßig privilegierte Konten,
- fehlerhafte nicht in Anführungszeichen gesetzte Programmpfade mit Leerzeichen,
- unnötige Netzwerkfreigaben oder Ports,
- veraltete Dienstsoftware.

Kann ein Standardbenutzer die ausführbare Datei eines hoch privilegierten Dienstes verändern, kann daraus eine Rechteausweitung entstehen.

### Dienstisolierung

Windows verwendet verschiedene Mechanismen, um Dienste zu begrenzen, beispielsweise:

- getrennte Hostprozesse,
- dienstbezogene SIDs,
- eingeschränkte Tokens,
- Schutzstufen für besonders sensible Prozesse,
- Firewallregeln pro Dienst,
- kontrollierte Zugriffsrechte.

## 7. Abhängigkeiten

Ein Dienst kann von anderen Diensten oder Systemkomponenten abhängig sein. Der Service Control Manager berücksichtigt solche Abhängigkeiten beim Starten und Stoppen.

Beispiel:

```text
Dienst A benötigt Dienst B
```

Beim Start von A wird B zuerst gestartet. Beim Stoppen von B müssen möglicherweise auch abhängige Dienste beendet werden.

### Abhängig von und abhängig durch

Es sind zwei Blickrichtungen zu unterscheiden:

- **Abhängigkeiten dieses Dienstes:** Was benötigt der ausgewählte Dienst?
- **Abhängige Dienste:** Welche anderen Dienste benötigen den ausgewählten Dienst?

### Praktische Bedeutung

Startet ein Dienst nicht, kann die eigentliche Ursache in einer nicht verfügbaren Abhängigkeit liegen. Ein Fehler des abhängigen Dienstes ist dann möglicherweise nur ein Folgefehler.

Nicht jede technische Abhängigkeit ist ausdrücklich als Dienstabhängigkeit registriert. Ein Dienst kann zusätzlich auf DNS, Datenbanken, Zertifikate, Dateien oder Netzwerkziele angewiesen sein.

## 8. Wiederherstellungsaktionen

Windows kann nach einem unerwarteten Dienstfehler automatische Aktionen ausführen.

Mögliche Aktionen sind:

- keine Aktion,
- Dienst neu starten,
- ein Programm ausführen,
- Computer neu starten.

Für den ersten, zweiten und weitere Fehler können unterschiedliche Reaktionen festgelegt werden.

### Fehlerzähler

Der Fehlerzähler kann nach einer definierten Zeit zurückgesetzt werden. Dadurch unterscheidet Windows wiederholte Fehler in kurzer Zeit von seltenen Einzelproblemen.

### Neustartverzögerung

Eine kurze Verzögerung verhindert, dass ein Dienst unmittelbar und ohne Pause immer wieder startet und abstürzt.

### Grenzen automatischer Wiederherstellung

Ein automatischer Neustart verbessert die Verfügbarkeit, beseitigt aber nicht die Ursache. Wiederholte Abstürze müssen anhand von Ereignisprotokollen, Anwendungsprotokollen und gegebenenfalls Speicherabbildern untersucht werden.

Bei fehlerhafter Konfiguration kann eine aggressive Neustartstrategie Ressourcen verbrauchen oder weitere Fehler verdecken.

## 9. Dienste verwalten

### Dienste-Konsole

Die grafische Dienste-Konsole wird über folgenden Befehl geöffnet:

```text
services.msc
```

Sie zeigt unter anderem:

- Dienst- und Anzeigenamen,
- Beschreibung,
- aktuellen Status,
- Starttyp,
- Anmeldekonto,
- Abhängigkeiten,
- Wiederherstellungsoptionen.

### Computerverwaltung

Auch die Computerverwaltung enthält die Diensteansicht:

```text
compmgmt.msc
```

### `sc.exe`

`sc.exe` kommuniziert mit dem Service Control Manager.

Beispiele:

```cmd
sc.exe query
sc.exe query EventLog
sc.exe qc EventLog
sc.exe start Dienstname
sc.exe stop Dienstname
```

Bei Konfigurationsbefehlen besitzt `sc.exe` eine ungewöhnliche Syntax: Nach dem Gleichheitszeichen wird häufig ein Leerzeichen erwartet. Änderungen sollten deshalb anhand der dokumentierten Syntax erfolgen.

### PowerShell

PowerShell stellt objektorientierte Befehle bereit:

```powershell
Get-Service
Get-Service -Name EventLog
Start-Service -Name Dienstname
Stop-Service -Name Dienstname
Restart-Service -Name Dienstname
Set-Service -Name Dienstname -StartupType Automatic
```

`Get-Service` zeigt nicht jede Konfigurationseigenschaft. Für weitergehende Informationen können CIM-Abfragen verwendet werden:

```powershell
Get-CimInstance Win32_Service
```

### Berechtigungen

Das Anzeigen eines Dienststatus ist häufig ohne Erhöhung möglich. Starten, Stoppen und Konfigurieren erfordern abhängig von der Dienst-ACL administrative oder delegierte Rechte.

## 10. Geplante Aufgaben

Die **Windows-Aufgabenplanung** führt Programme oder Skripte abhängig von definierten Bedingungen aus.

Die grafische Verwaltung wird über folgenden Befehl geöffnet:

```text
taskschd.msc
```

### Task Scheduler Service

Der Dienst **Aufgabenplanung** verwaltet und startet geplante Aufgaben. Die Definitionen werden in einer hierarchischen Aufgabenplanungsbibliothek organisiert.

Microsoft und installierte Anwendungen legen häufig eigene Unterordner und Aufgaben an. Eine große Zahl vorhandener Aufgaben ist daher nicht automatisch verdächtig.

### Bestandteile einer Aufgabe

Eine Aufgabe besteht im Wesentlichen aus:

- allgemeinen Eigenschaften,
- Triggern,
- Aktionen,
- Bedingungen,
- Einstellungen,
- einem Sicherheitskontext.

## 11. Trigger

Ein **Trigger** bestimmt, wann eine Aufgabe gestartet werden soll.

Typische Trigger sind:

- nach Zeitplan,
- einmalig,
- täglich, wöchentlich oder monatlich,
- beim Systemstart,
- bei der Anmeldung,
- beim Sperren oder Entsperren einer Sitzung,
- beim Auftreten eines Ereignisses,
- beim Erstellen oder Ändern einer Aufgabe,
- bei Leerlauf.

### Ereignisbasierte Trigger

Eine Aufgabe kann auf einen Eintrag im Ereignisprotokoll reagieren. Dabei werden typischerweise Protokoll, Quelle und Ereignis-ID ausgewählt.

Dies ermöglicht beispielsweise:

- eine Benachrichtigung bei einem bestimmten Fehler,
- das Starten einer Diagnose,
- eine definierte Reaktion auf ein Systemereignis.

Der Trigger beweist nicht, dass die Reaktion erfolgreich war. Ausführung und Ergebnis müssen getrennt kontrolliert werden.

### Wiederholung

Zeittrigger können in Intervallen wiederholt werden. Eine zu kurze Wiederholung kann parallele Instanzen oder unnötige Systemlast verursachen.

## 12. Aktionen

Eine **Aktion** legt fest, was die Aufgabe ausführt.

Die übliche Aktion ist:

- Programm oder Skript starten.

Für die Ausführung sind drei Angaben besonders wichtig:

```text
Programm/Skript
Argumente
Starten in
```

### Programm und Argumente trennen

Der Pfad zum Interpreter oder Programm gehört in das Programmfeld, die Parameter in das Argumentfeld.

Beispiel für PowerShell:

```text
Programm/Skript: powershell.exe
Argumente:       -NoProfile -File "C:\Scripts\Backup.ps1"
Starten in:      C:\Scripts
```

### Arbeitsverzeichnis

Geplante Aufgaben verwenden häufig ein anderes Arbeitsverzeichnis als ein interaktiv gestartetes Programm. Relative Pfade können deshalb scheitern.

Skripte sollten möglichst:

- absolute Pfade verwenden,
- Fehler und Ergebnisse protokollieren,
- geeignete Exitcodes zurückgeben,
- nicht von einem interaktiven Desktop abhängen.

### Veraltete Aktionen

Historische Windows-Versionen boten zusätzliche Aktionsarten wie E-Mail-Versand oder Nachrichtenanzeige. Diese gelten in modernen Windows-Versionen als veraltet. Üblicherweise wird stattdessen ein geeignetes Programm oder Skript gestartet.

## 13. Bedingungen und Einstellungen

### Bedingungen

Bedingungen schränken die Ausführung zusätzlich ein. Beispiele:

- nur im Leerlauf starten,
- nur bei Netzstrom starten,
- bei Wechsel in den Akkubetrieb beenden,
- nur bei verfügbarer Netzwerkverbindung starten,
- Computer zum Ausführen reaktivieren.

Ein ausgelöster Task kann wegen einer nicht erfüllten Bedingung trotzdem nicht starten.

### Einstellungen

Einstellungen steuern das Verhalten der Aufgabe, beispielsweise:

- verpassten Start nachholen,
- Aufgabe bei Fehler erneut starten,
- nach einer Höchstdauer beenden,
- auf Anforderung starten lassen,
- Verhalten bei bereits laufender Instanz festlegen,
- Aufgabe nach Ablauf automatisch löschen.

### Mehrfachinstanzen

Wenn eine Aufgabe erneut ausgelöst wird, während sie noch läuft, sind je nach Konfiguration unterschiedliche Reaktionen möglich:

- keine neue Instanz starten,
- neue Instanz parallel starten,
- neue Instanz in eine Warteschlange stellen,
- bestehende Instanz beenden und neu starten.

Parallele Ausführung kann zu konkurrierenden Datei- oder Datenbankzugriffen führen.

## 14. Sicherheitskontext geplanter Aufgaben

Eine Aufgabe wird unter einem bestimmten Konto ausgeführt. Dieses Konto bestimmt die Zugriffsrechte des gestarteten Prozesses.

### Nur bei Benutzeranmeldung ausführen

Die Aufgabe läuft nur, wenn der angegebene Benutzer interaktiv angemeldet ist. Sie kann mit einer sichtbaren Benutzeroberfläche interagieren.

### Unabhängig von der Benutzeranmeldung ausführen

Die Aufgabe kann im Hintergrund ausgeführt werden, ohne dass eine interaktive Sitzung besteht. Eine Benutzeroberfläche ist dann normalerweise nicht sichtbar oder bedienbar.

Je nach Konto und Anmeldemethode müssen Anmeldeinformationen gespeichert oder geeignete Anmelderechte vorhanden sein.

### Mit höchsten Privilegien ausführen

Diese Option startet die Aufgabe mit erhöhtem Token, sofern das angegebene Konto die erforderlichen Rechte besitzt.

Die Option macht aus einem Standardkonto kein Administratorkonto. Sie beeinflusst, wie die vorhandenen Rechte des Kontos verwendet werden.

### Lokale und Netzwerkressourcen

Aufgaben können interaktiv funktionieren und im Hintergrund dennoch scheitern, weil:

- Netzlaufwerke nicht verbunden sind,
- andere Anmeldeinformationen gelten,
- Benutzerprofile oder Umgebungsvariablen fehlen,
- Zugriff auf Netzwerkressourcen nicht möglich ist,
- das Konto kein Recht für die Stapelverarbeitungsanmeldung besitzt.

UNC-Pfade wie `\\server\freigabe\ordner` sind für Hintergrundaufgaben meist verlässlicher als gemappte Laufwerksbuchstaben.

### Anmelden als Stapelverarbeitungsauftrag

Für bestimmte geplante Aufgaben benötigt das Konto das Benutzerrecht **Anmelden als Stapelverarbeitungsauftrag**. Lokale oder zentrale Sicherheitsrichtlinien können dieses Recht steuern.

## 15. Aufgaben verwalten

### Aufgabenplanung

Die grafische Aufgabenplanung zeigt:

- Ordnerstruktur,
- Trigger und Aktionen,
- nächsten und letzten Ausführungszeitpunkt,
- letzten Ergebniscode,
- Aufgabenverlauf,
- Sicherheitskontext.

Der „letzte Ausführungsstatus“ sollte nicht isoliert betrachtet werden. Ein erfolgreich gestartetes Skript kann seine eigentliche fachliche Aufgabe trotzdem nicht erfüllt haben, wenn es Fehler nicht korrekt zurückmeldet.

### `schtasks.exe`

Beispiele:

```cmd
schtasks.exe /Query /FO LIST /V
schtasks.exe /Run /TN "\Ordner\Aufgabe"
schtasks.exe /End /TN "\Ordner\Aufgabe"
```

Das Erstellen und Ändern von Aufgaben kann vertrauliche Angaben und komplexe Parameter betreffen. Kennwörter sollten nicht ungeschützt in Skripten oder Befehlsverläufen hinterlegt werden.

### PowerShell

PowerShell bietet unter anderem:

```powershell
Get-ScheduledTask
Get-ScheduledTaskInfo -TaskName "Aufgabenname"
Start-ScheduledTask -TaskName "Aufgabenname"
Disable-ScheduledTask -TaskName "Aufgabenname"
Enable-ScheduledTask -TaskName "Aufgabenname"
```

Für die Erstellung werden Trigger, Aktion, Einstellungen und Sicherheitskontext als getrennte Objekte definiert und anschließend registriert.

## 16. Dienste, Aufgaben und Autostart

Windows besitzt mehrere Mechanismen für automatische Ausführung:

| Mechanismus | Geeignet für |
|---|---|
| Dienst | Dauerhafte Hintergrundfunktion mit Dienststeuerung |
| Geplante Aufgabe | Zeit- oder ereignisgesteuerte Ausführung |
| Autostart-Ordner | Einfache Programme nach Benutzeranmeldung |
| Registry-Autostart | Programme bei Anmeldung oder Systemstart |
| Gruppenrichtlinie | Zentrale Start-, Anmelde- und weitere Skripte |

### Auswahl des Mechanismus

Ein Dienst ist passend, wenn eine Komponente dauerhaft, ohne Benutzeranmeldung und mit Dienststeuerung laufen soll.

Eine geplante Aufgabe ist passend, wenn eine Aktion:

- nur zu bestimmten Zeiten,
- nach einem Ereignis,
- in festen Intervallen,
- oder nur bei bestimmten Bedingungen ausgeführt werden soll.

Ein normales Autostartprogramm ist für benutzerbezogene Anwendungen nach der Anmeldung geeignet.

## 17. Protokollierung

### Dienste

Dienstprobleme erscheinen häufig in folgenden Ereignisprotokollen:

- **System**, insbesondere Meldungen des Service Control Managers,
- **Anwendung**, wenn die Dienstsoftware dort protokolliert,
- anwendungs- und dienstspezifische Protokolle.

Zusätzlich können Dienste eigene Logdateien schreiben.

### Aufgabenplanung

Die Aufgabenplanung besitzt einen Verlauf und ein eigenes Ereignisprotokoll. Es enthält unter anderem Informationen zu:

- Registrierung und Änderung einer Aufgabe,
- Auslösung,
- gestarteter Aktion,
- Abschluss oder Fehler,
- verwendeter Instanz.

Ist der Aufgabenverlauf deaktiviert, fehlen möglicherweise wichtige Diagnosedaten.

### Exitcodes

Ein Prozess gibt beim Beenden üblicherweise einen numerischen Exitcode zurück. Häufig bedeutet `0` Erfolg, doch die genaue Bedeutung legt das jeweilige Programm fest.

Ein Hexadezimalwert in der Aufgabenplanung muss anhand der Dokumentation des gestarteten Programms und des Windows-Fehlerkontexts interpretiert werden.

## 18. Sicherheit geplanter Aufgaben

Geplante Aufgaben können mit hohen Rechten laufen und sind deshalb sicherheitsrelevant.

Zu prüfen sind:

- Wer darf die Aufgabe ändern?
- Unter welchem Konto läuft sie?
- Welche Rechte besitzt dieses Konto?
- Wer darf das gestartete Programm oder Skript verändern?
- Sind Argumente und Pfade sicher angegeben?
- Enthält die Definition Kennwörter oder andere Geheimnisse?
- Greift sie auf unsichere Netzwerkpfade zu?
- Werden Ausführung und Änderungen protokolliert?

### Rechteausweitung vermeiden

Läuft eine Aufgabe als Administrator oder `SYSTEM`, dürfen Standardbenutzer das aufgerufene Skript, Programm und dessen Verzeichnisse nicht verändern können. Andernfalls könnten sie eigenen Code mit höheren Rechten ausführen lassen.

### Verdächtige Aufgaben

Schadsoftware verwendet geplante Aufgaben häufig für Persistenz. Eine unbekannte Aufgabe ist aber nicht automatisch schädlich. Untersucht werden sollten:

- Hersteller und Pfad,
- Trigger,
- Aktion und Argumente,
- Sicherheitskontext,
- Erstellungs- oder Änderungszeit,
- digitale Signatur der ausgeführten Datei,
- zugehörige Ereignisse und Dateien.

## 19. Systematisches Troubleshooting bei Diensten

### Schritt 1: Fehlerbild eingrenzen

- Welcher Dienst ist betroffen?
- Wie lauten Dienstname und Anzeigename?
- Soll er automatisch oder bei Bedarf laufen?
- Startet er gar nicht oder beendet er sich später?
- Seit wann besteht das Problem?
- Welche Änderung ging ihm voraus?

### Schritt 2: Status und Konfiguration prüfen

- aktueller Status,
- Starttyp,
- Programmpfad,
- Dienstkonto,
- Abhängigkeiten,
- Wiederherstellungsaktionen,
- erforderliche Benutzerrechte.

### Schritt 3: Fehlermeldung vollständig erfassen

Fehlernummer, Zeitpunkt und genauer Wortlaut sind wichtig. Eine allgemeine Meldung in der Dienste-Konsole wird häufig durch detailliertere Ereignisse ergänzt.

### Schritt 4: Ereignisse und Anwendungslogs prüfen

Zuerst werden zeitlich passende Meldungen im System- und Anwendungsprotokoll gesucht. Anschließend sind gegebenenfalls herstellerspezifische Protokolle zu prüfen.

### Schritt 5: Abhängigkeiten und Ressourcen prüfen

- Laufen benötigte Dienste?
- Existieren Dateien und Verzeichnisse?
- Sind die Berechtigungen korrekt?
- Ist ein benötigter Port bereits belegt?
- Sind DNS, Netzwerk, Datenbank oder Zertifikat verfügbar?
- Ist ausreichend Speicherplatz vorhanden?

### Schritt 6: Dienstkonto prüfen

- Ist das Konto aktiv?
- Ist das Kennwort abgelaufen oder geändert?
- besitzt es „Anmelden als Dienst“?
- hat es Zugriff auf lokale und entfernte Ressourcen?
- blockiert eine Richtlinie die Anmeldung?

### Schritt 7: Ursache beheben und testen

Nach einer gezielten Änderung wird geprüft:

- startet der Dienst?
- bleibt er stabil?
- erfüllt er seine eigentliche Funktion?
- entstehen neue Warnungen oder Folgefehler?

Ein laufender Status allein beweist nicht, dass die Anwendung fachlich korrekt funktioniert.

## 20. Systematisches Troubleshooting bei Aufgaben

### Schritt 1: Trigger prüfen

- Ist die Aufgabe aktiviert?
- Ist der Trigger aktiviert und korrekt geplant?
- Wurde das Ereignis tatsächlich erzeugt?
- Ist die Systemzeit korrekt?
- wurde ein geplanter Start verpasst?

### Schritt 2: Bedingungen prüfen

- Wird Netzstrom verlangt?
- Muss der Computer im Leerlauf sein?
- Ist eine Netzwerkverbindung erforderlich?
- darf der Computer für die Aufgabe reaktiviert werden?

### Schritt 3: Aktion prüfen

- Existiert die ausführbare Datei?
- Sind Programm, Argumente und Arbeitsverzeichnis richtig getrennt?
- werden absolute Pfade verwendet?
- sind Anführungszeichen korrekt?
- funktioniert der Befehl im gleichen Kontokontext?

### Schritt 4: Sicherheitskontext prüfen

- Welches Konto führt die Aufgabe aus?
- Ist das Konto aktiv?
- besitzt es die erforderlichen Datei- und Netzwerkrechte?
- benötigt die Aufgabe höchste Privilegien?
- besitzt das Konto das erforderliche Anmelderecht?

### Schritt 5: Ergebnis untersuchen

- letzter Laufzeitpunkt,
- letzter Ergebniscode,
- Aufgabenverlauf,
- Ereignisprotokoll,
- eigene Logdatei des Skripts,
- Exitcode der gestarteten Anwendung.

### Schritt 6: Umgebung berücksichtigen

Eine geplante Aufgabe läuft häufig mit:

- anderem Arbeitsverzeichnis,
- anderen Umgebungsvariablen,
- nicht verbundenen Netzlaufwerken,
- nicht geladenem oder anderem Benutzerprofil,
- unsichtbarer Benutzeroberfläche.

Ein interaktiv erfolgreicher Befehl muss daher nicht automatisch als geplante Aufgabe funktionieren.

## 21. Nützliche Diagnosebefehle

### Dienste

```powershell
Get-Service
Get-Service -Name Dienstname | Format-List *
Get-CimInstance Win32_Service -Filter "Name='Dienstname'"
sc.exe query Dienstname
sc.exe qc Dienstname
tasklist.exe /svc
```

### Prozesse und Netzwerk

```powershell
Get-Process
Get-NetTCPConnection -State Listen
Get-Process -Id <PID>
```

### Aufgaben

```powershell
Get-ScheduledTask
Get-ScheduledTaskInfo -TaskName "Aufgabenname"
schtasks.exe /Query /TN "\Ordner\Aufgabe" /V /FO LIST
```

### Ereignisse

```powershell
Get-WinEvent -LogName System -MaxEvents 50
Get-WinEvent -LogName Application -MaxEvents 50
```

Für eine zielgerichtete Analyse sollten Ereignisse nach Zeitraum, Anbieter und Ereignis-ID gefiltert werden.

## 22. Typische Fehlermeldungen und Ursachen

| Fehlerbild | Mögliche Ursachen |
|---|---|
| Dienst startet nicht | Abhängigkeit fehlt, Konto fehlerhaft, Datei fehlt, Konfiguration ungültig |
| Zugriff verweigert | Fehlende Dienst-ACL, fehlende Erhöhung oder Ressourcenberechtigung |
| Anmeldefehler | Falsches Kennwort, fehlendes Anmelderecht, gesperrtes Konto |
| Dienst beendet sich sofort | Anwendungsfehler, ungültige Konfiguration, Portkonflikt |
| Start dauert zu lange | blockierte Abhängigkeit, Netzwerk- oder Datenträgerproblem |
| Aufgabe wurde nicht gestartet | Trigger, Bedingung, deaktivierte Aufgabe oder ausgeschalteter Computer |
| Aufgabe läuft manuell, aber nicht geplant | anderer Kontokontext, Pfad, Arbeitsordner oder Anmelderecht |
| Aufgabe endet erfolgreich, Ergebnis fehlt | Skript behandelt Fehler nicht oder liefert falschen Exitcode |
| Netzwerkpfad nicht erreichbar | fehlende Berechtigung, DNS, Konto oder gemapptes Laufwerk |
| Mehrere Instanzen kollidieren | ungeeignete Mehrfachinstanz-Einstellung |

Diese Zuordnungen sind Ausgangspunkte und keine eindeutigen Diagnosen.

## 23. Typische Missverständnisse

### „Automatisch bedeutet, dass der Dienst immer läuft.“

Nein. Der Starttyp fordert einen Start beim Systemstart an. Der Dienst kann später abstürzen, beendet werden oder nicht erfolgreich starten.

### „Manuell bedeutet, dass nur ein Mensch den Dienst starten kann.“

Nein. Anwendungen, andere Dienste und Systemauslöser können einen manuellen Dienst bei Bedarf starten.

### „Jeder `svchost.exe`-Prozess ist verdächtig.“

Nein. Windows verwendet normalerweise viele Service-Host-Prozesse. Entscheidend sind Dateipfad, Signatur, gehostete Dienste und Verhalten.

### „Ein laufender Dienst funktioniert vollständig.“

Nicht zwingend. Er kann laufen und trotzdem seine Datenbank, Netzwerkziele oder fachliche Funktion nicht erreichen.

### „Eine geplante Aufgabe ist dasselbe wie ein Dienst.“

Nein. Aufgaben sind auslösergesteuerte Ausführungen; Dienste sind durch den SCM verwaltete Hintergrundkomponenten.

### „Mit höchsten Privilegien macht jedes Konto zum Administrator.“

Nein. Die Option nutzt die vorhandenen Rechte des angegebenen Kontos in erhöhter Form. Sie verleiht einem Standardkonto keine nicht vorhandenen Administratorrechte.

### „Ein Rückgabecode 0 beweist den fachlichen Erfolg.“

Nicht immer. Er bedeutet nur das, was die Anwendung dafür definiert. Ein schlecht geschriebenes Skript kann trotz internem Fehler `0` zurückgeben.

### „Zum Testen kann man einen problematischen Dienst einfach deaktivieren.“

Das kann abhängige Systemfunktionen beeinträchtigen. Zuerst sollten Zweck, Abhängigkeiten und Folgen geprüft werden.

## 24. Bewährte Grundprinzipien

- Dienste und Aufgaben dokumentieren.
- Das Prinzip der minimalen Rechte anwenden.
- Integrierte oder verwaltete Dienstkonten bevorzugen.
- Dienstkennwörter nicht in Skripten speichern.
- Programme, Skripte und Konfigurationsdateien vor unbefugten Änderungen schützen.
- Absolute Pfade und eindeutige Anführungszeichen verwenden.
- Aufgaben und Dienste aussagekräftig protokollieren lassen.
- Fehler mit passenden Exitcodes zurückgeben.
- Neustartschleifen und parallele Instanzen vermeiden.
- Unbekannte Windows-Dienste nicht pauschal deaktivieren.
- Änderungen zunächst auf Testsystemen prüfen.
- Dienst- und Aufgabenänderungen regelmäßig kontrollieren.

## 25. Zusammenfassung

- Dienste sind vom Service Control Manager verwaltete Hintergrundkomponenten.
- Ein Dienst läuft in mindestens einem Prozess; mehrere Dienste können sich einen Hostprozess teilen.
- Starttyp und aktueller Status sind unterschiedliche Eigenschaften.
- Dienstkonten bestimmen den Sicherheitskontext und sollten minimale Rechte besitzen.
- Abhängigkeiten beeinflussen Start- und Stoppreihenfolgen.
- Wiederherstellungsaktionen erhöhen die Verfügbarkeit, beseitigen aber keine Fehlerursache.
- Geplante Aufgaben bestehen aus Triggern, Aktionen, Bedingungen, Einstellungen und Sicherheitskontext.
- Hintergrundaufgaben besitzen oft eine andere Umgebung als interaktiv gestartete Programme.
- Dienste und Aufgaben mit hohen Rechten müssen gegen unbefugte Änderungen geschützt werden.
- Ereignisprotokolle, Ergebniswerte und eigene Anwendungslogs sind zentrale Diagnosequellen.
- Ein laufender Dienst oder Rückgabecode allein beweist noch keinen fachlichen Erfolg.

## 26. Kontrollfragen

1. Worin unterscheiden sich Programm, Prozess, Dienst und geplante Aufgabe?
2. Welche Aufgaben übernimmt der Service Control Manager?
3. Warum sind mehrere `svchost.exe`-Prozesse normal?
4. Was unterscheidet Dienstname und Anzeigename?
5. Was ist der Unterschied zwischen Starttyp und Dienststatus?
6. Was bedeutet der Starttyp „Manuell“ tatsächlich?
7. Worin unterscheiden sich LocalSystem, LocalService und NetworkService?
8. Welchen Vorteil bieten verwaltete Dienstkonten?
9. Warum sind Dienstabhängigkeiten für die Fehlersuche wichtig?
10. Welche Grenzen haben automatische Wiederherstellungsaktionen?
11. Aus welchen Hauptbestandteilen besteht eine geplante Aufgabe?
12. Was unterscheidet einen Trigger von einer Bedingung?
13. Warum sollten Skripte in geplanten Aufgaben absolute Pfade verwenden?
14. Was bewirkt „Mit höchsten Privilegien ausführen“?
15. Warum funktionieren gemappte Netzlaufwerke in Hintergrundaufgaben oft nicht?
16. Welche Risiken entstehen, wenn ein Standardbenutzer das Skript einer SYSTEM-Aufgabe ändern kann?
17. Wo findest du Protokolle zu Dienst- und Aufgabenfehlern?
18. Warum beweist ein Exitcode `0` nicht zwingend den fachlichen Erfolg?
19. Welche Punkte prüfst du, wenn ein Dienst nicht startet?
20. Welche Punkte prüfst du, wenn eine Aufgabe nur bei manueller Ausführung funktioniert?

## 27. Begriffsübersicht

| Begriff | Kurzbeschreibung |
|---|---|
| Dienst | Vom SCM verwaltete Hintergrundkomponente |
| Prozess | Laufende Instanz eines Programms |
| SCM | Zentrale Verwaltung der Windows-Dienste |
| `svchost.exe` | Hostprozess für zahlreiche Windows-Dienste |
| Starttyp | Regel für den Start eines Dienstes |
| Dienststatus | Aktueller Ausführungszustand |
| Dienstkonto | Sicherheitsidentität eines Dienstes |
| gMSA | Automatisch verwaltetes AD-Dienstkonto |
| Abhängigkeit | Benötigter Dienst oder benötigte Komponente |
| Wiederherstellungsaktion | Automatische Reaktion auf einen Dienstfehler |
| Aufgabe | Definition einer auslösergesteuerten Ausführung |
| Trigger | Ereignis oder Zeitplan, der eine Aufgabe auslöst |
| Aktion | Auszuführendes Programm oder Skript |
| Bedingung | Zusätzliche Voraussetzung für die Ausführung |
| Aufgabeninstanz | Konkrete laufende Ausführung einer Aufgabe |
| Sicherheitskontext | Konto und Token, unter denen Code ausgeführt wird |
| Exitcode | Rückgabewert eines beendeten Prozesses |
| Autostart | Automatischer Programmstart, häufig nach Anmeldung |
