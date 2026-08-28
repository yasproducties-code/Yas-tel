# HANDOFF — Herbouw draaiplanning-template (Google Sheets + Apps Script)

Plak dit hele bestand als eerste bericht in een nieuwe sessie.
Alles wat al is uitgezocht staat in deze map; niets hoeft opnieuw te worden geanalyseerd.

---

## WIE JE BENT

Je werkt als een zeer ervaren Google Apps Script-ontwikkelaar die begrijpt hoe een
filmplanning dagelijks door een First AD op de set wordt gebruikt. Een technisch correcte
oplossing is niet genoeg als die in dagelijks gebruik onlogisch of omslachtig is.

De opdrachtgever is Yassin van Veenendaal, First AD. Hij is Nederlandstalig; antwoord in het
Nederlands. Hij typt snel en met typefouten — lees door de spelling heen.

---

## DE MAP

Alle documentatie staat in `docs/the-leecher/` in de repo `yasproducties-code/Yas-tel`,
branch `claude/decoupage-shotlist-format-rvm8a1`.

| Bestand | Wat erin staat |
|---|---|
| `00-HANDOFF.md` | dit bestand |
| `01-opdracht.md` | de definitieve opdracht van de opdrachtgever, letterlijk |
| `02-analyse-bestaand-systeem.md` | hoe het oude systeem (Luza, generator v3.1) technisch werkt |
| `03-projectdata-the-leecher.md` | alle projectgegevens: dagen, scènes, tijden, cast, locaties |
| `04-shotlist.md` | 15 scènes, 50 shots, met zoom-kolom |
| `05-ontwerp-nieuwe-template.md` | **het technische ontwerp dat gebouwd moet worden** |
| `06-besluiten.md` | de antwoorden op alle 8 vragen + de 2 resterende punten |
| `bronnen/` | ruwe uitlezingen: PDF-tekst, XLSX-tabbladen, shotlist-brontekst |

**Vraag de opdrachtgever om de originele Apps Script-code v3.1 mee te sturen** als je die
nodig hebt. Die staat niet in deze map (hij is ~1200 regels en wordt toch vervangen);
`02-analyse-bestaand-systeem.md` bevat alle relevante mechanismen, formules en valkuilen eruit.

---

## STATUS

| Fase | Status |
|---|---|
| Analyse bestaande XLSX (27 tabbladen) | **klaar** |
| Analyse bestaande Apps Script v3.1 | **klaar** |
| Analyse planning + shotlist THE LEECHER | **klaar** |
| Vragenblokken met opdrachtgever | **klaar** — hij heeft de definitieve opdracht gegeven (`01-opdracht.md`) |
| Ontwerp nieuwe template | **klaar**, zie `05-ontwerp-nieuwe-template.md` |
| 8 laatste vragen | **beantwoord** — zie `06-besluiten.md` |
| Code geschreven | **nee, nog niets** |

**Alles is beslist. Het ontwerp staat vast en je kunt direct bouwen.**
Twee kleine gegevens ontbreken nog (naam + Maps-link van de koffiezaak, en de figuratie bij
scène 1, 8 en 11 — zie `06-besluiten.md`). Die blokkeren de bouw niet: de structuur kan gebouwd
worden en die twee waarden kunnen daarna in de inputtabbladen worden ingevuld. Vraag ze wel na.

---

## WAT JE MOET DOEN

1. Lees `01-opdracht.md` (de opdracht), dan `05-ontwerp-nieuwe-template.md` (het ontwerp),
   dan `06-besluiten.md` (alle genomen besluiten).
2. Bouw af. Er hoeft niets meer gevraagd te worden behalve de twee kleine gegevens uit
   `06-besluiten.md`, en die kun je tijdens het bouwen stellen.
3. Loop na afloop de controlelijst hieronder af en rapporteer eerlijk wat wel en niet is getest.

### Manier van opleveren — belangrijk

Je kunt vanuit een sessie geen Google Sheet aanmaken. De afgesproken oplossing:

> **Eén Apps Script-bestand dat de spreadsheet zelf bouwt.**
> De opdrachtgever maakt een lege Google Sheet, plakt de code in Extensies ▸ Apps Script,
> draait één keer `Draaiplanning ▸ Installeer template`, en dan staan alle tabbladen,
> de volledige opmaak, de configuratie én de LEECHER-projectdata erin.
> Voor een volgende productie: dezelfde installer in een nieuwe sheet, alleen de inputrijen
> vervangen.

Dat is met de opdrachtgever besproken en akkoord bevonden als aanpak.

### Eisen aan de code (van de opdrachtgever)

- Volledige, werkende Apps Script-code. Geen pseudocode, geen placeholders, geen losse
  fragmenten, geen functies die hij zelf moet aanvullen, geen half geïmplementeerde functies.
- Geen hardcoded rijnummers of celadressen zonder centrale, verklaarde structuur.
- Geen aannames die alleen voor dit ene project werken. De template moet werken voor een
  wisselend aantal draaidagen, scènes, shots, locaties, castleden en planregels.
- Geen oude verwijzingen naar verwijderde onderdelen, geen stille verwijdering van
  functionaliteit.
- Verzin geen projectinformatie. Kies niet zelf als meerdere interpretaties mogelijk zijn.

### Controles die na de bouw moeten slagen

Scène zonder shots · scène met één shot · scène met meerdere 0-minutenshots · meerdere scènes
met verschillende duren · hele scène verplaatsen inclusief shots · shotvolgorde binnen een
scène wijzigen · shot zonder scènekoppeling · ontbrekende of ongeldige duur · pauzes en
verplaatsingen tussen scènes · totale draaitijd per dag · starttijd van een scène ná meerdere
0-minutenregels · alle formules na verschoven rijposities · opnieuw genereren zonder verlies
of verdubbeling van gegevens.

---

## DE VIJF DINGEN DIE JE MOET WETEN VOORDAT JE BEGINT

Deze zijn met moeite uitgezocht. Als je ze negeert bouw je de fouten van het oude systeem na.

**1. De scène draagt de tijd, de shots staan op 0.**
Dit is de kern van de opdracht en het is precies andersom dan in het oude systeem. Daar
droegen de shotregels de tijd en stond de scènekop op 0 — en `validateBlockCoverage_` dwong
af dat de som van de shotminuten exact gelijk was aan de scèneduur. Die validatie moet weg.
Zie `02-analyse-bestaand-systeem.md` §4 en `05-ontwerp-nieuwe-template.md` §5.

**2. Nul-minutenregels zijn al transparant.**
De formuleschil `=IF(B18="","",…)` zorgt dat een regel met 0 minuten `D = C` geeft, en dat een
regel met een **lege** minutencel een lege Van én Tot geeft. Je hoeft daar niets voor te
bouwen; je moet het alleen goed gebruiken.

**3. De volgorde komt uit de rijvolgorde, niet uit de tijd.**
Er wordt nergens op tijd gesorteerd. Dat is precies waarom shots met dezelfde starttijd niet
door elkaar kunnen raken. Ga niet alsnog sorteren.

**4. De oude generator schreef naar tab `DD#1`, terwijl de tabbladen `DD#1 + fotos` heten.**
Daardoor vond hij ze niet terug en maakte hij bij elke run een nieuwe lege tab ernaast. Dát is
de reden dat het oude bestand vol staat met "versie 2", "oud", "(1)", Blad22 en Blad23. In het
nieuwe ontwerp is dit opgelost (zie `05-ontwerp-nieuwe-template.md` §7).

**5. De oude code schreef de kop naar vaste celadressen (B2, F2, J4, F12…).**
Zodra iemand een kolom invoegde, schreef hij ernaast. Dat is in het oude bestand al gebeurd.
In het nieuwe ontwerp worden kopcellen opgezocht via hun label in het sjabloon.

---

## TOON EN WERKWIJZE

- Werk in vragenblokken, niet alles tegelijk, en wacht op antwoord — behalve als hij
  uitdrukkelijk zegt door te gaan.
- Waar je een voorstel doet, zet er `[voorstel: …]` bij zodat hij met "ok" kan antwoorden.
- Maak onderscheid tussen: wat letterlijk in een bron staat · wat logisch af te leiden is ·
  wat ontbreekt · wat tegenstrijdig is · wat een ontwerpkeuze is die hij moet maken.
- Benoem het als iets niet betrouwbaar te controleren is. Verzin dan niet hoe het waarschijnlijk werkt.
- Hij vraagt uitdrukkelijk om kritisch meedenken: zie je iets dat slimmer, logischer of
  professioneler kan, doe dan eerst een voorstel voordat je het implementeert.
