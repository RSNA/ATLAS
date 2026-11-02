# ATLAS Crosswalk: CHAI, Hugging Face, and ACR AI Documentation Standards

This document maps the ATLAS 2026-12 schemas (`model.json`, `dataset.json`) against four other
AI model/dataset documentation efforts: the Coalition for Health AI (CHAI)'s **Applied Model
Card**, **Hugging Face**'s Model Card and Dataset Card metadata, the **American College of
Radiology (ACR)**'s Assess-AI registry and AI Central directory, and the foundational **Google
Model Cards** framework (Mitchell et al., 2019) that both ATLAS's and CHAI's field vocabularies
descend from.

## Methodology and source confidence

Findings below come from three dedicated research passes (one per standard) plus the author's own
work on ATLAS. Network access in this environment blocked several primary domains
(`arxiv.org`, `research.google`, `chai.org`/`mc.chai.org`/`registry.chai.org`, `acr.org` and its
subdomains), so confidence varies by standard — stated explicitly so this document doesn't imply
more certainty than the sources support:

| Standard | Primary source actually read | Confidence |
|---|---|---|
| **CHAI Applied Model Card** | The published XML Schema (`.xsd`, v0.1 and v0.2) and a real example instance, via CHAI's GitHub mirror (`coalition-for-health-ai/mc-schema`) | High on field *names* and structure; the one-line descriptions below are inferred from element names and the one example read, since CHAI's own `DOCUMENTATION.md` has no per-field prose |
| **Hugging Face** | `modelcard.md`, `datasetcard.md`, and the Hub's model-card/dataset-card/eval-results guides, direct from the `huggingface/hub-docs` source repo | High — this is the Hub's own documented contract, confirmed field-by-field |
| **ACR (Assess-AI / AI Central)** | Secondary: ACR's own web pages and press releases, via search-result snippets (the actual Data Dictionary and Ingest Specification PDFs are gated/not directly fetchable here) | **Category-level only.** Program purpose and broad data categories are confirmed from ACR's own statements; exact field-level names are not confirmed — flagged per item below |
| **Google Model Cards** | Secondary sources corroborating the paper's structure (the original paper, `arXiv:1810.03993`, was not directly fetchable in this environment) | Section list is consistent across multiple independent secondary sources and matches the well-established secondary literature, but is not a direct quote from the paper |

Where a cell below says "not found" or "no analog," that means the researcher didn't find it in
the available material — not that it's proven absent from the standard.

## What each standard actually is

These four are not all the same *kind* of artifact, which matters for reading the tables below:

- **ATLAS** (`model.json`/`dataset.json`) — a formal **JSON Schema**, authored per model/dataset,
  with three conformance profiles (minimal, procurement-ready, regulatory-ready).
- **CHAI Applied Model Card** — a formal **XML Schema**, authored per deployed health-AI
  application (explicitly scoped to "the application in a health use case," not a bare model;
  CHAI recommends a revised card when the underlying system changes, e.g. swapping foundation
  models). Still self-labeled "draft" as of the version read.
  Source: [coalition-for-health-ai/mc-schema](https://github.com/coalition-for-health-ai/mc-schema).
- **Hugging Face Model/Dataset Cards** — YAML frontmatter in a repo's `README.md`, validated
  server-side at push time against a documented (but not formally schema'd) contract, plus a
  Mitchell-et-al.-based prose template for everything the YAML doesn't cover.
  Source: [huggingface/hub-docs](https://github.com/huggingface/hub-docs).
- **ACR Assess-AI** — a **live, post-deployment performance-monitoring registry** (part of ACR's
  National Radiology Data Registry), not a pre-deployment authoring format. It ingests case-level
  data via HL7v2/FHIR/DICOM/DICOMweb/CSV and reports aggregated performance back to sites.
  **ACR AI Central** — a **curated directory** of FDA-cleared imaging AI products (read-only;
  ACR populates it from FDA filings plus voluntary vendor disclosure), not an authoring schema
  either. Neither publishes a public machine-readable schema.
- **Google Model Cards** — the original prose template (2019) that the "Model Details / Intended
  Use / Factors / Metrics / Evaluation Data / Training Data / Quantitative Analyses / Ethical
  Considerations / Caveats and Recommendations" section structure traces back to. No machine-
  readable schema at all — it predates that concern.

## Crosswalk by theme

### Identity & provenance

| Concept | ATLAS | CHAI | Hugging Face | ACR | Google Model Cards |
|---|---|---|---|---|---|
| Developer/author, contact | `General metadata.Authors[]`, `.Organizations[]`, `.Contact` | `BasicInfo.ModelName/ModelDeveloper/DeveloperContact` | No YAML field — prose "Model Details" only | AI Central: Manufacturer, Product name (from FDA filings) | "Model Details" section (prose) |
| Version, release date | `General metadata.Version`, `.Date.{Created,Published,Updated}` | `ReleaseInfo.ReleaseVersion/ReleaseDate/ReleaseStage/GlobalAvailability` | No dedicated field (repo commit history serves this role) | — | "Model Details" section (prose) |
| Lineage (built on another model) | `Model-properties.Foundation model`, `.Third-party components` | `AISystemFacts.FoundationModels`, `TransparencyInformation.ThirdPartyInformation` | `base_model` (Hub model ID), `base_model_relation` (enum: adapter/merge/quantized/finetune), `new_version` (successor pointer) | — | — |

Note: Hugging Face's `base_model_relation` enum is more granular than ATLAS's or CHAI's free-text
lineage fields — it explicitly distinguishes adapter/merge/quantization/fine-tune relationships,
which ATLAS's `Foundation model` field doesn't currently differentiate.

### Licensing

| Concept | ATLAS | CHAI | Hugging Face | ACR | Google Model Cards |
|---|---|---|---|---|---|
| License | `General metadata.License.{Text,URL}` | Not found as a dedicated field | `license` (SPDX-like controlled list, or literal `other`), `license_name`/`license_link` when `other` | n/a (not an authoring format) | Mentioned under "Model Details" (prose) |

ATLAS's free-text `License.Text` plus `License.URL` is less rigorous than Hugging Face's
controlled-vocabulary `license` field with an explicit escape hatch — worth considering if ATLAS
ever wants license values to be machine-comparable across cards.

### Intended use, users, out-of-scope

| Concept | ATLAS | CHAI | Hugging Face | ACR | Google Model Cards |
|---|---|---|---|---|---|
| Intended use/workflow | `Model-properties.Indications for use`, `.Use.Intended[]` | `UsesAndDirections.IntendedUseAndWorkflow/HowToUse` | Prose only (`direct_use`) | AI Central: "Indications for Use" (from FDA 510(k) summary) | "Intended Use" section (prose) |
| Intended/out-of-scope users | `Model-properties.User.{Intended,Out-of-scope,Excluded}` | `UsesAndDirections.PrimaryIntendedUsers` | Prose only | AI Central: intended user (radiologist/other) | "Intended Use" section (prose) |
| Out-of-scope / cautioned use | `Model-properties.Use.{Out-of-scope,Excluded}` | `UsesAndDirections.CautionedOutOfScopeSettings` | Prose only (`out_of_scope_use`) | — | "Intended Use" section (prose) |
| **Targeted patient population** | **No dedicated field** (only prose within `Indications for use`/`Limitations`) | `UsesAndDirections.TargetedPatientPopulation` | — | AI Central: patient age range (from FDA filing) | — |

The targeted-patient-population gap was already identified independently during this project's own
conformance-profile work (`procurement-ready.json`'s `$comment` lists it as DEFERRED pending a
dedicated field). This crosswalk confirms it with two independent external precedents — CHAI has
a dedicated field for it, and it's one of ACR AI Central's core listing categories — which raises
the case for prioritizing it.

### Input, output, architecture

| Concept | ATLAS | CHAI | Hugging Face | ACR | Google Model Cards |
|---|---|---|---|---|---|
| Model type/architecture | `Model-properties.Architecture`, `.Model class` | `AISystemFacts.ModelType` | No structured field (prose) | — | — |
| Input description | `Model-properties.Input.Description` | `AISystemFacts.InputDataSource`, `.OutputAndInputDataTypes` | No structured field | — | — |
| Output description/type | `Model-properties.Output.{Description,Type,Coded data}` | `AISystemFacts.OutcomesAndOutputs` | No structured field | — | — |

### Training & evaluation data characterization

| Concept | ATLAS | CHAI | Hugging Face | ACR | Google Model Cards |
|---|---|---|---|---|---|
| Link to the data used | `Model.Datasets[]` (`Role`-tagged pointers to a separate dataset card) | Single free-text-like field: `AISystemFacts.DevelopmentDataCharacterization` | Prose `training_data` field, "recommended to link a Dataset Card" | Assess-AI tracks case-level data for live monitoring, not pre-deployment characterization | "Training Data" / "Evaluation Data" sections (prose) |
| Structured dataset shape/size | `Dataset.Partitions[]` → `Subset.{Patient count, Exam count, Image count, Series count, Site count}` | — | `dataset_info.{features, splits[name,num_examples,num_bytes], download_size}` — auto-generatable via the `datasets` library | — | — |
| Demographic/clinical breakdown | `Subset.{Demographic, Sex, Age range, Criterion}` | — | No structured demographic field | Assess-AI: patient demographics (exact fields not confirmed — gated) | "Quantitative Analyses > Intersectional results" (prose) |
| Dataset properties (bias, sampling, confidentiality) | `Dataset-properties.{Motivation, Sampling, Missing information, Relationships between instances, Partitioning scheme, Noise, External data, Confidentiality, Re-identification, Sensitive data}` | `AISystemFacts.BiasMitigationApproaches` (model-level, not dataset-level) | — | — | — |

Hugging Face's recommendation to link a separate Dataset Card from a Model Card is the same
two-document split ATLAS already uses (`model.json` + `dataset.json`, joined by
`Model.Datasets[]`/`Dataset.Models[]`) — independent convergence on the same pattern. Where Hugging
Face is more structured than ATLAS is the *shape* of the data (`dataset_info`'s features/splits/
byte counts, which ATLAS has no equivalent for); where ATLAS is far more structured is the
*clinical* characterization (demographics, age range, site count, confidentiality/re-identification
risk), which Hugging Face's schema doesn't address at all.

### Performance, validation, and metrics reporting

| Concept | ATLAS | CHAI | Hugging Face | ACR | Google Model Cards |
|---|---|---|---|---|---|
| Structure | `Model.Model performance[]` — one entry per evaluation, each with a `Results[]` array of scalar / outcome-count / non-scalar tuples | `KeyMetrics` — **three parallel named blocks**: `UsefulnessUsabilityEfficacy`, `FairnessEquity`, `SafetyReliability`, each with `{MetricGoal, Results/MetricResult(Label+ValueJson), Interpretation, TestType, TestingDataDescription, ValidationProcessAndJustification}` | `model-index.results[]` — `{task, dataset, metrics[type,value,name,args,verifyToken], source}` (flat, Papers-with-Code-derived); newer work-in-progress `eval_results` system layered on top | Assess-AI: live concordance between AI output and the radiology report, aggregated and reported back to sites (post-deployment monitoring, not a validation report) | "Metrics" + "Quantitative Analyses" sections (prose, incl. "Unitary results" and "Intersectional results") |
| Internal vs. external testing | `Test type` enum (Internal/External/Local/Prospective) | `TestType` field (free text per the KeyMetrics block) | Not distinguished | — | Not distinguished |
| Claimed vs. independently verified | `Provenance` enum (Claimed/Observed — self-declared) | Not found | `verifyToken` — a mechanism for cryptographically proving an evaluation was independently run on Hugging Face's own infrastructure, not just self-reported | — | — |
| Metric "pillar" (what the metric is *for*) | **Not tagged** — a Result's purpose is inferred from context, not declared | Explicit: every metric lives inside one of the three named pillar blocks | Not tagged | — | Not tagged |

Two concrete, specific comparison points worth flagging:

- **CHAI's three-pillar structure has no ATLAS analog.** ATLAS's `Factors`/`Subgroup` mechanism
  covers fairness disaggregation for any individual result, but nothing in ATLAS declares "this
  metric is a safety/reliability metric" vs. "this metric is a usefulness/efficacy metric" the way
  CHAI's schema does by construction (three separate blocks). Whether that's worth adopting depends
  on whether RSNA wants ATLAS cards to answer the same three-pillar framing CHAI does.
- **Hugging Face's `verifyToken` is a sharper provenance mechanism than ATLAS's `Provenance`
  enum.** ATLAS currently only records a self-declared Claimed/Observed distinction; Hugging Face's
  token is cryptographic proof tied to its own evaluation infrastructure. ATLAS has no
  infrastructure to replicate that directly, but it illustrates that "claimed" and "independently
  verified" can be more rigorously distinguished than a plain enum value.

### Fairness, bias, subgroup analysis

| Concept | ATLAS | CHAI | Hugging Face | ACR | Google Model Cards |
|---|---|---|---|---|---|
| Declared disaggregation axes | `Model.Factors[]` | Implicit in `KeyMetrics.FairnessEquity` block | Not found | — | "Factors" section (prose, e.g. demographic/instrumentation groups) |
| Per-result subgroup tagging | `Model-performance-results.items.*.Subgroup{Variable,Value}` | `KeyMetrics.FairnessEquity.Results` (same mechanism, different scope: whole block is fairness-only rather than taggable per-result) | Not found | — | "Quantitative Analyses > Intersectional results" (prose) |
| Bias mitigation approach | `Model-properties.Bias mitigation approach` | `AISystemFacts.BiasMitigationApproaches`, `Warnings.KnownBiasesOrEthicalConsiderations` | Prose only (`bias_risks_limitations`) | — | "Ethical Considerations" (prose) |

ATLAS's design — any individual `Results[]` entry can carry a `Subgroup`, regardless of which
metric it is — is more flexible than CHAI's approach of segregating fairness metrics into their own
block, since it lets the *same* metric be reported both overall and per-subgroup within one
`Model performance` entry.

### Regulatory status and clearance

| Concept | ATLAS | CHAI | Hugging Face | ACR | Google Model Cards |
|---|---|---|---|---|---|
| Authorization records | `Regulatory information.Authorizations[]` — repeatable, per-jurisdiction (`Jurisdiction`, `Authority`, `Pathway`, `Status`, `Identifier`, `Risk class`, `Product code`, `Regulation number`, `Predicate device`, UDI fields, dates) | `ReleaseInfo.RegulatoryApproval` — a **single, unstructured field** (the one example read holds "FDA 510(k) K221240" as plain text) | None | AI Central: FDA product code, clearance pathway, predicate device (curated from FDA filings — read-only directory data, not an authoring field) | None |
| Clinical risk level | `Model-properties.Clinical risk level` | `Warnings.ClinicalRiskLevel` (example: "Low, Class II (FDA)/IIa (EU)") | None | — | None |
| EU AI Act co-classification | `Regulatory information.EU AI Act.{Risk classification, Technical documentation}` | Not found | None | — | None |
| Predetermined change control | `Regulatory information.Change control` | Not found (though `ReleaseInfo` tracks version/stage) | None | — | None |

This is the sharpest contrast in the whole crosswalk: **ATLAS's `Authorizations[]` array is
considerably more structured than CHAI's single free-text `RegulatoryApproval` field**, supporting
multiple jurisdictions/pathways per model with jurisdiction-specific validation (e.g. ATLAS
validates that a `510(k)` pathway's `Identifier` matches the `K\d{6}` pattern). Neither Hugging
Face nor Google Model Cards address regulatory status at all — unsurprising, since neither targets
regulated domains.

### Security, maintenance, and post-deployment monitoring

| Concept | ATLAS | CHAI | Hugging Face | ACR | Google Model Cards |
|---|---|---|---|---|---|
| Cybersecurity measures | `Model-properties.Cybersecurity measures` (free text) | `AISystemFacts.Security` (optional; example cites ISO 27001/SOC2 certifications) | None found | — | None |
| Ongoing maintenance/monitoring | `Model-properties.Maintenance.{Monitoring program, Validity monitoring, Fairness monitoring, Update process, Comment}` — 4-way split | `AISystemFacts.OngoingMaintenance` — single field | `new_version` pointer only (no monitoring concept) | **Assess-AI itself is a monitoring program** — external to any card, a live registry a card's `Monitoring program` field could *name* | None |

Worth noting: `Monitoring program`'s own example values in `model.json` already include "ACR
Assess-AI" — so the relationship between ATLAS and ACR here isn't competitive, it's compositional:
an ATLAS card can (and the schema already anticipates that it would) name Assess-AI as the external
program it's enrolled in.

### Integration / technical interoperability

| Concept | ATLAS | CHAI | Hugging Face | ACR |
|---|---|---|---|---|
| Clinical-systems integration | `Model-properties.Integration` — DICOM network services, DICOMweb, HL7 v2, FHIR, IHE Radiology profiles, result-delivery channels, invocation triggers, conformance statements, validated systems | `ReleaseInfo.EHRCompatibility` (v0.2 only; a simple list) | None (ML-tooling integration via `library_name`/`pipeline_tag`, a different kind of "integration" entirely) | Assess-AI's *ingest* spec defines HL7v2/FHIR/DICOM/DICOMweb/CSV — but that's the registry's own ingestion interface, not a property of the model being documented |

ATLAS's `Integration` section is, by a wide margin, the most detailed treatment of clinical-systems
interoperability of any standard reviewed here.

### Governance, transparency, stakeholders

| Concept | ATLAS | CHAI | Hugging Face | ACR | Google Model Cards |
|---|---|---|---|---|---|
| Funding source | `General metadata.Funding` | `TransparencyInformation.FundingSource` | None | — | None |
| Third-party components | `Model-properties.Third-party components` | `TransparencyInformation.ThirdPartyInformation` | None | — | None |
| **Stakeholders consulted** | **No analog** | `TransparencyInformation.StakeholdersConsulted` | None | — | None |
| Ethical review | `General metadata.Ethical review` | Not found as a distinct field | None | — | "Ethical Considerations" (prose) |

## Machine-readability and validation

| Standard | Format | Formal schema? | Validation |
|---|---|---|---|
| **ATLAS** | JSON (Draft 2020-12 JSON Schema) | Yes — public `.json` schema files, plus 3 conformance-profile overlays | Fully machine-validatable against a published schema; this project runs `jsonschema` checks and a representative pass/fail instance suite |
| **CHAI** | XML | Yes — public `.xsd` files (v0.1, v0.2), still labeled "draft" | Machine-validatable against the XSD; supports an optional XML digital-signature block |
| **Hugging Face** | YAML (README frontmatter) | No public schema file — `modelcard.md`/`datasetcard.md` are documented-but-informal Markdown specs | Validated server-side at `git push`, but the validator itself is closed-source |
| **ACR (Assess-AI / AI Central)** | Prose / PDF / web tables | No public machine-readable schema found | Not applicable — these are documentation portals and a registry ingest spec, not authoring schemas |
| **Google Model Cards** | Prose (free-form document following a template) | None | None — predates the concept |

ATLAS is the only standard reviewed here with both a fully formal JSON Schema *and* tiered
conformance profiles with automated pass/fail semantics.

## Where ATLAS already leads

- **Regulatory authorization structure** — a repeatable, per-jurisdiction `Authorizations[]` array
  with pathway-specific identifier validation, versus CHAI's single free-text field or ACR's
  read-only directory entries.
- **Integration / interoperability detail** — DICOM/DICOMweb/HL7v2/FHIR/IHE coverage with no
  analog of comparable depth in any standard reviewed.
- **Dataset-role vocabulary** — a 13-value, ROADMAP-backed enum (pretraining through
  development/hold-out/other) shared identically across `Model.Datasets[].Role`,
  `Dataset.Models[].Role`, and `Dataset.Partitions[].Partition`; none of the other standards have a
  comparably granular, controlled vocabulary for this.
- **Reciprocal model↔dataset linkage** — `Model.Datasets[]` / `Dataset.Models[]`, both
  `Role`-tagged. CHAI has no separate dataset document to link to at all; Hugging Face's `datasets:`
  field is an untyped flat list of repo IDs with no role tagging.
- **Conformance tiers** — the only standard here with formal, machine-checkable tiers (minimal /
  procurement-ready / regulatory-ready).
- **Clinical dataset characterization** — `Subset`'s demographic/age-range/site-count breakdown has
  no analog in Hugging Face's `dataset_info` (which describes shape/size, not clinical composition)
  or in CHAI (a single free-text characterization field).

## Candidate improvements (for consideration, not applied here)

These are observations from the crosswalk, not changes made to the schema — several overlap with
gaps this project's own conformance-profile work had already flagged independently.

1. **Targeted patient population.** Already tracked as DEFERRED in `procurement-ready.json`.
   Now confirmed against two independent external precedents (CHAI's dedicated field, ACR AI
   Central's listing category) — raises the case for adding a structured field rather than leaving
   it as prose within `Indications for use`.
2. **Stakeholders consulted.** CHAI has a dedicated field for who was consulted during
   development (clinicians, patients, etc.); ATLAS has no analog. Worth considering if RSNA wants
   to document community/clinician engagement as part of `General metadata` or `Model-properties`.
3. **Metric "pillar" tagging.** CHAI requires every metric to live under Usefulness/Fairness/Safety.
   ATLAS could add an optional enum to `Model-performance-results` if that three-pillar framing
   would be useful for regulatory or procurement audiences — a judgment call, not a clear gap, since
   ATLAS's current design (tag any result with a `Subgroup`, independent of metric type) is more
   flexible in a different way.
4. **Independent-verification provenance.** Hugging Face's `verifyToken` cryptographically proves
   an evaluation ran on independent infrastructure. ATLAS's `Provenance` enum is self-declared only.
   Not directly replicable without infrastructure ATLAS doesn't have, but worth knowing the
   precedent exists if claims of independent verification become important for the
   regulatory-ready tier.
5. **License as controlled vocabulary.** Hugging Face validates `license` against a known
   identifier list with an `other` escape hatch; ATLAS's `License.Text` is free text. Low priority,
   but would make license values machine-comparable across cards.
6. **Human-oversight measures** — already tracked as DEFERRED in `regulatory-ready.json`. This
   crosswalk found **no external precedent** for a dedicated structured field in CHAI, Hugging
   Face, or ACR either; none of the four standards reviewed has operationalized this HTI-1/EU AI
   Act Annex IV requirement as a discrete field yet. That reduces the urgency somewhat — ATLAS
   wouldn't be behind an established practice, it would be ahead of one that doesn't exist yet
   anywhere reviewed.
7. **Base-model relation granularity.** Hugging Face's `base_model_relation` enum
   (adapter/merge/quantized/finetune) is more specific than ATLAS's free-text `Foundation model`
   field. Could be added as an optional sub-field if that distinction matters for RSNA's use cases.

## Sources

**CHAI:** [coalition-for-health-ai/mc-schema](https://github.com/coalition-for-health-ai/mc-schema)
(README, `DOCUMENTATION.md`, `v0.1/schema.xsd`, `v0.2/schema.xsd`, `v0.2/examples/Aidoc_ICH-02-RT.xml`).

**Hugging Face:** [huggingface/hub-docs](https://github.com/huggingface/hub-docs) —
`modelcard.md`, `datasetcard.md`, `docs/hub/model-cards.md`, `docs/hub/model-card-annotated.md`,
`docs/hub/datasets-cards.md`, `docs/hub/eval-results.md`, `docs/hub/model-cards-co2.md`; and
[huggingface/huggingface.js](https://github.com/huggingface/huggingface.js) (`packages/tasks/src/pipelines.ts`).

**ACR:** ACR's own press releases and web pages describing Assess-AI and AI Central (via search
snippets — gated primary documents such as the Assess-AI Data Dictionary and Ingest Specification
PDF were not directly read in this research pass):
[Assess-AI launch announcement](https://acr.org/News-and-Publications/American-College-of-Radiology-Launches-Landmark-Artificial-Intelligence-Quality-Registry),
[Assess-AI Data Dictionary (reference)](https://nrdrsupport.acr.org/support/solutions/articles/11000129783-assess-ai-data-dictionary),
[Assess-AI Ingest Specification v2.1 (PDF, not directly read)](https://edge.sitecorecloud.io/americancoldf5f-acrorgf92a-productioncb02-3650/media/ACR/Files/DSI/Assess-AI-Ingest-Specification-Designed-Document-v3.pdf),
[What Data Is Used](https://www.acr.org/Data-Science-and-Informatics/AI-in-Your-Practice/Performance-Monitoring/What-Data-is-Used),
[AI Central relaunch, category taxonomy](https://itnonline.com/article/american-college-radiology-acr-data-science-institute-dsi-introduces-new-aicentralorg).

**Google Model Cards:** "Model Cards for Model Reporting," Mitchell et al., 2019
(`arXiv:1810.03993` — not directly fetchable in this environment; section structure corroborated
across multiple independent secondary sources, not quoted from the paper itself).
