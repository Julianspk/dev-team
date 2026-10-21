# dev-team

Ein Plugin für Claude Code, das eine Aufgabe nacheinander an vier Agenten gibt: Planer, Entwickler, Tester und QS. Du schreibst `/dev Bearbeite PROJ-42`, das Team liest das Ticket, setzt es um, testet und prüft es. Unterwegs fragen sich die Agenten gegenseitig, statt zu raten: Entwickler, Tester und QS fragen den Planer, der Planer fragt dich.

## Voraussetzungen

- Claude Code mit Plugin-Unterstützung. Validiert wurde das Plugin mit `claude plugin validate` unter Claude Code 2.1.304.
- Für Jira-Tickets wie `PROJ-42`: ein verbundener Atlassian-MCP-Server. Ohne ihn kann der Planer das Ticket nicht lesen. Beschreib die Aufgabe dann im Befehl selbst.
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
/dev Bearbeite PROJ-42
/dev Baue einen Filter für die Eventliste, nur Events der nächsten 7 Tage
```

Die Aufgabe kann ein Jira-Ticket sein oder ein Satz Freitext. Das Team arbeitet in dem Projekt, in dem die Session läuft.

### Die Agenten

| Agent | Aufgabe | Ändert Dateien | Ergebnis |
| --- | --- | --- | --- |
| `dev-team:planer` | Ticket lesen, Code untersuchen, Plan schreiben, Rückfragen des Teams beantworten | nein | Plan mit Dateien, Schritten, Prüfung und Risiken |
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

1. Das Hauptmodell koordiniert und stellt Nachrichten zu. Es startet jeden Agenten einmal, merkt sich seine Agent-ID und spricht ihn danach per `SendMessage` an. So behält jeder Agent seinen Kontext über den ganzen Lauf.
2. Der Planer klärt die Aufgabe und schreibt den Plan. Was er nicht klären kann, fragt er dich. Der fertige Plan geht an den Entwickler.
3. Der Entwickler setzt den Plan um und prüft mit Typecheck, Lint und Build, soweit das Projekt sie hat.
4. Der Tester führt die vorhandenen Tests aus und ergänzt neue für das neue Verhalten.
5. Die QS prüft den Diff auf Akzeptanzkriterien, Fehler, Sicherheit, unnötige Komplexität und Änderungen, die nicht zur Aufgabe gehören.
6. Melden Tester oder QS Mängel, geht die Liste an denselben Entwickler, danach sein Bericht an denselben Tester und dieselbe QS. Höchstens zwei solche Runden. Bleiben dann Mängel, hält das Team an und nennt sie.
7. Am Ende steht eine kurze Zusammenfassung: was geändert wurde, Testergebnis, QS-Urteil, Rückfragen mit Antwort, offene Punkte.

### Rückfragen im Team

Ein Agent, dem etwas fehlt, beendet seine Antwort mit einer Zeile wie `FRAGE AN PLANER: Soll der Filter auch vergangene Events ausblenden?`. Das Hauptmodell stellt die Frage dem Planer zu, schickt dessen Antwort zurück, und der Fragende arbeitet weiter.

| Wer | fragt wen | wozu |
| --- | --- | --- |
| Planer | dich (`NUTZER`) | was Ticket und Code nicht klären |
| Entwickler | Planer, Tester, QS | unklarer Plan, unklarer Mangel |
| Tester | Planer, Entwickler | erwartetes Verhalten, Grund einer Änderung |
| QS | Planer, Entwickler | Absicht der Aufgabe, Grund einer Änderung |

Kann der Planer eine Rückfrage nicht beantworten, gibt er sie an dich weiter. Nach fünf Rückfragen zwischen den Agenten in einem Lauf gehen weitere direkt an dich.

### Zusammenspiel mit dem Modell-Router

Die Agenten haben bewusst kein festes Modell. Ist das Plugin `model-router` installiert, stuft es jeden Start eines Agenten selbst ein: ein Planer für eine unklare Aufgabe läuft auf Opus, ein Tester, der nur Tests startet, auf Sonnet oder Haiku. Der Router stuft nur herab. Das Modell deiner Session bleibt die Obergrenze.

Willst du für einen Agenten ein festes Modell, trag `model: sonnet` (oder `haiku`, `opus`) in den Kopf seiner Datei unter `agents/` ein.

Die laufenden Agenten und ihren Fortschritt zeigt dir das Plugin `agenten-monitor`, falls du es installiert hast.

## Anpassen

Es gibt keine Optionen. Angepasst wird in den Dateien:

| Datei | Inhalt |
| --- | --- |
| `commands/dev.md` | Ablauf, Reihenfolge, Zustellung der Rückfragen, Zahl der Mängelrunden und Rückfragen, Form der Zusammenfassung |
| `agents/planer.md` | was der Plan enthalten muss, wie Rückfragen beantwortet werden |
| `agents/entwickler.md` | Regeln für die Umsetzung, zum Beispiel Commit-Verhalten, wen er fragt |
| `agents/tester.md` | was getestet und wie das Ergebnis gemeldet wird, wen er fragt |
| `agents/qs.md` | die Prüfpunkte der QS, wen sie fragt |

Nach einer Änderung `/reload-plugins` ausführen.

## Grenzen

- Die Reihenfolge und die Mängelschleife stehen als Anweisung im Befehl. Das Hauptmodell hält sich in der Regel daran, erzwungen wird es nicht.
- Dass Planer, Tester und QS keinen Produktivcode ändern und der Entwickler nicht committet, steht in ihren Anweisungen. Es ist keine technische Rechtesperre, alle Agenten haben alle Werkzeuge.
- Beim ersten Start bekommt jeder Agent die vollständigen Ergebnisse der vorherigen mit. Bei großen Aufgaben füllt das den Kontext schnell.
- Rückfragen laufen über das Hauptmodell, nicht direkt von Agent zu Agent. Ein Agent kann nicht mitten in der Arbeit auf eine Antwort warten: er beendet seinen Durchgang mit der Frage und wird mit der Antwort fortgesetzt.
- Das Fortsetzen per `SendMessage` klappt nur innerhalb derselben Session. Nach `/clear` oder einem Neustart beginnt das Team von vorn.
- Ein Lauf mit mehreren Agenten und Mängelrunden kostet deutlich mehr als ein einzelner Prompt.
- Die QS ist ein Modellurteil und ersetzt kein menschliches Review.

## Entwicklung

Im Ordner des Plugins, mit Claude Code 2.1.304:

```
claude plugin validate .
```

Der Ablauf steht in `commands/dev.md`, die vier Agenten stehen in `agents/`.
