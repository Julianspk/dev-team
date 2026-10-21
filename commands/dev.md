---
description: Planer, Entwickler, Tester und QS auf eine Aufgabe ansetzen, mit Rückfragen untereinander
argument-hint: <Aufgabe, z.B. Bearbeite PROJ-42>
---

Arbeite diese Aufgabe mit dem Agenten-Team ab: $ARGUMENTS

Ist oben keine Aufgabe angegeben, frag nach ihr und starte nichts.

Du koordinierst und stellst Nachrichten zwischen den Agenten zu, die Arbeit selbst machen die Agenten. Es arbeitet immer nur ein Agent: warte auf sein Ergebnis, bevor du weitermachst.

Starten und fortsetzen:
- Starte jeden Agenten nur einmal mit dem Agent-Tool, gib kein model an (der Modell-Router wählt es) und merk dir seine Agent-ID.
- Beim Start kennt ein Agent den Verlauf nicht: gib ihm die Aufgabe und die vollständigen Ergebnisse mit, die er braucht.
- Danach sprichst du ihn nur noch per SendMessage an seine Agent-ID an. Er behält dabei seinen Kontext, schick ihm nur das Neue.

Rückfragen:
- Ein Agent rät nicht. Fehlt ihm etwas, beendet er seine Antwort mit je einer Zeile `FRAGE AN PLANER: ...` (ebenso ENTWICKLER, TESTER, QS oder NUTZER).
- Stell jede Frage per SendMessage dem genannten Agenten zu und nenne den Absender, z.B. "Rückfrage vom Tester: ...". Seine Antwort schickst du per SendMessage an den Fragenden zurück, der dann weiterarbeitet.
- `FRAGE AN NUTZER` stellst du mir. Meine Antwort geht an den Fragenden.
- Nach fünf Rückfragen zwischen den Agenten in einem Lauf gehen weitere direkt an mich.

Ablauf:
1. dev-team:planer: Aufgabe klären (ein Ticket wie PROJ-42 aus Jira lesen), Code untersuchen, Plan schreiben. Der fertige Plan geht an den Entwickler.
2. dev-team:entwickler: Plan umsetzen.
3. dev-team:tester: Umsetzung ausführen und testen.
4. dev-team:qs: Änderungen gegen Aufgabe, Plan und Projektregeln prüfen.

Meldet der Tester MÄNGEL oder die QS MÄNGEL, schick die Mängelliste per SendMessage an den Entwickler, danach seinen Bericht per SendMessage an Tester und QS. Höchstens zwei solche Runden; bleiben dann Mängel, hör auf und nenne sie.

Zum Schluss eine kurze Zusammenfassung: was geändert wurde, Testergebnis, QS-Urteil, Rückfragen mit Antwort, offene Punkte.
