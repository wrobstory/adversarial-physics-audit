# Audit report template

Use this structure without inventing evidence. Omit empty optional
sections, but keep the claim and mutation ledgers complete for the
declared scope.

## Scope and environment

- Target commit, branch, initial dirty state
- Audit mode: execution, static, or mixed
- Included claims and configurations
- Explicit exclusions and sampling rationale
- Toolchain, precision, hardware backend, and relevant runtime settings

## Findings

Order adverse findings by impact. For each finding record:

- Verdict
- Finding type
- Impact
- Confidence
- Evidence class: reproduced, independently derived, primary-sourced,
  inferred, or unverified
- Exact artifact location, command and result, or shown derivation
- Affected claim IDs
- Disconfirming evidence considered

## Claim ledger

| ID | Claim and source | Quantity/metric/domain | Claimed tolerance | Evidence | Verdict | Notes |
|---|---|---|---:|---|---|---|

## Mutation ledger

| ID | Defect | Targeted claim | Reachability evidence | Outcome | Catching assertion / gap | Independence | Cleanup verified |
|---|---|---|---|---|---|---|---|

Use `caught`, `survived`, `not exercised`, `invalid`, `equivalent`, or
`indeterminate/flaky` in execution mode. Use `predicted-caught`,
`predicted-uncaught`, or `indeterminate` in static mode.

## Oracle-independence ledger

| Oracle | Shared lineage | Independent lineage | Grade | Protected physics | Common-mode exposure |
|---|---|---|---|---|---|

Grade each oracle `fully independent`, `partially independent`, or
`common-mode vulnerable`.

## Provenance ledger

| Dataset | Primary source location | Transcription | Transformations | Applicability | Verdict |
|---|---|---|---|---|---|

Include domain-construction artifacts (meshes, clipped/scaled/digitized
representations) as rows; record closure and dimension cross-checks
under Transformations. Any gate failure attributed to source data must
cite the differential-diagnosis evidence supporting that attribution.

## Execution boundary

- Exact commands and results
- Unavailable or skipped checks and why
- Unverified claims
- Final commit, branch, and dirty state
