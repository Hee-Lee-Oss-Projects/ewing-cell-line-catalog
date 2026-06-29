# PLAN — ewing-cell-line-catalog

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated · Risk tier: low (with medium-risk sub-tasks, explicitly tagged)

> **For the families.** Ewing sarcoma is a rare, aggressive bone-and-soft-tissue cancer that
> mostly strikes children, teenagers, and young adults. Five-year survival for metastatic or
> relapsed disease remains poor and has barely moved in decades. This project does not treat a
> single patient and makes no clinical claim. What it does is unglamorous and real: it makes the
> *laboratory models* researchers use to study Ewing sarcoma **trustworthy and findable** — which
> cell lines are authentic, which are misidentified, what fusion they carry, what open data exists
> for them, and what genetic dependencies the open screens report. Bad models produce
> irreproducible science, and irreproducible science wastes the scarce time and funding that
> families are counting on. Getting the catalog right is a way of taking care.

---

## Executive summary

Ewing sarcoma research runs on a small number of immortalized **cell-line models** (A673, SK-ES-1,
RD-ES, TC-71, TC-32, SK-N-MC, EW8, CHLA-9/10/25, and roughly two dozen others). These models, and
the open genomic/functional-genomic data generated on them, are scattered across several public
resources — the **Cancer Dependency Map (DepMap)** and the legacy **Cancer Cell Line Encyclopedia
(CCLE)** at the Broad Institute, the **Cell Model Passports** at the Wellcome Sanger Institute,
**Cellosaurus** at the SIB, and the **ICLAC Register of Misidentified Cell Lines**. No single,
source-verified, machine-readable place tells a researcher, for one Ewing model: *is it
authenticated, is it on the misidentified register, what fusion does it carry and at what
breakpoint, what open omics and functional-genomics data exist for it, and what selective genetic
dependencies do the open CRISPR screens report* — each assertion carried with its provenance and
license.

This project builds that catalog: a **source-verified, fully-provenanced, openly-licensed metadata
catalog of Ewing sarcoma cell-line models and their open-data dependencies**, plus the lightweight
tooling (identity resolution, validation, dependency aggregation) to keep it correct and current as
DepMap ships quarterly releases. **The deliverable is metadata and derived aggregates from
open-access sources — never patient data, never a clinical recommendation, never a therapeutic
claim.**

The hard constraint and central design principle is the **open-data-only gate**: every source is
verified to be open-access and to permit derivative redistribution under our output license before
any data enters the catalog. Controlled-access human data (dbGaP, EGA, individual-level biobanks)
is **categorically out of scope**. Non-commercial / custom-license sources (COSMIC, OncoKB) are
**flagged and link-only** — never redistributed into a CC-BY catalog. Genetic dependency scores are
presented as *research observations with provenance and caveats*, never as treatment guidance.

Risk tier is **low** for the project as a whole (research-facing metadata on de-identified
in-vitro models from open sources), with specific sub-tasks elevated to **medium** where judgement
is required: license interpretation, donor-provenance handling, misidentification calls, and the
framing of dependency data. There is **no patient-facing content** in this project; any such
derivative would be a separate `high`-risk project gated behind oncologist + patient-advocate
review and is explicitly out of scope here.

## Problem & beneficiaries

**Who is helped (directly).** Ewing sarcoma researchers — wet-lab biologists, computational
biologists, and translational scientists in academic labs, children's-oncology consortia, and
non-profit research foundations — who must choose, authenticate, and characterize cell-line models
and find the open data that exists for them. Today this is done by hand, lab by lab, from
half-remembered papers and scattered portals, and it goes wrong often enough that misidentified and
contaminated lines remain in active use years after they are flagged.

**Who is helped (ultimately).** Children, adolescents, and young adults with Ewing sarcoma and
their families — *indirectly but really* — because reproducible model selection is a precondition
for research that translates. A dependency "discovered" in a line that is secretly a different
tumor, or a fusion mischaracterized in a key model, sends real effort and real money down dead
ends. We are explicit that this benefit is **upstream and indirect**: this project ships a research
tool, not a patient resource, and we will not let the emotional weight of the disease tempt the
catalog into clinical-sounding claims it cannot support.

**The verified need.** The *general* need is well established and independently documented: cell-line
misidentification and the resulting irreproducibility are among the most-cited problems in
preclinical cancer research (the ICLAC register exists precisely because of it), and the
fragmentation of model metadata across DepMap / Cell Model Passports / Cellosaurus is a daily
friction for the field. We treat that general need as real. The **per-consumer, per-partner need is
TO BE SECURED**: we have not yet confirmed a named lab, consortium (e.g. a Ewing/sarcoma research
network or a children's-oncology cooperative group), or upstream resource maintainer who has agreed
to adopt the catalog or accept contributed corrections. Until a specific beneficiary confirms,
individual tasks carry `verifiedNeed: false`. This honesty matters: "delivered, not merged" requires
the output to actually be *used* by a beneficiary, not merely produced.

**Partner / requestor.** TO BE SECURED. Candidate channels: academic Ewing-sarcoma labs;
sarcoma-focused non-profits and patient foundations that fund research infrastructure; the
upstream resource maintainers themselves (Cellosaurus / Cell Model Passports / DepMap accept
community corrections). M0 includes explicit outreach; no partner is assumed.

## Goals and non-goals

**Goals**

- Produce a **reusable cell-line record schema** and provenance model: every field on every model
  carries a citation to a specific source at a specific version.
- Deliver a **source-verified, authenticated catalog** of Ewing sarcoma cell-line models —
  identity-resolved across DepMap / Cell Model Passports / Cellosaurus (one model, all its IDs),
  with authentication status and misidentification/contamination flags surfaced prominently.
- Annotate each model's **fusion status** (EWSR1-FLI1 vs. EWSR1-ERG vs. variant/other; breakpoint
  type where published) with provenance.
- Map, per model, the **availability of open data types** (expression, copy number, open-tier
  mutations, fusions, drug sensitivity) so researchers know what exists before they go looking.
- Aggregate **open genetic-dependency data** (DepMap CRISPR gene-effect scores) per model into
  research-framed summaries — with version, methodology, and a standing "research observation, not
  clinical guidance" caveat on every dependency assertion.
- Make the **open-data-only / license gate** a non-skippable, auditable, per-source step.
- Feed **corrections upstream** (e.g. to Cellosaurus / Cell Model Passports) where the catalog finds
  errors — so the public good compounds beyond our own repo.
- Ship small, dependency-light **tooling** (identity resolver, validator, dependency aggregator) so
  the catalog stays correct and refreshable across quarterly DepMap releases.

**Non-goals**

- We do **not** ingest, store, or redistribute **patient-level / controlled-access** data (dbGaP,
  EGA, individual-level biobank genotypes). Out of scope, full stop.
- We do **not** produce **any patient-facing content or medical advice**, and we do **not** make
  therapeutic, prognostic, or "this target is druggable in patients" claims. A dependency in a cell
  line is a lab observation, not a treatment.
- We do **not** redistribute **non-commercial / custom-license** source content (COSMIC mutation
  calls, OncoKB annotations) into our CC-BY catalog — we link to it instead.
- We do **not** generate, re-derive, or re-call raw omics (no re-alignment, no variant calling, no
  re-running screens) — that is the remit of sibling projects (`ewing-expression-reanalysis`,
  `ewing-fusion-detection-benchmark`). We catalog and reference; we do not re-compute the science.
- We do **not** rank models as "best," prescribe experiments, or interpret dependencies as drug
  recommendations.
- We do **not** invent donor identities or attempt re-identification; donor fields record only what
  authoritative public resources already publish, and nothing more.
- We do **not** auto-publish corrections upstream; a human submits after review.

## Success metrics (outcomes)

Outcome-based and beneficiary-centric. Vanity metrics (e.g. "entries written") are excluded. An
outcome counts only if it is **externally verifiable**.

| Metric | Baseline | Target (first 6 months) |
| --- | --- | --- |
| Authenticated Ewing models with a complete, source-verified, fully-provenanced catalog entry | 0 | ≥ 20 models |
| Models with identity resolved across all three core ID systems (DepMap ↔ RRID ↔ Cell Model Passports) | 0 | ≥ 20 (100% of catalogued) |
| Misidentified/contaminated lines explicitly flagged (cross-checked against ICLAC + Cellosaurus) | 0 | 100% of catalogued models carry an authentication-status field |
| Corrections accepted upstream (Cellosaurus / Cell Model Passports / DepMap) | 0 | ≥ 2 accepted (with permalink/issue/PR evidence) |
| External reuse events (catalog cited, forked, or adopted by a third party) | 0 | ≥ 3 verifiable events |
| Confirmed research consumer/partner adopting or requesting the catalog | 0 | ≥ 1 secured |
| Provenance coverage: catalogued assertions carrying a source + version citation | n/a | 100% (a record with any uncited assertion fails review) |
| License/scope defects in review (NC source redistributed, controlled-access data ingested, therapeutic claim) | n/a | **0** (any single occurrence is a release blocker) |

**Quantifying "complete" so DoDs are checkable.** Each model carries a **completeness score
(0–100)**: the fraction of canonical-schema fields populated *and* source-verified (identity IDs,
authentication status, disease/Oncotree, fusion status with evidence, donor fields where publicly
available, omics-availability map, dependency summary with version, license per source-block). A
delivered entry must reach **≥ 90/100**, recorded as a before/after pair against the as-found state
captured at triage. The pair is stored in the entry's provenance artifact.

**What "accepted upstream" means and the evidence the Steward records.** A single canonical
**outcome artifact** per event (`outcomes/<event-id>.json`: channel, URL/permalink, timestamp,
before/after detail). Acceptance is defined per channel: Cellosaurus correction = the maintainer's
applied change / acknowledgement reference; Cell Model Passports = applied correction or accepted
issue; DepMap = accepted community-feedback reference; third-party reuse = a citation, a fork that
builds on the catalog, or an issue/PR referencing it. Self-reported reuse never counts.

## Scope

**In scope**

- Catalog **metadata records** for Ewing sarcoma cell-line models: identity/cross-reference IDs,
  authentication & misidentification status, disease classification (Oncotree), fusion status and
  breakpoint type, donor fields *as published by authoritative resources*, culture conditions where
  published, and model availability (repository + catalog number / RRID, link-only).
- **Omics-availability mapping**: which open data types exist for each model (expression, copy
  number, open-tier mutations, fusions, drug sensitivity), with the source and release version.
- **Open genetic-dependency aggregation**: derived per-model summaries from DepMap CRISPR
  gene-effect (Chronos) data, framed as research observations with version + methodology + caveat.
- **Tooling**: identity resolver, schema validator + cross-source consistency checks, dependency
  aggregator, catalog publisher (machine-readable + human-readable).
- **Per-source license + open-data + donor-privacy triage** and recording.
- **Upstream correction** contributions where the catalog finds errors.

**Out of scope**

- **Controlled-access / patient-level data** of any kind (dbGaP, EGA, individual biobanks) — never.
- **Patient-facing content, medical advice, therapeutic/prognostic claims** — never (would be a
  separate `high`-risk project).
- **Re-deriving raw science**: re-alignment, variant calling, fusion calling, re-running CRISPR/drug
  screens, expression reanalysis — owned by sibling projects; this catalog references their inputs/
  outputs, it does not recompute them.
- **Redistributing non-commercial/custom-license source content** (COSMIC, OncoKB) — link-only.
- **Hosting or mirroring raw omics matrices** — we reference them at source with version pins; we do
  not re-host DepMap/Sanger data dumps. (Small derived aggregates we *do* compute are CC-BY and
  redistributable; bulk source matrices are referenced, not mirrored.)
- **PDX / organoid / single-cell model curation** — adjacent projects (`ewing-pdx-model-index`,
  `ewing-single-cell-atlas`); this catalog cross-links to them but does not own them.
- **Model ranking, experiment design, or any "use this line / hit this target" recommendation.**

**Candidate roster.** The pool of models we *might* catalog is maintained in `MODEL-ROSTER.md` — a
license-classified, RRID-keyed list of Ewing sarcoma cell lines drawn from DepMap/Cell Model
Passports/Cellosaurus. The roster is a **candidate backlog only**: no model becomes a catalog entry
until it passes the per-model open-data/license/donor-privacy gate. The roster honestly flags
known-misidentified lines (so they are catalogued *with their warning*, not silently dropped) and
records each source's license up front.

## Solution approach & architecture

This is a **content/data-curation project with light, dependency-minimal software**. It assembles a
derived metadata catalog from open sources; it is **not** a pipeline that moves or recomputes raw
omics.

**Canonical record model (source of truth).** One internal record per model; all outputs (JSON,
CSV, human-readable site) are *projections* of it. Every value-bearing field is a `{value,
source, sourceVersion, retrievedAt, license}` provenance cell (or an array of them where multiple
sources agree/disagree). Fields:

- `id` (internal slug); `identity { depmapId (ACH-…), rrid (CVCL_…), cellModelPassportId (SIDM…),
  ccleName, synonyms[] }`
- `authentication { strProfileAvailable, misidentified (bool), misidentifiedRegister (ICLAC ref),
  contaminationNotes, authenticationSource }` — **surfaced first** in every entry
- `disease { oncotreeCode (e.g. EWS), oncotreeName, lineage (bone), subtype }`
- `fusion { partner5p (EWSR1), partner3p (FLI1|ERG|other), breakpointType (e.g. type 1),
  evidence[] }`
- `donor { sexAtSampling, ageAtSampling, siteOfSampling, sampleType (primary|metastasis|relapse) }`
  — **populated only from values already published by Cellosaurus/Cell Model Passports**; never
  inferred, never enriched, never cross-linked to external person data
- `culture { medium, notes }` (where published)
- `availability[] { repository (ATCC|DSMZ|COG|...), catalogNumber, url }` — **link + fact only**, no
  copied vendor prose
- `omicsAvailability { expression, copyNumber, mutationsOpenTier, fusions, drugSensitivity }` each
  `{ available (bool), source, sourceVersion, url }`
- `dependencies { source: "DepMap CRISPR (Chronos)", release, selectiveDependencies[] {gene,
  geneEffect, caveat}, methodologyRef, disclaimer }` — research observation, not guidance
- `license` per source-block; `completenessScore { before, after }`; `provenance { compiledAt,
  compiledFromReleases[] }`

**Pipeline (per model).**
1. **Triage & gate** — confirm each contributing source is open-access and permits derivative
   redistribution under our output license; confirm no controlled-access data; confirm donor fields
   are limited to already-published values. PASS/FLAG/EXCLUDE recorded as a committed artifact.
2. **Identity resolution** — resolve and cross-check DepMap ID ↔ RRID (Cellosaurus) ↔ Cell Model
   Passports ID; record conflicts. RRID is the cross-resource anchor.
3. **Authentication check** — look up ICLAC register + Cellosaurus "problematic" flags; record
   misidentification/contamination status prominently.
4. **Metadata assembly** — disease/Oncotree, fusion (with evidence), donor (published-only),
   availability, culture.
5. **Omics-availability mapping** — record which open data types exist + where + at what version.
6. **Dependency aggregation** — derive per-model selective-dependency summaries from DepMap CRISPR
   gene-effect data at a pinned release, attach methodology + caveat.
7. **Validation** — schema-valid; RRID/ID format checks; cross-source consistency checks
   (e.g. fusion partner consistent with disease; flag disagreements rather than silently picking).
8. **Review & publish** — license reviewer + domain reviewer sign-off; publish machine-readable +
   human-readable; file upstream corrections where errors were found.

**Tech stack.** TypeScript, ESM, pnpm workspaces (Elyos convention). Tooling is small Node packages
with minimal dependencies. Catalog authored as JSON (canonical) with CSV + Markdown/static-site
projections. No runtime services; everything runs locally or in CI. Source data is accessed via the
providers' published download endpoints / files at pinned release versions; **we reference, we do not
re-host** bulk matrices.

**Pinned source releases** (recorded in each record's `compiledFromReleases`, bumped only via a
deliberate refresh task): DepMap public release (e.g. `DepMap Public 24Q2` — exact release pinned at
M2), Cell Model Passports release date, Cellosaurus version, ICLAC register version. Any drift is an
isolated refresh task, never a silent change.

**Key decisions (locked).**
- **Canonical-model-first**: never hand-maintain parallel JSON/CSV/site formats.
- **Provenance is mandatory at the field level**, not the record level: an uncited assertion fails
  review.
- **RRID (Cellosaurus CVCL_) is the identity anchor** across resources.
- **Open-data/license/donor gate is blocking** and committed as an artifact per model.
- **Dependencies are descriptive aggregates with a standing caveat**, never interpreted clinically.
- **Upstream corrections are a first-class output**, not a side effect.

## Data, licensing & compliance

**THIS IS THE CRITICAL SECTION. It leads with the binding cancer-domain guardrails; everything below
serves them.**

### Binding cancer-domain guardrails (non-negotiable)

1. **Open-access / aggregate / de-identified data ONLY.** Every datum in the catalog derives from an
   open-access resource. **Controlled-access human data — dbGaP, EGA, individual-level biobanks — and
   ANY identifiable patient data are OUT OF SCOPE.** They require authorized access + IRB oversight,
   which is not what donated AI tasks are for. A task that would require such access is refused and
   flagged, not worked around.
2. **Per-source license verification, conservatively.** TCGA/GDC open-tier and GEO are open;
   **COSMIC and OncoKB are non-commercial / custom-license and are FLAGGED — link-only, never
   redistributed** into our CC-BY catalog. Each source's license is verified and recorded *before*
   any of its data enters a record.
3. **No medical advice; no therapeutic/prognostic claims.** This project ships research metadata.
   It is not patient-facing. Any patient-facing derivative is a separate `high`-risk project
   requiring **oncologist + patient-advocate review** and is out of scope here. Every dependency
   assertion carries a standing "research observation; not medical advice; not a treatment
   recommendation" caveat.
4. **Provenance on every assertion.** No uncited claims. Each value records its source + version +
   retrieval date.

### Cell lines and donor privacy — the specific judgement

Cell-line models are immortalized in-vitro reagents, **de-identified by their nature** and decades
of redistribution. However, they originate from real patients — many of them children with Ewing
sarcoma. We therefore hold a conservative line:

- **Donor fields are populated *only* from values already published by authoritative resources**
  (Cellosaurus, Cell Model Passports): typically sex, age-at-sampling, sample site/type. We record
  *only* what those resources already publish and **add nothing**.
- **No re-identification, no enrichment, no linkage.** We never attempt to link a model's donor
  fields to any external person-level dataset, obituary, case report, or registry. Any task that
  would do so is refused and flagged.
- If a source's donor record looks unusually identifying (e.g. an ultra-rare combination tied to a
  named case), the entry is escalated to the License+privacy reviewer and the donor block is reduced
  to the minimum or omitted — never expanded.

### Source register and license disposition

| Source | What we take | License | Disposition |
| --- | --- | --- | --- |
| **DepMap (Broad)** — CRISPR gene-effect (Chronos), CCLE expression/CN/mutations(open-tier)/fusions, model metadata | dependency aggregates, omics-availability, IDs | CC BY 4.0 (DepMap public releases) — **VERIFY exact release terms at gate** | ACCEPT (attribution; pin release) |
| **Cell Model Passports (Sanger)** | model metadata, IDs, omics availability | CC BY 4.0 — **VERIFY at gate** | ACCEPT (attribution) |
| **Cellosaurus (SIB)** | RRID, synonyms, donor (published-only), misidentification flags, STR availability | CC BY 4.0 | ACCEPT (attribution; identity anchor) |
| **ICLAC Register of Misidentified Cell Lines** | misidentification/contamination status | published register (terms **VERIFY at gate**; attribute) | ACCEPT as factual flag (cite register + version) |
| **COSMIC / Cell Lines Project (Sanger)** | mutation calls | non-commercial / custom | **FLAG — link-only, NOT redistributed** |
| **OncoKB** | variant/oncogenicity annotation | non-commercial / custom | **FLAG — link-only, NOT redistributed** |
| **GDSC drug sensitivity** | drug-response availability | **VERIFY** (terms not assumed) | availability-flag only until terms verified; aggregates only if license permits |
| **ATCC / DSMZ / COG repositories** | catalog number, RRID, availability link | vendor catalog (copyrighted prose) | **fact + link only**; never copy descriptive text |
| **PubMed / PMC** | citations for fusion/provenance assertions | PMC-OA subset for any quoted text; otherwise cite, don't quote | ACCEPT as citation source |
| **dbGaP / EGA / individual biobanks** | — | controlled-access | **OUT OF SCOPE — never ingested** |

**Objective "permits redistribution" criterion.** A source-block is admitted only if its license is
verified to permit derivative redistribution under our output license, recorded with a cited
clause/URL and `permitsRedistribution: true`. Missing/unparseable evidence, NC terms, or any
controlled-access provenance = FLAG/EXCLUDE (link-only or omitted), never default-allow.

**Provenance model.** Every record stores, per source-block: source name, source version/release,
retrieval timestamp, source URL, license identifier + license URL + a captured snapshot reference
(committed copy + SHA-256 + Wayback save URL for license/terms pages), and the required attribution
string. Field-level provenance ties each value to one of these blocks.

**Attribution & output license.** Catalog **metadata + derived aggregates** are licensed **CC BY
4.0**; **code** (resolver, validator, aggregator, publisher) is **MIT**. All outputs attribute
DepMap, Cell Model Passports, Cellosaurus, and ICLAC per their licenses and clearly state that the
catalog — not the underlying source data — is our contribution.

## Quality, review & risk gates

**Risk tier: low (project), with medium-risk sub-tasks explicitly tagged.** The project is
research-facing metadata on de-identified in-vitro models from open sources — low risk. Specific
judgements raise individual tasks to **medium**: license interpretation, donor-provenance handling,
misidentification calls, and dependency framing. **No task is `high`** because nothing here is
patient-facing; any patient-facing work is out of scope and would require a separate project with
oncologist + patient-advocate sign-off.

**Required review before a deed is "done":**
- **License + open-data + privacy reviewer** (mandatory, every source-block and every model):
  confirms open-access provenance, verified redistributable license (or correct link-only/exclude
  handling for NC/controlled sources), and that donor fields are published-only with no
  re-identification risk. **Hard, non-skippable gate.**
- **Domain reviewer** (cancer biologist / cancer bioinformatician, mandatory for metadata,
  misidentification, fusion, and dependency tasks): confirms biological accuracy — correct
  identity resolution, correct fusion characterization, correct reading of dependency data, correct
  "research observation, not advice" framing.
- **Technical reviewer**: confirms schema validity, resolver/validator/aggregator correctness, CI
  green, golden fixtures.

**Test fixtures & golden files (so "CI green" means something).** Each tool ships committed,
synthetic/public fixtures exercised in CI: resolver golden cases (known model → known
DepMap/RRID/SIDM triples, including a deliberate cross-resource conflict that must be flagged);
validator golden cases (valid records pass, records with an uncited field or a malformed RRID
fail); aggregator golden cases (a synthetic gene-effect matrix → expected selective-dependency
summary). No fixture contains controlled-access data.

**Definition of Shipped.** A model's catalog entry is **shipped** when: every source-block passed
the open-data/license/privacy gate (artifact committed); identity is resolved across the three core
systems; authentication/misidentification status is present; all assertions are field-level
provenanced; the completeness score is ≥ 90/100; the entry validates in CI; the domain reviewer and
license reviewer have signed off; and the entry is **published openly** (CC-BY catalog) **and used or
adopted by a real research consumer** *or* fed back as an accepted upstream correction (per-channel
outcome artifact recorded by the Steward). **Producing the entry is not shipped; verifiable use or
accepted upstream correction is.**

## Roadmap & milestones

**M0 — Foundation & cold-start (thin).**
- Goal: build the schema + gate + core tooling, prove the end-to-end flow on one pilot model, and
  open beneficiary/reviewer outreach.
- Cold-start de-risking: the pilot (A673 — the most widely used, well-characterized Ewing model) is
  chosen because every core source covers it, so the full pipeline can be exercised; and the first
  *real outcome* is reachable via a **self-serve channel** (an accepted upstream correction to
  Cellosaurus / Cell Model Passports does not depend on us first signing a partner).
- Exit criteria: (1) canonical record schema + field-level provenance model published; (2) source
  register with verified per-source licenses published; (3) open-data/license/donor gate checklist
  exists and is applied to the pilot; (4) identity resolver + validator working in CI with golden
  fixtures; (5) one pilot model (A673) catalogued end-to-end at ≥ 90/100 with gate artifact; (6)
  License+privacy reviewer and Domain reviewer **named**; (7) ≥ 1 beneficiary/partner outreach
  thread opened.

**M1 — Authenticated core catalog.**
- Goal: catalog the most widely used authenticated Ewing models, with misidentification audit and
  fusion annotation.
- Exit criteria: (1) authoritative roster compiled and gate-triaged in `MODEL-ROSTER.md`; (2) ≥ 10
  models fully catalogued at ≥ 90/100; (3) 100% of catalogued models carry authentication status
  cross-checked against ICLAC + Cellosaurus; (4) fusion status annotated with provenance for every
  catalogued model; (5) ≥ 1 upstream correction submitted (accepted counts toward M3).

**M2 — Dependencies & omics availability.**
- Goal: add open-dependency aggregates and omics-availability mapping, and publish the catalog.
- Exit criteria: (1) DepMap release pinned; dependency aggregator working in CI with golden
  fixtures; (2) omics-availability map populated for the core roster; (3) dependency summaries (with
  version + methodology + standing caveat) for the core roster; (4) catalog published
  machine-readable (JSON + CSV) and human-readable, all assertions provenanced; (5) catalog reaches
  ≥ 20 fully-catalogued models.

**M3 — Upstream corrections, reuse & sustainability.**
- Goal: demonstrate real reuse and an upstream-correction + refresh model.
- Exit criteria: (1) ≥ 2 corrections accepted upstream (evidence artifacts recorded); (2) ≥ 3
  verifiable external reuse events; (3) ≥ 1 confirmed research consumer/partner; (4) refresh process
  documented for quarterly DepMap releases with a named steward.

Dependencies: M1 depends on the M0 schema + gate + resolver; M2 dependency aggregation depends on
the M1 authenticated roster (don't aggregate dependencies for a misidentified line without its
flag); M3 depends on a body of published entries from M1–M2.

## Work breakdown

The itemized, schema-mapped backlog lives in `TASKS.md`, organized by the milestones above, each
with a task table (`ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer`),
acceptance criteria for the most important tasks, and a milestone Definition of Done. A
sized-but-unscheduled backlog and one complete, schema-valid example Task JSON are included there.
The candidate model roster that feeds per-model tasks lives in `MODEL-ROSTER.md`.

## Governance, roles & stakeholders

- **Maintainer (Owner):** TBD — owns the schema, gate, tooling, and backlog.
- **License + open-data + privacy reviewer:** TBD (name TO BE SECURED) — mandatory, **non-skippable**
  gatekeeper for open-data provenance, license redistribution, and donor-privacy. Must be filled
  **before the M0 pilot is reviewed**. Can read open-data/bioinformatics licenses (CC/Sanger/Broad
  terms) and apply the donor-privacy rule. May rotate among ≥ 2 qualified reviewers, but at least one
  named reviewer must always exist or triage halts. Until named, all tasks remain
  `verifiedNeed:false` and no source-block passes the gate.
- **Domain reviewer (cancer biologist / bioinformatician):** TBD (name TO BE SECURED) — mandatory
  for metadata/misidentification/fusion/dependency tasks; verifies biological accuracy and the
  "research observation, not advice" framing.
- **Technical reviewer(s):** rotation of contributors verifying schema, resolver, validator,
  aggregator (CI green).
- **Steward (last-mile owner):** TBD — owns relationships with research consumers and upstream
  maintainers; records acceptance/reuse outcome artifacts (the "delivered" signal).
- **Partner / requestor:** TO BE SECURED — a named lab, consortium, foundation, or upstream resource
  maintainer.

## Dependencies & integrations

- **External data sources (pinned):** DepMap public release, Cell Model Passports, Cellosaurus,
  ICLAC register — versions recorded per record; COSMIC/OncoKB link-only; GDSC terms-pending. (See
  Data, licensing & compliance.)
- **External standards:** RRID / Cellosaurus CVCL identifiers, Oncotree disease ontology, HGNC gene
  symbols (for fusion partners and dependency genes), SPDX license identifiers.
- **Sibling Elyos projects (cross-link, not dependency):** `ewing-open-data-catalog`,
  `ewing-pdx-model-index`, `ewing-single-cell-atlas`, `ewing-expression-reanalysis`,
  `ewing-fusion-detection-benchmark`, `ewing-drug-target-evidence` — this catalog links to and is
  linkable from these; it does not block on them.
- **Elyos pieces:** Task JSON schema (`packages/schema`), donated-lane CLI workspace/PR flow
  (`packages/cli`), good-deed definition + refusal guardrails. No funded-lane/runner dependency.

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
| --- | --- | --- | --- | --- |
| Ingesting or redistributing controlled-access / patient-level data | Low | Critical | Open-data-only gate as a blocking pre-step; dbGaP/EGA/biobank categorically excluded; reviewer sign-off per source-block | License+privacy reviewer |
| Redistributing an NC/custom-license source (COSMIC/OncoKB) into the CC-BY catalog | Medium | High | License gate; COSMIC/OncoKB hard-coded as link-only; objective `permitsRedistribution` criterion; reviewer sign-off | License+privacy reviewer |
| Catalog implies a therapeutic/clinical claim from a dependency | Medium | High | Standing "research observation, not advice" caveat on every dependency; domain-reviewer framing check; non-goal enforced in review | Domain reviewer |
| Cataloguing a misidentified line without its warning | Medium | High | Mandatory authentication check vs. ICLAC + Cellosaurus; misidentification surfaced first in every entry | Domain reviewer |
| Wrong cross-resource identity resolution (merging two different models) | Medium | High | RRID as anchor; cross-source consistency checks; conflicts flagged not auto-resolved; resolver golden tests | Technical reviewer |
| Donor fields drift toward re-identification | Low | High | Published-only rule; no enrichment/linkage; escalate unusual donor records; reduce/omit on doubt | License+privacy reviewer |
| Fusion mischaracterization (FLI1 vs ERG, breakpoint type) | Medium | Medium | Provenance + domain review; cite primary source; flag disagreements across sources | Domain reviewer |
| Source release drift (DepMap quarterly) makes catalog stale | High | Medium | Pin release per record; refresh task; staleness flagged when a newer release ships | Maintainer |
| No research consumer secured → catalog built but unused (fails "delivered") | Medium | High | M0 outreach; self-serve upstream-correction path gives a real outcome independent of a partner; `verifiedNeed:false` until secured | Steward |
| Emotional weight of pediatric cancer pressures scope toward patient guidance | Medium | High | Explicit non-goal; "research tool, not patient resource" stance; reviewers reject any patient-facing/clinical drift | Maintainer |

## Security & privacy

- **Threat surface is small** (no runtime service, no data hosting). Main surfaces: CI, the
  published catalog files, and the references we publish.
- **Secrets handling:** tooling needs no credentials by default. If a source-download token is ever
  needed, it is supplied by the human running the task and must never be written into logs, receipts,
  or committed files (per Elyos rules).
- **PII / donor privacy:** the dominant concern is *upstream* donor provenance in cell-line records.
  Handled by the published-only rule, the no-re-identification/no-linkage prohibition, and the
  License+privacy gate. Controlled-access data is never touched.
- **Abuse/misuse prevention:** refuse and flag any task steering the catalog toward re-identifying
  donors, laundering controlled-access/NC data as open, or producing patient-facing clinical claims.
  The catalog stays descriptive, source-verified, and research-framed.

## Sustainability & maintenance

- **Post-delivery ownership:** the steward maintains consumer/upstream relationships; the maintainer
  keeps the tooling and schema current.
- **Refresh:** DepMap ships ~quarterly; records pin a release and a refresh task re-runs aggregation
  against the new release, recording version deltas. Cellosaurus/ICLAC updates trigger
  re-authentication checks. Stale entries become `maintenance` tasks.
- **Outcome tracking:** the steward records accepted upstream corrections and external reuse events
  against the success metrics, reviewed each milestone.

## Open questions

- Which named lab/consortium/foundation or upstream maintainer becomes the first confirmed
  beneficiary/partner?
- **DepMap release-terms confirmation:** confirm the exact license of the specific public release we
  pin (assumed CC BY 4.0; must be verified at the gate before any aggregate is published).
- **GDSC drug-sensitivity terms:** confirm whether GDSC permits redistributable aggregates or only
  an availability flag.
- How do we represent **cross-source disagreement** (e.g. two sources give different fusion
  breakpoints) in the public catalog — show both with provenance, or designate a primary? (Default:
  show both with provenance; never silently pick.)
- What is the minimal, safe **donor field set** — do we publish age-at-sampling at all, or band it?
  (Default: publish only what the source publishes; escalate unusual cases.)
- For upstream corrections, which channel per resource (Cellosaurus correction form vs. Cell Model
  Passports issue vs. DepMap feedback)?

## References

- Elyos work rules — `C:\code\elyos\CLAUDE.md`
- Good Deed Definition + risk tiers — `C:\code\elyos\docs\good-deed-definition.md`
- Task JSON schema — `C:\code\elyos\packages\schema\src\schemas.ts`
- Portfolio roadmap (Track 8 cancer guardrails) — `C:\code\elyos\planning\ROADMAP.md`
- Sibling exemplar (catalog/datasheets house style) — `C:\code\elyos\planning\projects\open-data-datasheets\PLAN.md`
- DepMap / Cancer Dependency Map (Broad Institute) — CRISPR gene-effect (Chronos), CCLE
- Cell Model Passports (Wellcome Sanger Institute)
- Cellosaurus (SIB) — RRID/CVCL identifiers, misidentification flags
- ICLAC Register of Misidentified Cell Lines
- Oncotree disease ontology; HGNC gene nomenclature; SPDX license list; RRID initiative

---

## Appendix A — Improvements applied

The following 25 specific improvements were identified against the first draft and **have been
applied** to the body above. Each is concrete and reflected in the corresponding section.

1. **Field-level provenance, not record-level.** Changed the data model so every value is a
   `{value, source, sourceVersion, retrievedAt, license}` cell; an uncited field now *fails review*
   (Solution approach; Success metrics 100% provenance row).
2. **Open-data-only gate hard-coded as the lead guardrail.** The Data/licensing section now opens
   with the four binding cancer guardrails before any source table — controlled-access categorically
   excluded.
3. **COSMIC/OncoKB pinned as link-only.** Added explicit non-commercial disposition in both the
   non-goals and the source register, with an objective `permitsRedistribution` criterion.
4. **GDSC moved to "terms-pending."** Rather than assuming GDSC is freely redistributable, it is an
   availability-flag-only source until terms are verified (source table + open question).
5. **Donor-privacy rule made specific.** Added the "published-only, no enrichment, no linkage,
   escalate unusual records" rule for cell-line donors — addressing that lines come from real
   (often pediatric) patients (Data/licensing; Security & privacy).
6. **Misidentification surfaced first.** Authentication/misidentification status is the first block
   in every entry and a 100%-coverage success metric, cross-checked against ICLAC *and* Cellosaurus.
7. **RRID chosen as the explicit identity anchor.** Resolves the "merge two different models" risk;
   resolver golden tests include a deliberate cross-resource conflict.
8. **Cross-source disagreement handled, not hidden.** Default rule added: show all sources with
   provenance, never silently pick a value (Solution approach; Open questions).
9. **"Research observation, not advice" caveat made standing and per-assertion.** Added to the
   dependency field, the guardrails, and the risk table — the catalog cannot drift into clinical
   claims.
10. **Explicit "indirect benefit" honesty.** Problem section now states the family benefit is
    upstream/indirect and refuses to let disease gravity justify clinical-sounding claims.
11. **No-recompute boundary drawn vs. sibling projects.** Non-goals now cite
    `ewing-expression-reanalysis` and `ewing-fusion-detection-benchmark` as the owners of
    re-derivation; this catalog references, never recomputes.
12. **Release pinning + refresh model.** Added `compiledFromReleases`, pinned-release decision, and
    a quarterly DepMap refresh task (Solution approach; Sustainability; risk table).
13. **Completeness score (0–100) with before/after.** Makes "complete entry" checkable and
    DoD-verifiable (Success metrics).
14. **Outcome artifact + per-channel acceptance defined.** `outcomes/<event-id>.json` and explicit
    upstream-acceptance evidence per resource (Success metrics; Definition of Shipped).
15. **Self-serve cold-start path.** M0's first real outcome is an accepted upstream correction,
    which does not depend on first signing a partner — de-risks the cold start.
16. **Reviewer roles split and sequenced.** Separated License+privacy reviewer from Domain
    reviewer; both named before pilot review; rotation rule to avoid single-person bottleneck
    (Governance).
17. **Risk tier corrected to low with tagged medium sub-tasks and explicit "no high tasks."**
    Clarifies that nothing here is patient-facing (Quality gates).
18. **No-re-host rule.** We reference bulk source matrices at pinned versions and only redistribute
    small derived CC-BY aggregates — avoids mirroring/license overreach (Scope; Solution approach).
19. **Vendor-catalog handling.** ATCC/DSMZ/COG entries are fact + link only; no copied descriptive
    prose (avoids copyright + for-profit-benefit issues).
20. **Deliverable type corrected to `dataset` for catalog entries** (vs. the datasheets project's
    document-only stance), since this project genuinely produces a derived, redistributable metadata
    dataset — reflected in TASKS field mapping.
21. **Consistency-validation golden fixtures specified.** Resolver/validator/aggregator each ship
    synthetic golden fixtures, including failing cases, exercised in CI (Quality gates).
22. **MODEL-ROSTER.md as the candidate funnel.** A license-classified, RRID-keyed candidate roster
    feeds per-model tasks through the gate — mirrors the proven datasheets catalog→gate→task pattern.
23. **Emotional-pressure risk row added.** Names the specific failure mode (pediatric-cancer gravity
    pushing scope toward patient guidance) with a mitigation owner.
24. **Dependency aggregation sequenced after authentication.** M2 depends on M1 so dependencies are
    never aggregated for a line before its misidentification status is known (Roadmap dependencies).
25. **Attribution + dual licensing made explicit.** Metadata/aggregates CC BY 4.0, code MIT, with
    mandatory attribution to DepMap/Cell Model Passports/Cellosaurus/ICLAC (Data/licensing).

---

## Review sign-off

**Reviewed for completeness and correctness against PLAN_SPEC (17 sections), CLAUDE.md guardrails,
the good-deed definition, the Track-8 cancer guardrails, and the Task JSON schema.**

- **Structure:** all 17 required H2 sections present and in order; metadata header present;
  Data/licensing section leads with the binding cancer guardrails as required. ✔
- **Cancer guardrails:** open-access-only enforced; controlled-access (dbGaP/EGA/biobanks)
  categorically out of scope; COSMIC/OncoKB flagged link-only; GDSC terms-pending; no medical
  advice / no therapeutic claims; provenance on every assertion. ✔
- **Honesty:** `verifiedNeed:false` until a beneficiary is confirmed; partner/reviewers "TO BE
  SECURED"; benefit to families stated as indirect/upstream; no invented partner. ✔
- **Quality bar:** "delivered, not merged" encoded as Definition of Shipped (verifiable reuse or
  accepted upstream correction, not mere production); outcome artifacts defined. ✔
- **Schema mapping:** TASKS.md maps every field; `deliverable: dataset` justified; risk tiers
  applied (low project, medium sub-tasks, no high); example Task JSON validates against the schema
  (required fields present, enums respected). ✔
- **Corrections made during review:** (a) confirmed M2's dependency aggregation lists its M1
  dependency so no line is dependency-profiled before authentication; (b) ensured every
  dependency-bearing task references the standing caveat; (c) verified the example JSON `type`/
  `deliverable`/`riskTier` enums are legal; (d) confirmed no task is `high` and no task ingests
  controlled-access data.
- **Residual human decisions (flagged, not resolved):** name the License+privacy and Domain
  reviewers; secure a research consumer/partner; confirm the pinned DepMap release's exact license;
  resolve GDSC redistribution terms; decide donor age-band policy.

**Sign-off:** Plan is internally consistent and ready for maintainer review. No blocking
inconsistencies remain; outstanding items are genuine human/governance decisions, correctly surfaced
rather than assumed.
