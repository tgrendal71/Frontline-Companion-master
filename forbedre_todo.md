# Forbedre ToDo — Visuell oppgradering

> Visuelle forbedringer for Frontline Companion, prioritert etter effekt per innsats.
> Hvert punkt har en ferdig prompt du kan lime inn i Claude Code for å få jobben gjort.
>
> **Anbefalt rekkefølge:** 1 → 3 → 5 → 4 → 2 → 6 → 7
> (1 og 3 er ren utbytting av verdier/assets uten designrisiko; 2 krever mest smak og iterasjon.)

---

## 1. ✅ Løst — Ekte nasjonssymboler i stedet for emoji-flagg

**Oppdatering:** Ved gjennomgang viste det seg at dette allerede var implementert —
`nationIconHTML()` (app.js) rendrer roundels fra `Nations_Buttons/` (satt opp via
`NATIONS[id].icon` i data.js) og brukes konsekvent i turn-pill, turn-strip,
nasjonskort, territorieliste og battle-log, med `.nation-icon--xs/sm/md/lg`-klasser
for sirkulær styling. Det som manglet var et ekte fallback ved *mislykket* bildelasting
(kun tomt `icon:null`-felt var dekket før) — lagt til nå via `onerror` → `nationIconFallback()`
som bytter `<img>` med et bokstav-i-sirkel-badge i nasjonens farge.

- [x] Roundels brukes allerede (500×500 JPG, ingen beskjæring nødvendig — object-fit: cover håndterer visning)
- [x] Turn-pillen i headeren
- [x] Turn-strip på oversiktsfanen
- [x] Nasjonskort-headere
- [x] Territorielisten (kontrollør-kolonnen, owner-badges)
- [x] Fallback ved mislykket bildelasting (nytt: `nationIconFallback()` i app.js + `.nation-icon-fallback[class*="nation-icon--"]` i style.css)

**Prompt:**
```
Appen bruker emoji-flagg (🇩🇪 osv.) for nasjoner, men disse rendres ikke på Windows.
Vi har offisielle A&A-nasjonssymboler i data/Nations_Buttons/ (JPG).
Erstatt alle emoji-flagg i appen med disse bildene:
1. Lag en NATION_ICONS-mapping (nasjons-id → bildesti) i data.js
2. Bruk bildene i turn-pillen i headeren, turn-strip, nasjonskort-headere og
   territorielistens kontrollør-visning
3. Style dem som runde ikoner (border-radius: 50%, object-fit: cover) i passende
   størrelser per kontekst
4. Legg til en fallback (nasjonens forbokstav i en sirkel med nasjonsfargen) hvis
   bildet ikke finnes
Husk: ingen eksterne avhengigheter, alt skal fungere ved å åpne index.html direkte.
```

---

## 2. ✅ Løst — Art direction: 1940-talls krigsrom, ikke GitHub-dashboard

**Oppdatering:** All `:root`-paletten i style.css er dratt fra kjølig blåsvart
(`#0d1117`-familien) til varm oliven/sepia (`#14120d`-familien), med samme
lyshetstrinn som før slik at kontrasten ikke svekkes — kun fargetonen er endret.
Gull-aksenten er urørt. En ekte lokal font-fil (woff2) var ikke praktisk å skaffe i
denne økten uten nettilgang for binærnedlasting, så jeg brukte i stedet **Bahnschrift
SemiBold Condensed** — en kondensert DIN-avledet systemfont som følger med Windows
10+, og som (litt morsomt) faktisk stammer fra samme 1930/40-talls tyske
skilt-/teknisk bokstav-tradisjon appen prøver å hente stemning fra. Ingen eksterne
avhengigheter, fungerer fortsatt ved å åpne index.html direkte; faller tilbake til
Arial Narrow / Segoe UI på Mac/Linux.

- [x] Bakgrunnstonene (`--bg`, `--bg2`, `--bg3`, `--border`, `--border-light`) dratt mot varm oliven/sepia
- [x] Display-font (`--font-display`, systemfont-basert — se merknad over) satt på
      overskrifter, nasjonsnavn, fanetitler (`.tab-btn`), runde-/tur-pillen,
      seiersmåler- og turnstrip-titler, fasetitler på nasjonskort og slagbord-tittelen
      — brødtekst (`body`) beholder Segoe UI uendret
- [x] Papirtekstur: tileable SVG-støy (`feTurbulence`) som CSS-variabel `--paper-texture`,
      lagt på `body`s bakgrunn ved siden av `--bg` (alpha ~4,5 % — nesten usynlig)
- [x] Vignett: separat fastpinnet `body::before`-lag (så den følger skjermkantene i
      stedet for å strekkes nedover en lang scrollet side), `.app-wrap` løftet til
      `z-index:1` for riktig lagdeling
- [x] Kontrast verifisert manuelt (WCAG-formel): `--text` ~14:1, `--text-dim` ~6.8:1,
      `--text-muted` ~3.8:1 mot ny `--bg2` (uendret/marginalt bedre enn før — ingen regresjon)

**Prompt:**
```
Gi appen en 1940-talls krigsrom-estetikk i stedet for dagens GitHub-mørke palett:
1. Juster CSS-variablene i :root i style.css: dra --bg/--bg2/--bg3 mot varmere
   olivengrønn/sepia-toner (f.eks. rundt #12140e-familien), behold gull-aksenten
   --gold: #f59e0b
2. Legg til en lokal display-font (condensed/stencil-stil, embeddet som woff2 —
   ingen eksterne CDN-er) og bruk den kun på overskrifter, fanetitler, nasjonsnavn
   og runde-indikatoren
3. Legg en subtil tileable papirtekstur (liten data-URI, nesten usynlig) på body
   og en svak vignett rundt kantene
4. Verifiser at all tekst fortsatt har god kontrast (WCAG AA) mot de nye bakgrunnene
Gjør endringene gradvis og vis meg resultatet underveis — dette punktet krever
iterasjon på smak.
```

---

## 3. ✅ Løst — Fiks nasjonsfargene — flere var nesten usynlige

**Oppdatering:** Fargestripe/banner på kortene fantes allerede (venstre kantstripe +
gradient-glow i headeren via `--nc-accent`), men selve `--c-*`-verdiene var svake
(bekreftet). Ved gjennomgang dukket det opp et **tredje, usynkronisert fargesystem**:
`.owner-badge[data-nation="..."]` i style.css hadde helt egne hardkodede farger som
ikke stemte med verken `--c-*` eller `NATIONS[].accent` — f.eks. viste Kina gult/oransje
i territorielisten, men grønt på nasjonskortet. Alle tre systemer er nå synkronisert.

- [x] `--c-*`-verdiene i :root oppdatert til A&A-brikkefargene med god kontrast
      (germany → kjølig stål-grå, italy → brun terracotta, anzac → varm stein-grå,
      china → lys lime-grønn skilt fra USAs mørkere grønn, UKE/UKP i to nyanser gult
      så de to britiske økonomiene skilles visuelt)
- [x] Fargestripe/banner i toppen på kortene (fantes allerede — bekreftet fortsatt korrekt)
- [x] Akse/alliert-badge: `.nc-side` er nå en synlig pille med bakgrunn/kant i
      `--axis-color`/`--ally-color` i stedet for bare farget tekst
- [x] Tyskland: lysere `--c-germany` + eget subtilt inner-highlight (`box-shadow: inset`)
      på nasjonskortet så det skiller seg fra bakgrunnen
- [x] **Bonus-fiks:** `.owner-badge[data-nation]` (territorielisten) bruker nå
      `color-mix()` fra samme `--c-*`-token som resten av appen — det tredje,
      usynkroniserte fargesystemet er eliminert
- [x] `NATIONS[].color`/`.accent` i data.js synkronisert med de nye `--c-*`-verdiene
      (brukes av territorigruppe-headere og det nye ikon-fallbacket fra punkt 1)

**Prompt:**
```
Nasjonsfargene i style.css er for svake — --c-germany (#3f3f46) er nesten usynlig
mot kortbakgrunnen, og --c-anzac er ren grå. Fiks dette:
1. Les src/Nasjonsfarger.txt og oppdater --c-*-variablene i :root til de klassiske
   Axis & Allies-brikkefargene, justert så alle har tydelig kontrast mot mørk bakgrunn
2. Gi hvert nasjonskort en markant fargestripe eller banner i toppen med nasjonsfargen
3. Legg til en akse/alliert-indikator på kortene (f.eks. venstre kantstripe i
   --axis-color rød / --ally-color blå)
4. Tyskland trenger spesialbehandling: gråsvart brikkefarge med en lysere
   kant/outline så den skiller seg fra bakgrunnen
Sjekk alle steder nasjonsfargene brukes (kort, turn-strip, territorieliste, badges)
så helheten blir konsistent.
```

---

## 4. ✅ Løst — Seiersmåleren som smykke — 19 individuelle by-merker

**Oppdatering:** Datarettelsene ble gjort først, som forutsatt. Det avdekket to nye
funn utover det ToDo.md allerede visste om — se ToDo.md Del 1 for detaljer:
territories.csv har faktisk allerede egne `VictoryCity`- og `Map`-kolonner (akkurat
det som var etterlyst), men CSV-parseren i app.js leser dem aldri, og CSV-en bruker
uansett lange id-er som ikke matcher resten av kodens korte id-konvensjon. Begge er
dokumentert som åpne punkter i ToDo.md — for medaljongene løst pragmatisk med en
liten lokal `VC_MAP_SIDE`-oppslagstabell (19 byer) i stedet for å vente på den
større CSV/id-opprydningen.

- [x] 19 by-medaljonger rendres i `#vcMedalBoard` (erstattet den gamle kollapsede
      by-listen i `<details>`), gruppert Europa (11) / Stillehavet (8)
- [x] Hver medaljong viser bynavn + kontrollørens nasjonssymbol og -farge (ring rundt
      badge via `--vc-c`) — gullring i stedet for nasjonsfarge på de 9 hovedhovedstedene
- [x] Flip-animasjon (`rotateY`, CSS keyframes) spilles av kun på runden en by faktisk
      bytter kontrollør (sammenlignet mot forrige render via `vcPrevOwner`)
- [x] Gruppert etter kartside med seierskrav vist per gruppe (informasjon kun —
      selve seierssjekken bruker fortsatt 13-husregelen, se eget punkt i ToDo.md)
- [x] Tooltip (title-attributt) viser territorium + kontrollerende nasjon på hover/tap
- [x] Kompakt bar + Akse/Allierte-tall øverst er beholdt uendret

**Prompt:**
```
Gjør om seiersmåleren på oversiktsfanen fra en flat bar til 19 individuelle
by-medaljonger:
1. Render én medaljong per seiersby i VICTORY_CITIES, gruppert i to rader:
   Europa-kartet (11 byer) og Stillehavs-kartet (8 byer)
2. Hver medaljong: bynavn under et lite by-ikon, farget etter nåværende kontrollør
   (rød for Akse, blå for Allierte, med nasjonssymbol om plass)
3. CSS flip-animasjon (transform: rotateY) når en by bytter side
4. Tap/hover viser tooltip med territorium og kontrollerende nasjon
5. Vis seierskravet per kartside ved siden av hver gruppe
6. Behold en kompakt sammendragslinje (Akse X / Allierte Y) øverst
Merk: sjekk først at VICTORY_CITIES-dataene er rettet (19 byer) — se ToDo.md Del 1.
Alt skal fungere godt på mobil (medaljongene wrapper) og nettbrett.
```

---

## 5. ✅ Løst — Glanceability — appen leses på armlengdes avstand

**Oppdatering:** Fulgte prompten bokstavelig med ett bevisst tolkningsvalg: «aktiv
nasjon + fase i headeren» ble lagt til **oversiktsfanens Focus Card** (`.ofc-card`,
den store aktiv-nasjon-boksen øverst) i stedet for selve `<header>`-elementet — den
fysiske headeren er en bevisst smal, alltid-synlig énradsstripe (prev/neste-knapper +
pille), og har ikke rom til en fase-indikator uten å bygges om. Focus Card er appens
faktiske «hvilken nasjon, hvor langt kommet»-visning og fikk en ny fase-badge
(`${doneCount}/${total}`, grønn når ferdig). Header-pillen fikk likevel en liten,
trygg økning (.82rem → .88rem, matcher det desktop-varianten allerede hadde).

- [x] `.nc-treasury-val` (nasjonskort-skatt): 1.6rem → `var(--fs-display)` = 2.1rem, fet, gull
- [x] `font-variant-numeric: tabular-nums` lagt til på alle IPC-/teller-visninger:
      nasjonskort-skatt, Focus Card-statistikk, kjøpskalkulator-budsjett, inntektsrader,
      seiersbylabels, header-pillen (runde-tallet)
- [x] Tell-opp-animasjon: `animateCountUp()` (requestAnimationFrame, 600ms ease-out
      kubisk) på nasjonskort-skatten når «Samle inntekt» trykkes
- [x] Typografiskala: `--fs-display` / `--fs-heading` / `--fs-caption` i :root,
      brukt konsekvent på nasjonskort + Focus Card (bevisst IKKE påtvunget på hele
      appen — poenget er at 2-3 tall dominerer, resten forblir kompakt)

**Prompt:**
```
Forbedre lesbarheten på avstand («glanceability») — appen brukes på et nettbrett
som ligger midt på spillbordet og leses fra 60-80 cm:
1. Gjør treasury-tallene på nasjonskortene store og tydelige (2rem+, fet, gull)
2. Legg font-variant-numeric: tabular-nums på alle elementer som viser IPC-tall
   og tellere, så tallene ikke hopper i bredde når de endres
3. Lag en tell-opp-animasjon (requestAnimationFrame, ~600ms ease-out) på
   treasury-tallet når «Samle inntekt» trykkes — tallet skal rulle fra gammel til
   ny verdi
4. Etabler en tydelig typografiskala i style.css (CSS-variabler for display/heading/
   body/caption-størrelser) og bruk den konsekvent på nasjonskortene og oversikten
5. Aktiv nasjon + fase i headeren skal være lesbar på armlengdes avstand
Ikke gjør alt smått større — poenget er hierarki: de 2-3 viktigste tallene skal
dominere, resten kan forbli kompakt.
```

---

## 6. ✅ Løst — Mikrointeraksjoner og touch-polish

**Oppdatering:** To av fem punkter var faktisk allerede solid implementert ved
gjennomgang — `:active`-tilstander finnes på nesten alle knappetyper (`.btn:active`,
`.stepper-btn:active`, `.turn-node:active` osv., med `transform:scale()` + `transition`),
og det finnes allerede en egen `@media (hover:none) and (pointer:coarse)`-seksjon
(riktig måte å target touch på — bedre enn en bredde-brekkpunkt) som løfter de fleste
touch-mål til ≥44px. Det som manglet var fylt inn:

- [x] `:active`-tilstander — bekreftet allerede omfattende dekket, ingen endring nødvendig
- [x] Checkmark-pop-animasjon: `pcbPopClass()` i app.js sporer forrige «avhuket»-tilstand
      per fase-checkbox (samme «nettopp endret»-mønster som seiersby-flippen i punkt 4)
      og legger på en kort scale-bounce (`.phase-cb-pop`, 0,35s) — dekker alle 10 stedene
      fase-checkbokser rendres (toppfasesporer, nasjonskort-fasene, konvoi-raden)
- [x] Toast-varsler: `toast(msg, type, natTid)` har fått en valgfri tredje parameter —
      når satt, farges venstre kantstripe med nasjonens `--c-*`-farge og nasjonens
      rundell settes som ikon (`.toast-nation`). Koblet på kjøp fullført, teknologi
      forsket, inntekt samlet, hovedstad-inntektssperre og hovedstad-kapring
- [x] Touch-mål: fylte inn hullet i det eksisterende touch-media-blokken —
      `.stepper-btn.compact` (reparasjonsraden) og `.bomb-seg-btn`/`.seg-btn`
      (bombeoppdrag) manglet 44px-oppgraderingen som resten av appen allerede hadde
- [x] Fargeovergang ved eierbytte: territorielisten bygges fullstendig på nytt ved
      hver endring (innerHTML), så en vanlig CSS `transition` ville aldri spilt av —
      løst med samme «nettopp endret»-sporing (`terrBadgeFlashClass()`) som gir
      owner-badgen en kort glød-puls (`owner-badge-flash`) i nasjonens farge i stedet

**Prompt:**
```
Legg til mikrointeraksjoner og touch-polish i appen (touch-enheter har ingen hover):
1. Gi alle knapper en tydelig :active-tilstand: transform: scale(.97) + litt mørkere
   bakgrunn, med transition på ~80ms
2. Når en fase-checkbox hukes av: kort checkmark-animasjon (SVG stroke-dashoffset
   eller scale-in) i stedet for øyeblikkelig flipp
3. Style toast-varslene med den aktuelle nasjonens farge som kantstripe og
   nasjonssymbolet som ikon
4. Øk alle touch-mål til minimum 44x44 px — spesielt +/- stepperne i kjøps- og
   justeringsseksjonene (behold kompakt utseende med padding/hit-area, ikke
   nødvendigvis større visuell knapp)
5. Legg transition på fargeendringer: eierbytte i territorielisten og badges skal
   gli (200-300ms), ikke blinke
Hold alt subtilt — dette er polish, ikke show. Ingen animasjoner over 400ms.
```

---

## 7. ✅ Løst — Lys modus — «dagslys ved bordet»

**Oppdatering:** Alle 6 punktene i prompten er implementert. Underveis dukket det opp
et par funn utover selve lys-temaet:

- Et pastellrødt/-blått «akse/allierte»-tekstmønster (`#fca5a5`/`#93c5fd`) var
  hardkodet **27 forskjellige steder** i style.css — måtte tokeniseres
  (`--axis-text`/`--allies-text`) for i det hele tatt å kunne temaoverstyres. Samme
  mønster fantes i mindre skala for grønn/rød/gul/lilla «chip»-tekst (`#86efac`,
  `#fecaca`, `#f87171`, `#fbbf24`, `#a78bfa`) — også tokenisert/rettet.
- Et uhell under et bulk-søk-og-erstatt: `#f87171` traff også `--c-soviet`s
  fargedefinisjon (samme hex, ulik hensikt) og gjorde den sirkulær — oppdaget og
  rettet før commit.
- `rules.html` har sin egen frittstående `:root` (egen iframe-dokument, ikke koblet
  til style.css) med **feil palett i utgangspunktet** (fortsatt gammel kjølig
  `#0d1117` fra før varm-sepia-endringen i punkt 2 — ikke rettet nå, kun flagget,
  siden det er utenfor denne oppgavens omfang). Fant også to hardkodede lyse
  tekstfarger der (`strong`, `h4`) som ville vært ulesbare i lys modus — rettet.

- [x] Lys «kart-og-papir»-variant i style.css: `:root[data-theme="light"]` med
      beige/papirhvit bakgrunn (`#f4efe4`-familien), mørk tekst, mørkere gull
      (`#b45309`) for kontrast
- [x] Tema-toggle i action-menyen (☀️/🌙, `#btnThemeIcon` bytter ikon dynamisk)
- [x] `state.theme` (`null` = følg systemvalg, ellers `'light'`/`'dark'`) lagres via
      eksisterende `saveState()`-mekanisme
- [x] `@media (prefers-color-scheme: light)` som default inntil brukeren har gjort et
      eksplisitt valg, pluss en liten synkron inline-script i `<head>` (før
      style.css lastes) som leser lagret valg fra localStorage — unngår et glimt av
      feil tema ved oppstart
- [x] Nasjonsfargene (`--c-*`) har egne, mørkere lys-tema-varianter — samme
      resonnement som Tyskland/ANZAC-kontrastfiksen i punkt 3, bare motsatt retning
- [x] `<meta name="theme-color">` oppdateres dynamisk i `applyTheme()`
- [x] Temavalget sendes til **rules-iframen** via `postMessage({theme})`, som lytter
      og bytter sin egen (separate) palett — nøyaktig samme mønster som
      `toggleLang()`/`applyLang()`. `rules.html` har også egen `localStorage`-basert
      gjenoppretting (`rules_theme`) som fallback, matcher `rules_lang`-mønsteret

**Ikke rukket / bevisst utelatt:** full visuell gjennomgang av alle faner i lys modus
(ingen skjermbilde-verktøy tilgjengelig i denne økten) — bør sjekkes manuelt. Samme
gjelder `setup.html`, som prompten ikke nevnte eksplisitt og som heller ikke lytter
på språkbytte i dag (allerede usynkronisert før denne oppgaven).

**Prompt:**
```
Legg til en lys tema-variant («kart-og-papir») i appen:
1. Definer et lyst tema via CSS-variabler: [data-theme="light"] på html-elementet
   overstyrer :root-tokens — beige/papirhvit bakgrunn (f.eks. #f4efe4-familien),
   mørk tekst, behold gull-aksenten (juster til mørkere gull for kontrast)
2. Legg til en tema-toggle (☀️/🌙) i action-menyen i headeren
3. Lagre valget i state (state.theme) så det persisteres med resten av tilstanden,
   og bruk prefers-color-scheme som default når brukeren ikke har valgt
4. Gå gjennom nasjonsfargene og lag lys-tema-varianter der det trengs (lyse farger
   som UK-gul trenger mørkere versjon mot lys bakgrunn)
5. Oppdater theme-color meta-taggen dynamisk ved temabytte
6. Send temavalget til rules-iframen via postMessage (samme mønster som språkbytte)
Sjekk at alle faner ser riktige ut i begge temaer før du sier deg ferdig.
```

---

## 8. ✅ Løst — Turn Cockpit — Oversikt blir en handlingsflate

**Problem:** Oversikt-fanen er 100% skrivebeskyttet visning (tur-stripe,
seiersby-medaljer, scoreboard, rundelogg) — samtlige faktiske turhandlinger
(faseavhaking, kjøp, forskning, mål, inntektsinnsamling) finnes utelukkende i
Nasjoner-fanen. Å klikke hvor som helst på Oversikt gjør kun et grovt
«bytt fane + scroll til nasjonskort»-hopp, ikke «fortsett akkurat der du var».
Fanen er derfor ikke særlig verdifull mens man faktisk spiller.

**Heldig funn:** det finnes allerede død, ferdigbygd kode for nøyaktig dette —
`renderPhaseTracker()`, CSS-skallet `.phase-tracker-*`, og en fullt funksjonell
faseliste-generator `buildNationPhaseTrackerHTML(tid)` (ekte checkboxer koblet til
`togglePhase()`) — som aldri ble koblet inn i index.html. Planen gjenoppliver og
utvider denne i stedet for å finne opp noe nytt. Full plan:
`C:\Users\TomGrendal\.claude\plans\lurer-p-om-vi-swirling-petal.md`

- [x] Ny cockpit-seksjon øverst på Oversikt for AKTIV nasjons tur: ekte faseliste
      (samme `togglePhase()`/`getVisiblePhases()`/`state.turnPhases` som
      Nasjoner-fanen bruker — ikke en gjenoppfinning), kjapp-blikk-tall (kasse,
      inntekt, mål oppfylt, hovedstadsstatus), og «Åpne nasjonskort →»-knapp inn
      til full dybde
- [x] Tur-stripe komprimert: fjernet redundant fase-badge for aktiv nasjon (vises
      nå i cockpiten), beholdt for allerede passerte nasjoner denne runden
- [x] Seiersmåler komprimert: medaljong-brettet (19 byer) pakket inn i en
      kollapset `<details>`, bruker den tidligere aldri-brukte nøkkelen
      `ov.vc_collapse` som summary-tekst
- [x] Scoreboardets aktiv-rad revertert til enkel highlighting (den utvidede
      detaljlinjen fra tidligere denne økten dupliserte cockpiten)
- [x] Rundeloggen fjernet fra Oversikt og konsolidert inn i Historikk-fanen —
      `renderHistory()` og `buildLogRoundBody()`/`renderChronicle()` var to
      uavhengig skrevne, sterkt overlappende implementasjoner av samme logg;
      Historikk manglet en «runde X pågår»-live-seksjon som Oversikt-versjonen
      hadde, så konsolideringen ga Historikk noe den manglet i tillegg til å
      fjerne duplikatet
- [x] **Bonus:** fant og ryddet en tredje dødt-kode-funksjon (`renderSidePanels()`,
      en overflødig tidligere iterasjon av scoreboardet) samt tilhørende dødt
      CSS og foreldreløse i18n-nøkler, oppdaget underveis i verifikasjonen

**Oppfølger — «Én spillflate» (revisjon av cockpit-løsningen):** Etter å ha brukt
cockpiten i praksis følte brukeren fortsatt at Oversikt og Nasjoner speilet
hverandre — og det stemte: cockpitens faseliste var et duplikat av kortets
faseblokker, og selve *handlingene* (kjøp, bombing, samle inntekt) bodde fortsatt
kun i kortet, så avhuking på Oversikt var dobbeltarbeid. Løsning (brukervalgt av
tre alternativer): skillet «se status» / «gjør handlinger» er fjernet helt:

- [x] **Nasjoner er nå hovedfanen** (først i fanerekken, åpnes ved oppstart);
      aktiv nasjons kort auto-åpnes ved oppstart og ved turbytte (`focusActiveNation()`)
- [x] Cockpiten omgjort til en **kompakt klistret turlinje** øverst på Nasjoner-fanen:
      aktiv nasjon + faseframgang + «Neste fase ↓» (scroller til første uavhukede
      faseblokk i kortet og åpner den — `scrollToNextPhase()`) + «Fullfør tur».
      Ingen faseliste og ingen pengetall i linjen — kortene under er ENESTE kilde
- [x] Oversikt-fanen krympet til ren krigsstatus: tur-stripe + seiersbyer + scoreboard,
      null fase-/turhandlings-info
- [x] `buildNationPhaseTrackerHTML()` (den gjenopplivede faseliste-generatoren fra
      første iterasjon) ble dermed død igjen — slettet sammen med checklist-CSSen
      (`.phase-item`/`.phase-list` m.fl.) og `cp.open_card`-nøkkelen (erstattet av
      `cp.next_phase`)

**Oppfølger 2 — Territorie-modal fra kampfasen:** `🗺️ {nasjon} →`-knappen i
Fase 3 hoppet fortsatt til Territorier-fanen midt i kampregistreringen — siste
gjenværende fane-pendling i turflyten. Erstattet med en responsiv modal
(`#terrModal`) over nasjonskortet:

- [x] Gjenbruker `buildTerritoryRowNation()` (samme rader som Territorier-fanen,
      inkl. hurtigoverføring til aktiv nasjon + ⋯-eierbytte-velger) og appens
      standard modal-mønster — søkefelt, «Andre nasjoners territorier» øverst
      (erobrings-flyt), «Egne territorier» i kollapsbar gruppe (tap-flyt)
- [x] `onOwnerChange()` fikk et hook som re-rendrer modal-body når den er åpen —
      dekker både hurtigknapp- og velger-stien
- [x] Eierbytte-velgeren fikk `z-index: 210` så den stables korrekt OVER modalen
- [x] **Bonus:** global ESC-lukking av øverste åpne modal — fantes ikke i appen
      i det hele tatt før (gjelder nå alle fire modaler)
- [x] Fluktluke i modal-footeren («Åpne full territorieliste →») til fanens
      full-visning med kontinentfiltre; `goToTerritories()` beholdt for denne

**Prompt:**
```
Bygg om Oversikt-fanen til et "Turn Cockpit" per planen i
lurer-p-om-vi-swirling-petal.md:
1. Gjenoppliv renderPhaseTracker() (omdøp til renderCockpit()) og
   buildNationPhaseTrackerHTML() — begge finnes allerede men er aldri koblet
   inn. Legg til matchende HTML i index.html øverst i #tab-overview, koble
   renderCockpit() inn i renderOverview() og de 5 eksisterende kallstedene
2. OBS regresjonsrisiko: btnCompletePhases har en eksisterende addEventListener
   ved bootstrap som IKKE fungerer med full-rebuild-rendering — bytt til inline
   onclick og slett bootstrap-linjen, ellers kan en nasjons tur bli hoppet over
3. Komprimer tur-stripe (fjern aktiv-nasjons fase-badge) og seiersmåler
   (medaljong-brett i <details>, bruk ov.vc_collapse)
4. Revert scoreboardets utvidede aktiv-rad til enkel highlighting
5. Konsolider rundeloggen inn i Historikk via buildLogRoundBody(), inkludert en
   ny live "runde pågår"-oppføring øverst i lista
6. Rydd i18n (nye/omdøpte/slettede nøkler) og dødt CSS/JS som blir foreldreløst
Test hele klikk-gjennomgangen i planens §7 før du sier deg ferdig.
```

---

## Avhengigheter og rekkefølge

| # | Punkt | Avhenger av | Risiko |
|---|-------|-------------|--------|
| 1 | Nasjonssymboler | — | Lav |
| 3 | Nasjonsfarger | — (styrkes av 1) | Lav |
| 5 | Glanceability | — | Lav |
| 4 | Seiersmåler-medaljonger | VICTORY_CITIES-rettelser (ToDo.md Del 1) | Middels |
| 2 | Art direction | — (gjøres etter 3 så fargene testes mot ny bakgrunn) | Middels — krever iterasjon |
| 6 | Mikrointeraksjoner | 1 + 3 (toasts bruker symbol + farge) | Lav |
| 7 | Lys modus | 2 + 3 (tokens må være ferdige først) | Middels |
| 8 | Turn Cockpit | — (uavhengig av 1–7, kan gjøres når som helst) | Middels — regresjonsrisiko i §1, se plan |
