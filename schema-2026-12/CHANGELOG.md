# ATLAS JSON Schema Changelog: 2025-11 → 2026-12

This document describes what changed between the 2025-11 schemas (repository root:
[`model.json`](../model.json), [`dataset.json`](../dataset.json)) and the 2026-12 schemas in this
folder ([`model.json`](model.json), [`dataset.json`](dataset.json)). The 2026-12 version is a
substantial revision: it adds several new sections, redesigns model performance reporting from the
ground up, introduces three conformance profiles, and tightens validation rigor throughout. It
remains aligned with the [RSNA/ROADMAP](https://github.com/RSNA/ROADMAP) ontology — every field that
corresponds to a ROADMAP concept carries a `$comment: "ROADMAP:NNNNN"` pointer to it, and this
version was checked against ROADMAP's current content (`owl:versionInfo "2026-11"`) as part of its
release.

## New model schema sections

| Added | Description |
|---|---|
| `Model.Datasets` | Reciprocal links from a model to the datasets used in its development, disaggregated by `Role` (pretraining, training, fine-tuning, optimization, validation, tuning, testing, internal/external testing, hold-out, development, test). Mirrors `Dataset.Models` on the dataset side. |
| `Model.Factors` | Card-level declaration of which axes of subgroup disaggregation (e.g. age, sex, site) the model's performance results are examined across. |
| `Model-properties.Integration` | How the model is integrated into an imaging environment: `Interfaces` (DICOM network services, DICOMweb REST services, HL7 v2 message types, FHIR resources, IHE Radiology profiles), `Result delivery` channels, `Invocation` triggers, `Conformance statements`, `System types`, and `Validated systems`. |
| `Model-properties.Model class` | Coarse, queryable architectural/algorithmic category (e.g. CNN, transformer, foundation model, ensemble). |
| `Model-properties.Foundation model` | The pretrained foundation model this model is built upon or fine-tuned from, if any. |
| `Model-properties.Third-party components` | Third-party or pre-trained components incorporated into the model, other than a declared foundation model. |
| `Model-properties.Maintenance` | How the model's real-world performance and fairness are monitored post-deployment, and the process by which it is updated in response (monitoring program, validity/fairness monitoring, update process). |
| `Model-properties.Bias mitigation approach` | How bias was identified, managed, or reduced during development. |
| `Model-properties.Clinical risk level` | A declared level of clinical risk associated with use of the model. |
| `Model-properties.Cybersecurity measures` | Security posture (e.g. access controls). |
| `Model-properties.Distribution form` | The form(s) in which the model is placed on the market or put into service. |
| `Model-properties.Computational resources` | Processor/memory/runtime requirements. |
| `Model-properties.User interface` | Basic description of the deployer-facing UI. |
| `Model-properties.Acceptance test procedure` | Recommended local verification procedure for a receiving site. |
| `Model-properties.Operating range` | Declared expected-performance range or minimum-performance floor. |
| `Model-properties.Suspend-use conditions` | Conditions under which use of the model should be suspended. |

`Model-properties.EHR compatibility` (a flat enumerated list of EHR vendor names) was removed; the
much more rigorous `Integration` section above is the closest conceptual successor for describing a
model's interoperability characteristics, though it is standards-based (DICOM/HL7/FHIR/IHE) rather
than vendor-named.

## New dataset schema sections

| Added | Description |
|---|---|
| `Dataset.Models` | Reciprocal links from a dataset to the models known to use it, disaggregated by `Role` — the mirror image of `Model.Datasets` above. |

## Model performance: redesigned from the ground up

The 2025-11 schema's `Model.Model performance` was a thin `$ref` to a `Metrics` object: a flat array
of metric *names* (scalar, matrix, or graphical, all in one enumerated list) plus a single free-text
field for reporting their values. 2026-12 replaces this with a structured, repeatable
`Model-performance` array, one entry per evaluation:

- **`Evaluation dataset`**, **`Provenance`** (claimed vs. observed), **`Test type`** (internal,
  external, local, prospective), **`Testing agent`**, and **`Metric justification`** describe the
  evaluation itself.
- **`Results`** is an array of index-test/metric/value tuples, each one of:
  - a **scalar-metric** result (`Scalar-metric` enum of 232 terms, a numeric `Value`, and an optional
    `Confidence interval`),
  - an **outcome-count** result (`Outcome-counts`: TP/FP/TN/FN/P/N/PP/PN/Total), or
  - a **non-scalar-metric** result (`Nonscalar-metric` enum — confusion matrices, ROC/FROC/AFROC
    curves, calibration curves, Bland-Altman analysis, etc. — with a string `Value`, e.g. a markdown
    table or a link to an external graph).
  - Each result can carry a **`Subgroup`** (`{Variable, Value}`), tying it back to one of the model's
    declared `Factors`.

This closes a real gap in the old design: metric *names* and their *values* were previously
disconnected (one array of enumerated names, one free-text field for all values combined), making it
impossible to machine-read "this model's AUC was 0.91." The new structure also distinguishes claimed
from independently observed performance, and internal from external/prospective testing — needed for
any procurement or regulatory use of the card.

## Vocabulary unification: dataset Role / Partition

2025-11 had no notion of a model-dataset relationship vocabulary. 2026-12 introduces one and then
unifies it everywhere it appears: `Model.Datasets[].Role`, `Dataset.Models[].Role`, and
`Dataset.Partitions[].Partition` all now share the identical 13-value enum (Pretraining, Training,
Fine-tuning, Optimization, Validation, Tuning, Testing, Internal testing, External testing, Hold-out,
Development, Test, Other), all pointing to the same ROADMAP class (`ROADMAP:03763`, "dataset role").
Several other fields were converted from free-text "examples only" guidance to true enums against a
ROADMAP-backed controlled vocabulary during this cycle as well, including authorization
`Authority`/`Pathway`/`Status`, EU AI Act `Risk classification`/`Technical documentation`, `Output`
`Type`, performance `Provenance`, and `Distribution form`.

## New: conformance profiles

2025-11 shipped a single schema with no notion of tiers. 2026-12 adds three conformance profiles under
[`profiles/`](profiles/), each a thin `allOf` overlay over the base model schema that narrows
`required`/cardinality without adding new properties:

- **[`minimal.json`](profiles/minimal.json)** — identical to the base schema; only `Name` and `Link`
  are required. Exists so all three tiers are addressable at a stable, dated URL.
- **[`procurement-ready.json`](profiles/procurement-ready.json)** — what a buyer needs to evaluate
  fit: identity/provenance, imaging modality, intended use/users, input/output/architecture/
  limitations/availability/instructions, a declared clinical risk level, at least one performance
  record with a declared `Test type`, at least one `Datasets` entry, and at least one
  regulatory-authorization record (an explicit "Not submitted" is an acceptable answer for a research
  model).
- **[`regulatory-ready.json`](profiles/regulatory-ready.json)** — cumulative on procurement-ready;
  additionally requires a declared bias-mitigation approach, cybersecurity measures, predetermined
  change control, and a `Maintenance` object, and requires that the `Datasets`/`Model performance`
  arrays actually demonstrate what they claim: at least one `Datasets` entry with `Role: "Training"`
  and at least one performance record with `Test type: "External"`.

## Validation rigor

A systematic review added floors that the 2025-11 schema lacked throughout:

- **`minLength` ≥ 1 on every non-enumerated string field**, so an empty string can no longer
  vacuously satisfy a `required` check.
- **`minItems`/`minProperties`** on every array and object that lacked them, for the same reason.
- **Fixed bugs found during the review:**
  - `Indexing.SNOMED` applied `maxLength` to an `integer`-typed field — silently ignored by every
    validator, leaving codes effectively unbounded. Replaced with a string/integer `anyOf` where the
    string form (canonical going forward) is length- and pattern-constrained.
  - `Subset.Age range` applied one flat numeric ceiling regardless of unit, so `[240, "years"]`
    validated despite being physically nonsensical. Replaced with per-unit `if`/`then` bounds (days,
    months, years).
  - An `additionalProperties: false` sitting alongside a `oneOf`/`anyOf` with no sibling `properties`
    key rejects *every* property regardless of which branch would otherwise match — found and fixed
    in two places (`Model-performance-results`, `Coded-data`).
  - A schema's own embedded `examples` value failed its own validation (`Partitions`' example used a
    `Partition` value not present in the then-current enum).
- **Every embedded `examples` array in both files is now validated against its own local schema** as
  a standing check (79 nodes across both files as of this writing).
- `Date` tightened from `anyOf` to `oneOf` (a value cannot simultaneously be both a date-string and an
  integer year) and its integer-year range narrowed from unbounded to 1900–2100. `Org`/`Person` name
  fields raised their floor from no minimum to `minLength: 2`. `Person.ORCID`, `Indexing.RadLex`, and
  `Imaging.Procedures.RadLex` patterns were anchored (`^...$`) where they previously were not,
  closing a hole where a partially-matching string would pass.

## ROADMAP alignment

Both schemas were cross-checked against RSNA/ROADMAP's current content as part of this release
(`owl:versionInfo "2026-11"`, highest assigned entity ID `04234`). Result: zero dangling
`ROADMAP:NNNNN` references in either file, and two concrete drifts found and fixed — see this
repository's commit history for the "Resync schemas against current RSNA/ROADMAP ontology content"
commit for details (a missing pair of scalar metrics, and a vocabulary asymmetry between
`Dataset.Models[].Role` and its reciprocal field). Two smaller items were identified as ROADMAP-side
follow-ups rather than schema changes: the Role/Partition vocabulary's "Hold-out" and "Development"
values have no corresponding ROADMAP class, and ROADMAP's `Output.Type` individuals lack the short
`altLabel`s their sibling enumerations have.
