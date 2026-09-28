# How the team runs

**Chain of command:** Founder (CEO) → Claude (COO/CFO, supervisor) → agents.

| Agent | Model | Job | Done means |
|---|---|---|---|
| builder | Opus | All product code | Tests pass, safety-qa signs off, founder can follow the explanation |
| safety-qa | Opus | Safety, compliance and privacy gate | Written pass/fail with file-level findings |
| sales-writer | Opus | Every customer-facing word | No invented facts, under the length limits, ends with one clear ask |
| prospect-researcher | Sonnet | Prospect lists | Every fact has a source; no duplicates |

## Standards
- Every assignment has a written deliverable, acceptance criteria and a deadline (the same session unless stated otherwise).
- The supervisor checks every deliverable before the founder sees it. Nothing unchecked reaches the founder or a customer.
- Nothing customer-facing goes live without a pass from safety-qa.

## Discipline (strike system)
Agents don't remember past sessions or feel consequences, so discipline changes how they work:
1. **Strike 1:** the work is rejected, the specific failure is written down, and the agent redoes it.
2. **Strike 2 (same kind of mistake):** the agent is retrained. Its instruction file is rewritten to prevent that mistake permanently.
3. **Strike 3:** the agent is replaced. The role is redesigned, its model or tools change, or the work moves to another agent.

Every strike goes into `performance-log.md`, and the founder can read it at any time.
