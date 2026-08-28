# 05 — Ontwerp nieuwe draaiplanning-template

Dit is wat gebouwd moet worden. Het volgt de opdracht in `01-opdracht.md`; alles wat hier extra
staat is een **voorstel** en is als zodanig gemarkeerd.

---

## 1. Opleveringsvorm

Eén Apps Script-bestand dat de spreadsheet **zelf bouwt**.

```
Lege Google Sheet  →  Extensies ▸ Apps Script  →  code plakken  →  opslaan  →  herladen
                   →  menu Draaiplanning ▸ Installeer template
                   →  alle tabbladen, opmaak, config en projectdata staan erin
```

Voor een volgende productie: dezelfde installer in een nieuwe sheet, alleen de inputrijen
vervangen. Dat is wat "één definitieve template voor al mijn toekomstige producties" praktisch
betekent.

De installer moet **idempotent** zijn: opnieuw draaien mag bestaande invoer niet wissen.
Voorstel: `installeerTemplate()` maakt ontbrekende tabbladen aan en herstelt het sjabloon en de
config, maar laat gevulde inputtabbladen met rust. Een aparte
`installeerTemplate_metVoorbeelddata()` vult ook de LEECHER-data.

## 2. Tabbladen

Zes systeemtabbladen, Nederlandse namen (voorstel — het oude systeem gebruikte Engelse):

| Tabblad | Rol | Was |
|---|---|---|
| `Dagen` | 1 rij per draaidag: kopgegevens | Day_Info |
| `Opbouw` | activiteiten tussen crew call en shooting call | Opbouw_Input |
| `Blokken` | geordende blokken per dag, mét duur | Day_Blocks |
| `Shots` | shots per blok, gekoppeld via `blok_id` | Decoupage_Input |
| `Config` | key/value-instellingen | Template_Config |
| `Sjabloon` | één voorbeeldrij per rijsoort; alle opmaak komt hier vandaan | Draaiplanning_Template |

Output per draaidag: `DD1`, `DD2`, … — **exact de naam die de generator ook terugvindt**.
Geen suffix, geen hernoemen (zie §7).

## 3. Kolomindeling

Elf inhoudskolommen, precies zoals opgedragen, plus twee randkolommen:

| Kolom | Inhoud | Breedte (voorstel) |
|---|---|---|
| A | zwarte linkermarge | 10 px |
| B | Tijd (min) | 55 |
| C | Van | 52 |
| D | Tot | 52 |
| E | Notes | 210 |
| F | Foto | 150 |
| G | Nr. | 46 |
| H | Soort | 105 |
| I | Movement | 95 |
| J | Zoom | 90 |
| K | Beschrijving | 300 |
| L | Cast/Figuratie | 130 |
| M | helper + zwarte rechtermarge | 10 px |

Kolom M is tegelijk de verborgen rekenkolom en de rechterrand: zwarte vulling met zwarte tekst,
zodat waarden onzichtbaar zijn maar de rand even dik als links. Dat mechanisme uit het oude
systeem werkt goed en kan mee.

## 4. Header — 12 velden, drie kolommen van vier

```
rij 1    spacer
rij 2    B: THE LEECHER                    I: DD1   |   zo 30-08-2026
rij 3    Regisseur       Crew call         Scènes                  <- labels
rij 4    Daniël Bakker   09:30             1, 12, 13, 9, 4, ...    <- waarden
rij 5    D.O.P.          Shooting call     Cast                    <- labels
rij 6    Sean Louw       10:15             Rens, Vrouw, Jongen...  <- waarden
rij 7    First AD        Lunch             Locatie                 <- labels
rij 8    Yassin van V.   15:00             Vondelpark  (hyperlink) <- waarden
rij 9    Shots           Wrap              Lengte draaidag crew    <- labels
rij 10   17              19:30             10:00                   <- waarden
rij 11   TIJD | VAN | TOT | ACTIVITEIT
rij 12   crew call
rij 13.. opbouwregels (N stuks, uit `Opbouw`)
  +1     shooting call            <- ankercel van de tijdcascade
  +1     stripkop (bevroren)
  +1..   de planregels
  +1     footer
  +1     afsluitrand
```

Merges: linkerblok B:E, middenblok F:H, rechterblok I:L.
De kop is hiermee **twee rijen korter** dan het oude sjabloon (crewcall stond op rij 14, nu 12),
precies zoals gevraagd.

**Locatie** is één veld met een hyperlink; geen apart adresveld.
**Lengte draaidag crew** is één veld: wrap − crew call.

### Kopcellen worden opgezocht, niet hardcoded

Dit is de belangrijkste technische verbetering ten opzichte van het oude systeem. Daar stonden
`B2`, `F2`, `J4`, `F12` hard in de code, waardoor één ingevoegde kolom alles verschoof.

Nieuw: bij het genereren scant de generator het sjabloon op de **labelteksten**
(`Regisseur`, `Crew call`, `Locatie`, …) en schrijft de waarde in de cel eronder. Verplaatst
iemand een blok of voegt hij een kolom in, dan blijft het werken. Config bevat alleen nog de
lijst met veldnamen en hun volgorde, niet hun celadres.

## 5. Tijdlogica — scène draagt de tijd

| Rijsoort | Tijd (min) | Van | Tot | In de cascade? |
|---|---|---|---|---|
| Scènerij | duur van de scène | formule | formule | **ja** |
| Eerste shot van die scène | leeg | `=C<scenerij>` | `=D<scenerij>` | nee |
| Overige shots | leeg | leeg | leeg | nee |
| Move / lunch / repetitie / opbouw / pick-up | eigen duur | formule | formule | ja |
| Call / wrap per acteur | leeg | absolute tijd | absolute tijd | nee |

De formules blijven zoals in het oude systeem:

```
eerste rij in de cascade   C = =IF(B<r>="","",$C$<shooting_call_rij>)
elke rij in de cascade     D = =IF(B<r>="","",C<r>+(B<r>/1440))
volgende rij               C = =IF(B<r>="","",D<vorige rij IN de cascade>)
```

Omdat shotregels een **lege** minutencel krijgen, geven `C` en `D` daar vanzelf een lege
waarde — precies wat de opdracht vraagt ("alle overige shots: geen tijd tonen"). En omdat
ze buiten de cascade staan, verwijst de volgende scènerij naar de **scènerij** erboven, niet
naar de laatste shot. Dubbeltelling is daarmee structureel onmogelijk in plaats van via een
optelregel.

**De validatie `validateBlockCoverage_` uit het oude systeem moet verdwijnen.** Die dwong af dat
de som van de shotminuten gelijk was aan de scèneduur. In plaats daarvan:
- elke scène moet een duur hebben (0 is toegestaan, bv. scène 7 en 10);
- geen enkele shotregel mag een duur dragen;
- elke shot moet aan een bestaande scène gekoppeld zijn;
- elke scène in `Blokken` moet voorkomen in de dagvolgorde.

**Volgorde:** nooit op tijd sorteren. De rijvolgorde van `Blokken` en `Shots` is de waarheid.

## 6. Rijsoorten

| Soort | Opmaak | Duur | Gebruik in dit project |
|---|---|---|---|
| `scene` | donkerblauwe kopregel | eigen duur | alle 15 scènes |
| `blok` | zalmroze regel met minuten | eigen duur | move, company move, repetitie, opbouw, pick-up |
| `lunch` | donkerrode regel | eigen duur | de lunch |
| `call` | lichte regel, absolute tijd | 0 | call per acteur/figurant |
| `wrap` | donkerrode afsluiting | 0 | einde draaidag |
| `shot` | witte regel | leeg | de 50 shots |

**Bloklabels** (Castblok, Docublok, Rens solo blok) krijgen géén eigen regel. `Blokken` krijgt
een kolom `label`; staat daar iets in, dan zet de generator die tekst als **extra regel binnen
de scènerij**, boven de scènetitel. Zo staat "Docublok" waar het hoort zonder een lege
tijdregel te kosten.

## 7. Opnieuw genereren

Het oude systeem schreef naar `DD#1` terwijl de tabbladen `DD#1 + fotos` heetten; daardoor
ontstond bij elke run een nieuwe tab. In het nieuwe ontwerp:

- de generator schrijft naar `DD1` en vindt die de volgende keer terug;
- vóór het wissen leest hij de **Foto-kolom** uit, gekoppeld aan het shotnummer, en zet die
  daarna terug. **[voorstel — nieuwe functionaliteit, expliciet akkoord nodig]** Dit werkt als
  de foto's als `=IMAGE("url")` in de cel staan in plaats van als zwevende afbeelding. De
  installer zet daarom een korte instructie in de Foto-kop.
- alles wat níét in de inputtabbladen staat en niet de Foto-kolom is, gaat bij regeneratie
  verloren. Dat moet zo zijn: de inputtabbladen zijn de bron van waarheid.

## 8. Menu

`onOpen` bouwt het menu **Draaiplanning**:

```
Draaiplanning
├── Genereer DD1
├── Genereer DD2
├── … (één regel per rij in `Dagen`)
├── ───────────────
├── Genereer alles
├── ───────────────
├── Controleer invoer
└── Installeer / herstel template
```

**Technische noot:** Apps Script kan geen menu-items koppelen aan functies die pas tijdens het
draaien ontstaan. Oplossing: een vaste pool van functies `genDag1()` … `genDag12()` die elk
`genereerDagOpIndex_(n)` aanroepen; het menu toont er zoveel als er rijen in `Dagen` staan, met
het daglabel uit de sheet als tekst. Twaalf draaidagen is ruim; verhoog de pool als het ooit
meer moet zijn. Dit is de enige plek waar een vast maximum zit, en dat moet in de code
gedocumenteerd staan.

`Controleer invoer` is de opvolger van `checkTemplate()`: leest alle inputtabbladen, meldt
ontbrekende duren, losse shots, scènes zonder shots, dubbele blok-ID's en kolomproblemen, en
wijzigt niets.

## 9. Calltijden automatisch berekenen

Regel uit de opdracht: acteurs 45 minuten vóór het eerste draaiklaar moment. De opdrachtgever
heeft daarna besloten dat **figuratie diezelfde 45-minutenregel krijgt**, niet de eerder
genoemde 30 minuten.

De generator kan dat zelf uitrekenen: zoek per rol de eerste scène van de dag waarin die rol
voorkomt, trek 45 minuten van de starttijd af, en zet een callregel boven aan de dag.
Schuift een scène op, dan schuift de call automatisch mee — dat scheelt de First AD handwerk en
voorkomt fouten.

Handmatig overschrijven blijft mogelijk door in `Blokken` een regel van soort `call` met een
expliciete tijd op te nemen; die wint dan van de berekening.

**Weergave (besloten):** één regel per persoon, niet kleding en make-up apart. Bijvoorbeeld
`Call Rens 09:30 — kl/mu — draaiklaar 10:15`.
**Figuratie krijgt eigen callregels én wordt vermeld in de kolom Cast/Figuratie bij de scène.**
**Est. wrap per acteur: niet opnemen** — daar is uitdrukkelijk nee op gezegd.

## 10. Wat er níét in komt

Uitdrukkelijk uit de opdracht: geen takes, geen afvinkvakjes, geen "gedraaid ja/nee", geen
opmerkingenvelden, geen extra kolommen, geen aparte blokregels, geen kinduren, geen apart
adresveld, geen sublocatienamen, geen pre-calls.

## 11. Aandachtspunten bij de bouw

- Centraliseer élke positie (rijsoorten, kolommen, veldnamen) in één configuratieblok boven in
  de code, met commentaar erbij. Geen losse getallen in functies.
- Alle afgeleide tijden zijn formules, nooit getypte waarden.
- Werk met `LockService` zodat twee runs niet door elkaar lopen.
- Controleer na het genereren op `#`-fouten in Van/Tot en geef een leesbare foutmelding met
  celadres, zoals `verifyCascade_` in het oude systeem deed.
- Denk aan het scheidingsteken in formules: Google Sheets gebruikt afhankelijk van de landinstelling
  een komma of een puntkomma. Het oude systeem detecteerde dat door een testformule te schrijven
  en weer te wissen; dat werkt, maar een nettere oplossing verdient de voorkeur.
- Verberg de systeemtabbladen na installatie, maar laat ze via het menu terug te halen zijn.
