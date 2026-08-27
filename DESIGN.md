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
- Mehr als eine Akzentfarbe, ausser als Kategorienfarbe im Kreisdiagramm
- Rot ausser bei echten Fehlern und bei einem negativen Ergebnis

Zur letzten Regel: Ein negativer Sparbetrag ist kein Programmfehler, soll
aber sofort auffallen. Ergebniszahlen — die grosse Kennzahl und die
betonte Schlusszeile eines Blocks — sind darum grün im Plus und rot im
Minus. Für alles andere gilt die Regel unverändert.

## Kategorienfarben

Genau eine Stelle trägt mehr als eine Farbe: das Kreisdiagramm im Analyse-Tab.
Dort ist Farbe kein Schmuck, sondern der Kanal, über den ein Stück seiner
Gruppe zugeordnet wird. Überall sonst gilt die eine Akzentfarbe weiter — auch
in den beiden anderen Diagrammen.

Regeln dazu:

- **Die Farbe hängt an der Gruppe, nie an der Position.** Fällt eine Gruppe
  weg, behalten die übrigen ihre Farbe. Die Zuordnung steht in
  `gruppenFarbe()`, die Werte als `--kat-1 … --kat-10` in `:root` und im
  Dunkelblock; `--kat-0` ist der neutrale Platz für „Ohne Gruppe" und für
  Gruppen aus älteren Dateien.
- **Feste Farbtöne im Abstand von 36 Grad, abwechselnd tiefe und helle
  Stufe.** Die Helligkeitsalternanz hält Farbtöne auseinander, die einander
  ähnlich sind — nicht der Farbton allein.
- **Transparent.** `--kat-deck` steht in beiden Modi auf 78 %, damit die
  Farben zum ruhigen Rest der App passen. Umgesetzt als `fill-opacity`, nicht
  als `opacity`, damit die Trennfuge zwischen den Stücken voll deckend bleibt;
  das gewählte Stück läuft auf 100 %. Unter 75 % wird es unzulässig: bei 70 %
  fällt die Trennung benachbarter Stücke unter Normalsicht auf ΔE 14.7 und
  reisst damit die harte Grenze.
- **Geprüft, nicht geschätzt.** Gemessen wird die über der Karte
  zusammengerechnete Farbe, nicht der rohe Farbwert. Bei 78 % erreichen
  benachbarte Stücke ΔE 10.6 hell und 8.7 dunkel unter simulierter
  Rot-Grün-Schwäche (Ziel ≥ 8), unter Normalsicht 16.8 und 16.5 (harte
  Grenze 15).
- **Farbe steht nie allein — und seit der Sortierung nach Grösse erst recht
  nicht.** Die Stücke stehen nach Betrag absteigend, im Ring im Uhrzeigersinn
  und in der Legende von oben. Welche Farben dabei nebeneinander liegen, hängt
  damit von den Zahlen ab und nicht mehr von einer festen Reihenfolge; die
  frühere Zusicherung über benachbarte Stücke gilt nur noch für den Regelfall,
  nicht für jede mögliche Kombination. Über alle Paare gerechnet ist das Set
  nicht trennbar — zehn Kategorien sind mehr, als eine Palette leisten kann
  (die Methode deckelt bei acht, für „jede kann neben jeder liegen" bei drei),
  und keine andere Palette und keine Umsortierung ändert das. Getragen wird
  das Diagramm darum von den Trennfugen zwischen den Stücken, vom
  hervorgehobenen Stück samt Namen in der Ringmitte und davon, dass die
  Legende mit Name, Anteil und Betrag in derselben Reihenfolge immer neben dem
  Ring steht. Ein Diagramm, bei dem die Zuordnung allein an der Farbe hängt,
  wäre nicht zulässig.
