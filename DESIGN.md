# Design: Twint-nahe Bedienlogik

## Prinzipien
- Ein Screen = eine Aufgabe. Im Zweifel Inhalt weglassen.
- Jede Screen hat genau eine Hauptaktion, unten, volle Breite,
  min. 52px hoch, abgerundet (16px).
- Die wichtigste Zahl ist die grösste: 40-48px, halbfett.
  Zweitwichtigste 17px. Nichts dazwischen.
- Grosszügiges Weiss: min. 24px zwischen Blöcken.

## Bottom Sheets
- Jede Aktion (erfassen, bearbeiten, bestätigen) öffnet ein Sheet
  von unten, nicht eine neue Seite.
- Aufbau: Griff-Balken oben, Titel, Inhalt, Primärbutton unten.
- Schliessen per Wischen nach unten, Tap auf den abgedunkelten
  Hintergrund und X oben links.
- Hintergrund abdunkeln (rgba(0,0,0,.4)), Seite dahinter darf
  nicht scrollen.
- Höhe nur so hoch wie nötig, max. 90% der Bildschirmhöhe.
- padding-bottom: env(safe-area-inset-bottom) beachten.
- Nie ein Sheet aus einem Sheet öffnen.

## Beträge
- Bei Betragseingabe: grosse Zahl mittig im Sheet,
  inputmode="decimal", Fokus automatisch.
- Format CHF 1'234.55, tabular-nums, gleiche Schriftfamilie
  wie der Rest.

## Nach dem Speichern
- Sheet schliesst, kurze Bestätigung (Toast), Liste aktualisiert.
  Keine Erfolgsseite.

## Verboten
- Tabellen mit mehr als 2 Spalten auf dem Handy
- Emoji als Icons
- Mehr als eine Akzentfarbe
- Rot ausser bei echten Fehlern
