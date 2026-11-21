---
title: "3-2-1-regelen for sikkerhetskopiering: tenk i uavhengige feil"
description: "En sikkerhetskopi bedømmes bare på gjenopprettingen. 3-2-1 er en modell for uavhengige feil: hver del svarer på en trussel, og du ser hvorfor regelen er slik."
date: "2026-11-21"
tags: ["Sikkerhet"]
draft: true
---

En sikkerhetskopi bedømmes etter én ting: om du får dataene tilbake når du trenger dem. Alt som kommer før, er forberedelse, og det som ikke er prøvd, er en påstand.

3-2-1-regelen, som ofte tilskrives fotografen Peter Krogh, huskes som regel som tre tall. Tallene er en forkortelse for en modell: kopiene dine må ikke kunne gå tapt av samme årsak. Når du har modellen, ser du hvorfor regelen ser ut som den gjør, og når den ikke rekker.

Dette er en påminnelse om prinsipper som er lette å slutte å tenke på etter mange år i arbeidslivet, og som sjelden blir tenkt på hjemme.

## Modellen: uavhengige feil

Alt som kan ødelegge data, har en rekkevidde. En slettet fil rammer én fil, en død disk én disk og en brann et helt rom. En kopi beskytter deg bare mot feil som ikke når fram til den, så uavhengighet gjelder alltid i forhold til en bestemt trussel. To kopier som kan gå tapt av samme årsak, er én kopi mot den årsaken. Originalen og en kopi i samme rom er én kopi mot brann, men to mot en død disk.

Spørsmålet å stille til et oppsett er dette: **for hver hendelse jeg må overleve, hvor mange kopier er det igjen etterpå?** Null betyr ingen beskyttelse mot hendelsen, og én betyr ingen margin. Bestem hvilke hendelser som skal ha margin.

To tall avgjør hvor mye av dette du trenger. RPO (Recovery Point Objective) er hvor mye data du tåler å miste, målt i tid, og RTO (Recovery Time Objective) er hvor lenge du tåler å være nede. De avgjør hvor ofte du tar kopi og hvilke medier og steder du bruker, så skriv dem ned før du velger løsning. Sorter dataene også, for uerstattelige data trenger hele regelen, reproduserbare data bare koden og konfigurasjonen som bygger dem opp igjen, og midlertidige data ingenting.

## Regelen, del for del

| Del | Trusselen den svarer på | Uten den |
| --- | --- | --- |
| 3 kopier av dataene, originalen medregnet | En ny feil mens du gjenoppretter etter den første | En feil under gjenopprettingen, og du står uten noe |
| 2 ulike slags lagring | Feil som rammer en hel type: samme produksjonsfeil, samme slitasje, samme programfeil | Én systematisk feil tar alle kopiene samtidig |
| 1 kopi på et annet sted | Hendelser som tar et helt sted: brann, vann, tyveri, lynnedslag, overspenning | Alle kopiene står innenfor samme hendelse |

**Hvorfor tre.** Med tre kopier er det fortsatt to igjen etter ett enkelt tap (en død disk, en slettet fil), én å gjenopprette fra og én i reserve hvis gjenopprettingen går galt. Reserven trengs fordi en gjenoppretting leser alt, under press og kanskje for første gang på lenge, og da kommer skjulte feil for en dag. Mot tap av et helt sted eller en feil i én type lagring garanterer 3-2-1 bare at én kopi overlever, uten margin. Det er det minste regelen gir. Data som skal ha margin også der, trenger flere kopier, flere slags lagring eller flere steder.

**Hvorfor to slags.** To disker av samme type, kjøpt og tatt i bruk samtidig, slites likt og kan ha samme skjulte feil, og det samme gjelder to kopier som styres av samme programvare. Det avgjørende er om feilmåtene er forskjellige, uansett hva de heter. Med tre kopier på to slags lagring deler to av dem type, og rammes den typen av en feil, sitter du igjen med én kopi uten margin.

**Hvorfor ett annet sted.** Hvor langt unna det må ligge, avhenger av hendelsen du vil beskytte deg mot. Kopien må ligge utenfor rekkevidden til alt som kan ramme originalen, så samme rom, samme bygning eller samme strømkurs betyr samme skjebne.

## Det som ikke er en sikkerhetskopi

Tre ting ligner på sikkerhetskopi og er nyttige, men de svarer på et annet spørsmål.

- **RAID og replikering** holder tjenesten i gang når en disk eller en maskin faller ut, og i vanlig drift sender de hver endring videre i løpet av øyeblikk, så en sletting eller en ødelagt fil kopieres like raskt som alt annet.
- **Synkronisering** holder to steder like, så en fil som slettes det ene stedet, forsvinner det andre også. Et skybasert fotobibliotek på telefonen er synkronisering hos én leverandør under én konto, så sletter du et bilde eller mister kontoen, er det borte overalt.
- **Øyeblikksbilder på samme system** er greie å ha for rask angring, men de deler maskinvare, filsystem og administratorer med det de beskytter, så de blir en kopi først når de er kopiert ut.

En enkel test er om én kommando eller én feil i produksjon kan fjerne kopien. Kan den det, er den ikke en sikkerhetskopi. Å bestå testen er nødvendig, men ikke nok, for kopien må også ligge utenfor maskinvare- og stedsdomenet.

## Uavhengighet er den egentlige regelen

Maskinvare og sted er lette å få øye på, men det administrative domenet blir glemt. Kan alle kopiene nås med samme konto, samme legitimasjon eller fra samme administrasjonsmaskin, kan én feil slette dem alle, enten det er en kompromittert maskin eller et opprydningsskript med feil sti. Kopiene kan ligge fysisk atskilt og likevel være én kopi administrativt.

- **Atskilt tilgang.** Sikkerhetskopiene har egne kontoer og egen legitimasjon, og den som administrerer produksjonen, blir ikke administrator av kopiene av den grunn. Produksjonssiden kan skrive nye kopier, men ikke slette eller overskrive gamle, og enda bedre er det om sikkerhetskopisystemet henter dataene selv med bare lesetilgang.
- **Annen leverandør, avtale eller konto.** Kopier hos samme leverandør, under samme avtale, juridiske enhet eller betaling kan dele skjebne selv med atskilte kontoer og uforanderlighet. En konto kan bli stengt ved en feil eller for brudd på vilkårene, data kan bli slettet etter en ubetalt regning, og en leverandør kan legge ned tjenesten. Minst én kopi bør ligge utenfor dette domenet.
- **Versjoner i en kjede.** Versjonene i en inkrementell eller deduplisert kjede avhenger av hverandre, så én skadet basisblokk eller en skadet indeks i arkivet kan ta alle senere versjoner. Kontroller arkivet jevnlig, og lag minst én kopi direkte fra kilden inn i et separat arkiv, siden replikering fra samme arkiv kopierer skaden.

## Data råtner i det stille

Lagrede data forringes selv om ingen rører dem. Disker får sektorer som ikke lenger lar seg lese, flashminne mister ladning, fortere når det er slitt eller har ligget lenge uten strøm, og optiske plater og bånd eldes. Feil kan også snike seg inn underveis, ved overføring, i minnet eller i fastvaren. Det meste skjer i det stille, og filen åpner seg, eller ser ut til det, til dagen den ikke gjør det. Lagringssystemer på jobb sjekker ofte for dette. En disk i en skuff hjemme, eller en minnepinne med de eneste bildene, får ingen slik oppfølging.

Forsvaret er det samme overalt. Lagre sjekksummer og kontroller hver kopi jevnlig, slik at en råtten kopi blir funnet mens en god fortsatt finnes, og erstatt den fra den gode. Flytt data over på nye medier før de gamle eldes ut. Og ta vare på nok historikk, for råte som blir kopiert inn i hver sikkerhetskopi og lever lenger enn oppbevaringstiden, tar den siste gode versjonen med seg.

## 3-2-1-1-0 og oppbevaringstid

Den utvidede formen legger til to ledd for feil den enkle regelen ikke dekker.

**+1: en kopi som er frakoblet eller uforanderlig.** Den trenger ikke være en fjerde kopi og kan være en av de tre, ofte den utenfor stedet, men den gir ikke margin i seg selv. En frakoblet kopi er fysisk skilt fra nettet bortsett fra mens den skrives, så ingenting som kjører der, kommer til den. En uforanderlig kopi kan bare skrives én gang og beholdes i en fastsatt tid som ingen kan korte ned, heller ikke en administrator. Garantien gjelder bare hvis lagringen selv håndhever låsen, for en innstilling som samme legitimasjon kan endre, garanterer ingenting.

**0: null feil i verifiserte gjenopprettinger.** «Jobben meldte suksess» sier lite. Det som teller, er en gjenoppretting som er lest tilbake og sammenlignet, og som ga null feil.

En sikkerhetskopi som overskriver gårsdagens med dagens, er et speil med litt forsinkelse, for ødelagte data blir kopiert ved neste kjøring. Du trenger historikk, vanligvis i nivåer som tynnes ut med alderen: daglige kopier for de siste ukene, ukentlige for månedene og månedlige for lenger tilbake.

Sammenlign historikken med hvor lang tid det tar før du oppdager et problem. Begynner stille ødeleggelse i mars, du merker det i mai og beholder seks uker, finnes det ingen hel versjon. **Er tiden til oppdagelse lengre enn oppbevaringstiden, er dataene tapt.** Oppbevaringen må derfor være lengre enn den lengste oppdagelsestiden du realistisk kan vente.

Går du gjennom spørsmålet for et typisk oppsett, med originalen og en lokal kopi på sted 1 og en uforanderlig kopi på sted 2, blir resultatet slik.

![Et rutenett som viser hvor mange kopier som overlever hver hendelse. Kolonnene er originalen (sted 1, type A), en lokal kopi (sted 1, type B), en ekstern kopi (sted 2, type A, uforanderlig) og antall kopier igjen. En slettet fil eller en død disk: originalen er tapt, de to andre er i behold, 2 kopier igjen. En feil i type A: originalen og den eksterne kopien er tapt, den lokale kopien er i behold, 1 igjen. Brann på sted 1: originalen og den lokale kopien er tapt, den eksterne kopien er i behold, 1 igjen. En kompromittert felles adminkonto: originalen og den lokale kopien er tapt, den eksterne kopien er i behold, 1 igjen. Skade som er eldre enn oppbevaringstiden: alle tre er tapt, 0 igjen. Hvert resultat har også et ord: margin for 2, ingen margin for 1 og tapte data for 0.](/images/3-2-1-regelen-for-sikkerhetskopiering-tenk-i-uavhengige-feil/kopier-igjen.svg)

## Kryptering og det som ligger utenfor sikkerhetskopien

Krypter alle kopier. For kopier som forlater din kontroll, enten de havner et annet sted, hos andre eller på medier som flyttes, er kryptering et krav, og den skjer hos deg, før dataene forlater deg, med nøkler du selv har. Kryptering i ro med leverandørens nøkler beskytter verken mot en kompromittert konto eller mot leverandøren.

Nøkkelen blir en enkeltfeil for alle kopiene. Den må aldri bare ligge i sikkerhetskopien den låser opp, for da trenger du kopien for å åpne kopien, og aldri bare på systemet som kan dø. Oppbevar den på minst to steder som ikke deler skjebne, gjerne med en forseglet papirkopi et annet sted.

Sikkerhetskopien rommer dataene, konfigurasjonen og hemmelighetene. Utenfor ligger det som må komme opp uten noe fra systemet som gjenopprettes: gjenopprettingsveiledningen, nøkkelen, verktøyet til å lese formatet og tilgangen til lagringen, lagret atskilt fra sikkerhetskopien og lesbart uten den. Krever det å lese kopien en agent, en lisensserver eller en konsoll som døde med infrastrukturen eller ikke finnes lenger, er kopien ubrukelig. Gjenopprettingsrekkefølgen må også være uten sykluser, for eksempel når nøkkelen ligger i et hemmelighetslager som trenger en innloggingstjeneste som selv må gjenopprettes først.

## Gjenopprettingsprøven

Kopier svikter stille, med feil sti, manglende filer, en jobb som «lykkes» med å kopiere en tom mappe, et format som krever programvare du ikke lenger har eller et ødelagt medium, og ingenting av det viser seg før du gjenoppretter.

- **Stikkprøver, ofte.** Gjenopprett noen tilfeldige filer og sjekk innholdet. Det er billig, så gjør det jevnlig.
- **Full gjenoppretting, på fast tid.** Start fra rent utstyr med bare kopien og det som ligger utenfor sikkerhetskopien, og la prøven omfatte tap av et helt sted, scenarioet det er lettest å la være å teste. Ta tiden og sammenlign med RTO.
- **Integritet.** Kontroller sjekksummene, som beskrevet under datarot. En gjenopprettet database skal starte og bestå sin egen konsistenssjekk, og siden en database som kjører og kopieres på filnivå kan bli ubrukelig, bør du bruke databasens egen konsistente dump eller øyeblikksbilde.

Etter et innbrudd bygger du systemene opp igjen fra konfigurasjon og kode og gjenoppretter bare data fra et tidspunkt før innbruddet, for hele systembilder får inntrengeren med.

Automatiser tidsplanen, kontrollen og oppryddingen, for manuelle sikkerhetskopier blir før eller siden glemt. Et varsel om feil er ikke nok, for en jobb som aldri starter, sender ingen feilmelding, så varsle på fravær av suksess for både kopieringen og kontrollen, og la stillhet regnes som et funn, slik som i [Hvem vokter vokterne? Tilsyn i IT-infrastruktur](/blogg/hvem-vokter-vokterne-i-it-infrastruktur/).

## Sjekkliste

- Vi har skrevet ned RPO og RTO, og for hver hendelse vi må overleve, vet vi hvor mange kopier som er igjen etterpå, med den marginen vi har bestemt.
- Det finnes minst tre kopier, originalen medregnet, på minst to ulike slags lagring, og minst én ligger et annet sted, utenfor leverandøren og avtalen de andre hører til.
- Ingen enkelthendelse (en feil, en konto, en maskin eller en person) kan slette alle kopiene. Produksjonen kan ikke slette eller overskrive dem, og best er at den ikke har tilgang i det hele tatt. Minst én kopi lages direkte fra kilden inn i et separat arkiv, og arkivet kontrolleres jevnlig.
- Minst én kopi er frakoblet eller uforanderlig, og lagringen selv håndhever det.
- Vi har flere versjoner, og oppbevaringen er lengre enn tiden det tar å oppdage et problem.
- Hver kopi kontrolleres jevnlig mot sjekksummer, og medier fornyes før de eldes ut.
- Alle kopier er kryptert med nøkler vi selv holder, og nøkkelen ligger på minst to steder som ikke deler skjebne.
- Veiledningen, nøkkelen, verktøyet til å lese formatet og tilgangen til lagringen ligger utenfor sikkerhetskopien og er lesbare uten den, og gjenopprettingsrekkefølgen har ingen sykluser.
- Kopieringen og kontrollen kjører automatisk, og vi får alarm når en vellykket kjøring av en av dem uteblir.
- Vi har gjenopprettet fra rent utstyr, også med tap av et helt sted, tatt tiden mot RTO og gjentar det på fast tid.

## En historie til slutt

En venn fortalte meg en gang om en harddisk han brukte halvannet år på å berge. Den tilhørte en venninne av ham, en kvinne som hadde mistet mannen sin, og på den lå alle bildene av ham, fra ferier og turer de to hadde hatt sammen.

Firmaer som driver med datagjenoppretting, ville ikke ta jobben. Det vanskelige var å finne en identisk donordisk fra samme produksjonsparti, og deretter å gjennomføre selve gjenopprettingen. Han lette selv etter disken, gjorde arbeidet på egen tid og tok ikke betalt.

Hun fikk tilbake hvert eneste bilde av mannen sin, og hun gråt da hun fikk dem.
