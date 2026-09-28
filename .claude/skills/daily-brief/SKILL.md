---
name: daily-brief
description: Start-of-session and end-of-session operating brief for Returnline. At the start, it reports current state, the biggest bottleneck, the highest-value task and today's revenue objective. At the end, it records what was completed, learned and failed, and tomorrow's top action. Use when the founder starts or ends a work session, says "what should I do", "brief me", "let's start" or "wrapping up".
---
# Daily brief

## Start of session
1. Read `CLAUDE.md`, `docs/01-business-blueprint.md` (the decisions log tail), `docs/metrics.md`, `sales/pipeline.md` and the latest entry in `docs/daily-log.md` if it exists.
2. Reply in this exact format, kept short:
   - **Current state:** paying clients, MRR, the pipeline stage counts, and where we are in Phases 1-10.
   - **Biggest bottleneck:** name ONE stage of the funnel (researched → contacted → replies → positive → calls booked → calls done → proposals → closed) and the evidence for it.
   - **Highest-value task today:** one task.
   - **Jordan does:** the tasks only Jordan can do (selling, calls, signing, paying).
   - **Claude does:** what Claude will produce this session.
   - **Claude Code / agents do:** delegated build or research tasks.
   - **Can be automated:** anything repetitive we noticed.
   - **Revenue objective for today:** a concrete number, like "3 walk-ins, 1 trial yes".
3. **Anti-procrastination check:** if no sales activity was logged in the last 48 hours and today's plan is building, say so and put a sales action first.

## End of session
1. Ask Jordan for the outcomes if they aren't already in the chat.
2. Append an entry to `docs/daily-log.md`: date, completed, learned, failed, what should change, and tomorrow's single highest-priority action.
3. Update `sales/pipeline.md` and the HQ dashboard stages for any shop that moved. Commit.
