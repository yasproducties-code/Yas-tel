# 01 — De definitieve opdracht (letterlijk, 28-08-2026)

> **Definitieve opdracht – Herbouw draaiplanning-template**
>
> Je gaat mijn bestaande draaiplanning-template volledig herstructureren. Het doel is één
> definitieve template die ik op al mijn toekomstige producties kan gebruiken. Denk dus niet
> alleen aan deze film, maar ontwerp een professionele, generieke workflow voor een First AD.
>
> **Algemeen**
> - Maak een nieuwe, schone spreadsheet.
> - Verwijder alle oude SIIMON-restanten, dubbele tabbladen en legacy-onderdelen.
> - Bouw alleen de definitieve systeemtabbladen opnieuw op.
> - Denk kritisch mee. Zie je iets dat slimmer, logischer of professioneler kan, doe dan een
>   voorstel voordat je het implementeert.
>
> **Generator**
> Ik wil géén Apps Script-editor meer hoeven openen. Maak daarom een menu **Draaiplanning** met:
> Genereer DD1 · Genereer DD2 · Genereer DD3 · … (voor iedere draaidag een eigen knop) ·
> Genereer alles.
> Dus niet één algemene dagknop, maar één knop per draaidag én één knop om alles tegelijk te
> genereren.
>
> **Header opnieuw ontwerpen**
> De huidige header mag opnieuw worden opgebouwd. Verwijder de huidige rijen 11 en 12 uit de
> template. Verwijder: kinduren · alle oude LUZA/Heuvelaar-velden · overige overbodige informatie.
>
> De header moet uiteindelijk uit 12 velden bestaan:
> 1. Regisseur 2. D.O.P. 3. First AD 4. Shots 5. Crew call 6. Shooting call 7. Lunch 8. Wrap
> 9. Scènes 10. Cast 11. Locatie 12. Lengte draaidag crew
>
> Belangrijk: Lengte draaidag crew is één veld. Shots blijft een eigen veld. Er is nog maar
> één locatieveld. Geen apart adresveld meer.
>
> **Locatie**
> In de header staat alleen: Locatie → bijvoorbeeld "Vondelpark". Dat veld bevat een hyperlink
> naar Google Maps. Niet ook nog een apart adresveld. Per scène mag wél een eigen Google
> Maps-link bestaan. De sublocatienamen zoals grasveld, bankje, poort, fietspad hoeven nergens
> meer zichtbaar te zijn. Alle scènes hebben simpelweg als locatie: Vondelpark, met de juiste
> hyperlink.
>
> **Productiegegevens**
> Regisseur: Daniël Bakker · D.O.P.: Sean Louw · First AD: Yassin van Veenendaal
>
> **Dagstructuur**
> Crew call 09:30 · Wrap 19:30 · Geen pre-calls · Lunch 15:00, 45 minuten in de planning.
> In communicatie blijft dat: 30 minuten lunch + 15 minuten buffer.
>
> **Cast** — gebruik alleen rolnamen. Nummering:
> 1 Rens · 2 Selma · 3 Vrouw · 4 Jongen · 5 Prullenbakman · 6 Man
>
> **Calls**
> Acteurs: standaard 45 minuten vóór eerste draaiklaar moment callen. Daarna: kleding,
> make-up, naar set. Figuratie: 30 minuten vóór inzet aanwezig.
>
> **Bronnen**
> Gebruik de Planning PDF als leidend voor: draaidagen · volgorde · tijden · lunch · cast.
> Gebruik de shotlist alleen voor: de shots binnen de scènes.
>
> **Conflicten opgelost** — gebruik onderstaande versie:
> - Scène 4: de vrouw wijst Rens af.
> - Scène 8: is het fietspad.
> - Scène 10: is het bankje.
> - Scène 12: Prullenbakman is rol 5. Niet dezelfde als rol 6.
> - Scène 7: wordt samen met scène 5 gedraaid.
> - Scène 15 en 15.2: zijn één scène. De lunch valt ertussen.
>
> **Synopsis** — gebruik altijd de korte synopsis uit de planning. Niet de uitgebreide uit de
> shotlist.
>
> **Bloklabels** — gebruik geen aparte blokregels. Wanneer nodig mag "Docublok", "Castblok"
> enzovoort als extra regel binnen de scènerij worden weergegeven.
>
> **Kolommen** — gebruik: Tijd · Van · Tot · Notes · Foto · Nr. · Soort · Movement · Zoom ·
> Beschrijving · Cast/Figuratie. Geen extra kolommen toevoegen.
>
> **Tijden** — op de scènerij: toon de volledige tijd. Bij het eerste shot: herhaal die tijd.
> Alle overige shots: geen tijd tonen.
>
> **Invulvelden** — geen takes, afvinkvakjes, gedraaid ja/nee of opmerkingenvelden.
> De planning is bedoeld als leesbare draaiplanning.
>
> **Tot slot** — controleer na afloop de volledige spreadsheet alsof je hem zelf als ervaren
> First AD zou gebruiken. Zie je nog mogelijkheden om de template overzichtelijker, sneller of
> professioneler te maken zonder functionaliteit te verliezen, doe dan eerst een voorstel
> voordat je het implementeert.

---

## Eerder in het gesprek al vastgelegd

- **Nieuw bestand**, niet doorbouwen op het oude.
- **Scènes zijn de geplande tijdseenheid.** Shots staan eronder, krijgen 0 minuten, voegen
  geen tijd toe en verschuiven de starttijd van de volgende scène niet — maar blijven wel
  zichtbare, eigen planregels in de juiste volgorde, gekoppeld aan hun scène, ook na
  verplaatsen, sorteren of opnieuw genereren.
- **Shotnummering scène.shot** (1.1, 1.2, 15.13) wordt aangehouden.
- **Zoom krijgt een eigen kolom** — dat is nieuw ten opzichte van het oude systeem.
- **Foto-kolom** blijft leeg; die vult de opdrachtgever zelf.
- De opdrachtgever wil **niet** meer de Apps Script-editor in.
