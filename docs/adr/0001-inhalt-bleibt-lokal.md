# ADR 0001 — Madeleines Inhalt bleibt lokal

**Datum:** 11.09.2026 · **Status:** angenommen

## Lage

Bene hat am 11.09.2026 das Repo `benediktirsch-rgb/madelene-agent` angelegt, als Schwester von
`john-agent`. Beide sind **öffentlich** (geprüft über die GitHub-API ohne Anmeldung: `private: false`).

Madeleines Arbeitsstoff ist das Gegenteil von öffentlich: Kontostände, Darlehen und Restschulden,
Vorsorge, Verträge, Gehälter, das Steuerbüro, Kundschaft und Zahlungsverhalten, Entscheidungen über
Geldflüsse zwischen GmbH, privat und Verein.

## Entscheidung

Dieses Repo enthält **nur**, wie Madeleine arbeitet: Architektur, Entscheidungen, ihre Rolle in allgemeinen
Worten, Grenzen, Pläne. Ihr Inhalt bleibt in `C:\dev\madeleine`, einem Ordner **ohne Git**:

| bleibt lokal | warum |
|---|---|
| `CLAUDE.md` (Persona im Wortlaut) | nennt Steuerbüro, Kundschaft, Entscheidungsnummern mit Beträgen, die Vorsorgelage |
| `wissen\*.md` | Zahlen und Verträge zu GmbH, Verein und privat |
| `privat\stand.json`, `privat\aktualisieren.ps1` | Kontostände, erzeugt aus Kontoauszügen |
| `notizen\beratung.md` | was sie aus Gesprächen festhält |

`wissen/rolle.md` in diesem Repo beschreibt ihre Rolle so, dass ein fremder Mensch sie lesen darf. Die
arbeitende Fassung ist und bleibt die lokale Persona.

## Durchgesetzt durch

- `.gitignore` schließt `privat/`, `notizen/`, `*.json` außer Konfigurationsbeispielen und jede Datei mit
  „privat" im Namen aus.
- `.githooks/pre-commit` bricht ab bei Beträgen in Euro, IBAN-Mustern, Dateinamen wie `stand.json` und bei
  einer Liste von Namen, die nur in Madeleines Inhalt vorkommen. Übersteuern nur bewusst (`ALLOW_INHALT=1`)
  und nur, wenn der Treffer nachweislich kein Inhalt ist.

## Wenn das Repo privat wird

Dann ändert sich an dieser Entscheidung zunächst nichts. Ein privates Repo ist nur eine Einstellung
entfernt von einem öffentlichen, und eine einmal gepushte Zahl bleibt in der Historie. Wer Madeleines
Inhalt versionieren will, braucht ein **eigenes** privates Repo nur dafür und eine neue Entscheidung hier.
