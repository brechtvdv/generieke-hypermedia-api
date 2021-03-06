Discussie: [#6](https://github.com/pietercolpaert/generieke-hypermedia-api/issues/6)

# CRUD operaties

CRUD of Create, Read, Update & Delete zijn de vier basisoperaties die op resources kunnen worden uitgevoerd.
* Create: aanmaken van een nieuwe resource
* Read: opvragen van gegevens over een specifieke resource
* Update: wijzigen van gegevens over een specifieke resource
* Delete: verwijderen van een resource

Deze sectie beschrijft een abstract algoritme dat een generieke client toelaat deze basis operaties te herkennen die toegelaten zijn op een bepaalde resource.

## Nakomingsniveaus

### HTTP

Een server _MOET_ ondersteuning bieden voor één of meerdere HTTP verbs. Welke HTTP verbs precies ondersteund worden voor een bepaalde resource dient meegegeven te worden in de HTTP 'Allow' header.

Daarnaast _MOET_ een API HTTP verbs gebruiken in lijn met [RFC7231](https://tools.ietf.org/html/rfc7231) en in het [RFC5789](https://tools.ietf.org/html/rfc5789) voor de PATCH methode.

Onderstaande tabel geeft aan welke HTTP verbs _ZOUDEN_ ondersteund kunnen worden door een API (één of meerdere afhankelijk van de functionaliteit aangeboden door de API). De operatie idempotentie _MOET_ voldaan worden. 

Methode  | Omschrijving  | Is idempotent | CRUD operatie
-------- | ------------- | ------------- | -------------
GET     | Geef de huidige waarde van een object. | True | Read
PUT     | Vervang een object of, wanneer van toepassing, creëer een nieuw object. | True | Update
DELETE  | Delete een object | True | Delete
POST    | Creëer een nieuw object gebaseerd op de data zoals voorzien in de request body, of voer een commando uit. | False | Create
HEAD    | Geef de header van een object voor een GET response terug. Resources die de GET methode ondersteunen _MOGEN_ ook de HEAD methode ondersteunen. | True | Read
PATCH   | Voer een gedeeltelijke update uit op een object. | False | Update

### Semantisch

Een server die ondersteuning wil bieden voor Linked Data _MOET_ bijkomend volgende specificatie ondersteunen:

+ [Hydra Core Vocabulary](http://www.hydra-cg.com/spec/latest/core/) 'Resources' en 'Operations'

Ook hier geldt dat het gebruik van HTTP verbs in lijn moet zijn met [RFC7231](https://tools.ietf.org/html/rfc7231).

## Code voorbeelden

### Linked Data Platform

De toegelaten operaties worden meegegeven in de HTTP `Allow` response header.

```
GET /api/resource/1 HTTP/1.1
Host: example.org
Accept: application/ld+json
```

```
HTTP/1.1 200 OK 
Content-Type: application/ld+json; charset=UTF-8
Link: <http://www.w3.org/ns/ldp#Resource>; rel="type"
Allow: GET,PUT,DELETE
```

### Hydra

De toegelaten operaties worden meegegeven in de response body via de eigenschap [hydra:operation](http://www.w3.org/ns/hydra/core#operation). Verder kan, waar relevant, de verwachte input worden meegegevens via de eigenschap [hydra:expects](http://www.w3.org/ns/hydra/core#expects). Ten slotte kan via de eigenschap [hydra:returns](http://www.w3.org/ns/hydra/core#returns) aangeduid worden welke output teruggegeven wordt. 

```json
{
  "@context": "http://www.w3.org/ns/hydra/context.jsonld",
  "@id": "/api/resource/1",
  "title": "Een voorbeeld resource",
  "description": "Deze resource kan verwijderd worden met een HTTP DELETE request of aangepast worden met een HTTP PUT request",
  "operation": [
    {
      "@type": "Operation",
      "method": "GET",
      "returns": "schema:Event"
    },
    {
      "@type": "Operation",
      "method": "PUT",
      "expects": "schema:Event"
    }
  ]
}
```

De informatie die verwacht wordt als input (`hydra:expects`) of output (`hydra:returns`) door de API kan verder gespecifieerd worden door gebruik te maken van de [Shapes Constraint Language (SHACL)](https://www.w3.org/TR/shacl/):

```json
{
  "@context": {
    "hydra": "http://www.w3.org/ns/hydra/context.jsonld",
    "sh": "http://www.w3.org/ns/shacl#",
    "schema": "https://schema.org/"
  },
  "@id": "/api/resource/1",
  "operation": [
    {
      "@type": "Operation",
      "method": "GET",
      "returns": "http://example.org/EventShape"
    },
    {
      "@type": "Operation",
      "method": "PUT",
      "expects": {
        "@id": "http://example.org/EventShape",
        "sh:targetClass": "schema:Event",
        "sh:property": [
          {
             "sh:name": "eventName",
             "sh:description": "Een naam voor het event",
             "sh:path": "schema:eventName",
             "sh:minCount": "1",
             "sh:maxCount": "1"             
          }
        ]
      }
    }
  ]
}
```

## Algoritme voor herkennen van crud operaties

Onderstaande reeks stappen definieert een algoritme die door een generieke client kan worden toegepast om CRUD operaties te herkennen.

Namespace prefix rdfs: http://www.w3.org/1999/02/22-rdf-syntax-ns#
Namespace prefix hydra: http://www.w3.org/ns/hydra/core#

1. Als de ALLOW header van een Resource response een GET, PUT, DELETE, POST, HEAD en/of PATCH methode bevat, voorzie in een generieke afhandeling in lijn met [RFC7231](https://tools.ietf.org/html/rfc7231). 
2. Anders, als de response header `application/ld+json` als Content-Type bevat, gebruik het [JSON-LD 1.1 processing algoritme](https://json-ld.org/spec/FCGS/json-ld-api/20180607/#expansion-algorithms) om de response om te zetten in zijn geëxpandeerde vorm.
3. Als de response een object bevat met als attribuut `hydra:operation`, gebruik het corresponderende `hydra:Operation` object om te voorzien in een generieke afhandeling in lijn met [RFC7231](https://tools.ietf.org/html/rfc7231).
4. Als de `hydra:Operation` een attribuut `hydra:expects` of `hydra:returns` bevat, voer indien nodig een `GET` operatie uit op de corresponderende resource om de lijst met ondersteunde attributen (`sh:property`) voor een create of update operatie te bekomen. Gebruik deze response om eventueel een dynamisch input formulier op te stellen. Verwijs naar de verwachte vorm (zogenoemde `SHACL shape`) van een applicatie profiel om aan te duiden dat de response hiermee in lijn is. 

## Herbruikbare library

* [generic-hypermedia-api-client](https://github.com/Informatievlaanderen/generic-hypermedia-api-client)
