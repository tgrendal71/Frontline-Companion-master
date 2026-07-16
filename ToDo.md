# ToDo — Frontline Companion

> Basert på full gjennomgang av `data/data.js` OG `data/app.js` mot de offisielle
> Axis & Allies Global 1940 Second Edition-reglene
> (axisallies.com/global-1940-second-edition-rules/) og kryssjekk av samtlige
> territorieverdier mot TripleA sin offisielle G40 2nd Edition-datafil.
>
> Prioritet: 🔴 Kritisk · 🟠 Høy · 🟡 Middels · 🟢 Lav
>
> **VIKTIG:** data.js sier at `src/territories.csv` er kanonisk kilde ved kjøretid.
> Alle territorie-rettelser må gjøres BÅDE i data.js (fallback) OG i territories.csv.

---

## DEL 1 — Funksjonalitet og regelnøyaktighet

### ✅ Løst (data.js) / ⚠️ delvis — Seiersbyer (Victory Cities)

Spillet har **19 seiersbyer** (11 på Europa-kartet, 8 på Stillehavs-kartet).

- [x] **Skrivefeil-bug fikset:** `vologda` hadde `isCapital:true` — flyttet til
      **`volgograd`** (Stalingrad er seiersby, Vologda er det ikke).
- [x] Satt `isCapital:true` på `egypt` (Cairo), `kiangsu` (Shanghai),
      `kwangtung` (Hong Kong), `philippines` (Manila) og `hawaii` (Honolulu) i data.js.
      `VICTORY_CITIES.length` er nå 19 (11 Europa + 8 Stillehavet) i fallback-dataene.
- [ ] **Nyoppdaget under samme gjennomgang:** `territories.csv` har allerede en egen
      **`VictoryCity`-kolonne** adskilt fra `IsCapital` (nøyaktig det forrige punktet
      «vurder eget felt» etterlyste!) — og den er *riktig utfylt* der (Egypt har
      `VictoryCity=Yes` men `IsCapital=No`, som er korrekt). Problemet: CSV-parseren
      i app.js (`_parseTerritoriesCSV`, rundt linje 4633) leser kun `IsCapital`-kolonnen
      og ignorerer `VictoryCity` helt. Fiks: parse `VictoryCity`-kolonnen til et eget
      `isVictoryCity`-felt, og bygg `VICTORY_CITIES` fra det i stedet for `isCapital`.
- [ ] **Alvorligere nyoppdaget bug:** CSV-en bruker lange beskrivende id-er som
      `pacific-philippine-islands-manilla` / `europe-egypt` / `europe-volgograd` som
      `t.id`, mens data.js og *hele* resten av koden (OBJECTIVE_RULES, STARTING_FACILITIES,
      `isBurmaRoadOpen()`, m.fl.) er hardkodet mot korte id-er (`philippines`, `egypt`,
      `india`, `burma`...). Når CSV-en faktisk lastes (krever HTTP-server, f.eks.
      `saves-api.py` — `fetch()` av lokale filer feiler stille under `file://`, så i
      praksis kjører de fleste brukere alltid på data.js-fallbacken uten å vite det),
      vil `getController('egypt')` og lignende kortid-oppslag returnere `'neutral'`
      i stedet for faktisk kontrollør, og en rekke mål-/regelsjekker slutter å virke
      korrekt. Dette er trolig den reelle årsaken til at data.js og CSV-en har driftet
      fra hverandre over tid (se også Del 2 sitt punkt om «Én kilde til sannhet»).
      Krever en bevisst beslutning: normaliser CSV-id-ene til korte id-er ved parsing,
      eller gjør kort-id-konvensjonen kanonisk og oppdater CSV-en.

### 🔴 Kritisk — Seiersbetingelser (app.js linje 10–11)

- [ ] `AXIS_WIN_VC = 13` / `ALLIES_WIN_VC = 8` er en husregel, ikke offisiell.
      Offisielt: **Aksen vinner med hvilke som helst 8 seiersbyer på Europa-kartet
      ELLER 6 på Stillehavs-kartet**, holdt en hel runde, mens de kontrollerer en
      Akse-hovedstad. **De Allierte vinner med Berlin + Roma + Tokyo** en hel runde,
      mens de kontrollerer en Alliert hovedstad. Implementer dette i
      `getAxisVC()`-logikken, eller merk 13-regelen tydelig som husregel i UI.
- [x] ~~Legg til `map: 'europe' | 'pacific'` per territorium~~ — bekreftet at
      `territories.csv` allerede har en `Map`-kolonne (`Europe`/`Pacific`) korrekt
      utfylt (`continent` alene stemmer, Washington=Europe/San Francisco=Pacific
      bekreftet i CSV-en). Samme parser-hull som over: `_parseTerritoriesCSV` leser
      ikke `Map`-kolonnen. For seiersby-medaljongene (se `forbedre_todo.md` punkt 4)
      er dette midlertidig løst med en liten frittstående oppslagstabell
      (`VC_MAP_SIDE` i app.js, kun de 19 seiersbyene) — resten av territoriene har
      fortsatt ikke et parset `map`-felt.

### 🔴 Kritisk — Feil IPC-verdier (verifisert mot offisielle data)

Rett i både data.js og territories.csv. Etter disse stemmer alle nasjoners
territoriesum med offisiell startinntekt (som allerede er korrekt i NATIONS):

- [ ] `iwo_jima`: 0 → **1** og `okinawa`: 0 → **1** (Japan summerer i dag 24, skal være 26)
- [ ] `new_zealand`: 0 → **2** (ANZAC summerer 8, skal være 10)
- [ ] `fr_indochina`: 1 → **2** (Frankrike summerer 18, skal være 19)
- [ ] `alberta`: 0 → **1** (UK Europa summerer 27, skal være 28)
- [ ] `turkmenistan`: 1 → **0** (Sovjet summerer 38, skal være 37)
- [ ] `switzerland`: 2 → **0**

NB: `calcIncome()` bruker `startIncome + delta(t.ipc)`, så feil territorieverdi gir
direkte feil inntekt i det øyeblikket et av disse territoriene bytter eier.

### ✅ Løst — Krigsstatus-modellen (app.js linje 39, 3089–3137)

Erstattet med per-nasjon `atWarWith`-array (bilaterale relasjoner), `isAtWarWith` /
`declareWar` / `declarePeace`-hjelpere, korrekte startverdier (Tyskland/Italia vs.
UK-E/UK-P/ANZAC/Frankrike, Japan vs. Kina, USA/Sovjet nøytrale), fjernet
`round > 3`-tvangslåsen fra `getEffectiveAtWar`, og migrering + persistering av
krigsstatus på tvers av lagrede spill (ikke lenger nullstilt ved load/import).

- [x] Krig er per motpart, ikke globalt — `atWarWith: ['uk_europe', ...]` per nasjon.
- [x] `round > 3` tvinger ikke lenger alle i krig.
- [x] Krigsstatus persisteres ved load/import (med migrering fra gammel `atWar`-boolean).
- [x] Korrekte startverdier implementert.

### ✅ Løst — Nasjonale mål viste feil bonuser (fant appen selv i bruk, oppfølger til krigsstatus-fiksen)

**Oppdaget via brukerrapport:** Tyskland fikk ikke vist «Ikke i krig med Sovjet
+5 IPC»-målet som standard (kun synlig via «Vis alle»), mens alle Sovjet-relaterte
krigsmål (Leningrad, Stalingrad, Moskva, Kaukasus) *var* synlige — stikk motsatt av
hva som er sant runde 1 (Tyskland er i fred med Sovjet, i krig med UK/Frankrike).

**Rotårsak:** Da krigsstatus-modellen over ble bygget om til per-motpart
(`atWarWith`), ble selve mål-*reglene* (`OBJECTIVE_RULES`) riktig oppdatert til å
sjekke spesifikke relasjoner (`isAtWarWith('germany','soviet')`) — men
*synlighets*-filtreringen tre andre steder i koden (`buildObjectivesHTML`,
`evalObjectivesForNation`, `_syncObjectivesAfterWarChange`) brukte fortsatt en
generell «er nasjonen i krig med noen som helst»-sjekk. Siden Tyskland er i krig med
UK/Frankrike fra runde 1, var denne generelle sjekken alltid sann — og skjulte/viste
*alle* krigs-/fredsmål feil, uansett hvilken motstander de faktisk gjaldt. Samme feil
rammet Japan (jap_us_trade, jap_perimeter m.fl. — Japan er alltid i krig med Kina
fra start, så «i krig»-sjekken var alltid sann for disse også).

- [x] Ny `OBJECTIVE_ELIGIBILITY`-oppslagstabell i app.js, utledet direkte fra de
      allerede riktige relasjonene i `OBJECTIVE_RULES` — dekker alle mål på tvers av
      Tyskland, Sovjet, Japan, USA, Kina, UK-E, UK-P, Italia og ANZAC
- [x] Ny `isObjectiveEligible(tid, o)`-hjelpefunksjon brukt alle tre steder
      synlighet/avhaking styres, med fallback til den generelle sjekken for mål uten
      navngitt motstander (f.eks. `ita_persia`, `fra_liberation`)

**Oppfølger — verifisert mot offisiell kilde etter brukerspørsmål:** brukeren spurte
om Tysklands Skandinavia-bonus («Kontrollerer Danmark OG Norge») virkelig skal være
tilgjengelig fra runde 1. Kryssjekket mot **axisallies.com/global-1940-second-edition-rules/**
(samme kilde som resten av dette dokumentet bygger på) og bekreftet: ja, korrekt —
Tyskland starter i krig med både UK og Frankrike OG kontrollerer allerede
Danmark+Norge, så bonusen er reelt tilgjengelig fra runde 1. MEN samme oppslag
avdekket at **fire Tyskland-mål manglet eller hadde ufullstendig krigssjekk**:
`ger_scandinavia` sjekket kun krig med UK (manglet Frankrike), og
`ger_iraq`/`ger_persia`/`ger_nw_persia` hadde ingen krigssjekk i det hele tatt i
`OBJECTIVE_RULES` — alle fire skal offisielt kreve krig med **både** UK og Frankrike
samtidig. Rettet i både `OBJECTIVE_RULES` (fullføringslogikk) og
`OBJECTIVE_ELIGIBILITY` (synlighet). Merkes ikke som synlig feil i standardoppsettet
(Tyskland er i krig med begge fra runde 1 uansett), men ville gitt feil resultat hvis
fred noensinne reforhandles med kun én av de to.

- [x] `ger_scandinavia`, `ger_iraq`, `ger_persia`, `ger_nw_persia` krever nå
      `isAtWarWith(tid,'uk_europe') && isAtWarWith(tid,'france')` i både
      fullførings- og synlighetssjekk

**Ikke rettet — trenger egen verifisering:** samme gjennomgang fant at Italias
tilsvarende mål (`ita_mediterranean_land`, `ita_sea_control`, `ita_north_africa`)
sjekker spesifikt `isAtWarWith('italy','uk_europe') || isAtWarWith('italy','usa')`,
mens både et web-søk og appens egen rules.html (som ikke viser noen «I krig med
X»-overskrift for Italias mål, i motsetning til Tysklands og Japans) antyder disse
kanskje bare skal kreve generell krigsstatus. Ikke endret nå — kilden var kun ett
AI-oppsummert nettsøk (lavere pålitelighet enn den direkte sitat-verifiserte
Tyskland-fiksen over), og dagens kode kan være bevisst slik. Bør dobbeltsjekkes mot
selve regelheftet før dette eventuelt endres.

### ✅ Løst — STARTING_FACILITIES var keyet med feil id-format (fant appen selv i bruk)

**Oppdaget via brukerrapport:** mål-nedtrekkslisten i strategisk bombing var alltid
tom («— velg territorium —» med ingen alternativer), uansett nasjon eller runde.

**Rotårsak:** `STARTING_FACILITIES` (data.js) var keyet med de lange
CSV-stil-id-ene (`'europe-germany-berlin'`, `'pacific-japan-tokyo'`, …) — nøyaktig
samme id-format-forvirring som ble flagget under seiersby-arbeidet lenger opp, bare
denne gangen INNI data.js selv, ikke bare i territories.csv-parseren. Siden
`TERRITORIES` (data.js-fallbacken, som kjører i praksis — se CSV/id-notatet over)
bruker korte id-er (`'germany'`, `'japan'`), matchet `state.facilities`-nøklene
aldri noen faktiske territorier. `hasFacility()` returnerte `false` for **alt**,
som stille brøt: mål-listen for strategisk bombing, reparasjons-UI-en
(`getDamagedFacilitiesForNation`), og kjøpssperren mot å bygge dobbel fasilitet på
samme territorium.

- [x] Alle 34 nøklene i `STARTING_FACILITIES` byttet fra lange CSV-id-er til de
      korte `TERRITORIES[].id`-verdiene (kryssjekket mot faktiske id-er i data.js)

### 🔴 Kritisk — Startfasiliteter (STARTING_FACILITIES i data.js)

- [ ] **USAs tre industrikomplekser starter som MINOR** (Eastern/Central/Western US)
      og oppgraderes gratis til major idet USA går i krig (offisiell 2nd ed-regel).
      Endre til `ic:'minor'` + auto-oppgradering ved krigsstatus.
- [ ] **Southern France mangler**: skal ha `ic:'minor'` + `navalBase:true`.

### 🟠 Høy — Bekreftede logikkfeil i nasjonale mål (app.js OBJECTIVE_RULES)

- [ ] `anz_perimeter` (linje 228): bruker `!isAxis(ctrl(t))` — Dutch New Guinea
      starter **Dutch-kontrollert** og teller dermed som oppfylt fra spillstart.
      Offisielt må de Allierte (ikke nederlenderne) faktisk kontrollere alle fire.
      Endre til `isAllied(ctrl(t))`.
- [ ] `getSovAxisTerritories` (linje 168): teller kun `startController` germany/italy.
      Offisielt teller også **opprinnelig pro-Akse-nøytrale** territorier
      (Bulgaria, Finland, Irak). Legg til `t.neutralType === 'pro_axis'`.
- [ ] `anz_malaya` / `chi_burma_road`: bruker `!isAxis` som tilnærming — offisielt
      kreves Alliert kontroll. Fungerer i praksis (territoriene starter Allierte),
      men bytt gjerne til `isAllied()` for konsistens.
- [ ] `ger_scandinavia` (linje 183): sjekker `!isAllied(ctrl('sweden'))`, men
      offisielt diskvalifiserer også at Sverige er blitt **pro-Alliert** (skjer hvis
      Aksen angriper en streng nøytral). Appen mangler tilstand for nøytrales
      alignment — legg til et globalt flagg «Aksen har angrepet streng nøytral» /
      «Allierte har angrepet streng nøytral» som vipper alle gjenværende nøytrale.

### 🟠 Høy — Startkontrollør-feil (data.js)

- [ ] `new_hebrides`: `anzac` → **`france`**. Viktig fordi `anz_malaya`-målet krever
      at ANZAC kontrollerer *alle sine opprinnelige* territorier — feil datagrunnlag
      gir feil målevaluering den dagen New Hebrides bytter hender.

### ✅ Løst — Kina-kjøpsrestriksjoner (app.js)

Handlekurven filtreres nå for Kina: kun enheter med `chinaAllowed:true` i UNITS
(infanteri alltid, artilleri kun når `isBurmaRoadOpen()` er sann) vises som kjøpbare
grupper, og et hint-banner forklarer restriksjonen (`pc.china_restriction`, begge
språk). `isBurmaRoadOpen()` er brutt ut som delt hjelpefunksjon og brukes både her og
i `chi_burma_road`-målregelen. Konvoi-tap-feltet er skjult for Kina i nasjonskortet.

- [x] Enhetslisten filtreres datadrevet (`chinaAllowed` / `chinaRequiresBurmaRoad` i data.js).
- [x] Konvoifeltet er skjult for Kina.

### 🟠 Høy — Manglende territorier (data.js + CSV)

- [ ] Legg til **Crete** (0 IPC, pro-Alliert nøytral) og **Rio de Oro**
      (0 IPC, streng nøytral).

### 🟡 Middels — UK: én tur, to økonomier

- [ ] Offisielt tar UK **én samlet tur** (felles bevegelse/kamp, men separate kjøp
      og inntekt per økonomi). Appen kjører uk_europe og uk_pacific som to fulle
      turer med hver sin fasesyklus. Fungerer for IPC-sporing, men vurder å slå
      sammen til én UK-tur med to økonomipaneler — eller dokumenter avviket i appen.
      (Økonomitilhørighet er ellers korrekt: West India → Stillehavet,
      Western Canada → Europa. ✓)

### 🟡 Middels — Politiske regler som hint

- [ ] USA kan erklære krig ved starten av **inntektsfasen på sin tredje tur** —
      vis som hint/knapp når runde 3 nås.
- [ ] Sovjet kan ikke erklære krig mot europeiske Aksemakter før tur 4 (med mindre
      angrepet først eller London faller).
- [ ] Japans handelsmål (10 IPC) tapes permanent ved angrep på Fransk Indokina
      eller uprovosert krigserklæring mot UK/ANZAC — vurder et «tapt for godt»-flagg
      i stedet for manuell avhuking.

### 🟢 Lav — Datasjekk og kosmetikk

- [ ] Teknologi-ID-ene `comb_bombardment` (= Increased Factory Production) og
      `mech_artillery` (= Improved Mechanized Infantry) er misvisende — navnene er
      riktige; rydd ID-ene ved anledning (krever migrering av lagrede spill).
- [ ] Verifiser `neutralArmy`-antall mot oppsettkartet (stikkprøvene så riktige ut).
- [ ] Kamikaze: Japan har 6 tokens (soner rundt Japan, Okinawa, Iwo Jima, Formosa,
      Marianas, Filippinene) — legg til teller (se Del 2).

---

## DEL 2 — Kodekvalitet, arkitektur, mobil/nettbrett-UX og nye funksjoner

### 🔴 Kritisk

- [ ] **Persister `purchaseCart` og `repairTokens`** (app.js linje 16–18, i dag
      session-only). En reload midt i kjøpsfasen skal ikke tømme handlekurven.
      localStorage-autolagring av `state` finnes allerede og fungerer (saveState
      etter hver mutasjon ✓) — utvid samme mekanisme til kurv/reparasjoner.
- [ ] **Én kilde til sannhet for territoriedata:** data.js-fallbacken og
      territories.csv kan drifte fra hverandre (alle Del 1-rettelser må gjøres to
      steder). Generer fallbacken fra CSV-en med et byggeskript, eller dropp
      fallbacken og feil tydelig hvis CSV mangler.
- [ ] **Valideringssjekk ved oppstart:** verifiser at (a) hver nasjons territoriesum
      = startIncome, (b) alle territorier har id/ipc/startController/map,
      (c) VICTORY_CITIES har nøyaktig 19 innslag (11 Europa + 8 Stillehavet),
      (d) hvert mål har gyldige felter. Samtlige datafeil i Del 1 ville blitt
      fanget automatisk av en slik sjekk.

### 🟠 Høy

- [ ] **Angre-funksjon (undo):** stack av state-snapshots (f.eks. siste 20) med én
      angre-knapp. Viktigste enkeltforbedring ved et fysisk spillbord — feilklikk
      på eierbytte eller «Samle inntekt» er i dag irreversible.
- [ ] **Splitt app.js (4569 linjer) i moduler:** state.js, income.js, objectives.js,
      territories.js, phases.js, battle.js, bombing.js, ui/render.js — flere
      `<script>`-tagger i riktig rekkefølge eller ES-moduler. Splitt data.js
      tilsvarende (nations.js, territories.js, objectives.js, units.js).
- [ ] **Feilhåndtering mot lagrings-API-et:** tydelig banner/toast når `saves-api.py`
      er nede (fetch-kallene rundt linje 457–551), med beskjed om at lokal
      autolagring fortsatt er aktiv.
- [x] ✅ **Stor, alltid synlig tur/fase-indikator** med touch-mål ≥ 44×44 px:
      aktiv nasjon + aktiv fase + «neste»-knapp øverst.
      **Løst (i to iterasjoner):** først som «Turn Cockpit» på Oversikt-fanen
      (gjenopplivet død kode `renderPhaseTracker()` → `renderCockpit()`), men
      den dupliserte nasjonskortets faseliste uten å huse selve handlingene.
      Endelig løsning: **klistret turlinje** øverst på Nasjoner-fanen (som nå er
      hovedfanen) — aktiv nasjon, faseframgang, «Neste fase ↓»-hopp til første
      uavhukede faseblokk, og «Fullfør tur». Alltid synlig mens man scroller i
      spillets faktiske handlingsflate, som var poenget med ønsket.

### 🟡 Middels

- [x] ✅ **Turlogg/historikk-visning:** grunnlaget finnes allerede
      (`territoryChanges`, `bombingEvents`, `purchaseLogs`, `history`) — egen
      fane som viser per runde/nasjon: inntekt (basis + mål − konvoi), kjøp,
      territorieendringer og bombing, med mulighet for å bla bakover.
      **Løst:** dette fantes faktisk allerede TO ganger, uavhengig bygget
      (Oversikt sin `renderChronicle()`/`buildLogRoundBody()` og Historikk sin
      egen `renderHistory()`) — konsolidert til én kilde
      i Historikk-fanen, med en ny live «runde pågår»-seksjon Historikk manglet.
- [ ] **Kamikaze-teller for Japan** (6 tokens, tell ned).
- [ ] **Konvoikalkulator:** slagskip/krysser/jager = 1 terning, ubåt/fly = 2 terninger,
      treff på ≤3, tak per tilstøtende territoriums IPC-verdi — skriver resultatet
      rett inn i `convoyLoss`.
- [ ] **Mobil-/nettbrettgjennomgang av style.css:** test stående/liggende på ekte
      enheter; sjekk fontstørrelser, at nasjonskort stacker, og at territorielisten
      (200+ rader) har søk/filter som fungerer godt med berøring.
- [ ] **Bekreftelsesdialog** på destruktive handlinger (nullstill runde, hovedstads-
      overføring, slett parti). Kasseoverføring ved hovedstadsfall skjer i dag
      umiddelbart ved eierbytte (onOwnerChange) — et feilklikk flytter hele kassen.
- [ ] **Etterlev egne prosjektregler:** opprett/oppdater `CHANGELOG.md` og
      `PROMPTS.md` (kreves av copilot-instructions.md).

### 🟢 Lav

- [ ] **PWA / «Legg til på hjemskjerm»** med service worker for helt offline bruk.
- [ ] **Wake lock** (`navigator.wakeLock`) så nettbrettet ikke slukker midt i partiet.
- [ ] Flere navngitte partier lokalt (ikke bare via serveren) — JSON-eksport/-import
      finnes allerede ✓ og kan gjenbrukes.
- [ ] Rydd død kode og dupliserte hjelpefunksjoner under modulsplittingen; håndhev
      `'use strict'` og 2-mellomroms innrykk.
      **Konkret funn (Turn Cockpit-gjennomgangen):** `renderPhaseTracker()` og
      `buildNationPhaseTrackerHTML()` var 100 % ferdigbygd, men aldri koblet inn
      i index.html (`if (!wrap) return;` gjorde hvert kall til et stille no-op)
      — gjenopplivet, ikke slettet. `renderSidePanels()` (mål `#axisNations`/
      `#alliesNations`, heller ikke i index.html) var en tidligere, nå overflødig
      iterasjon av scoreboardet — slettet sammen med sine foreldreløse
      i18n-nøkler (`ov.axis_label`, `ov.allies_label`, `ov.neutral_label`,
      `ov.total_label`, `ov.vc_count_axis`, `ov.vc_count_allies`).

---

## Verifisert OK — ingen endring nødvendig ✓

**Data (data.js):**
- Startinntekt/kasse for alle 10 makter (30/37/26/52/12/28/17/10/10/19)
- Alle enhetskostnader, kampverdier, bevegelse og Improved Shipyards-priser
- Teknologilistene (begge breakthrough-charts, riktige terningverdier)
- Faselisten: R&D valgfri, Kina ekskludert fra R&D, «kun ved krig» på fase 2–3,
  raketter under kampfasen, konvoi som delsteg under inntekt
- Kinas 12 startterritorier vs Japans 6 okkuperte kinesiske territorier
- isMainCapital-flaggene (San Francisco er ikke hovedstad; Kina har ingen)

**Logikk (app.js):**
- `collectIncome`: riktig formel (inntekt + bonus + warBonds − konvoi ± justering),
  nullstilling per runde, engangsmål markeres som hentet
- Hovedstadsfall: kassen overføres kun når opprinnelig eier mister hovedstaden;
  `capturedTreasury` holdes utenfor inneværende kjøpsfase; inntektssperre når
  hovedstaden er okkupert; UK-økonomiene håndteres separat
- localStorage-autolagring etter hver mutasjon + JSON-eksport/-import
- Fasesporing per nasjon med automatisk fase 6 via «Samle inntekt»
- Målregler for Tyskland (territoriemål), Japan, USA, Kina, UK-E (originale
  territorier), UK-P og Italia er korrekt implementert; mål som krever
  brikkeinformasjon (Egypt-landenhet, USA-enhet i Frankrike, sjøsoner, Lend-Lease)
  er bevisst manuelle avkrysninger — riktig designvalg
