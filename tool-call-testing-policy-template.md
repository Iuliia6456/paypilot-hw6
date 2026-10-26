# Tool Call Testing Policy - [System]

## 1. System and Scope

System / model / tools: [fill in]. Environment and evaluation date: [fill in]. Scenario set and conditions: [fill in]. Covered actions: [fill in]. Exclusions: [fill in].

## 2. Prediction and Results

Record predictions before running tests. If none were recorded, say so.

| Question | Prior prediction | Measured result | Evidence |
|---|---|---|---|
| Forbidden action frequency | [fill in] | [fill in] | [report] |
| Cost increase | [fill in] | [fill in] | [report] |

Distinguish attempted actions from confirmed state changes and keep separate batches separate.

## 3. Text, Actions and Dialogue Steps

An answer is not proof of an action. Earlier dialogue turns use fixed mock model replies while tools establish actual test state. One selected step uses the real model. Define unique mock matches and fixed call IDs; verify preparation and reset state between runs. Check calls, results, state and cost independently of answer wording.

Describe the chosen live step and required starting state: [fill in].

## 4. Three Checks: Tool, Arguments and Order

| Dimension | Scenario | Assertion | Ladder level | Gate / runs | Result |
|---|---|---|---|---|---|
| Tool choice | [ID] | Required and forbidden tools | [levels] | [fill in] | [fill in] |
| Arguments from history | [ID] | Correct entity; reject alternative | [levels] | [fill in] | [fill in] |
| Order | [ID] | Required check before action | [levels] | [fill in] | [fill in] |

Levels: exact equality, numeric threshold/tolerance, pattern, negative assertion.

## 5. State and Repeated Actions

### 5.1. State after a write

Require the target action, then inspect the saved entity, owner, values and row count. Expected state / scenario / evidence: [fill in].

### 5.2. Repeated action

Perform the action during preparation; repeat the request on the live step. Verify no duplicate state. State whether repeated tool calls and concurrent retries were also tested: [fill in].

## 6. Writes That Must Not Happen

Forbidden condition: [fill in]. Require the relevant validation and assert that no forbidden write call occurs. If claiming no saved write, inspect state too. Scenario / gate / runs / result / evidence: [fill in].

## 7. Tool Risk Classes

List every tool. Distinguish reads, calculations, handoffs and irreversible actions; do not assume cancellation is possible.

| Tool | Risk class | Possible harm | Reversible? |
|---|---|---|---|
| [tool] | [class] | [harm] | [evidence] |

Irreversible boundary: [fill in]. Access checks for read tools: [fill in].

## 8. Blocking Checks and Gates

| Irreversible action | Server check before execution | Measured or estimated latency | Legitimate request affected |
|---|---|---|---|
| [action] | [check] | [value or unmeasured] | [example] |

Separate proposed protections from implemented ones. Post-run tests do not themselves prevent actions.

Daily: one run for deterministic state, repetition and result-contract checks once the target action is reached. Release: five runs for model choices and variable behaviour. List scenario assignments, approval criteria and final suite evidence: [fill in].

## 9. Tool Specifications and Result Contracts

Required descriptions, arguments, defaults, ownership rules and side effects: [fill in]. Required output fields and types: [fill in]. Distinguish schema checks from correctness checks and implemented validation from test-only requirements. Scenario / result / evidence: [fill in].

## 10. Cost and Cache

Cache threshold, calls included and acceptable failure frequency: [fill in]. Baseline and defect costs with sample sizes: [fill in]. Prefix changes, tool-definition order and instruction changes to investigate: [fill in]. Evidence: [fill in].

## 11. Limits of Ready-Made Metrics

Identify the chosen metric and what it actually compares. Add independent checks for saved state, output contracts, value correctness and cache/cost. Explain how preparation makes the same live step reproducible. Remaining limitations: [fill in].

## 12. Residual Risk

| Uncovered area | Consequence | Why tolerate it now | Accept or defer; revisit trigger |
|---|---|---|---|
| [gap] | [harm] | [reason] | [decision and trigger] |

Make at least one explicit acceptance within the stated scope. Do not equate accepting a test gap with approving an unsafe action.
