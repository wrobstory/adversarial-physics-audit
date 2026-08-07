# adversarial-physics-audit

An agent skill that runs an adversarial, evidence-disciplined
correctness audit of a physics, simulation, scientific-computing, or
numerical library. It exists to answer one question honestly: **"How
do you KNOW this code is correct — that it wasn't hallucinated, that
the test data isn't hardcoded?"**

The auditor assumes the implementation may be wrong until evidence
forces it off that position. Docs and READMEs are treated as claims,
not evidence. The hostile posture generates experiments, not
accusations: suspicious patterns and evidence gaps are never reported
as fabrication without affirmative evidence.

Works with any agent that reads the open SKILL.md format (Claude Code,
Codex CLI, and others).

## What the audit does

1. **Maps the target** — inventories every in-scope quantitative claim,
   the numerical environment, every selected configuration, every
   test (including ignored/uncompiled ones), all reference data with
   its claimed provenance, all oracles, and the governing equations.
   Every in-scope claim gets a verdict; exclusions and sampling are
   explicit.
2. **Controlled mutation analysis** — establishes a stable baseline,
   then seeds one deliberate physics bug at a time in a disposable tree:
   sign-flipped restoring terms, perturbed constants, transposed
   couplings, frozen integrator sub-states, or corrupted data. It
   verifies reachability, rebuilds, classifies invalid/equivalent/flaky
   mutants, and restores the baseline before drawing conclusions.
3. **Hardcoded-expectation hunting** — traces the origin of every
   expected value in the tests, reconstructs tolerance history from
   version control, and flags vacuous or tautological assertions.
4. **Circularity analysis** — grades oracle independence across shared
   equations, data, conventions, dependencies, calibration, and
   authorship, then checks model-specific invariants with their
   boundary-condition and forcing preconditions stated.
5. **Numerical and statistical integrity** — examines error budgets,
   conditioning, absolute/relative metrics, convergence regimes,
   precision and hardware effects, stochastic power, and flakiness.
6. **Data provenance** — treats checksums as self-consistency only and
   separately checks source authority, transcription, transformations,
   and applicability. Suspicious numeric patterns are screening signals,
   not proof.
7. **Claims audit** — when relevant, checks benchmarks that do real
   work, fallbacks that fail loudly, and a finding for every uncited
   superlative.

The report keeps verdict, finding type, impact, and confidence separate;
labels evidence as reproduced, derived, primary-sourced, inferred, or
unverified; and includes reusable claim, mutation, oracle, and provenance
ledgers.

### Three modes

- **Execution mode** — the agent builds, runs tests, seeds mutations,
  regenerates fixtures. Anything not personally reproduced is reported
  as unverified.
- **Static mode** — for read-only access. The words "passes" and
  "works" are banned; evidence is *artifact-traceable* /
  *untraceable* / *contradicted*, and mutation predictions are
  *predicted-caught* / *predicted-uncaught* / *indeterminate*.
- **Mixed mode** — execution verdicts apply only to reproduced
  configurations; everything else retains static evidence labels.

Three standing rules guard against the auditor's own failure modes: do
not trust memory of constants or papers; treat docs as pointers rather
than facts; and keep reproduced, derived, sourced, inferred, and
unverified evidence distinct.

## Install

Claude Code (personal skill, all projects):

```bash
git clone https://github.com/wrobstory/adversarial-physics-audit
cp -r adversarial-physics-audit/skills/adversarial-physics-audit ~/.claude/skills/
```

Codex CLI — same format, so symlink the same copy:

```bash
ln -s ~/.claude/skills/adversarial-physics-audit ~/.codex/skills/adversarial-physics-audit
```

Or copy it into a repo's `.claude/skills/` to scope it to one project.

## Use

Ask the agent things like:

- "How do we know this solver is actually correct? Run an adversarial
  audit."
- "Audit the validation suite — assume the test data could be
  hardcoded."
- "Do a read-only hostile review of this repo's physics claims."

The skill triggers on correctness-skepticism phrasing; you can also
invoke it by name.

## Origin

Distilled from a pair of hand-written audit prompts for a ship-motions
solver, where the pointed question above was actually asked. The
generic version replaces library-specific targets with a discovery
phase and derives the mutation list from the target's own physics.

## License

MIT
