## SmallVariant

A small variant will define a SNV or a small insertion/deletion (although there is no a limit defined for 
the size of the small indels, it is highly recommended to use the `StructuralVariant` models if the size is 
greater than 200). `Smallvariant` is defined using the following inner schemas:

 * `VariantCoordinates`
 * `VariantCall` (list)
 * `ReportEvent` (list)
 * `VariantAttributes`
 
As said before, the models will reuse this schemas to build other types of variants, in this section will 
explain the general usage of them and in the following ones the particular cases for each type of variant.

### VariantCoordinates
It is a very simple model which contains the coordinates of the variants, the models are thought to be 
compatible with VCF format, so it will be expected that the coordinates match what it would appear in a VCF.


```json
/** VFC: chr1    10177   rs201752861     A       C */

{
 "position": 10177, 
 "alternate": "C", 
 "assembly": "GRCh38", 
 "reference": "A", 
 "chromosome": "chr1"
}

```


### VariantCall
VariantCall schema represents the information for samples information about the variant (this is the 9th 
onward of the VCF file). Is expected that at the level of the SmallVariant we got the information about
all the members in that case, even if the variant is missing in that participant.


In the following example it is represented one of the VCF columns (for sample `S1` belonging to `P1`).

```json

/** VCF: ...GT:GQ:GQX:DP:DPF:AD:PL:PS  1|0:3:8:1:0:20,18:46,3,0:10 */

{
 "zygosity": "heterozygous", 
 "participantId": "P1", 
 "numberOfCopies": null, 
 "sampleId": "S1", 
 "depthReference": 20, 
 "depthAlternate": 18, 
 "alleleOrigins": [
  "germline_variant"
 ], 
 "phaseGenotype": {
  "sortedAlleles": [
   "1", 
   "0"
  ], 
  "phaseSet": 10
 }, 
 "supportingReadTypes": null, 
 "sampleVariantAlleleFrequency": null
}


```
!!! tip "Tip"

    `numberOfCopies`, `supportingReadTypes` and `sampleVariantAlleleFrequency` are null, this is
    because they are field that are relevant only to other type of variants. It will be explained later in this
    document.
    


### ReportEvent

ReportEvent represent data associated to the variant added by an algorithm or a human being. Report events can 
be defined as a the set of properties (variant and/or case specific) that taken together provide a reason to 
report that variant. There are several consideration that need to be taken when working with reportEvents:

1. A report event is created for one variant in one case, this means that any information may not be relevant
in other case (even if the variant is actually the same one), and all the information in report event 
should represent the whole case, this is, if the case contains 3 samples, the information in report event 
it is associated to the 3 of them and not to a subset of them.

2. Each report event has a `reportEventId` this is an `id` for the report event inside the document, 
 i.e, InterpretedGenome or ClinicalReport, and it only makes sense inside it. In other words, it should 
 be unique at the document level, but it doesn't have to be  unique across any system.

3. Almost everything in the schema is optional, this is because given the nature of the different 
interpretation process or analysis types, some of the fields are not relevant. The only thing that always
should be present is phenotypes, genomicEntities and modeOfInheritance* (which are explained below).

4. Every property of a report event should contribute be part of the same explanation. 

5. There are variants that are reported together, i.e compound heterozygous, to do this they are required to be
reported in 2 different smallVariant, but they should be linked by `groupOfVariants`. `groupOfVariants` is a integer value
that can be used to identify those variants that provides an explanation when, and only when, they are together. This 
linked is done at the report event level, given the user the possibility to create a many-to-many relationships between
variants and report events. For example 2 pair of variants in a compound heterozygous are reported based on 2 panels,
in this case each variant will have 2 report event each one of those will a group `groupOfVariants` but shared with the 
other 2 report events of the other variant.

!!! note "Example of a Compound het"

    Given 2 variants which should be reported by a given interpretation service as a compound heteozygous, here the
    report events for each one of those.

``` json

/** report event for the first variant*/
{
  "fullyExplainsPhenotype": false, 
  "groupOfVariants": 1001, 
  "genePanel": {
   "source": "panelApp", 
   "panelVersion": "1.0", 
   "panelName": "P1"
  }, 
  "genomicEntities": [
   {
    "geneSymbol": "G1", 
    "type": "gene", 
    "ensemblId": "ESNG00001"
   }, 
   {
    "geneSymbol": "G1TX1", 
    "type": "transcript", 
    "ensemblId": "ESNT00001"
   }
  ], 
  "phenotypes": {
   "nonStandardPhenotype": [
    "D1"
   ]
  }, 
  "penetrance": "complete", 
  "reportEventId": "RE1", 
  "variantConsequences": [
   {
    "id": "SO:0001587", 
    "name": "stop_gained"
   }
  ], 
  "tier": "TIER1", 
  "modeOfInheritance": "biallelic",
  "segregationPattern": "CompoundHeterozygous" 
 }
 
 /** report event for the second variant*/
 {
  "fullyExplainsPhenotype": false, 
  "groupOfVariants": 1001, 
  "genePanel": {
   "source": "panelApp", 
   "panelVersion": "1.0", 
   "panelName": "P1"
  }, 
  "genomicEntities": [
   {
    "geneSymbol": "G1", 
    "type": "gene", 
    "ensemblId": "ESNG00001"
   }
  ], 
  "phenotypes": {
   "nonStandardPhenotype": [
    "D1"
   ]
  }, 
  "penetrance": "complete", 
  "reportEventId": "RE2", 
  "variantConsequences": [
   {
    "id": "SO:0001583", 
    "name": "missense_variant"
   }
  ], 
  "tier": "TIER2", 
  "modeOfInheritance": "biallelic",
  "segregationPattern": "CompoundHeterozygous"
 }

```



!!! note "Example 2 disorders + 2 genes"

    For a given unigenerational family formed by father, mother, son 
    (affected by disorder D1) and daughter (affected by disorder D2). A variant present in both parents (het) and in both 
    children (alt_hom) overlap with 2 genes (G1, G2), G1 has 2 transcripts (G1TX1 and G1TX2) being G1TX2 a long noncoding 
    transcript and G2 has 1 transcript (G2TX1). A very smart Interpretation Service has detected that this variant
    should be reported because, G1 is associated to D1 by gene panel P1, and the variant introduce a stop codon in the 
    G1T1X, but not only that, even though G2 is not known to be associated to D1 or D2, the interpretation service still
    considers that it is a good candidate in G2 to explain D1 and D2, given that the variant is producing a change of 
    one aa in the G2 product.
    
    Now, we got 1 variant in 2 genes that can actually explained 2 disorders in one single family. In this case, the 
    interpretation service should report this variant ONCE (one single `smallVariant`) with 4 report event on it. 
    In the following code block it can be appreciated how this should done.
    
``` json

[
 {
  "fullyExplainsPhenotype": true, 
  "groupOfVariants": 1, 
  "genePanel": {
   "source": "panelApp", 
   "panelVersion": "1.0", 
   "panelName": "P1"
  }, 
  "genomicEntities": [
   {
    "geneSymbol": "G1", 
    "type": "gene", 
    "ensemblId": "ESNG00001"
   }, 
   {
    "geneSymbol": "G1TX1", 
    "type": "transcript", 
    "ensemblId": "ESNT00001"
   }
  ], 
  "phenotypes": {
   "nonStandardPhenotype": [
    "D1"
   ]
  }, 
  "penetrance": "complete", 
  "reportEventId": "RE1", 
  "variantConsequences": [
   {
    "id": "SO:0001587", 
    "name": "stop_gained"
   }
  ], 
  "tier": "TIER1", 
  "modeOfInheritance": "biallelic", 
  "score": 10000
 }, 
 {
  "fullyExplainsPhenotype": true, 
  "groupOfVariants": 2, 
  "genePanel": {
   "source": "panelApp", 
   "panelVersion": "1.0", 
   "panelName": "P1"
  }, 
  "genomicEntities": [
   {
    "geneSymbol": "G2", 
    "type": "gene", 
    "ensemblId": "ESNG00002"
   }, 
   {
    "geneSymbol": "G2TX1", 
    "type": "transcript", 
    "ensemblId": "ESNT00003"
   }
  ], 
  "phenotypes": {
   "nonStandardPhenotype": [
    "D1"
   ]
  }, 
  "penetrance": "complete", 
  "reportEventId": "RE2", 
  "variantConsequences": [
   {
    "id": "SO:0001583", 
    "name": "missense_variant"
   }
  ], 
  "tier": "TIER3", 
  "modeOfInheritance": "biallelic", 
  "score": 100
 }, 
 {
  "fullyExplainsPhenotype": true, 
  "groupOfVariants": 4, 
  "genePanel": {
   "source": "panelApp", 
   "panelVersion": "1.2", 
   "panelName": "P2"
  }, 
  "genomicEntities": [
   {
    "geneSymbol": "G2", 
    "type": "gene", 
    "ensemblId": "ESNG00002"
   }, 
   {
    "geneSymbol": "G2TX1", 
    "type": "transcript", 
    "ensemblId": "ESNT00003"
   }
  ], 
  "phenotypes": {
   "nonStandardPhenotype": [
    "D2"
   ]
  }, 
  "penetrance": "complete", 
  "reportEventId": "RE4", 
  "variantConsequences": [
   {
    "id": "SO:0001583", 
    "name": "missense_variant"
   }
  ], 
  "tier": "TIER3", 
  "modeOfInheritance": "biallelic", 
  "score": 10
 }, 
 {
  "fullyExplainsPhenotype": true, 
  "groupOfVariants": 3, 
  "genePanel": {
   "source": "panelApp", 
   "panelVersion": "1.2", 
   "panelName": "P2"
  }, 
  "genomicEntities": [
   {
    "geneSymbol": "G1", 
    "type": "gene", 
    "ensemblId": "ESNG00001"
   }, 
   {
    "geneSymbol": "G1TX1", 
    "type": "transcript", 
    "ensemblId": "ESNT00001"
   }
  ], 
  "phenotypes": {
   "nonStandardPhenotype": [
    "D2"
   ]
  }, 
  "penetrance": "complete", 
  "reportEventId": "RE3", 
  "variantConsequences": [
   {
    "id": "SO:0001587", 
    "name": "stop_gained"
   }
  ], 
  "tier": "TIER3", 
  "modeOfInheritance": "biallelic", 
  "score": 200
 }
]

```

 
### variantAttributes

Variant attributes contains variant annotation relevant for interpretation. There isn't any required field in this 
schema but it should be populated with all the available information. Here an example of a fully filled variantAttribute:

``` json

{
 "additionalTextualVariantAnnotations": null, 
 "genomicChanges": [
  "NC_000011.9:g.111959695G>T"
 ], 
 "fdp50": 0.1, 
 "alleleOrigins": [
  "germline_variant"
 ], 
 "ihp": 0, 
 "variantIdentifiers": {
  "cosmicIds": null, 
  "clinVarIds": [
   "6897"
  ], 
  "otherIds": [
   {
    "source": "uniprot", 
    "identifier": "VAR_010039"
   }, 
   {
    "source": "OMIM", 
    "identifier": "602690.0004"
   }
  ], 
  "dbSnpId": "rs80338845"
 }, 
 "recurrentlyReported": false, 
 "comments": null, 
 "cdnaChanges": [
  "NG_012337.1(TIMM8B_v001):c.-2203C>A", 
  "NG_012337.1(SDHD_v001):c.274G>T"
 ], 
 "references": [
  "29925701"
 ], 
 "alleleFrequencies": [
  {
   "study": "1KGenomeProject", 
   "alternateFrequency": 0.003, 
   "population": "EAS"
  }
 ], 
 "others": null, 
 "additionalNumericVariantAnnotations": null, 
 "proteinChanges": [
  "NG_012337.1(TIMM8B_i001):p.(=)", 
  "NG_012337.1(SDHD_i001):p.(Asp92Tyr)"
 ]
}

```
