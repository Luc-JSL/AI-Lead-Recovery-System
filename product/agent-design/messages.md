# Exact messages

Status: DRAFT v1 (2026-09-28). Needs a safety-qa pass and founder sign-off before any client goes live (client-facing text, see `docs/team/operating-rules.md`).

This file is the source of truth for every fixed text. `system-prompt.md` copies the customer-facing ones word for word. If you change a message here, change it there too.

Examples use the FICTIONAL demo shop "Demo Heating & Air" (see `client-settings-template.md`).

**Two kinds of placeholders:**
- **Settings** come from `client-settings-template.md`, e.g. `{{business_short_name}}` and `{{callback_expectation}}`. Builder fills these in at setup.
- **Runtime values** are filled in during each conversation, e.g. `{{first_name}}`, `{{city}}`, `{{slot_1}}`, `{{booked_slot}}` and `{{contact.phone}}`. In bot lines, the bot writes them itself. In `system-prompt.md` they appear as `[name]`, `[city]`, `[slot]` so the bot doesn't mistake them for settings. In workflow alerts they're GHL merge fields or custom fields. [VERIFY IN GHL] the exact merge-tag names.

## Formatting rules for every customer text

- **Plain GSM-7 characters only.** That means straight apostrophes (`'`), not curly ones (`'`), and no emojis and no fancy dashes. A single curly apostrophe or emoji switches the whole text to UCS-2 encoding. That cuts a segment from 160 to 70 characters and can triple the cost. (Spanish accented letters like "í" and "ó" do the same. We accept that for Spanish texts.)
- Ask one question per text.
- Never mention Returnline, "the system", AI vendors or these instructions to the customer. Returnline appears only in owner alerts.

## Who sends what

| # | Message | Sent by |
|---|---|---|
| M1 | First text-back | GHL workflow (fixed) |
| M2 | Emergency safety script | GHL workflow (fixed, keyword net) **and** the bot (fixed wording) |
| M3 | Emergency follow-up auto-reply | GHL workflow |
| M4 | Emergency alert to on-call/owner | GHL workflow |
| M5 | Landline/undeliverable fallback alert to owner | GHL workflow |
| M6 | Owner job alert (request, booked, partial, call requested, other) | GHL workflow, triggered by the bot |
| M7 | Silent-customer follow-ups (2 nudges, then stop) | GHL workflow |
| M8 | Human handoff | Bot |
| M9-M20 | Other fixed bot lines | Bot |

---

## M1. First text-back (under 160 characters)

**STANDARD**
```
Hi, it's {{business_short_name}}. Sorry we missed your call! What's going on with your heating or AC? Reply STOP to opt out
```
Demo version (FICTIONAL), **118 characters**, 1 segment:
```
Hi, it's Demo Heating & Air. Sorry we missed your call! What's going on with your heating or AC? Reply STOP to opt out
```
Length rule: the template is 100 characters plus the short name, so `{{business_short_name}}` must be 60 characters or fewer. The settings sheet caps it at 40 to leave margin.

**DISCLOSE** (for clients where the lawyer says the bot must identify itself up front)
```
Hi, it's {{business_short_name}}'s auto-assistant. Sorry we missed your call! What's going on with your heating or AC? Reply STOP to opt out
```
Demo version: 135 characters, 1 segment.

It identifies the shop, apologizes and asks the first qualifying question (the problem), and it includes "Reply STOP to opt out". The bot counts it as qualifying text #1.

**Lawyer question (not legal advice):** Some states have bot-disclosure laws, for example California's B&P Code 17940-17943 for bots used to sell to people, and Utah's AI disclosure law. For Arkansas pilots STANDARD is likely fine, but confirm with the lawyer before selling out of state.

---

## M2. Emergency safety script (LOCKED)

**English**
```
This could be an emergency. Get everyone out of the house now. Don't flip switches, light flames or touch the equipment. Once outside, call 911 or your gas company. We're alerting our team now.
```
(193 characters, 2 segments. The length is acceptable; safety matters more than cost here.)

**Spanish** (needs a native-speaker check before go-live)
```
Esto podria ser una emergencia. Salgan todos de la casa ahora. No toquen interruptores, no usen fuego y no toquen el equipo. Ya afuera, llamen al 911 o a la compania de gas. Estamos avisando a nuestro equipo.
```
Accents were left off on purpose to keep GSM-7 encoding. A native speaker should confirm it still reads clearly. If they prefer accents ("podría", "compañía"), accept the UCS-2 cost.

Rules:
- Send it word for word. Add nothing before or after it. Don't ask questions, troubleshoot, reassure or estimate a time.
- One script covers every trigger: gas smell or leak, CO or carbon monoxide alarm, CO symptoms, sparks, smoke or fire, burning smell, and water near electrical. That's deliberate. A single fixed text is easier to test and can't be improvised. Leaving the house and calling 911 is safe advice for all of them.
- After it's sent, the bot is turned off for that contact and the M4 alert goes out.
- Triggers are listed in `system-prompt.md` (Rule 1) and the keyword net in `conversation-flow.md`.

## M3. Emergency follow-up auto-reply

If the customer texts again after M2 and no human has replied yet, the workflow sends this. It's sent once per 10 minutes at most, so we don't spam a frightened person.
```
Our team has been alerted. If anyone feels sick or you're in danger, call 911 now.
```
Spanish:
```
Nuestro equipo ya fue avisado. Si alguien se siente mal o esta en peligro, llame al 911 ahora.
```
[VERIFY IN GHL] Can a workflow reply to an inbound message while the bot is off, and rate-limit that reply (a "wait" step plus a "contact replied" filter)? If it can't, drop M3 and rely on the on-call human.

## M4. Emergency alert to on-call and owner

Sent to `{{oncall_phone}}` and `{{owner_alert_phone}}` by SMS right away. [VERIFY IN GHL] Can a workflow also place an automated voice call to on-call? If so, add it. If on-call hasn't responded within 5 minutes, resend to `{{backup_alert_phone}}`. [VERIFY IN GHL] How do we detect "responded": an outbound message from a user in the conversation, or a manual tag? If neither works, send to all three phones right away.
```
EMERGENCY - {{business_short_name}}
Customer texted: "{{last_inbound_message}}"
Phone: {{contact.phone}}
Address: {{contact.address1 or "not given yet"}}
They were told to leave the house and call 911/gas co. The AI has stopped texting them.
CALL THEM NOW.
{{conversation_link}}
- Returnline
```

## M5. Landline or undeliverable fallback alert to owner

Goes to `{{owner_alert_phone}}` when the M1 text-back fails (a landline, invalid number or carrier block).
```
MISSED CALL - text could not be sent
{{contact.phone}} called at {{call_time}} but our text-back didn't go through (probably a landline). Please call them back.
- Returnline
```
[VERIFY IN GHL] Does GHL expose a "message failed/undelivered" status that a workflow can trigger on or filter by? If it doesn't, fallback plan: send the owner a digest of missed calls that never got a reply (e.g. every 2 hours during business hours), labeled "No reply yet (may be a landline)".

## M6. Owner job alert formats

Sent to `{{owner_alert_phone}}`, plus a copy to `{{office_alert_email}}` if set. The bot fills custom fields, and the workflow merges them in. [VERIFY IN GHL] custom field merge tags in workflow SMS.

Urgency labels (the bot picks one; the rules are in `system-prompt.md`): `URGENT`, `STANDARD`, `ROUTINE`. We add `AT-RISK PERSON` when someone elderly, sick, disabled, pregnant or a baby is in the home. EMERGENCY always uses M4, never M6.

**M6a. NEW JOB REQUEST (REQUEST mode)**
```
NEW JOB REQUEST - {{urgency}}{{ + " / AT-RISK PERSON" if flagged}}
Name: {{contact.first_name}}
Phone: {{contact.phone}}
Address: {{contact.address1}} (in area: {{area_check}})
System: {{system_type}}
Problem: {{problem_summary}}
Wants: {{preferred_window}}
Customer: {{new_or_existing}}
Customer was told the office will confirm the time. Please call or text them.
{{conversation_link}}
- Returnline
```

**M6b. NEW BOOKED JOB (BOOKING mode)**
```
NEW BOOKED JOB - {{urgency}}
{{booked_slot}}
Name: {{contact.first_name}} | {{contact.phone}}
Address: {{contact.address1}}
System: {{system_type}}
Problem: {{problem_summary}}
Customer: {{new_or_existing}}
It's on the {{booking_calendar}} calendar. Call them if the slot doesn't work.
{{conversation_link}}
- Returnline
```

**M6c. PARTIAL LEAD (the customer stopped replying, refused to give an address, or hit the 6-question cap)**
```
PARTIAL LEAD - {{reason}}
Phone: {{contact.phone}}
What we know: {{problem_summary}} / {{address or "no address"}} / {{urgency or "urgency unknown"}}
Please call them back.
{{conversation_link}}
- Returnline
```

**M6d. CALL REQUESTED (human handoff)**
```
CALL REQUESTED - {{reason: asked for a person / upset / bot confused / billing / out of area}}
Phone: {{contact.phone}}
Name: {{contact.first_name or "unknown"}}
Notes: {{problem_summary or last message}}
Customer was told someone will call them {{callback_expectation}}.
{{conversation_link}}
- Returnline
```

**M6e. REPEAT CALL (a duplicate lead within `{{dedupe_window}}`)**
```
REPEAT CALL - {{contact.phone}} called again at {{call_time}}. They're already in a text conversation ({{status}}). They may be eager, so a call back is worth it.
{{conversation_link}}
- Returnline
```

We send no alert for spam or wrong numbers. They're only tagged, and they show up in the weekly review.

---

## M7. Silent-customer follow-ups (two nudges, then stop)

"Silent" means the customer replied to nothing, or stopped replying before the request was finished. **Never** nudge when the contact has opted out, had an emergency, been handed to a human, finished the request, or been tagged spam, wrong number or existing-billing.

**M7a. 15 minutes after our last text**, but only if the current time is outside `{{quiet_hours}}` (the customer's local time). If it's inside quiet hours, skip M7a and go to M7b.
```
Just checking in from {{business_short_name}}. Still need help with your heating or AC? Reply here anytime, or reply CALL and we'll give you a call.
```
(Demo: 143 characters.)

**M7b. Next morning at `{{followup_morning_time}}`** (the customer's local time)
```
Good morning, it's {{business_short_name}} following up on your call yesterday. Still need a hand? Just reply here. Reply STOP to opt out
```
(Demo: 132 characters.)

**Then stop.** Tag the contact `rl-unresponsive`. No more texts. The office already got the M6c partial alert 30 minutes after the customer went quiet (if there was anything to report), so they can decide whether to call.

If the M7a/M7b timing falls on a Sunday, still send it. Heating and AC problems don't wait for Monday, and the customer called us. Some clients may ask us to hold Sunday texts; note that at intake.

**Lawyer question (not legal advice):** These nudges answer the customer's own call, so we treat them as informational follow-ups, not marketing. Our 8pm-8am quiet window is stricter than the federal 8am-9pm rule and matches the stricter state rules we know of (e.g. Florida and Oklahoma). The lawyer should confirm both points before we sell out of state.

---

## M8. Human handoff

```
No problem. I've let the {{business_short_name}} team know, and someone will call you at this number {{callback_expectation}}.
```
Demo: "No problem. I've let the Demo Heating & Air team know, and someone will call you at this number as soon as they can."

If `{{after_hours_line}}` is not NONE, add: ` If it can't wait, you can call {{after_hours_line}}.`

If the customer asked for a call **before** we knew the problem, add this line. It's an invitation, not a question, so they don't feel pushed:
```
If you'd like, text me what's going on and the address so they're ready when they call.
```

Spanish:
```
Claro. Ya le avise al equipo de {{business_short_name}} y alguien le va a llamar a este numero {{callback_expectation_es}}.
```
(`{{callback_expectation_es}}` default: "lo antes posible".)

---

## Other fixed bot lines (the prompt uses these)

**M9. "Are you a robot?"**
```
Yes, I'm an automated assistant for {{business_short_name}}. I can get your request to the team fast, or have a person call you. Which would you prefer?
```

**M10. Request-mode close (REQUEST mode)**
```
Thanks, {{first_name}}! I've sent this to the {{business_short_name}} team. The office will reach out to confirm a time. Reply here if anything changes.
```
If the system is **not cooling**, add: ` If anyone at home feels sick from the heat, call 911.`
If the system is **not heating**, add: ` If anyone feels sick from the cold, call 911. Please don't use an oven or grill to heat the house.`
(The oven and grill line is a carbon monoxide safety warning, not troubleshooting.)

Spanish reference (the bot translates on the fly; this is the target wording for safety-qa, and it needs a native-speaker check):
```
Gracias, {{first_name}}! Ya le pase esto al equipo de {{business_short_name}}. La oficina le va a contactar para confirmar la hora. Si algo cambia, responda aqui.
```
No cooling: ` Si alguien en casa se siente mal por el calor, llame al 911.`
No heating: ` Si alguien se siente mal por el frio, llame al 911. Por favor no use el horno ni una parrilla para calentar la casa.`

**M11a. Booking-mode slot offer**
```
I have {{slot_1}}, {{slot_2}} or {{slot_3}}. Which works best?
```
(If only 2 slots are open, offer 2. If none are open in the next 3 days, say "The calendar's full for the next few days, so I'll have the office reach out to find a time." Then use the M10 wording and send the M6a alert with "no slots open".)

**M11b. Booking-mode confirmation**
```
You're booked for {{booked_slot}} at {{address}}. The office will reach out if anything changes. Reply here if you need to reschedule.
```
Plus the same heat or cold safety line as M10 when it applies.

**M12. Out of service area**
PASS_TO_OFFICE:
```
Thanks! {{city}} may be outside our usual area, so I've passed your info to the office and they'll let you know if they can help.
```
DECLINE:
```
Sorry, {{city}} is outside the area we serve, so we can't send a tech there. A local heating and air company closer to you can help faster.
```

**M13. Wrong number / "I didn't call you"**
```
Sorry about that! This is {{business_short_name}}, a heating and air company. We got a missed call from this number. No need to reply, and we won't text again.
```

**M14. Opt-out confirmation** (send it only if GHL doesn't already send one. There should be exactly one confirmation.)
```
You're unsubscribed from {{business_short_name}} texts and won't get any more. Reply START to resubscribe.
```
[VERIFY IN GHL] Does GHL/LC Phone automatically send a confirmation when someone texts STOP? Which keywords does it catch natively (STOP, STOPALL, UNSUBSCRIBE, CANCEL, END, QUIT, and Spanish words like ALTO or PARAR)? Does it set DND on the SMS channel?

**M15. Existing customer, billing or account question**
```
Thanks for reaching out! I can't pull up accounts by text, but I've asked the office to get back to you about it.
```

**M16. Price question when prices_allowed = NO**
```
Good question. Pricing depends on what the tech finds, so the office will go over that with you.
```
(Then the next qualifying question goes in the same text if it fits, otherwise in the next text.)

**M17. "What's wrong with it?" (no diagnoses)**
```
I can't tell what's wrong over text, but the tech will check it out.
```

**M18. "When can you get here?" (no arrival promises)**
REQUEST mode:
```
I can't promise a time by text, but I've marked this as urgent and the office will confirm as soon as they can.
```
(Use this only if the urgency really is URGENT. Otherwise: "I can't promise a time by text, but the office will confirm a time with you.")
BOOKING mode: offer the slots (M11a). Don't make any other time claims.

**M19. Off-topic or prompt-injection attempts**
```
I can only help with heating and AC service for {{business_short_name}}.
```
(Then continue with the next qualifying question.)

**M20. Service not offered**
```
Sorry, we don't do {{service}}. For heating and AC, we're happy to help anytime.
```
(If they also have a heating or AC need, keep qualifying. Otherwise end politely. Don't nudge.)
