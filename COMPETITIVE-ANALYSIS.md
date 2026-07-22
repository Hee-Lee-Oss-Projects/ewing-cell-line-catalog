# Competitive & Improvement Analysis — `ewing-cell-line-catalog`

Analyst review of the Hee-Lee Oss good-deed project. Scope: an open, curated, fully-provenanced
metadata catalog of Ewing sarcoma cell-line models — identity/STR, fusion type, source/availability,
open-omics availability, misidentification flags — cross-linking authoritative resources
(Cellosaurus, DepMap/CCLE, Cell Model Passports, ICLAC, ATCC/DSMZ). Cell lines are de-identified
reagents (low re-identification risk), but **provenance, misidentification, and aggregated-metadata
licensing** are the live issues. Competitor claims below are grounded in cited public sources.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually mature: field-level provenance, a blocking open-data/license gate, RRID as the
identity anchor, misidentification surfaced first, dependency caveats, and an honest
`verifiedNeed:false` posture. It is one of the stronger plans in the portfolio. The gaps below are
real and mostly about **authoritative-source mechanics, STR/identity rights, and a few metric/scope
soft spots** — not structural flaws.

**A. Identity / Cellosaurus accession handling**

1. **CVCL accession vs. RRID conflation.** The plan uses "RRID (Cellosaurus CVCL_)" as one anchor.
   These are *related but not identical*: the Cellosaurus primary accession is `CVCL_xxxx`; the RRID
   form is `RRID:CVCL_xxxx`. Cellosaurus also carries **secondary/merged accessions** when two
   entries are unified. The schema's `rrid` field should store the *primary* CVCL accession,
   normalize the `RRID:` prefix, and explicitly capture `secondaryAccessions[]` so a later merge in
   Cellosaurus doesn't silently break the anchor. Not addressed.
2. **DepMap ↔ Sanger ID provenance.** The plan treats DepMap ID (`ACH-`) and Cell Model Passports ID
   (`SIDM`) as independent anchors to cross-check against RRID. In reality DepMap (Broad) and Cell
   Model Passports (Sanger) **already cross-reference each other and Cellosaurus**, so an apparent
   "independent agreement" may be a shared upstream assertion, not corroboration. The provenance
   model should record whether a cross-ID came from the resource *directly* or was *inherited* from
   another resource — otherwise the "resolved across all three" success metric overstates
   independence.
3. **CCLE name instability.** `ccleName` (e.g. `A673_BONE`) is a legacy tissue-suffixed string that
   has changed across releases; it is a weak join key. Fine to record, but it must never be a
   resolution anchor. The plan lists it in `identity` without that caveat.

**B. STR profile sourcing and rights (under-specified — material gap)**

4. The plan's `authentication.strProfileAvailable` is only a **boolean**. But STR is the heart of
   "authentication," and the plan never says *where the STR profile comes from or whether the actual
   profile can be redistributed*. Cellosaurus publishes STR profiles for many lines (CC BY 4.0), but
   vendor STR (ATCC/DSMZ) is often **copyrighted catalog content** — the plan correctly forbids
   copying vendor prose but is silent on vendor STR *tables*. Recommendation: store
   `strProfile { markers[], source, license }` only when the source license permits redistribution
   (Cellosaurus), else `available:true` + link-only. Without this, the catalog either under-delivers
   (boolean only) or risks redistributing vendor-licensed STR.

**C. Misidentification / ICLAC flagging**

5. **ICLAC is a PDF register, not an API.** The current ICLAC Register is **version 14 (15 Feb 2026),
   608 lines**, distributed as a versioned PDF ([ICLAC](https://iclac.org/databases/cross-contaminations/)).
   The plan pins "ICLAC register version" but doesn't acknowledge that ingestion is **manual
   PDF parsing** with no machine endpoint — a real maintenance cost and a refresh-fragility the
   roadmap should budget for. Cellosaurus *re-encodes* ICLAC status in a machine-readable form
   (`Problematic cell line` comments), so the practical pattern is: **read misidentification status
   from Cellosaurus, cite ICLAC as the underlying authority.** The plan treats them as two
   independent lookups; it should treat Cellosaurus as the machine surface over ICLAC and reconcile.
6. **Two distinct "authentication" axes are collapsed.** ICLAC lists *cross-contaminated/misidentified*
   lines (wrong cell of origin). Separately, a line can be **authentic but historically misdiagnosed**
   — the canonical Ewing example is the pilot itself: **A673 was described in 1973 as a
   rhabdomyosarcoma and later reclassified as Ewing sarcoma via EWS-FLI1 / cytogenetics**
   ([PubMed 12606131](https://pubmed.ncbi.nlm.nih.gov/12606131/),
   [ECACC](https://www.culturecollections.org.uk/products/celllines/generalcell/detail.jsp?refId=85111504&collection=ecacc_gc)).
   That is *not* a misidentification in the ICLAC sense — the cells are the right patient's cells; the
   original diagnosis was wrong. The schema needs to separate `misidentified` (ICLAC/contamination)
   from `diagnosisReclassified` (disease-of-origin history). Conflating them would wrongly flag A673,
   the project's flagship pilot.
7. ICLAC also distinguishes **"misidentified, no authentic stock"** from **"authentic stock since
   found"** (560 vs 48 of the 608). The boolean `misidentified` can't represent the second state.
   Recommend an enum: `clean | misidentified_no_authentic | authentic_stock_exists | contaminated`.

**D. Fusion-type accuracy**

8. Fusion breakpoint nomenclature is **non-uniform** in the literature: "Type 1" (EWSR1 ex7–FLI1 ex6)
   vs HGVS-style transcript breakpoints vs exon-pair notation are all in use
   ([PMC3575406](https://pmc.ncbi.nlm.nih.gov/articles/PMC3575406/)). The schema's free-text
   `breakpointType` invites inconsistency. Recommend a controlled representation: 5'/3' partner +
   exon pair (e.g. `EWSR1 ex7 :: FLI1 ex6`) plus the *legacy* type label as a separate annotated
   field, each with its own evidence citation. Also: a few "Ewing-family" lines carry **non-canonical
   fusions** (e.g. CIC-rearranged, BCOR) now classified as *distinct* entities, not Ewing — the
   roster gate should decide explicitly whether those are in or out (currently ambiguous).

**E. Omics-availability mapping**

9. "Available (bool)" is too coarse to be trustworthy. DepMap/CCLE may have *expression but not
   CRISPR* for a given line, or a line may be **screen-failed/dropped** in a later release. Recommend
   per-datatype `{available, datasetId, releaseVersion, url, lastVerifiedRelease}` and an explicit
   `absent` vs `unknown` distinction (not-checked ≠ confirmed-absent).
10. The plan does not say **how availability is verified** — by parsing the release manifest/model
    list, or by per-line lookup. Manifest-diffing is the only scalable, auditable method and should
    be named as the mechanism (also feeds the quarterly refresh).

**F. Metadata-license aggregation**

11. **DepMap license is confirmable as CC BY 4.0** for recent public releases (e.g.
    [DepMap 24Q4 Public on Figshare](https://plus.figshare.com/articles/dataset/DepMap_24Q4_Public/27993248)),
    so the plan's "VERIFY at gate" is correct and now de-riskable — but note it is *per-release*; pin
    the exact Figshare DOI, not "DepMap public."
12. **Cell Model Passports licensing is the weakest assumption.** Sanger's DepMap/Cell Model Passports
    data has **per-dataset terms** and historically *non-CC, academic-login* gating on some
    components; the plan's "CC BY 4.0 — VERIFY" may not hold for all model-metadata fields. This is a
    higher-risk "VERIFY" than DepMap and should be flagged as such, not bucketed with DepMap.
13. **CC BY *aggregation* of multiple CC BY 4.0 sources still requires per-source attribution to
    survive into every projection (JSON/CSV/site).** The plan mandates attribution but the CSV
    projection is the likely failure point (attribution often dropped in flat exports). Add an
    explicit acceptance criterion: every projection carries machine-readable source+license per field
    or a row-level/source manifest.
14. **COSMIC Cell Lines Project** is correctly link-only: COSMIC is **free for academic use with
    registration but explicitly prohibits redistribution** and requires a commercial license
    otherwise ([COSMIC non-commercial terms](https://cancer.sanger.ac.uk/cosmic/license?genome=37)).
    The plan's disposition is right; just ensure even *derived* COSMIC counts aren't emitted.

**G. Versioning / metrics / governance soft spots**

15. **No catalog-level semantic version or per-entry change history.** Records pin source releases but
    there's no `catalogVersion` or per-entry `revisionHistory` so downstream reusers can diff. For a
    resource whose whole value is trust, this is a notable omission.
16. **"≥2 corrections accepted upstream" is exogenous and slow.** Cellosaurus/Sanger acceptance is on
    *their* timeline; gating M3 success on it risks an indefinitely-open milestone. Keep it as a
    goal, but the *primary* delivered-outcome signal should be the catalog's adoption, with upstream
    acceptance as a bonus (the plan's self-serve framing is good but the metric table still leans on
    acceptance).
17. **Completeness score is unweighted.** A 90/100 entry could be missing `authentication` (the most
    important block) while full on `culture`. Weight authentication/identity/fusion as
    mandatory-for-shipping rather than letting them be traded against minor fields.
18. **No false-flag / liability framing for publishing a "misidentified" label.** Publicly asserting a
    line is misidentified is reputationally consequential; the plan should require the claim to be
    *quoted from ICLAC/Cellosaurus with version*, never an independent determination (see §5 Claude
    guardrails). Currently implied, not explicit in the metrics/DoD.

---

## 2. Competitive landscape (grounded)

**Cellosaurus (SIB) — the identity authority.**
Knowledge resource describing >100,000 cell lines; issues the `CVCL_`/RRID identifiers; aggregates
synonyms, donor info, STR profiles, and **machine-readable misidentification/"problematic" flags**;
CC BY 4.0; downloadable in multiple formats + API.
([Cellosaurus](https://www.cellosaurus.org/),
[PMC5945021](https://pmc.ncbi.nlm.nih.gov/articles/PMC5945021/),
[GitHub calipho-sib/cellosaurus](https://github.com/calipho-sib/cellosaurus)).
*Strengths:* the canonical anchor; broad; open; encodes ICLAC status. *Gaps:* breadth over depth —
no Ewing-specific curation, no linkage to *which open omics/dependency data exist*, no
functional-genomics view, no opinion on dependency evidence. **This is the project's anchor, not a
threat.**

**DepMap / CCLE (Broad).**
Largest compendium of genome-scale CRISPR (Chronos gene-effect), RNAi, expression, CN, mutations,
PRISM drug screens across cell lines; public releases CC BY 4.0 on Figshare.
([Registry of Open Data on AWS](https://registry.opendata.aws/depmap-omics-ccle/),
[DepMap 24Q4 Figshare](https://plus.figshare.com/articles/dataset/DepMap_24Q4_Public/27993248)).
*Strengths:* the dependency data itself; quarterly cadence; open. *Gaps:* pan-cancer, not
disease-curated; portal UX is per-gene/per-line, not "give me the trustworthy Ewing roster with
authentication + fusion + what data exists"; no misidentification curation; release-to-release model
churn is on the user to track.

**Cell Model Passports (Wellcome Sanger).**
Curated hub for >2,000 cancer cell/organoid models with clinical, genetic, functional datasets;
REST API; cross-refs Cellosaurus.
([Cell Model Passports](https://cellmodelpassports.sanger.ac.uk/),
[NAR D923](https://academic.oup.com/nar/article/47/D1/D923/5107576)).
*Strengths:* clean model metadata, organoids, programmatic access. *Gaps:* pan-cancer; **per-dataset
licensing is mixed** (not uniformly CC BY); not Ewing-curated; no consolidated misidentification
register view; UK/Sanger-screen-centric.

**ICLAC Register of Misidentified Cell Lines.**
Authoritative register of cross-contaminated/misidentified lines — **v14, 15 Feb 2026, 608 lines**
(560 no authentic stock; 48 authentic stock found). Distributed as a versioned **PDF**.
([ICLAC register](https://iclac.org/databases/cross-contaminations/), [ICLAC](https://iclac.org/)).
*Strengths:* the misidentification authority. *Gaps:* PDF, not an API; not per-disease; no
omics/identity linkage; updated ~annually.

**COSMIC Cell Lines Project (Sanger).**
~1,020 cell lines systematically characterized for somatic mutations/expression.
([About](https://cancer.sanger.ac.uk/cell_lines/about),
[license](https://cancer.sanger.ac.uk/cosmic/license?genome=37)).
*Strengths:* deep mutation curation. *Gaps:* **non-commercial / no-redistribution license** — usable
only as a link-only reference here; registration-gated.

**ATCC / DSMZ public catalogs.** Authoritative *sourcing/availability* + vendor STR for the lines
they distribute. *Strengths:* provenance of where to buy + authentication at source. *Gaps:*
copyrighted catalog prose; vendor-scoped (no DSMZ data for ATCC-only lines); no cross-resource
identity or dependency view.

**ExPASy.** The SIB portal that *hosts/links* Cellosaurus — a directory layer, not an independent
competitor ([sib.swiss](https://www.sib.swiss/)).

**Adjacent / partial overlaps:** Atlas of Genetics & Cytogenetics in Oncology (gene/fusion biology,
not a model catalog, [link](https://atlasgeneticsoncology.org/)); ECACC/Sigma culture collections
(availability); GDSC (drug sensitivity, terms-pending). **No existing resource is the
Ewing-specific, misidentification-aware, omics-availability-linked, fully-provenanced open catalog —
that whitespace is real.**

---

## 3. Gaps we can fill

1. **Disease-scoped trust layer.** None of the authorities curate *Ewing specifically*; researchers
   hand-assemble the roster today. A vetted ~20-30 line Ewing roster with authentication + fusion in
   one place is novel.
2. **"What open data exists for this model" index.** Cellosaurus has identity; DepMap has data;
   nobody publishes a per-Ewing-line **availability matrix** (expression/CN/open-mutations/fusions/
   CRISPR/drug) pinned to releases. High-value, currently a manual scavenger hunt.
3. **Reconciled misidentification view.** Joining ICLAC (PDF authority) + Cellosaurus (machine
   surface) + the *diagnosis-reclassification* axis (A673-type history) into one explicit,
   versioned, enum-typed status — no one publishes this reconciliation per disease.
4. **Fusion-type harmonization with provenance.** A consistent EWSR1 partner + exon-breakpoint
   representation across lines, each citing primary literature — replacing the current
   per-paper free text.
5. **Field-level provenance + license per assertion.** The authorities are CC BY but rarely expose
   field-level source+license in a redistributable join; that is exactly the catalog's deliverable.
6. **Release-drift surfacing.** A standing "this line was dropped/changed in DepMap 25Qx" signal that
   none of the upstreams hand the user.

---

## 4. Differentiators to win

- **Single strongest differentiator: a per-model, versioned "trust + data-availability passport" for
  Ewing — every assertion (identity, authentication-with-axis, fusion-with-breakpoint, what open data
  exists, what dependencies are reported) carried with field-level provenance, source version, and
  license — that no pan-cancer authority offers at disease depth.** It turns four portals + one PDF
  into one citable, machine-readable, openly-licensed object.
- **Misidentification-aware by construction**, with the ICLAC-vs-reclassification distinction the
  big resources blur.
- **Provenance and license as first-class data**, not prose — auditable, CI-gated, reproducible.
- **Upstream-correction loop**: errors found flow back to Cellosaurus/Sanger, compounding the good
  beyond the repo.
- **Honest scope**: descriptive aggregates with standing "research observation, not advice" caveats —
  trustworthy precisely because it refuses to over-claim.
- **Refresh discipline**: pinned releases + quarterly diff, so the catalog stays *current and
  auditable*, which the static PDF/portal status quo does not.

---

## 5. Claude API leverage

**Where Claude clearly helps (with human verification + provenance):**

1. **Name/synonym harmonization & candidate identity resolution.** Ewing lines carry many aliases
   (A-673/A673/A 673; TC-71/TC71; CHLA-9/10/25). Claude proposes synonym clusters and *candidate*
   DepMap↔CVCL↔SIDM mappings for the resolver, **flagging conflicts for human confirmation** — speeds
   the manual cross-walk without being the authority.
2. **Literature fact-extraction with citations.** Extract fusion partner + exon breakpoint, donor
   fields *as published*, culture conditions, and derivation history from open-access papers (PMC-OA),
   emitting `{value, quote, PMID, section}` so every assertion is traceable. Directly populates the
   field-level provenance cells.
3. **Discrepancy / drift detection.** Cross-compare a line's fusion/disease/donor across DepMap,
   Cell Model Passports, Cellosaurus and **surface disagreements** (e.g. differing breakpoint type,
   diagnosis-reclassification flags) as review items — never silently picking a winner.
4. **License-clause triage assistant.** Read a source's terms page and draft a `permitsRedistribution`
   rationale with the quoted clause + URL for the human license reviewer to ratify (NOT to decide).
5. **Release-diff summarization.** Summarize what changed for Ewing lines between DepMap releases to
   drive the quarterly refresh task.

**Where Claude must NOT decide (hard guardrails):**

- **Identity & misidentification determinations are authoritative-source-only.** Whether a line *is*
  misidentified/contaminated is read **verbatim from ICLAC + Cellosaurus with version**, never an
  independent LLM judgment. Claude may *retrieve and reconcile*; humans + authorities decide.
- **No fabricated or inferred provenance.** If a fact lacks a citable open source, the field stays
  empty — Claude must never synthesize a plausible PMID, breakpoint, or donor value. Every extracted
  value must round-trip to a real, fetchable source.
- **License/redistribution calls are human-ratified.** Claude drafts the rationale; the License+
  privacy reviewer makes the binding PASS/FLAG/EXCLUDE call.
- **No donor enrichment or re-identification**, no clinical/therapeutic interpretation of
  dependencies — schema- and review-forbidden regardless of model confidence.
- **Diagnosis-of-origin reclassification** (the A673 case) is asserted only from primary literature +
  Cellosaurus, never inferred from fusion presence alone.

---

## 6. Ten concrete optimizations

1. **Split the authentication field** into `misidentified` (ICLAC enum: `clean |
   misidentified_no_authentic | authentic_stock_exists | contaminated`) + `diagnosisReclassified`
   (disease-of-origin history). Prevents wrongly flagging A673.
2. **Read misidentification status from Cellosaurus as the machine surface, cite ICLAC vX as the
   authority**, and reconcile the two; don't hand-parse the PDF as the primary path.
3. **Promote STR from boolean to `strProfile{markers[], source, license}`**, redistributing the
   actual profile only from CC-BY sources (Cellosaurus), link-only for vendor STR.
4. **Normalize identity:** store primary `CVCL_` accession + `secondaryAccessions[]`, normalize the
   `RRID:` prefix, and mark each cross-ID as `direct` vs `inherited` to avoid false "independent
   agreement."
5. **Controlled fusion representation:** 5'/3' partner + exon pair (`EWSR1 ex7 :: FLI1 ex6`) as the
   canonical, with the legacy "Type N" label as an annotated secondary field, each separately cited.
6. **Omics-availability granularity:** per-datatype `{available, datasetId, releaseVersion, url,
   lastVerifiedRelease}` with explicit `absent` vs `unknown`; verify by **manifest-diffing** release
   model lists.
7. **Add `catalogVersion` (semver) + per-entry `revisionHistory`** so downstream reusers can diff and
   cite a stable snapshot.
8. **Weight the completeness score:** identity + authentication + fusion are *mandatory-to-ship*
   (cannot be traded against `culture`/minor fields) before the 90/100 gate applies.
9. **Pin exact source artifacts** (DepMap Figshare DOI per release; Cellosaurus release number;
   ICLAC vN PDF SHA-256 + Wayback) rather than named-but-unversioned sources.
10. **Ship a tiny stable JSON schema + JSON-LD/RO-Crate context and a 5-line query example** at M2 so
    adoption (a real outcome) is frictionless; bundle a CSV that carries per-field source+license via
    a row-level source manifest so attribution survives the flat export.

---

## 7. Parallel & perpendicular spin-offs

- **`ewing-pdx-model-index`** — same provenance/gate pattern applied to PDX/xenograft models (donor
  provenance matters more there; tighter privacy posture). Cross-links by RRID/PDX ID.
- **`ewing-expression-reanalysis`** — consumes this catalog's authenticated+available roster as its
  trusted input set (don't reanalyze a misidentified line); catalog supplies the "which lines, what
  data" map.
- **`ewing-open-data-catalog`** — superset dataset index (GEO/SRA/ArrayExpress studies) that this
  cell-line catalog links into per model.
- **`ewing-single-cell-atlas`** — cross-link single-cell availability per line/model.
- **Generalized rare-cancer model catalog** — the schema + gate + resolver are disease-agnostic;
  template it for other rare/pediatric cancers (rhabdomyosarcoma, osteosarcoma, DIPG). The Ewing
  build is the reference implementation.
- **MCP server over the published catalog** — a read-only Model Context Protocol server exposing
  `resolveIdentity`, `getAuthentication`, `getOmicsAvailability`, `getDependencies` so any agent can
  query the trust layer (Claude API leverage + adoption channel in one).
- **Misidentification-alert feed** — a versioned diff/RSS that fires when a catalogued Ewing line
  newly appears on ICLAC/Cellosaurus problematic lists, or is dropped/changed in a DepMap release.
  Genuinely useful to labs and a recurring "outcome" generator.
- **`ewing-fusion-detection-benchmark`** — the harmonized fusion truth set here can seed a benchmark
  of fusion callers (catalog supplies ground-truth breakpoints with provenance).

---

## 8. Open questions for the maintainer

1. **ICLAC ingestion:** accept Cellosaurus as the machine surface over ICLAC (cite ICLAC vX as
   authority), or commit to parsing the ICLAC PDF directly? (Affects refresh cost.)
2. **Diagnosis-reclassification vs misidentification:** confirm these become *two* schema axes so
   A673 (authentic, reclassified) isn't flagged as misidentified.
3. **Non-canonical "Ewing-like" entities** (CIC-rearranged, BCOR): in scope as *flagged-distinct*, or
   excluded at the roster gate?
4. **STR redistribution:** publish actual STR marker tables where Cellosaurus permits, or stay
   availability-flag only to avoid any vendor-STR rights question?
5. **Cell Model Passports licensing:** does it actually clear CC-BY redistribution for the specific
   model-metadata fields used, or should it be downgraded to "VERIFY-high-risk / possibly link-only"
   like COSMIC?
6. **Cross-ID independence:** how to record/penalize `inherited` vs `direct` cross-references so the
   "resolved across three systems" metric isn't inflated by shared upstream assertions?
7. **Catalog versioning & DOI:** mint a Zenodo DOI per catalog release for citeable adoption, or
   rely on git tags + `catalogVersion`?
8. **Outcome metric balance:** is gating M3 on ≥2 *accepted* upstream corrections (exogenous,
   slow) wise, or should adoption-by-a-consumer be the primary delivered signal with upstream
   acceptance as bonus?
9. **Variant/derivative lines** (e.g. A673 sublines, luciferase/Cas9 derivatives): one entry with
   derivatives, or separate entries? Affects identity resolution and donor inheritance.
