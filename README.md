# projekt-autopilot

Skill für Claude (Claude Code / claude.ai): macht jedes neue Projekt zu einem öffentlichen GitHub-Repo, zerlegt die Arbeit in claimbare Karten, entscheidet mit 3 Stimmen (Hauptmodell + 2 Agenten), nutzt passende [CLI-Anything](https://github.com/HKUDS/CLI-Anything)-CLIs und setzt nach einem Token-Limit automatisch fort.

## Inhalt
| Pfad | Zweck |
|---|---|
| `SKILL.md` | Anweisungen des Skills |
| `templates/AGENT_PROMPT.md` | Ablauf, den jeder Coder im Projekt befolgt |
| `templates/tools/tasks.py` | Karten, atomare Claims, `board`, `batch`, Ideen |
| `templates/tools/cli_scout.py` | CLI-Hub durchsuchen, Register `docs/CLI_TOOLS.md` pflegen |
| `templates/tools/auto_continue.py` | Terminal-Auto-Weiter nach Limit-Reset |

## Laufende Projekte
Der Skill lässt sich auch in ein bestehendes Repo einbinden (SKILL.md, Abschnitt E): ergänzt nur, verschiebt nichts, Pfade über `tools/autopilot.json`. Erstes Beispiel: `ClaudeAIDriven/sternenhausmeister` (PR #110).

## Installation
**Claude Code (persönlich):**
```bash
git clone https://github.com/ClaudeAIDriven/project-autopilot ~/.claude/skills/projekt-autopilot
```
**claude.ai:** `SKILL.md` und `templates/` als ZIP hochladen (Einstellungen → Fähigkeiten).

## Teststand
- 🟢 `tasks.py`: Claim inkl. Wettlauf zweier gleichzeitiger Versuche, Abhängigkeiten, `board`, `batch`, Ideen (gegen lokales Git-Remote)
- 🟢 `cli_scout.py`: `ensure`, `scout`, `register`, `list` (gegen echten `cli-hub`)
- 🟢 `auto_continue.py`: Limit-Erkennung mit Stub
- 🟡 GitHub-Issue-Markierung, `gh repo create`, Wächter-Trigger, `send_later`, `cli_scout.py install`
- 🔴 Format der echten Limit-Meldung (Rückfall: 5 h 5 min Wartezeit)

Lizenz: MIT
