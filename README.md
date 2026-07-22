# Industriële monumenten en bodembeperkingen

Vervolganalyse op het stapelingsrapport van 6 juli 2026. Onderzoekt of rijksmonumenten met een bodemgerelateerde publiekrechtelijke beperking vaker een industriële functie hebben dan rijksmonumenten in het algemeen.

## Bestanden

- `industrieel_bodem.html` — het rapport zelf, een op zichzelf staand HTML-artefact met grafieken (Chart.js via cdnjs), een doorzoekbare tabel en de gebruikte SPARQL-query's. Werkt offline op de tabellen en tekst; de grafieken hebben internettoegang nodig voor Chart.js.
- `data.py` — de samengevoegde tellingen (bodem-subset, basislijn, gematchte functienaam-labels) die in het HTML-bestand zijn verwerkt.

## Vraagstelling

Zijn bodem-beperkingen oververtegenwoordigd bij industriële monumenttypen, ten opzichte van rijksmonumenten in het algemeen?

## Data en methode

**Bodem-subset**: 2.436 rijksmonumenten met minstens één beperking op basis van de Wet bodembescherming of de vergelijkbare Omgevingswet-nazorgbepalingen (grondslagcodes KW, WBI, WBD, WBC, WBA, WBE, WBB, OLD). Afkomstig uit het stapelingsrapport van 6 juli, dat op zijn beurt draait op de Kadaster Kennisgraaf.

**Basislijn**: alle 63.097 actieve rijksmonumenten (`ceo:heeftJuridischeStatus` = "rijksmonument") met een geregistreerde functienaam.

**Bron**: RCE CHO SPARQL-endpoint (`linkeddata.cultureelerfgoed.nl`), via `ceo:heeftOorspronkelijkeFunctie` / `ceo:heeftFunctieNaam` voor de functie en `ceo:heeftJuridischeStatus` voor de status.

**Classificatie industrieel**: eigen regex-classificatie op de functienaam, geen officiële RCE-typering. Twee varianten:

- **Smal** (kern-industrieel):
  ```
  fabriek|nijverheid|industrie|gemaal|molen|spoor|mijn|scheepv|werf|
  elektriciteitscentrale|gasfabriek|smederij|brouwerij|ketelhuis|
  machinegebouw|maalderij|hoogoven|pompstation|transformator|leerlooierij
  ```
- **Breed**: smal + `pakhuis|opslag|magazijn|werk-woonhuis`

Labels die overwogen maar bewust **niet** meegeteld zijn (matchen geen van beide lijsten): Branderij, Loods, Drukkerij, Silo, Pompgebouw, Remise, Zoutziederij, Zuiveringshuis, Locomotievenremise, Perrongebouw, IJzergieterij, Koperslagerij, Olieslagerij, Pellerij, Schoenmakerij, Goederenloods, Wasserij, Vernisstokerij. Zouden deze wel meetellen, dan wordt het effect eerder groter dan kleiner.

## Belangrijkste query

```sparql
PREFIX ceo: <https://linkeddata.cultureelerfgoed.nl/def/ceo#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT
  (COUNT(DISTINCT ?m) AS ?totaal)
  (COUNT(DISTINCT ?msmal) AS ?smal)
  (COUNT(DISTINCT ?mbreed) AS ?breed)
WHERE {
  # VALUES ?nr { ... } -- alleen voor de bodem-subset; weggelaten voor de basislijn
  ?m ceo:rijksmonumentnummer ?nr .
  ?m ceo:heeftJuridischeStatus ?status1 . ?status1 skos:prefLabel "rijksmonument" .
  ?m ceo:heeftOorspronkelijkeFunctie ?f . ?f ceo:heeftFunctieNaam ?fn . ?fn skos:prefLabel ?functienaam .
  BIND(IF(REGEX(STR(?functienaam), "fabriek|nijverheid|industrie|gemaal|molen|spoor|mijn|scheepv|werf|elektriciteitscentrale|gasfabriek|smederij|brouwerij|ketelhuis|machinegebouw|maalderij|hoogoven|pompstation|transformator|leerlooierij", "i"), ?m, ?UNBOUND1) AS ?msmal)
  BIND(IF(REGEX(STR(?functienaam), "fabriek|nijverheid|industrie|gemaal|molen|spoor|mijn|scheepv|werf|elektriciteitscentrale|gasfabriek|smederij|brouwerij|ketelhuis|machinegebouw|maalderij|hoogoven|pompstation|transformator|leerlooierij|pakhuis|opslag|magazijn|werk-woonhuis", "i"), ?m, ?UNBOUND2) AS ?mbreed)
}
```

Voor de bodem-subset is deze query in 3 batches van ~800 monumentnummers gedraaid (via `VALUES ?nr { ... }`) en de resultaten zijn opgeteld.

## Resultaten

| Groep | Totaal | Smal | Smal % | Breed | Breed % |
|---|---:|---:|---:|---:|---:|
| Bodem-subset | 2.436 | 222 | 9,1% | 393 | 16,1% |
| Alle rijksmonumenten | 63.097 | 2.279 | 3,6% | 6.906 | 11,0% |

Factor 2,5 (smal) resp. 1,5 (breed) oververtegenwoordiging. Twee-proporties z-toets: z = 13,9 (smal), z = 8,0 (breed) — beide ruim boven de gangbare significantiedrempel van 1,96.

## Kanttekeningen

- Eigen classificatie, geen RCE-standaard — vraagt een check door iemand die de functietypering kent.
- De basislijn is niet gecorrigeerd voor overlap met de bodem-subset.
- Regex-matching op vrije tekst mist een aantal plausibel-industriële labels (zie hierboven); dit werkt conservatief.
- Eén monument kan meerdere functienaam-labels hebben.
- Correlatie, geen aangetoonde causaliteit — zie met name de "Mijnwerkerswoningen"-nuance in het rapport zelf: de beperking kan aan de locatie hangen, niet aan het gebouw.

## Vervolgstappen

- Classificatie herhalen met de gemiste labels meegeteld.
- Basislijn opschonen (basislijn min bodem-subset) voor een zuiverdere vergelijking.
- Geografische in plaats van functionele koppeling onderzoeken voor de mijnbouw-nuance.

## Herkomst

Rijksdienst voor het Cultureel Erfgoed, juli 2026. Vervolg op *Stapeling van publiekrechtelijke beperkingen op rijksmonumentpercelen* (6 juli 2026).
