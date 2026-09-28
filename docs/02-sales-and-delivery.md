# Sales & delivery playbook

Live team status: **Returnline HQ** at https://claude.ai/artifact/EYKPhtHm5DqMPqaGraGBMf (private to the owner)

## Part 1: How we sell (8 steps)

| # | Step | Who | What happens |
|---|---|---|---|
| 1 | Research | lead-researcher | List of local shops with address, hours and whether they claim 24/7 |
| 2 | After-hours test | Jordan (Mon 7-8pm) | Call each shop after hours and log who doesn't answer, e.g. "Mon 7:14pm, voicemail" |
| 3 | Walk-in | Jordan (Wed/Fri) | Ask the hook question, give the 45-second pitch, ask for the small yes: a free baseline |
| 4 | Debrief | Jordan → sales-coach | Right after each visit, 3 lines: what happened, what they said, next step |
| 5 | Baseline (days 1-5) | Owner + Jordan | Count missed calls from their phone provider's call log. This is the "before" number |
| 6 | Free 14-day pilot | builder / safety-qa | System goes live (see Part 2) |
| 7 | Day-14 results meeting | Jordan, CEO prepares the report | Missed calls caught, conversations, jobs booked, estimated revenue. Then ask: "Want to keep it running?" |
| 8 | Close | Jordan | Founding rate $397/mo (first 3 clients) or $497/mo + $497 setup, month to month. Simple signed agreement (lawyer-reviewed), card on file |

If they don't sign: follow up on days 2, 5 and 10 (`sales/follow-up-texts-emails.md`), then move them to a 60-day check-back.

## Part 2: How we deliver

**Day 0, intake (15-minute meeting or form).** We collect:
- business name and hours
- service area and services
- the on-call or emergency contact
- how jobs get booked: their software, Google Calendar, or "text the office"
- whether prices may be quoted
- anything the AI must never say

**Day 0, start texting registration.** US carriers require business-texting registration (A2P 10DLC) before we can send at scale. Approval can take days to weeks, so we file it the day they say yes. It is the most likely delay.

**Day 1, build (builder).**
1. Create the client's account and a local phone number.
2. The shop turns on **conditional call forwarding**: calls they don't answer (busy or no answer after about 20 seconds) forward to our number. Their main number and phones don't change.
3. Load the client's settings into the texting AI designed by agent-designer.
4. Set up owner alerts: a text for every new lead and an immediate call or text for emergencies.

**Day 2, test (safety-qa).** Run the test conversations: gas smell, angry customer, "are you a robot?", STOP, Spanish. Then Jordan and the owner place real test calls. **Nothing goes live without safety-qa's written pass.**

**Days 3-14, pilot live.**
- Week 1: we check conversations daily.
- Each Friday: a short results text to the owner.
- Day 14: the results report for the close meeting.

**Ongoing, paying client.**
- Monthly ROI report: leads recovered, jobs booked, estimated revenue vs. what they paid us. This is what keeps clients.
- Monthly tune-up of the AI based on real conversations.
- A quarterly check-in in person, which is easy locally.

**Offboarding.** The shop turns off call forwarding. We export their lead data to them and then delete it.

## Why the baseline matters
Without a "before" number there's no proof, and without proof small businesses cancel. Never skip step 5.
