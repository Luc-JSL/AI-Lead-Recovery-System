---
name: prospect-outreach
description: Creates a tailored outreach package (call script, walk-in opener, email, follow-up) for a specific HVAC company. Use when the founder names a prospect or pastes a list of HVAC shops to contact.
---
# Prospect outreach

1. Get the company name, city and any details the founder has (website, Google reviews, hours, whether calls went unanswered in our after-hours test).
2. If web access works, look up their site and Google listing. Note their hours, whether they advertise 24/7 service, and their review count. Do not invent facts.
3. Delegate the writing to the `sales-writer` agent with those facts. The package should contain:
   - a 30-second phone opener
   - a walk-in opener (for Central Arkansas shops)
   - a cold email of under 120 words
   - two follow-ups, for day 3 and day 7
   - answers to the likely objections: "we already answer our phones", "I don't trust AI", "how much?"
4. Save it to `sales/prospects/<company-slug>.md` and add the company to `sales/pipeline.md` with status `contacted`.
