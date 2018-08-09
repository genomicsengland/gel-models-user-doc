## Summary Finding

Summary finding report is defined by the `ClinicalReport` schema. It contains administrative information about the 
Interpretation Request `interpretationRequestId` and `interpretationRequestVersion` used to generated the results. It 
also contains information about user, creation date and reference databases and software information used to collect 
the findings, list of variants: [small variants](../variants/small-variants.md), 
[short tandem repeats](../variants/short-tandem-repeat-variants.md), [structural variants](../variants/structural-variants.md) and 
[chromosomal rearrangements](). 
Finally, it contains a summary of the interpretation statement field called `genomicInterpretation`.


!!! Tip "Negative Summary Findings"

    None of the variant lists are required to produce a summary of finding report. If none of them is present it will be
    assumed a negative result. Here an example:
    
```json

{
 "genomicInterpretation": "Negative report. No relevant findings.", 
 "referenceDatabasesVersions": {
  "db1": "v1"
 }, 
 "reportingDate": "2018-01-01", 
 "interpretationRequestVersion": 1, 
 "softwareVersions": {
  "s1": "v1"
 }, 
 "user": "user1", 
 "interpretationRequestId": "1"
}
```

### Additional Panels

If any additional panel has been applied (from those used originally by the Interpretation Services), they can be added
into the report using the field `additionalAnalysisPanels`:
```json
{
 "genomicInterpretation": "Negative report. No relevant findings.", 
 "referenceDatabasesVersions": {
  "db1": "v1"
 }, 
 "reportingDate": "2018-01-01", 
 "interpretationRequestVersion": 1, 
 "softwareVersions": {
  "s1": "v1"
 }, 
 "user": "user1", 
 "additionalAnalysisPanels": [
  {
   "specificDisease": "d1", 
   "panel": {
    "source": "panel-db", 
    "panelVersion": "v2", 
    "panelName": "p1"
   }
  }
 ], 
 "interpretationRequestId": "1"
}

```

### Guideline Based Variant Classification

Although this is a property of the variants ([ReportEvent](../variants/small-variants.md#reportevent)) is explained here as this is especially relevant for manually
classified variants. At the moment the models support for both ACMG and AMP classification.

#### ACMG

Based on the [American College of Medical Genetics and Genomics and the Association for Molecular Pathology guideline](https://www.acmg.net/docs/standards_guidelines_for_the_interpretation_of_sequence_variants.pdf)
The model is form by a list of evidences associated to the variant `AcmgEvidence`, an overall clinical Significance 
`clinicalSignificance` and a final assessment for the classification (free text). `AcmgEvidence` is built according
with the ACMG guideline:

* `category` (i.e, population_data, computational_and_predictive_data...) 
* `type` (i.e, bening, pathogenic) 
* `weight` (i.e, stand_alone, supporting, moderate..)
* `modifier` (the number defining each criteria in the table)

With these properties any criterion defined by the guideline can be defined, here an example for `PS3`: Well-established 
functional studies show a deleterious effect.

```json
{
 "category": "computational_and_predictive_data", 
 "modifier": 3, 
 "type": "pathogenic", 
 "description": "PS3 Well-established functional studies show a deleterious effect"
}
```

And this is how it will be as part of a full classification:

```json
{
 "clinicalSignificance": "pathogenic", 
 "acmgEvidences": [
  {
   "category": "computational_and_predictive_data", 
   "modifier": 3, 
   "type": "pathogenic", 
   "weight": "strong", 
   "description": "PS3 Well-established functional studies show a deleterious effect"
  }
 ], 
 "assessment": "This is a pathogenic variant"
}
```
