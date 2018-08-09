## ShortTandemRepeat

A ShortTandemRepeat will define a STR amplification, Short Tandem repeat alteration is used taken as reference the data
recorded in `ShortTandemRepeatReferenceData`. `ShortTandemRepeatReferenceData` is defined using the following 
inner schemas:

* `Assembly` (enumeration)
* `Coordinates`
* `ReportEvent` (list)
* `VariantCall` (list)
* `VariantAttributes`
* `ShortTandemRepeatReferenceData`
 
This section covers all of the items that are specially relevant for short tandem repeat.   

### Coordinates
Coordinates represent the position of the region of the STR in the reference genome.

```json
/**
chr1    57367043        .       C       <STR8>,<STR14>,<STR2>   .       PASS    END=57367118
 */

{
  "chromosome": "1",
  "start": 57367043,
  "end": 57367118
}

```

### VariantCall
Variant call for STRs follow the same model than any other variant, although in this case the most relevant field is
numberOfCopies

```json

/** ...GT:SO:REPCN:REPCI:ADSP:ADFL:ADIR        1/2:SPANNING/FLANKING:8/14:6-10/12-16:16/6:18/19:0/0*/

{
 "numberOfCopies": [
  {
   "confidenceIntervalMinimum": 6, 
   "numberOfCopies": 8, 
   "confidenceIntervalMaximum": 10
  }, 
  {
   "confidenceIntervalMinimum": 12, 
   "numberOfCopies": 14, 
   "confidenceIntervalMaximum": 16
  }
 ], 
 "supportingReadTypes": [
  "spanning", 
  "flanking"
 ], 
 "zygosity": "na", 
 "participantId": "p1", 
 "sampleId": "s1"
}

```