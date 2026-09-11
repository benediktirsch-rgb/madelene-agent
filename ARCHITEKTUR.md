# Madeleine — Architektur (Stand 11.09.2026)

> Bene, 07.09.2026: „2 Berater (John und Madeleine), die unterschiedliche Stärken haben und im Idealfall
> auch transparent miteinander kommunizieren."

## Womit sie denkt

GPT über die **Codex CLI** (`codex exec`), angemeldet mit Benes ChatGPT-Konto, ohne API-Schlüssel und
ohne zweite Rechnung. Die Anmeldung muss als Datei vorliegen (`~\.codex\auth.json`,
`cli_auth_credentials_store = "file"`), sonst sieht der Server-Prozess sie nicht. Beim Aufruf wird
`OPENAI_API_KEY` bewusst aus der Umgebung genommen; sonst liefe die Abrechnung doch über die API.

Codex kennt keine Systemprompt-Datei. Persona, Wissen und Frage gehen deshalb als **ein** Text über stdin.
Werkzeuge hat sie keine (kein MCP über Codex). Will sie etwas festhalten, schreibt sie als letzte Zeile
`NOTIZ: …`, und der Server legt den Satz in ihre Notizen. So bleibt es sichtbar und nachlesbar.

## Wo sie heute läuft

```
   Compass (Karte + Chat)          Finanz-Raumschiff             Compass am Handy / Martin
   compass-madeleine.js            raumschiff/madeleine.php      gate.php?briefkasten=…
          │ POST /api/madeleine            │ Briefkasten                 │ Briefkasten
          ▼                                ▼                             ▼
   ┌─────────────────────────────────────────────────────────────────────────────┐
   │ Benes Rechner                                                               │
   │   john-server.ps1  ── dot-sourced ──►  john-madeleine.ps1  ──► codex exec  │
   │   madeleine-abholen.ps1 (Aufgabe alle 10 Min): Briefkästen leeren,         │
   │   antworten lassen, Antwort zurücktragen                                    │
   │   Wissen: C:\dev\madeleine  (lokal, nie im Repo)                            │
   └─────────────────────────────────────────────────────────────────────────────┘
```

- **Direkt:** Karte „👩‍💼 Madeleine" und Chat im Compass → `POST /api/madeleine` am Cockpit-Server.
- **Briefkästen:** Wer nicht an Benes Rechner sitzt (Bene am Handy, Martin an seinem Rechner), fragt über einen
  Briefkasten. Bis zu zehn Minuten später steht die Antwort dort. Zwei Kästen, ein Abholer. Die Sichtgrenze,
  wer welche Zahl sehen darf, steht **serverseitig** an der Stelle, die das Lagebild baut, nie im Prompt allein.
- **Beraterrunde:** John eröffnet (Coach-Sicht, eine Frage), Madeleine antwortet mit Zahlen und darf
  widersprechen, John schließt mit einer Ja/Nein-Entscheidung. Alles in einer Datei, die beide bei jedem
  Aufruf lesen.

## Was sie beim Denken bekommt

Ihre Persona und ihre Wissensdateien (lokal), Johns Coaching-Ordner, die offenen Rückfragen und
Entscheidungen des Compass, die Live-Zahlen aus Finanzlauf und Verein. **Nicht:** Claude-Memory,
CLAUDE.md-Regeln, Zugangsdaten.

## Was sie mit John teilt, und was nicht

| | John | Madeleine |
|---|---|---|
| Modell | Claude (Claude Code, Abo) | GPT (Codex CLI, Abo) |
| Feld | Karriere, Prioritäten, Entscheidungen | Finanzen, Steuern, Organisation |
| Eigener Takt | ja (Worker, john-agent) | noch nicht |
| Im WWW erreichbar | Rezeption `hotel-vaikuntha.de/john` | nur über Briefkästen |
| Lobby bei Ausfall | ja | nein — ihr Chat sagt es selbst |

Die zwei unteren Zeilen sind die Lücke, die dieses Repo schließen soll (siehe `docs/stand.md`).

## Die Regeln, an denen nicht gedreht wird

1. **Kein Inhalt in diesem Repo.** Siehe ADR 0001. Der Hook `.githooks/pre-commit` prüft es.
2. **Sichtgrenzen stehen serverseitig.** Was eine Person nicht sehen darf, kommt gar nicht erst ins Lagebild.
3. **Nichts versenden, nichts buchen, nichts kündigen.** Madeleine bereitet vor, Bene entscheidet.
4. **Erfinde nichts.** Fehlt eine Zahl, sagt sie das und nennt das Stichdatum ihrer Quelle.
