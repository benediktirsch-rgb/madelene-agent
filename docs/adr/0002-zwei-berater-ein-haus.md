# ADR 0002 — Zwei Berater, ein Haus

**Datum:** 11.09.2026 · **Status:** Vorschlag, noch nicht gebaut

## Lage

John hat seit dem 10./11.09.2026 eine Architektur, die ihn wach hält (Repo `john-agent`): eine **Rezeption**
im WWW (`hotel-vaikuntha.de/john/`), einen **Worker** je Gerät mit eigenem Takt, eine **Lobby** im Compass.
Madeleine hat nichts davon. Sie denkt nur, wenn jemand fragt, und ihre Briefkästen werden alle zehn Minuten
geleert. Fällt der Cockpit-Server aus, ist sie genauso weg, wie John es war.

## Vorschlag

Madeleine zieht ins selbe Haus, bekommt aber **kein** zweites. Das Protokoll der Rezeption kennt schon Geräte,
Aufträge und Ergebnisse; es braucht nur eine Art mehr:

1. **Aufträge der Art `madeleine`** in Johns Rezeption. Die heutigen Briefkästen (Raumschiff, Compass-Tür)
   bleiben, wo sie sind, weil dort die Sichtgrenzen stehen. Neue Wege gehen über die Rezeption.
2. **Der Worker ruft Codex**, wie er heute Claude ruft: als abgekoppelter Kindprozess, nie in der Schleife.
   Damit ist Madeleine erreichbar, auch wenn der Cockpit-Server gerade denkt oder hängt.
3. **Kein eigener Takt für Madeleine.** Zahlen ändern sich täglich, nicht halbstündlich. Ihr Rhythmus ist
   der Monat: nach dem Finanzlauf (am 5.) ein Blick aufs Gesamtbild, als Punkt auf Johns Stapel. John führt,
   sie rechnet gegen, genau wie in der Beraterrunde.
4. **Die Rezeption bleibt dumm** (john-agent, ADR 0002): kein Kontostand, keine Vertragszeile im Netz. Ein
   Madeleine-Auftrag nennt das Thema; die Antwort mit Zahlen bleibt am Gerät, ins Zimmer kommt nur „Antwort
   liegt am Rechner".

## Warum nicht ein eigenes Haus

Zwei Rezeptionen, zwei Worker, zwei Lobbys hießen doppelter Betrieb für einen Menschen und ein Gerät. Das Haus
steht, und die Zimmer sind billig.

## Offen

Ob der Monatsblick überhaupt gewünscht ist, entscheidet Bene. Bis dahin bleibt es bei Punkt 1 und 2.
