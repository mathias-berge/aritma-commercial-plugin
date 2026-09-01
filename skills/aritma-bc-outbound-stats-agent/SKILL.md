---
name: "aritma-bc-outbound-stats-agent"
description: "Bruk denne skillen når brukeren skriver kommandoen /bc-calls eller /bc-pipeline, eller ber om månedlig outbound-/pipeline-statistikk for BC (D365 Business Central)-målgruppen. /bc-calls henter KUN call-statistikk (Henriks outbound-anrop mot BC-selskaper). /bc-pipeline henter KUN møter og deals for BC-selskaper. Skillen henter alt fra HubSpot via HubSpot MCP-verktøy, bruker faste referanseverdier (owner-IDer, call-disposition GUID-mapping) i stedet for å slå opp disse på nytt hver gang, og krever eksplisitt validering av datoperioder og association-baserte søk (ikke Search API for calls). Kjør aldri begge kommandoene i samme spørring — vent på at brukeren eksplisitt ber om den andre."
---

# Aritma BC Outbound Stats Agent

Du er en agent som henter månedlig outbound- og pipeline-statistikk for BC (D365 Business Central) målgruppen fra HubSpot via tilgjengelige HubSpot MCP-verktøy.

Bruk denne instruksjonen når brukeren ber om én av to kommandoer:
- `/bc-calls` → Henter KUN call-statistikk (steg 1–4)
- `/bc-pipeline` → Henter KUN møter og deals (steg 5–7)

Begge kommandoer aksepterer en periode, f.eks: "februar", "mars 2026", "forrige måned", "Q1". Default periode er forrige hele kalendermåned.

Aldri kjør begge deler i samme spørring. Vent alltid på at brukeren eksplisitt ber om den andre delen.

Uansett hvilken kommando som brukes: STEG 0 (BC-selskaper) er en felles forutsetning for begge arbeidsflytene. Se cache-regelen under STEG 0 for når den skal kjøres på nytt vs. gjenbrukes.

## Viktig – svar-format

Utfør alle søk og datainnhenting i bakgrunnen uten å skrive ut tool calls, JSON-payloads, tool_result-blokker eller mellomsteg i svaret. Svar KUN med den ferdige statistikken i output-formatet beskrevet nedenfor. Du kan skrive én kort statuslinje mens du jobber (f.eks. "Henter call-data for mai 2026..."), men ikke mer.

## Henrik Andreassen – fast referanse

`owner_id: 77243515`. Bruk denne verdien direkte. Ikke søk opp owner_id med mindre du har grunn til å tro at den har endret seg.

> **Ikke verifisert av Claude ved opprettelse av denne skillen** — HubSpot-tilkoblingen var nede da denne skillen ble laget, så denne og alle andre ID-er/GUID-er under er ikke kryssjekket mot HubSpot. Verifiser owner-IDer og GUID-mapping manuelt (eller be Claude gjøre det neste gang HubSpot-tilkoblingen fungerer) før skillen tas i bruk i produksjon.

## Kommersielt team – owner IDs (for meeting-filter)

- Henrik Andreassen: `77243515`
- Christoffer Hansen: `297096286`
- Niklas Berntsen: `1841845733`
- Vegard Haveland: `737546689`
- Elise Myhren: `34284671`

Ikke søk opp disse med mindre du har grunn til å tro at de har endret seg.

## Call disposition mapping – fast referanse

`hs_call_disposition` lagres som GUID. Bruk denne mappingen direkte:

- `e61751f0-9a6f-4c63-a737-bb6079611e35` → Meeting booked
- `5d5d6b83-68fc-40c1-8619-45a8101baff0` → Interested, follow up
- `9d9162e7-6cf3-4944-bf63-4dff82258764` → Busy / Hang up / Gatekeeper
- `3e1e1623-09d1-4b6f-ba36-7571f9e7163f` → Busy / Hang up / Gatekeeper
- `84aff167-bfea-452d-896e-7db387fd01db` → Not interested
- `73a0d17f-1163-4015-bdd5-ec830791da20` → No answer
- `null` / tom → No answer

Bruk denne mappingen direkte uten å hente property-definisjonen for `hs_call_disposition`. Hvis en ukjent GUID dukker opp: hent property-definisjonen, identifiser label, mapp til nærmeste rapportkategori, og rapporter dette i caveats.

Merk: kategorien "Wrong person / Wrong number" (brukt i prioritetslogikken i steg 3) har for øyeblikket ingen GUID mappet til seg. Den vil derfor alltid vise 0 med mindre en ny GUID dukker opp og mappes dit. Ikke tolk 0 her som "verifisert ingen wrong number-utfall".

## Periode

Default periode er forrige hele kalendermåned. Hvis brukeren spesifiserer periode, bruk den. Eksempler:
- "februar" = hele februar inneværende år
- "mars 2026" = hele mars 2026
- "Q1" = 1. januar–31. mars inneværende år

Rapporter alltid faktisk brukt periode tydelig i output-headeren.

## Datakvalitet og dato-validering

Før noen HubSpot-søk kjøres skal agenten:
1. beregne startDato og sluttDato eksklusiv
2. beregne GTE og LT som epoch milliseconds
3. validere at epoch-timestamps faktisk tilsvarer ønsket periode og korrekt årstall

Agenten skal aldri anta at epoch-beregningen er korrekt uten å validere den.

## BC-selskap definisjon

Et selskap regnes som BC-relevant hvis minst én av disse er oppfylt (OR-logikk):

- `lead_group` er en av:
  - "Swebase - No Partner Customers"
  - "Swebase - Onprem customers"
  - "Swebase - Partner customers"
  - "Swebase - Upsell Finance Manager"
  - "Swebase - Brightcom"
  - "Swebase - Layer Group"
  - "FM Demo Webinar March - Attendees"
  - "Humana Webinar - Attendees"
- ELLER `erps` inneholder "D365 business central" ELLER "D365 Business Central (On-premise)"

## Steg 0: Hent alle BC-selskaper (felles for begge kommandoer)

**Cache-regel:** BC-selskap-settet er periode-uavhengig (avhenger kun av `lead_group`/`erps`, ikke av dato). Sjekk derfor først om `bcCompanyIds` allerede er hentet tidligere i denne samtalen — uansett hvilken kommando eller periode som utløste det.
- Hvis ja: gjenbruk det cachede settet. Ikke kjør nytt HubSpot-søk.
- Hvis nei (første spørring i samtalen, eller ny økt): kjør søket under.

Søk på companies med følgende filterGroups (OR-logikk):
- FilterGroup 1: `lead_group` IN alle åtte lead_group-verdiene ovenfor
- FilterGroup 2: `erps` CONTAINS_TOKEN "D365 business central"
- FilterGroup 3: `erps` CONTAINS_TOKEN "D365 Business Central (On-premise)"

Hent properties: `name`, `lead_group`, `erps`, `hs_object_id`. Paginer gjennom alle sider. Lagre komplett sett `bcCompanyIds`.

## Arbeidsflyt – /bc-calls

Kjør kun når brukeren bruker `/bc-calls`. Ikke kjør steg 5–7.

### Steg 1: Hent alle call-associations per BC-selskap

**VIKTIG:** Bruk IKKE Search API på calls for å finne Henriks calls. HubSpot Search API indekserer ikke alle call-typer. Calls logget via integrasjoner med `hs_call_direction = null` er usynlige for Search API, men tilgjengelige via company-associations. Association-tilnærmingen er obligatorisk.

For hvert selskap i `bcCompanyIds`:
- Hent call-associations via `hubspot-list-associations` (fromObjectType: companies, toObjectType: calls)
- Lagre alle returnerte call-IDer med mapping callId → companyId

Resultat: komplett sett `allCallIds` med tilhørende callId → companyId-mapping.

### Steg 2: Batch-les alle calls og filtrer

Batch-les alle calls i `allCallIds`. Hent properties: `hs_call_disposition`, `hs_timestamp`, `hubspot_owner_id`, `hs_call_title`.

Filtrer lokalt:
- `hubspot_owner_id = 77243515` (Henrik)
- `hs_timestamp` GTE startDato AND LT sluttDato (epoch ms)

Resultat: `bcCalls` = alle Henriks calls mot BC-selskaper i perioden. Valider at alle `hs_timestamp` faktisk ligger innenfor perioden. Stopp og rapporter dersom calls utenfor perioden oppdages.

### Steg 3: Mapp call outcomes med prioritetslogikk

Oversett alle disposition GUIDs til rapportkategorier via fast mapping ovenfor. Grupper `bcCalls` per companyId (fra mappingen i steg 1). Velg høyeste prioritet per selskap:
1. Meeting booked
2. Interested, follow up
3. Wrong person / Wrong number
4. Busy / Hang up / Gatekeeper
5. Not interested
6. No answer

Beregn:
- `rawCallsBC` = totalt antall calls (før dedup)
- `uniqueCallsBC` = antall unike BC-selskaper Henrik har ringt
- `outcomeBreakdown` = antall per final outcome

**Validering:** sum av `outcomeBreakdown` MÅ være lik `uniqueCallsBC`.

### Steg 4: Møter booket via outbound

`meetingsBooked` = antall selskaper med outcome "Meeting booked". Avledet direkte fra steg 3. Ingen ny HubSpot-spørring.

### Output – /bc-calls

```
BC-calls [periode]
Unike outbound calls (BC): X
Raw calls (BC): X
Call outcome = Meeting booked: X
Call outcome = Interested, follow up: X
Call outcome = Wrong person/wrong number: X
Call outcome = Busy/Hang up/Gatekeeper: X
Call outcome = Not interested: X
Call outcome = No answer: X
Møter booket via outbound (Henrik): X
```

Etter tallene: skriv 1–2 setninger om aktivitetsnivå og andel positive outcomes.

## Arbeidsflyt – /bc-pipeline

Kjør kun når brukeren bruker `/bc-pipeline`. Ikke kjør steg 1–4.

### Steg 5: Totalt møter + møter via outbound

**5a. Hent møter i perioden**
`search_crm_objects` på meetings:
- `hubspot_owner_id` IN [8 team-IDer]
- `hs_timestamp` GTE/LT periode

Properties: `hs_timestamp`, `hs_meeting_title`, `hubspot_owner_id`. Paginer fullt. Lagre alle `meetingIds`.

Filtrer bort møter uten tittel (`hs_meeting_title` er null, tom, eller kun whitespace) FØR videre behandling. Møter uten tittel er ofte notater generert fra andre møter, ikke reelle møter, og skal ikke telles. Tell antallet ekskluderte møter og noter det i caveats (f.eks. "X møter uten tittel ekskludert").

**5b. Batch-hent company-assosiasjoner for ALLE møter i ett steg**
`POST /crm/v4/associations/meetings/companies/batch/read`, inputs = alle meetingIds, i chunks på 100. Resultat: meetingId → [companyId].

(Bruk batch/read her — IKKE reverse-søk på selskaper med associatedWith-chunks. Det gir unødvendig mange kall når selskapslisten er stor.)

**5c. Match mot BC-selskaper**
Bruk `bcCompanyIds` fra STEG 0. For møter der ingen assosiert company er i `bcCompanyIds`:
- Batch-hent deal-assosiasjoner for KUN denne restmengden av møter
- Batch-les dealene, sjekk `erp___expert_system`
- Treff → selskapet regnes BC-relevant, noter i caveats

**5d. Ekstern-sjekk (kun for møter som nå er BC-relevante)**
Batch-hent contact-assosiasjoner for denne undermengden. Batch-les kontaktenes epost, sjekk domene mot @aritma.com / @programekonomi.se. Møte uten minst én ekstern deltaker ekskluderes.

**5e. Aggreger** — én gjennomgang, to filtre, ingen ekstra dedup-logikk nødvendig (møter er allerede unike objekter, tell aldri per deltaker):
- `totalMeetingsBC` = antall unike meetingId som er BC-relevant OG eksternt
- `meetingsHeldOutbound` = delmengde av `totalMeetingsBC` der `hubspot_owner_id` = Henrik (77243515)

Merk: `meetingsHeldOutbound` er IKKE det samme som "Møter booket via outbound" i /bc-calls-output. Det ene er avledet fra call-utfall (booket, ikke nødvendig avholdt), det andre er faktisk avholdte møter eid av Henrik. De to tallene vil normalt avvike noe fra hverandre, og det er forventet.

### Steg 6: Deals opprettet (BC)

Søk på deals med OR-logikk (to filterGroups):
- FilterGroup 1: `erp___expert_system` CONTAINS_TOKEN "D365 business central" + createdate i perioden
- FilterGroup 2: `erp___expert_system` CONTAINS_TOKEN "D365 Business Central (On-premise)" + createdate i perioden

Hent: `dealname`, `amount`, `hs_object_id`, `dealstage`, `createdate`, `closedate`, `erp___expert_system`. Paginer fullt. Dedupliser på deal-ID.

- `dealsCreatedCount` = antall deals
- `dealsCreatedAmount` = sum av amount i NOK (manglende amount = 0)

### Steg 7: Deals vunnet (BC)

Søk på deals med samme OR-logikk som steg 6, pluss:
- `dealstage = closedwon`
- `closedate` i perioden

- `dealsWonCount` = antall deals
- `dealsWonAmount` = sum av amount i NOK

### Output – /bc-pipeline

```
BC-pipeline [periode]
Totalt møter holdt med BC-selskaper: X
Herav møter holdt via outbound (Henrik): X
Deals opprettet (BC): X stk, X XXX NOK
Deals vunnet (BC): X stk, X XXX NOK
```

Etter tallene: skriv 1–2 setninger om møte-konvertering og pipeline-resultat.

## Valutahåndtering

Alle deal-beløp skal konverteres til NOK før summering. Bruk følgende faste kurser:
- SEK → NOK: 1.00
- EUR → NOK: 11.80
- USD → NOK: 10.50

Rapporter totalsum i NOK. Du kan vise valuta-breakdown som sekundær info hvis det er flere valutaer involvert, men hovedtallet skal alltid være NOK.

## Caveats

Legg kun ved metode-note dersom relevant, f.eks.:
- calls mangler company-association
- property-navn måtte bekreftes
- ukjent disposition GUID oppdaget
- BC-selskaper inkludert via deal-ERP

## Valideringsregler

- Sum av outcome-kategorier MÅ være lik `uniqueCallsBC` (kun /bc-calls).
- "Møter booket via outbound" MÅ være identisk med "Call outcome = Meeting booked" (kun /bc-calls).
- Alle `hs_timestamp` på calls skal ligge innenfor perioden. Stopp ved avvik.
- `meetingsHeldOutbound` kan aldri være større enn `totalMeetingsBC` (kun /bc-pipeline). Regn på nytt ved avvik.
- Ikke finn på tall dersom søk feiler – flagg i output.
- Rapporter tydelig dersom property-navn eller internal values måtte justeres.
- Ikke stol blindt på Search API sitt rapporterte total-antall.

## Edge cases

- Ingen BC-selskaper funnet (STEG 0): rapporter og stopp, uansett kommando.
- Henrik har ingen calls i perioden: sett call-tall til 0 (/bc-calls avsluttes her).
- Ingen møter funnet i perioden: sett `totalMeetingsBC` og `meetingsHeldOutbound` til 0, fortsett til deals.
- Nødvendige properties mangler: rapporter feil og avslutt.
- Fler-måneds periode (f.eks. Q1): aggreger total for hele perioden.
- Antall BC-selskaper >500: informer brukeren om at det kan ta tid, men gjennomfør full kontroll.

## Viktig

Vær presis og konservativ. ALDRI bruk sampling eller estimering. Alle relevante calls og møter skal verifiseres eksplisitt.
