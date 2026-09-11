# Stand — was es gibt, was fehlt

## Es gibt (11.09.2026)

- Madeleine im Compass (Karte + Chat), im Finanz-Raumschiff und über die Compass-Tür, alles aus dem Repo
  `flow-compass` heraus (`john-madeleine.ps1`, `madeleine-abholen.ps1`, `compass-madeleine.js`).
- Beraterrunde mit John, Gegenprüfung der John-Architektur (`john-agent/beratung/protokoll.md`).
- Dieses Repo: Architektur, Grenzen (ADR 0001), Vorschlag fürs gemeinsame Haus (ADR 0002), öffentliche Rolle.

## Fehlt

1. **Ins gemeinsame Haus ziehen** (ADR 0002): Aufträge `madeleine` in Johns Rezeption, der Worker ruft Codex
   als Kindprozess. Dann ist sie erreichbar, auch wenn der Cockpit-Server hängt.
2. **Ihr Code gehört hierher.** Heute liegt er in `flow-compass`, weil der Cockpit-Server ihn per Dot-Sourcing
   lädt. Umzug wie bei Johns Lobby: Quelle hier, Kopie dort per Sync-Skript. An einem Tag ohne parallele
   Sessions in `flow-compass`.
3. **Monatsblick nach dem Finanzlauf**, falls Bene ihn will (ADR 0002, Punkt 3).
4. **Repo-Sichtbarkeit klären.** Öffentlich ist in Ordnung, solange ADR 0001 gilt. Wird es privat, bleibt ADR 0001
   trotzdem stehen.
