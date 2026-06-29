# TASKS — ewing-cell-line-catalog

> Status: Draft · Version: 0.2.0 · Last updated: 2026-06-29 · Owner: TBD (maintainer) · Lane: donated

## How these tasks map to Elyos

Each task below becomes an Elyos **Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- `id` — stable slug from the tables (e.g. `ewing-cell-line-catalog-schema-001`).
- `title` — the table's Title.
- `project` — `ewing-cell-line-catalog`.
- `type` — `code | research | writing | data | design-spec | maintenance` (per table).
- `lane` — `donated` for all tasks (no funded escrow). A funded task would add `fundedBudgetUsd`.
- `priority` — `high | medium | low`.
- `domain` — array, e.g. `["cancer-research","ewing-sarcoma","bioinformatics","open-data","cell-lines"]`.
- `riskTier` — `low | medium | high`. **No `high` tasks** (nothing here is patient-facing). License/
  privacy, donor-provenance, misidentification, fusion, and dependency-framing tasks are `medium`.
- `urgent` — boolean; `false` for all current tasks.
- `deliverable` — `pr | dataset | document | translation`. Tooling → `pr`; catalog metadata records
  / derived aggregates → `dataset` (CC-BY, redistributable); specs/policies/reports → `document`.
- `tokenEstimate` — `small | medium | large` (Size column).
- `status` — `open | in-progress | review | delivered | done`; all start `open`.
- `context`, `objective`, `acceptanceCriteria[]`, `resources[]`, `output` — per task.
- `requestor` — **TO BE SECURED** until a beneficiary/partner is confirmed.
- `verifiedNeed` — **`false`** until a named research consumer/upstream maintainer confirms use/
  acceptance (general need is real; per-task delivery need is unproven).
- `outputLicense` — `CC-BY-4.0` for catalog metadata/aggregates and documents; `MIT` for code.

**Standing constraints on every data task:** open-access sources only (no dbGaP/EGA/biobank/
controlled-access); COSMIC/OncoKB link-only; donor fields published-only with no re-identification;
field-level provenance on every assertion; every dependency assertion carries the standing
"research observation; not medical advice" caveat.

---

## Milestone M0 — Foundation & cold-start

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ewing-cell-line-catalog-schema-001 | Canonical cell-line record schema + field-level provenance model | design-spec | small | low | document | — | Technical, Domain |
| ewing-cell-line-catalog-gate-002 | Open-data / license / donor-privacy triage gate (per source-block) | design-spec | small | medium | document | — | License+privacy |
| ewing-cell-line-catalog-sources-003 | Source register with verified per-source licenses (DepMap pinned by Figshare DOI; Cell Model Passports flagged VERIFY-high-risk; Cellosaurus, ICLAC, COSMIC/OncoKB/GDSC dispositions) | research | small | medium | document | gate-002 | License+privacy |
| ewing-cell-line-catalog-reviewers-004 | Name/secure License+privacy reviewer + Domain reviewer; open beneficiary outreach | research | small | low | document | — | Maintainer |
| ewing-cell-line-catalog-resolver-005 | Identity resolver (DepMap ID ↔ RRID ↔ Cell Model Passports) + golden fixtures | code | medium | low | pr | schema-001 | Technical |
| ewing-cell-line-catalog-validator-006 | Schema validator + cross-source consistency checks + golden fixtures | code | small | low | pr | schema-001 | Technical |
| ewing-cell-line-catalog-pilot-007 | End-to-end catalog entry for pilot model A673 | data | medium | medium | dataset | schema-001, gate-002, sources-003, resolver-005, validator-006, reviewers-004 | License+privacy, Domain, Technical |

**Acceptance criteria — key tasks**

- **schema-001 (record schema + provenance model)**
  - [ ] Canonical record documents all fields: `identity` (depmapId, **primary `rrid`/CVCL +
        `secondaryAccessions[]`, each cross-ID tagged `direct`/`inherited`**, cellModelPassportId,
        `ccleName` recorded but flagged **never-an-anchor**, synonyms), `authentication`, `disease`
        (Oncotree), `fusion`, `donor`, `culture`, `availability[]`, `omicsAvailability`,
        `dependencies`, `license`, `completenessScore`, `revisionHistory[]`, `provenance`; the catalog
        carries a semantic `catalogVersion`.
  - [ ] Every value-bearing field is a `{value, source, sourceVersion, retrievedAt, license}`
        provenance cell (or array); an uncited assertion is defined as a review failure.
  - [ ] `authentication` is the first-surfaced block and splits two axes: `strProfile {markers[],
        source, license}` (marker table redistributed only from CC-BY sources, else link-only),
        `misidentified` (enum `clean | misidentified_no_authentic | authentic_stock_exists |
        contaminated`, read from Cellosaurus, citing ICLAC vN), and a SEPARATE `diagnosisReclassified`
        axis (so A673's RMS→Ewing reclassification is not flagged as misidentification); `dependencies`
        includes a mandatory standing caveat field ("research observation; not medical advice").
  - [ ] `fusion` uses the controlled `EWSR1 ex_ :: partner ex_` form with a separately-cited
        `legacyType` ("Type N"); `omicsAvailability` is per-datatype `{available (true|false|unknown),
        datasetId, releaseVersion, url, lastVerifiedRelease}`; `completenessScore` is **weighted**
        (identity + authentication + fusion mandatory-to-ship).
  - [ ] States output license: metadata/aggregates CC-BY-4.0, code MIT; deliverable is metadata, not
        patient data; controlled-access data is schema-forbidden.
  - [ ] `pnpm build && pnpm test && pnpm lint` pass for any committed schema tooling; DCO signed-off.

- **gate-002 (open-data / license / donor-privacy gate)**
  - [ ] Opens with the binding cancer guardrails: open-access-only; dbGaP/EGA/biobank/controlled-
        access categorically EXCLUDED; COSMIC/OncoKB link-only; no medical advice; provenance required.
  - [ ] Objective license criterion: a source-block PASSes only if `permitsRedistribution: true` is
        set from a cited clause/URL under the output license; missing/unparseable/NC/controlled =
        FLAG (link-only) or EXCLUDE — never default-allow.
  - [ ] Donor-privacy rule encoded: donor fields published-only; no enrichment, no external linkage,
        no re-identification; escalate unusually identifying records; reduce/omit on doubt.
  - [ ] Requires recording license id + URL + snapshot (committed copy + SHA-256 + Wayback URL) per
        source-block.
  - [ ] Produces a committed PASS/FLAG/EXCLUDE artifact per source-block recording which checks ran.

- **resolver-005 (identity resolver)**
  - [ ] Resolves and cross-checks DepMap ID ↔ **primary CVCL accession** (Cellosaurus) ↔ Cell Model
        Passports ID using the primary CVCL accession as the anchor (normalizing the `RRID:` prefix
        and capturing `secondaryAccessions[]`); tags each cross-ID `direct` vs `inherited`; **never
        uses `ccleName` to resolve**; reports unresolved/conflicting mappings rather than guessing.
  - [ ] Ships golden fixtures incl. a deliberate cross-resource conflict that must be flagged, an
        `inherited`-vs-`direct` case, and a merged/secondary-accession case; valid triples resolve,
        malformed accessions fail; synthetic/public fixtures only.
  - [ ] Code MIT-licensed; `pnpm build && pnpm test && pnpm lint` green; no credentials embedded.

- **pilot-007 (A673 end-to-end)**
  - [ ] A673 chosen because all core sources cover it; every contributing source-block passed
        gate-002 (artifact committed); identity resolved across all three ID systems.
  - [ ] Authentication recorded on both axes: `misidentified` enum (read from Cellosaurus, citing
        ICLAC vN) **and** `diagnosisReclassified` — A673 is `clean`/authentic with a documented
        RMS→Ewing reclassification, and must NOT be flagged misidentified (validates the two-axis
        design); disease/Oncotree, fusion (controlled `EWSR1 ex7 :: FLI1 ex6` + legacy type +
        evidence), donor (published-only), availability, omics-availability populated; all
        field-level provenanced.
  - [ ] Completeness score recorded (before/after; after ≥ 90/100); entry validates in CI.
  - [ ] License+privacy and Domain reviewers signed off; no NC/controlled data present; no
        therapeutic/clinical claim.
  - [ ] Real outcome reachable via self-serve path: any error found in A673's upstream records is
        prepared as an upstream correction (submission counts toward M0; acceptance toward M3).

**M0 Definition of Done:** schema + field-level provenance model published; open-data/license/donor
gate published and applied to the pilot; source register with verified licenses published;
License+privacy reviewer and Domain reviewer **named** before pilot review; identity resolver +
validator green in CI with golden fixtures; A673 catalogued end-to-end at ≥ 90/100 with gate
artifact; ≥ 1 beneficiary-outreach thread opened.

---

## Milestone M1 — Authenticated core catalog

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ewing-cell-line-catalog-roster-008 | Compile + gate-triage authoritative Ewing model roster (MODEL-ROSTER.md) | research | medium | medium | document | gate-002, sources-003 | License+privacy, Domain |
| ewing-cell-line-catalog-entries-009 | Full catalog entries for top ~10 authenticated Ewing lines | data | large | medium | dataset | pilot-007, roster-008 | License+privacy, Domain, Technical |
| ewing-cell-line-catalog-misid-010 | Misidentification + contamination audit vs. ICLAC + Cellosaurus | research | small | medium | document | roster-008 | Domain |
| ewing-cell-line-catalog-fusion-011 | Fusion-status annotation in controlled `EWSR1 ex_ :: partner ex_` form (+ legacy "Type N") with provenance | data | medium | medium | dataset | entries-009 | Domain |
| ewing-cell-line-catalog-upstream-012 | Submit first upstream correction(s) (Cellosaurus / Cell Model Passports) | research | small | low | document | misid-010, entries-009 | Domain, Steward |

**Acceptance criteria — key tasks**

- **roster-008 (model roster + triage)**
  - [ ] `MODEL-ROSTER.md` lists candidate Ewing lines keyed by **primary CVCL accession** with
        DepMap/Cell Model Passports cross-IDs and each source's license disposition.
  - [ ] Every candidate assigned a gate disposition (PASS / FLAG / EXCLUDE) with rationale; no model
        promoted to an entry task without a committed gate artifact.
  - [ ] **Explicit in/out decision recorded for non-canonical "Ewing-like" entities (CIC-rearranged,
        BCOR)** — included as flagged-distinct or excluded at the gate, not left ambiguous.
  - [ ] Known-misidentified lines are retained on the roster *with their warning* (not silently
        dropped) so they can be catalogued with the flag.
  - [ ] Controlled-access-only models (no open data) are excluded; COSMIC/OncoKB-only data marked
        link-only.

- **entries-009 (top ~10 authenticated lines)**
  - [ ] ≥ 10 models fully catalogued at ≥ 90/100, each with committed gate artifact and full
        field-level provenance.
  - [ ] Identity resolved across all three ID systems for each; authentication status present;
        cross-source disagreements shown with provenance (not silently resolved).
  - [ ] No NC/controlled data redistributed; deliverable `dataset` licensed CC-BY-4.0; domain +
        license reviewers signed off.

- **misid-010 (misidentification audit)**
  - [ ] Status read from **Cellosaurus's machine-readable "problematic" flag as the machine surface**,
        reconciled against and **citing the ICLAC register vN as the underlying authority** (version
        recorded); recorded as the `misidentified` enum, a provenanced field.
  - [ ] The separate `diagnosisReclassified` axis recorded where applicable (e.g. A673) — NOT conflated
        with misidentification.
  - [ ] Any `misidentified` label is **quoted verbatim from ICLAC/Cellosaurus with version, never an
        independent determination**; flagged models carry a prominent, sourced warning.
  - [ ] Findings that contradict an upstream record are routed to upstream-012.

**M1 Definition of Done:** roster compiled + gate-triaged; ≥ 10 models fully catalogued at
≥ 90/100; 100% of catalogued models carry an authentication status cross-checked vs. ICLAC +
Cellosaurus; fusion status annotated with provenance for every catalogued model; ≥ 1 upstream
correction submitted.

---

## Milestone M2 — Dependencies & omics availability

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ewing-cell-line-catalog-depmap-013 | DepMap dependency aggregator (CRISPR Chronos) + golden fixtures; pin release | code | medium | medium | pr | schema-001, validator-006 | Technical, Domain |
| ewing-cell-line-catalog-omics-014 | Omics-availability map per model — per-datatype `{available(true/false/unknown), datasetId, releaseVersion, lastVerifiedRelease}` verified by manifest-diffing | data | medium | low | dataset | entries-009 | Domain, Technical |
| ewing-cell-line-catalog-deps-015 | Per-model dependency summaries for the core roster (with caveat) | data | large | medium | dataset | depmap-013, entries-009, misid-010 | Domain, License+privacy |
| ewing-cell-line-catalog-publish-016 | Publish catalog: machine-readable (JSON+CSV) + human-readable site | code | medium | low | pr | entries-009, omics-014, deps-015 | Technical, Maintainer |

**Acceptance criteria — key tasks**

- **depmap-013 (dependency aggregator)**
  - [ ] Pins a specific DepMap public release **by exact Figshare DOI** (recorded in
        `compiledFromReleases`); aggregates CRISPR gene-effect (Chronos) into per-model
        selective-dependency summaries.
  - [ ] DepMap release license verified to permit redistributable aggregates (gate-002 artifact);
        only CC-BY-permitted aggregates emitted; bulk source matrices referenced, not re-hosted.
  - [ ] Ships golden fixtures: synthetic gene-effect matrix → expected summary; methodology + caveat
        attached to output. Code MIT; CI green; no credentials embedded.

- **deps-015 (per-model dependency summaries)**
  - [ ] Dependency summaries produced only for models whose authentication status is known
        (depends on misid-010) — never profile a line before its flag.
  - [ ] Each dependency assertion carries source + release version + methodology + the standing
        "research observation; not medical advice; not a treatment recommendation" caveat.
  - [ ] No therapeutic/prognostic/"druggable in patients" claim; domain reviewer confirms framing.

- **publish-016 (publish catalog)**
  - [ ] Catalog emitted as canonical JSON + CSV projection + human-readable site, all assertions
        field-level provenanced; attribution to DepMap/Cell Model Passports/Cellosaurus/ICLAC present;
        stamped with a semantic `catalogVersion`.
  - [ ] Ships a **small stable JSON schema + a JSON-LD/RO-Crate context + a ~5-line query example** to
        make adoption frictionless; the **CSV carries per-field source+license via a row-level source
        manifest** so attribution survives the flat export (the known failure point).
  - [ ] Catalog metadata/aggregates CC-BY-4.0; site code MIT; CI green; no controlled/NC data present.

**M2 Definition of Done:** DepMap release pinned and dependency aggregator green in CI with golden
fixtures; omics-availability map populated for the core roster; dependency summaries (with version +
methodology + caveat) for the core roster; catalog published machine-readable + human-readable;
≥ 20 models fully catalogued.

---

## Milestone M3 — Upstream corrections, reuse & sustainability

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ewing-cell-line-catalog-corrections-017 | Land ≥ 2 accepted upstream corrections (evidence artifacts) | research | small | low | document | upstream-012 | Steward, Domain |
| ewing-cell-line-catalog-reuse-018 | Track + verify external reuse/adoption events | research | small | low | document | publish-016 | Steward |
| ewing-cell-line-catalog-refresh-019 | Quarterly DepMap refresh + re-authentication process | maintenance | small | low | document | depmap-013, misid-010 | Maintainer |

**Acceptance criteria — key tasks**

- **corrections-017 (accepted upstream corrections)**
  - [ ] ≥ 2 corrections accepted by an upstream resource (Cellosaurus / Cell Model Passports /
        DepMap), each with a recorded `outcomes/<event-id>.json` (channel, permalink, timestamp,
        before/after).
  - [ ] Each accepted correction links to externally verifiable evidence; self-reported does not count.

- **reuse-018 (external reuse tracking)**
  - [ ] ≥ 3 verifiable external reuse events recorded (citation, fork building on the catalog, or
        issue/PR referencing it); each with verifiable evidence.
  - [ ] On the first confirmed research consumer, relevant tasks flip `verifiedNeed: true` with
        `requestor` set.

**M3 Definition of Done:** ≥ 2 upstream corrections accepted; ≥ 3 verifiable external reuse events;
≥ 1 confirmed research consumer/partner; quarterly DepMap refresh + re-authentication process
documented with a named steward.

---

## Backlog / future

| ID | Title | Type | Size | Risk | Deliverable | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| ewing-cell-line-catalog-drugsens-020 | Drug-sensitivity availability (PRISM/GDSC) annotation | data | medium | medium | dataset | Only if GDSC redistribution terms verified; else availability-flag only |
| ewing-cell-line-catalog-scrna-021 | Cross-link to single-cell availability (`ewing-single-cell-atlas`) | data | small | low | dataset | Cross-link, not recompute |
| ewing-cell-line-catalog-pdx-022 | Cross-link to PDX/model index (`ewing-pdx-model-index`) | data | small | low | dataset | Cross-link, not recompute |
| ewing-cell-line-catalog-client-023 | Small R/Python client to query the published catalog | code | medium | low | pr | Lowers reuse friction; reads CC-BY catalog |
| ewing-cell-line-catalog-i18n-024 | Translate human-readable catalog intro/caveats | translation | small | medium | translation | Needs language reviewer; widens reuse |
| ewing-cell-line-catalog-expand-025 | Catalog remaining authenticated/variant Ewing lines | data | large | medium | dataset | Throughput from MODEL-ROSTER.md via the gate |
| ewing-cell-line-catalog-mcp-026 | Read-only MCP server over the published catalog (resolveIdentity / getAuthentication / getOmicsAvailability / getDependencies) | code | medium | low | pr | Adoption channel + agent-query surface; reads CC-BY catalog only |
| ewing-cell-line-catalog-alertfeed-027 | Misidentification-alert feed (versioned diff/RSS) when a catalogued line newly appears on ICLAC/Cellosaurus or is dropped/changed in DepMap | code | medium | medium | pr | Recurring outcome generator; wired to refresh; never an independent misid call |
| ewing-cell-line-catalog-registry-028 | Generalize schema+gate+resolver into a disease-agnostic reagent-authenticity registry template (RMS/osteosarcoma/DIPG) | design-spec | medium | low | document | Ewing build is the reference implementation |

---

## Example task JSON

```json
{
  "id": "ewing-cell-line-catalog-schema-001",
  "title": "Canonical cell-line record schema + field-level provenance model",
  "project": "ewing-cell-line-catalog",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer-research", "ewing-sarcoma", "bioinformatics", "open-data", "cell-lines"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "small",
  "status": "open",
  "context": "Ewing sarcoma research relies on a small set of cell-line models whose metadata and open data are fragmented across DepMap, Cell Model Passports, Cellosaurus, and the ICLAC misidentified-cell-lines register. Before cataloguing any model, the project needs one canonical record schema and a field-level provenance model that all outputs (JSON, CSV, human-readable site) are projections of. The deliverable is open-access metadata and derived aggregates only — never patient-level or controlled-access data, and never a clinical claim.",
  "objective": "Define the reusable canonical cell-line record schema and the field-level provenance model that every per-model catalog task will use.",
  "acceptanceCriteria": [
    "Schema documents all fields: identity (depmapId, primary rrid/CVCL + secondaryAccessions[] with each cross-ID tagged direct/inherited, cellModelPassportId, ccleName flagged never-an-anchor, synonyms), authentication, disease (oncotreeCode, lineage, subtype), fusion (partner5p, partner3p, breakpointExons, legacyType, evidence[]), donor (sexAtSampling, ageAtSampling, siteOfSampling, sampleType), culture, availability[], omicsAvailability (per-datatype available true/false/unknown + datasetId + releaseVersion + lastVerifiedRelease), dependencies, license, completenessScore (weighted), revisionHistory[], provenance; catalog-level catalogVersion.",
    "Every value-bearing field is a {value, source, sourceVersion, retrievedAt, license} provenance cell (or array); an uncited assertion is defined as a review failure.",
    "Authentication is the first-surfaced block and splits two axes: strProfile{markers[],source,license} (redistributed only from CC-BY sources), misidentified (enum: clean | misidentified_no_authentic | authentic_stock_exists | contaminated, read from Cellosaurus citing ICLAC vN, never an independent determination), and a separate diagnosisReclassified axis; the dependencies field includes a mandatory standing caveat ('research observation; not medical advice; not a treatment recommendation').",
    "Schema forbids controlled-access provenance (no dbGaP/EGA/biobank source values) and marks COSMIC/OncoKB source-blocks as link-only (not redistributed).",
    "Donor fields are constrained to published-only values with no enrichment/linkage, per the donor-privacy rule.",
    "Schema states output license: metadata/aggregates CC-BY-4.0, code MIT; deliverable is metadata, never patient data.",
    "pnpm build && pnpm test && pnpm lint pass for any committed schema tooling; commit is DCO signed-off."
  ],
  "resources": [
    "C:\\code\\elyos\\planning\\projects\\ewing-cell-line-catalog\\PLAN.md",
    "C:\\code\\elyos\\packages\\schema\\src\\schemas.ts",
    "DepMap / Cancer Dependency Map (Broad Institute)",
    "Cell Model Passports (Wellcome Sanger Institute)",
    "Cellosaurus (SIB) — RRID/CVCL identifiers",
    "ICLAC Register of Misidentified Cell Lines"
  ],
  "output": "A canonical cell-line record schema plus a field-level provenance model definition, committed to the project repo and ready for reuse by per-model catalog tasks.",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "CC-BY-4.0"
}
```
