# Corpus Scope, Provenance, and Non-Circularity Policy

## Corpus purpose

This indexed corpus is a domain handbook for mathematical formulation in UAV-assisted logistics and mobile edge computing. It supports two activities:

- a Responder selects and adapts reusable modeling primitives to a user-defined system;
- a Verifier checks logic, notation, units, assumptions, and missing constraint families.

Retrieved material is guidance. It is not an answer key, a complete problem instance, or a substitute for reasoning from the user description.

## Indexed-user scope

Only generic knowledge files in the indexed directory may be retrieved during generation and internal verification. The Responder and Verifier must not access held-out formulations, expert scoring rules, expected answers, or evaluation labels.

The Responder must decide which primitives apply, define one consistent notation system, and assemble the model independently. The Verifier must judge the proposed model against the user request and generic modeling principles rather than against a hidden reference answer.

## Included knowledge categories

The corpus may contain broad, reusable knowledge on:

- system-boundary selection for logistics, computing, communication, and energy;
- sets, indices, variables, parameters, domains, and physical units;
- multi-criteria objective design and normalization;
- assignment, routing, depot connectivity, capacity, distance, and timing;
- alternative subtour-elimination mechanisms;
- processor-mode selection and communication-path activation;
- per-slot capacity and multi-slot service accumulation;
- completion, flow time, deadlines, and late or unfinished tasks;
- route-induced service availability and resource gating;
- propulsion, dwell, computing, reception, and relay energy;
- boundary cases and model variants.

## Excluded information

The indexed corpus must not contain:

- a complete held-out reference formulation;
- reference equation numbers, labels, or ordering;
- a reconstructed reference notation table;
- a prompt-specific objective or complete constraint sequence;
- held-out expected answers;
- expert rubrics, scoring labels, or pass/fail annotations;
- sentence-level rewrites of a held-out answer;
- a held-out formulation split across several files;
- a held-out formulation copied with symbols renamed or terms reordered.

## Provenance and corpus-freeze rule

The corpus should be derived from independently selected domain sources, textbooks, standards, and broadly applicable modeling practice. A source manifest may be stored outside the index for audit purposes.

After an evaluation prompt or reference answer is created, the indexed corpus must be frozen. A failure on that evaluation must not be repaired by adding prompt-specific equations or phrases to the corpus and then rerunning the same test. Necessary corpus revisions require a new corpus version and a new held-out evaluation set created after the revision.

This freeze rule is essential. Excluding an exact answer is not sufficient if the corpus is later tailored to reproduce that answer at the schema or equation-family level.

Category overlap with a disclosed task specification is not, by itself, answer leakage. A domain corpus is expected to cover the modeling categories requested by its intended users. The prohibited case is indexing a held-out answer, reconstructing its prompt-specific equation sequence, or modifying the corpus after observing held-out results and then presenting the same case as independent generalization evidence. If an existing formulation informed corpus development, that formulation should be treated as a development or regression case rather than the sole held-out test.

## Category-complete, not answer-complete

The corpus follows this principle:

> Cover the reusable concepts of the problem class without storing a ready-to-assemble answer for one evaluation case.

Category coverage means that an agent can learn how assignment, routing, scheduling, communication, completion, coupling, and energy relations are commonly modeled. It does not mean that one chunk or one ordered file reproduces every component of a particular formulation.

## Generic formulas and alternatives

A formula in this corpus is a candidate primitive. It should be accompanied by assumptions, units, applicability conditions, alternatives, and failure cases. Common formulas may resemble equations used in many optimization models; similarity caused by a standard modeling pattern is acceptable only when the corpus is not tailored to a held-out answer and does not reproduce its complete structure.

## Responder obligations

The Responder should:

1. identify the entities, decisions, resources, and time representation in the user request;
2. retrieve by subtopic when a broad step contains several constraint families;
3. choose only applicable primitives;
4. remove unavailable modes and resources;
5. define every symbol and unit;
6. check dimensional and logical consistency;
7. avoid presenting retrieved text as the final formulation.

## Verifier obligations

The Verifier should:

1. check the user-requested component list;
2. verify mode, path, capacity, timing, completion, deadline, service-window, and energy logic;
3. flag unsupported components and undefined symbols;
4. distinguish an alternative valid formulation from an error;
5. avoid accepting an answer only because it resembles retrieved material.

## Evaluation separation

Held-out formulations and expert rubrics are maintained outside the indexed directory. They are used only after generation for quantitative evaluation, leakage analysis, and human assessment. They are never embedded, retrieved, or displayed to the Responder or Verifier.
