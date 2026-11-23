# gUFO SHACL Shapes

The objective of gUFO is to provide a lightweight implementation of the Unified Foundational Ontology (UFO) suitable for Semantic Web OWL 2 DL applications.

See http://purl.org/nemo/doc/gufo for usage guide and complete documentation of gUFO.

This project (gUFO SHACL Shapes) provides constraints in SHACL/SPARQL to check for correct usage of the gUFO taxonomy of types, and other aspects of well-formedness that cannot be established in OWL directly.

Authors:

- João Paulo A. Almeida
- Claudenir M. Fonseca
- Ítalo Oliveira

## Where the shapes come from

The taxonomy and higher-order shapes come from listings 24–30 in Section 7 of [gUFO: A Gentle Foundational Ontology for Semantic Web Knowledge Graphs](https://arxiv.org/html/2603.20948v1). Listings 24–28 are SHACL constraints on `rdfs:subClassOf`: rigidity, non-sortals, identity for sortals, Kinds as ultimate sortals, and endurant types not specializing events, situations, or other non-endurants. Listings 29–30 are SPARQL shapes for `gufo:categorizes` and `gufo:partitions`. These seven shapes are unchanged, except `KindShape`.

The chain-wide checks come from listings 12–18 of Guizzardi, Fonseca, Almeida, Sales, Benevides, and Porello, [Types and taxonomic structures in conceptual modeling](https://doi.org/10.1016/j.datak.2021.101891), *Data & Knowledge Engineering* 134 (2021). There they are SPARQL queries that return violating classes. Here each query is a `sh:SPARQLConstraint`: the violating class is `$this`, and the other class, when there is one, is `?value`. Listings 13 and 15–18 follow the printed queries. They repeat the direct rigidity and sortality checks over the whole `rdfs:subClassOf` chain, and add the rules that a sortal specializes at most one Kind and that every non-sortal is specialized by a sortal.

Three checks do not copy the source literally:

- `KindShape` follows the repository, which treats another `gufo:Kind` as a forbidden direct superclass. Listing 27 of the arXiv paper omits `gufo:Kind` from that list. The check is SPARQL and ignores a class declared as a subclass of itself, which OWL-RL adds for every class.
- The listing 14 shape uses the same path as the paper (`rdfs:subClassOf` then `rdf:type gufo:Kind`) and ignores that same reflexive triple.
- The listing 12b shape flags a type that specializes two of `Object`, `Relator`, `Quality`, `ExtrinsicMode`, and `IntrinsicMode`, transitively. The query printed in the paper uses `FILTER (?n > 0)` on a single direct subclass step, which selects every class that specializes at least one of those five.

A domain class is in scope when it is declared as an instance of a gUFO type stereotype (`:Person a gufo:Kind`) and specialized with `rdfs:subClassOf`. Some shapes look only at the direct superclass. The shapes named below as walking the chain follow `rdfs:subClassOf` through intermediate classes. A class counted as a subclass of itself is not a violation.

## Rigidity

A rigid type applies to each of its instances for as long as the instance exists. An anti-rigid type applies only contingently. A semi-rigid type is necessary for some instances and contingent for others.

| Shape | Violation |
|---|---|
| `gufoshapes:RigidOrSemiRigidShape` | A rigid or semi-rigid type (`RigidType`, `Category`, `Kind`, `SubKind`, `SemiRigidType`, `Mixin`) has an anti-rigid direct superclass (`AntiRigidType`, `Phase`, `PhaseMixin`, `Role`, `RoleMixin`). |
| `gufoshapes:RigidAndSemiRigidCantSpecializeAntiRigidShape` | The same prohibition, over the whole specialization chain. |

## Sortals and identity

A sortal applies to instances that share one principle of identity. A Kind supplies that principle. A SubKind, Phase, or Role inherits it. A non-sortal applies to instances of different Kinds and must not carry an identity principle.

| Shape | Violation |
|---|---|
| `gufo:NonSortalShape` | A non-sortal (`NonSortal`, `Category`, `PhaseMixin`, `RoleMixin`, `Mixin`) has a sortal direct superclass (`Sortal`, `Kind`, `SubKind`, `Phase`, `Role`). |
| `gufoshapes:NonSortalCantSpecializeSortalShape` | The same prohibition, over the whole specialization chain. |
| `gufoshapes:NonSortalMustHaveSortalSpecializationShape` | A non-sortal is not specialized by any sortal, directly or through another non-sortal. |
| `gufoshapes:BaseSortalShape` | A `SubKind`, `Phase`, or `Role` has no direct superclass that is a `Kind`, `SubKind`, `Phase`, or `Role`. |
| `gufoshapes:SortalMustSpecializeUltimateSortalShape` | A `SubKind`, `Phase`, or `Role` does not reach a `gufo:Kind` through the specialization chain. |
| `gufoshapes:KindShape` | A `Kind` has a direct superclass that is a `Kind`, `SubKind`, `Phase`, or `Role`. |
| `gufoshapes:UltimateSortalCantSpecializeAnotherShape` | A `Kind` reaches another `gufo:Kind` through the specialization chain. |
| `gufoshapes:CantSpecializeMoreThanOneUltimateSortalShape` | A `SubKind`, `Phase`, or `Role` reaches more than one `gufo:Kind`. |

## One stereotype, one endurant category

| Shape | Violation |
|---|---|
| `gufoshapes:OnlyOneOntoumlStereotypeTypesShape` | A type carries more than one of `Category`, `RoleMixin`, `PhaseMixin`, `Mixin`, `Kind`, `SubKind`, `Role`, `Phase`. |
| `gufoshapes:OnlyOneOntoumlStereotypeEndurantsShape` | An endurant type specializes more than one of `Object`, `Relator`, `Quality`, `ExtrinsicMode`, `IntrinsicMode`. |
| `gufoshapes:EndurantTypeShape` | An endurant type specializes a class disjoint from endurants: an abstract individual, a quality value, a time instant, an event, a participation, or a situation. |

## Higher-order types

`gufo:categorizes` links a higher-order type to the base type its instances classify. `gufo:partitions` is a form of categorization in which those instances are pairwise disjoint. With OWL-RL, a partition is also checked by the categorization shape, because `gufo:partitions` is a subproperty of `gufo:categorizes`.

| Shape | Violation |
|---|---|
| `gufoshapes:CategorizationShape` | An instance of a type that `gufo:categorizes` a base type is not a subclass of that base type. |
| `gufoshapes:PartitionPairwiseDisjointnessShape` | Two instances of a type that `gufo:partitions` a base type are not declared disjoint (`owl:disjointWith`, `owl:AllDisjointClasses`, or `owl:disjointUnionOf`). |
