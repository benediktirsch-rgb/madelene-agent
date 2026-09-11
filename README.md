# madelene-agent

Madeleines Architektur: die Teile, mit denen Benes zweite Beraterin arbeitet. Finanzen, Steuern und
Organisation für GmbH, Verein und privat. Angelegt am 11.09.2026, Schwester-Repo von
[`john-agent`](https://github.com/benediktirsch-rgb/john-agent).

> **Dieses Repo ist öffentlich.** Madeleines Arbeit besteht zu großen Teilen aus Zahlen, die niemanden
> etwas angehen: Kontostände, Darlehen, Verträge, Gehälter. Nichts davon liegt hier und nichts davon
> kommt je hierher. Hier steht, **wie** sie arbeitet, nicht **was** sie weiß. Die Grenze steht in
> [`docs/adr/0001-inhalt-bleibt-lokal.md`](docs/adr/0001-inhalt-bleibt-lokal.md), und ein Hook prüft
> sie vor jedem Commit.

## Lies zuerst

| Frage | Datei |
|---|---|
| Wie läuft Madeleine heute? | [`ARCHITEKTUR.md`](ARCHITEKTUR.md) |
| Was darf hier nie hinein? | [`docs/adr/0001-inhalt-bleibt-lokal.md`](docs/adr/0001-inhalt-bleibt-lokal.md) |
| Wie spielen John und Madeleine zusammen? | [`docs/adr/0002-zwei-berater-ein-haus.md`](docs/adr/0002-zwei-berater-ein-haus.md) |
| Wozu ist sie da, und wo endet ihr Auftrag? | [`wissen/rolle.md`](wissen/rolle.md) |
| Was steht noch aus? | [`docs/stand.md`](docs/stand.md) |

## Drei Orte, klar getrennt

| Ort | Inhalt | öffentlich? |
|---|---|---|
| **dieses Repo** | Architektur, Entscheidungen, Rolle, Grenzen | ja |
| `C:\dev\madeleine` (lokal, ohne Git) | Persona im Wortlaut, Wissen mit Zahlen, Notizen, privater Kontostand | **nie** |
| Repo `flow-compass` | der Code, der sie heute aufruft (`john-madeleine.ps1`, `madeleine-abholen.ps1`, `compass-madeleine.js`) | ja |
