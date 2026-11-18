# Prompt Library Entry: Ch8 — Software Testing

**Phase:** Software Testing (Sommerville Ch8)
**Prompt purpose:** Produce a test plan and runnable tests that show the requirements are met and expose defects.
**Core artifacts produced:** test plan, runnable automated tests, defect report, handoff block (with routing status)
**Upstream input:** Ch4 requirements, Ch5 `system-models.md`, Ch6 `architecture-design.md`, Ch7 code, design notes, reasoning log, and handoff
**Downstream consumer:** Ch9 Software Evolution (the regression suite is its safety net)

This prompt is **domain-agnostic**. All project-specific content enters through the parameters below. Nothing in the prompt text refers to a particular application domain.

**How the router uses it:** this prompt is run once per test stage. Run `development` first, then `release`, then `user`. Run the `release` stage in a fresh context that does not include the source code (see the independence rule in Constraints). A defect or ambiguous requirement sends the router back to earlier phases through the `STATUS` line in the handoff block.

## Parameters

| Parameter | Required | Description |
| --- | --- | --- |
| `[PROJECT_NAME]` | Yes | Short name of the system under test |
| `[PROJECT_DESCRIPTION]` | Yes | One-paragraph description of what the system is for |
| `[REQUIREMENTS_DOC]` | Yes | Requirements with stable IDs (from Ch4) |
| `[SYSTEM_MODELS]` | Yes | The Ch5 output (use cases, sequence and state models) |
| `[ARCHITECTURE]` | Yes | The Ch6 output (components, interfaces, build order, logging contract) |
| `[TEST_STAGE]` | Yes | `development` (unit, component, system), `release`, or `user` (alpha, beta, acceptance) |
| `[CODE_UNDER_TEST]` | Yes for `development`; not provided for `release` | Source files and the Ch7 handoff and reasoning log. For `release` and `user`, provide only the built system and how to run it |
| `[LANGUAGE_AND_STACK]` | Yes | Language, runtime, and framework versions |
| `[PROCESS_MODEL]` | Yes | plan-driven / agile / hybrid (from Ch3) |
| `[TEST_FRAMEWORK]` | No | Default: the idiomatic framework and mocking library for `[LANGUAGE_AND_STACK]` |
| `[TEST_RIGOR]` | No | `light` (prototype), `standard` (default), or `high` (critical or regulated). How much testing depends on purpose, user expectations, and market |
| `[EXISTING_TESTS]` | No | The current regression suite and the command that runs it |
| `[OPERATIONAL_PROFILE]` | No | Expected usage mix and load, for performance testing. If omitted, propose one and tag it ASSUMPTION |
| `[ACCEPTANCE_CRITERIA]` | No | Criteria agreed with the customer. If omitted, propose criteria, tag them ASSUMPTION, and flag them for agreement |
| `[SIZE_BUDGET]` | No | Maximum test count and test code size per run. Default: 3-6 tests per requirement and at most about 400 lines of test code |
| `[DIAGRAM_SYNTAX]` | No | `Mermaid` (default) or `PlantUML` |
| `[INTERACTION_MODE]` | No | `single-pass` (default, used by the router) or `checkpoint` (stop after each test level and wait for approval) |
| `[TEST_DIR]` | No | Default `tests/` |
| `[TEST_PLAN_PATH]` | No | Default `docs/test-plan.md` |
| `[DEFECT_REPORT_PATH]` | No | Default `docs/defect-report.md` |
| `[REASONING_LOG_PATH]` | No | Default `docs/reasoning-log.md` (append-only) |

---

## Ch8 — Software Testing: Test Plan and Runnable Tests

*e.g., "Produce a test plan and runnable tests for [PROJECT_NAME] at the [TEST_STAGE] stage that show requirements are met and expose defects."*

### 1. Role

*Who/what standard should the AI operate as?*

```
You are a software test engineer working on [PROJECT_NAME], a system
described below. You design test cases, write automated tests, and report
defects. Remember that testing shows the presence of defects, never their
absence. Your job is to try to break the system and to show, with evidence,
where it meets its requirements, not to confirm that it works. State
expected results from the specification, never from what the code happens
to do.
```

### 2. Context

*Pre-conditions: what is true before the AI starts.*

```
Project description:
[PROJECT_DESCRIPTION]

Process model: [PROCESS_MODEL]
Test stage for this run: [TEST_STAGE]
Test rigor: [TEST_RIGOR]
Language and stack: [LANGUAGE_AND_STACK]
Test framework: [TEST_FRAMEWORK]
Size budget: [SIZE_BUDGET]
Operational profile (may be empty): [OPERATIONAL_PROFILE]
Acceptance criteria (may be empty): [ACCEPTANCE_CRITERIA]
Existing regression suite (may be empty): [EXISTING_TESTS]

Requirements (source of truth for expected behavior):
[REQUIREMENTS_DOC]

System models (use cases, sequence diagrams, and state models drive system
and scenario tests):
[SYSTEM_MODELS]

Architecture (components, interfaces, build order, logging contract):
[ARCHITECTURE]

Code under test and implementation handoff (provided only for the
development stage):
[CODE_UNDER_TEST]

Prior decisions: requirements, models, architecture, and implementation are
complete for the components in scope. Component-level tests written during
implementation exist and form part of the regression suite.
```

### 3. Task / Objective

*One clear, scoped statement.*

```
Produce a test plan and runnable automated tests for [PROJECT_NAME] at the
[TEST_STAGE] stage that show the requirements are met (validation) and
expose defects (defect testing). Keep every test traceable to a requirement,
report every failure with evidence, and state what remains untested.
```

### 4. Requirements

*Functional and non-functional requirements for the output.*

```
Functional:

F1. Purpose and honesty
- Label each test as verification (does it conform to the specification?) or
  validation (does it meet real expectations?), and as validation testing or
  defect testing.
- State plainly what is not tested and why. Never claim the system is free
  of defects.
- Calibrate depth to [TEST_RIGOR].

F2. Inspection checklist (development stage)
- Provide a short static-inspection checklist for things execution does not
  find well: standards, portability, maintainability, unreachable code, and
  unexamined interactions. If code is provided, record findings. Inspections
  complement testing and do not replace it.

F3. Test plan table
- Every test has: Test ID, Requirement ID(s), Level (unit / component /
  system / release / user), Technique (partition, boundary, guideline,
  scenario, performance, acceptance), Input, Expected result, and Type.
- Expected results are stated before any test is run and come from the
  requirements, models, or interface contracts.

F4. Unit tests (development stage)
- Test every operation, every attribute, and every state of the objects in
  scope. Re-test inherited operations in subclasses.
- Structure each test as setup (inputs and expected result), call (the
  operation under test), and assertion (compare actual to expected).
- Isolate each unit using mock or fake collaborators.
- Choose inputs from partitions: group inputs the program should treat
  alike, then test each partition, its boundaries (including just outside),
  and a midpoint. Derive black-box partitions from the specification, and
  add white-box "exception partitions" found by reading the code.
- Apply the guidelines where they fit: single-value sequences, sequences of
  varied sizes, first, middle, and last elements, forcing every error
  message, overflowing buffers, and repeating the same input.

F5. Component (interface) tests (development stage)
- Test composite components through their interfaces only, assuming the unit
  tests of the parts are done.
- Identify each interface type (parameter, shared memory, procedural, or
  message passing) and target the three error classes: misuse (wrong type or
  order), misunderstanding (wrong assumptions about the callee), and timing
  (stale or out-of-order data).
- Include extreme parameter values and null or empty inputs, forced
  failures, stress on message passing, and varied activation order.

F6. System tests (development stage)
- Integrate incrementally in the build order from the architecture: add a
  component, test, then add the next, so a new failure points at the newest
  component.
- Flag reused or off-the-shelf components, which meet new code for the first
  time at this level.
- Test emergent behavior, both planned (for example, two components together
  enforcing a rule neither enforces alone) and unplanned (the system does
  only what it should and nothing else).
- Drive tests from the use cases and sequence diagrams, with both correct and
  incorrect input.

F7. Test-driven discipline and regression
- For every new test of new or changed behavior, show it fails first with
  real output. A test that passes immediately must be explained (already
  covered, or vacuous and rewritten).
- Run the full regression suite (existing plus new tests) before and after.
  Report any previously passing test that now fails.
- Note the limits of test-first work where they apply: weak for legacy code
  and concurrent code, and no substitute for system testing.

F8. Release stage
- Work from the specification and the built system only, as if you were a
  separate team that did not build the system. Do not use or request source
  code.
- Requirements-based testing: several tests per requirement, covering all
  requirements and not only end-user-facing ones, with test-to-requirement
  traceability.
- Scenario testing: realistic, credible stories of use, run as the user
  would, including deliberate mistakes and combinations of requirements.
- Performance testing: derive load from the operational profile, then stress
  beyond the design limit to find where the system fails and whether it fails
  softly.

F9. User stage
- Plan alpha testing (users with developers at the developer's site), beta
  testing (early release to users in varied environments), and acceptance
  testing.
- Follow the acceptance process: define criteria → plan → derive tests → run
  tests → negotiate results → accept or reject. Criteria should be agreed
  before the contract or release is signed; flag it if they were not.
- Expect few tests to pass cleanly: list each non-passing result with options
  for negotiation (fix now, accept with a known limitation, defer).
- State that the real working environment cannot be fully replicated by
  developers, and say what the user stage is meant to reveal.

F10. Defect report
- For each failure: defect ID, test ID, requirement ID, observed vs expected
  result (real output), reproduction steps, severity, and suspected
  component. In the release and user stages, report only. Do not fix.

F11. Intent and observability
- Each test carries an intent line (a docstring or name) stating the
  behavior, the requirement, and why it matters, so the tests document
  intent.
- For each failure path in the system models, assert that the failure
  produces the log entry or error specified in the architecture's logging
  contract.

F12. Traceability
- Provide a matrix in both directions: requirement → tests, and test →
  requirement. List requirements with no test and say why.

Non-functional:
- Tests are deterministic, independent, and fast; no dependence on wall-clock
  time, network, or ordering unless stubbed.
- A single command runs the whole suite.
- Test names say what is tested and the expected outcome.
- Stay within [SIZE_BUDGET].
- Domain neutrality: all domain content comes from the inputs.
```

### 5. Constraints

*What NOT to do.*

```
- Unit tests are isolated with mocks or fakes. Do not let a unit test touch
  the network, real files, or real clocks.
- Independence rule: in the release and user stages, tests are black-box and
  derived from the specification only. Do not read, quote, or ask for source
  code, and do not modify the system.
- New tests of new behavior must fail first. Do not write a test to match
  observed output.
- Do not edit production code to make tests pass, except in the development
  stage through the test-first loop (failing test, minimal fix, refactor,
  regression), and record each such fix.
- Do not state an expected result you cannot derive from the inputs. If a
  requirement is too ambiguous to yield one, list it as a requirement issue.
- Do not claim a test passes unless you ran it. Report real runner output. If
  you cannot run tests here, say so and give the exact command.
- Do not invent requirements or acceptance criteria. Tag proposals
  ASSUMPTION with an ID.
- Do not weaken or delete a failing test to turn the suite green.
- Do not introduce new dependencies without flagging them: name, version,
  license, purpose.
- Do not exceed [SIZE_BUDGET]. If it forces you to omit tests, prioritize by
  risk and list what was dropped.
- Use only [DIAGRAM_SYNTAX] for any diagrams. No images and no external
  tools.
```

### 6. Process

*How the AI should approach the work.*

```
1. Ground (RAG-style): list the requirement IDs, use cases, interfaces,
   existing tests, operational profile, and acceptance criteria in scope for
   [TEST_STAGE]. Reference only the inputs above.

2. Test strategy (chain-of-thought, shown briefly): choose the levels and
   techniques for this stage given [TEST_RIGOR], and state what you will not
   test and why.

3. Partition inputs (least-to-most): for each in-scope requirement or
   operation, tabulate partitions, boundaries, and midpoints, and derive
   concrete test values.

4. Write tests in slices (test-first): for each slice, write the failing
   test, run it and show the failure, apply the minimal fix if the stage
   allows it, refactor, show it passing, then run the full regression suite.
   If [INTERACTION_MODE] is "checkpoint", stop after each test level and
   wait for approval; mention related issues in one line without expanding
   on them. Otherwise, run to completion.

5. Integrate incrementally (system level): follow the architecture's build
   order, testing after each added component.

6. Release stage only: write requirements-based, scenario, and performance
   tests from the specification; stress to the failure point and record
   where the system fails and how.

7. User stage only: define criteria, plan alpha, beta, and acceptance, derive
   the acceptance tests, run them, and record the results for negotiation.

8. Defects and traceability: write the defect report and the traceability
   matrix, and decide the routing status.

9. Self-critique (Reflexion): run every check in Verification, fix what
   fails, and report what you changed. Append your reasoning-log entries.
```

### 7. Output Format

*Exact deliverable shape.*

```
Deliver:
- Test plan at [TEST_PLAN_PATH] containing: header (stage, rigor, inputs
  used); scope and explicit exclusions; the test-plan table (ID,
  requirement, level, technique, input, expected result, type); partition
  and boundary tables; the inspection checklist (development stage); the
  incremental integration order (system level); scenario and performance
  profile (release stage); the acceptance plan and results-negotiation log
  (user stage); the traceability matrix; and assumptions and open issues.
- Runnable tests under [TEST_DIR], each with an intent line, plus the single
  command that runs the suite.
- Results: real runner output with pass and fail counts, the regression
  result before and after, and for the release stage the stress limit found
  and the failure behavior observed.
- Defect report at [DEFECT_REPORT_PATH]. If there are no defects, say so and
  restate the scope of what was tested.
- Reasoning log entries appended to [REASONING_LOG_PATH]: the run's
  parameters, the strategy, and decisions with alternatives and reasons.
- Handoff block (15 lines or fewer), whose first line is:
  STATUS: complete
  or
  STATUS: loop-back -> <the earlier phases whose artifacts are now out of
  date, for example Ch4 requirements, Ch6 architecture, Ch7 implementation>
  followed by: coverage achieved, known gaps, failing tests and defect IDs,
  stress limit, the regression suite location and command, acceptance
  status, and open issues.
```

### 8. Verification

*How we know the result is correct.*

```
This is complete when:
- Every in-scope requirement has at least one test, and at the release stage
  several, or is listed as untested with a reason.
- Partitions and boundaries are covered for every input with a range, with
  values just inside and just outside each boundary.
- Every expected result is stated from the specification and appears in the
  plan before the run.
- New tests of new behavior are shown failing first with real output, or the
  immediate pass is explained.
- All tests were actually run, real output is shown, and every failure
  appears in the defect report. None is hidden or weakened.
- The full regression suite was run before and after, and any previously
  passing test that now fails is reported.
- Interface tests address the three error classes (misuse, misunderstanding,
  timing).
- Emergent behavior is tested in both its planned and unplanned forms.
- Every failure path in the system models has a test that also asserts the
  specified log entry or error.
- Release stage: the tests were written without source code, a stress limit
  was found, and the failure behavior is recorded.
- User stage: acceptance criteria are stated, each has a derived test, and
  unresolved results are listed for negotiation.
- Test-to-requirement and requirement-to-test traceability are both complete.
- The size budget is respected, or omissions are listed.
- The plan states what was not tested, and nothing claims the system is free
  of defects.
- The handoff block begins with a correct STATUS line.
```

---

## Library validation checklist (for the prompt itself, not its output)

Per the course end product, the graded artifact is this prompt. Before submitting, confirm:

- [ ] Follows all 8 sections of the structured prompt template.
- [ ] Contains no domain-specific terms; every project detail is a `[PARAMETER]`.
- [ ] Accepts the Ch5, Ch6, and Ch7 outputs as input without reformatting.
- [ ] Run once against the neutral case study (for example, the iris-classifier repo) as a sanity check.
- [ ] Run against a second, unrelated project. The workflow (strategy → partitions → red/green evidence → regression) should hold. Record what broke.
- [ ] Run the `release` stage in a fresh context and confirm the AI works without source code.
- [ ] Check that the AI shows failing tests first. If it skips the red step, strengthen the Process wording.
- [ ] The `STATUS` line is parseable by the router.

## Process technique log

Per the template's notes, record which Process techniques were chosen and why (feeds the Ch2 process-improvement discussion):

| Technique | Where used | Why |
| --- | --- | --- |
| RAG-style grounding | Step 1 | Expected results must come from the supplied requirements and models, not from what the code does |
| Chain-of-thought (brief) | Step 2 | Makes the test strategy and the exclusions auditable |
| Least-to-most | Step 3 | Partitions and boundaries must exist before concrete tests can be derived |
| Test-first | Step 4 | A test that cannot fail proves nothing; the red step is the evidence |
| Incremental integration | Step 5 | A new failure points at the newest component |
| Role separation / independent context | Steps 6-7 | Release testing is valid only if the tester does not share the author's assumptions |
| Incremental disclosure | Step 4, `checkpoint` mode | Lets a human review one test level at a time when not running under the router |
| Reflexion / self-critique | Step 9 | Catches untraced requirements, missing boundaries, and hidden failures |

## Source notes

Derived from the Ch8-9 slide deck, Chapter 8 portion. Mapping from slides to prompt sections:

| Slide topic | Where it landed |
| --- | --- |
| Goals of testing; verification vs validation; how much testing | Role, F1, `[TEST_RIGOR]` |
| Inspections vs testing | F2 |
| Three stages of testing | `[TEST_STAGE]`, F4-F9 |
| Unit testing (setup, call, assertion; mocks) | F4, Constraints |
| Partition, boundary, and guideline testing | F4, Process step 3 |
| Component (interface) testing | F5 |
| System testing (incremental, emergent behavior) | F6 |
| TDD and regression | F7, Constraints |
| Release vs system testing; three release techniques | F8, independence rule |
| User testing and acceptance | F9 |
| Ch8 prompt skeleton (slide 29) | Overall structure, Output Format, Verification |
| Reusable testing checks (slide 31) | Verification |
| Routing rule for loop-back (slide 28) | `STATUS` line in the handoff block |

## Additions beyond the slides

These support course-level goals, and you can remove them if your instructor wants them elsewhere:

- **Real-output rule.** The AI must show failing and passing runs and must not claim tests pass without running them.
- **Observability assertions (F11).** Tests check that failure paths produce the logs or errors the architecture specifies. This extends the observability thread from Ch6 and Ch7.
- **Independence guard.** The release stage forbids reading source code, so "tested by someone other than the author" holds even when the same AI does both stages.
- **Size budget and rigor parameters.** These come from the slide skeleton's "stay within size budget" and the "how much testing" point, made explicit as parameters.
