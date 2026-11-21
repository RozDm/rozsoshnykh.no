---
title: "3-2-1-regelen for sikkerhetskopiering: tenk i uavhengige feil"
description: "En sikkerhetskopi bedømmes bare på gjenopprettingen. 3-2-1 er en modell for uavhengige feil: hver del svarer på en trussel, og du ser hvorfor regelen er slik."
date: "2026-11-21"
tags: ["Sikkerhet"]
draft: true
---

En sikkerhetskopi bedømmes etter én ting: om du får dataene tilbake når du trenger dem. Alt som kommer før, er forberedelse, og det som ikke er prøvd, er en påstand.

3-2-1-regelen, som ofte tilskrives fotografen Peter Krogh, huskes som regel som tre tall. Tallene er en forkortelse for en modell: kopiene dine må ikke kunne gå tapt av samme årsak. Når du har modellen, ser du hvorfor regelen ser ut som den gjør, og når den ikke rekker.

## Modellen: uavhengige feil

Alt som kan ødelegge data, har en rekkevidde. En slettet fil rammer én fil, en død disk én disk og en brann et helt rom. En kopi beskytter deg bare mot feil som ikke når fram til den, så uavhengighet gjelder alltid i forhold til en bestemt trussel. To kopier som kan gå tapt av samme årsak, er én kopi mot den årsaken. Originalen og en kopi i samme rom er én kopi mot brann, men to mot en død disk.

Spørsmålet å stille til et oppsett er dette: **for hver hendelse jeg må overleve, hvor mange kopier er det igjen etterpå?** Null betyr ingen beskyttelse mot hendelsen, og én betyr ingen margin. Bestem hvilke hendelser som skal ha margin.

## Regelen, del for del

| Del | Trusselen den svarer på | Uten den |
| --- | --- | --- |
| 3 kopier av dataene, originalen medregnet | En ny feil mens du gjenoppretter etter den første | En feil under gjenopprettingen, og du står uten noe |
| 2 ulike slags lagring | Feil som rammer en hel type: samme produksjonsfeil, samme slitasje, samme programfeil | Én systematisk feil tar alle kopiene samtidig |
| 1 kopi på et annet sted | Hendelser som tar et helt sted: brann, vann, tyveri, lynnedslag, overspenning | Alle kopiene står innenfor samme hendelse |

**Hvorfor tre.** Med tre kopier er det fortsatt to igjen etter ett enkelt tap (en død disk, en slettet fil), én å gjenopprette fra og én i reserve hvis gjenopprettingen går galt. Reserven trengs fordi en gjenoppretting leser alt, under press og kanskje for første gang på lenge, og da kommer skjulte feil for en dag. Mot tap av et helt sted eller en feil i én type lagring garanterer 3-2-1 bare at én kopi overlever, uten margin. Det er det minste regelen gir. Data som skal ha margin også der, trenger flere kopier, flere slags lagring eller flere steder.

**Hvorfor to slags.** To disker av samme type, kjøpt og tatt i bruk samtidig, slites likt og kan ha samme skjulte feil, og det samme gjelder to kopier som styres av samme programvare. Det avgjørende er om feilmåtene er forskjellige, uansett hva de heter. Med tre kopier på to slags lagring deler to av dem type, og rammes den typen av en feil, sitter du igjen med én kopi uten margin.

**Hvorfor ett annet sted.** Hvor langt unna det må ligge, avhenger av hendelsen du vil beskytte deg mot. Kopien må ligge utenfor rekkevidden til alt som kan ramme originalen, så samme rom, samme bygning eller samme strømkurs betyr samme skjebne.

## Bestem først hva du må tåle

To tall styrer alt annet, og begge er avgjørelser som må tas før du velger løsning. RPO (Recovery Point Objective) er hvor mye data du tåler å miste, målt i tid. RTO (Recovery Time Objective) er hvor lenge du tåler å være nede. Skriv dem ned, og mål mot dem senere.

RPO avgjør hvor ofte du tar kopi, og en kopi hver natt betyr at du kan miste opptil et døgn. Lav RPO får du sjelden ved å kopiere alt ofte. Den får du med en basiskopi og et kontinuerlig arkiv av transaksjonsloggen, slik at RPO avhenger av loggen og ikke av hvor ofte fullkopiene kjører. RTO avgjør medier og plassering, for det tar tid å hente tilbake store datamengder over en trang linje. En kopi som trenger en uke for å komme tilbake, beskytter mot katastrofer, men hjelper ikke mot en feil en tirsdag. Mange har derfor to nivåer, en nær og rask kopi mot hverdagens feil og en fjern kopi mot det som tar hele stedet.

Sorter dataene også, for ikke alt trenger hele regelen og lang historikk. Uerstattelige data får begge deler. Reproduserbare data trenger bare det som gjenskaper dem, altså kode og konfigurasjon, som du uansett tar kopi av. Midlertidige data som hurtigbuffere, byggeresultater og indekser som kan bygges på nytt, trenger ingen kopi. Uten denne sorteringen vokser både kostnaden og RTO ut av proporsjoner.

## Det som ikke er en sikkerhetskopi

Tre ting ligner på sikkerhetskopi og er nyttige, men de svarer på et annet spørsmål.

- **RAID og replikering** gir tilgjengelighet, slik at tjenesten går videre når en disk eller en maskin faller ut. I vanlig drift sender de hver endring videre i løpet av øyeblikk, så en sletting, en ødelagt fil eller en feilaktig overskriving kopieres like raskt som alt annet. Forsinket replikering gir et vindu til å stoppe feilen før den sprer seg og gjenopprette fra den etterslepende repliken, men ligger fortsatt i samme administrative domene.
- **Synkronisering** holder to steder like, så en fil som slettes det ene stedet, forsvinner det andre også. Versjonshistorikk eller papirkurv i en synkroniseringstjeneste hjelper mot noen feil og ligger i samme administrative domene.
- **Øyeblikksbilder på samme system** er greie å ha for rask angring, men de deler maskinvare, filsystem og administratorer med det de beskytter. Faller systemet, faller bildene med det. De blir en kopi først når de er kopiert ut.

En enkel test er om én kommando eller én feil i produksjon kan fjerne kopien. Kan den det, er den ikke en sikkerhetskopi. Å bestå testen er nødvendig, men ikke nok, for kopien må også ligge utenfor maskinvare- og stedsdomenet. Øyeblikksbilder som produksjonen ikke kan slette, deler fortsatt maskinvaren.

## Konsistens

Kopierte filer er ikke det samme som en konsistent tilstand. En database som kopieres på filnivå mens den kjører, kan bli ubrukelig, og systemer som hører sammen, kan være tatt på ulike tidspunkt, for eksempel databasen klokken 02.00 og fillageret den peker på klokken 02.40. Bruk databasens egen konsistente dump eller øyeblikksbilde. Endrer systemene seg hele tiden, blir de ikke konsistente uansett rekkefølge, og rekkefølgen avgjør bare hvilken vei de glir fra hverandre. Ta systemet som refererer først, og det det refererer til etterpå, altså databasen før fillageret. Da finnes hvert objekt databasen nevner, og ekstra objekter gjør ingen skade, så lenge et objekt slettes først etter en forsinkelse som er lengre enn tiden mellom de to uttakene, og objekter ikke overskrives på stedet.

## Uavhengighet er den egentlige regelen

Maskinvare og sted er lette å få øye på, men det administrative domenet blir glemt. Kan alle kopiene nås med samme konto, samme legitimasjon eller fra samme administrasjonsmaskin, kan én feil slette dem alle, enten det er en kompromittert maskin eller et opprydningsskript med feil sti. Kopiene kan ligge fysisk atskilt og likevel være én kopi administrativt.

- **Atskilt tilgang.** Sikkerhetskopiene har egne kontoer og egen legitimasjon, og den som administrerer produksjonen, blir ikke administrator av kopiene av den grunn.
- **Minste privilegium.** Produksjonssiden kan skrive nye kopier, men ikke slette eller overskrive gamle. Enda bedre er det om sikkerhetskopisystemet henter dataene selv med bare lesetilgang, så en overtatt produksjon ikke har noen nøkkel til kopiene. Oppbevaringsreglene settes og håndheves av sikkerhetskopisystemet også.
- **Få felles avhengigheter.** Samme passordbehandler eller samme administrasjonsverktøy er et felles feilpunkt, uansett hvor mange kopier som ligger bak.
- **Organisatorisk og avtalemessig domene.** Kopier hos samme leverandør, under samme avtale, juridiske enhet eller betaling kan dele skjebne selv med atskilte kontoer og uforanderlighet. En konto kan bli stengt ved en feil eller for brudd på vilkårene, data kan bli slettet etter en ubetalt regning, og en leverandør kan legge ned tjenesten. Minst én kopi bør ligge utenfor dette domenet.
- **Uavhengighet i tid.** Versjonene i en inkrementell eller deduplisert kjede avhenger av hverandre. Én skadet basisblokk eller en skadet indeks i arkivet kan ta alle senere versjoner, og en loggkjede er det ytterste eksemplet, for ett manglende logg-segment stopper gjenopprettingen akkurat der, selv om basiskopien er hel. Kontroller integriteten til selve arkivet jevnlig, ta uavhengige fullkopier med mellomrom, og lag minst én kopi direkte fra kilden inn i et separat arkiv, siden replikering fra samme arkiv kopierer skaden.

## 3-2-1-1-0

Den utvidede formen legger til to ledd for feil den enkle regelen ikke dekker.

**+1: en kopi som er frakoblet eller uforanderlig.** Den trenger ikke være en fjerde kopi og kan være en av de tre, ofte den utenfor stedet, men den gir ikke margin i seg selv. En frakoblet kopi er fysisk skilt fra nettet bortsett fra mens den skrives, så ingenting som kjører der, kommer til den. En uforanderlig kopi kan bare skrives én gang og beholdes i en fastsatt tid som ingen kan korte ned, heller ikke en administrator. Garantien gjelder bare hvis lagringen selv håndhever låsen, for en innstilling som samme legitimasjon kan endre, garanterer ingenting. Dette dekker sletting og kryptering utført med lovlig tilgang, og feil i automatikken. Du kan ikke rydde bort kopien før tiden er ute, og det motsatte kan også skje. Den som kan skrive, kan fylle uforanderlig lagring med søppel som ingen kan slette, slik at kvoten blir full og nye kopier uteblir, eller regningen vokser til kontoen stenges. Begrens skriverettighetene, og sett en grense for hvor mye én kjøring, eller én periode, kan legge til, slik at en kjøring langt over det vanlige stopper og slår alarm før lagringen er full. En kvote alene verner bare budsjettet. Samme grense fanger en trukket kopi av søppel eller chiffertekst fra en kompromittert kilde.

**0: null feil i verifiserte gjenopprettinger.** «Jobben meldte suksess» sier lite. Det som teller, er en gjenoppretting som er lest tilbake og sammenlignet, og som ga null feil.

## Versjoner og oppbevaringstid

En sikkerhetskopi som overskriver gårsdagens med dagens, er et speil med litt forsinkelse, for ødelagte data blir kopiert ved neste kjøring. Du trenger historikk, vanligvis i nivåer som tynnes ut med alderen: daglige kopier for de siste ukene, ukentlige for månedene og månedlige for lenger tilbake.

Sammenlign historikken med hvor lang tid det tar før du oppdager et problem. Begynner stille ødeleggelse i mars, du merker det i mai og beholder seks uker, finnes det ingen hel versjon. **Er tiden til oppdagelse lengre enn oppbevaringstiden, er dataene tapt.** Oppbevaringen må derfor være lengre enn den lengste oppdagelsestiden du realistisk kan vente.

Oppbevaringen har også en øvre grense, for noen data skal slettes etter en tid, og slettingen må før eller siden nå kopiene også. Uforanderlighet og påkrevd sletting må forenes i utformingen. Enten er uforanderlighetsvinduet ikke lengre enn tiden slettingen må skje innen, eller du krypterer datasett med hver sin nøkkel og sletter nøkkelen (kryptografisk sletting), også de deponerte kopiene av den. Det virker bare med grov granularitet (per kunde, datasett eller periode), og det flytter problemet til nøkkellageret, som trenger sin egen utforming. Det må overleve like godt som dataene og likevel la nøkler bli slettet.

Går du gjennom spørsmålet for et typisk oppsett, med originalen og en lokal kopi på sted 1 og en uforanderlig kopi på sted 2, blir resultatet slik.

![Et rutenett som viser hvor mange kopier som overlever hver hendelse. Kolonnene er originalen (sted 1, type A), en lokal kopi (sted 1, type B), en ekstern kopi (sted 2, type A, uforanderlig) og antall kopier igjen. En slettet fil eller en død disk: originalen er tapt, de to andre er i behold, 2 kopier igjen. En feil i type A: originalen og den eksterne kopien er tapt, den lokale kopien er i behold, 1 igjen. Brann på sted 1: originalen og den lokale kopien er tapt, den eksterne kopien er i behold, 1 igjen. En kompromittert felles adminkonto: originalen og den lokale kopien er tapt, den eksterne kopien er i behold, 1 igjen. Skade som er eldre enn oppbevaringstiden: alle tre er tapt, 0 igjen. Hvert resultat har også et ord: margin for 2, ingen margin for 1 og tapte data for 0.](/images/3-2-1-regelen-for-sikkerhetskopiering-tenk-i-uavhengige-feil/kopier-igjen.svg)

## Kryptering

Krypter alle kopier. For kopier som forlater din kontroll, enten de havner et annet sted, hos andre eller på medier som flyttes, er kryptering et krav, og den skjer hos deg, før dataene forlater deg, med nøkler du selv har. Kryptering i ro med leverandørens nøkler beskytter verken mot en kompromittert konto eller mot leverandøren. Kanalen bør også være kryptert under overføring. Hver kopi er også en kopi for en tyv, så flere kopier gir større angrepsflate, og en sikkerhetskopi verner tilgjengelighet og integritet, men ikke mot at data blir stjålet.

Nøkkelen blir en enkeltfeil for alle kopiene. Den må aldri bare ligge i sikkerhetskopien den låser opp, for da trenger du kopien for å åpne kopien, og aldri bare på systemet som kan dø. Oppbevar den separat, på minst to steder som ikke deler skjebne, gjerne med en forseglet papirkopi et annet sted. Enda bedre er det å dele den i deler som holdes av ulike personer eller steder, slik at ingen enkeltdel åpner dataene og tapet av én del ikke stopper gjenopprettingen (M av N). Test at en som ikke har bygget oppsettet, kommer til nøkkelen.

## Hva en gjenoppretting trenger

I sikkerhetskopien ligger dataene, konfigurasjonen (hvordan systemene ble satt opp, slik at de kan bygges opp igjen fra grunnen) og hemmelighetene, krypterte som beskrevet over.

Utenfor ligger et lag null som kommer opp uten noe fra systemet som gjenopprettes. Det rommer gjenopprettingsveiledningen (trinn for trinn, skrevet for en som er stresset og ikke har bygget oppsettet), nøkkelen, verktøyet til å lese formatet og tilgangen til lagringen med kopiene, lagret atskilt fra sikkerhetskopien og lesbart uten den. Krever det å lese kopien en agent, en lisensserver eller en konsoll som døde med infrastrukturen eller ikke finnes lenger, er kopien ubrukelig.

Gjenopprettingsrekkefølgen må være uten sykluser. En nøkkel i et hemmelighetslager som trenger innloggingstjenesten, som selv må gjenopprettes, er en syklus, og det samme er kode i et arkiv bak den samme innloggingen. Tjenestene alt annet hviler på, som innlogging og navnetjenester, kommer tidlig i rekkefølgen.

Det som bevisst står utenfor, bør stå på en liste.

## Gjenopprettingsprøven

Kopier svikter stille, med feil sti, manglende filer, en jobb som «lykkes» med å kopiere en tom mappe, et format som krever programvare du ikke lenger har eller et ødelagt medium, og ingenting av det viser seg før du gjenoppretter.

- **Stikkprøver, ofte.** Gjenopprett noen tilfeldige filer og sjekk innholdet. Det er billig, så gjør det jevnlig.
- **Full gjenoppretting, på fast tid.** Start fra rent utstyr med bare kopien og det lag null rommer: nøkkelen, veiledningen, verktøyet og tilgangen til lagringen. Ta tiden og sammenlign med RTO. Mange oppdager her at RTO var et ønske.
- **Integritet.** Lagre sjekksummer når kopien lages, og kontroller dem ved lesing og med jevne mellomrom, så stille datarot i en lagret kopi blir oppdaget før du trenger den. For en database betyr gjenoppretting at den starter og består sin egen konsistenssjekk.
- **Granulær gjenoppretting.** Én tabell eller ett objekt fra en bestemt dag, uten å rulle tilbake andres arbeid. Et helt systembilde kan bestå testen for full gjenoppretting og likevel feile her.

Lag null råtner stille også, med et utløpt token, en andre faktor knyttet til noen som har sluttet eller en forseglet nøkkel fra før siste rotasjon. Prøven avslører dette, og det som overrasket deg, føres inn i veiledningen.

## Etter et innbrudd

Etter et innbrudd gjenoppretter du ikke hele systembilder, for da får du inntrengeren med. Bygg systemene på nytt fra konfigurasjon og kode, gjenopprett bare data fra et tidspunkt før innbruddet, og behandle dem som mistenkelige. Det forutsetter at en ren versjon fortsatt finnes, og siden innbrudd ofte oppdages sent, hører de med blant problemene oppbevaringen må vare lenger enn (se over).

## Automatisering og overvåking

Manuelle sikkerhetskopier blir før eller siden glemt, i en travel uke eller når en person har sluttet. Automatiser tidsplanen, kontrollen og oppryddingen etter oppbevaringstiden.

Automatikk svikter også stille, og et varsel om feil er ikke nok, for en jobb som aldri starter, sender ingen feilmelding. Varsle på fravær av suksess. Forvent én vellykket kjøring av hver planlagte jobb, både kopieringen og kontrollen, per periode, og slå alarm når en uteblir. Se også på alderen og størrelsen til den nyeste kopien. Et plutselig utslag begge veier er et signal, for et fall kan bety at data mangler, og vekst kan bety at data er kryptert av andre enn deg, noe som verken lar seg deduplisere eller komprimere, så inkrementene vokser brått. Dette er pulsen fra [Hvem vokter vokterne? Tilsyn i IT-infrastruktur](/blogg/hvem-vokter-vokterne-i-it-infrastruktur/). En kontroll må ikke dele skjebne med det den overvåker, og stillhet må regnes som et funn.

## Sjekkliste

- Vi har skrevet ned RPO og RTO og sortert dataene i uerstattelige, reproduserbare og midlertidige.
- For hver hendelse vi må overleve, vet vi hvor mange kopier som er igjen etterpå, og tallet gir den marginen vi har bestemt.
- Det finnes minst tre kopier, originalen medregnet, på minst to ulike slags lagring, og minst én ligger et annet sted. Ingen enkelt leverandør, avtale eller betalingskonto har alle kopiene.
- Ingen enkelthendelse (en feil, en konto, en maskin eller en person) kan slette alle kopiene. Produksjonen kan ikke slette eller overskrive dem, og best er at den ikke har tilgang i det hele tatt.
- Minst én kopi er frakoblet eller uforanderlig, med begrensede skriverettigheter og en grense for hvor mye én kjøring, eller én periode, kan legge til.
- Vi har flere versjoner, og oppbevaringen er lengre enn tiden det tar å oppdage et problem.
- Arkivet kontrolleres jevnlig for integritet, og minst én kopi lages direkte fra kilden inn i et separat arkiv, ikke replikert fra samme arkiv.
- Alle kopier er kryptert med nøkler vi selv holder, og nøkkelen er delt eller oppbevart på minst to steder som ikke deler skjebne.
- Kopieringen og kontrollen kjører automatisk, og vi får alarm når en vellykket kjøring av en av dem uteblir.
- Konfigurasjon og hemmeligheter er med i sikkerhetskopien. Gjenopprettingsveiledningen, nøkkelen, verktøyet til å lese formatet og tilgangen til lagringen ligger utenfor, atskilt og lesbart uten den, og rekkefølgen har ingen sykluser.
- Kopiene er konsistente, med relaterte systemer tatt på ett tidspunkt eller i en rekkefølge der ingenting peker på et manglende objekt.
- Vi har gjenopprettet fra grunnen, også granulært, tatt tiden og sammenlignet med RTO, og gjentar det på fast tid.

## En historie til slutt

Noen fortalte meg en gang om en jobb vedkommende hadde gjort for en kvinne hvis mann var død. Alle bildene av ham, fra ferier og turer de to hadde gjort sammen, lå på én harddisk, og harddisken sviktet.

Det tok halvannet år å få dataene tilbake. Det innebar å finne en donordisk, en identisk disk å ta deler fra, og å jobbe med å berge dataene. Vedkommende kjente henne og tok derfor ikke betalt.

Vedkommende fortalte at hun gråt da hun fikk dem tilbake. Alle bildene av mannen hennes, alt var berget.
