# 02 — Analyse van het bestaande systeem (Luza, generator v3.1)

Bron: `Luza_kan_alles_zelf__draaiplanning.xlsx` (27 tabbladen, 22 MB) en de Apps Script-code v3.1
(~1200 regels, één bestand, geen bibliotheken). Ruwe uitlezingen staan in `bronnen/`.

Dit systeem wordt **vervangen**, niet aangepast. Deze analyse staat er zodat je de bruikbare
mechanismen overneemt en de valkuilen niet herhaalt.

---

## 1. Tabbladen

Zes systeemtabbladen voeden één generator die per draaidag een outputtabblad wegschrijft.

| Tabblad | Rol |
|---|---|
| `Day_Info` | 1 rij per draaidag: kopgegevens, shooting call, opbouwminuten |
| `Opbouw_Input` | activiteiten tussen crew call en shooting call |
| `Day_Blocks` | geordende blokken per dag, mét duur |
| `Decoupage_Input` | rijen binnen een scèneblok, gekoppeld via `block_id` |
| `Template_Config` | key/value-instellingen |
| `Draaiplanning_Template` | één voorbeeldrij per rijsoort; alle opmaak komt hier vandaan |

De rest van de 27 tabbladen is rommel: gegenereerde dagen, handmatige kopieën (`Blad22`,
`Blad23`), "oud"-versies en `(1)`-duplicaten van de zes systeemtabbladen.

## 2. Rijopbouw van een gegenereerde dag

De generator kopieert sjabloonrijen met `copyTo(PASTE_NORMAL)` en telt de outputrijen:

```
rij 1..crewcall_row      1-op-1 uit het sjabloon (de hele kop)
+ N                      opbouw_activity_row, 1 per rij in Opbouw_Input (minimaal 1)
+ 1                      shooting_call_row   <- ankercel van de tijdcascade
+ 1                      strip_header_row    <- bevroren
+ 1 per strip            scene_header_row / setup_row / sub_row / special_row /
                         marker_row / est_wrap_row / lunch_row / wrap_row
+ 2                      footer_row, closing_row
```

De rijnummers in `Template_Config` verwijzen naar **rijen in het sjabloon** (stijldonoren),
niet naar rijen in de output. Het aantal opbouwregels bepaalt waar alles daaronder landt.

## 3. Tijdlogica — overnemen

Alle tijden zijn formules, nooit getypte waarden.

```
eerste strip   C = =IF(B18="","",$B$<shooting_call_row>)
elke strip     D = =IF(B18="","",C18+(B18/1440))
volgende       C = =IF(B19="","",D<vorige strip IN de cascade>)

kind call      C = =TIMEVALUE("12:00")   D = =TIMEVALUE("12:00")+(60/1440)
kind wrap      C = D = =TIMEVALUE("19:00")
parallel       C/D = de twee tijden uit de tijdrange-kolom

scenekop       kolom 7 = =TEXT(C<kop>,"HH:MM")&"-"&TEXT(D<laatste strip van blok>,"HH:MM")
```

Vier eigenschappen die er direct toe doen:

1. **0 minuten is transparant** — `D = C`, de volgende regel begint op dezelfde tijd.
2. **Lege minutencel geeft lege Van én Tot** — dankzij de `IF(B="","",…)`-schil.
3. **De volgorde komt uit de rijvolgorde van de invoer**, niet uit de tijd. Er wordt nergens
   op tijd gesorteerd (alleen `plaatsKindStrips_` verplaatst call/wrap-strips). Daardoor kunnen
   regels met dezelfde starttijd niet door elkaar raken.
4. **Strips buiten de cascade worden overgeslagen.** `writeStripValues_` houdt bij wat de vorige
   strip *in* de cascade was, zodat een call of parallelle strip de tijdlijn niet verschuift.
   Precies dit mechanisme hebben de shots straks nodig.

## 4. De scène/shot-verhouding — dit moet omgekeerd

In het oude systeem staat de **scènekop op 0 minuten** en dragen de **shotregels de tijd**.
Dat is geen slordigheid maar wordt afgedwongen:

```js
var sum = rows.reduce(function(total, row) { return total + row.minutes; }, 0);
if (sum !== block.durationMinutes) {
  throw new Error('Blok "' + block.blockId + '": decoupage minuten tellen op tot ' + sum +
    ' maar Day_Blocks zegt ' + block.durationMinutes + '. Die moeten gelijk zijn.');
}
```

Gecontroleerd over alle 37 scèneblokken en 88 shotregels: de som klopte overal exact.
**In het nieuwe systeem moet deze validatie verdwijnen** en draagt de scène de tijd.

## 5. Validaties die de oude code afdwingt

| Functie | Eis |
|---|---|
| `readDayInfo_` | precies 1 rij per dag; verplicht: titel, datum, regisseur, dop, first_ad, scenes, cast, locatie, adres, shooting_call, opbouw_minuten |
| `readBlocksForDay_` | precies 1 wrap per dag, max 1 lunch, unieke block_id per scène, na de wrap alleen kind_call/kind_wrap of parallelle strips |
| `validateBlockCoverage_` | elk scèneblok heeft shots én de minuten tellen exact op tot de scèneduur |
| `validateEstWraps_` | na een est_wrap mag die acteur nergens later meer voorkomen |
| `readDecoupageForDay_` | block_id verplicht; `setup` vereist een `nr`; kind_call/kind_wrap vereisen een call_tijd |
| `getTemplateBindings_` | `notes`, `beschrijving` en `acteurs` zijn verplichte output_columns |

De laatste vier zijn de moeite waard om in aangepaste vorm mee te nemen.

## 6. Wat er misgaat in het oude systeem

**Outputtab-naam.** `resolveOutputSheet_` maakt of overschrijft de tab `DD#1`. De tabbladen in
de spreadsheet heten `DD#1 + fotos` — met de hand hernoemd na generatie. De generator vindt ze
daardoor niet meer terug en maakt bij elke run een verse lege `DD#1` ernaast. Dat verklaart alle
duplicaten in het bestand. Het verklaart óók waarom foto's en handmatige aanvullingen nooit
verloren gingen: ze stonden in een tabblad dat de generator niet aanraakt.
`renderDay_` → `resetOutputSheet_` doet `breakApart()` + `clear({contentsOnly:false})` op het
hele tabblad.

**Hardcoded celadressen.** De kop wordt geschreven naar vaste A1-adressen, ongeacht de
kolomindeling:

```
B2 titel · F2 daglabel · B4 regisseur · B6 dop · B8 first AD · B10 adres
J4 scenes · J6 cast · J8 locatie · B12 aantal shots
F4 crew call · F6 shooting call · F8 lunch · F10 wrap · J10 lengte draaidag crew
F12 uren Heuvelaar · J12 uren Luza      <- namen EN cellen hardcoded
kolom 7 = tijdrange scenekop · kolom 8 = synopsis · kolom 2/3/4/5 = opbouwsectie
```

`setKop_` vangt samengevoegde cellen op, maar niet ingevoegde kolommen. In het Luza-bestand zijn
twee kolommen bijgestoken voor foto's, waardoor F2 naar H2 schoof en J4 naar L4. Code en
zichtbare tabbladen lopen daardoor uit de pas.

**Dode instellingen.**
- `luza_uren_cel`, `heuvelaar_uren_cel`, `daglabel_cel`, `tijdrange_kolom_dd2` staan in
  Template_Config maar worden **nergens door de code gelezen**.
- `kind_strips_op_inputpositie` wordt door de code gelezen maar **ontbreekt in Template_Config**.
  Gevolg: `plaatsKindStrips_` is in dit bestand actief en herschikt call/wrap-strips op tijd,
  terwijl de v3.1-toelichting zegt dat ze op invoerpositie blijven.
- `ignore_input_columns` idem: gelezen, ontbreekt. Onschadelijk zolang output_columns bestaat.
- De cast-helperformules die `writeKop_` in de helperkolom zet worden door niets gelezen.
  Dat zijn precies de "Moussa"/"Leonor"-weesformules die nog in het bestand staan.
- `est_wrap` en `special` bestaan in de code maar worden in de data nergens gebruikt.

**Bediening.** Geen `onOpen`, geen menu, geen knoppen, geen triggers. Alles draait vanuit de
Apps Script-editor. `genDD1()` t/m `genDD5()` zijn hardcoded op vijf dagen.

**"Kim Uren"** — de opdrachtgever noemde drie te verwijderen namen: Kim Uren, Heuvelaar, Loesa.
"Kim" en "Loesa" komen nergens voor, niet in de spreadsheet en niet in de code. "Kim Uren" is
**"kinduren"**: de code heeft een functie `writeKindDayLength_` met commentaar over
"de kinduren in de kop" en "kindcall-tijd (BOVK)" — werktijden van kindacteurs. Heuvelaar en
Luza waren de twee kindacteurs; "Loesa" is Luza. Drie namen, één concept. **In de nieuwe
template vervalt dit volledig** (bevestigd door de opdrachtgever).

## 7. Wat niet uit de XLSX te halen was

- Het bestand bevat nul voorwaardelijke opmaakregels, nul gegevensvalidaties en nul benoemde
  bereiken. De code stelt ze ook niet in; alle opmaak komt uit het sjabloon. Een XLSX-export
  van Google Sheets neemt hier niet alles betrouwbaar van mee, dus dat blijft een klein voorbehoud.
- De code plaatst geen afbeeldingen. De foto's in het Luza-bestand zijn handmatig toegevoegd,
  net als de REF-kolom die in geen enkele kolomset voorkomt.
- De zichtbare dagtabbladen zijn **niet** de scriptoutput maar handmatig bewerkte kopieën.
  Behandel ze als voorbeeld van het gewenste eindresultaat, niet als referentie voor wat de
  generator produceert.
