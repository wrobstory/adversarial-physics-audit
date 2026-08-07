# adversarial-physics-audit

An agent skill that runs a maximally adversarial correctness audit of a
physics, simulation, or numerical library. It exists to answer one
question honestly: **"How do you KNOW this code is correct — that it
wasn't hallucinated, that the test data isn't hardcoded?"**

The auditor's working hypothesis is that the library is an elaborate
fake, held until the evidence forces it off that position. Docs and
READMEs are treated as the claims under indictment, not as evidence.

Works with any agent that reads the open SKILL.md format (Claude Code,
Codex CLI, and others).

## What the audit does

1. **Maps the target** — inventories every quantitative claim, every
   test (including ignored/uncompiled ones), all reference data with
   its claimed provenance, all oracles, and the governing equations.
   The claim list becomes an indictment sheet; every claim gets a
   verdict.
2. **Mutation analysis** — the decisive experiment. Seeds deliberate
   physics bugs (sign-flipped restoring terms, perturbed constants,
   transposed couplings, frozen integrator sub-states, corrupted data
   with updated checksums) and scores whether any test catches each
   one. A surviving mutation names a claim the suite does not actually
   test.
3. **Hardcoded-expectation hunting** — traces the origin of every
   expected value in the tests, reconstructs tolerance history from
   version control, and flags vacuous or tautological assertions.
4. **Circularity analysis** — maps everything shared between the code
   and its "independent" oracles or second implementations, then
   cross-checks with physics neither side defines: analytic limits,
   conservation laws, symmetry, convergence order, dimensional
   consistency.
5. **Data provenance** — treats checksums as self-consistency only,
   spot-checks reference data against primary sources, and hunts
   fabrication signatures (too-smooth values, repeated digit patterns,
   impossible precision).
6. **Claims audit** — benchmarks that do real work, fallbacks that
   fail loudly, and a finding for every uncited superlative.

The report is ranked most-damning-first with a fixed severity
taxonomy: fabrication, circular validation, vacuous test, tolerance
gaming, unsupported claim, honest-but-overstated, verified.

### Two modes

- **Execution mode** — the agent builds, runs tests, seeds mutations,
  regenerates fixtures. Anything not personally reproduced is reported
  as unverified.
- **Static mode** — for read-only access. The words "passes" and
  "works" are banned; verdicts are limited to *traceable* /
  *untraceable* / *contradicted*, and mutation analysis is done on
  paper by tracing which assertion would catch each defect. The report
  must end with an explicit list of what could not be established
  without execution.

Two standing rules guard against the auditor's own failure modes: it
may not trust its own memory of physical constants or paper contents
(every reference number must be derived or fetched during the audit),
and a doc citing a test is a pointer, not a fact — it must follow the
pointer and quote what is actually there.

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
