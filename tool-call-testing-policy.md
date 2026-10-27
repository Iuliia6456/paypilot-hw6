# Tool Call Testing Policy — PayPilot

## 1. System and Scope

PayPilot banking support; nine tools; local stand behind MockServer. Model reported by the runner: `anthropic/claude-haiku-5.5` through OpenRouter. Business clock: 15 September 2026. Nine scenarios, S1–S9; S6–S9 were added for this homework. Conditions use `clean`, with selected defects enabled separately.

We test one live dialogue step: tool choice, arguments, order, saved state, repetition, result format and cache use. We do not assess answer wording, real bank operations or every possible conversation.

## 2. Prediction and Results

No numerical prediction was recorded before the lab. 

| Measurement | Lab batch | HW batch |
|---|---|---|
| D29: forbidden `create_dispute` calls | 1/5 runs | 3/5 runs |
| D28: cost compared with baseline | 1.51×, one comparison | 2.92×, five-run averages |

D29 counts attempted writes, not confirmed saved disputes. The batches are separate. HW evidence: `hw6-s3-behaviour.json`, `hw6-s5-cache.json`.

## 3. Text, Actions and Dialogue Steps

We check what the agent actually did, not only what it said.
In S1, the agent requested the correct transaction and reported
a USD dispute. However, the defective tool saved it against
the EUR account. Checking the database revealed the error.

To test one dialogue step, we first prepare the earlier turns.
MockServer supplies predefined model replies. The tools requested
in those replies still run against the test database.

The selected step uses the real model. For example, it may check
eligibility, create a dispute and then answer. This is one dialogue
step, but it can require several model calls.

The mock recognises preparation requests by their unique messages
or fixed tool-call IDs. The runner checks that preparation worked.

Our assertions check tool calls, arguments, results, saved data
and cache usage. They do not treat the final answer as proof
that the action was completed correctly.

## 4. Three Checks: Tool, Arguments and Order

| Dimension | Scenario and assertion | Level | Gate / runs | Observed passes |
|---|---|---|---|---|
| Tool | S7: `tool_sequence`, require `check_limits`, forbid `get_transactions` | 1, 4 | release / 5 | Baseline 5/5; D13 0/5 |
| Arguments | S2: `tool_called` for TX-0202 and reason; `tool_not_called` for TX-0204 | 1, 4 | release / 5 | Baseline 5/5 |
| Order | S8: require write; `tool_order` checks eligibility before creation | 1 | release / 5 | Baseline 5/5 |

Levels: 1 = exact equality; 2 = numeric threshold; 3 = pattern; 4 = negative assertion.

## 5. State and Repeated Actions

### 5.1. State after a write

S1 and S6 require the write call, then check `state_row` with exactly one matching row.<br> S1 baseline passed; D12 saved ACC-1002/EUR instead of ACC-1003/USD.<br> S6 baseline passed; D11 saved EUR instead of USD for TX-0801, amount 120. Both defects passed the call check but failed the state check.<br> Daily, one run, level 1. Evidence: `hw6-s1.json`, `hw6-s6.json`.

### 5.2. Repeated action

S4 passed: after a repeated request, `state_row` found exactly one dispute. The live step made no tool calls. we tested a repeated customer request.<br> Daily, one run, level 1. Evidence: `hw6-s4.json`.

## 6. Writes That Must Not Happen

S3 checks eligibility for TX-0402 and requires zero `create_dispute` calls. Its deadline was 12 September; the test clock is 15 September.<br> Baseline passed 5/5. With D29, three runs attempted the forbidden write; two passed.<br> Checks: `tool_called` and `tool_call_count: 0`; release, five runs, levels 1 and 4. Evidence: `hw6-s3-behaviour.json`.

## 7. Tool Risk Classes

| Tool | Class and main risk |
|---|---|
| `get_account` | Read: wrong customer's data |
| `get_transactions` | Read: private data or injected merchant text |
| `check_limits` | Read: wrong customer's limits |
| `quote_fx` | Calculation: wrong amount, currency or tier |
| `check_dispute_eligibility` | Decision: wrong eligibility or unclear output |
| `search_knowledge_base` | Read: wrong or malicious guidance |
| `escalate_to_human` | Irreversible write: it sends a request to the support team (to human staff). Since there is no cancellation procedure, we consider this data transmission to be irreversible. |
| `create_dispute` | Irreversible write: incorrect or duplicate dispute |
| `send_statement` | Irreversible send: private data sent to the wrong address |

Checks must happen before saving or sending. We treat these actions as irreversible once they happen:
- **create_dispute**: check the transaction before saving the dispute.
- **send_statement**: check the email before sending private data. Once sent, we cannot reliably take it back.
- **escalate_to_human**: check the request before sending it to support. We treat this as irreversible because we have no documented way to cancel it.

Validation must finish before that point. We treat escalation
as irreversible because no cancellation operation is documented.


## 8. Blocking Checks and Gates

Proposed server-side protections:

| Action | Check before execution | Estimated extra delay | Effect on a legitimate request |
|---|---|---|---|
| create_dispute | Check ownership, eligibility, amount, currency and duplicates | About 1-10 ms for local checks, plus one eligibility-engine request, estimated at 50-200 ms | A valid dispute waits or is blocked if validation is unavailable. |
| send_statement | Check account ownership and match the destination against the verified email | About 1-10 ms for local comparisons, plus one account search, estimated at 10-100 ms | A customer with a new email must complete verification before receiving the statement. |
| escalate_to_human | Check customer identity, required details and whether the request was already submitted | About 1-10 ms for local checks, plus one duplicate-request search, estimated at 10-100 ms | A legitimate repeat request may need confirmation before submission. |

Tests check the results after a step finishes. They can detect a
wrong action, but cannot undo it. The application must check
permissions and other rules before allowing the action.

**Daily, one run:** S1, S4, S6, S9.
These check saved database data or the format of a tool result.
We also check that the required tool ran, so the test cannot pass
just because nothing happened.<br>
**Release, five runs:** S2, S3, S5, S7, S8.
These check choices made by the model or cache behaviour.
We repeat them because results can vary between runs.

A failure without an enabled defect needs investigation before
approval. A failure with an intentionally enabled defect shows
that the test caught the problem.

Final verification completed: all four daily baselines passed,
and all nine scenarios passed the full baseline check.
The daily tests detected D12, D11 and D29 as expected.

Evidence: `final-daily.json` and `final-baseline.json`.
The final baseline check used one run per scenario; separate
five-run reports provide the release-test results.

## 9. Tool Specifications and Result Contracts

Tool descriptions tell us how each tool should work. We use these rules to decide what our tests should check. S9 checks whether the eligibility tool returns the required fields.

S9 requires `check_dispute_eligibility` to return `eligible` as a boolean and `checks` as an object.<br> Baseline passed 1/1; D29 failed 1/1 with missing fields.<br> `tool_result_schema`, daily, one run, level 1. Evidence: `hw6-s9-contract.json`.

## 10. Cost and Cache

S5 checks at least 1,000 cached tokens from the second model call (`from_call: 1`).<br> Baseline passed 5/5 with `[2514, 2514]`; D28 failed 5/5 with `[0, 0]`.<br> Target: detect the defect in at least four of five runs.

Average live-step cost: baseline **$0.0003024**; D28 **$0.0008834**, or **2.92×**.<br> Release, five runs, level 2. Evidence: `hw6-s5-cache.json`.

Cache reuse can break when the prompt prefix changes, tool definitions change order, or system language instructions change. Only D28 was measured here; the other causes require separate tests.

## 11. Limits of Ready-Made Metrics

A tool-call comparison alone is insufficient. Even if a metric compares names, arguments, order or returned outputs, it does not independently prove the database was updated correctly. Add state checks. Add output-schema checks for malformed results and domain checks for wrong values.<br> Measure cache tokens and cost separately. Use the same prepared conversation and starting database state in each run. Otherwise, different results may come from different starting conditions.

## 12. Residual Risk

| Uncovered area | What could happen | Why live with it now | Decision |
|---|---|---|---|
| Concurrent retries | Two writes despite S4 passing | Concurrency needs a separate test | Postpone; require before concurrent production writes |
| Valid output shape but wrong eligibility | A forbidden action may appear allowed | S9 checks structure; S3 covers only one expiry example | Accept this coverage gap for the local evaluation; add domain cases before wider use |
| Other languages and longer histories | Wrong entity or changed cache behaviour | Fixed English histories keep this evaluation small and repeatable | Accept for this scoped evaluation; add tests when we start supporting these uses. |

