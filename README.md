# Moduler og gjenbruk i CSS

**Skrevet av Vegar Norman @ Making Waves, 2016**


## Problemet

Frontend-utvikling har kommet langt på kort tid. Teknologiene vi bruker har modnet og måten vi skriver kode på har endret seg. Vi har fantastiske verktøy for deling og gjenbruk av kode gjennom [NPM](http://npmjs.org), og vi skriver moduler i JavaScript som vi ville skrevet klasser i objektorienterte språk som Java eller C#. Når vi jobber med JavaScript-baserte prosjekter er det lett å finne det vi trenger i NPM. Trenger du et server-rammeverk til et API? Express kan lastes ned og integreres i prosjektet på kort tid. Hva med enhetstesting? Mocha, Jasmine og Tape er tilgjengelige. Problemer med cross-origin requests? `npm install cors --save`, så er du all set.

Men av en eller annen grunn skriver vi fortsatt CSS som om vi befinner oss på starten av 2000-tallet. Vi laster fortsatt ned Bootstrap i minifisert format og trekker det inn i prosjektene våre gjennom en `<style>`, og vi gjør den samme jobben om igjen og om igjen når vi starter nye prosjekter. Vi skriver (grovt sett) de samme reglene og klassedefinisjonene, forsøker å håndtere (grovt sett) de samme problemene vi støter på, og går automatisk ut fra at siden vi jobber med et nytt prosjekt med et nytt design og grensesnitt, må vi starte på nytt. *Do it right*, rett og slett.

Hvorfor klarer vi ikke å overføre de samme prinsippene vi bruker i andre programmeringsspråk til CSS? Hvorfor skriver vi CSS-regler på nytt mange ganger når både vi selv og andre har skrevet dem før? Hvorfor løser vi de samme utfordringene og problemene om igjen og om igjen? Hva om det å skrive CSS var som å skrive JavaScript, og vi hadde NPM? Om jeg i et prosjekt kunne avgjøre hvilke "pakker" jeg hadde behov for, og deretter laste dem ned og ta dem i bruk med det samme, kunne jeg ha spart mye tid. Spaltesystem i flexbox og floats for eldre nettlesere? Check. En pakke som gjør det lett å lage pene skjemaer? Check. Sidenavigasjon som elegant brekker om når brukeren er på mobil? Check.

Problemet ligger ikke i teknologien vi bruker - vi har preprosessorer som gir oss tilgang på funksjoner, løkker, valgsetninger og variabler. Vi har byggverktøy som håndterer linting, testing, prefixing og minifisering for oss. Vi har alt vi kunne forvente av et hvilket som helst annet programmeringsspråk, så problemet ligger ikke i teknologien.

*Problemet ligger i at vi ofte ikke er gode nok til å skrive CSS for gjenbruk.*


## Modularisering

### Gjenbrukbarhet vs. organisering

Modularisering er et kjent begrep for utviklere, uavhengig av språk. Fremfor å skrive en enkelt, lang programfil når vi lager applikasjoner, deler vi opp koden i flere mindre filer (moduler) og bruker disse. Dette er et konsept vi ofte også bruker når vi skriver CSS, enten vi bruker `@import` i vanlig CSS, eller bruker en preprosessor som kompilerer til en enkelt CSS-fil for oss. Problemet er at vi sjeldent skriver moduler for *gjenbrukbarhet*, men heller for *organisering av prosjektet*.

```
.navigation {
  width: 100%;
  background-color: $light-grey;
  padding: 10px 0;

  nav {
    width: $container-width;
    margin: 0 auto;

    ul {
      list-style: none;

      li {
        display: inline-block;
        text-align: center;

        a {
          color: $black;

          &:hover {
            color: $blue;
          }

          &#current {
            color: $blue !important;
          }
        }
      }
    }
  }
}
```

Et eksempel kan være når vi lager en webapplikasjon og skiller ut navigasjonen i en egen fil i Sass kalt `_navigation.scss`. Vi legger gjerne klassene våre for navigasjonen i denne filen, beskriver hvordan den skal se ut og oppføre seg, kanskje ved å bruke variabler for å hente inn riktige fargekoder og navn på fonter. Vi har laget en modul - men vi har ikke laget en modul som egentlig kan brukes igjen i et senere prosjekt.

+ Vi har brukt klassenavn og ID'er som kanskje ikke finnes i HTML-strukturen
+ Vi har brukt variabler som kanskje ikke eksisterer
+ Vi har gått ut fra en HTML-struktur som kanskje ikke lar seg bruke
+ Vi bruker hardkodede verdier som kanskje ikke er korrekte

Når en modul skal skrives for gjenbrukbarhet, bør vi ikke basere koden på antakelser eller forutsetninger. I enkelte tilfeller kan dette være vanskelig, men som en grunnregel bør gjenbrukbare moduler være fullstendig isolerte fra prosjektet de finnes i.


### Hva er en god modul?

En god modul bør møte et enkelt sett med kriterier:

+ Ha "zero impact" (ikke generere CSS-regler uten å bli bedt om det)
+ Være godt dokumenterte (for eksempel gjennom [SassDoc](http://www.sassdoc.com))
+ Være testbare (gjerne gjennom enhetstester)
+ Kunne konfigureres og hackes
+ Ikke basere seg på forutsetninger eller antakelser om prosjektet
+ Oppfordre til gjenbruk og DRY
+ Kunne brukes uten å generere CSS-regler
+ Ikke gå utenfor sitt eget scope (fokus på problemet modulen skal løse fremfor f.eks browserstøtte eller a11y)

Det handler med andre ord om å skrive CSS med en *programmatisk* tilnærming, på samme måte som vi ville ha skrevet Java eller JavaScript. Men, CSS er et veldig statisk språk av natur, og gjør det vanskelig å følge en del av disse kriteriene som for eksempel konfigurerbarhet. Vi tar derfor utgangspunkt i at modulene skrives ved hjelp av en preprosessor som [Sass](http://www.sass-lang.com) eller [LESS](http://www.lesscss.org). Preprosessorer gir oss tilgang til svært viktige verktøy som variabler og funksjoner.

I dette tilfellet velger jeg å ta utgangspunkt i Sass, ettersom Sass later til å ha størst spredning i fagmiljøet.


### Mixins vs. placeholders

For at vi skal kunne lage CSS-output uten å generere CSS-regler i den ferdig kompilerte CSS-filen, er vi avhengige av et verktøy som lar oss gjøre dette "on demand". De to mest naturlige valgene å se på er *mixins* og *placeholders*. Av de to fremstår mixins som det naturlige valget, fordi...

+ Mixins kan ta i mot parametere og generere ulik output basert på parameterene
+ Placeholders ikke kan brukes i media queries, noe som dermed gjør dem ubrukelige i responsivt design

Ved å skrive moduler ved hjelp av mixins fremfor placeholders eller vanlige CSS-regler oppnår vi ønsket resultat: modulen er tilgjengelig gjennom `@include`-regler og kan brukes uten å måtte generere unødvendige CSS-regler.


### Konfigurasjon

Moduler bør konfigureres gjennom variabler i sine egne konfigurasjonsfiler, slik at dette er separat fra variabler som brukes i prosjektet. På denne måten kan utviklere som bruker modulen selv avgjøre hvorvidt de ønsker å blande disse variablene eller holde dem separate, avhengig av deres behov i prosjektet. I tillegg kan vi legge spesielle funksjoner modulen er avhengig av, som for eksempel spesielle mattefunksjoner som ikke er tilgjengelige direkte i Sass, i en egen fil.


### Filstruktur

En modul kan struktureres på følgende måte, selv om det naturligvis finnes grunner til å gjøre det annerledes. For eksempelets skyld tar vi her utgangspunktet i en modul for sidenavigasjon kalt `navigation`.

```
navigation (folder)
↳ _navigation.scss
↳ _navigation-config.scss
↳ _navigation-functions.scss
```

Denne filstrukturen går ut fra at vi importerer `_navigation.scss` inn i prosjektet, og at de andre filene importeres inn i modulen direkte.


## Men... Hvorfor?

Dette virker kanskje som en veldig komplisert måte å skrive CSS på, og vil garantert ikke passe for alle utviklere i alle prosjekter, akkurat som at funksjonell programmering ikke alltid vil passe, og at React ikke alltid vil passe. Allikevel er det klare fordeler med å skrive gjenbrukbare moduler:

+ **Moduler er betraktelig lettere å vedlikeholde og oppgradere ved behov.** Fremfor å maintaine 5000 linjer med CSS i et prosjekt, kan man heller maintaine mindre moduler og utbedre disse når feil oppdages. I tillegg er det lettere for forfatteren å fokusere på at koden skal gjøre bestemte ting veldig bra, fremfor mange ting sånn passe bra.
+ **Det er lettere å holde koden levende.** God kode, uansett språk, er kode som lever og puster. Kode som konstant forbedres, refaktoreres og lekes med. Mens vi svært ofte jobber aktivt med å forbedre funksjonaliteten i webapplikasjoner, skrives CSS ofte kun én gang og tas bare frem igjen hvis noe er galt. Dette er sjeldent, om ikke aldri, en god måte å skrive kode på. Hørt om utvikleren som skrev en AngularJS-app som ikke trengte å oppdateres på to år? Ikke jeg heller.
+ **Tettere tilknytning til programmeringsprinsipper.** De fleste utviklere er glade i å skrive modulær kode fordi den gir bedre oversikt, er lett å skrive enhetstester for, og tillater innkapsling av logikk på en enkel måte. Ved å skrive gjenbrukbare moduler, åpner vi for nettopp disse prinsippene.


## Videre arbeid

Det er selvfølgelig mye å ta tak i rundt gjenbrukbarhet og modularisering av CSS-kode, og mye handler om hvordan dette kan fungere i praksis. Hvordan kan vi garantere at alle skriver moduler på samme måte? Hvordan kan vi best inkludere disse i et faktisk prosjekt? Hvordan kan vi best garantere langsiktig arbeid med dette? Og viktigst av alt: er dette egentlig en god måte å skrive moduler på? Kan vi gjøre det bedre?

Vil du komme med innspill? Legg gjerne igjen en kommentar i [Issues](https://github.com/vegarnorman/css/issues), både ris og ros mottas med takk! Send gjerne også innspill på e-post, i Slack eller ved frontend-bordene oppe i fjerde etasje.
