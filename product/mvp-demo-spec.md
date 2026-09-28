# Returnline MVP Sales Demo: GoHighLevel Build Spec

Status: v1, 2026-09-28. Owner: builder. Needs a safety-qa pass before any prospect sees it.
Audience: Jordan (founder). No custom code, no n8n. Everything is built by clicking in GoHighLevel (GHL).

**How to read the tags**
- **[VERIFY IN GHL]**: I'm not certain of this menu name, setting or feature. GHL renames things often. Check it on screen before relying on it. If it's different, note the real name in this file.
- **[VERIFY PRICE]**: a price reported by third-party sites in September 2026. I couldn't reach GHL's official pricing and help pages from my environment, so none of these prices are confirmed. Check each one on GHL's billing pages before you pay.
- **[FROM AGENT-DESIGN]** / **M1-M20**: customer-facing wording and AI behavior. agent-designer owns it in `product/agent-design/`. Copy it from there and don't rewrite it here. Those files are still in progress, so if one isn't ready yet, the demo waits for it.

**What comes from `product/agent-design/`** (don't duplicate it here):

| Needed for | agent-designer file |
|---|---|
| The AI bot's instructions (bot "prompt") | `product/agent-design/system-prompt.md` [in progress] |
| The order of questions (max 6 texts) and the **emergency keyword net** | `product/agent-design/conversation-flow.md` [in progress]. If its keyword list differs from the starter list in section 2(g), **its list wins**. |
| Every fixed text, by ID: M1 text-back, M2 emergency script, M3 emergency follow-up, M4 emergency alert, M5 landline fallback alert, M6a-e owner alerts, M7a/b nudges, M8 handoff, M11a/b slot offer + booking confirmation, M13 wrong number, M14 opt-out confirmation | `product/agent-design/messages.md` (the source of truth for wording) |
| Demo shop details (name, hours, services, "no prices", BOOKING mode, alert phones, quiet hours, 24 h repeat-call window) | `product/agent-design/client-settings-template.md`, "Filled example: fictional demo shop" |
| What to test | agent-designer's 25+ test conversations [in progress] (safety-qa runs them) |

This spec refers to messages by their ID (e.g., "send **M1**"). Copy the text from `messages.md` into GHL, and don't retype it from memory.

---

## 1. Requirement

Build a working, live demo of the Returnline system inside one GHL sub-account for the **fictional** shop "Demo Heating & Air" (agent-designer's demo settings). Don't use a real HVAC company's name, not even the prospect's: the texts would be sent under a brand that didn't agree to it, and our A2P registration covers Returnline only. The demo runs like this. Jordan asks the prospect to call the demo number **from the prospect's own phone**. Nobody answers. Within seconds the prospect gets a text from "Demo Heating & Air", replies, and has a short text conversation with the AI. The AI collects the problem, urgency, address and a time window, then books a slot on a demo calendar. The lead then shows in the pipeline as **Booked**, the appointment is on the calendar, and Jordan's phone (playing the shop owner) gets an alert. Jordan can also show safety: texting "I smell gas" gets a fixed safety reply plus an owner alert, and STOP ends all texts.

> Why the prospect's own phone: "hands a prospect their phone" can be read two ways. If the prospect called from Jordan's phone, the caller and the "owner" would be the same number. The customer texts and owner alerts would then land in one thread, and the contact record would be wrong. So the prospect uses their own phone and Jordan's phone plays the owner. If a prospect won't use their own phone, carry a cheap second phone as the "customer" phone. Jordan's phone should never be both.

**Demo success criteria** (all must be true on the day-before run, section 6):

| # | Criterion | Target |
|---|---|---|
| S1 | The missed call produces a text on the caller's phone | ≤ 60 s every time (the promise we sell); aim for ≤ 15 s |
| S2 | The AI answers each customer text | ≤ 30 s per reply [VERIFY IN GHL: bot reply delay setting] |
| S3 | The AI gets to a booking | ≤ 6 texts from us (M1 counts as #1), one question at a time |
| S4 | The appointment is on the demo calendar and the opportunity is in the **Booked** stage with issue, urgency, address and window filled | 100% |
| S5 | Jordan's phone gets the "new booked job" alert | ≤ 60 s after booking |
| S6 | The customer gets the booking confirmation (M11b), and a reminder is scheduled | Confirmation ≤ 60 s, exactly one. The reminder shows as scheduled in the workflow history |
| S7 | "I smell gas" gets the fixed safety script M2 plus the M4 alert | 100%, no other content: no troubleshooting, no questions |
| S8 | STOP gets exactly one opt-out confirmation (GHL's own or M14, never both), then silence | 0 texts after that |
| S9 | Asking for a person gets a handoff message, the bot goes quiet and the owner is alerted | 100% |
| S10 | The AI never quotes a price, diagnoses equipment or promises an arrival time, and admits it's automated if asked | 0 violations |
| S11 | The whole demo, from dialing to the owner alert, runs in about 3-5 minutes | ≤ 5 min |

---

## 2. Architecture

### How the pieces fit

```
Prospect's phone                      GHL sub-account "Demo Heating & Air"                       Jordan's phone ("owner")
     |                                                                                                   ^
     | 1. calls demo number  -> LC Phone number (501 area code): nobody answers -> call status = missed  |
     |                                        |                                                          |
     | <- 2. text-back ----------  WF-A  Missed call -> text-back + contact + opportunity "New lead"     |
     |                                        |                                                          |
     | 3. replies <-> AI ---------  Conversation AI bot (Auto-Pilot, SMS)      <- prompt from agent-design
     |                                        |  (saves fields, adds tags, books on demo calendar)       |
     | <- M11b confirmation (bot)  WF-C  tag ai-qualified -> check fields -> stage "Qualified" (alert: production only)
     |                             WF-D  appointment booked -> stage "Booked" -> reminder ---------------> M6b alert
     |                             WF-E  no reply -> 2 nudges, then stop
     |                             WF-F  STOP words -> DND on, bot off, remove from workflows
     |                             WF-G  emergency words -> M2 safety script, bot off -------------------> M4 alert
     |                             WF-H1/H2  "human" words or bot handover -> bot off, M8 --------------> M6d alert
```

"WF" means a GHL workflow (Automation → Workflows [VERIFY IN GHL]). The only AI step is the Conversation AI bot. **Every safety rule (emergency, STOP, human handoff) has a plain keyword workflow behind it, so it never depends only on an AI decision.** That's how we meet the blueprint's rule of "in code, not only in prompts" without writing code: the bot's prompt tells it to behave, and the workflow makes sure it does.

**Known limit (be honest about it):** GHL doesn't let us guarantee that the emergency workflow runs *before* the bot replies to the same message. agent-designer's design has both the workflow and the bot send the same fixed M2 script (belt and braces), so the customer may get M2 twice. That's acceptable. What's **not** acceptable is the bot sending anything *other than* M2. Protections: (1) `system-prompt.md` Rule 1 tells the bot to send only M2, and (2) WF-G turns the bot off for that contact right away. safety-qa must test this race (T-G2 in section 6). A true "check before the AI speaks" guarantee is one of the reasons for custom code later (section 7).

### Automation specs

#### (a) Missed call → text-back (WF-A)

| Field | Spec |
|---|---|
| **Trigger** | Workflow trigger **Call Status**, filtered to missed / no-answer / voicemail / busy [VERIFY IN GHL: the exact status names in the filter; test which one a real unanswered call produces]. Direction: inbound only. Never fire on "completed" calls, or we'd text people we just talked to. |
| **Inputs** | Caller's phone number (caller ID), call time, the demo number called. |
| **Logic** | 1) If the contact has the `opted-out` tag or SMS DND is on → stop. 2) If WF-A already texted this number within the **24 h repeat-call window** (`{{dedupe_window}}`), or the contact is tagged `emergency` / `needs-human` → no new text-back; send the owner **M6e** (repeat call) → stop [VERIFY IN GHL: If/Else on "tag added within last X hours", or a `last_textback_at` date field compared to now]. 3) Otherwise: add tag `missed-call`, set custom field `lead_source` = Missed call and `missed_call_at` = now, create or update the opportunity in pipeline "Service Leads", stage **New lead**, 4) send **M1**, 5) turn the bot on for this contact [VERIFY IN GHL: "Conversation AI" workflow action / bot status per contact], 6) add the contact to WF-E (nudges). |
| **AI reasoning** | None, on purpose. M1 is fixed wording, so it's instant, predictable and compliant. It names the shop, asks the first question and says "Reply STOP to opt out". |
| **Actions** | Add tag, update fields, create/update opportunity, send M1, set bot status, add to workflow. |
| **Error handling** | The caller ID is blocked or it's a landline → M1 fails. Send the owner **M5** (landline fallback) if GHL lets a workflow see a failed status; otherwise use agent-designer's fallback digest (see M5) [VERIFY IN GHL]. The GHL wallet is empty → texts stop, so turn on wallet auto-recharge (section 3.3). A2P isn't approved → texts get blocked or filtered (section 5). |
| **Escalation** | M5 / M6e to the owner so a human calls back. |
| **Logging** | Call record + recording/voicemail in Conversations; workflow execution history; tag `missed-call`; field `missed_call_at`; opportunity created. These records count "missed calls caught". |
| **Testing** | T-A1..T-A4 in section 6. |
| **Success criteria** | M1 arrives ≤ 60 s after the caller hangs up (or after voicemail starts), exactly once, with one opportunity in New lead. |

#### (b) Inbound SMS → AI conversation (Conversation AI bot)

| Field | Spec |
|---|---|
| **Trigger** | The customer's reply on SMS. The bot is on in Auto-Pilot for the SMS channel [VERIFY IN GHL: in current bot versions, whether the bot answers every inbound SMS on its own or only after a workflow turns it on]. |
| **Inputs** | The message text, conversation history, contact fields, the filled demo settings from `client-settings-template.md` (hours, service area, services, "no prices"), and the demo calendar's open slots. |
| **Logic** | Bot doesn't run if the contact has tag `opted-out`, `emergency` or `needs-human` [VERIFY IN GHL: bot "exclude contacts with tag" setting, or turn the bot off per contact from WF-F/G/H]. At most ~6 AI texts per conversation, then hand off [VERIFY IN GHL: max-messages setting]. |
| **AI reasoning** | All from `system-prompt.md`: one question at a time, get problem → urgency → address → time window, never quote prices (M16), diagnose (M17) or promise arrival times (M18), admit it's automated if asked (M9), offer a human (M8). |
| **Actions** | Bot actions [VERIFY IN GHL: names in the bot's "Actions"/"Goals" list]: **Update contact field** (problem_summary, urgency, system_type, preferred_window, new_or_existing, standard address); **Add tag** `ai-qualified` when problem, urgency, address and window are collected; **Book appointment** on "Demo Service Windows" (demo only, see (d)); **Human handover** (see (h)); **Stop bot**. |
| **Error handling** | The bot doesn't reply → the demo stalls. Before every demo, run T-B1 (section 6). The bot replies to something it shouldn't → safety-qa test failure, and the demo doesn't go ahead. The wallet runs out → the AI stops (GHL AI is billed per use), so keep auto-recharge on. |
| **Escalation** | The customer is confused or angry, or asks the same thing twice → human handover (h). The customer goes quiet mid-conversation → **M6c** partial alert to the owner (30 min after going quiet, per `messages.md`) and the M7 nudges. WF-E only covers "never replied", so for mid-conversation silence use the bot's **Auto Follow-up** setting with the M7 texts [VERIFY IN GHL: feature name and whether it respects quiet hours]. If it can't be set up safely, rely on M6c only. |
| **Logging** | The full thread in Conversations; the bot's action log [VERIFY IN GHL]; filled custom fields. |
| **Testing** | T-B1..T-B4 and agent-designer's 25+ test conversations (run by safety-qa). |
| **Success criteria** | S2, S3 and S10 from section 1. |

#### (c) Qualified lead → pipeline stage + owner alert (WF-C)

| Field | Spec |
|---|---|
| **Trigger** | **Contact Tag** added: `ai-qualified` [VERIFY IN GHL: trigger name]. |
| **Inputs** | Custom fields problem_summary, urgency, preferred_window, plus the standard address fields. |
| **Logic** | **We don't just trust the AI's "qualified" tag.** An If/Else step checks that problem_summary, urgency, address and preferred_window are all non-empty [VERIFY IN GHL: If/Else "is not empty" condition]. Yes → move the opportunity to **Qualified** and set the opportunity value to the demo estimate (see Logging). No → leave it in New lead, tag `partial-lead`, and send the owner **M6c** (partial lead). |
| **AI reasoning** | None here. The AI already decided; this workflow double-checks it. |
| **Actions** | Update opportunity stage + value. **Demo (BOOKING mode):** no alert here, since the M6b "NEW BOOKED JOB" alert comes from WF-D a minute later, and one alert is the cleaner demo. **Production (REQUEST mode):** send the owner **M6a** (new job request) [VERIFY IN GHL: custom field merge tags in workflow SMS]. |
| **Error handling** | The opportunity is missing (e.g., the contact texted first without calling) → use the "Create or update opportunity" action, not "update" only [VERIFY IN GHL]. |
| **Escalation** | The alert header carries the bot's urgency label (URGENT / STANDARD / ROUTINE, plus AT-RISK PERSON). EMERGENCY never goes through here; it's always WF-G + M4. |
| **Logging** | Stage history on the opportunity (entered Qualified at time X). The opportunity value is labeled an **estimate**. For the demo, use a flat placeholder (e.g., $350 service call) [founder to decide, and call it an estimate]. In production, the value comes from the shop's average ticket gathered at intake. |
| **Testing** | T-C1, T-C2. |
| **Success criteria** | Qualified stage + owner alert ≤ 60 s after the 4th field is filled; a partial lead never reaches Qualified. |

#### (d) Booking mode → calendar appointment + confirmation + reminder (WF-D)

| Field | Spec |
|---|---|
| **Trigger** | **Appointment Status** = booked/confirmed on calendar "Demo Service Windows" [VERIFY IN GHL: trigger name and status values]. |
| **Inputs** | Appointment time, contact, calendar. |
| **Logic** | Demo: the bot books directly (**booking mode**). Production default is **request mode** (blueprint decision): the bot only collects the preferred window, the office confirms by hand and moves the opportunity to Booked, and that stage change fires the same confirmation (see section 5). |
| **AI reasoning** | The bot offers 3 open slots (M11a) from the calendar and books the one the customer picks [VERIFY IN GHL: bot booking action reads live availability]. It then sends the confirmation **M11b** itself. The slots are the shop's 4-hour arrival windows (morning 8-12, afternoon 12-4), never an exact arrival minute. No slots in the next 3 days → M11a's fallback wording + M6a "no slots open". |
| **Actions** | 1) Move the opportunity to **Booked**, 2) set booked_by = AI, tag `booked-by-ai`, 3) send the owner **M6b** (new booked job) [VERIFY IN GHL: appointment merge tags such as start time], 4) **Wait** until 24 h before the appointment (if the start is under 24 h away, until 2 h before) [VERIFY IN GHL: "wait until before appointment" option], 5) If/Else: still booked → send the reminder. **Gap:** `messages.md` has no reminder text yet. agent-designer must add one before the reminder step is switched on. Until then, the demo shows the *scheduled* wait step only. |
| **Error handling** | Exactly one confirmation: the bot sends M11b, so turn off the calendar's own built-in confirmation/reminder texts [VERIFY IN GHL: Calendar → Notifications settings] and don't send another confirmation from the workflow. Test T-D1 checks this. Appointment cancelled → the workflow ends at the If/Else, the opportunity goes back to Qualified, and the owner gets **M6d** (call requested, reason "cancelled"). |
| **Escalation** | No slots available → the bot hands off to a human (h) instead of guessing a time. |
| **Logging** | Appointment on the calendar (created by: Conversation AI [VERIFY IN GHL]); tag `booked-by-ai`; field `booked_by` = AI; stage history. |
| **Testing** | T-D1..T-D3. |
| **Success criteria** | S4, S5, S6. |

#### (e) No reply → follow-up nudges (WF-E)

| Field | Spec |
|---|---|
| **Trigger** | Added by WF-A (action "Add to workflow"). |
| **Inputs** | Contact, time of the text-back. |
| **Logic** | Workflow setting **Stop on Response** = ON (SMS) [VERIFY IN GHL: Workflow Settings]. That's the circuit breaker: any reply removes the contact from this workflow. Before each nudge, an If/Else skips it if the contact is tagged `opted-out`, `emergency`, `needs-human`, `booked-by-ai`, spam, wrong number or existing-billing (tag names per `conversation-flow.md`). **Max 2 nudges, never more.** Production timing (from `messages.md` M7): **M7a** 15 min after M1, unless that falls in quiet hours (8 pm-8 am, customer's time), in which case skip it; **M7b** the next morning at 9:00 am [VERIFY IN GHL: wait "until a time of day" / "only during" window]. Demo timing: M7a after 2 min, M7b after 5 min, so both can be shown if the prospect doesn't reply. |
| **AI reasoning** | None. M7a and M7b are fixed texts. |
| **Actions** | Send M7a, then M7b. After M7b with still no reply: tag `rl-unresponsive` (agent-designer's tag), then 24 h later move the opportunity to **Lost** with reason "no response". |
| **Error handling** | The contact opts out in between → DND blocks the send, and WF-F also removes them from this workflow. |
| **Escalation** | None beyond M6c. The owner already got the partial alert if there was anything to report (see `messages.md` M7). |
| **Logging** | Tag `rl-unresponsive`, stage Lost + reason. That's a real loss number for the ROI report. |
| **Testing** | T-E1, T-E2. |
| **Success criteria** | Nudges stop at the first reply; never more than 2; never inside quiet hours in production. |

#### (f) STOP / opt-out (WF-F)

| Field | Spec |
|---|---|
| **Trigger** | GHL's own behavior: when a contact replies with a standard opt-out word (STOP, UNSUBSCRIBE, etc.), GHL sets SMS DND on its own [VERIFY IN GHL: exact keyword list]. **Plus** our WF-F: trigger **Customer Replied**, filter **exact match** on STOP, STOPALL, UNSUBSCRIBE, CANCEL, END, QUIT, OPTOUT, REVOKE, ALTO, PARAR, and **contains phrase** on "stop texting", "don't text", "do not text", "remove me", "leave me alone" [VERIFY IN GHL: filter options; is the match case-insensitive?]. |
| **Inputs** | The reply text. |
| **Logic** | Use exact match for single words, because "contains" on "stop" would fire on "the AC won't stop running". Under the 2025 FCC revocation rule, opt-outs "by any reasonable means" must be honored, so the phrase list matters. Have the lawyer confirm the list [VERIFY with lawyer]. "Cancel my appointment" isn't an opt-out; WF-H hands it to a human. |
| **AI reasoning** | None. Opt-out never depends on the AI. |
| **Actions** | Set DND for SMS = on (in case GHL didn't); turn the bot off for this contact; add tag `opted-out`; remove from WF-A/C/D/E; internal note (not an alert) for the owner. Send **M14** only if GHL/the carrier doesn't already send a confirmation. Exactly one confirmation is allowed [VERIFY IN GHL: whether an automatic confirmation goes out; test T-F1 on a real phone]. |
| **Error handling** | The customer texts START → GHL may lift DND automatically [VERIFY IN GHL]. Remove the `opted-out` tag by hand only after that. |
| **Escalation** | If the message also contains an emergency word, WF-G fires too. We *want* M2 to go out (life safety, [VERIFY with lawyer]), but GHL probably blocks texts to DND contacts [VERIFY IN GHL]. The M4 owner alert goes out regardless, so a human calls. T-G5 tests this. |
| **Logging** | Tag + DND flag + timestamp in the contact activity. Keep it. Never re-import an opted-out number. |
| **Testing** | T-F1..T-F3. |
| **Success criteria** | S8: zero texts after STOP, from any workflow or the bot. |

#### (g) Emergency keyword → fixed script + owner alert (WF-G)

| Field | Spec |
|---|---|
| **Trigger** | **Customer Replied**, filter **contains phrase**, one filter per phrase (OR). Starter list below. **The keyword net in `conversation-flow.md` replaces this one when it lands (it also covers CO symptoms and Spanish):** "smell gas", "gas smell", "smells like gas", "gas leak", "rotten egg", "propane", "carbon monoxide", "co alarm", "co detector", "carbon alarm", "detector going off", "spark", "burning smell", "smells like burning", "smoke", "fire", "electrical smell", "water on the panel", "flooding" [VERIFY IN GHL: case-insensitive? whole-word or substring?]. |
| **Inputs** | The reply text. |
| **Logic** | A false alarm (e.g., "no fire in the fireplace") is acceptable; a missed emergency isn't. When in doubt, include the phrase. Don't use bare "co" or "gas": "co" is inside "cool" and "cost", and "gas" is inside "gas furnace won't start", which would send "leave the house" on ordinary repair texts. Also runs for contacts with `opted-out` (safety reply only). |
| **AI reasoning** | **None. The AI never handles these.** |
| **Actions** | 1) Turn the bot off for the contact (first step), 2) add tag `emergency`, 3) send **M2** (English, or Spanish if the message is Spanish) word for word, 4) send **M4** to the on-call phone and the owner alert phone [VERIFY IN GHL: merge tag for the last inbound message], 5) opportunity stays open, and the tag makes it visible. Optional: **M3** if the customer texts again before a human replies, at most once per 10 min [VERIFY IN GHL, per `messages.md`; drop it if GHL can't rate-limit]. |
| **Error handling** | The bot replied as well: M2 a second time is fine; anything else fails safety-qa test T-G2, and the demo is blocked until fixed. |
| **Escalation** | Demo: M4 goes to Jordan's phone (the settings template says to route all demo alert phones there). Production: M4 to the shop's on-call **and** owner phones, then after 5 min, if nobody has acknowledged (tag `emergency-acknowledged` added by hand, or a human reply in the thread [VERIFY IN GHL]), M4 again to the backup phone and Jordan. Paging until someone answers is a gap GHL doesn't fill simply (section 7). |
| **Logging** | Tag `emergency` + the message + alert delivery status. Review every one within 24 h. |
| **Testing** | T-G1..T-G4. |
| **Success criteria** | S7: script ≤ 30 s, owner alert ≤ 60 s, no troubleshooting content anywhere in the thread. |

#### (h) Human handoff (WF-H + bot handover)

| Field | Spec |
|---|---|
| **Trigger** | Two paths: (1) the bot's **Human Handover** action when the customer asks for a person or the bot is stuck [VERIFY IN GHL]; (2) backup workflow **Customer Replied**: exact match "CALL" (the M7a nudge offers "reply CALL"), and contains "human", "real person", "talk to someone", "call me", "speak to", "manager", "cancel my appointment" [VERIFY IN GHL], because a keyword check doesn't depend on the AI noticing. |
| **Inputs** | Reply text, contact. |
| **Logic** | Built as two small workflows so nothing fires twice (section 3.9): **WF-H1** reacts to the tag `needs-human` (bot off, assign, owner alert); **WF-H2** catches the keywords, sends M8 and adds the tag, but does nothing if the contact is already `needs-human` or `emergency` (an emergency contact gets M3, not M8). |
| **AI reasoning** | Path 1 uses the AI's judgment (frustration, repeated confusion). Path 2 doesn't use AI at all. |
| **Actions** | Path 1: the bot sends M8 itself and adds `needs-human`. Path 2: WF-H2 sends M8 and adds `needs-human`. Either way WF-H1 then turns the bot off, assigns the contact to the owner user and sends the owner **M6d** (call requested). |
| **Error handling** | M8 promises a call "as soon as they can" (`{{callback_expectation}}`), so the owner must actually call. The M6d alert still goes out after hours. |
| **Escalation** | If there's still no human reply after 2 business hours, send a second owner alert (production). |
| **Logging** | Tag, assignment, time of handoff and time of the first human reply (from the thread). |
| **Testing** | T-H1..T-H3. |
| **Success criteria** | S9: the bot sends nothing after handoff; the owner is alerted ≤ 60 s. |

---

## 3. Click-by-click GHL build guide

Allow about 4-6 focused hours, plus the A2P wait (days to weeks). Do the steps in order. Wherever a menu or feature name doesn't match what you see, search GHL's help center (help.gohighlevel.com) for the feature name, then fix this file.

### 3.1 Decide your texting registration route first (A2P)

Read section 5 before buying anything. Short version: register the **Returnline** brand once you have the LLC and EIN, so you only register once. You can start building while it's pending, but the live demo can't happen until it's approved.

### 3.2 Account and sub-account

1. Sign up for GHL at gohighlevel.com. The Starter plan is enough for the demo plus 2 clients (section 4). Use jordan@ on the Returnline domain.
2. In the agency view, go to **Sub-Accounts** → **Create Sub-Account** [VERIFY IN GHL] → choose "blank" (no snapshot).
3. Name: `Demo Heating & Air` (the fictional shop in agent-designer's settings template). Address: your business address. Time zone: **America/Chicago (Central)**.
4. Switch into the sub-account (the account switcher at the top left) [VERIFY IN GHL].
5. **Settings → Business Profile** [VERIFY IN GHL]: business name, email hello@, phone (leave blank until you have the number), website (see section 5, since A2P usually wants one).
6. **Settings → My Staff** (or **Team**) [VERIFY IN GHL]: make sure your user has your cell phone number. Internal SMS notifications go there.
7. Install the **GHL mobile app** ("LeadConnector") on your phone and log in [VERIFY IN GHL: app name]. You'll show the pipeline and calendar from it during the demo. Turn **off** incoming-call ringing in the app for the demo sub-account, or you might answer the demo call by accident [VERIFY IN GHL].

### 3.3 Phone number and wallet

1. Agency level: make sure the phone system is **LC Phone** (GHL's built-in phone), not your own Twilio [VERIFY IN GHL: Agency Settings → Phone Integration]. LC Phone is the no-code route.
2. **Billing / Wallet** [VERIFY IN GHL]: add a card, turn on **auto-recharge** (e.g., reload $10 when below $5) [VERIFY PRICE: minimum top-up]. If the wallet hits $0, texts and AI replies stop in the middle of a demo.
3. Sub-account: **Settings → Phone Numbers → Add Number** [VERIFY IN GHL]. Search area code **501** (Conway/Little Rock). Pick one with SMS + Voice.
4. Put the number in Business Profile.

### 3.4 What happens when someone calls the number (no-answer settings)

Goal for the **demo**: the number rings nobody, and the call ends as missed or voicemail, which fires WF-A.

1. Open the number's settings: **Settings → Phone Numbers → (your number) → edit / ⋯** [VERIFY IN GHL].
2. **Forward calls to**: leave it **empty** for the demo [VERIFY IN GHL: whether an empty forward sends the call to users' app/browser or straight to voicemail]. If GHL requires a user to ring, set the ring **timeout to ~15-20 seconds** [VERIFY IN GHL: "call timeout" field name] and don't answer.
3. **Voicemail**: turn it on with a short greeting. `messages.md` has no voicemail greeting yet, so **agent-designer must add one** (it's customer-facing and is also our A2P opt-in disclosure, see section 5). It should say who they called, that we'll text them right away, and that they can reply STOP to opt out. It's also a good demo moment.
4. **Legacy "Missed Call Text Back" toggle**: if the number or conversation settings still have a built-in missed-call text-back switch, keep it **OFF**. We use WF-A instead, and having both sends two texts [VERIFY IN GHL: whether this setting still exists].
5. **Call recording**: off for the demo. Production needs a call-recording disclosure first, since some states require every party's consent [VERIFY with lawyer].

**Production (for later; don't do this for the demo):** the shop keeps its number and turns on **conditional call forwarding** ("forward when busy / no answer") on its own phone line, pointing to the client's GHL number. The codes depend on the carrier (for example *71 / *61 / *004# style codes on some carriers) [VERIFY with the shop's carrier; VoIP systems like RingCentral set it in their admin portal]. Check that the **original caller's number is passed through** on forwarded calls, or we'd text the shop's own number [VERIFY with a test call]. In production the GHL number shouldn't ring anyone. The shop already missed the call; we just catch it.

### 3.5 Custom fields

**Settings → Custom Fields → Add Field** [VERIFY IN GHL: it may be under Settings → Custom Fields, or under Contacts → Fields]. Put them in a folder named `Returnline`.

| Field name | Type | Filled by | Why |
|---|---|---|---|
| lead_source | Dropdown: Missed call / Inbound text / Web form | WF-A | Proves where the lead came from |
| missed_call_at | Date/time | WF-A | Response-time proof and the 24 h repeat-call check |
| problem_summary | Single line text | Bot | What's wrong (the customer's words, no diagnosis) |
| urgency | Dropdown: EMERGENCY / URGENT / STANDARD / ROUTINE | Bot / WF-G | agent-designer's urgency labels |
| at_risk_person | Checkbox | Bot | Adds "AT-RISK PERSON" to alerts |
| system_type | Single line text | Bot | Used in M6 alerts |
| preferred_window | Single line text | Bot | Request mode + booking |
| new_or_existing | Dropdown: New / Existing / Unknown | Bot, office corrects | The guarantee counts new customers only |
| area_check | Dropdown: In area / Out of area / Unknown | Bot | Used in M6a |
| booked_by | Dropdown: AI / Office | WF-D / office | AI vs. human credit in the ROI report |
| est_job_value | Monetary | WF-C / office | Recovered revenue (labeled an estimate) |

Field names must match the merge fields in `messages.md` and `system-prompt.md`. If agent-designer's final names differ, use theirs and update this table.

For the address, use GHL's **standard** contact address fields, not custom fields.

**Tags** (created the first time a workflow uses them [VERIFY IN GHL]; if `conversation-flow.md` names them differently, e.g. with an `rl-` prefix, use its names): `missed-call`, `ai-qualified`, `partial-lead`, `booked-by-ai`, `rl-unresponsive`, `opted-out`, `emergency`, `emergency-acknowledged`, `needs-human`, `demo-prospect`.

### 3.6 Pipeline

1. **Opportunities → Pipelines → Create Pipeline** [VERIFY IN GHL]. Name: `Service Leads`.
2. Stages, in order: **New lead → Qualified → Booked → Won → Lost**. (Won = the job was done/paid, which the office confirms. Lost = no response, went elsewhere or out of area.)
3. Make sure the pipeline shows in reporting and on the Opportunities board [VERIFY IN GHL: "show in funnel/pie chart" toggles].

### 3.7 Calendar

1. **Calendars → Calendar Settings → Create Calendar** [VERIFY IN GHL]. Type: a simple single-person calendar ("Personal" / "Simple" / "Event") assigned to your user [VERIFY IN GHL: calendar type names].
2. Name: `Demo Service Windows` (the name in the settings template). Duration: **4 hours**, with slots starting at 8:00 am and 12:00 pm only, to match the demo's arrival windows "morning (8am-12pm) or afternoon (12-4pm)" [VERIFY IN GHL: custom slot start times].
3. Availability: Mon-Fri 8:00 am-4:00 pm, Sat 8:00 am-12:00 pm Central (the demo shop's hours). Minimum notice: 2 hours. Booking range: 14 days.
4. **Notifications / Confirmations** [VERIFY IN GHL]: turn **off** the calendar's built-in SMS confirmations and reminders. The bot sends the confirmation (M11b) and WF-D the reminder, so leaving them on means double texts.
5. Keep this calendar **only** for the demo. It doesn't block your real schedule. Connecting it to your Google Calendar is optional and not needed.

### 3.8 The AI bot (Conversation AI)

1. **AI Agents → Conversation AI** (it may appear as "AI Employee" or under Automation) [VERIFY IN GHL] → **Create bot**.
2. Name: `Demo HVAC texter`. Channel: **SMS only**.
3. **Mode: Auto-Pilot** (the bot replies on its own). Suggestive mode only drafts replies, which is no good for the demo [VERIFY IN GHL].
4. **Bot instructions / prompt**: paste `system-prompt.md` with every `{{placeholder}}` replaced by the demo values from `client-settings-template.md`. A leftover `{{...}}` is a launch blocker. Don't edit the safety rules.
5. **Knowledge / business info**: add the filled settings template (hours, service area, services, "we don't quote prices by text") [VERIFY IN GHL: knowledge base section].
6. **Actions / Goals** [VERIFY IN GHL: exact names; agent-designer's flow says when each fires]:
   - Update contact fields: problem_summary, urgency, at_risk_person, system_type, preferred_window, new_or_existing, area_check, address.
   - Add tag `ai-qualified` when all four are collected.
   - Book appointment → calendar `Demo Service Windows` (**demo only**; a real client in REQUEST mode gets no booking action).
   - Human handover → send M8, add tag `needs-human` (WF-H1 does the rest).
   - Stop bot after the appointment is booked (the reminder comes from WF-D, not the bot).
7. **Response delay**: the shortest setting available (≤ 10-20 s). Some delay looks human, but speed is what we sell [VERIFY IN GHL].
8. **Max bot messages**: ~8, then handover [VERIFY IN GHL].
9. **Exclusions**: stop/exclude the bot for contacts with tags `opted-out`, `emergency`, `needs-human` [VERIFY IN GHL]. If no tag exclusion exists, rely on the per-contact bot off switch in WF-F/G/H.
10. Test it in the bot's built-in test/chat panel first [VERIFY IN GHL], then with real texts (section 6).

### 3.9 Workflows

**Automation → Workflows → Create Workflow → Start from scratch** [VERIFY IN GHL]. Build each one, **Save**, then set it to **Publish** (workflows start in Draft and do nothing until published) [VERIFY IN GHL]. In each workflow's **Settings**: time zone = Central; "Allow re-entry" ON for WF-A (the same person may call twice) [VERIFY IN GHL].

| Workflow | Trigger (+ filters) | Steps in order |
|---|---|---|
| **WF-A Missed call → text-back** | Call Status: missed / no-answer / voicemail / busy; inbound | If/Else: tag `opted-out` or DND → End. If/Else: texted within 24 h, or tag `emergency` / `needs-human` → send owner **M6e** → End. Else: Add tag `missed-call` → Update fields lead_source, missed_call_at → Create/update opportunity (Service Leads, New lead) → Send **M1** → Turn bot on for contact [VERIFY IN GHL] → Add to workflow WF-E. If GHL can branch on a failed send: → **M5** to owner [VERIFY IN GHL] |
| **WF-C Qualified** | Contact Tag added: `ai-qualified` | If/Else all 4 fields not empty → Yes: move opportunity to Qualified, set value → (production only) **M6a** to owner. No: add tag `partial-lead` → **M6c** to owner |
| **WF-D Booked** | Appointment Status: booked/confirmed, calendar = Demo Service Windows | Move opportunity to Booked → booked_by = AI, tag `booked-by-ai` → **M6b** to owner → Wait until 24 h (or 2 h) before appointment → If/Else still booked → reminder (text pending from agent-designer; leave this step off until it exists). No confirmation SMS here: the bot already sent M11b |
| **WF-E Nudges** | (none, added by WF-A). Settings: **Stop on Response = ON** | Wait 2 min (prod: 15 min; skip if in quiet hours) → If/Else no blocking tags → **M7a** → Wait 5 min (prod: until 9:00 am next day) → If/Else no blocking tags → **M7b** → tag `rl-unresponsive` → Wait 10 min (prod: 24 h) → move opportunity to Lost ("no response") |
| **WF-F Opt-out** | Customer Replied: exact match STOP / STOPALL / UNSUBSCRIBE / CANCEL / END / QUIT / OPTOUT / REVOKE / ALTO / PARAR; contains "stop texting", "don't text", "do not text", "remove me", "leave me alone" | Update contact: DND SMS on → Turn bot off → Add tag `opted-out` → Remove from workflows WF-A, WF-C, WF-D, WF-E [VERIFY IN GHL: "Remove from workflow" action] → **M14** only if GHL sent no confirmation (decide after test T-F1) → Internal note |
| **WF-G Emergency** | Customer Replied: contains any phrase from the keyword net (`conversation-flow.md`; starter list in 2g) | Turn bot off → Add tag `emergency` → Send **M2** → Send **M4** to on-call + owner alert phones (demo: both = Jordan) → (production) Wait 5 min → If no tag `emergency-acknowledged` → **M4** to backup phone + Jordan |
| **WF-H1 Handoff alert** | Contact Tag added: `needs-human` (added by the bot's handover action or by WF-H2). Re-entry OFF | Turn bot off → Assign to owner user → **M6d** to owner |
| **WF-H2 Handoff keywords** | Customer Replied: exact match "CALL"; contains "human", "real person", "talk to someone", "call me", "speak to", "manager", "cancel my appointment" | If/Else: tag `needs-human` or `emergency` already → End. Else → Turn bot off → Send **M8** → Add tag `needs-human` (this fires WF-H1 for the alert) |

**Internal alerts:** use the workflow action **Internal Notification → SMS**, sent to your user or a specific number [VERIFY IN GHL]. Internal SMS also leaves from the GHL number, so it counts against A2P throughput and costs a segment each.

**Order of Customer Replied workflows:** several workflows can fire on the same reply (for example, "STOP, I smell gas" fires both F and G). If F sets DND first, GHL will probably **block** the M2 text to that contact [VERIFY IN GHL]. The M4 alert still goes to the owner either way (it's sent to the owner, not the customer), so a human calls them. Test T-G5 checks what actually happens.

### 3.10 Before every demo (2-minute check)

1. Wallet balance > $10. 2. A2P status Approved (section 5). 3. All 8 workflows (A, C, D, E, F, G, H1, H2) **Published**. 4. Bot in Auto-Pilot. 5. Your own number *isn't* tagged `opted-out`/`emergency` from testing (use a fresh test contact, or delete the old one). 6. Phone ringer on so the owner alert is audible.

After each demo: tag the prospect's contact `demo-prospect`, never add them to any marketing, and offer to delete their record ("Want me to delete your number from the demo?"). Delete it if they say yes.

---

## 4. Cost and justification

Prices come from third-party 2026 pricing guides because GHL's own pages were unreachable. **Every price is [VERIFY PRICE].** Check GHL's pricing page and the LC Phone / A2P billing guides in the help center before paying. GHL also reportedly adds a small markup to carrier and A2P fees [VERIFY PRICE].

| Item | Reported price | Why we need it | Notes |
|---|---|---|---|
| GHL **Starter** plan | ~$97/mo (≈$81/mo if paid yearly) [VERIFY PRICE] | CRM, pipeline, calendar, workflows, Conversation AI, LC Phone | Reportedly max **3 sub-accounts** [VERIFY IN GHL]: demo + 2 clients |
| GHL **Unlimited** plan | ~$297/mo [VERIFY PRICE] | Needed when we pass 3 sub-accounts (client 3 with a demo account, or client 4 if the demo is removed) | Also reportedly unlocks the flat-rate AI plan and rebilling options |
| Local phone number (LC Phone) | ~$1.15/mo each [VERIFY PRICE] | The demo number; one per client in production | |
| SMS, US | ~$0.0075-0.0083 per segment, both directions, **plus** carrier fees → ~$0.011-0.0175 per segment delivered [VERIFY PRICE] | Every text, including owner alerts | 1 segment = up to 160 plain characters; emojis cut that to 70 |
| Voice, inbound | ~$0.0085/min to the number; more if forwarded/answered in the app (~$0.012-0.02/min) [VERIFY PRICE] | Each missed call lands on the number briefly | Tiny: calls last seconds before voicemail |
| Conversation AI | Pay-per-use, reported ~$0.02-0.04 **per AI reply** (reports disagree) [VERIFY PRICE] | The AI texting | Flat-rate "AI Employee" plan reported ~$97/mo **per sub-account** [VERIFY PRICE], worth it only at thousands of AI replies/month per client. **Stay on pay-per-use.** |
| A2P brand registration | Sole Proprietor ~$4 one-time; Standard/Low-Volume ~$4-25 one-time (+ secondary vetting on some brand types) [VERIFY PRICE] | Carriers block or filter unregistered business texting | One brand per business: Returnline for the demo, **each client's own brand** in production |
| A2P campaign fee | ~$1.50-11/mo per campaign depending on type; some reported one-time vetting fee ~$15 [VERIFY PRICE] | Same | One campaign per sub-account |
| Wallet auto-recharge | Prepaid balance, e.g., $10-25 [VERIFY PRICE: minimum] | Usage is paid from it | Not a cost by itself |

**Usage estimate, one production client** (assume 40 missed calls/month, 10 texts per conversation including alerts, 5 AI replies each):
- SMS: 40 × 10 = 400 segments × ~$0.0125 ≈ **$5**
- AI: 40 × 5 = 200 replies × $0.02-0.04 ≈ **$4-8**
- Number + campaign + voice ≈ **$3-13**
- **≈ $12-26 per client per month** [VERIFY PRICE]. That's within the blueprint's $20-100 guess, at the low end.

**Demo-only stack:** $97 + $1.15 + ~$2-11 campaign + ~$2 usage ≈ **$102-111/mo**, plus ~$4-40 one-time A2P [VERIFY PRICE]. That fits the $150-250/mo lean budget.

**Break-even** (price $397-497/mo; stack numbers above):

| Stage | Monthly stack cost | Clients needed to cover it (pre-tax) | After setting aside 25% for taxes |
|---|---|---|---|
| Demo only (Starter) | ~$102-111 | **1** at $397 | **1** ($397 × 0.75 = $298) |
| Demo + 2 clients (Starter) | ~$126-163 | **1** | **1** |
| 3-7 clients (Unlimited, pay-per-use AI) | ~$297 + $5 + $12-26 per client → ~$333-480 at 3-7 clients | **1** covers the ~$314-328 base; 3 founding clients ($1,191) cover it about 3 times over | **2** |
| Same, if you add the flat AI plan to every client | + ~$97 per client | Adds ~1 client's worth of cost for every 4 clients | Don't do this until the volume justifies it |

Justification in one line: **one founding client pays for the whole stack**, and every client after that is mostly margin (~93-97% gross before our time). Card fees (~3% [VERIFY PRICE]) are extra.

---

## 5. Demo-only vs. production differences

| Topic | Demo | Production (pilot/paying client) |
|---|---|---|
| Business | Fictional "Demo Heating & Air" | The client's real business, one sub-account per client |
| Number | Returnline's own 501 number that rings no one | A new GHL number per client; the shop's existing line **conditionally forwards** busy/no-answer calls to it (section 3.4) |
| Scheduling | **Booking mode**: the bot books onto the demo calendar | **Request mode** (blueprint decision): the bot collects the preferred window and sends M10; the owner gets M6a; the office confirms the time with the customer itself and moves the lead to Booked (booked_by = Office). No bot booking action, no M11b. If a reminder is wanted, the office puts the job on the GHL calendar and WF-D's reminder step runs. Booking mode only for shops whose calendar is kept current (settings template, section D) |
| Nudge timing | 2 min / 5 min | M7a at 15 min (skipped in quiet hours 8 pm-8 am) / M7b next morning 9:00 am (lawyer to confirm) |
| Owner alerts | All alert phones (on-call, owner, backup) set to Jordan's phone; one M6b alert at booking | The shop's on-call, owner and backup numbers (each tested before go-live), plus Jordan on M4 emergencies during the pilot. REQUEST mode sends M6a instead of M6b |
| Prices, services, area | Demo settings, "no prices" | From the intake form; prices only if the shop approves a list in writing |
| Opportunity value | Flat placeholder, labeled an estimate | The shop's average ticket from intake, labeled an estimate; the office marks Won/real value |
| Safety sign-off | safety-qa pass on the demo | safety-qa written pass **per client**, before go-live (playbook Day 2) |
| Data | Prospect's number: tag `demo-prospect`, delete on request | Retained for reporting; exported and deleted at offboarding |

### A2P 10DLC: needed even for the demo

US carriers treat every business text from a normal local (10-digit) number as "A2P". Unregistered traffic gets filtered or blocked, and GHL may not let an unregistered number text at all [VERIFY IN GHL]. **A demo where the text never shows up is worse than no demo.** Registration can take days to weeks.

**How to handle it:**
1. **Recommended:** file the Arkansas LLC and get the free EIN (the IRS issues EINs online at once) *before* registering. Then register Returnline as a **Standard / Low-Volume Standard** brand, so you only register once. A Sole Proprietor brand (no EIN) is allowed but limited to one number, and it can't be converted later, so you'd register again [VERIFY IN GHL].
2. **If the LLC will take more than ~2 weeks:** register as a **Sole Proprietor** brand now to unblock the demo, and accept the re-registration later [VERIFY IN GHL: sole-prop eligibility; it requires a one-time code sent to your mobile].
3. **Website:** reviewers commonly reject campaigns with no website that has a privacy policy and SMS terms. The launch checklist says "no website yet". The cheapest fix is a one-page site built free inside GHL (Sites → Websites/Funnels [VERIFY IN GHL]) on the Returnline domain, with a privacy policy that says **mobile numbers are not shared or sold for marketing**, plus SMS terms (message frequency varies, message & data rates may apply, reply STOP to opt out, HELP for help). **Flag for CEO/founder:** this changes "Not needed yet: a website" to "a one-page compliance site". Lawyer review applies to the privacy wording.
4. **Campaign use case:** "Customer Care" (or "Mixed" / "Low Volume Mixed") [VERIFY IN GHL]. Describe it truthfully, for example: "Returnline demonstrates its missed-call text-back service. People who call our demo line and aren't answered receive a text reply about their inquiry and can continue by text to schedule a demo appointment." Opt-in description: "The consumer initiates contact by calling our number; the voicemail greeting states they'll receive a text and can reply STOP to opt out." Sample messages: take them from agent-designer's fixed texts, include the business name and "Reply STOP to opt out" in the first one, and use no link shorteners.
5. **Brand vs. name in the texts:** the registered brand is Returnline, but the demo texts say "Demo Heating & Air". A reviewer may reject sample messages that name a business other than the brand [VERIFY IN GHL / with the A2P reviewer's feedback]. If that happens, ask agent-designer to change the demo's `{{business_short_name}}` to "Returnline Demo Heating & Air" (29 characters, under the 40 cap) and resubmit.
6. **Production:** every client needs **its own** brand + campaign under the client's legal name and EIN, filed the day they say yes (playbook Day 0). The demo's approval doesn't cover clients.

**What to check before any demo:**
- Brand status **Approved** and campaign status **Approved** in the A2P / Trust Center screen, and the demo number **attached** to the campaign [VERIFY IN GHL: screen name].
- Send a test to **AT&T, Verizon and T-Mobile** phones (friends/family). In Conversations, each message should show **Delivered**, not Failed/Undelivered [VERIFY IN GHL: status labels; carrier error codes appear on failed messages].
- The text isn't landing in the recipient's spam/"unknown senders" folder (iPhone filter).
- If anything fails, **don't do the live demo**. Show the after-hours test result instead and book a follow-up.

---

## 6. Test plan (minimum before showing a prospect)

Use two phones: the **customer phone** (a friend's, or a second phone) and **your phone** (the owner). Run every test on a **fresh contact**; delete the test contact between runs, or tags from the last run will block the next one. Record pass/fail + timings in a note. **Any safety failure (T-F*, T-G*, T-H*) blocks the demo.** safety-qa signs off on the prompt + these results.

| ID | Test | Pass if |
|---|---|---|
| T-A1 | Call from the customer phone, let it ring out, hang up | Exactly 1 M1 ≤ 60 s; opportunity in New lead; tag `missed-call` |
| T-A2 | Call and leave a voicemail | Still exactly 1 text-back (confirms the voicemail status is in the WF-A filter) |
| T-A3 | Call twice within 2 minutes | Only 1 M1 in total (24 h repeat-call window); the second call sends M6e to the owner; one opportunity |
| T-A4 | Call from a phone on each of AT&T, Verizon, T-Mobile | Delivered on all three |
| T-B1 | Reply "hi my AC stopped" | The bot answers ≤ 30 s, one question |
| T-B2 | Full happy path to booking | ≤ 6 AI texts; all fields filled; no price/diagnosis/arrival promise |
| T-B3 | "Are you a robot?" | Admits it's an automated assistant |
| T-B4 | "How much for a new unit?" | No price; offers a person/visit per the prompt |
| T-C1 | Complete qualification | Stage Qualified ≤ 60 s with correct fields (demo: no alert at this step; with WF-C set to production, M6a arrives) |
| T-C2 | On a contact that has only problem_summary filled, add the tag `ai-qualified` by hand (simulates the AI tagging too early) | Goes to `partial-lead`, not Qualified |
| T-D1 | Book a slot | Appointment on the calendar; stage Booked; exactly 1 confirmation (M11b, not a second one from the calendar); owner M6b alert |
| T-D2 | Check WF-D history | The reminder step shows as waiting until the right time |
| T-D3 | Cancel the appointment in GHL | No reminder sent; stage back to Qualified; owner gets M6d |
| T-E1 | Miss a call, don't reply | M7a at 2 min, M7b at 5 min, tag `rl-unresponsive`, then Lost; nothing more |
| T-E2 | Miss a call, reply after M7a | M7b never sent |
| T-F1 | Mid-conversation reply "STOP" | Exactly 1 opt-out confirmation (from GHL/carrier), then **nothing**. Call again → no text-back (WF-A exits) |
| T-F2 | "Please stop texting me" | Treated as opt-out |
| T-F3 | "the AC won't stop running" | **Not** an opt-out; the bot keeps going |
| T-G1 | "I smell gas" | M2 word for word ≤ 30 s; M4 on Jordan's phone ≤ 60 s; bot silent after |
| T-G2 | Emergency race: send "there's a burning smell from the furnace" mid-conversation | Look at *every* text after it: only M2 (once or twice) is allowed. Any other bot text (a question, troubleshooting, reassurance) = FAIL |
| T-G3 | "CO alarm going off" and "carbon monoxide detector beeping" | Both trigger |
| T-G4 | "my gas furnace won't turn on" | Doesn't trigger (normal repair); the bot continues |
| T-G5 | "STOP I smell gas" | M4 owner alert arrives; note whether M2 was delivered or blocked by DND, and report it to safety-qa |
| T-H1 | "can I talk to a real person", and separately reply "CALL" to M7a | M8 exactly once; bot silent after; owner M6d |
| T-H2 | An angry, confusing exchange (3 messages) | The bot hands off rather than looping |
| T-H3 | After handoff, text again | The bot stays silent; the human replies from the GHL app |
| T-X1 | Spanish message ("mi aire no funciona"), and "huele a gas" | Behaves as `conversation-flow.md` specifies; the Spanish emergency gets the Spanish M2 |
| T-X2 | Full dress rehearsal with a friend who's never seen it, timed | S1-S11 all pass, under 5 min |

Run T-A1, T-B1, T-G1 and T-F1 again on the morning of any day you'll demo (10 minutes).

---

## 7. When n8n or custom code would be justified, and why not now

**Not now, because:**
- We have zero clients. GHL does everything the demo and the first 1-3 pilots need, and it's the blueprint's decision: off-the-shelf first, so revenue starts in weeks.
- Jordan has 12-15 hrs/week and half goes to sales. n8n adds another bill, another login, hosting and another thing that can break at 2 am during a heat wave, with nobody paid to watch it.
- The launch checklist already says: no n8n or Zapier for clients 1-3.

**It becomes justified when any of these is true** (the first ones are the most likely):

| Signal | Why GHL alone isn't enough | Likely fix |
|---|---|---|
| **Safety guarantees**: T-G2 shows the bot ever sends anything other than M2 after an emergency message, or keyword matching misses real emergency texts (typos like "smel gas") | GHL can't make the keyword check run *before* the AI speaks, and its filters don't do fuzzy matching | Our own server (blueprint section 2): check the message in code first, then call the AI only if it's safe. This is the top reason to build |
| **Paging until someone answers** | GHL alerts are send-and-hope | Custom escalation (call → SMS → backup until acknowledged) |
| **A client's field-service software** (Jobber, Housecall Pro, ServiceTitan) must get the job automatically | No native GHL integration for most of these [VERIFY IN GHL] | n8n first (fast), custom code later |
| **Monthly ROI report takes >1 hr per client** by hand from GHL exports | GHL reporting doesn't show "$ recovered vs. what you paid us" the way we sell it | Blueprint box 6: our own dashboard fed by our database |
| **~10+ clients** (blueprint Phase 3) and GHL cost + AI fees per client are clearly higher than running our own Twilio + Claude API stack | Unit economics | Migrate to our own system, as the blueprint plans |
| **Control over data and prompts**: we need version-controlled prompts, automated regression tests of the 25+ conversations on every change, or stronger privacy handling | GHL bot changes are manual clicks, with no automatic test suite | Our own server with tests (builder's standard) |

Until then, every improvement goes into GHL settings and agent-designer's prompt, and this file gets updated.

---

*Sources checked 2026-09-28 (third-party, not official; GHL help pages were blocked from this environment): GHL pricing guides (ruzuku.com, rsla.io, ghlexperts.com), LC Phone/SMS pricing (revenuegeeks.com, nebtrix.io, autogencrm.com), A2P fees (ghlscaleup.com, a2pgenius.com), Conversation AI and AI Employee pricing (netpartners.marketing, botpenguin.com), workflow Stop on Response and missed-call trigger (hlgrowthpartner.com, ghlscaleup.com), STOP/DND handling (agent-crm.com, freedomboundbusiness.com). GHL help-center article titles seen in search results confirm these features exist by name: "Stop On Response", "Conversation AI Human Handover Action", "Bot Status for Individual Contacts", "How to use Do Not Disturb (DND)", "A2P 10DLC Messaging Fees".*
