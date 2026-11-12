# Webhacking mit TryHackMe

Die Quellen sind nach Themengebiet und in einer sinnvollen Lernreihenfolge geordnet.

## [Grundlagen, Analyse und Informationsbeschaffung](01_Grundlagen_und_Informationsbeschaffung.md)

Einführung in den Ablauf eines Web-Penetrationstests, manuelle Untersuchung einer Anwendung, Erkennung verwendeter Technologien sowie Suche nach versteckten Dateien, Verzeichnissen und Endpunkten.

### Quellen

- [Guided Pentest: Web](https://tryhackme.com/room/guidedpentestweb)
- [Walking an Application](https://tryhackme.com/room/walkinganapp)
- [Content Discovery](https://tryhackme.com/room/contentdiscoveryx)
- [Modern Web Stacks](https://tryhackme.com/room/modernwebstacks)

---

## [Webserver und Fehlkonfigurationen](02_Webserver_und_Fehlkonfigurationen.md)

Identifikation und Untersuchung verschiedener Webserver wie Apache, Nginx, Node.js, Python HTTP Server und IIS. Behandelt werden unter anderem exponierte Dateien, unsichere Konfigurationen, WebDAV und Webshells.

### Quellen

- [Web Server Attacks I](https://tryhackme.com/room/webserverattacks)
- [Web Server Attacks II](https://tryhackme.com/room/webserverattacks2)

---

## [Injection-Angriffe](03_Injection_Angriffe.md)

Einschleusen manipulierter Eingaben in Datenbankabfragen, Betriebssystembefehle oder Dateipfade. Dazu gehören SQL Injection, Command Injection sowie Local und Remote File Inclusion.

### Quellen

- [SQL Injection Introduction](https://tryhackme.com/room/sqlinjectionintroduction)
- [OS Command Injection](https://tryhackme.com/room/oscommandinjection)
- [File Inclusion](https://tryhackme.com/room/fileinc)

---

## [Client- und benutzerbezogene Angriffe](04_Client_und_Benutzerangriffe.md)

Manipulation des Browsers oder bereits authentifizierter Benutzer durch Cross-Site Scripting und Cross-Site Request Forgery.

### Quellen

- [XSS Introduction](https://tryhackme.com/room/xssintroduction)
- [CSRF Introduction](https://tryhackme.com/room/csrfintroduction)

---

## [Serverseitige Requests und interne Ressourcen](05_SSRF_und_interne_Ressourcen.md)

Ausnutzung von Server-Side Request Forgery, um den Anwendungsserver zu Anfragen an interne Dienste, Cloud-Metadaten oder andere geschützte Ziele zu veranlassen.

### Quellen

- [Intro to SSRF](https://tryhackme.com/room/ssrfqi)

---

## [Authentifizierung, Sitzungen und Zugriffskontrolle](06_Authentifizierung_Sitzungen_Zugriffskontrolle.md)

Schwachstellen bei Anmeldung, Sitzungsverwaltung und Berechtigungsprüfung. IDOR ermöglicht beispielsweise den Zugriff auf fremde Objekte durch veränderte Referenzen.

### Quellen

- [Broken Authentication](https://tryhackme.com/room/brokenauthentication)
- [Session Management](https://tryhackme.com/room/sessionmanagement)
- [IDOR](https://tryhackme.com/room/idor)

---

## [API-Penetrationstests](07_API_Penetrationstests.md)

Untersuchung von APIs auf typische Schwachstellen wie fehlerhafte Authentifizierung, unzureichende objektbezogene Autorisierung, übermäßige Datenfreigabe und Mass Assignment.

### Quellen

- [API Pentesting](https://tryhackme.com/room/apitesting)

---

## [Abschließende Praxis und Challenges](08_Praxis_und_Challenges.md)

Anwendung und Kombination der zuvor erlernten Methoden in realistischeren Angriffsszenarien.

### Quellen

- [Recruit Web Challenge](https://tryhackme.com/room/recruitwebchallenge)
- [Support](https://tryhackme.com/room/support)