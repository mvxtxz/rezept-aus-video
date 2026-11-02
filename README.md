# Rezept aus Video

Ein Claude-Skill, der aus einem Instagram-Reel oder TikTok-Kochvideo ein Rezept macht, dem man beim Einkaufen vertrauen kann: Zutaten mit Mengen, Schritte, Portionen sowie geschätzte kcal, Protein und Kosten pro Portion. Die Ausgabe gibt es zweimal, als Rezeptkarte zum Lesen und als JSON für den Import in eine Koch-Wochenplaner-App, die ich gerade baue.

## Warum ich ihn gebaut habe

Im Alltag ist man oft im Stress, und abends überfordert mich die fast endlose Auswahl an Rezepten, die ich mir gespeichert habe. Deshalb will ich mein Online-Kochbuch in Form von Videos hochladen und mir daraus eine Liste mit Abendessen für die Woche erstellen lassen. Dieser Skill ist der erste Schritt: Er macht aus einem Video ein Rezept, mit dem die Wochenplanung arbeiten kann. Tracking von Protein und Kosten habe ich als kleine Zusatzfunktion eingebaut.

## Wie er arbeitet

```
Input (Link, Caption oder Screenshots)
  → Quelle beschaffen (TikTok: öffentlicher oEmbed-Endpunkt; Instagram: Caption/Screenshots vom Nutzer)
  → Rezept extrahieren (deutsche Zutatennamen, metrische Einheiten, Lücken markieren)
  → Werte pro Portion schätzen (Richtwerttabelle, nachvollziehbare Rechnung)
  → Ausgabe: Rezeptkarte + JSON
```

Die wichtigste Regel: **Lücken markieren statt füllen.** Eine erfundene Menge, die plausibel aussieht, ist schlimmer als eine offene Lücke, weil man sonst falsch einkauft und die Protein- und Kostenwerte nicht mehr stimmen. Fehlt eine Menge, setzt der Skill einen üblichen Wert, markiert ihn mit `uncertain: true` und listet ihn unter „Bitte prüfen“.

Weitere Regeln:
- Bei Widerspruch zwischen Caption und Screenshot gilt der Screenshot (Creator korrigieren im Video oft die Caption).
- „Rezept in Bio“ heißt: Das Rezept ist nicht in der Caption. Der Skill fragt nach, statt aus dem Titel zu raten.
- Keine Scraper, keine Mirror-Seiten: Was die Plattform nicht offen bereitstellt, fragt der Skill beim Nutzer an.
- Basiszutaten (Salz, Öl, Wasser) laufen als Vorrat und werden nicht einzeln eingepreist.

## Wo er versagt

- **Instagram** lässt sich ohne Developer-Token nicht automatisch auslesen. Man muss Caption oder Screenshots selbst einfügen.
- **TikTok-Links** werden je nach Umgebung blockiert. Im Beispiel unten hat die Plattform den Abruf abgelehnt; der Skill hat dann wie vorgesehen nach Screenshots gefragt, statt einen Umweg zu suchen.
- **Gesprochene Mengen** erkennt er nicht. Er liest Caption und Bildtext, nicht den Ton des Videos.
- **Nährwerte und Kosten** sind Schätzungen aus gerundeten Richtwerten und typischen Supermarktpreisen, keine Laborwerte und keine Live-Preise. Hängen sie an einer unsicheren Menge (etwa fehlende Fleischmenge), steht das dabei.
- **Portionen** sind oft nicht genannt. Dann schätzt er sie aus den Mengen und markiert das.

## Aufbau

```
skill/
  SKILL.md                  Anweisungen für Claude (Ablauf, Regeln, Ausgabeformat)
  references/richtwerte.md  Nährwerte pro 100 g und Umrechnungstabelle
```

## Nutzung

In Claude als Skill hinzufügen (Ordner `skill/` hochladen), dann einen TikTok-Link, eine kopierte Caption oder Screenshots schicken.

## Beispiel

Echter Lauf vom 2. November 2026. Input: TikTok-Link von @brin.pirathapan. Der automatische Abruf wurde blockiert, deshalb lief der Skill über vier Screenshots der Caption.

**Rezeptkarte (Auszug)**

| Menge | Zutat | Hinweis |
|---|---|---|
| 0,25 g | Safran | |
| 2 Stück | Knoblauchzehen | |
| 1 TL | Chiliflocken | Caption: ½–1 TL ⚠ |
| 1 Stück | Orange | nur 2 Stücke Schale verwendet |
| 1 Stück | Schalotte | |
| 200 g | Risottoreis | |
| 200 ml | Weißwein | |
| 700 ml | Hühner- oder Gemüsebrühe, heiß | Menge geschätzt ⚠ |
| 50 g | Parmesan, gerieben | |
| 600 g | Entenbrust (2 Stück) | Gewicht geschätzt ⚠ |

**Pro Portion (geschätzt):** ~1.200 kcal · ~73 g Protein · ~9,40 €

**Bitte prüfen:**
- Die Brühmenge steht nicht in der Caption. Angenommen: 700 ml, die übliche Menge für 200 g Risottoreis.
- Das Gewicht der Entenbrust fehlt. Angenommen: 2 × 300 g. Kalorien, Protein und vor allem die Kosten (Ente ca. 12 € von 18,80 €) hängen stark daran.
- Die Portionen sind nicht genannt, 2 geschätzt aus 200 g Reis und 2 Entenbrüsten.
- Chiliflocken: Die Caption nennt ½–1 TL, Schritt 2 nennt 1 TL.

Das vollständige Ergebnis mit allen zwölf Schritten und dem JSON liegt in [`beispiele/safran-risotto-ente.md`](beispiele/safran-risotto-ente.md).

## Status

Der Skill läuft und ich nutze ihn selbst. Die Koch-Wochenplaner-App, in die das JSON importiert wird, baue ich gerade mit Base44.
