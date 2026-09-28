# Client settings template

Status: DRAFT v1 (2026-09-28). Needs a safety-qa pass before builder uses it.

We fill this in at the Day 0 intake (see `docs/02-sales-and-delivery.md`, Part 2). Every `{{placeholder}}` in `system-prompt.md` and `messages.md` comes from this sheet. Before pasting into GoHighLevel (GHL), builder replaces every placeholder by hand. A leftover `{{...}}` is a launch blocker.

[VERIFY IN GHL] Can the Conversation AI prompt read GHL Custom Values, such as `{{custom_values.business_name}}`, at runtime? If it can, builder may map these settings to Custom Values instead of hard-coding them. If not, or if we're unsure, hard-code them. Custom Values do work in workflow SMS actions, so `messages.md` can use them there.

## Rules for filling this in

- Write down what the owner actually said. Don't guess. If the owner hasn't decided something, use the default in the "Default" column.
- Settings marked **LOCKED** are Returnline safety rules. The client cannot change them. If a client asks to change one, the answer is no, and we tell the CEO.
- Keep the owner's signed-off copy of this sheet in the client's folder. When a setting changes later, date it.

---

## A. Business basics

| Setting | Placeholder | What it is | Required | Default |
|---|---|---|---|---|
| Business name | `{{business_name}}` | Public name, as customers know it | Yes | none |
| Short name for texts | `{{business_short_name}}` | The name used inside texts. **35 characters max**, so every templated text stays at 160 characters or under (the M7a nudge is the tightest) | Yes | same as business name |
| City and state | `{{business_city_state}}` | For example "Conway, AR" | Yes | none |
| Main business phone | `{{business_phone}}` | The number customers call. It's fine to show it to customers | Yes | none |
| Business hours | `{{business_hours}}` | Days and times the office answers | Yes | none |
| Time zone | `{{timezone}}` | Used for quiet hours and the follow-up timing | Yes | America/Chicago |
| Existing-customer software | `{{fsm_software}}` | Jobber, Housecall Pro, ServiceTitan, paper, etc. For our notes only; the AI doesn't use it | No | none |

## B. Service area

| Setting | Placeholder | What it is | Required | Default |
|---|---|---|---|---|
| Service area (words) | `{{service_area}}` | Cities, towns and counties served | Yes | none |
| Service area ZIPs | `{{service_area_zips}}` | ZIP list. The AI treats an address as in-area if its city or ZIP matches | Yes | none |
| Out-of-area policy | `{{out_of_area_policy}}` | `PASS_TO_OFFICE` (take the details, and the office decides) or `DECLINE` (say politely that we don't serve there) | Yes | `PASS_TO_OFFICE` |

## C. Services

| Setting | Placeholder | What it is | Required | Default |
|---|---|---|---|---|
| Services offered | `{{services_offered}}` | For example: AC repair, furnace repair, heat pumps, tune-ups, new system estimates, ductwork | Yes | none |
| Services NOT offered | `{{services_not_offered}}` | For example: plumbing, water heaters, commercial rooftop units, window units | Yes | "none listed" |
| Brands/equipment notes | `{{equipment_notes}}` | Anything the office wants flagged, e.g. "we don't service geothermal". The AI uses it only to flag, never to diagnose | No | none |

## D. Scheduling mode

| Setting | Placeholder | What it is | Required | Default |
|---|---|---|---|---|
| Mode | `{{mode}}` | `REQUEST` or `BOOKING` (see below) | Yes | **`REQUEST`** |
| Standard time windows | `{{time_windows}}` | How the shop describes its arrival windows, e.g. "morning (8am-12pm) or afternoon (12-4pm)" | Yes | "morning or afternoon" |
| Booking calendar | `{{booking_calendar}}` | The GHL calendar name. BOOKING mode only | BOOKING only | none |
| Slots offered | `{{slots_to_offer}}` | How many open slots to offer: 2 or 3 | BOOKING only | 3 |
| Calendar kept current? | (intake note) | Owner confirms that someone keeps this calendar up to date every day. **If the answer is no, the client uses REQUEST mode.** | BOOKING only | no, so REQUEST |

**REQUEST mode** is the default for real clients (decisions log, 2026-09-28). The AI collects the details and a preferred window. It tells the customer the office will confirm the time, and it alerts the owner. It never promises a time.

**BOOKING mode** is for the sales demo, and for clients whose calendar is kept current. After the same qualifying questions, it offers 2-3 open slots from the GHL calendar and books one. Then the owner gets a "NEW BOOKED JOB" alert.

## E. Prices, diagnoses, promises

| Setting | Placeholder | What it is | Required | Default |
|---|---|---|---|---|
| May the AI quote prices? | `{{prices_allowed}}` | `NO` or `YES` | Yes | **`NO`** |
| Approved price list | `{{approved_price_list}}` | Used only if prices_allowed = YES. Use the exact wording, e.g. "Diagnostic visit: $89, waived if you do the repair." The AI may repeat these lines word for word and nothing else. No ranges, estimates or "starting at" guesses | If YES | "none" |
| Diagnoses | (LOCKED) | The AI never says what's wrong or suggests a fix. **LOCKED to NO in v1.** | n/a | NO |
| Arrival or callback times | (LOCKED) | The AI never promises when a tech arrives or when someone calls. In BOOKING mode it can confirm only the slot the calendar actually booked. **LOCKED.** | n/a | NO |
| Callback wording | `{{callback_expectation}}` | The exact phrase used when someone wants a call. It must be something the owner will really do. Safe default: "as soon as they can" | Yes | "as soon as they can" |
| Things the AI must never say | `{{never_say}}` | Anything from the owner, e.g. "don't mention financing", "don't name competitors" | Yes | "none listed" |

## F. Emergencies and alerts

| Setting | Placeholder | What it is | Required | Default |
|---|---|---|---|---|
| Emergency safety script | `{{emergency_script}}` | **LOCKED.** The exact text is in `messages.md` (M2). The client cannot edit it | n/a | M2 text |
| On-call name | `{{oncall_name}}` | Who gets emergency alerts first | Yes | owner |
| On-call phone (alerts only) | `{{oncall_phone}}` | Gets emergency text alerts, plus a call if GHL can do it. **Never shared with customers** | Yes | none |
| Owner name | `{{owner_name}}` | | Yes | none |
| Owner alert phone | `{{owner_alert_phone}}` | Gets job alerts, emergency alerts and landline fallbacks | Yes | none |
| Backup alert phone | `{{backup_alert_phone}}` | Gets emergency alerts if on-call hasn't responded within 5 minutes | Strongly recommended | owner |
| Office alert email | `{{office_alert_email}}` | Copy of every job alert | No | none |
| Job alert recipient | `{{job_alert_to}}` | Who gets normal (non-emergency) job alerts: `OWNER`, `OFFICE` (office manager or dispatcher phone) or `BOTH`. The owner doesn't have to get texts | Yes | `OWNER` |
| Office alert phone | `{{office_alert_phone}}` | The office manager's or dispatcher's phone, used when `job_alert_to` is `OFFICE` or `BOTH` | If OFFICE/BOTH | none |
| Job alert channel | `{{job_alert_channel}}` | `TEXT`, `EMAIL`, `APP` (GHL mobile app push, where staff can also reply) or `DIGEST` | Yes | `TEXT` |
| Quiet hours for job alerts | `{{alert_quiet_hours}}` | Non-urgent job alerts are held during these hours and sent as one morning summary. Example: 8pm-7am | No | none |
| Morning summary time | `{{digest_time}}` | When the held alerts (or the DIGEST channel) go out. Example: 7:00am | If quiet hours or DIGEST | 7:00am |
| Public after-hours line | `{{after_hours_line}}` | A number the shop is happy for customers to call after hours, or `NONE` | Yes | `NONE` |
| Does the shop do after-hours service? | `{{after_hours_service}}` | `YES` or `NO`. It changes only how urgent leads are routed (the on-call person gets them at night). It never changes what the customer is told | Yes | `NO` |

**Alert routing rule (LOCKED):** quiet hours, email and digest apply ONLY to normal job alerts. Emergency alerts (M4) and URGENT or AT-RISK PERSON leads always go straight to the on-call phone as a text, at any hour. A shop that wants no night alerts at all must name an on-call person anyway, or it can't be a client. [VERIFY IN GHL: holding alerts until a set time with a wait step, and app push notifications to specific users.]

## G. Messaging and compliance

| Setting | Placeholder | What it is | Required | Default |
|---|---|---|---|---|
| First-text version | `{{first_text_variant}}` | `STANDARD`, or `DISCLOSE` (says "auto-assistant" in the first text). Use DISCLOSE for clients in states with AI or bot disclosure laws (a question for the lawyer, see messages.md) | Yes | `STANDARD` |
| Languages | `{{languages}}` | `English` or `English, Spanish` | Yes | `English, Spanish` |
| Quiet hours | `{{quiet_hours}}` | No follow-up nudges in this window, in the customer's local time. It doesn't apply to direct replies to the customer or to emergency scripts | Yes | 8:00pm-8:00am |
| Morning follow-up time | `{{followup_morning_time}}` | When the next-morning nudge goes out | Yes | 9:00am |
| Repeat-call window | `{{dedupe_window}}` | If the same number called within this many hours, there's no new text-back | Yes | 24 hours |
| A2P 10DLC status | (intake note) | Registration filed? Approved? **No live texting until approved.** This applies to every number, including the demo number. The brand name must match the name used in the texts | Yes | not filed |
| Voicemail greeting installed? | (intake note) | The M21 greeting is recorded or set up on the line callers reach when nobody answers | Yes | no |
| Appointment reminders | `{{reminder_timing}}` | BOOKING mode only: when the M22 reminder goes out | BOOKING only | 5:00pm the day before |
| Call forwarding set up? | (intake note) | Conditional forwarding (busy or no answer) goes to the GHL number | Yes | no |

---

## FILLED EXAMPLE: FICTIONAL DEMO SHOP

> **FICTIONAL. "Returnline Demo Heating & Air" is not a real HVAC business.** We use it only for the sales demo and for safety-qa testing. All phone numbers are 555-01xx fiction numbers. Never load this example into a real client's account, and never let demo alerts go to a real shop.
>
> Why "Returnline" is in the name: the demo number's A2P brand is Returnline. The name shown in the texts has to match the registered brand, or carriers may filter the texts. For real clients, the texts use the client's own name, and the client's A2P brand must match that name.

| Setting | Value (FICTIONAL) |
|---|---|
| `{{business_name}}` | Returnline Demo Heating & Air |
| `{{business_short_name}}` | Returnline Demo Heating & Air (29 characters, under the 35 cap) |
| `{{business_city_state}}` | Conway, AR |
| `{{business_phone}}` | (501) 555-0100 |
| `{{business_hours}}` | Mon-Fri 7:30am-5:30pm, Sat 8am-12pm, closed Sunday |
| `{{timezone}}` | America/Chicago |
| `{{fsm_software}}` | Google Calendar (demo) |
| `{{service_area}}` | Conway, Greenbrier, Vilonia, Mayflower, Wooster and Guy (Faulkner County, AR) |
| `{{service_area_zips}}` | 72032, 72034, 72058, 72173, 72106, 72181, 72061 (these are real ZIPs; confirm with USPS before reusing them for a real client) |
| `{{out_of_area_policy}}` | PASS_TO_OFFICE |
| `{{services_offered}}` | AC repair, furnace repair, heat pump repair, seasonal tune-ups, new system estimates, thermostat installs, ductwork |
| `{{services_not_offered}}` | plumbing, water heaters, window units, commercial rooftop units |
| `{{equipment_notes}}` | none |
| `{{mode}}` | **BOOKING** for the sales demo. (A real client would start on REQUEST.) |
| `{{time_windows}}` | morning (8am-12pm) or afternoon (12-4pm) |
| `{{booking_calendar}}` | "Demo Service Windows" (GHL calendar with 4-hour slots, fake availability) |
| `{{slots_to_offer}}` | 3 |
| Calendar kept current? | Yes (demo calendar maintained by Returnline) |
| `{{prices_allowed}}` | NO |
| `{{approved_price_list}}` | none |
| `{{callback_expectation}}` | as soon as they can |
| `{{never_say}}` | Don't mention financing. Don't name other HVAC companies. |
| `{{oncall_name}}` | Jordan (demo stand-in) |
| `{{oncall_phone}}` | (501) 555-0101 |
| `{{owner_name}}` | Pat Demo (fictional) |
| `{{owner_alert_phone}}` | (501) 555-0102. **For live demos, route this to the founder's own phone so no real shop gets alerts.** |
| `{{backup_alert_phone}}` | (501) 555-0103 |
| `{{office_alert_email}}` | demo-office@example.com |
| `{{after_hours_line}}` | NONE |
| `{{after_hours_service}}` | YES (on-call for no-heat/no-cool) |
| `{{first_text_variant}}` | STANDARD |
| `{{languages}}` | English, Spanish |
| `{{quiet_hours}}` | 8:00pm-8:00am Central |
| `{{followup_morning_time}}` | 9:00am Central |
| `{{dedupe_window}}` | 24 hours |
| A2P 10DLC status | **Required for the demo too.** Carriers filter texts from unregistered numbers, so the demo number must be A2P-registered under the Returnline brand, with a campaign for missed-call follow-up and appointment texts, before any demo. Status: not filed. The founder files it in GHL. No live demo until it's approved |
| Call forwarding | n/a for the demo (we call the demo number directly and don't answer) |
| Voicemail greeting | M21, set up on the demo number |
| `{{reminder_timing}}` | 5:00pm Central the day before |

### How a real client's sheet differs from the demo

- `{{mode}}` = REQUEST unless the owner confirms the calendar is kept current every day.
- Alert phones are the shop's real on-call and owner numbers. Before go-live, test that each one receives a text.
- The A2P status must say "approved" before go-live.
