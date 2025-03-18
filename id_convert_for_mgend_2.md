# convert to MedGen Concept ID for MGeND 2

* TogoID でできないものを暫定的に
  * ICD10 -(Orphanet)-> Orphanet -(MedGen)-> MedGen
    * <a href="/rest/id_convert_for_mgend">Mondo 経由 ver.</a>
  * MedGen UID -(MedGen)-> MedGen
  * SNOMED -(MedGen)-> MedGen

## Parameters

* `source`
  * default: icd10
  * example: icd10, medgen_uid, snomed
* `ids`
  * default: A85.8,C02.9,C03.9,C05.9,C08.9,C10.9,C15.0,C15.2,C15.3,C15.4,C15.5,C15.9,C16.0,C16.1,C16.2,C16.3,C16.9,C17.0,C17.1,C17.2,C17.9,C18.0,C18.1,C18.2,C18.4,C18.6,C18.7,C18.8,C18.9,C19,C20,C21.1,C22.0,C22.1,C22.2,C23,C24.0,C24.1,C25.0,C25.1,C25.2,C25.3,C25.9,C26.9,C34.1,C34.2,C34.9,C35.9,C37,C38.0,C38.3,C40.0,C40.2,C41.1,C41.9,C43.9,C44.4,C44.9,C47.9,C48.0,C48.2,C49.2,C49.3,C49.9,C50.9,C52,C53.9,C54.1,C54.9,C56,C57.0,C61,C62.9,C64,C66,C67.9,C69.2,C71.2,C71.9,C73,C74.0,C74.9,C75.5,C78.0,C78.7,C78.8,C79.3,C79.5,C79.6,C80.0,C80.9,C81.9,C85.9,C90.0,C91.0,C91.5,C92.0,C96.4,D05.1,D05.9,D12.4,D12.5,D12.6,D13.1,D13.2,D13.5,D13.6,D13.9,D14.3,D16.4,D16.9,D17.7,D18.0,D21.9,D22.9,D27,D32.9,D33.0,D33.4,D34,D35.0,D35.1,D35.2,D35.5,D36.1,D36.9,D37.1,D37.5,D37.6,D37.7,D38.0,D38.3,D39.1,D43.2,D43.3,D43.4,D44.0,D44.3,D44.5,D44.7,D48,D48.0,D48.1,D48.6,D48.7,E04.2,E04.9,E21.3,E23.1,E78.0,E78.1,E78.3,E78.6,F90.0,G40.4,G93.4,I42.0,I44.3,I42.2,I49.8,I45.8,I48.9,I49.5,I49.0,K31.7,K56.1,K63.5,K63.8,K74.3,K75.3,K82.8,K86.1,K86.2,L72.0,M31.1,N64.9,N87.9,Q65.8,Q85.0,Q85.8,Q87.8,R94.3,D46.9

## `set`
```javascript
({source, ids}) => {
  let prefix;
  let pre = "";
  let post = "";
  let icd = false;
  let uid = false;
  let sno = false;
  if (source == "icd10") {
    prefix = "ICD-10:";
    pre = "'";
    post = "'";
    icd = true;
  } else if (source == "medgen_uid") {
    prefix = "uid:";
    uid = true;
  } else if(source == "snomed") {
    prefix = "snomed:"
    sno = true;
  }
  return {
    ids: pre + prefix + ids.replace(/ /g, "").split(/,/).join(post + " " + pre + prefix) + post,
    icd: icd,
    uid: uid,
    sno: sno
  };
}
```

## Endpoint

https://rdfportal.org/bioportal/sparql


## `icd_orpha`
```sparql
PREFIX oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?icd ?orpha
FROM <http://rdf.integbio.jp/dataset/bioportal/ordo>
WHERE {
  {{#if set.icd}}
  VALUES ?icd10 { {{set.ids}} }
  ?orpha oboInOwl:hasDbXref ?icd10 .
  FILTER (REGEX (?icd10, "ICD-10:"))
  BIND (strafter(str(?icd10), 'ICD-10:') AS ?icd)
  {{/if}}
}
```

## `orpha_value`
```javascript
({set, icd_orpha}) => {
  if (set.icd) {
    return  icd_orpha.results.bindings.map(d => d.orpha.value.replace("http://www.orpha.net/ORDO/Orphanet_", "orpha:")).join(" ");
  }
  return 0;
}
```


## Endpoint

https://rdfportal.org/ncbi/sparql

## `orpha_medgen`
```sparql
PREFIX orpha: <http://www.orpha.net/ORDO/Orphanet_>
PREFIX mo: <http://med2rdf/ontology/medgen#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?orpha ?medgen
FROM <http://rdfportal.org/dataset/medgen>
WHERE {
  {{#if set.icd}}
  VALUES ?orpha { {{orpha_value}} }
  ?medgen_uri
    dct:identifier ?medgen ;
    mo:mgconso [
      dct:source mo:ORDO ;
      rdfs:seeAlso ?orpha
  ] .
 {{/if}}
}
```

## `uid_medgen`
```sparql
PREFIX uid: <http://www.ncbi.nlm.nih.gov/medgen/>
PREFIX mo: <http://med2rdf/ontology/medgen#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?uid ?medgen
FROM <http://rdfportal.org/dataset/medgen>
WHERE {
  {{#if set.uid}}
  VALUES ?uid_uri { {{set.ids}} }
  ?medgen_uri
    dct:identifier ?medgen .
  ?uid_uri rdfs:seeAlso ?medgen_uri .
  BIND (strafter(str(?uid_uri),  "http://www.ncbi.nlm.nih.gov/medgen/") AS ?uid)
 {{/if}}
}
```

## `sno_medgen`
```sparql
PREFIX snomed: <http://purl.bioontology.org/ontology/SNOMEDCT/>
PREFIX mo: <http://med2rdf/ontology/medgen#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?snomed ?medgen
FROM <http://rdfportal.org/dataset/medgen>
WHERE {
  {{#if set.sno}}
  VALUES ?snomed_uri { {{set.ids}} }
  ?medgen_uri
    dct:identifier ?medgen ;
    mo:mgconso [
      dct:source mo:SNOMEDCT_US ;
      rdfs:seeAlso ?snomed_uri
  ] .
  BIND (strafter(str(?snomed_uri), "http://purl.bioontology.org/ontology/SNOMEDCT/") AS ?snomed)
 {{/if}}
}
```

## `returen`
```javascript
({set, icd_orpha, orpha_medgen, uid_medgen, sno_medgen}) => {
   if (set.uid) {
     return uid_medgen.results.bindings.map(d => { return {source: d.uid.value, target: d.medgen.value} } );
   }
   if (set.sno) {
     return sno_medgen.results.bindings.map(d => { return {source: d.snomed.value, target: d.medgen.value} } );
   }
   let res = [];
   for (const d1  of icd_orpha.results.bindings) {
     for (const d2 of orpha_medgen.results.bindings) {
         if (d1.orpha.value == d2.orpha.value) {
           res.push({ source:  d1.icd.value, target:  d2.medgen.value});
         }
      }
   }
   return res;
}
```
