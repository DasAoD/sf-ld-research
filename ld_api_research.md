# Legendary Dungeon – API Research

> Status: **Work in Progress** – basierend auf 70+ Charakter-Runs (Räume 1–100)  
> Fehlend: Einige Sonderfälle, Segenstür param

---

## Endpoint

```
GET https://f{server}.sfgame.net/cmd.php?req=IADungeonInteract&params={base64}&sid={session}
```

- `{server}` = Servernummer, z.B. `f25`, `f9`, `f28`
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
| `40` | Tür mit Schlüssel / Hungrige Tür öffnen / Truhe bestätigen | Interaktion |
| `42` | Raum verlassen ohne Aktion (Truhe nicht öffnen) | Abbruch |
| `50` | Falle / Opfertür / erleuchteten Durchgang / Jungbrunnen / Goldenen Raum betreten | Navigation |
| `51` | Raum verlassen ohne Aktion (Händler ablehnen) | Abbruch |
| `60` | Kampfergebnis / Raum-Aktion bestätigen | Kampf / Interaktion |
| `70` | Poll / Zustand abfragen / nächster Schritt / Kauf bestätigen | Allgemein |
| `80` | Händler: in Waren blättern (kostet 1 Pilz) | Händler |
| `90` | Schere-Stein-Papier: Stein wählen | SSP-Raum |
| `91` | Schere-Stein-Papier: Papier? (unbestätigt) | SSP-Raum |
| `92` | Schere-Stein-Papier: Schere wählen | SSP-Raum |

> **Hinweis:** Händler-Käufe und Klunker-Auswahl nach Boss erfolgen über `param=70` – kein eigener param!  
> **Hungrige Tür:** `param=40` zum Bezahlen/Öffnen, danach steht Satte Kiste (monster=603) dahinter.  
> **Noch unbekannt:** param für Segenstør

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

> **Noch unbekannt:** viele andere Felder (Schlüssel-Anzahl, aktive Segen/Flüche, Ressourcen, etc.)

---

## Response: iadungeonstats Felder

Format: `iadungeonstats:{a}/{b}/{c}/{d}/{e}`

| Index | Bedeutung |
|-------|-----------|
| `[0]` | Anzahl Segen erhalten |
| `[1]` | Anzahl Bosse besiegt |
| `[2]` | Anzahl Monster getötet |
| `[3]` | Gold gesammelt |
| `[4]` | ? (meist 1–7) |

---

## Response: state-Werte (Raumzustände)

### Aktive Raumzustände

| state | Bedeutung |
|-------|-----------|
| `1` | Monster-Raum |
| `2` | Prüfungspforte-Monster-Raum (Variante 1) |
| `3` | Prüfungspforte-Monster-Raum (Variante 2) |
| `4` | Boss-Raum (Raum 25 / 50 / 75) |
| `5` | Endboss-Raum (Raum 100) |
| `100` | Interaktionsraum (Truhe / Fass / Skelett / Kiste) |
| `200` | Leerer Durchgang / Tür ohne Inhalt (kann Falle enthalten) |
| `301` | Jungbrunnen (HP-Heilung) |
| `303` | Goldene Tür → Steinhaufen (Steine für Festung) |
| `304` | Goldene Tür → Lavaraum (HP-Verlust beim Betreten) |
| `305` | Goldene Tür → Dungeon-Erzähler (Tee trinken = HP + Segen) |
| `306` | Goldene Tür → leerer Raum (kein Effekt) |
| `307` | Goldene Tür → Wunschbrunnen (Münze einwerfen → Item oder Segen; kein Auswahlfeld) |
| `308` | Schere-Stein-Papier-Raum |
| `309` | Goldene Tür (allgemein) |
| `310` | Erleuchteter/Goldener Durchgang |
| `312` | Goldene Tür → Sarkophag (Gold) |
| `314` | Goldene Tür → Ressourcenraum (Holzstapel, etc.) |
| `315` | Schlüsselmeister-Händler (Segen, Lebenselixiere, etc.) |
| `316` | Goldene Tür → Schatztruhe (Silberne o.ä.) |
| `317` | Schicksalstür / Glücksrad (zufällige Belohnung, kein Schlüssel nötig) |
| `321` | Goldene Tür → Seelenbad (Seelen für die Unterwelt) |
| `322` | Spezialraum ohne Kampf (Arkane Splitter-Höhle, leerer Raum, etc.) |
| `323` | Fluchhändler-Raum |
| `329` | Zeughaus / Waffenständer (episches Item, Räume 90–98) |

### Zugemauerte Tür
Blockiert den Weg. Kein Durchgang möglich, nur zum Anklicken.  
Post-state: `1015`

### Schicksalstür (Glücksrad)
Eine Tür hinter der das Glücksrad gedreht wird. Zufällige Belohnung (Segen, Gold, Fluch, etc.).  
Dahinter: leerer Raum, Monster-Raum oder Interaktionsraum.

### Schlüsselmeister-Händler (state=315)
Verkauft Segen, Lebenselixiere (25% oder 50% HP) und weitere Items gegen Schlüssel oder Pilze.  
**Kein Fluchhändler** – Flüche verkauft nur der Fluchhändler (state=323, Scheibenkleistermeister).

### Post-Raum-Zustände (nach Abschluss eines Raums)

| state | Bedeutung |
|-------|-----------|
| `1000` | Segen erhalten (aus Kiste/Fass/Jungbrunnen/Kampf) |
| `1001` | Normaler Abschluss (nächster Raum wählbar) |
| `1002` | Auswahl/Bestätigung (Item eingepackt, Klunker gewählt, Gold, etc.) |
| `1003` | Boss besiegt + Klunker gewählt (Variante) |
| `1006` | Notausgang-Truhe geöffnet / Raum verlassen (allgemein) |
| `1007` | Gold erhalten (Skelett zu Staub / Bronzene Schatztruhe / Holzkiste) |
| `1008` | Segen erhalten (nach Raum, z.B. Weg der Besserung aus Fass) |
| `1009` | Fluch erhalten (aus Tür/Händler/Fass) |
| `1010` | Schlüssel erhalten nach Kampf |
| `1011` | Fluch aus Fass erhalten / Gold aus Kiste (Kontext-abhängig) |
| `1012` | Schaden durch aktiven Fluch (HP sinkt pro Raum) |
| `1013` | Gold erhalten (aus Schatztruhe) |
| `1014` | Gold + Segen erhalten |
| `1015` | Zugemauerte Tür angeklickt (kein Durchgang) |
| `1016` | Fluch nach Kampf erhalten (Kaputte Rüstung o.ä.) |
| `1017` | Skelett besiegt + Schlüssel erhalten |
| `1018` | Boss besiegt + Klunker gewählt (Variante) |
| `1019` | 1. Prüfungspforte bestanden |
| `1020` | 2. Prüfungspforte bestanden |
| `1021` | 3. Prüfungspforte bestanden (mit Gold) |
| `1022` | 4. Prüfungspforte bestanden |
| `1023` | 5. Prüfungspforte bestanden / Notausgang mit Preis-Truhe geöffnet |

> **Hinweis:** `1002` ist vielseitig – erscheint nach Item einpacken, Klunker wählen nach Boss, Gold aus Kiste, etc.

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
| `603` | Satte Kiste (hinter Hungriger Tür) |

### Kisten & Fässer

| monster | Bedeutung |
|---------|-----------|
| `100` | Holzkiste (Gold / Ressourcen) |
| `101` | Fass (mit Item) |
| `102` | Holzkiste (mit Item) |
| `400` | Fass (mit Segen oder Fluch) |

### Skelette

| monster | Bedeutung |
|---------|-----------|
| `300` | Magierskelett |
| `301` | Kriegerskelett |

### Sonstiges

| monster | Bedeutung |
|---------|-----------|
| `90` | SSP-Gegner (Gleichstand = Stein) |
| `92` | SSP-Gegner (Gleichstand = Schere) |

### Monster-Räume (state=1/2/3/4/5)

Negative Werte = reguläre Monster-IDs.

| monster | Bedeutung |
|---------|-----------|
| `-5002` | Monster mit Laterne (nach erleuchtetem Durchgang) |
| `-5075` bis `-5099` | Reguläre Dungeon-Monster Ebene 1 (Räume 1–24) |
| `-5100` bis `-5141` | Reguläre Dungeon-Monster Ebene 2–4 (Räume 26–99) |
| `-5142` | Boss 1 Variante A (Raum 25) |
| `-5141` | Boss 1 Variante B (Raum 25, 2. Durchlauf) |
| `-5143` | Boss 2 Variante B (Raum 50, 2. Durchlauf) |
| `-5144` | Boss 2 Variante A (Raum 50) |
| `-5145` | Boss 3 Variante B (Raum 75, 2. Durchlauf) |
| `-5146` | Boss 3 Variante A (Raum 75) |
| `-5147` | Endboss Variante B (Raum 100, 2. Durchlauf) |
| `-5148` | Endboss Variante A (Raum 100) |
| `-5240` bis `-5245` | Spezial-Monster (Geburtstags-LD o.ä.) |

---

## Response: iamap

Erscheint beim Tod oder nach bestimmten Räumen (z.B. Zeughaus).  
Format: `iamap:{boss_raum}/{boss_monster_id}/{?}/{gem_typ}/...`

Beispiel: `iamap:25/-5142/1/315/25/-5144/1/-315/25/-5146/1/-315/25/-5148/1/-315`

- Zeigt alle 4 Bosse mit Raumnummer, Monster-ID und gewähltem/nicht gewähltem Klunker
- `315` = Klunker gewählt / `-315` = kein Klunker gewählt
- Der 4. Eintrag (Endboss) hat keinen Klunker – dort gibt es eine legendäre Truhe

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
| `1` | Segen | Plünderer (+100% Gold in 10 Kammern) oder Weg der Besserung |
| `2` | Segen | One Hit Wonder (Monster sofort töten) |
| `4` | Segen? | ? |
| `5` | Segen | Dietrich (nächste 4 Türen ohne Schlüssel öffnen) |
| `6` | Segen | Schlüsselerlebnis (70% Chance auf 2 Schlüssel in 8 Kämpfen) |
| `8` | Segen | Weg der Besserung (HP-Heilung nach Raum) |
| `101` | Fluch | Kaputte Rüstung (Gegner verursacht +50% Schaden für 4/8 Räume) |
| `102` | Fluch | 5% Schaden pro Raum für 5 Räume |
| `104` | Fluch | 50% Gold aus Truhen für 5/10 Kammern |
| `105` | Fluch | Starke Verschlüsselung (doppelte Schlüsselkosten für 4/8 Türen) |

---

## Prüfungspforte & Notausgang

### Prüfungspforte
- Erscheint maximal einmal pro Dungeon-Run, nie in Ebene 1
- Kein Schlüssel erforderlich
- Dahinter immer ein Monster-Raum (state=2 oder 3)
- Nach Sieg: Wahl zwischen weiterer Prüfungspforte oder Notausgang
- Maximal 5 Prüfungspforten hintereinander
- Post-states: `1019` → `1020` → `1021` → `1022` → `1023`

### Notausgang
- Erscheint nach jeder Prüfungspforte und nach allen 5 bestanden
- Kein Schlüssel erforderlich
- Dahinter immer eine Preis-Truhe (monster=`602`)
- Bessere Belohnungen je mehr Prüfungspforten bestanden
- Post-state nach Öffnen: `1006` oder `1023`

---

## Hungrige Tür

- Verlangt eine Ressource als Eintrittspreis (Arkane Splitter, Sanduhren, Seelen, Steine, etc.)
- Bezahlen: `param=40`
- Dahinter immer eine Satte Kiste (monster=`603`)
- Satte Kiste enthält Ressourcen (Seelen, Holz, Arkane Splitter, etc.)
- Kein Schlüssel nötig

---

## Goldene Räume

Goldene Räume erscheinen hinter goldenen Türen (state=309 allgemein).

| state | Raumtyp | Inhalt |
|-------|---------|--------|
| `301` | Lebensbrunnen | 25% HP-Heilung |
| `303` | Steinhaufen | Steine für Festung |
| `304` | Lavaraum | HP-Verlust beim Betreten |
| `305` | Dungeon-Erzähler | Tee trinken: HP + Segen; ablehnen: kein Effekt |
| `306` | Leerer goldener Raum | Kein Effekt |
| `307` | Wunschbrunnen | Münze einwerfen → Item oder Segen; kein Auswahlfeld |
| `308` | Schere-Stein-Papier | Segen + Item bei Gewinn; Fluch + 10% Schaden bei Verlust |
| `309` | Goldene Tür (allgemein) | Kanalisation, Spinne, etc. |
| `312` | Sarkophag | Gold erhalten |
| `310` | Erleuchteter Durchgang | Monster mit Laterne dahinter |
| `314` | Holzstapel / Ressourcenraum | Holz, Stein, Metall, etc. |
| `315` | Schlüsselmeister-Shop | Segen, Lebenselixiere gegen Schlüssel/Pilze |
| `316` | Schatztruhe | Silberne Schatztruhe o.ä. |
| `321` | Seelenbad | Seelen für die Unterwelt |
| `322` | Arkane Splitter-Höhle / leer | Arkane Splitter oder leer |
| `323` | Fluchhändler | Schlüssel gegen Flüche |
| `329` | Zeughaus (Räume 90–98) | Episches Item (10% Chance legendär bei 2 Waffen) |

---

## Schicksalsklunker (Gems of Fate)

Nach den Bossen in Räumen 25, 50 und 75 **muss** man einen von drei Klunkern wählen. Nach dem Endboss (Raum 100) gibt es stattdessen eine legendäre Truhe – kein Klunker.  
Tier-Liste aus ldgadget.12hp.de + Spieler-Screenshots.

| Tier | Name (DE) | Name (EN) | Effekt + | Effekt – |
|------|-----------|-----------|----------|----------|
| S | Seele des Hasen | Soul of the Rabbit | +40% Fluchtchance | Monster +25% Schaden |
| A | Verfluchter Mondstein | Cursed Moonstone | +20% Fluchtchance | Fluch-Dauer +1 |
| A | Spionageklunker | Spying Gem | Chance auf unverschlossene Türen | -15% Schlüssel aus Kämpfen |
| A | Anhänger des Schlüsselmeisters | Pendant of the Key Master | 40% Chance nach Fliehen: Schlüssel | Eine Tür immer verschlossen |
| B | Schmieriger Heilstein | Greasy Healing Stone | Nach Tod: Weg der Besserung 3R | Keine epischen Truhen |
| B | Glücksspielerbrocken | Boulder of the Gambler | Fallen → Flüche; +50% Segen aus Fässern | – |
| B | Brocken der Gier | Boulder of Greed | Mehr geheimnisvolle Türen; +Segen aus Truhen | – |
| C | Schatz des Helden | Treasure of the Hero | Monster weniger Schaden | -Chance auf Items |
| C | Diamant des Zeitreisenden | Diamond of the Time Traveler | Segen-Dauer verlängert | Fässer → immer Flüche |
| C | Hoffnung des Verdurstenden | Hope of the Thirsty One | Mehr verfluchte Türen | – |
| C | Verfluchte Perle | Cursed Pearl | – | – |
| C | Blutstropfen der Opfergabe | Blood Drop of Sacrifice | Weniger Schaden durch Opfertüren | – |
| D | Smaragd des Forschers | Emerald of the Explorer | – | Weniger geheimnisvolle Türen |
| E | Saphir des Pechvogels | Sapphire of the Misadventurer | Weniger verfluchte Türen | – |
| E | Kronjuwel des Teufels | Crown Jewel of the Devil | Chance auf epische Türen | Monster hinter Türen |
| F | Findling des Tölpels | ? | Weniger Opfertøren | +30% Schaden bei Flucht-Fail |
| F | Kiesel der Hinterlist | Pebble of Deceit | Monster weniger Schaden | Monster hinter Türen |
| F | Magnetstein | Lodestone | Doppelt verschlossene Türen; +Schlüssel | – |
| F | Auge des Stiers | Eye of the Bull | – | – |
| F | Irrender Brocken des Tölpels | Erratic Boulder of the Hick | Weniger Opfertøren | – |
| F | Nierenstein der Zielstrebigkeit | Kidney Stone of Determination | Verfluchte Truhen hinter Tøren | – |
| F | Alter Opferstein | Old Sacrifice Stone | Weniger Schaden Opfertruhen | – |

---

## Monster-Schadenstabelle (% der max. LD-HP)

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

- [x] Legendäre Truhe nach Endboss: Run endet nach Item einpacken, kein Post-state, direkt Auswahlbildschirm
- [x] Hungrige Tür: `param=40`, akzeptiert Arkane Splitter / Sanduhren / Seelen / Steine
- [x] SSP: `param=90` = Stein, `param=92` = Schere (Papier noch unbestätigt)
- [x] Sarkophag: state=312 (Gold)
- [ ] Segenstür (param=?)
- [ ] iadungeonsave Felder [4]–[16], [18], [20]–[21], [23]–[50]
- [ ] Dungeon-Typ Feld [1]: 2=normal, 3=LD Ultimate?
- [ ] buff_id=1 genauer klären (Plønderer vs. Weg der Besserung)
- [ ] Boss-Varianten A/B vollständig kartieren
- [ ] Spinne, Kanalraum, Wasserraum state-Werte

---

## Quellen

- HAR-Aufzeichnungen: 80+ Charakter-Runs auf verschiedenen Servern (F9, F25, F28)
- Playa Games Helpshift: https://playa-games.helpshift.com/hc/de/4-shakes-fidget-1653988985/faq/57-legendary-dungeon/
- ldgadget.12hp.de: https://ldgadget.12hp.de
