# How the team runs

**Chain of command:** Jordan (Founder & Owner: final say on money, legal and anything signed; the face of sales) → Claude (CEO/COO/CFO: strategy, priorities, supervision) → agents.

## Active roster

| Agent | Model | Covers (from the founder's role list) | Done means |
|---|---|---|---|
| builder | Opus | AI Builder/Engineer + Claude Code Developer | Tests pass, safety-qa signs off, founder can follow the explanation |
| agent-designer | Opus | AI Agent Designer | Prompt, flow, settings template and 25+ test conversations |
| safety-qa | Opus | QA Engineer + safety/compliance gate | Tests run; written pass/fail with file-level findings |
| copywriter | Opus | Copywriter | No invented facts, under the length limits, ends with one clear ask |
| sales-coach | Opus | Sales Coach | Specific feedback with exact words to try next time |
| lead-researcher | Sonnet | Lead Researcher | Every fact has a source; no duplicates |

## Bench (hired at a milestone)

| Role | Hire when | Until then |
|---|---|---|
| Data/ROI Analyst | First live pilot | CEO tracks baseline and ROI |
| Customer Success | 3 live clients | CEO + weekly review |
| SOP/Documentation Manager | 5 clients, or before hiring any human | CEO keeps docs/ |

## Not hired
- **CEO/Strategist:** that's Claude.
- **Sales Rep/SDR:** the founder is the rep. Agents draft outreach, but a human sends it and walks in. Autonomous agent outreach risks texting-law violations and burns local trust.

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
