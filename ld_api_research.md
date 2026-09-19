# Legendary Dungeon – API Research

> Status: **Work in Progress** – basierend auf 70+ Charakter-Runs (Räume 1–100)
> plus einem gezielten zweiten Durchgang am 2026-09-19 mit 7 frischen Accounts
> (Gweneth/f9, Eglenn/f23, Beedle/f25, Haui/f28, Berengar/s5, Medea/s6,
> Alexandra/s7), aufgezeichnet gegen den ungetesteten marenga-PR
> `feat/legendary-dungeon` (the-marenga/mfbot#454), um dessen Annahmen zu
> verifizieren.  
> Fehlend: Einige Sonderfälle, state=317 unbeobachtet, state=306 nicht per Video bestätigt

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
| `91` | Schere-Stein-Papier: Papier wählen | SSP-Raum |
| `92` | Schere-Stein-Papier: Schere wählen | SSP-Raum |

> **Korrektur (2026-09-19):** Die frühere Annahme "Händler-Käufe und Klunker-Auswahl
> laufen über `param=70`" war falsch. Beides hat einen eigenen Endpoint (siehe unten),
> bestätigt an echten Requests aus 7 Accounts.  
> **Hungrige Tür:** `param=40` zum Bezahlen/Öffnen, danach steht Satte Kiste (monster=603) dahinter.  
> **Segenstür gelöst:** kein eigener Aktionscode nötig – die Segenstür ist einfach ein
> Türtyp in der Türauswahl (siehe Abschnitt "Türauswahl" unten), man betritt sie wie
> jede andere Tür.

### Eigene Endpoints (nicht über IADungeonInteract)

Bestätigt an echten Requests, 2026-09-19 (siehe Abschnitt "Türauswahl" für den Kontext):

| Endpoint | Format | Kontext | Bestätigt an |
|----------|--------|---------|---------------|
| `IADungeonMerchantBuy` | `{effektId}/{schlüssel}` | Segen beim Schlüsselmeister kaufen | alle 7 Accounts |
| `IADungeonDebuffMerchantBuy` | `{effektId}/{schlüssel}` | Fluch beim Fluchhändler kaufen | noch nicht beobachtet (kein Account war im Fluchhändler) |
| `IADungeonSelectSoulStone` | `{klunkerId}` | Klunker-Wahl nach Boss | Alexandra (s7), Medea (s6) |

`IADungeonMerchantBuy` taucht bei jedem Account genau beim Betreten des
Schlüsselmeister-Shops (state=315) mit `params=2/0` auf, offenbar ein
automatischer "Angebot ansehen"-Call des Spiel-Clients, keine echte
Kauf-Transaktion (0 Schlüssel = kein Kauf).

---

## Türauswahl (Door Select)

**Bisher komplett undokumentiert gewesen – größte Lücke der alten Doku, jetzt geschlossen.**

Vor jedem Raum (auch vor Raum 1) steht ein Zwei-Türen-Auswahlbildschirm. Erkennbar
an `iadungeonsave[15]=1` (state/stage). In diesem Zustand stehen in `[19]`/`[20]`
die beiden Türtypen (statt Raum-`state` wie sonst), Fallen dazu in `[25]`/`[26]`.

**Auswahl:** `IADungeonInteract` mit `param = Position + 1` (Position 0 oder 1),
**+4 wenn die Tür ein Schlüssel-Typ ist** (Locked/DoubleLocked/Epic). Also
`param=1` oder `2` für normale Türen, `param=5` oder `6` wenn die gewählte Tür
verschlossen ist und mit Schlüssel geöffnet wird.

**Zugemauerte Tür (Wand):** Ist eine der beiden Optionen eine Wand, bleibt nur
die andere wählbar – bestätigt an 12 Fällen über alle 7 Accounts. Man kann die
Wand nicht anklicken/auswählen, sie blockiert nur diese eine Seite.

**Fixer Raum auf Etage 4:** Bei allen 6 Accounts, die so weit kamen, war Etage 4
(die 5. Türauswahl) identisch: eine Wand + eine Schlüsselmeister-Tür. Das wirkt
wie ein erzwungener früher Schlüsselmeister-Besuch, kein Zufall.

### Beobachtete Türtypen (Feld `[19]`/`[20]` im DoorSelect-Zustand)

| Wert | Bedeutung |
|------|-----------|
| `1`/`2`/`3` | Monstertür (3 Varianten) |
| `4`/`5` | Bosstür (2 Varianten) |
| `1000` | Zugemauerte Tür / Wand – nicht wählbar |
| `1001` | Fragezeichentür (zufälliger Inhalt) |
| `1002` | Verschlossene Tür (1 Schlüssel) |
| `1003` | Offene Tür |
| `1004` | Epische Tür (Schlüssel nötig, Epische Truhe dahinter) |
| `1005` | Doppelt verschlossene Tür (2 Schlüssel) |
| `1006` | Goldene Tür |
| `1007` | Opfertür (Lebensenergie-Kosten, Opfertruhe dahinter) |
| `1008` | Verfluchte Tür (Fluch beim Öffnen) |
| `1009` | Schlüsselmeister-Tür |
| `1010` | Segenstür – **löst die alte offene Frage**, kein Extra-Aktionscode nötig |
| `1011` | Glücksradtür |
| `1012`–`1017` | Hungrige Türen (Holz/Stein/Seelen/Metall/Arkan/Sanduhren) |
| `1018`–`1022` | Prüfungspforte 1–5 |
| `1023` | Prüfungspforten-Ausgang |

Alle beobachteten Werte lagen exakt in diesem Wertebereich (keine Ausreißer
außerhalb 1-5 bzw. 1000-1023) – die Aufzählung scheint vollständig.

---

## Response: iadungeonsave Felder

Format: `iadungeon.iadungeonsave:{f0}/{f1}/{f2}/...`

| Index | Bedeutung | Beispielwert |
|-------|-----------|--------------|
| `[0]` | Charakter-ID (konstant) | `664444180` |
| `[1]` | Dungeon-Typ / Run-Nummer (0=1. Run normal, 1=2. Run normal, 2=1. Run Ultimate) | `0` |
| `[2]` | Aktuelle HP | `82426523` |
| `[3]` | **Korrektur:** nicht Maximale HP, sondern HP vor der letzten Aktion (für die Lebensbalken-Animation) | `82426523` |
| `[4]` | Maximale HP (das war vorher fälschlich als `[3]` dokumentiert) | `139762968` |
| `[15]` | Stage/Raumzustand: `1`=Türauswahl, `10`=Raum betreten, `11`=interagiert, `12`=Sonderaktion, `100`=Raum fertig | `10` |
| `[17]` | Aktuelle Raumnummer/Etage | `8` |
| `[18]` | Maximale Etage (immer `100` gesehen) | `100` |
| `[19]` | Bei Stage=1: Türtyp Tür 0. Sonst: Raumzustand (state) | `100` |
| `[20]` | Bei Stage=1: Türtyp Tür 1. Sonst: Zusatzwert (z.B. `900`/`901` nach Interaktion) | – |
| `[22]` | Objekt/Monster im Raum | `-5085` |
| `[23]` | **Neu:** Gold-Betrag, der bei state=316 vergeben wird – exakt gegen `resources`-Delta bestätigt (2 Accounts) | `1315002840` |
| `[25]`/`[26]` | Bei Stage=1: Fallen-Typ auf Tür 0/1 (0 = keine Falle) | `0` |

> **Noch unbekannt:** viele andere Felder (Schlüssel-Anzahl, aktive Segen/Flýche, Ressourcen, etc.)

---

## Response: iadungeonstats Felder

Format: `iadungeonstats:{a}/{b}/{c}/{d}/{e}`

| Index | Bedeutung |
|-------|-----------|
| `[0]` | Anzahl Segen erhalten |
| `[1]` | Anzahl Bosse besiegt |
| `[2]` | Anzahl Monster getötet |
| `[3]` | Gold gesammelt |
| `[4]` | ? (meist 1-7) |

---

## Response: state-Werte (Raumzustände)

### Aktive Raoumzustände

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
| `304` | Goldene Tür → Lavaraum (HP-VerFgust beim Betreten) |
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
**Kein Fluchhändler** – Flüche verkauft nur der Fluci�ändler (state=323, Scheibenkleistermeister).

### Post-Raum-Zustände (nach Abschluss eines Raums)

| state | Bedeutung |
|-------|-----------|
| `1000` | Segen erhalten (aus Kiste/Fass/Jungbrunnen/Kampf) |
| `1001` | Normaler Abschluss (nächster Raum wählbar) |
| `1002` | Auswahl/Bestätigung (Item eingepackt, Klunker gewählt, Gold, etc.) |
| `1003` | Boss besiegt + Klunker gewählt (Variante) |
| `1005` | Schlüssel nach Monsterkampf erhalten |
| `1006` | Notausgang-Truhe geöffnet / Raum verlassen (allgemein) |
| `1007` | Gold erhalten (Skelett zu Staub / Bronzene Schatztruhe / Holzkiste) |
| `1008` | Segen erhalten (nach Raum, z.B. Weg der Besserung aus Fass) |
| `1009` | Fluch erhalten (aus Tür/Händler/Fass) |
| `1010` | Item erhalten (aus Interaktionsraum / goldener Tür, z.B. Kanalisation, Zeughaus) |
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
| `91` | SSP-Gegner (Gleichstand = Papier) |
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
| `306` | Leerer goldener Raum | Kein Effekt (2026-09-19: weder HP- noch Gold-/Loot-Änderung beobachtet – stützt "kein Effekt", aber kein Video zur endgültigen Bestätigung) |
| `307` | Wunschbrunnen | Münze einwerfen → Item oder Segen; kein Auswahlfeld |
| `308` | Schere-Stein-Papier | Segen + Item bei Gewinn; Fluch + 10% Schaden bei Verlust |
| `309` | Goldene Tür (allgemein) | Kanalisation, Spinne, etc. |
| `312` | Sarkophag | Gold erhalten |
| `310` | Erleuchteter Durchgang | Monster mit Laterne dahinter |
| `314` | Holzstapel / Ressourcenraum | Holz, Stein, Metall, etc. |
| `315` | Schlüsselmeister-Shop | Segen, Lebenselixiere gegen Schlüssel/Pilze |
| `316` | Schatztruhe | Gold erhalten – **bestätigt 2026-09-19**: Betrag steht in `iadungeonsave[23]`, exakt gegen `resources`-Delta nachgerechnet (Beedle/f25, Berengar/s5). Der marenga-PR klassifiziert `316` fälschlich als "Wheel of Fortune" (HP-gatete Risiko-Aktion) – widerlegt, keiner der beiden Runs nahm dabei Schaden. |
| `321` | Seelenbad | Seelen für die Unterwelt |
| `322` | Arkane Splitter-Höhle / leer | Arkane Splitter oder leer |
| `323` | Fluchhändler | Schlüssel gegen Flüche |
| `329` | Zeughaus (Räume 90–98) | Episches Item (10% Chance legendär bei 2 Waffen) |

---

## Schicksalsklunker (Gems of Fate)

Nach den Bossen in Räumen 25, 50 und 75 **muss** man einen von drei Klunkern wählen. Nach dem Endboss (Raum 100) gibt es stattdessen eine legendäre Truhe – kein Klunker.  
Tier-Liste aus ldgadget.12hp.de + Spieler-Screenshots.

**Bestätigt am 2026-09-19** (Medea/s6 + Alexandra/s7, Angebote wörtlich mit dem
Response-Feld `iadungeonsoulstones` abgeglichen – Effekttexte stimmen exakt):
Spionageklunker, Glücksspielerbrocken, Auge des Stiers, Findling des Tölpels.

Response-Format `iadungeonsoulstones`: Blöcke aus je 6 Werten
(`typ/vorteilCode/vorteilStärke/nachteilCode/nachteilStärke/spezialCode`),
erste 3 Blöcke = aktive Klunker, restliche Blöcke = angebotene Klunker zur Wahl.
Auswahl per `IADungeonSelectSoulStone:{typ}` (siehe Endpoints oben).

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
| C | Diamant des Zeitreisenden | Diamond of the Time Traveler | Segen-Dauer verlängert | Fässer → immer Flýche |
| C | Hoffnung des Verdurstenden | Hope of the Thirsty One | Mehr verfluchte Türen | – |
| C | Verfluchte Perle | Cursed Pearl | – | – |
| C | Blutstropfen der Opfergabe | Blood Drop of Sacrifice | Weniger Schaden durch Opfertüren | – |
| D | Smaragd des Forschers | Emerald of the Explorer | – | Weniger geheimnisvolle Tøren |
| E | Saphir des Pechvogels | Sapphire of the Misadventurer | Weniger verfluchte Tøren | – |
| E | Kronjuwel des Teufels | Crown Jewel of the Devil | Chance auf epische Türen | Monster hinter Türen |
| F | Findling des Tölpels | **Erratic Boulder of the Hick** (bestätigt 2026-09-19, id `16`) | -Opfertüren | +30% Schaden bei Flucht-Fail |
| F | Kiesel der Hinterlist | Pebble of Deceit | Monster weniger Schaden | Monster hinter Türen |
| F | Magnetstein | Lodestone | Doppelt verschlossene Türen; +Schlüssel | – |
| F | Auge des Stiers | Eye of the Bull (bestätigt 2026-09-19, id `1`) | -20% Monsterschaden | -30% Fluchtchance |
| F | Irrender Brocken des Tölpels | ~~Erratic Boulder of the Hick~~ **Duplikat?** | -Opfertüren | – |
| F | Nierenstein der Zielstrebigkeit | Kidney Stone of Determination | Verfluchte Truhen hinter Tøren | – |
| F | Alter Opferstein | Old Sacrifice Stone | Weniger Schaden Opfertruhen | – |

> **Datenqualität der Tier-Liste:** "Findling des Tölpels" und "Irrender Brocken
> des Tölpels" hatten in der Quelle (ldgadget.12hp.de) beide den gleichen
> englischen Namen "Erratic Boulder of the Hick" eingetragen, obwohl es zwei
> Zeilen mit leicht unterschiedlichem Nachteil sind. Die jetzt bestätigte id `16`
> und ihre Effektwerte passen exakt zu "Findling des Tölpels" – die Zeile
> "Irrender Brocken des Tölpels" ist vermutlich ein Duplikat/Fehler in der
> Quelle und braucht eine eigene Bestätigung, falls sie tatsächlich existiert.

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

## Heilung nach 0 HP

Fällt der Charakter im Run auf 0 HP, kommt man erst nach Warten oder gegen Pilze
wieder rein. **Bestätigt am 2026-09-19 per UI-Screenshot** (Alexandra bei 3%,
Gweneth bei 5% natürlich geheilt, beide zeigten dieselbe Formel):

- **Natürliche Heilrate:** `4,17% Leben pro Stunde` – das ist exakt `100% / 24h`.
  Deckt sich 1:1 mit der `GetHealingHealthPercent()`-Formel im marenga-Port.
- **Server-seitiges Minimum:** UI zeigt "Min. 20% Leben zum Betreten benötigt!" –
  eine harte Server-Regel, kein Bot-Setting. Erklärt, warum
  `LegendaryDungeonMinHealingPercent` im marenga-Port als Default genau `20.0` hat.
- **Pilz-Preis-Formel:** `Pilze = aufrunden(0,48 × fehlende %)`. Bestätigt an 2
  Accounts:
  - +20% Fixbutton: immer 10 Pilze (= `ceil(0,48×20)`), bei beiden Accounts gleich
  - Komplettheilung: 97% fehlend → 47 Pilze, 95% fehlend → 46 Pilze – beide exakt
    `ceil(0,48×x)`
  - Der "+20%"-Button ist also kein Sonderpreis, sondern dieselbe Formel, nur als
    fixer Menüpunkt für " +20 Prozentpunkte" angezeigt
- **Noch offen:** ob sich die 0,48er-Konstante mit Level/VIP-Status ändert (bisher
  nur an 2 Accounts unterschiedlichen Levels bestätigt, beide passten); der
  tatsächliche Request/die Response beim Bezahlen selbst (noch niemand hat real
  bezahlt) – würde auch das Feld `iadungeon20cost` endgültig klären

---

## Noch zu erforschen

- [x] Legendäre Truhe nach Endboss: Run endet nach Item einpacken, kein Post-state, direkt Auswahlbildschirm
- [x] Hungrige Tür: `param=40`, akzeptiert Arkane Splitter / Sanduhren / Seelen / Steine
- [x] SSP: `param=90` = Stein, `param=91` = Papier, `param=92` = Schere; `monster=90/91/92` = Gleichstand Stein/Papier/Schere
- [x] Sarkophag: state=312 (Gold)
- [x] Segenstür: kein eigener param, ist nur Türtyp `1010` in der Türauswahl (siehe "Türauswahl")
- [x] Türauswahl-Mechanik komplett dokumentiert (siehe "Türauswahl"), inkl. aller Türtypen und Blocked-Door-Verhalten
- [x] Klunker-Auswahl-Endpoint: `IADungeonSelectSoulStone`, kein `param=70` (siehe "Eigene Endpoints")
- [x] Heilungsrate und Pilz-Preisformel nach 0 HP (siehe "Heilung nach 0 HP") – natürliche Rate und Preis-Formel bestätigt, echter Bezahl-Request noch offen
- [x] Händler-Kauf-Endpoint: `IADungeonMerchantBuy`/`IADungeonDebuffMerchantBuy`, kein `param=70` (siehe "Eigene Endpoints")
- [x] iadungeonsave `[15]`=Stage, `[17]`=Etage, `[18]`=Max-Etage, `[19]`/`[20]`=Türtypen (im DoorSelect) bzw. Raumzustand, `[23]`=Gold-Betrag (state=316)
- [ ] iadungeonsave restliche Felder `[5]`–`[14]`, `[16]`, `[21]`, `[24]`, `[27]`–`[50]` (Segen/Fluch-Slots, Merchant-Angebote etc. – siehe marenga-Port `LegendaryDungeon.cs` für Kandidaten-Layout, aber ungetestet)
- [ ] buff_id=1 genauer klären (Plønderer vs. Weg der Besserung)
- [ ] Boss-Varianten A/B vollständig kartieren
- [ ] state=317 (im marenga-Port "SpiderWeb", in dieser Doku "Schicksalstür/Glücksrad") – noch nicht beobachtet, nach dem state=316-Fund mit Vorsicht zu genießen
- [ ] state=306 per Video bestätigen (aktuell nur "kein messbarer Effekt" belegt, nicht was visuell passiert)
- [ ] Restliche Golden-Room-states (302, 311, 313, 319, 320, 324–328 im marenga-Port benannt) gegen echte Captures prüfen – ungetestet übernommen
- [ ] Fluchhändler (state=323) und `IADungeonDebuffMerchantBuy` noch nicht in echten Daten gesehen

---

## Quellen

- HAR-Aufzeichnungen: 80+ Charakter-Runs auf verschiedenen Servern (F9, F25, F28)
- Gezielter zweiter Durchgang 2026-09-19: 7 Accounts (F9, F23, F25, F28, S5, S6, S7),
  aufgezeichnet um die Annahmen im marenga-PR `feat/legendary-dungeon`
  (the-marenga/mfbot#454, ungetestet) zu verifizieren
- Playa Games Helpshift: https://playa-games.helpshift.com/hc/de/4-shakes-fidget-1653988985/faq/57-legendary-dungeon/
- ldgadget.12hp.de: https://ldgadget.12hp.de
