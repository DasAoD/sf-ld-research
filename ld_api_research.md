# Legendary Dungeon – API Research

> Status: **Work in Progress** – basierend auf 12 Charakter-Runs (Räume 1–80)  
> Fehlend: Räume 80–100, Endboss, legendäre Truhe, einige Sonderfälle

---

## Endpoint

```
GET https://f{server}.sfgame.net/cmd.php?req=IADungeonInteract&params={base64}&sid={session}
```

- `{server}` = Servernummer, z.B. `f25`, `f9`
- `{base64}` = Base64-kodierter Integer (Aktionscode)
- `{session}` = Session-ID des Charakters

---

## Request: params-Werte (Aktionscodes)

| param | Bedeutung | Kontext |
|-------|-----------|---------|
| `1` | Links gehen | Navigation |
| `2` | Rechts gehen | Navigation |
| `5` | Schlüsselmeister betreten / Kiste interagieren | Interaktionsraum |
| `6` | Fass / Holzkiste öffnen (ohne Schlüssel) | Interaktionsraum |
| `20` | Kampf starten / Skelett bekämpfen | Kampf |
| `21` | Fliehen | Kampf / Schmatztruhe |
| `40` | Tür mit Schlüssel öffnen / Truhe/Kiste bestätigen | Interaktion |
| `42` | Raum verlassen ohne Aktion (Truhe nicht öffnen) | Abbruch |
| `50` | Falle / Opfertür / erleuchteten Durchgang / Jungbrunnen betreten | Navigation |
| `51` | Raum verlassen ohne Aktion (Händler ablehnen) | Abbruch |
| `60` | Kampfergebnis bestätigen | Kampf |
| `70` | Poll / Zustand abfragen / nächster Schritt | Allgemein |
| `80` | Händler: in Waren blättern (kostet 1 Pilz) | Händler |
| `91` | Schere-Stein-Papier: Papier? (unbestätigt) | SSP-Raum |
| `92` | Schere-Stein-Papier: Schere wählen | SSP-Raum |
| `93` | Schere-Stein-Papier: Stein? (unbestätigt) | SSP-Raum |

> **Noch unbekannt:** param für Schicksalstür (Glücksrad), hungrige Tør füttern, Gem-Auswahl nach Boss

---

## Response: iadungeonsave Felder

Format: `iadungeon.iadungeonsave:{f0}/{f1}/{f2}/...`

| Index | Bedeutung | Beispielwert |
|-------|-----------|--------------|
| `[0]` | Charakter-ID (konstant) | `664444180` |
| `[1]` | Dungeon-Typ (2=normal, 3=Ultimate?) | `2` |
| `[2]` | Aktuelle HP | `82426523` |
| `[3]` | Maximale HP | `139762968` |
| `[17]` | Aktuelle Raumnummer | `8` |
| `[19]` | Raumzustand (state) | `100` |
| `[22]` | Objekt/Monster im Raum | `-5085` |

> **Noch unbekannt:** viele andere Felder (Schløssel-Anzahl, aktive Segen/Fløche, Ressourcen, etc.)

---

## Response: iadungeonstats Felder

Format: `iadungeonstats:{a}/{b}/{c}/{d}/{e}`

| Index | Bedeutung |
|-------|-----------|
| `[0]` | Anzahl Segen erhalten |
| `[1]` | Anzahl Bosse besiegt |
| `[2]` | Anzahl Monster getötet |
| `[3]` | Gold gesammelt |
| `[4]` | ? (meist 2–3) |

---

## Response: state-Werte (Raumzustände)

### Aktive Raumzustände

| state | Bedeutung |
|-------|-----------|
| `1` | Monster-Raum |
| `4` | Boss-Raum (Raum 25 / 50 / 75 / 100) |
| `100` | Interaktionsraum (Truhe / Fass / Skelett / Kiste) |
| `200` | Leerer Durchgang / Tør ohne Inhalt |
| `301` | Jungbrunnen (HP-Heilung) |
| `308` | Schere-Stein-Papier-Raum |
| `310` | Erleuchteter/Goldener Durchgang |
| `315` | Schløsselmeister-Händler / Kostenloser Händler (Ebene 1) |
| `322` | Komplett leerer Raum (kein Objekt, kein Monster) |
| `323` | Fluchhändler-Raum |

### Post-Raum-Zustände (nach Abschluss eines Raums)

Diese erscheinen nach param=70 und signalisieren was nach dem Raum passiert ist.

| state | Bedeutung |
|-------|-----------|
| `1000` | Segen erhalten (aus Kiste/Fass/Jungbrunnen) |
| `1001` | Normaler Abschluss (nächster Raum wählbar) |
| `1002` | Item oder Ressourcen erhalten (eingepackt/Gold) |
| `1006` | Notausgang-Truhe geöffnet |
| `1007` | Gold erhalten (Bronzene Schatztruhe) |
| `1008` | Segen erhalten (nach Raum, z.B. Weg der Besserung) |
| `1009` | Fluch erhalten |
| `1010` | ? |
| `1011` | Lebenselixier gekauft (sofort HP + maxHP erhöht) |
| `1012` | Schaden durch aktiven Fluch (HP sinkt pro Raum) |
| `1013` | Boss besiegt + Klunker gewählt |
| `1014` | ? |
| `1015` | ? (nach Fluchhändler verlassen) |
| `1016` | ? |
| `1022` | Prüfungspforte bestanden (kein Item) |
| `1023` | Prüfungspforte bestanden (mit Item/Gold) |

---

## Response: monster-Werte (Objekte im Raum)

Gilt wenn `state=100` (Interaktionsraum).

### Schatztruhen

| monster | Bedeutung |
|---------|-----------|
| `0` | Bronzene Schatztruhe |
| `1` | Silberne Schatztruhe |
| `2` | Epische Schatztruhe |
| `500` | Gefräßige Schmatztruhe (verwandelt sich in Monster) |
| `600` | Opfertruhe |
| `601` | Verfluchte Truhe |
| `602` | Notausgang-Preis-Truhe |
| `603` | Satte Kiste (hinter hungriger Tür) |

### Kisten & Fässer

| monster | Bedeutung |
|---------|-----------|
| `100` | Holzkiste (Gold / Ressourcen) |
| `101` | Fass (mit Item) |
| `102` | Holzkiste (mit Item) |
| `400` | Fass (mit Segen/Fluch) |

### Skelette

| monster | Bedeutung |
|---------|-----------|
| `300` | Magierskelett |
| `301` | Kriegerskelett |

### Sonstiges

| monster | Bedeutung |
|---------|-----------|
| `92` | SSP-Gegner (Gleichstand = Schere) |

### Monster-Räume (state=1 oder state=4)

Negative Werte = reguläre Monster-IDs.

| monster | Bedeutung |
|---------|-----------|
| `-5002` | Monster mit Laterne (nach erleuchtetem Durchgang) |
| `-5075` bis `-5099` | Reguläre Dungeon-Monster Ebene 1 (Räume 1–24) |
| `-5106` bis `-5135` | Reguläre Dungeon-Monster Ebene 2–3 (Räume 26–80) |
| `-5142` | Boss 1 (Raum 25) |
| `-5144` | Boss 2 (Raum 50) |
| `-5146` | Boss 3 (Raum 75) |
| `-5148` | Endboss (Raum 100) |

---

## Response: iamap

Erscheint beim Tod. Format: `iamap:{boss_raum}/{boss_monster_id}/{?}/{gem_typ}/...`

Beispiel: `iamap:25/-5142/1/315/25/-5144/1/-315/25/-5146/1/-315/25/-5148/...`

- Zeigt alle 4 Bosse mit Raumnummer, Monster-ID und gewähltem/nicht gewähltem Gem
- `315` = Klunker gewählt / `-315` = kein Klunker gewählt

---

## Response: weitere Felder

| Feld | Format | Bedeutung |
|------|--------|-----------|
| `iadungeonstats` | `a/b/c/d/e` | Dungeon-Statistiken (Segen/Bosse/Monster/Gold/?) |
| `iadungeonstatstotal` | `a/b/c/d/e/f` | Gesamtstatistiken (erscheint beim Tod) |
| `iamerchant` | `count/item1/item2/...` | Händler-Angebot im aktuellen Raum |
| `ialootitem` | Item-Daten | Gefundenes Item nach Kampf/Truhe |
| `iapendingitems` | Item-Daten | Ausstehende Item-Entscheidung (einpacken/verkaufen) |
| `usedbuffs` | `buff_id1/buff_id2/...` | In diesem Raum aktive/verwendete Segen/Flüche |
| `resources` | `gold/...` | Ressourcen-Update |

---

## Segen & Flüche – bekannte usedbuffs-IDs

| buff_id | Typ | Bedeutung |
|---------|-----|-----------|
| `1` | Segen | Weg der Besserung (HP-Heilung sofort/nach Raum) |
| `2` | Segen | One Hit Wonder (Monster sofort töten) |
| `4` | ? | ? |
| `5` | Segen | Schløsselmeister-Segen? |
| `8` | Segen | Weg der Besserung (HP-Heilung nach Raum) |
| `101` | Fluch | Aktiver Fluch beim Kampf |
| `102` | Fluch | 5% Schaden pro Raum für 5 Räume |
| `104` | Fluch | 50% Gold aus Truhen für 5/10 Kammern |

---

## Prüfungspforte & Notausgang

### Prüfungspforte
- Erscheint maximal einmal pro Dungeon-Run, nie in Ebene 1
- Kein Schlüssel erforderlich
- Dahinter immer ein Monster-Raum (state=1)
- Nach Sieg: Wahl zwischen weiterer Prüfungspforte oder Notausgang
- Maximal 5 Prüfungspforten hintereinander
- Post-state: `1022` (kein Item) oder `1023` (mit Item/Gold)

### Notausgang
- Erscheint nach jeder Prüfungspforte und nach allen 5 bestanden
- Kein Schlüssel erforderlich
- Dahinter immer eine Preis-Truhe (monster=`602`)
- Bessere Belohnungen je mehr Prüfungspforten bestanden
- Post-state nach Öffnen: `1006`

---

## Schicksalsklunker (Gems of Fate)

Nach jedem besiegten Boss (Räume 25, 50, 75, 100) wählt man einen von drei Klunkern.  
Tier-Liste und Beschreibungen aus ldgadget.12hp.de + Spieler-Screenshots.

| Tier | Name (DE) | Name (EN) | Effekt + | Effekt – |
|------|-----------|-----------|----------|----------|
| S | Seele des Hasen | Soul of the Rabbit | +40% Fluchtchance aus Kämpfen | Monster verursachen 25% mehr Schaden |
| A | Verfluchter Mondstein | Cursed Moonstone | +20% Fluchtchance | Dauer von Flüchen +1 |
| A | Spionageklunker | Spying Gem | Chance auf unverschlossene Türen | -15% Schlüssel aus Kämpfen |
| A | Anhänger des Schlüsselmeisters | Pendant of the Key Master | 40% Chance nach Fliehen einen Schlüssel zu erhalten | Eine Tür ist immer verschlossen |
| B | Schmieriger Heilstein | Greasy Healing Stone | Nach Tod: Segen "Weg der Besserung" für 3 Räume | Keine epischen Truhen mehr |
| B | Glücksspielerbrocken | Boulder of the Gambler | Fallen geben Flüche statt Schaden; +50% Chance auf Segen aus Fässern | – |
| B | Brocken der Gier | Boulder of Greed | Mehr geheimnisvolle Türen; erhöhte Chance auf Segen aus Truhen/Kisten/Leichen | – |
| C | Schatz des Helden | Treasure of the Hero | Monster verursachen weniger Schaden | Geringere Chance auf Items |
| C | Diamant des Zeitreisenden | Diamond of the Time Traveler | Segen-Dauer verlängert | Fässer enthalten immer Flüche |
| C | Hoffnung des Verdurstenden | Hope of the Thirsty One | Mehr verfluchte Türen | – |
| C | Verfluchte Perle | Cursed Pearl | – | – |
| C | Blutstropfen der Opfergabe | Blood Drop of Sacrifice | Weniger Schaden durch Opfertüren; mehr Opfertüren | – |
| D | Smaragd des Forschers | Emerald of the Explorer | – | Weniger geheimnisvolle Türen |
| E | Saphir des Pechvogels | Sapphire of the Misadventurer | Weniger verfluchte Türen | – |
| E | Kronjuwel des Teufels | Crown Jewel of the Devil | Chance auf epische Türen | Hinter Türen können sich Monster verbergen |
| F | Kiesel der Hinterlist | Pebble of Deceit | Monster verursachen weniger Schaden | Hinter Türen können sich Monster verbergen |
| F | Magnetstein | Lodestone | Chance auf doppelt verschlossene Türen; erhöhte Schlüsselchance aus Kämpfen | – |
| F | Auge des Stiers | Eye of the Bull | – | – |
| F | Irrender Brocken des Tölpels | Erratic Boulder of the Hick | Weniger Opfertüren | – |
| F | Nierenstein der Zielstrebigkeit | Kidney Stone of Determination | Verfluchte Truhen auch hinter verschlossenen Türen | – |
| F | Alter Opferstein | Old Sacrifice Stone | Weniger Schaden durch Opfertruhen; Opfertruhen auch hinter verschlossenen Türen | – |

---

## Sonderbereiche

| Raum | Inhalt |
|------|--------|
| Schlüsselmeister-Shop | Segen gegen Schlüssel tauschen |
| Key to Failure Shop | Schlüssel gegen Flüche tauschen |
| Lavaraum | 10% Schaden |
| Wasserraum | Ertrinken nach 10 Sekunden |
| Erzähler-Raum | 25% Heilung + Segen |
| Sarkophag | Gold (episches Item mit Schlüssel) |
| Schere-Stein-Papier | Segen + episches Item bei Gewinn |
| Glücksrad | Segen, Fluch, Gold oder Schlüssel |
| Kanalraum | Episches Item |
| Lebensbrunnen | 25% Heilung |
| Spinne | 1/2/5 Schlüssel |
| Wunschbrunnen | Gold → episches Item oder Segen |
| Zeughaus (Räume 90–98) | Episches Item; 10% chance legendär bei 2 Waffen |

### Monster-Schadenstabelle (% der max. LD-HP)

| Bereich | Min | Avg | Max |
|---------|-----|-----|-----|
| Monster 1–24 | 10,98% | 14,24% | 17,49% |
| Boss 25 | 12,93% | 17,45% | 21,97% |
| Monster 26–49 | 14,50% | 19,00% | 23,49% |
| Boss 50 | 18,57% | 23,53% | 28,48% |
| Monster 51–74 | 14,50% | 19,00% | 23,49% |
| Boss 75 | 18,57% | 23,53% | 28,48% |
| Monster 76–99 | 17,30% | 23,65% | 30,00% |
| Boss 100 | 33,96% | 45,30% | 56,63% |

---

## Noch zu erforschen

- [ ] Schicksalstør / Glücksrad (param=?)
- [ ] Hungrige Tür füttern (param=?)
- [ ] Segenstür (param=?)
- [ ] Gem-Auswahl nach Boss (welcher param?)
- [ ] Restliche 1000er state-Werte (1010, 1014, 1015, 1016)
- [ ] SSP: param før Stein und Papier bestätigen
- [ ] iadungeonsave Felder [4]–[16], [18], [20]–[21], [23]–[50]
- [ ] Dungeon-Typ Feld [1]: 2=normal, 3=LD Ultimate?
- [ ] Bosse 2–4 komplett mit Klunker-Auswahl-Request
- [ ] Legendäre Truhe nach Endboss (Raum 100)
- [ ] Zeughaus (state=?, Räume 90–98)

---

## Quellen

- HAR-Aufzeichnungen: 12 Charakter-Runs auf verschiedenen Servern (F9, F25)
- Playa Games Helpshift: https://playa-games.helpshift.com/hc/de/4-shakes-fidget-1653988985/faq/57-legendary-dungeon/
- ldgadget.12hp.de: https://ldgadget.12hp.de
