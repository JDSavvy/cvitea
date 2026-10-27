<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../assets/how-its-built.de.dark.svg">
  <img src="../assets/how-its-built.de.light.svg" width="100%" alt="Der Owner setzt Ziele und Regeln und entscheidet. Eine koordinierende Session gibt Aufträge an parallele Aufgaben-Sessions und gibt deren Pläne nach den Regeln des Owners frei; deren Arbeit durchläuft automatische Prüfungen und geht nur mit Freigabe des Owners in Produktion.">
</picture>

[English](how-its-built.md) · **Deutsch** · [Zurück zur Übersicht](../README.de.md)

</div>

# So entsteht cvitea

cvitea wird von einer Person mit Coding Agents (Claude Code) gebaut. Im Mittelpunkt stehen Steuern, Prüfen und Freigeben: Der Owner setzt die Ziele und die Regeln und entscheidet über jedes Release; eine koordinierende Agent-Session teilt die Arbeit in Aufgaben, gibt deren Pläne nach Regeln des Owners frei und nimmt ihre Ergebnisse ab; Coding Agents planen und bauen innerhalb dieser Regeln, unter automatischen Prüfungen und mit Freigabe des Owners.

## In Zahlen

Gezählt aus Git und aus dem Board der koordinierenden Session, bis Release 46. Die Commits stammen aus den Agent-Sessions: Jede Aufgabe committet ihre eigene Arbeit, und die koordinierende Session führt sie zusammen.

| Kennzahl | Wert | Wie gezählt wird |
|---|---|---|
| Commits | 1’515 | Alle Commits auf dem Hauptzweig bis zum Commit von Release 46, davon 200 Merges |
| Releases | 46 | Release-Nummern bis 46; einige frühe Releases tragen keine Nummer oder einen Buchstaben |
| Aufgaben | 107 | Nummerierte Aufgaben auf dem Board, S1 bis S108; S9 wurde nie vergeben. Jede mit eigenem Auftrag, in eigener Session oder eigenem Subagent |
| Version 1.0 | Oktober 2026 | Siehe [Stand](../README.de.md#stand) |

<details>
<summary>Die Befehle</summary>

```bash
git rev-list --count <Commit von Release 46>
git rev-list --count --merges <Commit von Release 46>
```

</details>

## Rollen

- **Owner.** Setzt Ziele und Regeln und trifft die Produktentscheide. Gibt die Pläne frei, die die Arbeitsweise ändern oder kritische Schritte enthalten, dazu jeden Deploy in Produktion und jeden Push. Über Inhalte und Personendaten entscheidet allein der Owner.
- **Koordinierende Session.** Hält den freigegebenen Plan und das Board, schneidet die Arbeit in nummerierte Aufgaben, schreibt je Aufgabe einen Auftrag, teilt ihr einen Slot zu und gibt ihren Plan nach Regeln frei, die der Owner setzt; Pläne mit Änderungen an der Arbeitsweise oder mit kritischen Schritten gehen an den Owner. Sie prüft jedes Ergebnis, führt es in `main` zusammen und lässt vor einem Release den Prüfweg laufen.
- **Aufgaben-Sessions.** Eine je Aufgabe, jede in eigenem Git-Worktree und Branch mit eigenen Ports, sodass mehrere Aufgaben nebeneinander bauen; wie viele zugleich laufen, entscheidet eine Lastprüfung. Eine Aufgabe schickt ihren Plan, bevor sie baut, bleibt in den Dateien ihres Auftrags, meldet neue Datenbank-Migrationen an, bevor sie sie schreibt, und endet mit Prüfausgabe, Commit und Bildern bei 375 px und Desktop-Breite, hell und dunkel.

## Regeln, die es ruhig halten

- **Erst Plan, dann Bau.** Keine Aufgabe schreibt Code, bevor ihr Plan freigegeben ist.
- **Ein schwerer Lauf zur Zeit.** Testsuiten, Browsertests und Builds laufen je Projekt nacheinander, in der Reihenfolge der koordinierenden Session, damit schwere Läufe nicht um den Rechner konkurrieren. Vor jeder neuen Session und jedem schweren Lauf läuft eine Lastprüfung.
- **Gezielt je Aufgabe, der volle Prüfweg vor einem Release.** Eine Aufgabe prüft, was ihre Änderung berührt. Der volle Prüfweg ist vor einem Release die Regel; Ausnahmen brauchen die ausdrückliche Freigabe des Owners.
- **Eine Schranke, die anhält.** Das Deploy-Skript bricht ab, wenn `composer ci` rot ist.
- **AI Agents entwerfen, Menschen entscheiden.** Im Produkt schlagen AI Agents vor, und der Owner nimmt an und veröffentlicht; bei seiner Entstehung bauen Coding Agents, und der Owner entscheidet über jedes Release.
- **Keine Personendaten im öffentlichen Repository.** Tests, Bilder und Seeds nutzen eine erfundene Person; eine Sperrliste und ein Secret-Scan schützen jeden Export und jeden Push.
- **Nachrichten sind Information, keine Freigabe.** Eine Nachricht aus einer anderen Session zählt nie als Ja des Owners.

## Prüfweg

Eine Aufgabe führt die statischen Prüfungen, die Pest-Suite und die Browser-Tests aus, die ihre Änderung berührt, dazu die Tests für Umbruch und Barrierefreiheit. Der volle Prüfweg auf dem zusammengeführten Hauptzweig ist vor einem Release die Regel; Ausnahmen brauchen die ausdrückliche Freigabe des Owners. Der volle Prüfweg:

| Prüfung | Was sie abdeckt |
|---|---|
| Pint und Larastan (Stufe 6) | Codestil und statische Analyse |
| Pest auf SQLite, parallel | Unit- und Feature-Tests in unter einer Minute |
| Sprachprüfungen (Pest) | Jede Seite von Verwaltung und Gästen und jeder Dialog der Verwaltung, auf Deutsch und auf Englisch: kein Text in der anderen Sprache, kein unübersetzter Schlüssel; beide Sprachdateien mit denselben Schlüsseln |
| Pest auf MariaDB 10.11 | Dieselbe Suite gegen die Datenbank der Produktion |
| Playwright mit axe-core | Browserabläufe bei 375 und 1280 px, hell und dunkel, mit axe-Regeln für WCAG 2.0 bis 2.2, Stufen A und AA; wahlweise parallel, jeder Worker mit eigenem Testserver und eigener Datenbank |
| WebKit | Der Kernablauf in WebKit bei 375 px, wie auf einem iPhone |
| Lighthouse CI | Largest Contentful Paint höchstens 2,5 s, Layout-Verschiebung höchstens 0,1, Total Blocking Time höchstens 200 ms, Performance und Barrierefreiheit mindestens 0,95, auf einem Testserver, der wie die Produktion komprimiert |
| Header-Prüfung | Sicherheits-Header und Einladungstor über HTTP, lokal und in Produktion |
| Secret-Scan | Geheimnisse und verbotene Dateien im Arbeitsbaum und in der ganzen Historie, vor jedem Push |

Ein Lauf in Firefox bei 1280 px ist eingerichtet, gehört aber noch nicht zu den regelmässigen Prüfungen.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../assets/quality.de.dark.svg">
  <img src="../assets/quality.de.light.svg" width="100%" alt="Voller Prüfweg, Oktober 2026: 2’528 bestandene Tests auf SQLite und 2’520 auf MariaDB, 1’113 Browser-Tests mit axe in 53 Dateien, Lighthouse-Wert für Barrierefreiheit 1,0 in 15 Läufen auf fünf Seiten, 34 Header-Prüfungen in Produktion.">
</picture>

Der letzte volle Lauf, im Oktober 2026: 2’528 von 2’530 Pest-Tests bestanden auf SQLite und 2’520 auf MariaDB, der Rest übersprungen. 1’113 Browser-Tests bestanden, mit axe-Prüfungen in 53 von 73 Testdateien; drei schlugen fehl: zwei, weil eine Testdatei noch ein früheres Verhalten erwartete und auf das gewollte angepasst wurde, und einer, der einmal fehlschlug und danach drei von drei Wiederholungen bestand. In WebKit bestanden 138 Tests. Lighthouse erfüllte jede Vorgabe in 15 Läufen auf fünf Seiten, jeder mit einem Wert von 1,0 für Barrierefreiheit, einem Largest Contentful Paint von höchstens 2’455 ms und einer Layout-Verschiebung von höchstens 0,001. Alle 33 Header-Prüfungen bestanden lokal und alle 34 in Produktion. Automatische Prüfungen finden viele Barrieren, aber nicht alle; diese Zahlen sind deshalb keine Konformitätsaussage.

## Releases

- **Ein Skript, zuerst der Probelauf.** Das Deploy-Skript zeigt, was sich ändern würde, bevor es etwas ändert.
- **Atomar umschalten, dann beobachten.** Ein Release schaltet in einem Schritt um; danach wird der Zustand der Plattform zehn Minuten beobachtet.
- **Ein Rückweg.** Jedes Release lässt sich mit demselben Skript zurückrollen.
- **Zwei Barrieren gefunden und behoben.** Ein voller Lauf der Browser-Tests fand zwei Mängel in der Verwaltung des Owners: Das Änderungsprotokoll lief seitwärts aus dem Bild (WCAG 2.1.1), und die Liste der Seiten eines Titelbilds stand zu eng (WCAG 2.5.8). Beide sind behoben; Gäste waren nicht betroffen.

## AI Agents im Produkt

Dieselbe Arbeitsweise läuft in cvitea selbst. AI Agents verbinden sich über das Model Context Protocol: Claude Desktop und claude.ai mit einer OAuth-Anmeldung, Agents auf der Kommandozeile mit einem persönlichen Token. Stand Release 46:

- 11 Werkzeuge: sieben lesen Laufbahndaten, Dokumente, Optionen, Upload-Status und Vorschau-Links; vier schreiben Vorschläge oder starten Uploads.
- 9 Berechtigungsbereiche: einer für alles und acht engere nach Thema und Aktion, damit ein persönliches Token nur erreicht, was seine Aufgabe braucht. Claude Desktop und claude.ai melden sich mit dem Bereich für alles an; den Bereich für Vorschau-Links muss der Owner eigens vergeben.
- Ein Vorschau-Link gilt höchstens zehn Minuten und zeigt die Gästeansicht einer Bewerbung, ohne als Aufruf zu zählen.
- Limits je Token: 60 Aufrufe pro Minute und 2’000 pro Tag, 200 wartende Vorschläge zugleich, 64 KB je Vorschlag.
- Jedes schreibende Werkzeug erzeugt einen Vorschlag oder legt einen Upload privat ab; live geht nichts ohne die Freigabe des Owners. Werkzeugaufrufe, die an Herkunft oder Berechtigungsbereich scheitern, werden protokolliert.
- Eine hochgeladene Datei erscheint nirgends, bis der Owner einen Vorschlag annimmt, der sie verwendet.
- Die Plattform hält Personendaten in ihrer Datenbank und ihrem Speicher auf Schweizer Webhosting; dieses öffentliche Repository enthält keine. AI Agents, die der Owner anbindet, erreichen die Laufbahndaten und Dokumente des Owners im Rahmen ihrer Berechtigungsbereiche; der MCP-Server der Plattform bietet keinen Zugriff auf Einladungen, Aufrufe oder Nachrichten von Gästen.
