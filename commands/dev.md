---
description: Planer, Entwickler, Tester und QS nacheinander auf eine Aufgabe ansetzen
argument-hint: <Aufgabe, z.B. Bearbeite PROJ-42>
---

Arbeite diese Aufgabe mit dem Agenten-Team ab: $ARGUMENTS

Ist oben keine Aufgabe angegeben, frag nach ihr und starte nichts.

Du koordinierst nur und machst die Arbeit nicht selbst. Starte die Agenten mit dem Agent-Tool strikt nacheinander und gib bei keinem ein model an, der Modell-Router wählt es. Die Agenten kennen diesen Verlauf nicht: gib jedem die Aufgabe und die vollständigen Ergebnisse der vorherigen Agenten mit.

1. dev-team:planer: Aufgabe klären (ein Ticket wie PROJ-42 aus Jira lesen), Code untersuchen, Plan schreiben.
2. dev-team:entwickler: Plan umsetzen.
3. dev-team:tester: Umsetzung ausführen und testen.
4. dev-team:qs: Änderungen gegen Aufgabe, Plan und Projektregeln prüfen.

Meldet der Planer offene Fragen, die die Umsetzung blockieren, hör nach Schritt 1 auf und stell sie mir.
Meldet der Tester MÄNGEL oder die QS MÄNGEL, geht die Mängelliste an dev-team:entwickler, danach wieder an dev-team:tester und dev-team:qs. Höchstens zwei solche Runden; bleiben dann Mängel, hör auf und nenne sie.

Zum Schluss eine kurze Zusammenfassung: was geändert wurde, Testergebnis, QS-Urteil, offene Punkte.
