# Übergabe an Astra — Gesprächsraum auf Rezeption und Worker

**Von:** Claude (Architektur john-agent) · **Für:** Astra (Codex in ChatGPT Work, Madeleines Rolle im gemeinsamen Konzept)
**Stand:** 11.09.2026, mittags · **Status:** Das Backend steht und ist am laufenden Gerät geprüft. Die Oberfläche fehlt — die ist deine Seite.

**Ziel:** ein gemeinsamer Gesprächsraum im Flow Compass, in dem Bene, John und Madeleine sprechen, gebaut auf der
bestehenden **Rezeption** (hotel-vaikuntha.de/john) und dem **Worker** auf Benes Rechner. Keine weitere Persona,
keine zweite Agentenarchitektur, keine neuen Orte für Wissen oder Schlüssel.

## 0. Wer was macht (Bene, 11.09.: „wir machen das jetzt von 2 Seiten")

| Seite | wer | Stand |
|---|---|---|
| **Backend:** Protokoll, Rezeption (Art `raum`, Status `gestoppt`, `w=stopp`), Tür des Workers (`/raeume`, `/raum`, `/stopp`), Madeleine und John im Worker | Claude | **läuft** (Worker 1.2.0, Rezeption live), Prüfstände 21/21 (Rezeption) und 29/29 (Tür, echte Antworten von John, Stopp mitten im Denken samt Modellprozess, Stopp vom Handy über die Rezeption) |
| **Oberfläche:** der Raum im Compass (Verlauf, Live, Eingabe, Stopp, „nicht weitergeben") | Astra | offen — dein Prototyp hat Verlauf, Live und Stopp schon; er wechselt nur den Transport |

**Voraussetzung, die gilt:** Im Raum antwortet Madeleine als **lokaler Worker-Auftrag** (Codex CLI auf Benes Rechner
mit ihrem lokalen Wissen). Astra baut und prüft den Raum, antwortet aber nicht selbst darin. Sollte Astra selbst im
Raum sprechen, müsste Gesprächsinhalt in Astras Umgebung fließen — das wäre eine eigene Entscheidung von Bene.

---

## 1. Was deine Oberfläche spricht: die Tür des Workers

Maßgeblich ist `john-agent/docs/protokoll.md` › „Gesprächsraum" › „Tür des Workers". Kurzfassung:

**Adresse:** `window.JOHN_TUER` oder `http://127.0.0.1:8788`. Nur am Rechner erreichbar — dort liegt der Text.

| Aufruf | Antwort |
|---|---|
| `GET /raeume` | `{ok, raeume:[{id, thema, zuege, zuletzt, laeuft, wartet}]}` |
| `GET /raum?id=<raum>&seit=<zug>` | `{ok, id, thema, zuege:[{zug, wer, zeit, text, weitergeben}], laeuft:{an, seit}\|null, wartet:[an…]}` |
| `POST /raum {id?, thema?, text, an:"john"\|"madeleine"\|"beide"}` | `{ok, id, zug, wartet}` — ohne `id` ein neuer Raum |
| `POST /raum/weitergeben {id, zug, weitergeben:false\|true}` | `{ok, id, zug, weitergeben}` |
| `POST /stopp {id}` | `{ok, gestoppt}` |

- `wer`: `bene` · `john` · `madeleine` · `system`. **`system`-Zeilen gehören in den Verlauf** (Stopp, Fehler,
  Abbruch nach 16 Min) — „leer heißt nie nichts".
- **Live:** alle 2 s `GET /raum?id&seit=<letzter zug>`. Die Tür antwortet in Millisekunden; sie denkt nie selbst.
- **„Denkt gerade":** `laeuft.an` + `laeuft.seit` (John ca. 10–20 s, Madeleine 20–60 s, bei „beide" nacheinander).
  `wartet` nennt, wer danach noch dran ist. `POST /raum` antwortet `wartet: true`, wenn John gerade etwas anderes
  denkt (sein Takt) — die Oberfläche sagt dann „wartet, John ist beschäftigt", nicht „hängt".
- **Fehlercodes:** 400 (Feld fehlt/ungültig), 404 (Raum/Zug unbekannt), 405 (Handlung per GET), 413 (Text über
  8000 Zeichen), 403 (fremde Herkunft).
- **Herkunft:** Die Tür antwortet nur eigenen Seiten (`https://bene.vishnuartists.com`, `localhost`/`127.0.0.1`
  mit beliebigem Port). Weitere Herkünfte trägt Bene als User-Variable `JOHN_TUER_ORIGINS` ein. Handlungen nur per `POST`.
- **Nicht erreichbar** (Rechner aus, Worker aus, Handy): die Lobby erkennt das schon (`compass-john-lobby.js`,
  `GET /stand`). Am Handy zeigt der Raum nur den Stand aus der Rezeption (Abschnitt 3) und kann stoppen.

**Entwickeln ohne Benes Rechner:** `john-agent/tools/tuer-attrappe.php` spricht denselben Vertrag mit gespielten
Antworten (je Sprecher 4 s „Denken", bei „beide" nacheinander; das Wort „fehler" im Text spielt einen Fehlschlag).

```
php -S 127.0.0.1:8788 tools/tuer-attrappe.php
```

## 2. Wo die Oberfläche hingehört

- **Eine eigene Datei, von außen angehängt** — so wie `compass-john-lobby.js`: Vorschlag
  `john-agent/compass/compass-gespraechsraum.js`. Sie liest `window.JOHN_TUER` (für die Rezeption: `JOHN_HUB` und
  `JOHN_HUB_TOKEN_BROWSER`, beide setzt der Build der eigenen Instanz) und hängt sich an eine Karte bzw. einen Knopf
  in Johns Kachel. `dashboard.html` bekommt nur ein `<script>`-Tag.
- **Nie in die Verkaufs-Demo:** `build-compass-produkt.ps1` nimmt John-Dateien heraus. Eine neue Datei muss dort in
  die Ausschlussliste — sonst landet der Raum in der Demo. Das spiele ich beim Einbau nach.
- **Kein eigener Transport:** nicht `/api/john`, nicht `/api/madeleine` am Cockpit-Server. Der ist seriell — ein
  Denkvorgang blockiert ihn 60–90 s, genau das war der Grund für die neue Architektur.
- Deine Änderungen kommen als **Branch oder Pull Request**. Ich prüfe gegen den laufenden Worker und baue ein.

## 3. Was die Rezeption dazu weiß (am Handy)

`GET https://hotel-vaikuntha.de/john/api.php?w=stand` mit Browser-Schlüssel liefert
`raeume:[{id, raum, thema, zug, an, status, erstellt, fertig}]` — die letzten 20 Raum-Züge, **ohne Text**.
Status: `laeuft` · `fertig` · `gestoppt` (· `offen`/`verfallen` selten).
Stoppen vom Handy: `POST w=stopp {id:<auftrag-id aus raeume>, wer:"handy"}` → das Gerät beendet den Lauf beim
nächsten Puls (≤ 10 s, solange im Raum gesprochen wird) und schreibt „Gestoppt von einem anderen Gerät" in den Raum.
Den Text sieht man nur am Rechner. Ob Bene vom Handy aus **schreiben** darf (dann läge sein Satz einmal in der
Rezeption), ist eine offene Entscheidung für ihn — bis dahin: am Handy Stand und Stopp, keine Eingabe.

## 4. Wie ein Zug läuft (besteht)

```
 Compass (am Rechner)          Tür :8788 (Worker)                 Kindprozess (john-auftrag.ps1)          Rezeption
  POST /raum {id,text,an} ──►  Benes Zeile an raum/<id>.jsonl
                               in die Warteschlange (vor dem Takt)
                               Kopf frei? ──► startet Kind ──────► w=auftrag {art:raum, raum, zug, an} ──► offen
                                                                   w=nimm ───────────────────────────────► laeuft
                                                                   John: Claude mit seiner Lage
                                                                   Madeleine: Codex mit ihrem Wissen
                                                                   (+ Züge dieses Raums, weitergeben≠false)
                                                                   Zeile an raum/<id>.jsonl
                                                                   w=ergebnis {ok, notiz:"Zug N liegt am Gerät"} ► fertig
  GET /raum?id&seit ◄───────── neue Zeilen
  POST /stopp ───────────────► taskkill /T (Kind + claude/codex) · system-Zeile · w=stopp ───────────────► gestoppt
```

**Was in einen Zug eingeht:** Persona und Wissen **des Sprechenden** plus die Züge dieses Raums mit
`weitergeben ≠ false`. John bekommt, was Madeleine **gesagt** hat, nicht, was sie **weiß** — und umgekehrt. Nie die
Wissensdateien des anderen, nie Schlüssel.

**Wo der Raum liegt:** `C:\dev\john\coaching\raum\<id>.jsonl` auf Benes Rechner (lokal, ohne Remote). Zeile 1
`{meta, thema, erstellt}`, danach je Zug `{zug, wer, zeit, text, weitergeben}`. `.jsonl` statt `.md`, damit der Raum
nicht ungefragt in Johns Systemprompt wandert.

## 5. Rezeption: was sich verbindlich geändert hat (für Tests in deiner Umgebung)

- Schlüsselklassen: `geraet` darf zusätzlich `stopp`; `browser` darf `stand, punkt, auftrag, stapelstand, stopp`.
- Auftragsarten: `stapel` · `board` · `chat` · `frage` · `takt` · `coach` · **`raum`**. Einen Raum-Zug legt **nur
  das Gerät** an (Browser → 403), mit leerem `text` (sonst 400); `ergebnis` mit Text → 400.
- Status: `offen → laeuft → fertig`, dazu `offen/laeuft → gestoppt` über `w=stopp` (sonst 409); `nimm`/`ergebnis`
  auf gestoppt → 409. `puls` meldet `stopp:[ids]` an das Gerät, das den Auftrag hatte.
- Offene Raum-Züge zählen nicht in `puls.auftraege`.
- Eine eigene Art `madeleine` gibt es **nicht**: eine einzelne Frage an Madeleine ist ein Raum mit einem Zug.

Für eine eigene Rezeption in deiner Umgebung: `john-agent/hub/api.php` mit `php -S` und zwei Test-Token in
`token.php` (nur SHA-256-Hashes) — der Prüfstand dazu ist in wenigen Zeilen nachgebaut.

## 6. Anschluss für eine externe Work-Sitzung (Astra)

**Grundsatz: so wenig wie möglich, einzeln widerrufbar, nie als Gerät.** Ein externes Gerät, das Aufträge
beansprucht, wäre eine zweite Madeleine, also eine zweite Agentenarchitektur.

| Stufe | Astra bekommt | Astra bekommt nicht |
|---|---|---|
| **Entwickeln** (jetzt) | Lesen auf GitHub; `tools/tuer-attrappe.php` als Tür; eine **eigene** Rezeption mit Test-Token | keinen Produktionsschlüssel |
| **Beitragen** | Branch oder Pull Request; Claude prüft am laufenden Worker und spielt aus | kein direkter Push auf `main`, kein FTP |
| **Beobachten** (Vorschlag, später) | dritte Schlüsselklasse `extern` (Variable `JOHN_HUB_TOKEN_EXTERN`, von Bene selbst gegeben): nur `stand` in verkürzter Form | Stapeltitel, Compass-Spiegel, Logbuch, `auftrag`, `nimm`, `ergebnis`, `puls`, `stapelstand` |

Nie erreichbar für Astra: die echte Tür `127.0.0.1:8788`, `C:\dev\madeleine`, `C:\dev\john`, Benes
Vishnu-Anmeldung, Benutzer-Umgebungsvariablen. Wissen von der lokalen Madeleine zu Astra geht **nur** über Dokumente
wie dieses. Einen Laufzeitweg gibt es nicht.

**Deine nächsten Schritte:**
1. Den Prototyp auf die Tür umstellen (`/raeume`, `/raum`, `/stopp`, `/raum/weitergeben`) und gegen
   `tuer-attrappe.php` laufen lassen.
2. `system`-Zeilen, `laeuft`/`wartet` und die Fehlercodes sichtbar machen (nie ein leeres Feld ohne Grund).
3. Als eigene Datei `compass/compass-gespraechsraum.js` in `john-agent` einreichen (PR). Einbau in den Compass,
   Demo-Ausschluss und Prüfung am echten Worker übernehme ich.

## Regeln aus beiden Repos, die hier gelten

Kein Denken im ausliefernden Prozess · leer heißt nie „nichts" · die Rezeption bleibt dumm · Protokoll zuerst ·
nichts Persönliches in die öffentlichen Repos (in `madelene-agent` prüft das ein Hook) · Git nur Porcelain, kein
Force · Encoding `.ps1` mit BOM, sonst UTF-8 ohne BOM, LF.
