# Legendary Dungeon – API Research

> Status: **Work in Progress** – basierend auf 70+ Charakter-Runs (Räume 1–100)
> plus einem gezielten zweiten Durchgang am 2026-09-19 mit 7 frischen Accounts
> (F9, F23, F25, F28, S5, S6, S7), aufgezeichnet gegen den ungetesteten marenga-PR
> `feat/legendary-dungeon` (the-marenga/mfbot#454), um dessen Annahmen zu
> verifizieren.  
> Fehlend: Einige Sonderfälle, state=317 unbeobachtet

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
| `IADungeonStart` | `{themeId}/{modus}` | Dungeon (neu) betreten | alle 7 Accounts, bisher immer `modus=0` (Normal-Modus) |
| `IADungeonMerchantBuy` | `{effektId}/{schlüssel}` | Segen beim Schlüsselmeister kaufen | alle 7 Accounts |
| `IADungeonDebuffMerchantBuy` | `{effektId}/{schlüssel}` | Fluch beim Fluchhändler kaufen | **bestätigt 2026-09-19**, 2 Accounts kauften Effekt `104` (GoldRushHangover) für 1-2 Schlüssel |
| `IADungeonSelectSoulStone` | `{klunkerId}` | Klunker-Wahl nach Boss | S7, S6 |

**`modus` bei `IADungeonStart` vermutlich `0`=Normal, `1`=Ultimate** – unbestätigt,
da noch niemand den Ultimate-Modus betreten hat (kostet 650 Pilze pro Event laut
UI, gibt dafür 3-fache Ressourcen/Gold + 3 legendäre Items statt 1 am Ende).

### Event-Ablauf (aus der Einstiegs-UI, 2026-09-19)

- Event läuft **10 Tage**, Countdown wird auf dem Splash-Screen angezeigt
- Nach Endboss (Raum 100) **setzt sich der Dungeon zurück** – mehrere Runs pro
  Event möglich (erklärt vermutlich den Run-Zähler in `iadungeonsave[1]`)
- 100 Räume verteilt auf **4 Ebenen** à 25 Räume, ein Boss am Ende jeder Ebene

### Event-Themes (Zuordnung deutscher Name ↔ Theme-ID)

| Theme-ID | Deutscher Name | Bestätigt an |
|----------|-----------------|--------------|
| `6` | Abgründe des Wahnsinns (`AbyssOfMadness`) | F23, 2026-09-19 |
| `1`–`5`, `7`–`8` | noch unbeobachtet | – |

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

**Bosstür ist immer erzwungen:** Laut Playa-Wiki ist bei einer Bosstür **die
andere Tür immer zugemauert** – eine allgemeine Regel, nicht nur ein
Zufallsbefund. Deckt sich mit unseren Beobachtungen bei den Bossräumen
(Etage 24, 49 etc. zeigten immer Wand+Bosstür-Paare). Das ist unabhängig vom
Etage-4-Sonderfall oben (verschiedene Mechanik, gleiches Symptom "Wand +
erzwungene andere Tür").

**Offiziell dokumentiert:** [Doors-Wiki](https://playa-games.helpshift.com/hc/de/4-shakes-fidget-1653988985/faq/282-legendary-dungeon---doors/?p=web),
abgeglichen 2026-09-19 – deckt sich mit allen unten aufgeführten Türtypen.

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
| `[1]` | **Umstritten:** bisher als Dungeon-Typ/Run-Nummer dokumentiert (0=1. Run normal, 1=2. Run normal, 2=1. Run Ultimate). Der marenga-Port liest dasselbe Feld als `HealthStatus` (2=lebt). Neue Daten (2026-09-19) zeigen aber `[1]=2` bei **allen** frischen Erst-Runs im Normal-Modus (müsste nach der alten Lesart `0` sein) – und einmal sogar `[1]=2` bei `CurrentHp=0`. Beide Deutungen passen nicht zu den Daten, Feld bleibt ungeklärt. | `2` (fast immer) |
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
| `306` | Goldene Tür → Überfluteter Raum (10-Sekunden-Ertrink-Timer, siehe "Goldene Räume") |
| `307` | Goldene Tür → Wunschbrunnen (Münze einwerfen → Item oder Segen; kein Auswahlfeld) |
| `308` | Schere-Stein-Papier-Raum |
| `309` | Goldene Tür (allgemein) |
| `310` | Erleuchteter/Goldener Durchgang |
| `312` | Goldene Tür → Sarkophag (Gold) |
| `314` | Goldene Tür → Ressourcenraum (Holzstapel, etc.) |
| `315` | Schlüsselmeister-Händler (Segen, Lebenselixiere, etc.) |
| `316` | Goldene Tür → **Glücksrad** (bestätigt per Video, siehe "Goldene Räume" – nicht Schatztruhe, wie hier ursprünglich vermutet) |
| `317` | Unbestätigt – ursprünglich hier als "Schicksalstür/Glücksrad" vermutet, das war falsch (Glücksrad ist `316`, Schicksalstür ist ein Türtyp, kein Raum-state). Der marenga-Port nennt `317` "SpiderWeb" (Spinnennetz) – plausibel laut Wiki, aber noch nie beobachtet. |
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
| `0` | Bronzene Schatztruhe (Enum: `BronzeChest`) |
| `1` | Silberne Schatztruhe (Enum: `SilverChest`) |
| `2` | Epische Schatztruhe (Enum: `EpicChest`, bestätigt 2026-09-20) |
| `500` | Gefräßige Schmatztruhe (Enum: `MimicChest`, bestätigt 2026-09-20 – wird über `Interacted with` → `Fighting the encounter` abgewickelt, bestätigt "verwandelt sich in Monster") |
| `600` | Opfertruhe |
| `601` | Verfluchte Truhe |
| `602` | Notausgang-Preis-Truhe |
| `603` | Satte Kiste (Enum: `SatedChest`, bestätigt 2026-09-20, hinter Hungriger Tür) |

### Kisten & Fässer

| monster | Bedeutung |
|---------|-----------|
| `100` | Holzkiste (Gold / Ressourcen) |
| `101` | Fass (mit Item, Enum vermutlich `Barrel`, bestätigt 2026-09-20 für `Interacted with the encounter (Barrel)`) |
| `102` | Holzkiste (mit Item) |
| `400` | Fass (mit Segen oder Fluch) |

> **Neu, unbestätigt (2026-09-20):** Belohnungen hinter `LockedDoor` erscheinen
> in Logs als `Collected the reward (Crate1/2/3)` – drei Tiers gesehen, kein
> `Crate0`. Noch keine 1:1-Zuordnung zu den `monster`-IDs oben, evtl. ein
> separates, Tür-gebundenes Belohnungssystem statt der `state=100`-Interaktionsräume.

### Skelette

| monster | Bedeutung |
|---------|-----------|
| `300` | Magierskelett (Enum: `MageSkeleton`, bestätigt 2026-09-20) |
| `301` | Kriegerskelett (Enum: `WarriorSkeleton`, bestätigt 2026-09-20) |

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

> **Wichtige Korrektur (2026-09-19):** Die Boss-Monster-IDs sind offenbar
> **Theme-abhängig**, nicht global fix! Im aktuellen Event "Abgründe des
> Wahnsinns" (`AbyssOfMadness`, Theme-ID 6) zeigen **alle** Accounts, die Boss 1
> erreicht haben (6+ unabhängige Chars), exakt `monster=-5297` auf Raum 25 –
> nicht die oben dokumentierten `-5141`/`-5142`. Ein Account erreichte Raum 50
> mit `monster=-5298`. Die Tabelle oben stammt vermutlich aus einem anderen
> Event/Theme mit eigenem Monster-Pool. Bisher **keine zweite Variante**
> innerhalb dieses Events gesehen – möglich, dass Variante A/B nicht zufällig
> ist, sondern am Run-Zähler hängt (alle bisherigen Accounts waren im 1. Run,
> `iadungeonsave[1]`=0). Braucht Bestätigung durch einen 2. Run.

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

## Segen & Flüche – Mechanik-Regeln

**Offiziell aus dem In-Game-Infotext (2026-09-19), bisher nirgends dokumentiert:**

- Schlüssel kommen **ausschließlich von besiegten Monstern** ("Jedes im
  Legendary Dungeon besiegte Monster hat die Chance, einen Schlüssel in seinen
  Taschen zu haben") – nicht aus Truhen
- Ab **Ebene 2** (Raum 26+) können auch **normale Monster-Angriffe** einen
  Fluch verursachen, nicht nur Türen/Fässer/Truhen
- **Gleicher Segen/Fluch erneut erhalten → Dauer wird komplett überschrieben**
  (kein Stapeln, reiner Reset des Zählers)
- **Alle 3 Slots belegt + neuer Segen/Fluch → überschreibt Slot 0** (den
  ältesten/ersten), nicht den kürzesten oder zufällig
- **Gegen Bosse zeigen Segen, Flüche UND Schicksalsklunker keine Wirkung** –
  wichtig für jede Score-Funktion, die vor einem Bosskampf mit Effekten rechnet
- Der Fluchhändler heißt offiziell **Scheibenkleistermeister** (nicht nur
  "Fluchhändler")

## Segen & Flüche – bekannte usedbuffs-IDs

| buff_id | Typ | Bedeutung |
|---------|-----|-----------|
| `1` | Segen | Plünderer (+100% Gold in 10 Kammern) oder Weg der Besserung |
| `2` | Segen | One Hit Wonder (Monster sofort töten) |
| `4` | Segen? | ? |
| `5` | Segen | Dietrich (Enum: `LockPick`, bestätigt 2026-09-20 – gekauft beim Schlüsselmeister) (nächste 4 Türen ohne Schlüssel öffnen) |
| `6` | Segen | Schlüsselerlebnis (70% Chance auf 2 Schlüssel in 8 Kämpfen) |
| `8` | Segen | Weg der Besserung (HP-Heilung nach Raum) |
| `101` | Fluch | Kaputte Rüstung (Gegner verursacht +50% Schaden für 4/8 Räume) |
| `102` | Fluch | 5% Schaden pro Raum für 5 Räume |
| `104` | Fluch | 50% Gold aus Truhen für 5/10 Kammern |
| `105` | Fluch | Starke Verschlüsselung (doppelte Schlüsselkosten für 4/8 Türen) |
| `?` | Segen | Enum `KeyMoment`, bestätigt 2026-09-20 beim Schlüsselmeister gekauft – buff_id und genauer Effekt noch unbekannt (Name deutet auf schlüsselbezogenen Bonus) |
| `?` | Segen | Enum `EscapeAssistant`, bestätigt 2026-09-20 beim Schlüsselmeister gekauft – buff_id und genauer Effekt noch unbekannt (Name deutet auf Fluchtchance-Bonus) |
| – | Verbrauchsgegenstand | Enum `ElixirOfLife`, bestätigt 2026-09-20 – das im Schlüsselmeister-Shop erwähnte Lebenselixier (25%/50% HP), kein Segen/buff_id, sofort verbraucht |

---

## Prüfungspforte & Notausgang

### Prüfungspforte
- Enum im marenga-Port: `TrialRoom1`–`TrialRoom5` (bestätigt 2026-09-20, `TrialRoom1` als Türtyp in Live-Logs gesehen)
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
- **Bestätigt per Live-Logs (2026-09-20):** Türauswahl zeigt die Ressource direkt
  als eigenen Türtyp (nicht "MysteryDoor"), Wahl führt sofort zu `Collected the
  reward (SatedChest)` – kein Kampf/Encounter dazwischen. Enum-Namen im
  marenga-Port ↔ Ressource:

| Enum-Name | Ressource (DE) |
|-----------|-----------------|
| `Wood` | Holz |
| `Stone` | Stein |
| `Metal` | Metall |
| `Arcane` | Arkane Splitter |
| `Souls` | Seelen |
| `QuicksandGlasses` | Sanduhren |

---

## Goldene Räume

Goldene Räume erscheinen hinter goldenen Türen (leicht erkennbar am goldenen
Leuchten). **Offiziell dokumentiert** im Playa-Wiki:
[Golden Rooms](https://playa-games.helpshift.com/hc/de/4-shakes-fidget-1653988985/faq/284-legendary-dungeons---golden-rooms/?p=web)
(2026-09-19 abgeglichen, siehe Korrekturen unten).

| state | Raumtyp | Inhalt |
|-------|---------|--------|
| `301` | Lebensbrunnen (Enum: `FountainOfLife`, bestätigt 2026-09-20) | Heilt einen Teil der Lebensenergie (Basisversion). Spezialvariante entfernt zusätzlich Flüche. |
| `303` | Steinhaufen (Enum: `PileOfRocks`, bestätigt 2026-09-20) | Versperrt den Weg; Wegräumen bringt Steine fürs Festungslager |
| `304` | Der Boden ist Lava (Enum: `TheFloorIsLava`, bestätigt 2026-09-20) | Pflicht-Durchquerung, kostet Lebensenergie |
| `305` | Dungeon-Erzähler | Tee trinken: HP + Segen; ablehnen: kein Effekt |
| `306` | **Überfluteter Raum** (Enum: `FloodedRoom`, bestätigt) (nicht "kein Effekt") – Raum füllt sich mit Wasser, wer nicht **innerhalb von 10 Sekunden** verlässt, ertrinkt. Erklärt, warum unser HAR-Test keinen Effekt zeigte: der Bot verlässt sofort per `param=50`, lange bevor die 10s um sind. |
| `307` | Wunschbrunnen | Münze einwerfen → Item oder Segen; kein Auswahlfeld |
| `308` | Schere-Stein-Papier | Sieg: Segen + Item; Niederlage: Fluch + Schaden; Unentschieden: nichts |
| `309` | Kanalisation | Brühe durchsuchen → Item; Verlassen ohne Strafe |
| `310` | Laternenmonster (vermutlich Enum: `UndeadFiend`, unbestätigt – Log zeigt `Activated the room bonus (UndeadFiend)` direkt nach einer goldenen Tür, passt thematisch, aber keine 1:1-Bestätigung über eine Response mit explizitem `state=310`) | Kampf oder Flucht gegen das Monster mit Laterne |
| `312` | Sarkophag (Enum: `UnlockedSarcophagus`, bestätigt 2026-09-20) | Unverschlossen: Gold. Verschlossen (Schlüssel nötig): garantiert episches Item. |
| `314` | Holzstapel | Wegräumen für Festungslager-Ressourcen (Holz/Stein/Metall) |
| `315` | Schlüsselmeister-Shop | Segen gegen Schlüssel (siehe "Eigene Endpoints") |
| `316` | **Glücksrad – bestätigt per Videoaufnahme (2026-09-19), unsere frühere "vermutlich nicht Glücksrad"-These war falsch.** Das Rad hat 8 Felder (Gold, 2x Fluch-Symbol, 2x Segen-Symbol, Schlüssel+1, Schlüssel-1). Beide unabhängig aufgezeichneten Drehungen (F25, S5) landeten zufällig auf "Gold" – der angezeigte Münz-Betrag (`13.150.028` bzw. `13.609.424`, ×100 skaliert) deckt sich exakt mit dem in `iadungeonsave[23]` gefundenen und gegen `resources` verifizierten Betrag. Der marenga-PR hat mit `WheelOfFortune` also recht. **Bleibt aber ein offener Punkt:** Der marenga-Tasker behandelt `WheelOfFortune` als reinen HP-Risiko-Raum (`HandleDamageRoom`, HP-%-gated) – das eigentliche Risiko hier ist aber Fluch/Schlüsselverlust, nicht direkter HP-Schaden. Ob die HP-basierte Gating-Logik für diesen Raumtyp überhaupt die richtige Dimension ist, ist fraglich, siehe TODO in `LegendaryDungeon.cs`. |
| `317` | Vermutlich **Spinnennetz** (marenga-PR: `SpiderWeb`) – 3 Varianten mit steigendem Risiko: Beine sichtbar (viele Schlüssel, wenig Gift-Risiko), Kopf sichtbar (Gleichstand 2 Schlüssel oder Gift), ganze Spinne (hohes Gift-Risiko, seltene 5-Schlüssel-Belohnung). Noch nicht in echten Daten gesehen. |
| `321` | Seelenbad (Enum: `SoulBath`, bestätigt 2026-09-20) | Klicken schreibt Seelen in der Unterwelt gut |
| `322` | Arkane Splitter-Höhle | Splitter sammeln für den Schmied |
| `323` | Scheibenkleistermeister (Fluchhändler) | Kauft Schlüssel gegen Flüche – **bestätigt 2026-09-19**, Effekt `104` gekauft |
| `329` | Zeughaus (Räume 90–98) | Episches Item (10% Chance legendär bei 2 Waffen) |

### Edition-exklusive Goldene Räume

Der marenga-Port benennt weitere state-Werte, die wir bisher nie gesehen haben
(`302`, `311`, `313`, `319`, `320`, `324`–`328`). Grund gefunden: **das sind
Sonder-Editionen**, die nur während bestimmter Events im Raumpool sind, laut
Wiki nicht im aktuell laufenden Event ("Abgründe des Wahnsinns"):

| Edition | Räume |
|---------|-------|
| Geburtstags-Edition | Umkleide (episches Item), Flimmerkiste (10 Glücksmünzen), Beta-Raum (Kampf/Flucht), 3D-Shakes (Segen bei Sieg / Schaden bei Niederlage) |
| Herr-der-Ringe-Edition | Valaraukar (Kampf kostet HP, gibt "Weg der Besserung"-Segen; Flucht optional) |
| Halloween-Edition | Auktionshaus (Item, ggf. episch), Regenbogen-Raum (Segen kostet HP), Schweine-Raum (Kampf mit Netto-HP-Gewinn) |

Erklärt, warum diese über 10 Accounts (davon 3 mit mehreren Chars) nie
aufgetaucht sind – sie gehören schlicht nicht zum aktuellen Event. Zum Testen
bräuchten wir Captures während einer Geburtstags-/Halloween-/LOTR-Ausgabe des
LD.

---

## Schicksalsklunker (Gems of Fate)

Nach den Bossen in Räumen 25, 50 und 75 **muss** man einen von drei Klunkern wählen. Nach dem Endboss (Raum 100) gibt es stattdessen eine legendäre Truhe – kein Klunker.  
Tier-Liste aus ldgadget.12hp.de + Spieler-Screenshots.

**Offiziell bestätigt (In-Game-Infotext, 2026-09-19):** Gegen Bosse zeigen
Schicksalsklunker keine Wirkung (wie Segen/Flüche, siehe oben).

**Bestätigt am 2026-09-19** (Accounts auf S6 und S7, Angebote wörtlich mit dem
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
| D | Smaragd des Forschers | Emerald of the Explorer | – | Weniger geheimnisvolle Türen |
| E | Saphir des Pechvogels | Sapphire of the Misadventurer | Weniger verfluchte Türen | – |
| E | Kronjuwel des Teufels | Crown Jewel of the Devil | Chance auf epische Türen | Monster hinter Türen |
| F | Findling des Tölpels | **Erratic Boulder of the Hick** (bestätigt 2026-09-19, id `16`) | -Opfertüren | +30% Schaden bei Flucht-Fail |
| F | Kiesel der Hinterlist | Pebble of Deceit | Monster weniger Schaden | Monster hinter Türen |
| F | Magnetstein | Lodestone | Doppelt verschlossene Türen; +Schlüssel | – |
| F | Auge des Stiers | Eye of the Bull (bestätigt 2026-09-19, id `1`) | -20% Monsterschaden | -30% Fluchtchance |
| F | Irrender Brocken des Tölpels | ~~Erratic Boulder of the Hick~~ **Duplikat?** | -Opfertüren | – |
| F | Nierenstein der Zielstrebigkeit | Kidney Stone of Determination | Verfluchte Truhen hinter Türen | – |
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
wieder rein. **Bestätigt am 2026-09-19 per UI-Screenshot** (Account auf S7 bei
3%, Account auf F9 bei 5% natürlich geheilt, beide zeigten dieselbe Formel):

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
- **Abgleich mit Playa-Wiki:** Der allgemeine LD-Wiki-Artikel beschreibt die
  Pilzkosten als "10, 15, dann 20 Pilze für aufeinanderfolgende Heilungen
  innerhalb eines Runs" – das widerspricht unserer Formel nicht zwingend: beide
  UI-Screenshots waren jeweils die **erste** Nutzung in diesem Run (daher beide
  `10`), das Wiki beschreibt vermutlich eine **zusätzliche Eskalation pro
  Nutzung** des Fix-Buttons innerhalb desselben Runs, während unsere Formel für
  die "Komplettheilung"-Option gilt. Ungeklärt, bis jemand den Fix-Button
  mehrfach in einem Run nutzt.

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
- [ ] buff_id=1 genauer klären (Plünderer vs. Weg der Besserung)
- [ ] Boss-Varianten A/B vollständig kartieren
- [x] state=306 gelöst: **Überfluteter Raum**, laut Wiki 10-Sekunden-Ertrink-Timer – kein Widerspruch mehr zu "kein messbarer Effekt" im HAR (Bot verlässt sofort)
- [x] state=316 gelöst: **Glücksrad**, per Videoaufnahme bestätigt (2026-09-19) – marenga-PR hatte recht, unsere Zwischenthese "vermutlich nicht Glücksrad" war falsch. Offen bleibt, ob die HP-basierte Risiko-Bewertung im Tasker für diesen Raumtyp die richtige Dimension ist (Risiko ist Fluch/Schlüssel, nicht HP)
- [ ] state=317 (marenga-Port: "SpiderWeb") – laut Wiki ein eigenständiger, gut beschriebener Raumtyp (3 Risikostufen, Schlüssel vs. Gift), plausibel korrekt benannt, aber noch nie in echten Daten gesehen
- [x] Restliche Golden-Room-states (302, 311, 313, 319, 320, 324–328 im marenga-Port benannt) geklärt: **Edition-exklusiv** (Geburtstags-/Halloween-/LOTR-Editionen, siehe "Edition-exklusive Goldene Räume") – deshalb nie im aktuellen Event gesehen, kein Datenproblem
- [x] Fluchhändler (state=323) und `IADungeonDebuffMerchantBuy`: bestätigt 2026-09-19, Effekt `104` (GoldRushHangover) für 1-2 Schlüssel gekauft. Offizieller Name: Scheibenkleistermeister.
- [ ] **Neu aufgemacht:** `iadungeonsave[1]` – weder "Dungeon-Typ/Run-Nummer" (alte Doku) noch "HealthStatus" (marenga-Port) passen zu den Daten (siehe Korrektur-Hinweis oben bei den Feldern)
- [ ] **Neu:** Boss-Monster-IDs scheinen Theme-abhängig zu sein – aktuelles Event (`AbyssOfMadness`) nutzt `-5297`(Boss 1)/`-5298`(Boss 2), komplett anders als die alte Tabelle (`-514x`). Zweiter Run nötig, um Variante A/B zu klären
- [x] Segen/Fluch-Mechanik (Stapel-, Slot- und Boss-Immunitätsregeln) offiziell dokumentiert, siehe "Segen & Flüche – Mechanik-Regeln"
- [ ] Pilz-Preisstaffelung bei mehrfacher Heilnutzung im selben Run (10→15→20 laut Wiki) noch nicht mit echten Daten verifiziert
- [x] **Neu (2026-09-20):** Enum-Namen (marenga-Port) für zahlreiche bereits dokumentierte States/Monster live bestätigt: Hungrige Türen (`Wood`/`Stone`/`Metal`/`Arcane`/`Souls`/`QuicksandGlasses`), Goldene Räume (`FountainOfLife`=301, `PileOfRocks`=303, `TheFloorIsLava`=304, `UnlockedSarcophagus`=312, `SoulBath`=321), Truhen (`BronzeChest`/`SilverChest`/`EpicChest`/`MimicChest`=500/`SatedChest`=603), Skelette (`MageSkeleton`=300/`WarriorSkeleton`=301), Prüfungspforte (`TrialRoom1`), Segen (`LockPick`=buff_id 5)
- [ ] **Neu (2026-09-20):** Zwei neue Segen-Namen beim Schlüsselmeister gekauft, buff_id unbekannt: `KeyMoment`, `EscapeAssistant`. Sowie `ElixirOfLife` als vermutliches Lebenselixier (Verbrauchsgegenstand, kein Segen)
- [ ] **Neu (2026-09-20):** `UndeadFiend`-Raumbonus nach goldener Tür beobachtet – vermutlich das schon bekannte Laternenmonster (state=310), aber ohne expliziten State-Beleg noch unbestätigt
- [ ] **Neu (2026-09-20):** Belohnungen `Crate1`/`Crate2`/`Crate3` hinter `LockedDoor` – Zuordnung zu den `monster`-IDs der Kisten/Fässer-Tabelle noch offen

---

## Quellen

- HAR-Aufzeichnungen: 80+ Charakter-Runs auf verschiedenen Servern (F9, F25, F28)
- Gezielter zweiter Durchgang 2026-09-19: 7 Accounts (F9, F23, F25, F28, S5, S6, S7)
  plus ein Nachschlag mit weiteren Charakteren auf F9, F25 und F28 (teils mehrere
  pro Server, teils in Sammel-HARs mit mehreren Accounts pro Datei),
  aufgezeichnet um die Annahmen im marenga-PR `feat/legendary-dungeon`
  (the-marenga/mfbot#454, ungetestet) zu verifizieren
- Playa Games Helpshift (offizielles Wiki, 2026-09-19 vollständig ausgewertet):
  - Übersicht: https://playa-games.helpshift.com/hc/de/4-shakes-fidget-1653988985/faq/57-legendary-dungeon/
  - Türen: https://playa-games.helpshift.com/hc/de/4-shakes-fidget-1653988985/faq/282-legendary-dungeon---doors/?p=web
  - Truhen: https://playa-games.helpshift.com/hc/de/4-shakes-fidget-1653988985/faq/283-legendary-dungeon---chests/?p=web
  - Goldene Räume: https://playa-games.helpshift.com/hc/de/4-shakes-fidget-1653988985/faq/284-legendary-dungeons---golden-rooms/?p=web
  - Das In-Game-Infofenster verlinkt exakt auf dieselben Wiki-Seiten – deckt
    sich, keine widersprüchlichen Quellen
- ldgadget.12hp.de: https://ldgadget.12hp.de
