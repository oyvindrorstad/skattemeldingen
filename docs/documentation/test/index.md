---
icon: "cloud"
title: "Testmiljø og -klienter"
description: ""
---

# Testplan

Formålet med testen er å verifisere uthenting av skattemelding for en skattepliktig og bruk av valideringstjenesten hos Skatteetaten.
I tillegg legger vi opp til muligheten til å instansiere opp en instans for skattemelding på den nye Altinn 3 plattformen.
Det skal også være tilgjengelig funksjonalitet for å laste opp filer med skattemelding og næringsopplysninger på instansen i Altinn.

# Innhold i testen

Konkret må sluttbrukersystemet gjøre følgende:

1. Få etablert et kundeforhold hos Digitaliseringsdirektoratet (DigDir) og ta i bruk ID-porten.
2. Logge inn hos ID-porten for å få en sesjon (og token).
3. Utvikle en applikasjon/klient som:
   - Sende en request til Skatteetatens tjeneste for å hente ut skattemeldingen for en skattepliktig.
   - Sende en request til Skatteetatens tjeneste for å validere skattemelding med næringsinformasjon for en skattepliktig
   - Motta og tolke resultatet fra valideringstjenesten.
   - Når det ikke er feil i retur fra valideringstjenesten kan man gå til punkt E
   - Kalle Altinn sin nye plattform for å instansiere opp App'en for innsending av skattemelding for næringsdrivende.
   - Kalle Altinn for å laste opp skattemelding og næringsopplysninger på den instansierte App'en fra pkt e.

Skatteetaten har tilgjengeliggjort en test-applikasjon som viser hvordan trinnene beskrevet over kan utføres:
[skattemelding-eksternt-api-test.zip](../api/skattemelding-eksternt-api-test.zip)

## Ta i bruk ID-porten

Det første som en konsument av skattemeldings-API må gjøre er å etablere et kundeforhold hos Digitaliseringsdirektoratet (DigDir). Detaljer rundt hvordan ta i bruk ID-porten er beskrevet her: https://samarbeid.difi.no/felleslosninger/id-porten/ta-i-bruk-id-porten. Denne prosessen er delvis manuell så det er lurt å starte prosessen tidlig slik at dere kan komme i gang med testingen. Ved bestilling må dere oppgi at dere ønsker tilgang til skattemeldings-API fra skatteetaten.

Et kundeforhold hos DigDir gir tilgang til deres selvbetjeningsløsning som igjen gir tilgang til administrasjon av Kundens bruk av ID-porten. I selvbetjeningsløsning kan kunden genere et såkalt client_id og definere et callback-url:

- client_id: er unik automatisk generert identifikator for tjenesten.
- callback-url: Uri-en som klienten får lov å gå til etter innlogging. Etter en vellykket innlogging i ID-porten vil denne url-en bli kalt.
  Dersom det skulle ta for lang tid å få opprettet et kundeforhold hos DigDir kan sluttbrukersystemene i mellomtiden benytte Skatteetatens client_id. For denne testen har Skatteetaten opprettet følgende client_id som kan benyttes av sluttbrukersystemene:

      - `client_id: 38e634d9-5682-44ae-9b60-db636efe3156`
      - Callback-URL til denne client_id er satt til http://localhost:12345/token (Hvis det er konsumenter som ønsker andre callback-URL kan vi ordne det)

**Nyttige lenker:**

- Klienten bruker test-miljøet i DigDir, "verifikasjon 2": https://samarbeid.difi.no/node/232
- OIDC-integrasjonen er beskrevet her: https://difi.github.io/felleslosninger/oidc_index.html
- Hvordan lage en klient i selvregistreringen: https://minside-samarbeid.difi.no/organization-home/services/service-admin#/

## Logge inn hos ID-porten

Klienten må foreta følgende REST kall mot ID-porten:

- Starte system browser og gjøre autorisasjonskall mot ID-porten. Les mer om det her: https://difi.github.io/felleslosninger/oidc_protocol_authorize.html
- Brukeren blir da sendt til ID-porten for innlogging. Dere kan benytte eksisterende test-brukere som dere benytter til test mot skattemeldingen i dag.
- Sette opp en webserver som venter på callback fra browseren. Etter vellykket pålogging i ID-porten sendes brukes til denne webserveren. Denne webserveren må være satt til å lytte på callback-URL http://localhost:12345/token (ref. forrige kapittel).
- Gjøre et tokenforespørsel. Les mer om det her: https://difi.github.io/felleslosninger/oidc_protocol_token.html

Vi benytter følgende testmiljø hos ID-porten:

- /authorize endpoint: https://oidc-ver2.difi.no/idporten-oidc-provider/authorize
- /token endpoint: https://oidc-ver2.difi.no/idporten-oidc-provider/token

For detaljer rundt hvilken HTTP parametere som må sendes med i kallet, se filen hent.py: [skattemelding-eksternt-api-test.zip](../api/skattemelding-eksternt-api-test.zip)

## Kalle skattemeldings-API

Når callback-URL blir kalt må klienten plukke ut JWT-tokenet fra responsen og legge det i header-feltet Authorization og kalle skattemeldings-API. For detaljer, se filen hent.py: [skattemelding-eksternt-api-test.zip](../api/skattemelding-eksternt-api-test.zip)

URL til skattemeldings-API i test er: https://mp-test.sits.no/

Første testen bør være å teste at klienten når frem, dette kan gjøres ved å kalle ping tjenesten:

- `GET https://<env>/api/eksterntapi/formueinntekt/skattemelding/ping`
- `Eksempel: GET https://mp-test.sits.no/api/eksterntapi/formueinntekt/skattemelding/ping`

For nærmere beskrivelse av skattemelding-API og Altinn3-API, se kapitelet [API](../api/index.md)

# Se skattemelding i portal

URL til portal: https://skatt-ref.sits.no/web/skattemeldingen/2020

Bruk MinID innlogging; fnr på testbruker, password01 og Pinkode 12345.

Merk at dere også kan endre/tilføye opplysninger og sende inn skattemelding. Fastsatt skattemelding (endret skattemelding) vil da også være tilgjengelig for nedlasting. Se kapittel 4.3.6.3 i implementasjonsguide.

# Tidsplan, henvendelser og tilbakemelding på testresultater

Tidsplan, henvendelser og tilbakemelding på testresultater
Testmiljøet vil være tilgjengelig fra 10.6. Det er ikke bestemt sluttdato for gjennomføring, men vi ønsker at flest mulig får gjennomført testen så raskt det lar seg gjøre.

Tekniske spørsmål knyttet til forberedelser og gjennomføring kan rettes til nyskattemelding@skatteetaten.no

Etter gjennomført test ønsker vi at dere sender en testrapport på resultatet av gjennomført test. Den kan dere sende til samme adresse som over. I testrapporten beskriver dere hvordan dere har gjennomført test (om dere har brukt testklienten eller egenutviklet kode), resultatet og eventuelle kommentarer.
