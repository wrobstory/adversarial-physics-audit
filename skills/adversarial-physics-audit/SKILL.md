---
name: adversarial-physics-audit
description: Run an adversarial, evidence-disciplined correctness audit of a physics, simulation, scientific-computing, or numerical library — hunting invented or mistranscribed validation data, circular oracles, hardcoded expectations, tolerance gaming, and claims not backed by code. Use whenever the user questions whether numerical or scientific code is actually correct ("how do we KNOW this is right", "could this be hallucinated", "is the test data hardcoded", "adversarial review", "hostile review", "audit the validation"), asks to stress-test a solver's test suite, or wants a red-team review of simulation results. Supports execution, static, and mixed-evidence audits.
---

# Adversarial Physics Audit

Act as a hostile reviewer. Hold this working hypothesis until evidence
forces you off it: the physics may be hallucinated, validation data
invented or mistranscribed, oracles circular, expectations copied from
the code's own output, or tolerances widened until green. Treat docs
and READMEs as claims under indictment, not evidence. Do not summarize
the test suite; try to break it.

Hostility is symmetric. An adverse self-report ("our fixture fails
fidelity", "the source data is deficient", "diagnostic only") is not
evidence of honesty — it contains a causal attribution, and that
attribution is a claim under the same indictment as any "it works".
Honest labeling of a failure does not verify the *explanation* of the
failure; a pipeline defect misdiagnosed as a data deficiency survives
every consistency check precisely because nothing was over-claimed.
Audit the attribution: reproduce the failure, then test whether the
defect lives in the pipeline that computed it before accepting that it
lives in the data.

Use hostility to generate experiments, not accusations. Missing
provenance, suspicious patterns, and surviving mutations are evidence
gaps, not proof of fabrication. Reserve **fabrication** for affirmative
evidence of invention or knowing misrepresentation. Seek and record
disconfirming evidence as aggressively as incriminating evidence.

Three standing rules about your own limits:

1. **Never trust your own memory of physical constants, published
   coefficients, or paper contents** — you are the same kind of system
   that allegedly hallucinated the code. Every reference number must be
   independently derived (closed-form, dimensional analysis, limiting
   case, hand arithmetic shown step by step) or fetched from a primary
   source during the audit. Otherwise report "unverifiable here",
   never "looks right".
2. **A doc citing a test is a pointer, not a fact.** Follow every
   pointer to the file and line and quote what is actually there.
3. **Separate observation from inference.** Label evidence as
   reproduced, independently derived, primary-sourced, inferred, or
   unverified. Never promote an inference because it fits the hostile
   hypothesis.

Unless the user separately authorizes remediation, do not fix findings.
Make mutations only in a disposable worktree or copy; never overwrite
user changes. Keep scratch files uncommitted and report their location,
or use a reported scratch commit when explicitly authorized.

## Mode selection

If you can execute, use execution mode: build, test, and run scripts
yourself; capture exact commands, environment, and output. Report a
claim you did not reproduce as unverified.

If you cannot execute (read-only access, no shell): use static mode.
The words "passes" and "works" are then banned from your report;
classify claim evidence as **artifact-traceable** (an artifact asserts
exactly what the claim says), **untraceable**, or **contradicted**.
Artifact-traceable is not scientific verification. Classify static
mutation results only as **predicted-caught**, **predicted-uncaught**,
or **indeterminate**. Pretending to verify execution is the exact
failure mode being audited.

If only some commands or configurations can run, use mixed mode. Apply
execution verdicts only to reproduced evidence and static labels to
everything else.

## Phase 0 — Map the target

Before attacking, define the audit scope and inventory docs, manifests,
test directories, build scripts, and CI configuration:

- **Claims**: assign an ID to every in-scope quantitative or correctness
  claim and record quantity, metric, units, domain, configuration,
  tolerance, and source. Give every in-scope claim a verdict. For large
  repositories, state exclusions and justify any risk-based sampling.
- **Tests**: all test targets, including ignored/feature-gated/
  uncompiled ones; their discovery commands and configuration matrix;
  which physics each actually asserts.
- **Reference data**: fixtures, golden files, digitized experimental
  data, checksum manifests — and each file's claimed provenance.
- **Oracles**: any "independent" comparison implementation, and the
  data-flow between it and the code under test.
- **Governing equations**: what physics the library claims to
  implement, including boundary conditions, forcing, conventions, and
  numerical method — and the cited source for each equation, for
  verification in section 3.
- **Environment**: commit, branch, dirty state, dependency lockfiles,
  compiler/interpreter, precision mode, hardware backend, and relevant
  runtime settings.

## 1. Mutation analysis — the decisive experiment

A test suite measures correctness only if it fails when the physics is
wrong. First establish a stable baseline: run the canonical build and
test commands, enumerate selected configurations and skipped tests,
characterize flakiness, and confirm the harness discovers failures and
propagates a nonzero exit status. Do not score mutants against a broken
or irreproducible baseline.

Build a risk-ranked mutation list from the target's own physics. Cover
governing terms and conventions with applicable perturbations such as:

- Sign flip on a dominant force/restoring/source term.
- A physical constant off by 1–2% (density, gravity, diffusivity…).
- One coupling matrix transposed; one off-diagonal sign flipped.
- A phase/sign/frame convention applied once too often.
- A sub-state frozen across integrator stages instead of advanced.
- One semantic reference-data value corrupted by about 1% in a
  disposable copy. Update its checksum only when testing a claimed
  integrity or independent-validation mechanism.
- A factor-of-2 in one empirical/semi-empirical component.
- One element dropped from a domain-construction selection (a geometry
  patch, surface tag, data column, crop range) — tests whether the
  pipeline has integrity gates or silently computes on incomplete
  domains.
- Correlated instead of independent random draws where independence is
  a stated model assumption.

In execution mode, for each mutant:

1. Apply one semantic change in the disposable tree and record the diff.
2. Confirm the mutated code or data was built and exercised, using
   coverage or a controlled observable where practical.
3. Clean or rebuild enough to exclude stale artifacts, then run the
   targeted and relevant full suites under the baseline environment.
4. Classify the result as **caught**, **survived**, **not exercised**,
   **invalid**, **equivalent**, or **indeterminate/flaky**. For caught
   mutants, identify whether the catching evidence is independent or
   shares the defective path.
5. Revert the mutant, verify the expected tree hash/dirty state, and
   rerun the baseline check.

Only a reached, behavior-changing, valid mutant that survives stable
tests establishes a validation gap. In static mode, name the exact
assertion predicted to catch each mutant and show why its effect should
exceed the asserted tolerance; otherwise mark it indeterminate. Never
assume a differential test catches a defect when both sides share the
defective path.

## 2. Hardcoded expectations and tolerance archaeology

- For every numeric literal used as an expected value: where did it
  come from? Acceptable origins are a shown derivation, a committed
  independent source with provenance, or hand arithmetic reproduced in
  the report. "From a previous run of this code" is a regression pin —
  reclassify each in-scope claim leaning on one.
- Version-control history (execution mode): hunt commits where
  expectations changed to match new output, assertions were deleted or
  ignored, or tolerances widened with vague messages. Reconstruct the
  tolerance history of every headline number.
- Vacuous tests: values compared to re-computations of the same
  expression, self-normalized error metrics, `is_finite` posing as
  physics, possibly-empty iteration, tolerances loose enough to admit
  a zeroed output (estimate response magnitude to show it).
- Compare tolerance semantics, not digits alone: quantity, metric,
  absolute/relative form, norm, units, domain, percentile, dataset, and
  configuration must align. Call a claim contradicted only when the
  supporting assertion addresses the same quantity and admits weaker
  error than the claim permits.

## 3. Governing-equation verification

Consistency with the claimed model is not evidence the claimed model
is the true model: a faithfully implemented wrong equation satisfies
its own invariants beautifully. The equations themselves are claims;
audit them against primary sources fetched during the audit.

- For each governing equation, closure, and empirical correlation,
  obtain the authoritative form from a high-quality primary source —
  the cited paper or a standard reference text — during the audit,
  never from memory. Record source, edition or version, equation
  number, and page.
- The citation is itself a claim. Confirm the source exists, actually
  contains the equation attributed to it, and is being applied inside
  its stated validity regime and assumptions (flow regime, linearity,
  frames, boundary conditions, parameter ranges).
- Write out the sourced equation and check the implementation against
  it term by term: every term present, no undisclosed extra or dropped
  terms, each sign, prefactor, and nondimensionalization correct, and
  conventions consistent with the code's declared ones. Cite code
  file:line per term.
- Where no primary source can be fetched, derive the equation from
  first principles when feasible (show the derivation); otherwise mark
  every dependent claim "equation unverified" — do not soften it.
- Classify each discrepancy precisely: transcription error,
  undisclosed simplification, regime violation, or unverifiable
  citation.

## 4. Oracle and differential-test circularity

- Map shared equations, constants/data, conventions, conditioning,
  intermediate files, dependencies, calibration data, authorship, and
  execution environment between the code and each oracle. Grade the
  comparison as fully independent, partially independent, or
  common-mode vulnerable, and enumerate exactly what it protects.
- Derive applicable analytic limits, asymptotics, conservation laws,
  sign/positivity constraints, symmetries, and dimensional relations
  from the claimed model. State the boundary conditions, forcing,
  discretization, and conventions that make each property valid. Do
  not assume zero response, conservation, positivity, or symmetry when
  the model does not require it.

## 5. Numerical and statistical integrity

- Build an error budget separating discretization, iteration,
  roundoff, model, measurement, and reference-solution error where
  applicable. Check that the claimed tolerance is plausible within it.
- Inspect absolute and relative error near zero; norms, units,
  normalization, aggregation, grid/sample selection, conditioning, and
  expected roundoff amplification. Test whether zeroed or rescaled
  outputs can pass.
- For convergence claims, refine one error source at a time, establish
  an asymptotic regime, and verify observed order rather than comparing
  only two resolutions.
- Record precision, compiler flags, CPU/GPU backend, math libraries,
  fused operations, parallelism, and deterministic settings when they
  can affect results. Reproduce on another relevant configuration when
  the claim is portable and resources permit.
- For stochastic claims, record seed policy, repetitions, confidence
  intervals, effect size, test power, and multiple-comparison handling.
  Distinguish stochastic flakiness from a stable scientific failure.

## 6. Reference-data provenance and domain construction

- Checksums prove self-consistency only. Separately assess source
  authority, transcription integrity, transformation reproducibility,
  and applicability to the claimed case. Record source version or DOI,
  table/equation/page, units, and transformation chain. Fetch and
  spot-check primary sources where possible; otherwise report the gap.
- Use overly round or smooth values, repeating digit patterns,
  identical noise, or impossible precision only as screening signals.
  Do not report fabrication without affirmative evidence.
- Execution mode: regenerate generated fixtures with the documented
  pipeline and diff against committed ones; if the pipeline can't run
  here, mark every fixture-derived claim "provenance unverified" — do
  not soften it.
- Read generation/export scripts for undisclosed synthesis,
  extrapolation, smoothing, or "repair" of data before writing.
- **Domain construction.** Every pipeline that turns a source artifact
  into a computational representation — meshing, clipping, scaling,
  unit conversion, datum mapping, resampling, digitization — is itself
  under audit, upstream of all physics:
  - Verify units, scale, and datum against at least two independent
    published dimensions of the system (two different axes or
    features, not one quantity the scale factor was fitted to). A
    transform validated against only the quantity used to derive it
    is unvalidated.
  - Check geometric closure where the physics assumes it: a clipped or
    immersed mesh must have no open boundaries other than the declared
    cut. Count boundary edges, quantify open-seam length, and treat
    integral quantities (volume, area, mass) computed on a non-closed
    domain as invalid, not approximate.
  - Cross-check one conserved integral against an independently
    published value for canonical systems (a domain's computed volume,
    area, mass, or total charge vs its documented value). This is the
    cheapest whole-pipeline test that exists; its absence is a finding.
  - Hardcoded selections (surface/patch tags, column indices, band or
    crop ranges) require a recorded rationale and a completeness check.
    An unexplained magic subset of a source artifact is a provenance
    gap even when the result "looks right".
- **Gross-deviation differential diagnosis.** When a computed quantity
  for a well-characterized system misses an independent reference by
  far more than the plausible error budget, the null hypothesis is a
  defect in the computation, not in the reference or source artifact.
  Before accepting any attribution to external data, walk the standard
  defect ladder and show your work: units → scale → datum → selection/
  closure → sign/convention → the data itself. "Source data deficient"
  requires the same affirmative evidence bar as "fabrication"; a gate
  failure alone attributes nothing.

## 7. Performance and engineering claims

- Audit this phase only when performance or engineering claims are in
  scope. Give uncited superlatives ("fastest", "most accurate") their
  own finding: what, if anything, backs them?
- Execution mode: run benchmarks and verify they do real work (inputs
  black-boxed, outputs consumed, physics inside the measured loop).
  Verify allocation/latency proofs actually instrument the measured
  region. Force failure paths and confirm fallbacks are loud (typed,
  reported), never silent degradation.

## 8. The report

Use [references/report-template.md](references/report-template.md).
Rank adverse findings by impact, but keep these fields independent:

- **Verdict**: verified, partially verified, unverified, contradicted,
  or out of scope. In static mode use artifact-traceable, untraceable,
  or contradicted instead.
- **Finding type**: circular oracle, vacuous test, provenance gap,
  tolerance mismatch, mutation survivor, equation-transcription error,
  regime violation, unverifiable citation, unsupported claim,
  misattributed discrepancy (the deviation is real but blamed on the
  wrong cause — e.g. a pipeline defect reported as a source-data
  deficiency), or another precise mechanism.
- **Impact**: critical, high, medium, or low. Conservative labeling
  does not cap impact: a wrong causal attribution wrapped in honest
  "diagnostic only" caveats is rated by the consequences of the wrong
  conclusion (abandoned datasets, unnecessary procurement, a fixable
  defect left standing), not by the modesty of its wrapper.
- **Confidence**: high, medium, or low.

Reserve fabrication for a separately justified allegation backed by
affirmative evidence. Every finding must cite exact file:line, commit,
primary-source location, command output, or shown derivation; label the
evidence class; and name the documented claims affected. Include the
full mutation table, oracle-independence table, equation-verification
ledger (equation → source location → term-by-term result), and a
claim-by-claim verdict for the Phase-0 ledger.

In static or mixed mode, list everything execution could not establish.
End with exact commands and results, environment details, branch,
commit, initial and final dirty state, excluded configurations, and
skipped checks. A skipped or unavailable check is never a pass.

Do not grade on a curve; do not pad with praise. If the suite survives
the mutation campaign and the claims trace, say so plainly — that is
the strongest possible answer to "how do you know it's correct" — but
only after genuinely trying to kill it.
