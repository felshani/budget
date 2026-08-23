# PRD — Persönliche Finanz-App (Schweiz)

**Version:** 2.0 · **Datum:** 16.08.2026 · **Status:** bereit zur Umsetzung

**Änderung gegenüber v1.6 — bewusste Vereinfachung:**
Einbenutzer-App, kein Sync, keine Mehrbenutzer-Logik. Investments vollständig
vom Budget entkoppelt und um eine Renditeprojektion erweitert. Topf Wohneigentum
gestrichen. Neuer Abschnitt zur Änderungsfestigkeit.

---

## 1. Ziel

Eine private App zur Kontrolle der Finanzen eines Paares, zugeschnitten auf
Schweizer Verhältnisse. Sie beantwortet drei Fragen:

1. **"Wir wollten X sparen, geworden sind es Y — warum?"**
2. **"Geht mein persönlicher Sparplan auf, und geht unser gemeinsamer auf?"**
3. **"Wo stehen unsere Investments — und wo sollten sie stehen?"**

---

## 2. Die wichtigste Architekturentscheidung: ein Benutzer

Der Vorgänger scheiterte an der Mehrbenutzer-Logik, und Änderungen brachten das
System zum Einsturz. Beides wird hier von Anfang an ausgeschlossen.

### 2.1 Zwei Personen, aber ein Benutzer

Diese Unterscheidung trägt das ganze Design:

| | |
|---|---|
| **Mehrbenutzer** — ausgeschlossen | Konten, Anmeldung, Berechtigungen, Synchronisation, Konfliktauflösung |
| **Mehrpersonen** — enthalten | Zwei Personen als Auswertungsdimension. Reine Rechnung auf denselben Daten. |

Eine Person erfasst alles auf einem Gerät. Die zweite sieht das Ergebnis, wenn
sie es sehen will. Damit entfallen Anmeldung, Rechteverwaltung, Sync und jede
Form von Konfliktauflösung — also genau der Teil, der beim Vorgänger die
Komplexität erzeugt hat.

**Konsequenz:** Kein Backend. Kein Supabase. Kein Datenbankschema. Ein einziges
JSON-Dokument im Gerät.

### 2.2 Änderungsfestigkeit

Dass Änderungen alles zerstörten, ist ein Symptom von Kopplung. Fünf Regeln
dagegen:

1. **Ein Dokument, keine Fremdschlüssel-Constraints.** Nichts kaskadiert.
2. **Kein hartes Löschen.** Ein entfernter Kostenposten wird als `aktiv: false`
   markiert. Historische Monate referenzieren ihn weiterhin, ohne zu brechen.
3. **Abgeschlossene Monate speichern Ergebnisse, keine Verweise.** Ein
   Monatsabschluss enthält seine Zahlen ausgeschrieben. Wer später eine
   Kategorie umbenennt, verändert die Vergangenheit nicht.
4. **`schemaVersion` plus Migrationsfunktionen.** Jede Version liest alle
   älteren. Die Migration läuft beim Laden, nicht in der Datenbank.
5. **Alles Nutzerdefinierte ist Daten, nicht Code.** Kategorien, Personen,
   Konten, Investments, Töpfe — nichts davon steht im Programm. Eine neue
   Kostenstelle ist ein Eintrag, kein Release.

---

## 3. Die drei Bereiche

| Bereich | Frage | Verbunden mit |
|---|---|---|
| **Budget** | Wie viel sollte übrig bleiben? | liefert den Sollwert |
| **Konten** | Wo liegt das Geld heute? | liefert den Istwert |
| **Investment** | Wo stehen die Anlagen, wo sollten sie stehen? | **entkoppelt** |

Budget und Konten bilden zusammen die Soll/Ist-Brücke. Investment steht daneben
und hat eine eigene Logik — siehe Abschnitt 7.

---

## 4. Bereich Budget

### 4.1 Zwei Achsen statt einer Typenspalte

Jeder Kostenposten hat **Kostenart** und **Träger** als getrennte Felder.

**Kostenart** — beschreibt das Zahlungsverhalten:

| Kürzel | Name | Betrag | Zeitpunkt | Beispiele |
|---|---|---|---|---|
| **D** | Dauerauftrag | fix | regelmässig | Miete, Krankenkasse, ÖV-Abo, Säule-3a-Einzahlung |
| **R** | Rückstellung | bekannt oder schätzbar | unregelmässig, meist jährlich | Steuern, Hausrat, Serafe, Autoversicherung |
| **V** | Variabel | schwankend | laufend | Lebensmittel, Ladestrom, Kleider, Coiffeur |

**Träger** — wer trägt es: Haushalt, Ferien, Person 1, Person 2 oder ein neuer
Topf.

Zuordnung der bisherigen Excel-Kürzel:

| Excel | → Kostenart | → Träger |
|---|---|---|
| D | D | aus Spalte "Abrechnung Konto" |
| R | R | aus Spalte "Abrechnung Konto" |
| H | V | Haushalt |
| PA1 / PA2 | V | Person 1 / Person 2 |
| S | *kein Kostenposten* | wird zum Sparziel |

### 4.2 Warum die Kostenart zählt

Sie bestimmt, was monatlich abgefragt wird:

| Art | Ist-Erfassung |
|---|---|
| D | keine. Ist gleich Soll. Ändert sich der Betrag, ändert man den Budgetposten |
| R | nur im Fälligkeitsmonat, Betrag durch die Rechnung bekannt |
| V | hier liegt die Arbeit und die Erkenntnis |

Bei eurer Kostenliste sind das rund acht V-Posten pro Monat.

### 4.3 Zwei Sichten auf dasselbe Budget

| Sicht | Rechnung | Wofür |
|---|---|---|
| **Fälligkeitssicht** | Kosten im Monat der tatsächlichen Zahlung | die Soll/Ist-Brücke |
| **Durchschnittssicht** | Jahreskosten geteilt durch zwölf | Beitrag an die Töpfe, Jahresplanung |

Beide Sichten werden gebraucht, weil die Steuerrechnung im März rausgeht und
nicht in zwölf Raten.

Welche Zahl als Sparbetrag stimmt, hängt davon ab, welche Konten gemeint sind:

- **Nur die privaten Konten** — was am Monatsende auf dem eigenen Konto liegen
  bleibt: Lohn minus eigene Kosten minus die feste Topfeinzahlung. Diese Zahl
  ist jeden Monat gleich hoch, ausser im Monat des 13. Monatslohns.
- **Alle Konten samt Topfkonten** — die Zahl der Fälligkeitssicht. Sie schwankt,
  weil die Topfkonten sich füllen und im Zahlungsmonat wieder leeren.

Die beiden unterscheiden sich in jedem Monat um genau die Veränderung der
Topfsaldi; über ein Jahr sind sie identisch. Welche davon die Übersicht als
Hauptzahl zeigt, ist eine Frage der Darstellung und hier nicht festgelegt. Für
die Soll/Ist-Brücke in 6.3 wird die Fälligkeitssicht verwendet, weil dort gegen
die Veränderung **aller** Kontosaldi gemessen wird.

### 4.4 Weitere Bestandteile

- **Einnahmen** je Person: Nettolohn, 13. Monatslohn mit Auszahlmonat, Bonus,
  Familienzulagen, Nebenerwerb
- **Gültigkeitsperioden** je Posten, damit eine Mietzinserhöhung im März die
  Januar-Auswertung nicht rückwirkend verfälscht
- **Neue Kostenstellen** jederzeit anlegbar: Bezeichnung, Gruppe, Betrag,
  Kostenart, Träger, Rhythmus. Auch neue Gruppen.
- **Akontozahlungen** (Heiz- und Nebenkosten) als D führen, die
  Schlussabrechnung im Zahlungsmonat als Sonderposten in beide Richtungen

---

## 5. Töpfe und Beiträge

### 5.1 Betriebsmodell

Gemeinsame Töpfe mit eigenem Konto: **Haushalt** und **Ferien**. Beide zahlen
monatlich einen Betrag ein, davon laufen die jeweiligen Kosten. Individuelle
Kosten laufen über die eigenen Konten. Weitere Töpfe sind anlegbar.

### 5.2 Die App rechnet den Beitrag aus

Aus den Kosten eines Topfes und dem Verteilschlüssel ergibt sich, wie viel jede
Person monatlich einzahlt. Verwendet wird der **geglättete** Monatsbetrag, also
inklusive Rückstellungen für Steuern und Jahresprämien — nicht nur die Kosten,
die diesen Monat zufällig fällig sind.

### 5.3 Verteilschlüssel

Standard: **nach Einkommensanteil**. Die App berechnet den Vorschlag und zeigt
ihn an, schreibt ihn aber nicht vor.

| Modus | Verhalten |
|---|---|
| Nach Einkommensanteil | App berechnet, Nutzer kann übersteuern |
| Hälftig | fix 50/50 |
| Frei | Prozentsätze manuell, Summe 100 |

Bei einer Lohnänderung wird der Schlüssel **nicht automatisch überschrieben**.
Die App meldet die Verschiebung und wartet auf Bestätigung, sonst verändern sich
rückwirkend Auswertungen, ohne dass jemand etwas entschieden hat. Der Schlüssel
gilt je Topf und ist pro Posten überschreibbar.

### 5.4 Der Topfsaldo als Frühwarnsystem

Verhält sich je Topf unterschiedlich, und das ist die Kennzahl:

| Topf | Erwartetes Verhalten | Warnung bei |
|---|---|---|
| Haushalt | pendelt um einen Sockelbetrag | dauerhaftem Drift in eine Richtung |
| Ferien | wächst bis zur Reise, fällt dann | Unterdeckung vor dem gebuchten Datum |

Driftet der Haushaltssaldo über mehrere Monate, stimmt entweder das Budget nicht
oder die Beiträge sind falsch bemessen. Die App zeigt den Trend und schlägt eine
Anpassung mit konkretem Betrag vor.

---

## 6. Bereich Konten und die Brücke

### 6.1 Konten

Frei anlegbar: Bezeichnung, Typ (Giro, Spar, Bargeld, Fremdwährung), Währung,
Inhaber (Person 1, Person 2 oder ein Topf). Keine Kreditkarten im Einsatz.

**Vollständigkeit ist Pflicht.** Fehlt ein Konto, taucht das dort geparkte Geld
als unerklärte Differenz auf.

### 6.2 Nur Bestände, keine Geldflüsse

Es werden ausschliesslich **Saldi zu Stichtagen** erfasst, ein Feld je Konto,
vorbelegt mit dem Vormonatswert. Überweisungen zwischen eigenen Konten werden
nie erfasst — sie kürzen sich in der Gesamtrechnung von selbst weg und wären nur
eine zusätzliche Fehlerquelle.

### 6.3 Die Soll/Ist-Brücke

```
  Geplanter Sparbetrag           2'000.00   Budget, Fälligkeitssicht
- Abweichung variable Kosten       450.00   Ist minus Soll je V-Posten
- Sonderposten                     350.00   ausserhalb des Budgets
- Zusatzeinzahlungen Investment    000.00   siehe 7.4
- Unerklärte Differenz             200.00   Restposten, errechnet
─────────────────────────────────────────
= Veränderung aller Kontosaldi   1'000.00   gemessen
```

Vier Erklärungsstufen, jede spezifischer als die vorige. Die unerklärte Differenz
ist ein Ergebnis, kein Eingabefeld.

### 6.4 Zwei unabhängige Messungen

Der eigentliche Wert der App: Sie misst dasselbe auf zwei Wegen, die nichts
miteinander zu tun haben — von unten die Veränderung der Kontosaldi, von oben
Budget minus erfasste Abweichungen. Treffen sie sich, sind die Zahlen belastbar.
Klaffen sie auseinander, fehlt etwas. Eine einzelne Messung könnte das nie
aufdecken.

### 6.5 Vier Ebenen

Die Rechnung läuft je Träger über dessen Konten: **Person 1**, **Person 2**,
**Haushalt**, **Ferien** — und **gesamt**. Auf der Gesamtebene kürzen sich die
internen Beiträge weg, weshalb sie auch dann stimmt, wenn die Zuordnung auf
Personenebene einmal unsauber ist.

---

## 7. Bereich Investment — entkoppelt

Investments sind bewusst **kein Teil der Brücke**. Die monatliche
3a-Einzahlung ist bereits ein D-Kostenposten im Budget; das Geld verlässt das
Konto als geplante Ausgabe. Der Investmentbereich beantwortet eine andere Frage:
*Wo stehen wir, und wo sollten wir stehen?*

### 7.1 Je Investition

| Feld | Beispiel |
|---|---|
| Bezeichnung, Anbieter | VIAC Säule 3a |
| Typ | Säule 3a, Fonds, Depot, Krypto |
| Inhaber | Person 1 oder Person 2 |
| Aktuell investiertes Kapital | Startwert bei Einrichtung |
| Monatliche Einzahlung | aus dem Budget übernommen oder eigen |
| Erwartete Rendite p.a. | Annahme, frei setzbar |

### 7.2 Sollwert aus der Renditeannahme

Aus Startkapital, monatlicher Einzahlung und erwarteter Rendite projiziert die
App den Sollwert für jeden künftigen Monat — die klassische Sparplanrechnung mit
monatlicher Verzinsung.

Dieser **Sollwert ist etwas anderes als das Soll im Budget**. Im Budget heisst
Soll "geplante Ausgabe". Hier heisst es "erwarteter Depotwert bei angenommener
Rendite". Die App benennt das getrennt, um Verwechslung zu vermeiden.

### 7.3 Ist gegen Soll

Der tatsächliche Wert wird periodisch erfasst — monatlich oder seltener, das ist
kein Pflichtfeld im Monatsablauf. Gegenübergestellt werden:

- kumulierte Einzahlungen (was du hineingegeben hast)
- Ist-Wert (was es heute ist)
- Soll-Wert (was es bei angenommener Rendite sein müsste)
- effektive Rendite gegen angenommene Rendite

Bei Säule 3a zusätzlich der Fortschritt gegen das Jahresmaximum **je Person**,
mit Hinweis, wenn gegen Jahresende Spielraum offen bleibt.

### 7.4 Zusatzeinzahlungen — die einzige Verbindung zur Brücke

Eine ausserplanmässige Einzahlung verlässt das Konto und erscheint in der Brücke
als Sparlücke, obwohl nichts ausgegeben wurde — das Geld wurde nur verschoben.

Deshalb: Wer im Investmentbereich eine Zusatzeinzahlung erfasst, bekommt sie im
Monatsabschluss automatisch als Erklärungsposten angeboten. Eine Eingabe, zwei
Wirkungen. Sie zählt nicht als Kosten, sondern als Umbuchung.

---

## 8. Monatsablauf

1. **Kontosaldi erfassen** — vorbelegt mit dem Vormonatswert
2. **Ist-Kosten erfassen** — nur V-Posten und fällige R-Posten, mit dem
   Budgetwert vorbelegt. Du korrigierst nur, was abgewichen ist.
3. **Brücke ansehen** — gesamt und je Ebene
4. **Sonderposten** erfassen, bis die unerklärte Differenz plausibel ist
5. **Monat abschliessen**, Sicherung anbieten

Angestrebte Dauer: unter fünf Minuten. Etwa fünfzehn Zahlen, die meisten nur
bestätigt.

**Ehrliche Einschränkung:** Ist-Werte pro Kategorie setzen voraus, dass du sie
kennst — in der Praxis ein Blick ins E-Banking. Wer das in einem Monat nicht
will, lässt alle V-Posten stehen und schaut nur auf die unerklärte Differenz.
Die App funktioniert dann auch, nur gröber. Der Detailgrad ist eine Entscheidung
pro Monat, keine Einstellung.

Der Investmentbereich ist **nicht** Teil dieses Ablaufs. Werte dort werden
erfasst, wann es passt.

---

## 9. Auswertung

- Wasserfall der Brücke, umschaltbar zwischen Gesamt, Person 1, Person 2,
  Haushalt und Ferien
- Jahresverlauf: geplantes gegen effektives Sparen, kumuliert
- Budgettreue je Kategorie über mehrere Monate, mit konkretem
  Anpassungsvorschlag bei wiederkehrender Abweichung
- Qualität der Erfassung: Verlauf der unerklärten Differenz
- Topfsaldi mit Drift-Warnung
- Investment: Ist gegen Soll gegen Einzahlungen, je Anlage und gesamt

---

## 10. Plattform und Datenhaltung

| Aspekt | Entscheid |
|---|---|
| Gerät | ein iPhone |
| Technologie | Progressive Web App, Single-Page |
| Installation | Safari → Teilen → "Zum Home-Bildschirm" |
| Offline | vollständig |
| Hosting | statischer Gratis-Host mit HTTPS |
| Backend | keines |

### Speicherverhalten

| | Verhalten |
|---|---|
| Während der Nutzung | jede Eingabe sofort und lautlos im Gerät gespeichert, kein Speichern-Button |
| Nach dem Monatsabschluss | "Monat abgeschlossen — Sicherung erstellen?" Ein Tipp, dann iCloud Drive |
| Beim Öffnen | Statuszeile mit dem Datum der letzten Sicherung, deutlicher Hinweis nach über 40 Tagen |
| Wiederherstellung | Datei laden, Vorschau, dann überschreiben |

Lautloses Schreiben in eine feste lokale Datei ist auf iOS nicht möglich; iCloud
Drive hat keine Schnittstelle für Web-Apps. Bei einem Monatsrhythmus ist der
bewusste Export kein Reibungspunkt, sondern ein natürlicher Abschluss.

Wird das Icon vom Home-Bildschirm gelöscht, sind die Daten weg — der Hauptgrund
für die Sicherung.

---

## 11. Schweiz-Spezifika

Währung CHF im Format 1'234.55, Rundung auf 0.05, Deutsch, Datumsformat
TT.MM.JJJJ.

Kategorienbaum wird aus der bestehenden Budgetplanung übernommen: Wohnkosten,
Energie und Kommunikation, Steuern, Versicherungen und Vorsorge, öffentlicher
Verkehr, Auto, Verschiedenes, Haushalt, persönliche Ausgaben, Rückstellungen.

**Wichtig:** Säule-3a-Maximum, Serafe-Gebühr, Franchisestufen und
Selbstbehalt-Höchstbetrag ändern sich periodisch. Sie stehen **nicht im Code**,
sondern sind Stammdaten mit Jahresgültigkeit und überschreibbar. Beim Anlegen
eines neuen Jahres fragt die App nach den aktuellen Werten.

---

## 12. Datenmodell

```json
{
  "schemaVersion": 1,
  "geaendert": "2026-09-01T10:00:00Z",

  "einstellungen": {
    "waehrung": "CHF",
    "startmonat": "2026-09"
  },
  "stammdaten": {
    "2026": { "saeule3aMax": null, "serafeJahr": null, "franchise": null }
  },

  "personen": [
    { "id": "p1", "name": "", "aktiv": true },
    { "id": "p2", "name": "", "aktiv": true }
  ],
  "toepfe": [
    { "id": "t1", "name": "Haushalt", "sockel": 0, "aktiv": true,
      "verteilschluessel": { "art": "nachEinkommen",
                             "anteile": { "p1": 53, "p2": 47 },
                             "bestaetigtAm": "2026-09-01" } },
    { "id": "t2", "name": "Ferien", "sockel": 0, "aktiv": true,
      "verteilschluessel": { "art": "haelftig", "anteile": {},
                             "bestaetigtAm": "2026-09-01" } }
  ],

  "kategorien": [
    { "id": "k1", "name": "Nahrungsmittel, Getränke",
      "gruppe": "Haushalt", "aktiv": true }
  ],
  "budgetposten": [
    { "id": "b1", "kategorieId": "k1", "betrag": 600,
      "kostenart": "V", "traeger": "t1",
      "rhythmus": "monatlich", "faelligMonate": [],
      "verteilschluessel": null,
      "gueltigAb": "2026-09", "gueltigBis": null, "aktiv": true }
  ],
  "einnahmen": [
    { "id": "n1", "personId": "p1", "art": "nettolohn", "betrag": 0,
      "rhythmus": "monatlich", "faelligMonate": [],
      "gueltigAb": "2026-09", "gueltigBis": null, "aktiv": true }
  ],

  "konten": [
    { "id": "a1", "name": "Lohnkonto", "typ": "giro", "waehrung": "CHF",
      "inhaber": "p1", "aktiv": true }
  ],
  "kontostaende": [
    { "id": "s1", "kontoId": "a1", "stichtag": "2026-09-01",
      "betrag": 0, "kurs": 1 }
  ],

  "investments": [
    { "id": "i1", "name": "VIAC Säule 3a", "anbieter": "VIAC",
      "typ": "saeule3a", "waehrung": "CHF", "inhaber": "p1",
      "startkapital": 0, "startdatum": "2026-09-01",
      "monatlicheEinzahlung": 600, "renditeErwartetProJahr": 4.0,
      "aktiv": true }
  ],
  "investmentstaende": [
    { "id": "v1", "investmentId": "i1", "stichtag": "2026-09-30",
      "wert": 0, "kurs": 1 }
  ],
  "zusatzeinzahlungen": [
    { "id": "z1", "investmentId": "i1", "datum": "2026-11-15",
      "betrag": 0, "quelleKontoId": "a1" }
  ],

  "istwerte": [
    { "id": "w1", "monat": "2026-09", "budgetpostenId": "b1",
      "betrag": 600, "bestaetigt": false }
  ],
  "sonderposten": [
    { "id": "sp1", "monat": "2026-09", "betrag": 0, "bezeichnung": "",
      "kategorieId": "k1", "traeger": "t1", "art": "ausgabe" }
  ],
  "monatsabschluss": [
    { "monat": "2026-09", "abgeschlossen": false, "notiz": "",
      "ergebnisse": {
        "gesamt": { "geplant": 0, "abweichungV": 0, "sonderposten": 0,
                    "umbuchungen": 0, "unerklaert": 0, "effektiv": 0 }
      }
    }
  ]
}
```

### Bewusste Entscheide

- **Keine Buchungs- oder Überweisungstabelle.** Die wichtigste Eigenschaft des
  Modells.
- **`aktiv` überall statt Löschen.** Historische Monate brechen nie.
- **`ergebnisse` ausgeschrieben im Abschluss.** Die Vergangenheit ist eingefroren
  und hängt nicht an heutigen Kategorienamen.
- **`kostenart` und `traeger` getrennt.** Ersetzt die vermischte Typenspalte der
  Excel.
- **`traeger` nimmt eine Personen- oder eine Topf-ID.** Nicht auf zwei Personen
  oder zwei Töpfe verdrahtet.
- **`istwerte` als eigene Tabelle.** Ein Budgetposten gilt über Jahre, ein
  Ist-Wert gehört zu einem Monat.
- **`bestaetigt`** unterscheidet "vorbelegt und unangetastet" von "aktiv
  bestätigt" — macht sichtbar, wie belastbar ein Monat ist.
- **Investments mit eigener Zeitachse.** `investmentstaende` sind nicht an den
  Monatsabschluss gebunden.
- **`renditeErwartetProJahr` ist eine Annahme, kein Versprechen.** Die App weist
  sie als solche aus.

---

## 13. Nicht-funktionale Anforderungen

- Start bis zur Bedienbarkeit unter 2 Sekunden
- vollständig offline nutzbar
- Bedienung mit einer Hand, numerische Tastatur bei Betragsfeldern
- Dunkelmodus
- keine Datenübertragung an Dritte
- überschaubare Codebasis ohne schweres Framework-Gerüst

---

## 14. Nicht im Umfang

Bankanbindung, Belegfotos, vollständige Ausgabenerfassung, Anmeldung und
Benutzerkonten, Synchronisation zwischen Geräten, Steuererklärung,
Echtzeitkurse, Topf Wohneigentum.

---

## 15. Meilensteine

**M0 — Machbarkeit**
PWA-Gerüst auf dem iPhone installieren, Speicher und Export nach iCloud Drive
testen. Unabhängig von allem anderen.

**M1 — Stammdaten**
Personen, Töpfe, Kategorien, Konten, Investments anlegen. Kostenliste aus der
Excel übernehmen. Speichern, Export, Import.

**M2 — Budget**
Einnahmen, Kostenposten mit Kostenart und Träger, Rhythmus, Rückstellung,
Verteilschlüssel, Beitragsberechnung. Ergibt den geplanten Sparbetrag je Ebene.

**M3 — Erfassung und Brücke**
Monatliche Saldi, Ist-Werte, Sonderposten, Rechenlogik, Monatsabschluss.
Der Kern.

**M4 — Investment**
Anlagen, Renditeprojektion, Ist gegen Soll, Zusatzeinzahlungen,
3a-Jahresfortschritt.

**M5 — Auswertung und Feinschliff**
Wasserfall, Jahresverlauf, Drift-Warnungen, Dunkelmodus, Zugriffsschutz.

---

## 16. Offen

| # | Frage | Wann nötig |
|---|---|---|
| 1 | Startmonat: September 2026 oder rückwirkend August? | vor M1 |
| 2 | Anfangsbestände: Saldo je Konto und Kapital je Investment am Stichtag | vor M3 |
| 3 | Liste der Konten und Investments mit Inhaber | vor M1 |
| 4 | Erwartete Rendite je Investment | vor M4, änderbar |
| 5 | Sockelbeträge Haushalts- und Ferienkonto | jederzeit |
| 6 | Zugriffsschutz mit PIN oder Face ID | vor M5 |
| 7 | Hosting: bestehender Account oder Vorschlag? | vor M0 |
