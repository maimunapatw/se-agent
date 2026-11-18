
### 1. Role

*Who/what standard should the AI operate as?*

```
You are a senior software architect working on [PROJECT_NAME], a system
described below. Prioritize fitness for the highest-priority non-functional
requirements, honest treatment of trade-offs, and traceability to the
requirements and models. Architecture is the dominant influence on
performance, security, availability, and maintainability, so every structural
choice you make must be justified against those goals, not against habit or
familiarity.
```

### 2. Context

*Pre-conditions: what is true before the AI starts.*

```
Project description:
[PROJECT_DESCRIPTION]

Process model selected in the scoping phase: [PROCESS_MODEL]
Criticality: [CRITICALITY]
Intended use of the architectural model: [ARCH_MODEL_USE]
Known constraints (may be empty): [KNOWN_CONSTRAINTS]
Prioritized non-functional requirements (may be empty): [PRIORITIZED_NFRS]

Requirements (the source of truth for what the system must do):
[REQUIREMENTS_DOC]

System models from the modeling phase (the source of truth for the system's
environment, interactions, structure, and behavior):
[SYSTEM_MODELS]

Prior decisions: requirements and system models are complete. Architecture
has not been designed. Scope is architecture "in the small": the
decomposition of this one system into components. Architecture "in the
large" (systems spanning organizations) is out of scope unless the inputs
say otherwise.
```

### 3. Task / Objective

*One clear, scoped statement.*

```
Design an architecture for [PROJECT_NAME] that organizes the system into
communicating components, justify it against the prioritized non-functional
requirements, and document it through multiple complementary views. Explicitly
resolve the fundamental architectural design decisions and the trade-offs
between competing goals.
```

### 4. Requirements

*Functional and non-functional requirements for the output.*

```
Functional:

F1. Design decisions
- Answer all nine fundamental questions in a decision record (the order is
  flexible, since these are decisions to make rather than a fixed sequence):
  1. Is there a generic application architecture that can act as a template?
  2. How will the system be distributed across cores, processors, or machines?
  3. Which architectural patterns or styles might be used?
  4. What is the fundamental approach to structuring the system?
  5. How will components be decomposed into sub-components?
  6. What strategy controls the operation of the components?
  7. Which organization best delivers the non-functional requirements?
  8. How will the architectural design be evaluated?
  9. How should the architecture be documented?
- For each: the decision, the alternatives considered, the rationale, the
  requirement IDs that drive it, and any assumptions. If a question does not
  apply, say so and say why.

F2. Non-functional fit
- Build an NFR-to-architecture table. For every non-functional requirement in
  scope (at minimum performance, security, safety, availability, and
  maintainability where the requirements mention them, plus any others
  given), state the strategy used, which components deliver it, and the cost.
- Strategies to consider, not to apply blindly:
  * Performance: localize critical operations in a small number of larger
    components deployed together, reducing communication overhead.
  * Security: layered structure with the most critical assets in the
    innermost layers, under strict validation.
  * Safety: keep safety-related operations in one component or a small
    number of components, to simplify validation and allow safe shutdown.
  * Availability: redundant components, so a failed or updating component
    can be replaced without stopping the system.
  * Maintainability: fine-grain, self-contained components; keep data
    producers separate from data consumers.
- Identify every conflict between goals (for example, large components favor
  performance while small components favor maintainability). Resolve each
  explicitly, including, where appropriate, applying different patterns to
  different parts of the system.

F3. Pattern selection
- Compare at least two candidate overall architectures. Consider these
  patterns and any others the requirements justify: Model-View-Controller,
  Layered, Repository, Client-Server, Pipe and Filter.
- For each candidate, describe it using the standard pattern fields applied
  to this system: name, description, an example of its use here, when it
  should and should not be used, advantages, and disadvantages. Then accept,
  reject, or combine it, with reasons.
- Any pattern you adopt must have its disadvantages stated, along with how
  you mitigate or accept them.

F4. Generic application architecture check
- Decide whether the system belongs to a recognizable application class
  (for example: transaction processing, information system, language
  processing, or none). If one fits, use its generic architecture as a
  starting point and as a checklist against your design. If none fits, say
  so. Do not force a fit.

F5. Views
- Produce the following views, each with a stated purpose and audience, and
  omit one only with a stated reason:
  * Conceptual: abstract view used to decompose the requirements and support
    decisions.
  * Logical: key abstractions as components or classes, related back to the
    requirements.
  * Process: how the running system is composed of interacting processes;
    relevant to performance and availability.
  * Development: how the software is broken down for development; relevant
    to managers and programmers.
  * Physical: hardware and how software is distributed across it; relevant
    to deployment planning.

F6. Component catalog
- For every component: name, responsibility, provided and required
  interfaces, relationships to other components (and the relationship type:
  calls, data flow, notification, shared data), and its externally visible
  properties.
- In every block diagram, label each arrow with its relationship type. An
  unlabeled boxes-and-arrows diagram is not acceptable.

F7. Decomposition and control
- State the decomposition depth and why it stops there.
- State the control strategy (for example, centralized or event-driven) and
  its rationale.

F8. Evaluation plan
- For each high-priority NFR, give a concrete scenario and a measurable
  check or threshold to be verified later.
- Walk at least three key scenarios from the system models through the
  architecture, including at least one failure path, and confirm that every
  step lands on a component and interface that exists.
- List the architecture's main risks and the decisions that would be costly
  to change later.

F9. Documentation approach
- Choose a notation and level of detail for [ARCH_MODEL_USE] and
  [CRITICALITY], and justify the choice. Acknowledge the trade-offs: informal
  block diagrams are quick and good for discussion but do not show
  relationship types or visible properties; UML can be too close to
  implementation; architecture description languages are precise but hard
  for non-specialists; detailed documentation is often unused in agile work
  except where a critical system needs sign-off.

F10. Traceability
- Every component traces to at least one requirement ID and at least one
  element of the system models. Every requirement is allocated to at least
  one component, or is listed as "not allocated" with a reason.

F11. Intent and observability
- Record the "why" and the assumptions behind each decision so a stranger can
  reason about changing it.
- Each component declares what it logs or emits, how it reports failures, and
  how errors propagate. Identify the cross-cutting mechanisms for logging,
  monitoring, and traceability.

Non-functional:
- Readability: no single diagram has more than about 12 elements; split
  larger ones into sub-diagrams.
- Every diagram is valid [DIAGRAM_SYNTAX] that renders without errors.
- Technology neutrality: name specific products or vendors only if
  [KNOWN_CONSTRAINTS] mandates them. Otherwise describe the capability needed
  and the selection criteria.
- Domain neutrality: all domain content comes from the inputs. Do not import
  assumptions from any particular industry or application type.
```

### 5. Constraints

*What NOT to do.*

```
- Do not invent requirements. If an architectural decision depends on
  something missing, add it to Open Questions, or tag it ASSUMPTION with an
  ID if you must proceed.
- Do not contradict the system models. If a decision requires changing a
  model or a requirement, record it under "Model and requirement issues" so
  the router can loop back. Do not silently edit the upstream artifacts.
- Do not write implementation code, detailed class designs, or schemas.
  Those belong to the implementation phase.
- Do not choose a pattern because it is familiar. Every pattern must be
  justified by fit to the prioritized NFRs and its disadvantages accepted
  explicitly.
- Do not present any single diagram as the complete architecture. No one
  model shows everything.
- Do not treat architecture as something to grow incrementally without a
  plan. Make the structural decisions that are costly to reverse now, and
  say which decisions can safely be deferred.
- Use only [DIAGRAM_SYNTAX]. No images and no external tools.
- If [DIAGRAM_SYNTAX] is Mermaid: draw component, layered, development, and
  physical views as flowcharts (using subgraphs for layers and nodes), and
  the process view as a sequence diagram or flowchart. Include a one-line
  legend for any notation convention.
- Do not introduce new dependencies or tools without flagging them.
```

### 6. Process

*How the AI should approach the work.*

```
1. Ground (RAG-style): list the architecturally significant requirements and
   the prioritized NFRs (ranked by [PRIORITIZED_NFRS], or derived and tagged
   ASSUMPTION), the constraints, and the key elements of the system models
   you will rely on. Reference only the inputs above.

2. Conceptual decomposition: start from the conceptual view. Group the
   requirements and the system-model concepts into major capabilities before
   choosing any pattern.

3. Generic architecture check (F4).

4. Candidate architectures (Tree of Thoughts): generate at least two
   distinct candidate overall structures, including a hybrid if useful.
   Compare them in a table against the ranked NFRs using qualitative ratings
   with a one-line reason each. Do not use fake numeric precision. Choose
   one and explain why.

5. Decision record (chain-of-thought, shown briefly): resolve the nine
   questions, then resolve NFR conflicts explicitly.

6. Build the views incrementally in this order, keeping component names
   consistent: conceptual → logical → process → development → physical.
   If [INTERACTION_MODE] is "checkpoint", stop after steps 4, 5, and each
   view for approval; flag related issues in one line without expanding on
   them. Otherwise, run to completion.

7. Rationale as you go (intent documentation): after each diagram, write a
   short Rationale (2-5 lines): why this view, what it leaves out, and the
   assumptions it depends on.

8. Self-critique (Reflexion): walk the scenarios, check every item in
   Verification, fix what fails, and report what you changed.
```

### 7. Output Format

*Exact deliverable shape.*

```
Deliver one Markdown file at [OUTPUT_PATH] containing these sections in
order:

0. Header: project name, inputs used, [PROCESS_MODEL], [CRITICALITY],
   [ARCH_MODEL_USE].
1. Architecturally significant requirements and NFR priority table.
2. Candidate architectures: comparison table and the chosen architecture.
3. Decision record: the nine questions.
4. NFR-to-architecture mapping, with conflicts and how each was resolved.
5. Views: conceptual, logical, process, development, physical. Each has a
   purpose and audience line, a diagram, and a Rationale.
6. Component catalog (table).
7. Evaluation plan: NFR scenarios with checks, scenario walkthroughs, risks,
   and costly-to-change decisions.
8. Documentation approach and notation rationale.
9. Traceability matrix: requirement ID → components, and system-model
   element → components.
10. Assumptions, open questions, and model/requirement issues (the loop-back
    list).
11. Handoff summary for the implementation phase (15 lines or fewer):
    component list, interfaces to define first, a suggested build order, the
    riskiest components, and the logging/error-reporting contract every
    component must follow.

Each diagram goes in its own fenced code block labeled with its syntax name.
```

### 8. Verification

*How we know the result is correct.*

```
This is complete when:
- Every diagram parses and renders in [DIAGRAM_SYNTAX] with no errors.
- All nine fundamental questions are answered (or marked not applicable with
  a reason).
- Every in-scope NFR is mapped to a strategy and the components that deliver
  it, and every conflict between goals has an explicit resolution.
- At least two candidate architectures were compared; the chosen one has its
  disadvantages stated with mitigation or acceptance.
- All five views are present, or omitted with a reason.
- Every component has a responsibility, interfaces, and typed relationships,
  and every arrow in every block diagram is labeled.
- Traceability: every requirement ID is allocated to a component (or listed
  as not allocated with a reason), and no component lacks a requirement ID or
  an ASSUMPTION tag.
- Consistency with the system models: every external actor and system
  appears as an external interface; every structural concept is homed in a
  component; every sequence in the models can be walked through the
  architecture without a missing component or interface; every event in the
  state models is handled by some component.
- At least three scenarios were walked through, including one failure path.
- The evaluation plan gives each high-priority NFR a scenario and a
  measurable check.
- Each component states what it logs or emits and how it reports failure.
- Costly-to-change decisions are flagged.
- The documentation level matches [ARCH_MODEL_USE] and [CRITICALITY].
- No implementation code, detailed class design, or unmandated product names
  appear.
```
