# Regeln für Sessions in madelene-agent

Dieses Repo ist Madeleines **Architektur**, nicht ihr Inhalt, und es ist **öffentlich**. Ihr Inhalt
(Persona im Wortlaut, Wissen mit Zahlen, Notizen, privater Kontostand) liegt in `C:\dev\madeleine`, einem
Ordner ohne Git. Ihr Code liegt heute noch in `C:\dev\persoenliches-dashboard` (Repo flow-compass).
Schwester-Repo: `C:\dev\john-agent`.

## Nicht verhandelbar

1. **Kein Inhalt hierher** (ADR 0001): keine Beträge, keine IBAN oder Kontonummern, keine Namen aus
   Verträgen, Kundschaft oder Steuerbüro, keine Datei aus `C:\dev\madeleine`. Der Hook `.githooks/pre-commit`
   prüft Pfade, Beträge und eine lokale Sperrliste (`C:\dev\madeleine\.sperrliste`, nie ins Repo). Nie
   reflexhaft mit `ALLOW_INHALT=1` übersteuern.
2. **Sichtgrenzen stehen serverseitig**, nie nur im Prompt.
3. **Encoding:** `.ps1` UTF-8 mit BOM, alles andere UTF-8 ohne BOM, LF.
4. **Git nur Porcelain:** `pull --ff-only` → `add <datei>` → `commit` → `push`. Kein `add -A`, kein
   Plumbing, kein `push --force`. Vor jedem Commit `git status --short` lesen.
5. **Parallele Sessions:** `C:\dev\_tools\git-flow.ps1 -Modus claim -Repo madelene-agent …`, am Ende `release`.

## Mit Madeleine arbeiten

Gegenprüfen lassen über `C:\dev\john-agent\beratung\frag-madeleine.ps1`. Ihre Antwort geht ungefiltert ins
Protokoll, darunter steht, was übernommen wurde und warum. Eine Frage an sie enthält **keine** Zahlen aus
ihrem Wissen. Sie bekommt die Architektur, nicht die Konten.

## Ton

Deutsch, Du-Form, deutsche Anführungszeichen „…". Begriffe wie in john-agent: Rezeption, Gerät, Lobby,
Auftrag, Takt, dazu hier: **Briefkasten**, **Beraterrunde**, **Lagebild**, **Sichtgrenze**.
