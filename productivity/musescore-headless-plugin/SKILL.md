---
name: musescore-headless-plugin
description: Use when a MuseScore 4 QML plugin must run headless.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [musescore, plugin, qml, headless, xvfb, xdotool]
    related_skills: [musicxml-engraving-accessibility]
---

# MuseScore-4-Plugins ohne Fenster ausführen

## When to Use

* Ein QML-Plugin soll auf einer .mscz/.mscx-Datei laufen, ohne dass jemand
  am Rechner sitzt (Stapellauf, Server, Automatisierung).
* Ein Plugin soll aus einem Skript oder Cronjob heraus ausgelöst werden.
* Diagnose, warum ein Plugin-Aufruf per Kommandozeile wirkungslos bleibt.

MuseScore Studio 4 kann QML-Plugins NICHT von der Kommandozeile starten.
Verifiziert am 4.7.5-Binary (`strings` + `convertercontroller.cpp`): der
Batch-Job `-j job.json` kennt nur `in`/`out`/`transpose` – der in alter
Doku genannte Schlüssel `plugin` wird ignoriert (kein Fehler, kein Effekt!).
`-p/--plugin` gibt es nur in MuseScore 3, das wiederum keine 4.x-Dateien
öffnet. Auch das Plugins-Menü per Alt+P ist per xdotool unzuverlässig.

Funktionierender Weg: GUI in Xvfb + Mini-Fenstermanager + Tastaturkürzel.

## Ablauf

1. Plugin nach `~/Documents/MuseScore4/Plugins/<name>.qml` legen und
   aktivieren. Vorab per Datei, ohne GUI:
   `~/.local/share/MuseScore/MuseScore4/extensions/config.json` ergänzen um
   `{"actions":[{"code":"main","exec_point":"manually"}],"uri":"musescore://extensions/v1/<name>.qml"}`
   (Format der Liste: Array aus Objekten mit `uri` und `actions`).
2. Tastaturkürzel direkt in `~/.local/share/MuseScore/MuseScore4/shortcuts.xml`
   eintragen – Schema ist exakt
   `<SC><key>action://extensions/v1/<name>.qml?action=main</key><seq>Ctrl+Shift+L</seq></SC>`.
   Nur patchen, wenn kein MuseScore läuft: die Datei wird beim Beenden aus
   dem Speicher neu geschrieben und überschreibt sonst die Änderung.
3. Xvfb (`:99 -screen 0 1600x1200x24 -ac -nolisten tcp`) UND `openbox`
   starten. Ohne Fenstermanager bekommt das Fenster keinen verlässlichen
   Tastaturfokus und Tasteneingaben landen nicht in MuseScore.
4. `session/session.json` und `recent_files.json` löschen, sonst ignoriert
   MuseScore die als Argument übergebene Datei (Session-Wiederherstellung).
5. MuseScore mit der Datei starten, auf das Fenster warten, dessen PID zu
   unserem Prozess gehört (`xdotool search --name` + `getwindowpid`),
   `xdotool windowactivate --sync`, Escape (Hinweisdialog), dann das Kürzel.
6. Erfolg nicht am Exit-Code messen: die Datei kann unverändert sein. Auf
   das Plugin-Protokoll warten (siehe unten), dann `ctrl+s`, dann prüfen,
   dass sich die mtime geändert hat. Erst danach die Ausgabe kopieren.

## Protokoll und Diagnose

* `console.log` erscheint NICHT im Log, `console.warn` schon: Plugin-Meldungen
  immer über `console.warn("Praefix| ...")` ausgeben.
* Log: `~/.local/share/MuseScore/MuseScore4/logs/MuseScore_<datum>_<zeit>.log`
  (pro Start eine neue Datei). Darin auch `ActionsDispatcher::doDispatch |
  try call action: action://extensions/v1/...` – das ist der Beweis, dass das
  Plugin ausgelöst wurde.
* Beleg für den GUI-Weg ohne Plugin: `doDispatch ... pitch-down` /
  `system-break` – Tastendrücke landen im Notentext, das Menü war nicht offen.

## Fallen

* MuseScore cached QML: nach jeder Änderung am Plugin Neustart nötig,
  sonst läuft die alte Fassung.
* Prozesse sauber beenden: `pgrep -f <pfad-der-temp-score>` trifft den
  AppImage-Wrapper UND `mscore4portable`; ein Wrapper-PID allein lässt
  Kindprozesse verwaist zurück. Nie mit einem Muster killen, das in der
  eigenen Kommandozeile steht – das trifft die eigene Shell (Exit -15).
* `grep -c` gibt bei 0 Treffern Exit 1 aus: in `$(...) || echo 0` landen
  zwei Werte ("0\n0") und der `[ -gt ]`-Vergleich bricht. Stattdessen
  `n=$(grep -ac ...); n=${n:-0}`.
* `removeElement()` wirft bei Lyrics-Elementen in MuseScore 4: leere Texte
  werden geleert, nicht entfernt. Immer `try/catch` und Zähler für beide
  Fälle führen.
* Screenshots (`import -window root`) schliessen ein offenes Qt-Menü –
  Menüprüfung und Klick nie in einem Zug mischen.

## Sparen (Lehren aus dem ersten Durchlauf)

* Erst Doku, dann Binary: Die Frage "geht das per CLI?" kostet `strings`- und
  Quelltext-Analysen. Offizielle Doku zuerst lesen, dann am Binary nur noch
  bestätigen.
* Fenster-Manager von Anfang an installieren, nicht erst nach mehreren
  gescheiterten Versuchen.
* Das Tastaturkürzel in `shortcuts.xml` schreiben ist der kurze Weg zum
  Auslösen. Das Plugins-Menü zu bedienen kostet viele Versuche und
  Screenshots.
* Ein Screenshot liefert alle Koordinaten eines Dialogs – einmal messen, dann
  blind klicken. Screenshot und Klick nie in denselben Zug legen: der
  Screenshot schliesst das Menü.
* Textspuren sind billiger als Bilder: das ActionsDispatcher-Log und die
  eigenen Plugin-Meldungen zeigen Zustand und Fokus exakt und kosten keine
  Vision-Tokens. Vision gezielt für blind zugängliche GUI-Teile (Dialoge,
  Auswahlmöglichkeiten) einsetzen, nicht für Wiederholungen.
* Nie dem Exitcode des Wrappers glauben: Ausgabe immer inhaltlich prüfen
  (Lyrics extrahieren und gegen die Erwartung vergleichen). Ein Lauf kann
  "fertig" melden und die Datei unverändert lassen.

## Referenzimplementierung

`/home/ubuntu/musescore-plugins/lyrics-cleanup/` – Plugin (Lyrics aufräumen),
Wrapper `lyrics-cleanup.sh` (Xvfb + openbox + Kürzel + Protokollprüfung +
Speichern), Testsatz mit 26 Sonderfällen. Prüfmuster: Ausgabe-Lyrics gegen
erwartete Liste vergleichen (Python + zipfile + regex auf `<Lyrics><text>`),
nicht dem Wrapper-Exitcode glauben.
