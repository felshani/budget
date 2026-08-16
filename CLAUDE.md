# CLAUDE.md

Verbindliche Regeln für dieses Repository. Sie gelten für jede Änderung.
Bei Widerspruch zwischen diesen Regeln und einer Aufgabenbeschreibung:
zuerst nachfragen, nicht stillschweigend abweichen.

## Was das ist

Eine private Budget-App für einen Schweizer Zweipersonenhaushalt. Sie gleicht
den geplanten Sparbetrag gegen die tatsächliche Veränderung der Kontostände ab
und lässt die Differenz durch erfasste Posten erklären.

Vollständige Spezifikation: `docs/PRD.md`. Vor jeder inhaltlichen Änderung den
betreffenden Abschnitt dort lesen.

## Harte Architekturregeln

1. **Ein Benutzer.** Keine Anmeldung, keine Benutzerkonten, keine
   Berechtigungen, keine Synchronisation. Zwei *Personen* sind eine
   Auswertungsdimension in den Daten, kein Mehrbenutzersystem. Der Vorgänger
   ist genau daran gescheitert.
2. **Kein Backend.** Kein Server, keine Datenbank, keine externe API. Alles
   läuft im Browser.
3. **Ein einziges JSON-Dokument** in IndexedDB unter Schlüssel `dokument`.
   Keine zweite Ablage, kein localStorage.
4. **Keine Buchungs- oder Überweisungstabelle.** Erfasst werden nur Bestände zu
   Stichtagen sowie Ist-Werte und Sonderposten pro Monat. Überweisungen zwischen
   eigenen Konten werden nie erfasst.
5. **Kein hartes Löschen.** Entfernte Kategorien, Posten, Konten, Personen oder
   Töpfe bekommen `aktiv: false`. Historische Monate dürfen nie brechen.
6. **Abgeschlossene Monate sind eingefroren.** `monatsabschluss.ergebnisse`
   enthält ausgeschriebene Zahlen, keine Verweise. Eine spätere Umbenennung darf
   die Vergangenheit nicht verändern.
7. **`schemaVersion` beim Laden migrieren.** Jede Version liest alle älteren
   Dateien. Migration ist eine Funktion, kein manueller Eingriff.
8. **Alles Nutzerdefinierte ist Daten.** Kategorien, Personen, Töpfe, Konten,
   Investments stehen nie im Code. Eine neue Kostenstelle ist ein Eintrag.

## Technik

- Vanilla HTML, CSS und JavaScript. **Kein Build-Schritt, keine npm-Pakete,
  kein Framework.** Was im Repository liegt, ist das, was ausgeliefert wird.
- Alle Dateien flach im Wurzelverzeichnis, ausser `docs/`. GitHub Pages liefert
  aus `/ (root)`.
- Zielgerät ist ein iPhone in Safari, installiert über "Zum Home-Bildschirm".
  Nur Funktionen verwenden, die Safari auf iOS beherrscht. Keine File System
  Access API — die gibt es dort nicht.
- Muss vollständig offline funktionieren.
- **Nach jeder Änderung an einer Datei aus `SHELL` in `sw.js` die Konstante
  `CACHE` hochzählen** (`budget-m0-v1` → `-v2` …). Sonst bleibt auf dem iPhone
  die alte Fassung im Cache.
- Speichern erfolgt automatisch und entprellt. Es gibt keinen Speichern-Knopf
  für den Normalbetrieb.

## Sprache und Formate

- Sämtliche Oberflächentexte, Kommentare und Commit-Nachrichten auf **Deutsch**,
  Schweizer Rechtschreibung (`ss` statt `ß`).
- Beträge in CHF im Format `1'234.55`, Tausendertrennzeichen ist der Apostroph.
- Datumsangaben `TT.MM.JJJJ`, Monate intern als `JJJJ-MM`.
- Feldnamen im Datenmodell auf Deutsch, wie in `docs/PRD.md` Abschnitt 12.
- Beschriftungen sagen, was passiert: `Monat abschliessen`, nicht `Senden`.

## Niemals hartkodieren

Säule-3a-Maximum, Serafe-Gebühr, Franchisestufen, Selbstbehalt-Höchstbetrag,
Verteilschlüssel, Anzahl Personen, Anzahl Töpfe. Alles davon sind Stammdaten
mit Jahresgültigkeit oder Nutzereingaben.

## Zwei Bedeutungen von "Soll"

- **Budget:** geplante Ausgabe beziehungsweise geplanter Sparbetrag
- **Investment:** erwarteter Depotwert bei angenommener Rendite

In Code und Oberfläche getrennt benennen, nie vermischen.

## Rechenregeln, die leicht falsch gemacht werden

- Die Brücke rechnet in der **Fälligkeitssicht**: Kosten im Monat ihrer
  tatsächlichen Zahlung. Die geglättete Durchschnittssicht dient nur der
  Beitragsberechnung und der Jahresplanung.
- Die unerklärte Differenz ist immer ein **Ergebnis**, nie ein Eingabefeld.
- Kursbewegungen von Investments gehören nicht in die Sparrechnung.
- Eine Zusatzeinzahlung ins Investment ist eine **Umbuchung**, keine Ausgabe.

## Vorgehen bei Aufgaben

- Kleine, abgeschlossene Änderungen pro Branch.
- Vor dem Push selbst prüfen: Lädt `index.html` ohne Konsolenfehler? Ist
  `CACHE` hochgezählt? Ist das Datenmodell abwärtskompatibel?
- Wenn eine Anforderung dem PRD widerspricht, im Pull Request darauf hinweisen
  statt das PRD stillschweigend zu übergehen.
