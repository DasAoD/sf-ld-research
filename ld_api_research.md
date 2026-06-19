# Legendary Dungeon – API Research

> Status: **Work in Progress** – basierend auf 6 Charakter-Runs (Räume 1–46)  
> Fehlend: Prüfungspforte, Notausgang, Räume 50–100, viele Sonderfälle

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
| `5` | Schløsselmeister betreten / Kiste interagieren | Interaktionsraum |
| `6` | Fass / Holzkiste öffnen (ohne Schlüssel) | Interaktionsraum |
| `20` | Kampf starten / Skelett bekämpfen | Kampf |
| `21` | Fliehen | Kampf / Schmatztruhe |
| `40` | Tør mit Schløssel öffnen / Truhe/Kiste bestätigen | Interaktion |
| `42` | Raum verlassen ohne Aktion (Truhe nicht öffnen) | Abbruch |
| `50` | Falle / Opfertør / erleuchteten Durchgang betreten | Navigation |
| `51` | Raum verlassen ohne Aktion (Händler ablehnen) | Abbruch |
| `60` | Kampfergebnis bestätigen | Kampf |
| `70` | Poll / Zustand abfragen / nächster Schritt | Allgemein |
| `80` | Händler: in Waren blättern (kostet 1 Pilz) | Händler |
| `91` | Schere-Stein-Papier: Papier? (unbestätigt) | SSP-Raum |
| `92` | Schere-Stein-Papier: Schere wählen | SSP-Raum |
| `93` | Schere-Stein-Papier: Stein? (unbestätigt) | SSP-Raum |

> **Noch unbekannt:** param før Schicksalstør (Glücksrad), Prüfungspforte, Notausgang, hungrige Tør füttern, Gem-Auswahl nach Boss

---

## Response: iadungeonsave Felder

Format: `iadungeon.iadungeonsave:{f0}/{f1}/{f2}/...`

| Index | Bedeutung | Beispielwert |
|-------|-----------|--------------|
| `[0]` | Charakter-ID (konstant) | `664444180` |
| `[1]` | Dungeon-Typ (1=normal, 2=Ultimate?) | `2` |
| `[2]` | Aktuelle HP | `82426523` |
| `[3]` | Maximale HP | `139762968` |
| `[17]` | Aktuelle Raumnummer | `8` |
| `[19]` | Raumzustand (state) | `100` |
| `[22]` | Objekt/Monster im Raum | `-5085` |

> **Noch unbekannt:** viele andere Felder (Schløssel-Anzahl, aktive Segen/Fløche, Ressourcen, etc.)

---

## Response: state-Werte (Raumzustände)

### Aktive Raumzustände

| state | Bedeutung |
|-------|-----------|
| `1` | Monster-Raum |
| `4` | Boss-Raum (Raum 25 / 50 / 75 / 100) |
| `100` | Interaktionsraum (Truhe / Fass / Skelett / Kiste) |
| `200` | Leerer Raum / Durchgang |
| `301` | Jungbrunnen (HP-Heilung) |
| `308` | Schere-Stein-Papier-Raum |
| `310` | Erleuchteter/Goldener Durchgang |
| `315` | Schicksalsgem-Auswahl / Kostenloser Händler (Ebene 1) |
| `323` | Fluchhändler-Raum |

### Post-Raum-Zustände (nach Abschluss eines Raums)

Diese erscheinen nach param=70 und signalisieren was nach dem Raum passiert ist.

| state | Vermutliche Bedeutung |
|-------|----------------------|
| `1000` | Boss besiegt / Ebene abgeschlossen |
| `1001` | Normaler Abschluss (nächster Raum wählbar) |
| `1002` | Segen erhalten |
| `1006` | ? (nach epischer Truhe / Opfertruhe) |
| `1007` | ? (nach bestimmten Räumen) |
| `1008` | ? (nach bestimmten Räumen) |
| `1009` | ? |
| `1010` | ? |
| `1011` | ? |
| `1013` | ? |
| `1014` | ? |
| `1015` | ? (nach Fluchhändler verlassen) |
| `1016` | ? |

> **Noch unbekannt:** genaue Bedeutung der 1000er-Zustände

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
| `603` | Satte Kiste (hinter hungriger Tør) |

### Kisten & Fässer

| monster | Bedeutung |
|---------|-----------|
| `100` | Holzkiste (Gold / Ressourcen) |
| `101` | Fass |
| `102` | Holzkiste (mit Item) |
| `400` | Fass (Variante / anderer Typ?) |

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
| `-5075` bis `-5099` | Reguläre Dungeon-Monster (verschiedene Typen) |
| `-5142` | Boss 1 (Raum 25) |
| `-5144` | Boss 2 (Raum 50, aus iamap) |
| `-5146` | Boss 3 (Raum 75, aus iamap) |
| `-5148` | Endboss (Raum 100, aus iamap) |
| `-5240` bis `-5242` | Dungeon-spezifische Monster (Geburtstags-LD?) |

---

## Response: iamap

Erscheint beim Tod. Format: `iamap:{boss_raum}/{boss_monster_id}/{?}/{gem_typ}/...`

Beispiel: `iamap:25/-5142/1/315/25/-5144/1/-315/25/-5146/1/-315/25/-5148/...`

- Zeigt alle 4 Bosse mit Raumnummer, Monster-ID und gewähltem/nicht gewähltem Gem
- `315` = positiver Gem / `-315` = kein/negativer Gem

---

## Response: weitere Felder

| Feld | Format | Bedeutung |
|------|--------|-----------|
| `iadungeonstats` | `a/b/c/d/e` | Dungeon-Statistiken (Segen/Bosse/Monster/Gold/?) |
| `iadungeonstatstotal` | `a/b/c/d/e/f` | Gesamtstatistiken (erscheint beim Tod) |
| `iamerchant` | `count/item1/item2/...` | Händler-Angebot im aktuellen Raum |
| `ialootitem` | Item-Daten | Gefundenes Item nach Kampf/Truhe |
| `iapendingitems` | Item-Daten | Ausstehende Item-Entscheidung (einpacken/verkaufen) |
| `usedbuffs` | `buff_id1/buff_id2/...` | In diesem Raum aktive/verwendete Segen |
| `resources` | `gold/...` | Ressourcen-Update |

---

## Schicksalsklunker (Gems of Fate)

Nach jedem besiegten Boss (Räume 25, 50, 75, 100) wählt man einen von drei Klunkern.  
Tier-Liste und Beschreibungen aus ldgadget.12hp.de + Spieler-Screenshots.

| Tier | Name (DE) | Name (EN) | Effekt + | Effekt – |
|------|-----------|-----------|----------|----------|
| S | Seele des Hasen | Soul of the Rabbit | +40% Fluchtverschance aus Kämpfen | Monster verursachen 25% mehr Schaden |
| A | Verfluchter Mondstein | Cursed Moonstone | +20% Fluchtverschance | Dauer von Fløchen +1 |
| A | Spionageklunker | Spying Gem | Chance auf unverschlossene Türen | -15% Schløssel aus Kämpfen |
| A | Anhänger des Schløsselmeisters | Pendant of the Key Master | 40% Chance nach Fliehen einen Schløssel zu erhalten | Eine Tør ist immer verschlossen |
| B | Schmieriger Heilstein | Greasy Healing Stone | Nach Tod: Segen "Weg der Besserung" für 3 Räume | Keine epischen Truhen mehr |
| B | Gløcksspielerbrocken | Boulder of the Gambler | Fallen geben Flüche statt Schaden | – |
| B | Brocken der Gier | Boulder of Greed | Mehr geheimnisvolle Türen; erhöhte Chance auf Segen aus Truhen/Kisten/Leichen | – |
| C | Schatz des Helden | Treasure of the Hero | Monster verursachen weniger Schaden | Geringere Chance auf Items |
| C | Diamant des Zeitreisenden | Diamond of the Time Traveler | Segen-Dauer verlängert | Fässer enthalten immer Flüche |
| C | Hoffnung des Verdurstenden | Hope of the Thirsty One | Mehr verfluchte Türen | – |
| C | Verfluchte Perle | Cursed Pearl | – | – |
| C | Blutstropfen der Opfergabe | Blood Drop of Sacrifice | Weniger Schaden durch Opfertøren; mehr Opfertøren | – |
| D | Smaragd des Forschers | Emerald of the Explorer | – | Weniger geheimnisvolle Türen |
| E | Saphir des Pechvogels | Sapphire of the Misadventurer | Weniger verfluchte Türen | – |
| E | Kronjuwel des Teufels | Crown Jewel of the Devil | Chance auf epische Türen | Hinter Tøren können sich Monster verbergen |
| F | Kiesel der Hinterlist | Pebble of Deceit | Monster verursachen weniger Schaden | Hinter Türen können sich Monster verbergen |
| F | Magnetstein | Lodestone | Chance auf doppelt verschlossene Türen; erhöhte Schløsselchance aus Kämpfen | – |
| F | Auge des Stiers | Eye of the Bull | – | – |
| F | Irrender Brocken des Tölpels | Erratic Boulder of the Hick | Weniger Opfertøren | – |
| F | Nierenstein der Zielstrebigkeit | Kidney Stone of Determination | Verfluchte Truhen auch hinter verschlossenen Tøren | – |
| F | Alter Opferstein | Old Sacrifice Stone | Weniger Schaden durch Opfertruhen; Opfertruhen auch hinter verschlossenen Tøren | – |

---

## Segen (Blessings) – bekannte usedbuffs-IDs

| buff_id | Vermutliche Bedeutung |
|---------|-----------------------|
| `1` | Weg der Besserung (HP-Heilung pro Raum) |
| `2` | One Hit Wonder |
| `4` | ? |
| `5` | Schløsselmeister-Segen? |
| `8` | Weg der Besserung (HP-Heilung nach Raum) |
| `101` | ? |
| `102` | ? (evtl. Fluch) |
| `104` | ? |

---

## Sonderbereiche

| Raum | Inhalt |
|------|--------|
| Schløsselmeister-Shop | Segen gegen Schløssel tauschen |
| Key to Failure Shop | Schløssel gegen Flüche tauschen |
| Lavaraum | 10% Schaden |
| WAsserraum | Ertrinken nach 10 Sekunden |
| Erzähler-Raum | 25% Heilung + Segen |
| Sarkophag | Gold (episches Item mit Schløssel) |
| Schere-Stein-Papier | Segen + episches Item bei Gewinn |
| Gløcksrad | Segen, Fluch, Gold oder Schløssel |
| Kanalraum | Episches Item |
| Lebensbrunnen | 25% Heilung |
| Spinne | 1/2/5 Schløssel |
| Wunschbrunnen | Gold → episches Item oder Segen |
| Zeughaus (Räume 90-98) | Episches Item; 10% chance legendär bei 2 Waffen |

### Monster-Schadenstabelle

| Bereich | Min | Avg | Max |
|---------|-----|-----|-----|
| Monster 1-24 | 10.98% | 14.24% | 17.49% |
| Boss 25 | 12.93% | 17.45% | 21.97% |
| Monster 26-49 | 14.50% | 19.00% | 23.49% |
| Boss 50 | 18.57% | 23.53% | 28.48% |
| Monster 51-74 | 14.50% | 19.00% | 23.49% |
| Boss 75 | 18.57% | 23.53% | 28.48% |
| Monster 76-99 | 17.30% | 23.65% | 30.00% |
| Boss 100 | 33.96% | 45.30% | 56.63% |

---

## Noch zu erforschen

- [ ] Prüfungspforte + Notausgang
- [ ] Schicksalstør, hungrige Tør, Segenstør
- [ ] Gem-Auswahl nach Boss
- [ ] 1000er state-Werte entschlüsseln
- [ ] SSP: param før Stein und Papier
- [ ] iadungeonsave Felder [4]-[16], [18], [20]-[21], [23]-[50]
- [ ] Bosse 2-4 (Räume 50, 75, 100)
- [ ] Legendäre Truhe nach Endboss

---

## Quellen

- HAR-Aufzeichnungen: Beedle the Bard, Doriku Gebeule, Eglenn, Gweneth, Haui Kaputti, Medea
- Playa Games Helpshift: https://playa-games.helpshift.com/hc/de/4-shakes-fidget-1653988985/faq/57-legendary-dungeon/
- ldgadget.12hp.de: https://ldgadget.12hp.de
