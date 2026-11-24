---
name: projekt-autopilot
description: Bei Start jedes neuen Projekts automatisch öffentliches GitHub-Repo mit TASKS-, BRAINSTORM- und CLI-Register anlegen, Arbeit in claimbare Karten zerlegen, Entscheidungen immer mit 3 Stimmen (Hauptmodell + 2 Agenten) treffen, passende CLI-Anything-CLIs nutzen und Auto-Weiter nach Limit-Reset einrichten. Maximale Automatisierung, minimale Rückfragen.
---

# Projekt-Autopilot

Ziel: Jedes neue Projekt wird sofort zu einem GitHub-Repository. Die Arbeit liegt in kleinen Karten (`TASKS/`), Ideen in `BRAINSTORM/`, gemeinsame Werkzeuge in `docs/CLI_TOOLS.md`. Mehrere Coder (Menschen und Modelle) sichten das regelmäßig und beanspruchen Karten parallel. Der Mensch wird nur bei dem Wenigen gefragt, was kein Agent entscheiden darf. Nach einem Token-Limit läuft die Arbeit von selbst weiter.

Vorbild: `ClaudeAIDriven/sternenhausmeister`. Die Hilfsskripte liegen in diesem Skill unter `templates/tools/` und werden **unverändert** ins Projekt kopiert.

Ehrlichkeit: Ungetestetes immer markieren (🟢 getestet · 🟡 wahrscheinlich · 🔴 geraten · ❌ unbekannt).

## Regeln gegen Rückfragen
- Fehlende Angaben (Name, Sprache, Stack) selbst entscheiden (Schritt 4), in `PLAN.md` festhalten, weiterarbeiten.
- Standards: Repo-Name = Slug der Projektidee; Eigentümer = Organisation `ClaudeAIDriven`, wenn `gh` dort Schreibrecht hat, sonst eigener Account; Sichtbarkeit **öffentlich**; Lizenz MIT; Doku Deutsch, Code-Namen Englisch.
- Nur diese Fälle rechtfertigen eine Frage (dann **eine** gebündelte Nachricht, Rest läuft weiter): fehlende Anmeldung (`gh auth login`), Geld/Kosten, Löschen fremder Daten oder Branches, Veröffentlichen unter fremdem Namen, Sicherheits-/Berechtigungsfragen.

## Ablauf bei Projektstart

Lege zuerst eine Aufgabenliste mit diesen Schritten an und hake sie ab.

### 1. Repository anlegen (öffentlich, deshalb erst scannen)
Vor dem ersten Commit: `.env`, Schlüssel, Zugangsdaten, Kunden-, Mieter-, Finanz- und persönliche Daten kommen nie ins Repo (`.gitignore`, Beispielwerte als `*.example`). Scan, Treffer selbst bereinigen und in `PLAN.md` vermerken:
```bash
git grep -nIE "(ghp_|github_pat_|sk-[A-Za-z0-9]{20,}|AKIA[0-9A-Z]{16}|BEGIN [A-Z ]*PRIVATE KEY|password\s*[=:])" || true
gh auth status
git init -b main && printf '__pycache__/\nnode_modules/\nbuild/\n.env\n*.key\n' > .gitignore
git add -A && git commit -m "Projektstart"
gh repo create <OWNER>/<slug> --public --source . --remote origin --push
git remote set-head origin main
```
- Gibt es `mcp__claude-code-remote__add_repo`, danach `add_repo(owner, repo, access="push")`.
- `gh` nicht angemeldet: lokal weitermachen, am Ende EINE Meldung „bitte `gh auth login`“.
- Erster Push auf `main` nur im eigenen neuen Repo, danach alles per Branch + Pull Request. Vor jedem Push derselbe Scan.

### 2. Gerüst schreiben
`README.md`, `LICENSE` (MIT), `AGENT_PROMPT.md` (aus `templates/AGENT_PROMPT.md`), `CONTRIBUTING.md` (3 Zeilen, verweist auf AGENT_PROMPT.md), `PLAN.md` (Ziel, Entscheidungen, Fortschritt), `BRAINSTORM/INDEX.md` (per `python3 tools/tasks.py idea "Projektstart: erste Ideen"`), `docs/ARCHITECTURE.md` (Module und Schnittstellen), `docs/HANDOFF.md`, `docs/CLI_TOOLS.md` (legt `cli_scout.py` an), `tools/run_tests.sh` (muss mit `TESTS OK` enden), `.github/workflows/ci.yml` (führt `tools/run_tests.sh` bei Push und Pull Request aus) und `tools/tasks.py`, `tools/cli_scout.py`, `tools/auto_continue.py` aus `templates/tools/`.

### 3. Arbeit in Karten zerlegen
JSON-Liste schreiben, `python3 tools/tasks.py add < karten.json`, dann `python3 tools/tasks.py issues`.

Felder: `id` (`<BEREICH>-<NN>`), `area`, `title`, `deps`, `size` (S oder M), `prio` (`must`/`nice`), `owner` (A/B/C), `goal`, `files` (nur diese Dateien darf die Karte ändern), `accept` („Fertig, wenn“, durch Test oder Screenshot prüfbar).

- Eine Karte = eine Sitzung, ein Branch, ein Pull Request.
- Karten mit gemeinsamen Dateien bekommen eine Abhängigkeit; gemeinsame Dateien gehören einer Integrationskarte (`I-NN`).
- Zuerst `C-01` (Gerüst, startet und testet), danach möglichst viele unabhängige Karten.
- Schnittstellen vorab in `docs/ARCHITECTURE.md`. Letzte Karte immer: „Wächter entfernen“.

### 3b. Rhythmus für jeden Coder (Pflicht)
Bei jedem Start und nach jeder abgegebenen Karte:
1. `tasks.py status`: Stand, Claims, Startbares, offene Ideen.
2. `tasks.py board`: **wer baut gerade was** (Claim-Branches, Commits, Alter, offene PRs, `[VERWAIST?]` ab 48 h). `tasks.py list` zeigt `[IN ARBEIT: Name]`.
3. `tasks.py ideas`: jede Idee mit Status `neu` entscheiden (`umsetzen`/`später`/`abgelehnt` mit Grund). Eigene Ideen mit `tasks.py idea "Text"`.
4. `tools/cli_scout.py install`: alle `aktiv` gelisteten CLIs installieren (gleicher Werkzeugstand).

**Claim = Markierung:** `tasks.py claim <ID>` ist atomar (Schloss `lock/<ID>` mit einzigartigem Commit, zweiter gleichzeitiger Versuch wird abgelehnt), legt den sichtbaren Branch `task/<ID>/<WER>` an und setzt im Issue Label `in-progress` plus Kommentar. Nur bei `CLAIMED` arbeiten, bei `BELEGT` die nächste Karte. `release <ID>` gibt frei.

**Eine oder mehrere Karten?** `tasks.py batch [PUNKTE]` (S = 1, M = 2, Standard 3) liefert Karten ohne Dateiüberschneidung. 3 bis 4 kleine, unabhängige Karten nur bei reichlich Budget; bei M-Karte, unsicherem Budget oder vielen offenen PRs genau eine. Nie auf Vorrat beanspruchen, Unfertiges mit `release` freigeben.

`export TASK_WHO=<eindeutiger Name>` (z. B. `jan`, `bot-A`, `bot-waechter`).

### 3c. CLI-Anything: passende CLIs finden, nutzen, im Repo festhalten
CLI-Anything (https://github.com/HKUDS/CLI-Anything) macht Software für Agenten bedienbar (CLIs mit JSON-Ausgabe), dazu der Katalog CLI-Hub.
```bash
python3 tools/cli_scout.py ensure     # pip install cli-anything-hub
# optional in Claude Code (🟡 ungetestet): /plugin marketplace add HKUDS/CLI-Anything, /plugin install cli-anything
```
Bei jedem Kartenstart, nach jedem Reset und mindestens einmal pro Sitzung:
1. 2 bis 5 **englische** Stichworte aus der Karte, `python3 tools/cli_scout.py scout <Stichworte>` (zeigt nur noch nicht erfasste CLIs); zusätzlich `cli-hub can "<Aufgabe>"`, `cli-hub info <name>`.
2. Sinnvoll? Entscheidung mit 3 Stimmen, dann `cli_scout.py register <name> aktiv "Zweck" <Karte>`. Nicht sinnvoll: `abgelehnt` mit Grund eintragen.
3. Selbstständig nutzen: `cli-hub install <name>`, `cli-hub launch <name> ...` bzw. `cli-anything-<name> --json ...`. Steuert die CLI Desktop-Software, muss diese installiert sein. Fehlt eine CLI, kann `/cli-anything <Pfad oder Repo-URL>` eine erzeugen (eigene Karte, Größe M).
4. `docs/CLI_TOOLS.md` im selben Pull Request wie die Karte abgeben und in `PLAN.md` vermerken. So haben alle denselben Stand.

🟢 `ensure`, `scout`, `register`, `list` gegen den echten `cli-hub` getestet; 🟡 `install`; 🔴 ob eine CLI auf dem Rechner läuft, zeigt erst Installation und Probelauf.

### 4. Entscheidungsprotokoll: immer 3 Stimmen, Mehrheit gewinnt
Jede echte Entscheidung (mehr als eine vernünftige Option: Stack, Architektur, Bibliothek, CLI-Auswahl, Aufteilung) mit **drei unabhängigen Meinungen**: Hauptmodell + 2 Agenten. Triviales (Variablenname) ohne Abstimmung.
1. Eigene Empfehlung (2 Sätze) zuerst aufschreiben, **bevor** du die Agenten liest.
2. **Zwei Agenten parallel** starten, dieselbe Frage, dieselben Optionen, derselbe Kontext, ohne deine Empfehlung und ohne Kenntnis voneinander. Auftrag: „Empfehle genau eine Option. Format: `Empfehlung: <Option>`, höchstens 3 Zeilen Begründung, 1 Zeile Risiko.“
3. Mehrheit der 3 Stimmen gewinnt und wird **sofort umgesetzt**. Drei verschiedene Optionen: die am leichtesten umkehrbare, dann die einfachste.
4. In `PLAN.md`: `Entscheidung: X (2:1, Gründe; Gegenstimme: Y wegen …)`.
5. **Ausgenommen:** Sicherheitsfragen, abgelehnte Aktionen oder verweigerte Berechtigungen, Push auf `main` ohne Pull Request, Löschen fremder Branches/Daten, Geld, Veröffentlichen privater oder geheimer Daten. Eine Ablehnung wird nicht per Mehrheit umgangen.

### 5. Auto-Weiter nach Token-Limit
Den Tokenstand sieht ein Agent nicht zuverlässig 🔴, darum **absturzsicher statt vorausschauend**: jeden kleinen Schritt sofort committen und pushen, Sitzungen beginnen mit `tasks.py mine`.
1. **Wächter alle 30 Minuten** (immer beim Start): `mcp__claude-code-remote__create_trigger` mit `cron_expression` `7,37 * * * *` (bei Ablehnung kleinstes erlaubtes Intervall). Prompt: „Öffne Repo <OWNER>/<slug>, `export TASK_WHO=bot-waechter`. Prüfe zuerst nur `tasks.py mine` und `tasks.py next`. `mine` leer und `next` NONE: sofort beenden. Sonst AGENT_PROMPT.md abarbeiten.“ Trigger-ID in `PLAN.md`. 🟡 Ob die Umgebung das Repo bereitstellt, beim ersten Lauf prüfen.
2. **Diese Sitzung:** bei Limit mit Reset-Zeit erst committen und pushen, dann `send_later(at=<Reset + 2 min>, message="continue. Lies AGENT_PROMPT.md und arbeite den Ablauf weiter ab.")`; ohne Zeit `delay_minutes=305`. 🟡
3. **Terminal:** `python3 tools/auto_continue.py` (wartet bis zum Reset, macht weiter). 🔴 Format der echten Limit-Meldung ungeprüft, Rückfall 5 h 5 min. Berechtigungen vorher per Allow-Liste freigeben.

**Wenig Token übrig:** nichts Neues beanspruchen; laufende Karte sauber committen und pushen; Stand und nächsten Schritt in `PLAN.md`; Aufgabe mit Zeitpunkt nach dem Reset anlegen (`send_later` plus Wächter); dem Menschen höchstens eine Zeile („Limit, läuft um <Zeit> automatisch weiter“).

Ist alles `done`: Wächter mit `delete_trigger` entfernen.

### 6. Abschlussbericht (kurz)
Repo-Link, Anzahl Karten, Trigger-ID, eingetragene CLIs, was 🟢/🟡/🔴 ist, höchstens eine offene Entscheidung.
