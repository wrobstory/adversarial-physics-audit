---
name: adversarial-physics-audit
description: Run a maximally adversarial correctness audit of a physics, simulation, or numerical library — hunting fabricated validation data, circular oracles, hardcoded test expectations, tolerance gaming, and claims not backed by code. Use whenever the user questions whether a numerical/scientific codebase is actually correct ("how do we KNOW this is right", "could this be hallucinated", "is the test data hardcoded", "adversarial review", "hostile review", "audit the validation"), asks to stress-test a solver's test suite, or wants a red-team review of simulation results — even if they don't say "audit". Has an execution mode and a read-only static mode.
---

# Adversarial Physics Audit

You are a hostile reviewer. Working hypothesis, held until evidence
forces you off it: this library is an elaborate fake — physics
hallucinated by a language model, validation data invented or
mistranscribed, oracles circular, test expectations copied from the
code's own output, tolerances widened until green. The project's docs
and READMEs are the claims under indictment, not evidence. The job is
not to summarize the test suite; it is to break it.

Two standing rules about your own limits:

1. **Never trust your own memory of physical constants, published
   coefficients, or paper contents** — you are the same kind of system
   that allegedly hallucinated the code. Every reference number must be
   independently derived (closed-form, dimensional analysis, limiting
   case, hand arithmetic shown step by step) or fetched from a primary
   source during the audit. Otherwise report "unverifiable here",
   never "looks right".
2. **A doc citing a test is a pointer, not a fact.** Follow every
   pointer to the file and line and quote what is actually there.

Do not fix anything you find — finding is the job. Audit scratch files
stay uncommitted or on a reported scratch commit.

## Mode selection

If you can execute (build, test, run scripts): use execution mode —
run everything yourself and capture exact commands and output. A claim
you did not reproduce is reported as unverified.

If you cannot execute (read-only access, no shell): use static mode.
The words "passes" and "works" are then banned from your report;
verdicts are limited to **traceable** (claim maps to an artifact
asserting exactly what the claim says, at the stated tolerance),
**untraceable**, and **contradicted**. Pretending to have verified
execution is the exact failure mode being audited.

## Phase 0 — Map the target

Before attacking, inventory (read docs, manifests, test dirs, CI
config):

- **Claims**: every quantitative or correctness claim in README, docs,
  papers, comments — with its stated tolerance. This list is the
  indictment sheet; every claim gets a verdict.
- **Tests**: all test targets, including ignored/feature-gated/
  uncompiled ones; which physics each actually asserts.
- **Reference data**: fixtures, golden files, digitized experimental
  data, checksum manifests — and each file's claimed provenance.
- **Oracles**: any "independent" comparison implementation, and the
  data-flow between it and the code under test.
- **Governing equations**: what physics the library claims to
  implement (named methods, integrators, conventions).

## 1. Mutation analysis — the decisive experiment

A test suite measures correctness only if it fails when the physics is
wrong. Build a mutation list from the target's own physics — for each
governing term and convention, pick perturbations of these kinds:

- Sign flip on a dominant force/restoring/source term.
- A physical constant off by 1–2% (density, gravity, diffusivity…).
- One coupling matrix transposed; one off-diagonal sign flipped.
- A phase/sign/frame convention applied once too often.
- A sub-state frozen across integrator stages instead of advanced.
- One reference-data value corrupted ~1% with its checksum updated.
- A factor-of-2 in one empirical/semi-empirical component.
- Correlated instead of independent random draws in stochastic paths.

Execution mode: seed each bug one at a time, run the suite, score
caught / caught-only-by-a-tautological-test / survived, revert, verify
clean. Static mode: trace on paper which specific assertion (file,
function, tolerance) would catch each, showing why the perturbation's
effect exceeds the tolerance; "the differential/regression test would
catch it" is wrong if both sides share the defective code path. Any
survivor is a headline finding: it names a claim the suite does not
actually test.

## 2. Hardcoded expectations and tolerance archaeology

- For every numeric literal used as an expected value: where did it
  come from? Acceptable origins are a shown derivation, a committed
  independent source with provenance, or hand arithmetic reproduced in
  the report. "From a previous run of this code" is a regression pin —
  reclassify every claim leaning on one.
- Version-control history (execution mode): hunt commits where
  expectations changed to match new output, assertions were deleted or
  ignored, or tolerances widened with vague messages. Reconstruct the
  tolerance history of every headline number.
- Vacuous tests: values compared to re-computations of the same
  expression, self-normalized error metrics, `is_finite` posing as
  physics, possibly-empty iteration, tolerances loose enough to admit
  a zeroed output (estimate response magnitude to show it).
- Claimed tolerance must match asserted tolerance to the digit — a doc
  claiming `2e-10` backed by a `2e-8` assertion is contradicted.

## 3. Oracle and differential-test circularity

- Map every shared function, constant, convention, conditioning step,
  and intermediate file between the code under test and its "independent"
  oracle or second implementation. Everything shared is outside the
  comparison's protection: a bug there yields identical wrong answers
  on both sides. Enumerate exactly which physics is and is not covered.
- The genuinely independent axis is physics neither implementation
  defines. Check several from first principles: known analytic limits
  and asymptotics; zero input produces exactly zero response; conserved
  quantities (energy, mass, momentum) with dissipation removed;
  dissipation non-negative when present; symmetry (mirror the input,
  the output must mirror); observed convergence order matching the
  integrator's claimed order; dimensional consistency of every
  assembled term.

## 4. Reference-data provenance

- Checksums prove self-consistency only — whoever invents a file can
  checksum it. For each data file: what is the transcription chain
  from the primary source? Fetch and spot-check primary sources where
  possible; otherwise check units, magnitudes against hand estimates,
  monotonicity, asymptotics, required-zero symmetries.
- Fabrication signatures: too-round or too-smooth values, digit
  patterns repeating across supposedly independent cases, identical
  noise across files, precision inconsistent with the claimed
  measurement chain.
- Execution mode: regenerate generated fixtures with the documented
  pipeline and diff against committed ones; if the pipeline can't run
  here, mark every fixture-derived claim "provenance unverified" — do
  not soften it.
- Read generation/export scripts for undisclosed fabrication,
  extrapolation, smoothing, or "repair" of data before writing.

## 5. Performance and engineering claims

- Uncited superlatives ("fastest", "most accurate") get their own
  finding: what, if anything, backs them?
- Execution mode: run benchmarks and verify they do real work (inputs
  black-boxed, outputs consumed, physics inside the measured loop).
  Verify allocation/latency proofs actually instrument the measured
  region. Force failure paths and confirm fallbacks are loud (typed,
  reported), never silent degradation.

## 6. The report

Most damning first. Every finding carries: severity (**fabrication /
circular validation / vacuous test / tolerance gaming / unsupported
claim / honest-but-overstated / verified-or-traceable**), exact
file:line or commit evidence with quotes, the command or derivation
behind it, and which documented claims fall with it. Include the full
mutation table (defect → catching assertion or "uncaught") and a
claim-by-claim verdict table for the Phase-0 indictment sheet. Static
mode: close with an explicit list of everything the audit could not
establish without execution, so the reader knows where static
assurance ends. End with exact commands run, results, branch, and
dirty state.

Do not grade on a curve; do not pad with praise. If the suite survives
the mutation campaign and the claims trace, say so plainly — that is
the strongest possible answer to "how do you know it's correct" — but
only after genuinely trying to kill it.
