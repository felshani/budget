# Budget — Meilenstein 0

Prüfblatt für das Fundament der Finanz-App: Installation aufs iPhone,
Offline-Betrieb, dauerhafter Speicher und Sicherung.

## Dateien

| Datei | Zweck |
|---|---|
| `index.html` | die ganze App |
| `manifest.webmanifest` | Name, Icon, Vollbildmodus |
| `sw.js` | Service Worker für Offline-Betrieb |
| `icon-180.png` | Icon für den iPhone-Home-Bildschirm |
| `icon-192.png`, `icon-512.png` | Icons für das Manifest |

Alle Dateien gehören ins **selbe Verzeichnis**, ohne Unterordner.

## Auf GitHub Pages veröffentlichen

1. Neues Repository anlegen, zum Beispiel `budget`. **Public** — Pages
   veröffentlicht bei einem Gratis-Konto nur aus öffentlichen Repositories.
   Das ist unbedenklich: Hier liegt nur Programmcode. Deine Zahlen verlassen
   das iPhone nie und landen niemals in diesem Repository.
2. Alle Dateien hochladen (`Add file` → `Upload files`), dann `Commit changes`.
3. `Settings` → `Pages` → unter *Build and deployment* bei *Source*
   **Deploy from a branch** wählen, Branch `main`, Ordner `/ (root)`, `Save`.
4. Ein bis zwei Minuten warten. Die Adresse lautet dann
   `https://<dein-benutzername>.github.io/budget/`.

## Aufs iPhone legen

1. Die Adresse **in Safari** öffnen. Nicht in Chrome — nur Safari darf zum
   Home-Bildschirm hinzufügen.
2. Teilen-Symbol antippen → **Zum Home-Bildschirm** → `Hinzufügen`.
3. Die App über das neue Icon starten, nicht mehr über Safari.

Ab jetzt läuft sie im eigenen Fenster, ohne Adressleiste, und startet auch
ohne Netz.

## Was zu prüfen ist

Oben im Prüfblatt stehen sechs Befunde. Erwartet wird:

| Befund | Erwartung |
|---|---|
| Als App installiert | `ja`, sobald über das Icon gestartet |
| Offline-Betrieb | `bereit` beim zweiten Start |
| Speicher im Gerät | `liest` und `schreibt` |
| Dauerhafter Speicher | `geschützt`, sonst `nicht zugesichert` |
| Teilen von Dateien | `möglich` |
| Freier Platz | dreistellige MB-Zahl oder mehr |

Danach drei Dinge selbst ausprobieren:

1. **Speicher:** Zähler hochsetzen, Notiz schreiben, App vollständig schliessen
   (aus dem App-Umschalter wischen), neu öffnen. Werte müssen stehen.
2. **Offline:** Flugmodus einschalten, App über das Icon starten. Sie muss
   normal laden.
3. **Sicherung:** `Teilen` → `In Dateien sichern` → iCloud Drive. Danach
   `Zurücksetzen`, dann die Datei über *Sicherung wiederherstellen* laden.
   Zähler und Notiz müssen zurückkommen.

Falls `Teilen` nicht klappt: `Herunterladen` und `Kopieren` sind die beiden
Ausweichwege. Welcher davon funktioniert, entscheidet, wie die fertige App
sichert.

## Aktualisieren

Nach jeder Änderung an `index.html` die Zeile `const CACHE = 'budget-m0-v1';`
in `sw.js` hochzählen (`-v2`, `-v3` …). Sonst zeigt das iPhone die alte
Fassung aus dem Cache.
