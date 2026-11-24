# Verbindlicher Ablauf bei jedem "weiter"/Start/Reset (Menschen und Modelle)
Einzige Quelle der Wahrheit. `export TASK_WHO=<dein eindeutiger Name>`.

## 1. Stand prüfen und sichten
1. `git pull --ff-only origin main`, `python3 tools/tasks.py status`, `python3 tools/tasks.py board` (wer baut was?). OFFLINE/Fehler: abbrechen und melden.
2. Offene Ideen (`tasks.py ideas`, Ordner `BRAINSTORM/`): jede entscheiden, eigene mit `tasks.py idea "Text"` eintragen.
3. Werkzeuge: `python3 tools/cli_scout.py install` (alle `aktiv` im Register `docs/CLI_TOOLS.md`).
4. `tools/run_tests.sh` auf main. Rot: nur reparieren (Karte `fix-…`), nichts Neues beginnen.
5. Offene Pull Requests ansehen (`gh pr list`). Überschneidung mit deinen Dateien: melden statt doppelt bauen.

## 2. Was ist dran?
1. `python3 tools/tasks.py mine`: begonnene Karte zuerst weitermachen (`git fetch`, Branch `task/<ID>/<WER>`, vorhandene Commits lesen).
2. Sonst `python3 tools/tasks.py batch` (oder `next`). NONE: sofort beenden mit einer Zeile. Mehrere Karten (3 bis 4 kleine, unabhängige) nur bei reichlich Budget, sonst genau eine; nie auf Vorrat beanspruchen.
3. `python3 tools/tasks.py claim <ID>` für jede gewählte Karte, **vor** der ersten Änderung. Nur bei `CLAIMED` arbeiten. `BELEGT`: nächste nehmen. Das Repo ist öffentlich: nie Geheimnisse oder private Daten committen, vor dem Push den Scan aus dem Skill ausführen.
4. CLI-Check pro Karte: 2 bis 5 englische Stichworte, `python3 tools/cli_scout.py scout <Stichworte>`. Sinnvolle CLI per 3-Stimmen-Entscheidung mit `register ... aktiv` eintragen und nutzen, `docs/CLI_TOOLS.md` im selben Pull Request abgeben.

## 3. Arbeiten und abgeben
Nur Dateien der Karte ändern. Nach jedem kleinen Schritt committen und pushen. Tests grün + Beleg zu "Fertig, wenn". `tasks.py done <ID>`, Zeile in PLAN.md, Pull Request mit Beleg. Nicht selbst nach main mergen, außer der Besitzer-Rolle A. Danach wieder bei 1.

## 4. Entscheidungen: immer 3 Stimmen
Jede echte Entscheidung: eigene Empfehlung zuerst aufschreiben, dann 2 Agenten unabhängig mit derselben Frage befragen (ohne deine Empfehlung), Mehrheit sofort umsetzen, "Entscheidung: X (2:1, Gründe)" in PLAN.md. Den Menschen nicht fragen. Ausgenommen: Sicherheit, abgelehnte Aktionen, Push auf main ohne PR, Löschen fremder Branches, Geld, private Daten veröffentlichen.

## 5. Limit / wenig Token
Nichts Neues beanspruchen, committen, pushen, Stand in PLAN.md, dann Aufgabe mit Zeitpunkt nach dem Reset (Wächter alle 30 min / send_later / auto_continue.py). Nicht mehr begonnene Karten mit `tasks.py release <ID>` freigeben. Verwaiste Claims (>48 h ohne Commit/PR): Besitzer A prüft und gibt mit `tasks.py release <ID> <WER>` frei.

## 6. Ehrlichkeit
Ungetestetes als ungetestet kennzeichnen (🟢 🟡 🔴 ❌).
