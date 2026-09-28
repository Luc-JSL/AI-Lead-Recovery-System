# AI Lead Recovery for HVAC: Business Blueprint v1

This is the company's working memory. Claude doesn't remember anything between sessions, so decisions, numbers and lessons get written down here.

---

## 1. What we sell (in one sentence)

> "Every call your shop misses gets a text back in under 60 seconds. An AI qualifies the customer, books the job, and each month we show you exactly how many dollars we recovered."

"Lead recovery" means catching revenue the HVAC company is already losing:

| Leak | Why it happens | What we do |
|---|---|---|
| **Missed calls** (the biggest one) | Techs are on a roof, office is closed, it's a heat wave | Instant text-back, then an AI conversation, then a booking |
| **Slow web-form response** | Nobody checks the inbox for hours | Reply in seconds, qualify, book |
| **Unsold estimates** | A tech quoted a $12k system and nobody followed up | Automated follow-up sequence |
| **Dormant past customers** | No maintenance reminders | Seasonal tune-up campaigns (**consent required**) |

---

## 2. Architecture (beginner version)

Think of it as 6 boxes:

```
   CUSTOMER                                  HVAC COMPANY
      |                                            ^
      | calls / texts / fills form                 | "New booked job!" alert
      v                                            |
 +-------------+   webhook   +------------------+  |   +----------------------+
 | 1. PHONE &  | ----------> | 2. OUR SERVER    | -+-> | 5. THEIR CALENDAR /  |
 |  SMS LAYER  | <---------- | (the "brain      |      |  FIELD SERVICE APP   |
 |  (Twilio)   |   replies   |   stem")         |      | (Google Cal, Jobber, |
 +-------------+             +------------------+      |  Housecall Pro, etc) |
                               |      ^     |          +----------------------+
                     "what do  |      |     | save everything
                     I say?"   v      |     v
                          +-----------+  +--------------+     +---------------+
                          | 3. AI     |  | 4. DATABASE  | --> | 6. DASHBOARD  |
                          | (Claude   |  | (Postgres /  |     | "$ recovered  |
                          |  API)     |  |  Supabase)   |     |  this month"  |
                          +-----------+  +--------------+     +---------------+
```

1. **Phone & SMS layer (Twilio or Telnyx).** We give each HVAC client a number, or forward their "no answer" calls to us. When a call is missed, Twilio pings our server.
2. **Our server.** A small web app (Python/FastAPI or Node) that receives those pings ("webhooks"), decides what happens next, and sends texts.
3. **AI (Claude API).** It writes the conversation: asks what's wrong, how urgent it is, the address, and a good time. It follows strict rules and must never diagnose anything dangerous.
4. **Database.** Every lead, message, status and booked job. This is our proof of value.
5. **Calendar / field-service software.** Where the booked job lands so the office actually sees it.
6. **Dashboard.** Missed calls, recovered leads, booked jobs, estimated revenue. **This is what keeps clients paying.**

Also needed: **Stripe** (billing), **A2P 10DLC registration** (US carriers require it before you can text at scale; it takes days to weeks), error monitoring, backups.

### Safety rules that are NOT optional
- The words gas smell, carbon monoxide, CO alarm, burning smell or sparks trigger an immediate scripted reply: *leave the house, call 911 / the gas utility*. Then a human gets alerted. The AI never troubleshoots these.
- The AI never quotes prices unless the client explicitly approves a price list.
- Every conversation has a "talk to a human" escape hatch.
- STOP / opt-out is honored instantly.

---

## 3. Pipelines

### 3a. Lead pipeline (what happens to one customer)
```
Missed call -> Text-back (<60s) -> AI qualifies (issue, urgency, address, time)
 -> Emergency? --yes--> safety script + page the on-call tech
 -> Book appointment -> Notify office -> Reminder text -> Job done
 -> Review request -> (later) maintenance reminder, with consent
```

### 3b. Build & business pipeline (what WE go through)

| Phase | Goal | Exit criteria |
|---|---|---|
| **0. Validate** (wk 1-2) | Talk to 20+ HVAC owners/office managers | 3 shops agree to a pilot, ideally paid |
| **1. Concierge MVP** (wk 2-5) | Missed-call text-back + AI qualifying + owner alert, for 1-3 shops | Real leads recovered, with numbers |
| **2. Booking + follow-ups** | Calendar booking, estimate follow-up | Pilot converts to paid; a case study exists |
| **3. Productize** | Multi-client, dashboard, Stripe, self-serve onboarding | 10 paying clients |
| **4. Expand** | After-hours AI voice answering, FSM integrations | Retention above 90%/mo, clear upsell |
| **5. Scale sales** | Repeatable outbound + referrals | Predictable CAC & payback |

---

## 4. Unit economics (ASSUMPTIONS, to be verified with real data)

| Item | Rough guess |
|---|---|
| Price | $300-$800/mo + $500-$1,500 setup |
| Our cost per client (SMS, number, AI, hosting) | ~$20-$100/mo depending on volume |
| Gross margin | ~80%+ |
| Value to client | One recovered system replacement ($8k-$15k+) pays for a year or more |

Before go-live with every client, record a **baseline**: missed calls per week, from their phone logs. Without a baseline we can't prove ROI.

---

## 5. Open questions for the founder
- Monthly budget for tools and ads?
- Hours per week available?
- Technical comfort (never coded / some / comfortable)?
- Do you personally know any HVAC owners? Which city/region?
- Goal: side income, agency, or venture-scale SaaS?

---

## 6. Decisions log

**2026-09-28**
- **Goal:** a profitable agency that replaces the founder's job income so they can get through school. Not venture-scale SaaS.
- **Base:** Conway, AR. First pilots in Central Arkansas (Conway, Little Rock metro). After 1-2 case studies, sell remotely into hot-summer states (TX, OK, TN, LA, MS, AZ, FL), then nationwide. Stay HVAC-only until that's working.
- **Time:** 15 hrs/week (about 2 hrs on weeknights plus 5 on Saturday). At least half goes to sales.
- **Tech:** Claude owns the code. First 1-3 clients run on an off-the-shelf platform so revenue starts in weeks. Our own system gets built alongside and replaces it once it's proven.
- **Lean budget:** about $150-250/mo, plus about $100-300 one-time (see section 7).

## 7. Budget (estimates; verify prices before buying)

| Item | Cost | Why |
|---|---|---|
| Arkansas LLC | ~$45 filing + ~$150/yr franchise tax | Protects personal assets; makes clients take us seriously |
| Domain + Google Workspace email | ~$12/yr + ~$7/mo | A professional email address for outreach |
| GoHighLevel (starter plan) | ~$97/mo | Missed-call text-back and CRM on day one; dropped later |
| Twilio number(s) + SMS + carrier registration | a few $/mo + small one-time fees | The texting itself (passed through to clients) |
| Claude API | ~$5-30/mo early on | The AI conversations |
| Hosting / database (our build) | $0-25/mo | Free tiers cover us at first |
| Cold email tool (after local pilots) | ~$40/mo | Nationwide outreach |

**2026-09-28 (update)**
- **Founder capacity:** works 29-30 hrs/week, plus school 2-3 hrs/day. Business gets about 12-15 hrs/week.
- **Sales channel:** walk-ins first (a founder strength). Cold calling is dropped as the main channel; the after-hours "no answer" test calls stay, since they need no pitch.
- **Pricing:** standard $497/mo + $497 setup. The first 3 clients get a $397/mo founding rate locked for 12 months, in exchange for a case study and testimonial.
- **Goals:** quit the job at $2,000/mo take-home held for 3 months (6 clients). Then build a Corvette fund (C6, price TBD) before age 22, which needs about 11-12 clients.
- **Weekly review:** automated every Sunday at 6:47pm Central.

**2026-09-28 (names & schedule)**
- **Company name: Returnline.** Before buying, check the domain, the Arkansas Secretary of State and USPTO.
- **Car goal: 2018/2019 Corvette Z06.** That's a C7, not a C6. It will be financed: a strong down payment plus a monthly loan payment.
- **Founder's weekly schedule (Central time):**

| Day | Free for Returnline | Plan |
|---|---|---|
| Mon | after 11:50 | Walk-ins 1:00-4:00, after-hours test calls 7:00-8:00pm |
| Tue | 1:00-4:00 (work 4-11pm) | Follow-ups, email, pilot setup |
| Wed | 11:50-3:00 (lab 3pm) | Walk-ins 12:15-2:30 |
| Thu | 1:00-4:00 (work 4-11pm) | Follow-ups, client check-ins |
| Fri | 11:50-4:00 (work 4-11pm) | Walk-ins 12:15-3:30 |
| Sat | before 10am, after 5pm (work 10-5) | Rest / optional after-hours test calls |
| Sun | after 5pm (work 10-5) | Weekly review with Claude at 6:47pm |

**2026-09-28 (goals raised)**
- **$2,000/mo is the first milestone, not the ceiling.** The milestone ladder (estimates at $497/client, after tool costs and 25% set aside for taxes):

| Milestone | Clients | MRR | Est. take-home |
|---|---|---|---|
| Quit the job | 6 | ~$2,980 | ~$2,000/mo |
| Z06 fund | 12 | ~$5,960 | ~$4,200/mo |
| Full-time agency | 25 | ~$12,400 | ~$8,800/mo |
| Hire a team | 50 | ~$24,900 | ~$17,000/mo before hiring costs |

Past 25 clients, one founder can't serve everyone alone. That's when we hire (SOP manager first, then a human customer-success person) and add upsells such as AI after-hours call answering.
- **Business email:** jordan@ on the Returnline domain for outreach, plus hello@ as the public contact on the one-pager and website.

**2026-09-28 (pricing locked by founder)**
- **Founding clients (first 3):** $397/mo locked for 12 months, no setup fee, in exchange for a case study and testimonial.
- **Clients 4-10:** $497/mo, no setup fee (launch offer). **Later:** $497/mo + $297 setup, once we have case studies.
- **Guarantee:** if we don't recover at least one real job (a booked appointment from a new-customer lead that came through the system) in the first 30 days of paid service (the clock starts after the free trial ends), that month is free. The shop confirms booked jobs weekly.
- **Phase 1 scheduling is "request mode":** the AI collects the job details and a preferred window, and the office confirms the time. Live calendar booking comes later, for shops whose calendars are kept up to date.
- **Milestone ladder recalculated** (3 founding clients at $397, the rest at $497; after tool costs and 25% for taxes): quit the job = **7 clients** (~$2,150/mo take-home), Z06 fund = 12 (~$3,950), full-time agency = 25 (~$8,600), hire a team = 50 (~$17,500).

**2026-09-28 (master operating system adopted)**
- Founder's standing brief saved as `CLAUDE.md`. Every session loads it.
- **Product name:** "AI Revenue Recovery System" by Returnline.
- **Phase 1 is now a working sales demo in GoHighLevel** (no n8n, no custom code). It runs in parallel with prospecting; prospecting doesn't wait for the demo.
- **Demo vs. production:** the demo uses booking mode on a demo calendar to show the full flow. Real clients default to office-confirmed request mode until their calendar is proven reliable.
- Added `docs/finance.md` (P&L and tool justification), `docs/skills-registry.md`, the `daily-brief` skill, and funnel plus profit columns in the metrics.

**2026-09-28 (channels)**
- The founder asked for cold calling. Main channels are now: **(1) daytime calls to book a 10-minute visit, (2) audit-first email, (3) warm drop-offs**, all using the after-hours test result as the hook. Cold walk-ins become optional drop-offs, not pitches. Network asks (family and friends who know shop owners) run alongside.

**2026-09-28 (founder decisions)**
- **Normal texting and AI usage are included** in the monthly price. Watch it per client in `docs/finance.md`; revisit if one shop's usage gets unusual.
- **Support promise:** "Text me anytime; I reply within a few hours, 8am-9pm." Emergencies route to the shop's on-call person, not the founder.
- **LLC deferred** by the founder. It blocks texting registration, so it must be filed as soon as a shop agrees to a trial.
