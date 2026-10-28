<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../assets/architecture.de.dark.svg">
  <img src="../assets/architecture.de.light.svg" width="100%" alt="Architektur in drei Reihen. Gäste erreichen die Gästeseiten und über das Einladungstor Profil und Lebenslauf. Der Owner meldet sich mit zwei Faktoren in der Web-App an und bearbeitet die Bausteine. AI Agents erreichen den MCP-Server und reichen Vorschläge ein, ebenso die Anbindungen an GitHub und LinkedIn. Übernommene Vorschläge gelangen in die Bausteine, die das Profil speisen. Die Bausteine liegen in Datenbank und Speicher; geplante Aufgaben räumen auf und gleichen ab, ohne Hintergrund-Worker.">
</picture>

[English](architecture.md) · **Deutsch** · [Zurück zur Übersicht](../README.de.md)

</div>

# Architektur von cvitea

cvitea ist eine Laravel-13-Anwendung mit Filament 5 für die Web-App des Owners, auf PHP 8.5 und MariaDB 10.11, auf Schweizer Webhosting ohne Hintergrund-Worker. Sie hat drei Zugänge, und jeder hat seinen eigenen Schutz: Gäste kommen über das Einladungstor, der Owner über eine Web-App mit Anmeldung mit zwei Faktoren, AI Agents über einen MCP-Server mit Tokens, die auf Berechtigungsbereiche begrenzt sind. Diese Seite beschreibt die Teile, drei Wege durch sie, die wichtigsten Entwurfsentscheide und wie Sicherheit eingebaut ist. Sie nennt Mechanismen, keine Einstellungen.

## Die Teile

| Teil | Was er tut |
|---|---|
| Gästeseiten | Die öffentliche Visitenkarte, das Einladungstor und die Seiten dahinter, auf Deutsch und Englisch |
| Profil und Lebenslauf | Ein Profil je Bewerbung, aus Bausteinen zusammengesetzt, mit den Dokumenten und einem Download für alle |
| Web-App | Der Arbeitsplatz des Owners: Bewerbungen, Bausteine, Vorschläge, Übersetzungen, Medien und Titelbilder, Verbindungen und das Protokoll |
| Bausteine | Die eine Quelle der Inhalte: Stellen, Projekte, Ausbildung, Fähigkeiten, Dokumente und Texte, je auf Deutsch und Englisch |
| MCP-Server | Lässt AI Agents Laufbahndaten lesen und Vorschläge einreichen, über OAuth oder ein persönliches Token |
| Vorschläge | Jede Änderung eines AI Agent oder eines Imports wartet hier, bis der Owner sie annimmt oder ablehnt |
| Anbindungen | Import aus GitHub und LinkedIn mit geführtem Vergleich; auf Klick des Owners gehen Änderungen zurück zu GitHub |
| Geplante Aufgaben | Löschen alte Aufrufe und Nachrichten, gleichen Quellen ab und räumen Dateien auf, gestartet vom Zeitplaner des Hostings |

## Drei Wege durch die Plattform

### Ein Gast öffnet eine Einladung

1. Das Öffnen der Einladungsseite zeigt ein Formular und ändert nichts: Gästeseiten laufen ohne Sitzung, und das blosse Öffnen einer Seite setzt nie ein Cookie, sodass Link-Scanner und Mail-Vorschauen keine Spur hinterlassen. Ein Cookie gibt es erst für einen gültigen Code.
2. Der Gast schickt den Code ab. Das Tor nimmt das Formular nur von der eigenen Website an, begrenzt die Versuche und sucht den Code über seinen Schlüssel-Hash. Ein unbekannter, abgelaufener oder widerrufener Link bekommt dieselbe neutrale Seite, sodass niemand erfährt, welcher der drei Fälle vorliegt.
3. Hat die Einladung eine PIN, fragt ein zweiter Schritt danach, mit eigenen Grenzen.
4. Der Zugang ist ein verschlüsseltes Cookie, das nur über HTTPS reist, für Skripte unerreichbar ist und mit der Einladung endet.
5. Jede weitere Anfrage prüft die Einladung erneut in der Datenbank, sodass ein widerrufener Link sofort nicht mehr geht.
6. Der Aufruf wird erfasst, nachdem die Seite ausgeliefert ist, mit groben Angaben und ohne IP-Adresse; bekannte Bots bleiben aussen vor.

### Ein AI Agent schlägt eine Änderung vor

1. Ein AI Agent verbindet sich über MCP, mit OAuth oder einem persönlichen Token. Jedes Werkzeug prüft die Berechtigungsbereiche des Tokens; Aufrufe, die an Herkunft oder Berechtigungsbereich scheitern, werden protokolliert.
2. Ein schreibendes Werkzeug legt einen Vorschlag an und hält fest, auf welchem Stand der Inhalte er beruht. Sichtbar ändert sich nichts, und der Owner wird benachrichtigt.
3. Der Owner prüft den Vorschlag in der Web-App und nimmt ihn an oder lehnt ihn ab.
4. Die Annahme schreibt die Änderung in einer Datenbank-Transaktion. Hat sich der Inhalt seit dem Vorschlag geändert, gilt der Vorschlag als veraltet und geht nur durch, wenn der Owner das bestätigt.
5. Gästeseiten lesen die Bausteine direkt, sodass eine angenommene Änderung ohne eigenen Veröffentlichungsschritt sichtbar ist. Importe aus GitHub und LinkedIn nehmen denselben Weg.

### Ein Release geht hinaus

1. Das Deploy-Skript lässt zuerst `composer ci` laufen und bricht ab, wenn es rot ist.
2. Ein Probelauf zeigt, was sich ändern würde; erst ein zweiter, ausdrücklicher Lauf wendet es an.
3. Das neue Release ersetzt das alte in einem einzigen Wechsel.
4. Eine Zustandsprüfung und die Prüfung der Sicherheits-Header folgen; danach beobachtet die koordinierende Session die Plattform zehn Minuten lang.
5. Der Rückweg: Dasselbe Skript kehrt zum vorherigen Release zurück. Änderungen an der Datenbank nimmt es nicht zurück.

## Entwurfsentscheide

| Entscheid | Warum | Was er kostet |
|---|---|---|
| Webhosting ohne Hintergrund-Worker | Wenig zu betreiben und abzusichern, gehostet in der Schweiz | Nacharbeit wie das Erfassen eines Aufrufs oder die Nachricht an den Owner läuft nach der Antwort; geplante Aufgaben startet der Zeitplaner des Hostings statt eines Workers |
| Bausteine als einzige Quelle der Inhalte | Eine Änderung erreicht Profil, Lebenslauf, Übersetzungen und Dokumente | Jede Ansicht entsteht aus Bausteinen, ihre Struktur muss deshalb stabil bleiben |
| AI Agents schlagen nur vor | AI Agents können lesen und entwerfen, während jede Änderung an dem, was Arbeitgeber sehen, eine menschliche Entscheidung bleibt | Der Owner prüft jeden Vorschlag; veraltete Vorschläge brauchen eine Bestätigung |
| Einladungscodes am Tor nur als Schlüssel-Hash | Eine Kopie der Datenbank verrät keine gültigen Links | Damit der Owner einen Link wieder anzeigen kann, hält die Plattform eine getrennt verschlüsselte Kopie des Codes |
| Eine neutrale Antwort für jeden geschlossenen Link | Niemand erfährt, ob ein Link existiert, abgelaufen ist oder widerrufen wurde | Ein Gast mit einem alten Link sieht dieselbe Seite wie jemand, der rät |
| Keine Sitzung auf Gästeseiten | Weniger bewegliche Teile; das Öffnen einer Seite hinterlässt nichts | Formulare auf Gästeseiten brauchen einen eigenen Schutz, ohne Sitzung als Stütze |
| Strikte Content Security Policy auf Gästeseiten | Keine eingebetteten Skripte oder Styles, also kann eingeschleustes Markup nicht laufen | Jedes Skript und jeder Style der Gästeseiten ist eine eigene Datei |
| Aufrufe ohne IP-Adresse | Der Owner sieht Interesse an einer Bewerbung, keine Netzwerkdaten | Orte sind nur ungefähr |
| Kein Scraping von Jobportalen | Die Nutzungsbedingungen der Portale | Daten kommen aus den eigenen Exporten der Person und aus offiziellen Schnittstellen |

## Sicherheit im Entwurf

- **Anmeldung des Owners** mit zweitem Faktor als Pflicht; nur der Owner erreicht die Web-App.
- **Einladungen**: zufällige Codes, gefunden über Schlüssel-Hash, sonst nur verschlüsselt gespeichert, gedrosselt und sofort widerrufbar; eine PIN je Link nach Wahl.
- **Zugang der Gäste**: ein verschlüsseltes Cookie, an die Einladung gebunden und bei jeder Anfrage erneut geprüft.
- **Header bei jeder Antwort**: Content-Security-Policy, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Cross-Origin-Opener-Policy, Cross-Origin-Resource-Policy, Permissions-Policy und Strict-Transport-Security; Seiten hinter einem Link senden zusätzlich X-Robots-Tag, damit Suchmaschinen sie auslassen.
- **AI Agents**: Tokens mit Berechtigungsbereichen, eine Prüfung der Bereiche in jedem Werkzeug, Grenzen je Token, nur Vorschläge oder private Uploads und kein Zugriff auf Einladungen, Aufrufe oder Nachrichten von Gästen.
- **Uploads** werden beim Eingang geprüft und bleiben privat, bis der Owner einen Vorschlag annimmt, der sie verwendet.
- **Personendaten** werden nur in Datenbank und Speicher der Plattform gespeichert; AI Agents, die der Owner anbindet, lesen Laufbahndaten und Dokumente im Rahmen ihrer Berechtigungsbereiche. Tests, Bildschirmfotos und Seeds nutzen eine erfundene Person, und eine Sperrliste und ein Secret-Scan prüfen jeden Export und jeden Push.
- **Prüfungen**: Sicherheitstests in der Pest-Suite, die Header-Prüfung lokal und in Produktion und der Prüfweg, beschrieben in [So entsteht cvitea](how-its-built.de.md).
- **Meldungen**: siehe [SECURITY.md](../SECURITY.md).
