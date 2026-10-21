# dev-team

Ein Plugin für Claude Code, das eine Aufgabe nacheinander an vier Agenten gibt: Planer, Entwickler, Tester und QS. Du schreibst `/dev Bearbeite EVT-560`, das Team liest das Ticket, setzt es um, testet und prüft es.

## Voraussetzungen

- Claude Code mit Plugin-Unterstützung. Validiert wurde das Plugin mit `claude plugin validate` unter Claude Code 2.1.304.
- Für Jira-Tickets wie `EVT-560`: ein verbundener Atlassian-MCP-Server. Ohne ihn kann der Planer das Ticket nicht lesen. Beschreib die Aufgabe dann im Befehl selbst.
- Optional: das Plugin `model-router`. Es wählt für jeden Agenten das passende Modell (siehe unten). Ohne den Router laufen alle Agenten auf dem Modell deiner Session.

## Installation

Im Prompt einer Terminal-Session:

```
/plugin install dev-team --marketplace Julianspk/dev-team
```

Aus einer Shell:

```
claude plugin install dev-team --marketplace Julianspk/dev-team
```

In der Desktop-App, wo `/plugin` nicht läuft: auf + neben dem Prompt-Feld klicken, dann Plugins, Add marketplace mit `Julianspk/dev-team`, dann Add plugin.

Ablauf bei der Installation:

1. Claude Code fragt `Add marketplace?` und nennt `github:Julianspk/dev-team`. Mit `y` bestätigen.
2. Einen Gültigkeitsbereich wählen, zum Beispiel den für deinen Benutzer.
3. Danach steht `Installed dev-team. Plugin is now active.` Das Plugin ist sofort aktiv.

Aus einem lokalen Ordner, zum Beispiel zum Entwickeln:

```
claude plugin marketplace add <Ordner>
claude plugin install dev-team@dev-team --scope user
```

Prüfen, ob das Plugin läuft: `/dev` ohne Aufgabe eingeben. Claude fragt dann nach der Aufgabe und startet nichts.

## Benutzung

Schreib den Befehl mit der Aufgabe dahinter:

```
/dev Bearbeite EVT-560
/dev Baue einen Filter für die Eventliste, nur Events der nächsten 7 Tage
```

Die Aufgabe kann ein Jira-Ticket sein oder ein Satz Freitext. Das Team arbeitet in dem Projekt, in dem die Session läuft.

### Die Agenten

| Agent | Aufgabe | Ändert Dateien | Ergebnis |
| --- | --- | --- | --- |
| `dev-team:planer` | Ticket lesen, Code untersuchen, Plan schreiben | nein | Plan mit Dateien, Schritten, Prüfung, Risiken und offenen Fragen |
| `dev-team:entwickler` | Plan oder Mängelliste umsetzen | ja, kein Commit | geänderte Dateien, Abweichungen vom Plan |
| `dev-team:tester` | Tests, Typecheck und Lint ausführen, Tests ergänzen | nur Tests | `ERGEBNIS: BESTANDEN` oder `ERGEBNIS: MÄNGEL` |
| `dev-team:qs` | Diff gegen Aufgabe, Plan und Projektregeln prüfen | nein | `URTEIL: FREIGABE` oder `URTEIL: MÄNGEL` |

### Befehle

| Eingabe | Wirkung |
| --- | --- |
| `/dev <Aufgabe>` | startet das Team auf die Aufgabe |
| `/dev-team:dev <Aufgabe>` | derselbe Befehl mit vollem Namen, falls ein anderes Plugin `/dev` belegt |

### Befehl umbenennen oder kürzen

Der Name des Befehls ist der Dateiname in `commands/`. Aus `commands/dev.md` wird mit `commands/entwickeln.md` der Befehl `/entwickeln`. Danach `/reload-plugins` ausführen.

## Wie das Team arbeitet

1. Das Hauptmodell koordiniert nur. Es startet die Agenten strikt nacheinander und gibt jedem die Aufgabe und die vollständigen Ergebnisse der vorherigen Agenten mit, weil ein Agent den Verlauf nicht kennt.
2. Der Planer klärt die Aufgabe und schreibt den Plan. Hat er offene Fragen, die die Umsetzung blockieren, hält das Team an und stellt sie dir.
3. Der Entwickler setzt den Plan um und prüft mit Typecheck, Lint und Build, soweit das Projekt sie hat.
4. Der Tester führt die vorhandenen Tests aus und ergänzt neue für das neue Verhalten.
5. Die QS prüft den Diff auf Akzeptanzkriterien, Fehler, Sicherheit, unnötige Komplexität und Änderungen, die nicht zur Aufgabe gehören.
6. Melden Tester oder QS Mängel, geht die Liste zurück an den Entwickler, danach wieder an Tester und QS. Höchstens zwei solche Runden. Bleiben dann Mängel, hält das Team an und nennt sie.
7. Am Ende steht eine kurze Zusammenfassung: was geändert wurde, Testergebnis, QS-Urteil, offene Punkte.

### Zusammenspiel mit dem Modell-Router

Die Agenten haben bewusst kein festes Modell. Ist das Plugin `model-router` installiert, stuft es jeden Start eines Agenten selbst ein: ein Planer für eine unklare Aufgabe läuft auf Opus, ein Tester, der nur Tests startet, auf Sonnet oder Haiku. Der Router stuft nur herab. Das Modell deiner Session bleibt die Obergrenze.

Willst du für einen Agenten ein festes Modell, trag `model: sonnet` (oder `haiku`, `opus`) in den Kopf seiner Datei unter `agents/` ein.

Die laufenden Agenten und ihren Fortschritt zeigt dir das Plugin `agenten-monitor`, falls du es installiert hast.

## Anpassen

Es gibt keine Optionen. Angepasst wird in den Dateien:

| Datei | Inhalt |
| --- | --- |
| `commands/dev.md` | Ablauf, Reihenfolge, Zahl der Mängelrunden, Form der Zusammenfassung |
| `agents/planer.md` | was der Plan enthalten muss |
| `agents/entwickler.md` | Regeln für die Umsetzung, zum Beispiel Commit-Verhalten |
| `agents/tester.md` | was getestet und wie das Ergebnis gemeldet wird |
| `agents/qs.md` | die Prüfpunkte der QS |

Nach einer Änderung `/reload-plugins` ausführen.

## Grenzen

- Die Reihenfolge und die Mängelschleife stehen als Anweisung im Befehl. Das Hauptmodell hält sich in der Regel daran, erzwungen wird es nicht.
- Dass Planer, Tester und QS keinen Produktivcode ändern und der Entwickler nicht committet, steht in ihren Anweisungen. Es ist keine technische Rechtesperre, alle Agenten haben alle Werkzeuge.
- Jedes Ergebnis geht komplett an den nächsten Agenten. Bei großen Aufgaben füllt das den Kontext schnell.
- Ein Lauf mit mehreren Agenten und Mängelrunden kostet deutlich mehr als ein einzelner Prompt.
- Die QS ist ein Modellurteil und ersetzt kein menschliches Review.

## Entwicklung

Im Ordner des Plugins, mit Claude Code 2.1.304:

```
claude plugin validate .
```

Der Ablauf steht in `commands/dev.md`, die vier Agenten stehen in `agents/`.
