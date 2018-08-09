# Gel Models User Documentation

## Variant Interpretation Results Models

Gel Models have 2 schemas to define results for variant interpretation process:

* [Interpreted Genome](): This one is produced by interpretation services and contains variants that has been 
prioritised based on known rules, applied in a consistent way. They are usually algorithms which prioritised variants
in a automatic way.
* [Summary of Finding](summary-finding/summary-finding.md): (Historically called clinical report) This one is produced
 by experts and contains only variants relevant for the case, ideally all of the variants on it has been manually 
 classified.

## Variants in Gel Models

These models support 4 types of variants all of them are part of the `CommonInterpreted` protocol:

* [SmallVariant](variants/small-variants.md)
* [ShortTandemRepeat](variants/short-tandem-repeat-variants.md)
* `StructuralVariant`
* `ChromosomalRearrangement`
 
In this guide we will explain how to use them, in different scenarios and corner cases. The 4 models shared 
different properties, which are defined outside them so they can be reused. For example, each one of these 4
entities have `VariantCall` and `reportEvents`, and they follow the same schema.