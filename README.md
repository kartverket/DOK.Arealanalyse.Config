# Konfigurasjonsfiler for analyser

Denne katalogen (/config) inneholder YAML-filer som konfigurerer datasett og analyser brukt i DOK Arealanalyse Demonstrator.

Hver .yml-fil representerer en spesifikk type datasett eller analysetema, som f.eks:

- Faresoner (skred, flom, steinsprang)

- Naturtyper (utvalgte naturtyper som kalksjøer, slåttemark osv.)

- Reindrift (flyttleier, anlegg)

- Kulturminner

- Infrastruktur (farleder, tilfluktsrom)

- Geotekniske data (borehull)


## Struktur på en konfigfil

En typisk konfigurasjonsfil inneholder informasjon som:

### Id: 
Unik id for konfigurasjonen, bruk uuid.
```yml
type: dataset
config_id: c74656bb-267d-4546-aa14-6323070d06c7
```
### Metadata-id: 
Identifikator for datasett i geonorge. Brukes for å hente metadata om datasett fra geonorge og lage lenker som https://kartkatalog.geonorge.no/metadata/64cbb884-a19d-4356-a114-380cfe4a7314 
```yml
metadata_id: 64cbb884-a19d-4356-a114-380cfe4a7314
```
### Navn:
Kort navn på datasettet, brukes i logger el.l for evt feilsøking. Normalt brukes navn fra metadata i geonorge.
```yml
name: naturtyper_ku_verdi
```
### Datakilde:
Hvilken tjeneste eller API dataene hentes fra samt objekttyper/lag og geometrifelt som det søkes i. Tjenesten må støtte intersect.
```yml
arcgis: https://kart.miljodirektoratet.no/arcgis/rest/services/naturtyper_kuverdi/MapServer
wms: https://kart.miljodirektoratet.no/arcgis/services/naturtyper_kuverdi/MapServer/WMSServer
layers:
- arcgis: '0'
  wms:
  - kuverdi_svært_stor_verdi
  result_status: HIT-RED
```

```yml
ogc_api: https://api.ra.no/LokaliteterEnkeltminnerOgSikringssoner/collections
wms: https://kart.ra.no/wms/kulturminner
layers:
- ogc_api: enkeltminner
  wms:
  - Enkeltminner
  result_status: HIT-RED

geom_field: område
```

```yml
wfs: https://wfs.geonorge.no/skwms1/wfs.stormflo_havniva
wms: https://wms.geonorge.no/skwms1/wms.stormflo_havniva
layers:
- wfs: Stormflo20År_KlimaÅrNå
  wms:
  - stormflo20ar_klimaarna
  result_status: HIT-RED

geom_field: område
```

### Visningsfelter:
Hvilke felter som hentes ut til data visningen, feks navn eller url til mer informasjon som ligger på objekter en treffer i analysen.
```yml
properties:
- Naturtype
- Verdikategori
- Opphav
- Nøyaktighetsklasse
- Faktaark
```
### Filter:
Evt. filterbetingelser (for å velge ut spesifikke objekter)
```yml
layers:
  ...
  filter: Verdikategori = 'Svært stor verdi'
```
```yml
layers:
  ...
  filter: UtvalgtNaturtypeKode = 'UN03'
```
```yml
layers:
  ...
  filter: vernetype = 'AUT' AND enkeltminnekategori IN ('E-ARK', 'E-BER', 'E-MAR', 'E-RUI')
```
### Tema: 
Hvilke DOK tema hører datasettet til - https://register.geonorge.no/metadata-kodelister/nasjonal-temainndeling
```yml
themes:
- natur
```
```yml
themes:
- kulturminner
```
```yml
themes:
- samfunnssikkerhet
```
### Veiledningstekster: 
Det angis referanse til ID på aktuell veiledningstekst fra https://register.geonorge.no/geolett/. guid som vist i eksempel viser til oppslag på denne id i API for Veiledningsregister https://register.geonorge.no/geolett/api/. guid 9d803106-57f6-494e-bca8-67f1fd1f72a3 vil hente veiledningstekst fra Naturtypelokalitet, svært stor verdi - https://register.geonorge.no/geolett/1071/ 
- planning_guidance_id er id til veiledningstekst for bruksområdet plan
- building_guidance_id er id til veiledningstekst for bruksområdet bygg 

```yml
layers:
- arcgis: '0'

  result_status: HIT-RED
  planning_guidance_id: 9d803106-57f6-494e-bca8-67f1fd1f72a3
  building_guidance_id: 522da012-1633-40c5-ad96-e4784d278401
```

### Kvalitetsinformasjon:
Det er flere dimmensjoner på kvalitet som kan vises og det er lagt opp til noen felles slik som egnethet fra DOK som ligger under common.yaml. Kvalitet kan hentes fra datasett nivå eller fra hvert enkelt objekt.
property og warning_threshold brukes for å få fram advarsler som henter tekst fra quality_warning_text.
Underlag og inndeling av [kvalitetsinformasjon](https://standarder.geonorge.no/sosi/standarder-geografisk-informasjon/geodatakvalitet/1.0/geodatakvalitet-10-standarder-geografisk-informasjon.pdf) i kategoriene fullstendighet, egenskapskvalitet, logisk konsistens, Kvalitet på tidfesting, stedfestingsnøyaktighet og egnethet.

```yml
---

type: quality
config_id: c74656bb-267d-4546-aa14-6323070d06c7

indicators:
- type: object
  quality_dimension_id: stedfestingsnøyaktighet
  quality_dimension_name: Stedfestingsnøyaktighet
  quality_warning_text: Stedfestingsnøyaktighet er usikker på et eller flere objekter
  property: Nøyaktighetsklasse
  warning_threshold: Mindre god (> 50m)
```

```yml
- type: coverage
  quality_dimension_id: fullstendighet_dekning
  quality_dimension_name: Fullstendighetsdekning
  quality_warning_text: Området er ikke kartlagt for naturtyper
  warning_threshold: null
  arcgis:
    url: https://kart.miljodirektoratet.no/arcgis/rest/services/naturtyper_nin/MapServer
    layer: '1'
    property: Dekningskartverdi
    planning_guidance_id: 0685b79a-76a8-439a-8793-6dc31a0a9099
    building_guidance_id: 45ece8ff-e382-4c40-b469-045e2939f554
    properties:
    - Prosjektnavn
    - Dekningskartverdi
    - Årstall
    - Kartleggingsinstruks
    - Prosjektrapport
```

```yml
- type: coverage
  quality_dimension_id: fullstendighet_dekning
  quality_dimension_name: Fullstendighetsdekning
  quality_warning_text: Området er ikke kartlagt for flomsoner
  warning_threshold: Ikke kartlagt
  gpkg:
    url: file:///mnt/dokanalyse/geopackage/dekningskart_nve_flomsoner.gpkg
    property: dekningsstatus
    properties:
    - dataeier
    - dekningsstatus
    - dekningsinfo
```

```yml
- type: coverage
  quality_dimension_id: fullstendighet_dekning
  quality_dimension_name: Fullstendighetsdekning
  quality_warning_text: Området er ikke kartlagt for kvikkleireskred
  warning_threshold: Ikke kartlagt
  geojson:
    url: file:///mnt/dokanalyse/geojson/dekningskart_nve_aktsomhetsomr_kvikkleireskred.geojson
    property: dekningsstatus
    properties:
    - dataeier
    - dekningsstatus
    - dekningsinfo
```

```yml
- type: dataset
  quality_dimension_id: egnethet_byggesak
  quality_dimension_name: Byggesak
  quality_warning_text: Datasettet er lite egnet for byggesak
  warning_threshold: 1 OR 2
  input_filter: context = 'Byggesak'
```

### Deaktivering av enkelte konfigurasjoner: 
De kan være påbegynt men ikke komplette eller inneholde feil i tjenester som gjør at en ønsker å deaktivere de for en kortere eller lengre periode uten at de slettes.
```yml
name: referanseruter
disabled: true
```
