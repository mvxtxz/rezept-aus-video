---
name: rezept-aus-video
description: Macht aus einem Instagram-Reel oder TikTok-Kochvideo ein strukturiertes Rezept (Zutaten mit Mengen, Schritte, Portionen) plus geschätzte Kosten, kcal und Protein pro Portion, als Rezeptkarte und als JSON für den Koch-Wochenplaner. Verwende diesen Skill immer, wenn jemand einen Instagram- oder TikTok-Link, eine kopierte Video-Caption oder Screenshots aus einem Kochvideo schickt und daraus ein Rezept, eine Einkaufsliste oder Nährwerte haben will, auch wenn das Wort "Rezept" nicht fällt (z. B. "was brauch ich dafür?", "kannst du das für 2 Leute umrechnen?").
---

# Rezept aus Video

Ziel: Aus einem Kochvideo wird ein Rezept, dem man beim Einkaufen und Kochen vertrauen kann. Vertrauen ist hier wichtiger als Vollständigkeit. Eine erfundene Menge, die plausibel aussieht, ist schlimmer als eine offen markierte Lücke: Die Person kauft sonst falsch ein, und Protein- und Kostenwerte stimmen nicht mehr.

## 1. Quelle beschaffen

Je nach Eingabe gehst du so vor:

**TikTok-Link:** Rufe mit WebFetch den öffentlichen oEmbed-Endpunkt auf:
`https://www.tiktok.com/oembed?url=<Video-URL, URL-kodiert>`
Das Feld `title` enthält die Caption, `author_name` den Creator. Kurzlinks (vm.tiktok.com) zuerst so probieren. Wenn der Abruf scheitert, abgelehnt wird oder die Caption keine Zutaten enthält, bitte um die Caption oder Screenshots (siehe unten). Versuche keinen anderen Weg (kein curl, keine Scraper-Dienste, keine Mirror-Seiten): Die Plattformen erlauben das nicht, und es bricht ohnehin ständig.

**Instagram-Link:** Ohne Meta-Developer-Token gibt es keinen offenen Zugriff. Rufe den Link nicht ab, sondern bitte direkt um Caption und/oder Screenshots. Eine kurze Bitte reicht, zum Beispiel: "Instagram lässt sich nicht automatisch auslesen. Kopier mir bitte die Caption hier rein, und wenn Mengen nur im Video stehen, schick 2–3 Screenshots davon."

**Caption-Text oder Screenshots:** Direkt verwenden. Bei Screenshots die eingeblendeten Texte und erkennbaren Zutaten auswerten.

Liegen Caption und Screenshots vor, gelten Mengen aus den Screenshots vorrangig. Creator korrigieren im Video oft, was in der Caption steht.

"Rezept in Bio", "Link im Profil" oder Ähnliches heißt, die Caption enthält das Rezept nicht. Sag das und frag nach Screenshots oder dem verlinkten Rezepttext, statt ein Rezept aus dem Titel zu raten.

## 2. Rezept extrahieren

- **Zutaten** auf Deutsch benennen, Mengen in metrischen Einheiten (g, ml, Stück, EL, TL). US-Einheiten umrechnen (Tabelle in `references/richtwerte.md`) und die Originalangabe in `note` behalten.
- **Lücken markieren statt füllen:** Fehlt eine Menge, schätze einen üblichen Wert, setze `uncertain: true` und nenne den Punkt unter "Bitte prüfen". "Eine Handvoll", "nach Gefühl" und Ähnliches ebenfalls als unsicher markieren.
- **Portionen:** Nur übernehmen, wenn genannt. Sonst aus den Mengen schätzen, als unsicher markieren und erwähnen.
- **Basiszutaten** (Salz, Pfeffer, Öl, Wasser) aufnehmen, aber bei Kosten unter "Vorrat" führen statt einzeln einzupreisen.
- **Schritte** knapp, in Reihenfolge, mit Zeiten und Temperaturen, wenn genannt. Keine Schritte erfinden, die im Video nicht vorkommen. Ein offensichtlicher Zwischenschritt (z. B. "Nudeln kochen") darf rein.

## 3. Werte pro Portion schätzen

Alle Werte sind Schätzungen, und das muss sichtbar bleiben. Die Person nutzt sie für Protein-Tracking und Budget, daher lieber eine nachvollziehbare Rechnung als eine runde Zahl ohne Herkunft.

- **kcal und Protein:** Pro Zutat aus `references/richtwerte.md` rechnen (für fehlende Zutaten übliche Werte aus deinem Wissen), summieren, durch Portionen teilen. Protein auf ganze Gramm, kcal auf 10 runden.
- **Kosten:** Typische Preise im deutschen Supermarkt/Discounter, anteilig nach verwendeter Menge (nicht ganze Packungen). Auf 0,10 € runden. Wenn die Person eigene Preise genannt hat, haben die Vorrang.
- Hängen die Werte stark an unsicheren Mengen (z. B. Fleischmenge fehlt), schreib das dazu.

## 4. Ausgabe

Immer beide Teile, in dieser Reihenfolge:

**Teil A: Rezeptkarte (für Menschen)**

```
## <Titel>
Quelle: <Plattform> · @<Creator> · <Link>
<Portionen> Portionen · ca. <Zeit> Min

| Menge | Zutat | Hinweis |
|---|---|---|
| 400 g | Hähnchenbrust | |
| 2 EL | Gochujang | Menge geschätzt ⚠ |

1. ...
2. ...

**Pro Portion (geschätzt):** ~<kcal> kcal · ~<Protein> g Protein · ~<Kosten> €

**Bitte prüfen:** <unsichere Punkte, je ein kurzer Satz; leer lassen = "Nichts offen.">
```

**Teil B: JSON (für den Import in die App)** als einzelner ```json-Block:

```json
{
  "title": "Gochujang-Hähnchen mit Reis",
  "source": {"platform": "tiktok", "url": "https://...", "creator": "name", "input": ["oembed_caption"]},
  "servings": 2,
  "servings_uncertain": false,
  "time_min": 25,
  "ingredients": [
    {"name": "Hähnchenbrust", "amount": 400, "unit": "g", "note": "", "uncertain": false, "pantry": false}
  ],
  "steps": ["..."],
  "per_serving": {"kcal": 640, "protein_g": 42, "cost_eur": 3.1, "is_estimate": true},
  "open_questions": ["Gochujang-Menge nicht genannt, 2 EL angenommen"]
}
```

Feldregeln: `platform` ist `tiktok`, `instagram` oder `unbekannt`. `input` listet, was genutzt wurde (`oembed_caption`, `pasted_caption`, `screenshots`). `amount` ist eine Zahl oder `null`, wenn gar nichts schätzbar ist. `unit` ist eine von `g`, `ml`, `Stück`, `EL`, `TL`, `Prise`, `Bund` oder `""`. `pantry: true` für Basiszutaten aus Schritt 2. Hat die Person nach Portionen gefragt ("für 4 Leute"), rechne Mengen und `servings` darauf um.

Nach der Ausgabe nicht nachfragen, ob alles passt. Die "Bitte prüfen"-Zeile übernimmt das bereits.
