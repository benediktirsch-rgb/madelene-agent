# Übergabe an Astra — Gesprächsraum auf Rezeption und Worker

**Von:** Claude (Architektur john-agent) · **Für:** Astra (Codex in ChatGPT Work, Madeleines Rolle im gemeinsamen Konzept)
**Stand:** 11.09.2026 · **Status:** Integrationsvorlage. Was „besteht", läuft; was „Vorschlag" heißt, ist nicht gebaut.

**Ziel:** ein gemeinsamer Gesprächsraum im Flow Compass, in dem Bene, John und Madeleine sprechen, gebaut auf der
bestehenden **Rezeption** (hotel-vaikuntha.de/john) und dem **Worker** auf Benes Rechner. Keine weitere Persona,
keine zweite Agentenarchitektur, keine neuen Orte für Wissen oder Schlüssel.

**Voraussetzung, die ich annehme:** Im Raum antwortet Madeleine als **lokaler Worker-Auftrag** (Codex CLI auf Benes
Rechner mit ihrem lokalen Wissen). Astra baut und prüft den Raum, antwortet aber nicht selbst darin. Sollte Astra
selbst im Raum sprechen, müsste Gesprächsinhalt in Astras Umgebung fließen. Das wäre eine eigene Entscheidung von Bene
und steht hier bewusst nicht.

---

## 1. Protokoll der Rezeption — was verbindlich ist

Maßgeblich ist `john-agent/docs/protokoll.md`, der Code folgt ihm (`john-agent/hub/api.php`). Kurzfassung:

**Aufruf:** `https://hotel-vaikuntha.de/john/api.php?w=<was>`, Kopf `X-John-Token`, JSON rein und raus, immer `ok`.
Zeiten ISO 8601 mit Zone Europe/Berlin. Ein Zustand in einer Datei unter `flock`.

**Schlüsselklassen (besteht):**

| Klasse | darf | wer |
|---|---|---|
| `geraet` | stand, puls, stapel, punkt, auftrag, auftraege, nimm, ergebnis, log, spiegel, stapelstand | Worker auf Benes Rechner |
| `browser` | stand, punkt, auftrag, stapelstand | eigener Compass (eingesetzt beim Bauen) |

**Auftragsarten (besteht):** `stapel` · `board` · `chat` · `frage` · `takt` · `coach`. Der Worker kennt heute
`coach` (öffentliche Persona Bene digital, sonst nichts) und `frage` (Johns Persona); alle anderen laufen mit Johns
Persona und Lage.

**Statuswechsel eines Auftrags (besteht):**

```
            w=auftrag                     w=nimm (Gerät)                 w=ergebnis
  (neu) ───────────────► offen ─────────────────────────► laeuft ───────────────────► fertig
                           │  ▲                              │                   (ergebnis.ok true|false)
          24 h unberührt   │  │ 15 min ohne ergebnis         │
                           ▼  └──────────────────────────────┘
                       verfallen          (Beanspruchung verfällt, ein anderes Gerät darf)
```

- `nimm` ist die Sperre: zwei Geräte streiten nicht, das zweite bekommt **409**.
- Ein gescheiterter Auftrag ist `fertig` mit `ergebnis.ok = false`; einen eigenen Fehlerstatus gibt es nicht.
- `fertig` und `verfallen` räumt die Rezeption nach 7 Tagen weg. Das Logbuch hält nur Art und Länge, nie Text.
- **Einen Stopp gibt es noch nicht.** Siehe Abschnitt 3.

**Offene Lücke, die der Raum nicht erben darf:** `auftrag.text` (bis 2000 Zeichen) und `ergebnis.text` (bis 4000)
speichern heute Inhalt in der Rezeption. Für `coach` ist das gewollt: öffentliche Persona, Bene gibt vor der
Auslieferung frei. Für alle anderen Arten widerspricht es der Regel „die Rezeption bleibt dumm" (john-agent
ADR 0002). Der Gesprächsraum nutzt diese Felder **nicht**.

## 2. ADR 0002 „Zwei Berater, ein Haus" — die Madeleine-Aufträge

Quelle: `madelene-agent/docs/adr/0002-zwei-berater-ein-haus.md` (Vorschlag). Madeleine zieht in dieselbe Rezeption
und denselben Worker ein. Sie bekommt kein eigenes Haus, keinen eigenen Takt und keine eigene Lobby.

**Vorgesehene Aufträge (Vorschlag):**

| Art | wer denkt | Eingabe über die Rezeption | Ergebnis in der Rezeption |
|---|---|---|---|
| `madeleine` | Worker → Codex CLI, lokale Persona und Wissen | nur Thema-Kennung, **kein Text** | „liegt am Gerät (N Zeichen)" |
| `raum` | Worker → John (Claude) **oder** Madeleine (Codex), je nach `an` | `raum`, `zug`, `an` — **kein Text** | „Zug N liegt am Gerät" |

- Beide laufen wie heute jeder Denkvorgang: als abgekoppelter Kindprozess des Workers (`john-auftrag.ps1`), nie im
  bedienenden Prozess. Höchstens einer denkt gleichzeitig.
- **Voraussetzung, die fehlt:** Madeleines Prompt-Aufbau (`Build-SystemMadeleine`, `Invoke-CodexCli`) steckt in
  `flow-compass/john-madeleine.ps1` und wird nur vom Cockpit-Server geladen. Der Worker muss ihn ohne diesen Server
  rufen können. Plan: zusammen mit john-agent ADR 0003 ein gemeinsames Modul `john-ki.ps1` (Claude-CLI **und**
  Codex-CLI, Fehlercodes, Umgebung ohne API-Schlüssel), das Server und Worker beide laden.
- Der **Monatsblick** nach dem Finanzlauf (ADR 0002, Punkt 3) ist offen und kein Teil des Raums.

## 3. Gesprächskontext lokal, Rezeption inhaltsfrei

**Grundsatz:** Die Rezeption trägt nur das **Signal** (wer denkt, in welchem Raum, welcher Zug, welcher Status), der
Gesprächstext bleibt auf Benes Rechner. So bleibt ein Webspace ohne Tresor frei von Inhalten, und Handy und Netz
sehen den Stand, ohne den Text zu sehen.

**Wo der Raum liegt (Vorschlag):** `C:\dev\john\coaching\raum\<raum-id>.jsonl`, eine Zeile je Zug:
`{zug, wer: bene|john|madeleine, zeit, text, weitergeben: true|false}`. Der Ort liegt lokal, ohne Remote, und ist
schon heute die gemeinsame Ablage beider Berater (`beraterrunde.md`). `.jsonl` statt `.md`, damit der Raum nicht
ungefragt in Johns Systemprompt wandert, der alle `coaching/*.md` lädt. Die bisherige `beraterrunde.md` bleibt die
Zusammenfassung.

**Was „freigegebener Gesprächskontext" heißt:**

| geht in den nächsten Zug | geht nie über die Grenze |
|---|---|
| die Züge dieses Raums, die Bene sieht, außer denen mit `weitergeben: false` | Wissensdateien des jeweils anderen (`C:\dev\madeleine\wissen`, `privat\`, Johns Profil, Pipeline, Memory) |
| Benes eigene Beiträge im Raum | Schlüssel, Kontostände aus Dateien, Mail-Entwürfe |

John bekommt also, was Madeleine **gesagt** hat, aber nicht, was sie **weiß**, und umgekehrt. Beide Modelle laufen
auf Benes Abos. Dass Madeleines Zahlen an OpenAI gehen, hat Bene bestellt. Dass Johns Worte über den Raum bei ihr
landen, ist neu und genau durch die Zeilenliste begrenzt.

**Ein Zug im Ablauf (Vorschlag):**

```
 Compass (am Rechner)                Worker :8788                      Rezeption
  POST /raum {id, text} ───────────► Zeile an raum/<id>.jsonl
                                     w=auftrag {art:raum, raum, zug, an:'madeleine'} ──► offen
                                     puls ◄── auftraege: 1
                                     w=nimm ────────────────────────────────────────────► laeuft
                                     Kind: Codex mit Persona + Wissen + freigegebenen Zügen
                                     Zeile an raum/<id>.jsonl
                                     w=ergebnis {ok, notiz:"Zug 4 liegt am Gerät"} ─────► fertig
  GET /raum?id&seit=3 ◄────────────  neue Züge (Live: alle 2 s, Tür antwortet in ms)
```

**Neu nötig (Vorschlag):**
- **Tür des Workers:** `GET /raum?id=&seit=<zug>` (Züge seit N), `POST /raum {id, text, an}` (Beitrag von Bene plus
  Auftrag), `POST /stopp {id}` (laufenden Kindprozess beenden). Nur lokal erreichbar, wie heute `/stand`.
- **Rezeption:** Art `raum` in der Liste der Auftragsarten, und bei `raum` weist die Rezeption `text` in `auftrag`
  und `ergebnis` ab (400). Neuer Status `gestoppt` über `POST w=stopp {id}` (Gerät und Browser). Die Puls-Antwort
  meldet `stopp: [ids]`, damit der Worker den Kindprozess beendet, auch wenn Bene am Handy stoppt.
- **Protokoll zuerst:** jede dieser Änderungen erst in `john-agent/docs/protokoll.md`, dann im Code.

**Was am Handy geht:** Stand des Raums (läuft, Zug N fertig, gestoppt) aus der Rezeption und Stopp. Den Text sieht
man nur am Rechner. Ob Bene vom Handy aus **schreiben** darf (dann läge sein Satz einmal in der Rezeption), ist
eine offene Entscheidung für ihn.

**Zum Prototyp:** Er ruft `/api/john` und `/api/madeleine` am Cockpit-Server auf. Dieser Server arbeitet seriell: ein
Denkvorgang blockiert ihn 60–90 s, und genau das war der Grund für die neue Architektur. Die Oberfläche (Verlauf,
Live, Stopp) passt. Der Transport wechselt auf `POST/GET /raum` an der Tür und die Art `raum` in der Rezeption.

## 4. Anschluss für eine externe Work-Sitzung (Astra)

**Grundsatz: so wenig wie möglich, einzeln widerrufbar, nie als Gerät.** Ein externes Gerät, das Aufträge
beansprucht, wäre eine zweite Madeleine, also eine zweite Agentenarchitektur.

| Stufe | Astra bekommt | Astra bekommt nicht |
|---|---|---|
| **Entwickeln** (sofort) | Lesen auf GitHub (`john-agent`, `madelene-agent`, `flow-compass`); in der eigenen Linux-Umgebung eine **eigene** Rezeption (`hub/api.php` mit `php -S`, Test-Token, simulierter Worker über HTTP) | keinen Produktionsschlüssel |
| **Beitragen** | Änderungen als Branch oder Pull Request. Claude prüft sie und spielt sie aus, weil Deploy und Tests am laufenden Worker Benes Rechner brauchen | kein direkter Push auf `main`, kein FTP |
| **Beobachten** (Vorschlag, später) | dritte Schlüsselklasse `extern` (Variable `JOHN_HUB_TOKEN_EXTERN`, von Bene selbst an Astra gegeben): nur `stand` in **verkürzter** Form (Geräte und Puls, Auftragszähler, Takt, Status eines Raums) | Stapeltitel, Compass-Spiegel, Logbuch, `auftrag`, `nimm`, `ergebnis`, `puls`, `stapelstand` |

Nie erreichbar für Astra: die Tür `127.0.0.1:8788`, `C:\dev\madeleine`, `C:\dev\john`, Benes Vishnu-Anmeldung,
Benutzer-Umgebungsvariablen. Wissen von der lokalen Madeleine zu Astra geht **nur** über Dokumente wie dieses, die
Bene oder Claude ausdrücklich übergeben. Einen Laufzeitweg gibt es nicht.

**Erste Aufgaben, die ohne weiteren Zugang gehen:**
1. Den Prototyp gegen eine lokale Rezeption plus simulierten Worker umbauen (Art `raum`, Tür `/raum`, `/stopp`).
2. Die Protokoll-Erweiterung aus Abschnitt 3 als Pull Request an `john-agent/docs/protokoll.md` formulieren, bevor
   Code entsteht.
3. Tests, die beweisen, dass die Rezeption bei `raum` keinen Text annimmt und dass `gestoppt` einen laufenden
   Auftrag erreicht.

## Regeln aus beiden Repos, die hier gelten

Kein Denken im ausliefernden Prozess · leer heißt nie „nichts" · die Rezeption bleibt dumm · Protokoll zuerst ·
nichts Persönliches in die öffentlichen Repos (in `madelene-agent` prüft das ein Hook) · Git nur Porcelain, kein
Force · Encoding `.ps1` mit BOM, sonst UTF-8 ohne BOM, LF.
