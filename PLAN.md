# PLAN — ewing-cell-line-catalog

> Status: Draft · Version: 0.2.0 · Last updated: 2026-06-29 · Owner: TBD (maintainer) · Lane: donated · Risk tier: low (with medium-risk sub-tasks, explicitly tagged)

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

This catalog is built **on** the authoritative resources, not in competition with them: Cellosaurus
remains the identity authority (and the machine-readable surface over the ICLAC misidentification
register), DepMap/CCLE remains the dependency-data authority, and ICLAC remains the
misidentification authority. The contribution is the **Ewing-focused, RRID-anchored,
misidentification-aware join** that no pan-cancer authority offers at disease depth — and it is
careful to **distinguish authenticated identity (STR-supported) from merely asserted identity**, and
to never assert an authentication the underlying data does not support. (See *Competitive landscape &
differentiation*.)

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
captured at triage. The pair is stored in the entry's provenance artifact. The score is **weighted**:
`identity`, `authentication`, and `fusion` are **mandatory-to-ship** — a record cannot reach the
≥ 90 gate while missing any of them, even if every minor field (e.g. `culture`) is full.

**What "accepted upstream" means and the evidence the Steward records.** A single canonical
**outcome artifact** per event (`outcomes/<event-id>.json`: channel, URL/permalink, timestamp,
before/after detail). Acceptance is defined per channel: Cellosaurus correction = the maintainer's
applied change / acknowledgement reference; Cell Model Passports = applied correction or accepted
issue; DepMap = accepted community-feedback reference; third-party reuse = a citation, a fork that
builds on the catalog, or an issue/PR referencing it. Self-reported reuse never counts.

**Primary vs bonus outcome signal.** Upstream acceptance is **exogenous and slow** (it runs on the
maintainers' timeline), so it is held as a **goal/bonus**, not the sole gating signal — the
**primary delivered-outcome signal is catalog adoption by a real research consumer**, with accepted
upstream corrections counting as additional, compounding good. M3 is structured so it cannot be held
indefinitely open waiting on an upstream queue.

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

## Competitive landscape & differentiation

The relevant resources are authorities to build **on**, not duplicate. The whitespace is a
disease-scoped, misidentification-aware, omics-availability-linked, fully-provenanced **open** catalog
— which none of them provide.

- **Cellosaurus (SIB) — the identity authority, our anchor (not a threat).** Describes >100,000 cell
  lines; issues the `CVCL_`/RRID identifiers; aggregates synonyms, donor info, STR profiles, and
  **machine-readable misidentification/"problematic" flags** (re-encoding ICLAC status); CC BY 4.0;
  downloadable + API. *Gap we fill:* breadth over depth — no Ewing-specific curation, no linkage to
  *which* open omics/dependency data exist. We build **on** Cellosaurus as the canonical identity key
  and the machine surface over ICLAC.
- **DepMap / CCLE (Broad) — the dependency-data authority.** Largest compendium of genome-scale
  CRISPR (Chronos gene-effect), expression, CN, open-tier mutations, PRISM drug screens; public
  releases CC BY 4.0 on Figshare (pin the exact per-release DOI). *Gap we fill:* pan-cancer, not
  disease-curated; per-gene/per-line UX, not "give me the trustworthy Ewing roster with
  authentication + fusion + what data exists"; no misidentification curation; release-to-release model
  churn is the user's problem.
- **Cell Model Passports (Wellcome Sanger) — curated model hub** (>2,000 models, REST API,
  cross-refs Cellosaurus). *Gap we fill:* pan-cancer; **per-dataset licensing is mixed (not uniformly
  CC BY)** — a higher-risk license assumption than DepMap; no consolidated misidentification view.
- **ICLAC Register of Misidentified Cell Lines — the misidentification authority** (v14, 15 Feb 2026,
  608 lines: 560 with no authentic stock, 48 with authentic stock found), distributed as a versioned
  **PDF, not an API**. *Gap we fill:* not per-disease; no omics/identity linkage; manual ingestion.
- **COSMIC Cell Lines Project (Sanger) — deep mutation curation, but non-commercial /
  no-redistribution license** (free for academic use with registration; commercial license
  otherwise). **Link-only here — even *derived* COSMIC counts are not emitted.**
- **ATCC / DSMZ public catalogs — sourcing/availability + vendor STR.** Copyrighted catalog prose and
  vendor-licensed STR tables; vendor-scoped; no cross-resource identity or dependency view. **Fact +
  link only; vendor STR is link-only, never redistributed.**

**Differentiator (single strongest):** a per-model, versioned **"trust + data-availability passport"
for Ewing** — every assertion (identity, authentication-with-axis, fusion-with-breakpoint, what open
data exists, what dependencies are reported) carried with **field-level provenance, source version,
and per-source license** — turning four portals + one PDF into one citable, machine-readable, openly
(CC-BY) licensed object. Supporting differentiators: **misidentification-aware by construction** (with
the ICLAC-vs-reclassification distinction the big resources blur); **provenance + license as
first-class data, not prose**; an **upstream-correction loop**; **honest, descriptive framing** with
standing "research observation, not advice" caveats; and **refresh discipline** (pinned releases +
quarterly diff) that the static PDF/portal status quo lacks.

## Solution approach & architecture

This is a **content/data-curation project with light, dependency-minimal software**. It assembles a
derived metadata catalog from open sources; it is **not** a pipeline that moves or recomputes raw
omics.

**Canonical record model (source of truth).** One internal record per model; all outputs (JSON,
CSV, human-readable site) are *projections* of it. Every value-bearing field is a `{value,
source, sourceVersion, retrievedAt, license}` provenance cell (or an array of them where multiple
sources agree/disagree). Fields:

- `id` (internal slug); `identity { depmapId (ACH-…), rrid (the **primary** Cellosaurus `CVCL_`
  accession; the `RRID:` prefix is normalized off/on consistently), secondaryAccessions[] (merged
  Cellosaurus accessions, so a later upstream merge doesn't silently break the anchor),
  cellModelPassportId (SIDM…), ccleName (e.g. `A673_BONE` — a legacy, release-unstable string;
  **recorded but NEVER a resolution anchor**), synonyms[] }`. Each cross-ID is tagged **`direct`**
  (asserted by that resource itself) vs **`inherited`** (copied from another resource — DepMap,
  Sanger and Cellosaurus already cross-reference each other), so apparent "independent agreement" is
  not mistaken for corroboration.
- `authentication` — **surfaced first** in every entry — splits the two axes the big resources blur:
  - `strProfile { markers[], source, license }` — the actual marker table is stored **only when the
    source license permits redistribution** (Cellosaurus, CC BY 4.0); vendor STR (ATCC/DSMZ) is
    `available:true` + **link-only**, never copied. (Replaces the old `strProfileAvailable` boolean,
    which under-delivered.)
  - `misidentified` — an **enum**: `clean | misidentified_no_authentic | authentic_stock_exists |
    contaminated` — read from **Cellosaurus's machine-readable "problematic" flag**, citing the
    **ICLAC register vN as the underlying authority**. (Distinguishes ICLAC's "no authentic stock"
    from "authentic stock since found.")
  - `diagnosisReclassified` — a **separate** disease-of-origin-history axis (NOT misidentification):
    e.g. **A673 is authentic patient cells, originally called rhabdomyosarcoma in 1973, later
    reclassified as Ewing sarcoma via EWS-FLI1/cytogenetics**. Conflating this with `misidentified`
    would wrongly flag the project's flagship pilot.
  - `misidentifiedRegister (ICLAC ref + version)`, `contaminationNotes`, `authenticationSource`.
  - **A misidentification label is only ever quoted from ICLAC/Cellosaurus with version — never an
    independent determination** (publicly asserting a line is misidentified is reputationally
    consequential).
- `disease { oncotreeCode (e.g. EWS), oncotreeName, lineage (bone), subtype }`
- `fusion { partner5p (EWSR1), partner3p (FLI1|ERG|other), breakpointExons (controlled canonical
  form, e.g. `EWSR1 ex7 :: FLI1 ex6`), legacyType (annotated secondary field, e.g. "Type 1",
  separately cited), evidence[] }` — non-canonical **"Ewing-like"** fusions (CIC-rearranged, BCOR)
  are now classified as *distinct* entities; the roster gate decides explicitly whether each is in
  (flagged-distinct) or excluded.
- `donor { sexAtSampling, ageAtSampling, siteOfSampling, sampleType (primary|metastasis|relapse) }`
  — **populated only from values already published by Cellosaurus/Cell Model Passports**; never
  inferred, never enriched, never cross-linked to external person data
- `culture { medium, notes }` (where published)
- `availability[] { repository (ATCC|DSMZ|COG|...), catalogNumber, url }` — **link + fact only**, no
  copied vendor prose
- `omicsAvailability { expression, copyNumber, mutationsOpenTier, fusions, drugSensitivity }` each
  `{ available (`true | false | unknown` — **`unknown` (not-checked) ≠ confirmed-absent**), datasetId,
  releaseVersion, url, lastVerifiedRelease }`. Availability is verified by **manifest-diffing** the
  release model lists (the only scalable, auditable method — it also feeds the quarterly refresh),
  not by ad-hoc per-line lookup. (A line can have expression but no CRISPR, or be screen-dropped in a
  later release.)
- `dependencies { source: "DepMap CRISPR (Chronos)", release, selectiveDependencies[] {gene,
  geneEffect, caveat}, methodologyRef, disclaimer }` — research observation, not guidance
- `license` per source-block; `completenessScore { before, after }` — **weighted**: `identity`,
  `authentication`, and `fusion` are **mandatory-to-ship** and cannot be traded against
  `culture`/minor fields before the ≥ 90/100 gate applies; `revisionHistory[]` (per-entry change log);
  `provenance { compiledAt, compiledFromReleases[] }`. The catalog as a whole carries a semantic
  `catalogVersion` so downstream reusers can diff against — and cite — a stable snapshot.

**Pipeline (per model).**
1. **Triage & gate** — confirm each contributing source is open-access and permits derivative
   redistribution under our output license; confirm no controlled-access data; confirm donor fields
   are limited to already-published values. PASS/FLAG/EXCLUDE recorded as a committed artifact.
2. **Identity resolution** — resolve and cross-check DepMap ID ↔ primary CVCL accession (Cellosaurus)
   ↔ Cell Model Passports ID, tagging each cross-ID `direct` vs `inherited`; record conflicts and any
   merged/secondary accessions. The **primary CVCL accession is the cross-resource anchor**;
   `ccleName` is never used to resolve.
3. **Authentication check** — read misidentification/contamination status from **Cellosaurus's
   machine-readable "problematic" flag** and reconcile it against the **ICLAC register (cited as the
   underlying authority, by version)** — treating Cellosaurus as the machine surface over the ICLAC
   PDF, not as an independent second lookup. Record the `misidentified` enum AND the separate
   `diagnosisReclassified` axis prominently; never assert a misidentification the sources do not.
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
deliberate refresh task): the **exact DepMap public-release Figshare DOI** (not "DepMap public" —
the license is CC BY 4.0 *per release*, e.g. DepMap 24Q4 Public; pin the DOI), the Cell Model
Passports release number/date, the Cellosaurus release number, and the **ICLAC vN PDF (SHA-256 +
Wayback save URL)**. Any drift is an isolated refresh task, never a silent change.

**Claude API leverage (assistive only; every output human-verified, every fact provenanced).**
Claude is used to *accelerate* curation, never to *decide* identity, misidentification, or license:
- **Name/synonym harmonization & candidate identity resolution** — propose synonym clusters
  (`A-673/A673/A 673`; `TC-71/TC71`; `CHLA-9/10/25`) and *candidate* DepMap↔CVCL↔SIDM mappings for the
  resolver, **flagging conflicts for human confirmation**.
- **Literature fact-extraction with citations** — extract fusion partner + exon breakpoint, donor
  fields *as published*, culture conditions, and derivation/reclassification history from open-access
  papers (PMC-OA), emitting `{value, quote, PMID, section}` straight into the field-level provenance
  cells.
- **Discrepancy / drift detection** — cross-compare a line's fusion/disease/donor across DepMap, Cell
  Model Passports, and Cellosaurus and **surface disagreements** as review items, never silently
  picking a winner.
- **License-clause triage assistant** — read a source's terms page and *draft* a
  `permitsRedistribution` rationale with the quoted clause + URL **for the License+privacy reviewer
  to ratify** (Claude drafts; the human makes the binding PASS/FLAG/EXCLUDE call).
- **Release-diff summarization** — summarize what changed for Ewing lines between DepMap releases to
  drive the quarterly refresh.

**Hard guardrails on Claude's role** (schema- and review-enforced): identity & misidentification
determinations are **authoritative-source-only** (read verbatim from ICLAC + Cellosaurus with
version, never an LLM judgement — and never assert an authentication the data doesn't support); **no
fabricated or inferred provenance** (a fact without a citable open source stays empty — never a
synthesized PMID/breakpoint/donor value); license calls are **human-ratified**; **no donor
enrichment/re-identification** and **no clinical/therapeutic interpretation** of dependencies,
regardless of model confidence; diagnosis-of-origin reclassification (the A673 case) is asserted only
from primary literature + Cellosaurus, never inferred from fusion presence alone.

**Key decisions (locked).**
- **Canonical-model-first**: never hand-maintain parallel JSON/CSV/site formats.
- **Provenance is mandatory at the field level**, not the record level: an uncited assertion fails
  review.
- **The primary Cellosaurus CVCL_ accession is the identity anchor** across resources (RRID prefix
  normalized; `secondaryAccessions[]` captured; `ccleName` never an anchor; cross-IDs tagged
  `direct` vs `inherited`).
- **Authentication is two axes, not one**: `misidentified` (ICLAC enum, via Cellosaurus's machine
  surface) is separate from `diagnosisReclassified` (disease-of-origin history). A misidentification
  label is always quoted from the authority with version, never independently determined.
- **Authenticated (STR-supported) identity is distinguished from merely asserted identity**; the
  catalog never claims an authentication the data does not support.
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
| **DepMap (Broad)** — CRISPR gene-effect (Chronos), CCLE expression/CN/mutations(open-tier)/fusions, model metadata | dependency aggregates, omics-availability, IDs | CC BY 4.0 — confirmable **per release** (e.g. DepMap 24Q4 Public on Figshare) | ACCEPT (attribution; **pin the exact Figshare DOI per release**) |
| **Cell Model Passports (Sanger)** | model metadata, IDs, omics availability | **per-dataset, mixed** (NOT uniformly CC BY; some components historically academic-login-gated) — **VERIFY-HIGH-RISK at gate** | ACCEPT *only for fields whose terms clear redistribution*; else downgrade to link-only |
| **Cellosaurus (SIB)** | primary CVCL accession + secondary accessions, synonyms, donor (published-only), **machine-readable misidentification ("problematic") flags re-encoding ICLAC**, STR profiles | CC BY 4.0 | ACCEPT (attribution; **identity anchor + machine surface over ICLAC**) |
| **ICLAC Register of Misidentified Cell Lines** | misidentification/contamination status | published register (versioned **PDF, not an API**; terms **VERIFY at gate**; attribute) | ACCEPT as factual flag (cite **register vN**; read status via Cellosaurus, cite ICLAC as authority) |
| **COSMIC / Cell Lines Project (Sanger)** | mutation calls | non-commercial / custom (free academic w/ registration; **prohibits redistribution**) | **FLAG — link-only, NOT redistributed (even *derived* counts are not emitted)** |
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
catalog — not the underlying source data — is our contribution. Because aggregating multiple CC BY
sources requires per-source attribution to survive into **every** projection, the **CSV/flat export
is the likely failure point**: each projection must carry machine-readable source+license per field,
or a row-level/source manifest, so attribution is never silently dropped on flat export.

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
- Exit criteria: (1) canonical record schema + field-level provenance model published — **including
  the split `authentication` (ICLAC enum + separate `diagnosisReclassified`), `strProfile{}`,
  identity `secondaryAccessions[]` + `direct`/`inherited` cross-ID tagging, the controlled fusion
  representation, and the weighted completeness score**; (2) source register with verified per-source
  licenses published (DepMap Figshare DOI; Cell Model Passports flagged VERIFY-high-risk); (3)
  open-data/license/donor gate checklist exists and is applied to the pilot; (4) identity resolver +
  validator working in CI with golden fixtures (resolver tests `ccleName`-is-not-an-anchor and
  `inherited`-vs-`direct`); (5) one pilot model (A673) catalogued end-to-end at ≥ 90/100 with gate
  artifact — **A673 verifies the two-axis design (authentic cells, `diagnosisReclassified` from RMS
  to Ewing, NOT flagged `misidentified`)**; (6) License+privacy reviewer and Domain reviewer
  **named**; (7) ≥ 1 beneficiary/partner outreach thread opened.

**M1 — Authenticated core catalog.**
- Goal: catalog the most widely used authenticated Ewing models, with misidentification audit and
  fusion annotation.
- Exit criteria: (1) authoritative roster compiled and gate-triaged in `MODEL-ROSTER.md`, with an
  **explicit in/out decision on non-canonical "Ewing-like" entities (CIC-rearranged, BCOR)**; (2)
  ≥ 10 models fully catalogued at ≥ 90/100; (3) 100% of catalogued models carry the `misidentified`
  enum status (read from Cellosaurus, citing ICLAC vN) **and** the `diagnosisReclassified` axis; (4)
  fusion status annotated in the controlled `EWSR1 ex_ :: partner ex_` form (+ legacy type label),
  with provenance, for every catalogued model; (5) ≥ 1 upstream correction submitted (accepted counts
  toward M3).

**M2 — Dependencies & omics availability.**
- Goal: add open-dependency aggregates and omics-availability mapping, and publish the catalog.
- Exit criteria: (1) DepMap release pinned **by exact Figshare DOI**; dependency aggregator working
  in CI with golden fixtures; (2) omics-availability map populated for the core roster at per-datatype
  granularity (`available` true/false/`unknown`, `datasetId`, `releaseVersion`, `lastVerifiedRelease`),
  **verified by manifest-diffing** the release model lists; (3) dependency summaries (with version +
  methodology + standing caveat) for the core roster; (4) catalog published machine-readable (JSON +
  CSV) and human-readable, all assertions provenanced, **stamped with a semantic `catalogVersion`** —
  shipping a **small stable JSON schema + JSON-LD/RO-Crate context + a ~5-line query example**, and a
  CSV that carries per-field source+license via a **row-level source manifest** so attribution
  survives the flat export; (5) catalog reaches ≥ 20 fully-catalogued models.

**M3 — Upstream corrections, reuse & sustainability.**
- Goal: demonstrate real reuse and an upstream-correction + refresh model.
- Exit criteria: (1) **≥ 1 confirmed research consumer/partner adopting the catalog — the primary
  delivered signal** — plus ≥ 3 verifiable external reuse events; (2) ≥ 2 corrections accepted
  upstream (evidence artifacts recorded) as the **bonus/compounding** outcome, not a hard gate that
  could hold M3 open indefinitely on the maintainers' timeline; (3) refresh process documented for
  quarterly DepMap releases with a named steward; (4) **a misidentification-alert diff** (see
  *Adjacent opportunities*) fires when a catalogued line newly appears on ICLAC/Cellosaurus or is
  dropped/changed in a DepMap release.

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
| **Publishing a wrong/over-stated "misidentified" label** (reputationally consequential false flag) | Low | High | Label is **quoted verbatim from ICLAC/Cellosaurus with version**, never an independent determination; two-axis design separates `misidentified` from `diagnosisReclassified` so A673-type reclassification isn't mis-flagged | Domain reviewer |
| **Cell Model Passports per-dataset license is not uniformly CC BY** → over-redistribution | Medium | High | Flagged **VERIFY-high-risk** (not bucketed with DepMap); per-field redistribution check; downgrade to link-only if terms don't clear | License+privacy reviewer |
| **False "independent agreement" from inherited cross-IDs** inflates identity confidence | Medium | Medium | Tag each cross-ID `direct` vs `inherited`; don't count inherited refs as corroboration; `ccleName` never a resolution anchor | Technical reviewer |
| Wrong cross-resource identity resolution (merging two different models) | Medium | High | Primary CVCL accession as anchor (+ `secondaryAccessions[]`); cross-source consistency checks; conflicts flagged not auto-resolved; resolver golden tests | Technical reviewer |
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

## Adjacent opportunities

The schema + gate + resolver are reusable; several adjacent efforts compound the public good without
expanding this project's scope (each is a **cross-link or a separate project**, not a dependency):

- **Sibling Ewing projects (cross-link by RRID).** `ewing-expression-reanalysis` consumes *this*
  catalog's authenticated + available roster as its trusted input set (never reanalyze a misidentified
  line); `ewing-pdx-model-index` applies the same provenance/gate pattern to PDX/xenograft models
  (with a tighter donor-privacy posture); `ewing-open-data-catalog` is the superset GEO/SRA study
  index this per-model catalog links into; `ewing-single-cell-atlas` cross-links single-cell
  availability; `ewing-fusion-detection-benchmark` can seed from this catalog's harmonized, provenanced
  fusion truth set.
- **A reagent-authenticity registry.** Generalize the schema + gate + resolver into a
  disease-agnostic template for other rare/pediatric cancers (rhabdomyosarcoma, osteosarcoma, DIPG) —
  the Ewing build is the reference implementation.
- **A misidentification-alert feed.** A versioned diff/RSS that fires when a catalogued Ewing line
  newly appears on the ICLAC/Cellosaurus problematic lists, or is dropped/changed in a DepMap
  release — genuinely useful to labs and a recurring outcome generator (wired into M3 and refresh).
- **An MCP server over the published catalog.** A read-only Model Context Protocol server exposing
  `resolveIdentity`, `getAuthentication`, `getOmicsAvailability`, `getDependencies` so any agent can
  query the trust layer — an adoption channel and Claude-leverage surface in one.

## Open questions

- Which named lab/consortium/foundation or upstream maintainer becomes the first confirmed
  beneficiary/partner?
- **DepMap release-terms confirmation:** confirm the exact license of the specific public release we
  pin (CC BY 4.0 is confirmable *per release*; pin the exact Figshare DOI before any aggregate is
  published).
- **Cell Model Passports licensing:** does it actually clear CC-BY redistribution for the specific
  model-metadata fields used, or should it be downgraded to "VERIFY-high-risk / possibly link-only"
  like COSMIC?
- **GDSC drug-sensitivity terms:** confirm whether GDSC permits redistributable aggregates or only
  an availability flag.
- **ICLAC ingestion path:** accept Cellosaurus as the machine surface over ICLAC (citing ICLAC vX as
  the authority), or commit to parsing the ICLAC PDF directly? (Affects refresh cost; default:
  Cellosaurus as the machine surface, reconcile against ICLAC.)
- **Diagnosis-reclassification vs misidentification:** confirmed as *two* schema axes so A673
  (authentic, reclassified) is never flagged misidentified — re-validate the boundary as the roster
  grows.
- **Non-canonical "Ewing-like" entities** (CIC-rearranged, BCOR): in scope as *flagged-distinct*, or
  excluded at the roster gate?
- **STR redistribution:** publish actual STR marker tables where Cellosaurus permits, or stay
  availability-flag-only to avoid any vendor-STR rights question?
- **Cross-ID independence:** how to record/penalize `inherited` vs `direct` cross-references so the
  "resolved across three systems" metric isn't inflated by shared upstream assertions?
- **Catalog versioning & DOI:** mint a Zenodo DOI per catalog release for citeable adoption, or rely
  on git tags + `catalogVersion`?
- **Outcome metric balance:** confirmed — adoption-by-a-consumer is the primary delivered signal,
  with ≥ 2 *accepted* upstream corrections as a bonus rather than a hard M3 gate.
- **Variant/derivative lines** (e.g. A673 sublines, luciferase/Cas9 derivatives): one entry with
  derivatives, or separate entries? Affects identity resolution and donor inheritance.
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
- DepMap / Cancer Dependency Map (Broad Institute) — CRISPR gene-effect (Chronos), CCLE; public
  releases CC BY 4.0 on Figshare (pin exact per-release DOI, e.g. DepMap 24Q4 Public)
- Cell Model Passports (Wellcome Sanger Institute) — REST API; **per-dataset, mixed licensing**
- Cellosaurus (SIB) — primary `CVCL_`/RRID identifiers, machine-readable misidentification
  ("problematic") flags re-encoding ICLAC, STR profiles (CC BY 4.0)
- ICLAC Register of Misidentified Cell Lines — versioned PDF (v14, 15 Feb 2026; 608 lines)
- COSMIC Cell Lines Project (Sanger) — non-commercial / no-redistribution license (link-only)
- A673 diagnosis-reclassification (rhabdomyosarcoma → Ewing via EWS-FLI1/cytogenetics) — primary
  literature (e.g. PubMed 12606131) + Cellosaurus/ECACC
- EWSR1-ETS fusion breakpoint nomenclature (Type 1 vs exon-pair vs HGVS) — e.g. PMC3575406
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

- **Structure:** all required H2 sections present and in order, plus two strategy sections added in
  v0.2 (*Competitive landscape & differentiation*, *Adjacent opportunities*); metadata header
  present; Data/licensing section leads with the binding cancer guardrails as required. ✔
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

---

## Changelog — v0.2 (analysis merged)

Merges `COMPETITIVE-ANALYSIS.md` into the plan. Surgical/additive; guardrails preserved and
strengthened, never weakened. No invented facts — every new claim traces to the analysis or its cited
sources.

**Correctness / schema / license fixes applied**

1. **Identity anchored to the *primary* Cellosaurus CVCL accession** (RRID prefix normalized), with
   `secondaryAccessions[]` captured so an upstream merge can't silently break the anchor; `ccleName`
   recorded but **never a resolution anchor**; each cross-ID tagged **`direct` vs `inherited`** to
   stop inherited DepMap/Sanger/Cellosaurus cross-refs from posing as independent corroboration.
2. **Authentication split into two axes**: `misidentified` (ICLAC **enum** `clean |
   misidentified_no_authentic | authentic_stock_exists | contaminated`, read from Cellosaurus's
   machine-readable "problematic" flag, citing **ICLAC vN** as authority) **vs** `diagnosisReclassified`
   (disease-of-origin history) — so the flagship pilot **A673** (authentic cells, RMS→Ewing
   reclassification) is not wrongly flagged misidentified.
3. **STR promoted from a boolean to `strProfile { markers[], source, license }`** — actual marker
   tables redistributed **only** from CC-BY sources (Cellosaurus); vendor STR (ATCC/DSMZ) is link-only.
4. **Misidentification label is quoted-with-version only**, never an independent determination
   (false-flag liability) — added as a guardrail, a risk row, and a DoD expectation.
5. **Controlled fusion representation** (`EWSR1 ex7 :: FLI1 ex6`) with the legacy "Type N" label as a
   separately-cited secondary field; **non-canonical CIC/BCOR "Ewing-like" entities** get an explicit
   roster-gate in/out decision.
6. **Omics availability granularity**: per-datatype `{available (true|false|unknown), datasetId,
   releaseVersion, url, lastVerifiedRelease}` with `unknown ≠ confirmed-absent`, verified by
   **manifest-diffing** release model lists.
7. **License classes segregated**: DepMap CC BY 4.0 pinned by **exact Figshare DOI per release**;
   **Cell Model Passports flagged VERIFY-high-risk** (per-dataset, not uniformly CC BY) and split out
   from DepMap; **COSMIC non-commercial → link-only, even derived counts not emitted**; CC-BY
   attribution must **survive into the CSV/flat projection** via a row-level source manifest.
8. **Weighted completeness score** (identity + authentication + fusion mandatory-to-ship);
   **`catalogVersion` (semver) + per-entry `revisionHistory[]`** added for citeable, diffable
   snapshots.

**Strategy integrated**

9. New **## Competitive landscape & differentiation** (Cellosaurus = build-on authority + machine
   surface over ICLAC; DepMap/CCLE; Cell Model Passports; ICLAC PDF; COSMIC non-commercial;
   ATCC/DSMZ) with the single strongest differentiator: a per-model, versioned, RRID-anchored
   "trust + data-availability passport" for Ewing.
10. **Claude API leverage folded into the architecture** (synonym/identity candidates, literature
    fact-extraction with citations, discrepancy detection, license-clause drafting, release-diff) —
    all assistive, human-verified, with hard guardrails: identity/misidentification/license decisions
    stay authoritative-source + human; no fabricated provenance; never assert an authentication the
    data doesn't support.
11. **Optimizations folded into the Roadmap** (schema splits at M0; non-canonical-entity gate +
    controlled fusion form at M1; Figshare-DOI pin, manifest-diffing, `catalogVersion`, JSON-LD/
    RO-Crate context + query example + CSV source manifest at M2; adoption-primary + alert-feed at M3).
12. New **## Adjacent opportunities** (cross-links to `ewing-expression-reanalysis`,
    `ewing-pdx-model-index`, `ewing-open-data-catalog`, `ewing-single-cell-atlas`,
    `ewing-fusion-detection-benchmark`; a disease-agnostic **reagent-authenticity registry**; a
    **misidentification-alert feed**; an **MCP server** over the published catalog).
13. **Open questions merged** (ICLAC ingestion path, two-axis confirmation, CIC/BCOR scope, STR
    redistribution, Cell Model Passports licensing, cross-ID independence, catalog DOI, outcome-metric
    balance, variant/derivative lines).
14. **Outcome signal rebalanced**: catalog adoption is the **primary** delivered signal; ≥ 2 accepted
    upstream corrections are a **bonus** (exogenous/slow), so M3 can't hang on an upstream queue.

**Preserved unchanged:** the cancer guardrails (open-data-only; controlled-access categorically out
of scope; per-source license verify; provenance on every assertion; research-not-advice), the
17-section spine, the honesty posture (`verifiedNeed:false`, TO-BE-SECURED roles), and all valid v0.1
content (including Appendix A, which records the v0.1 improvements).
