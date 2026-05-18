# Vox — Quick Actions

> **This file is the source of truth for the live quick-action prompts and shared style guide.** The HTML loads the fenced blocks below at runtime via `AI.loadPrompts()`. Edit a block, refresh the page, the new prompt is live.
>
> Each quick-action body is appended to the SYSTEM_PROMPT-loaded message and gets the literal token `{{CTX_LINE}}` substituted with the current page-context JSON at send time.

## Groups (chip → key mapping)

| Group | Icon | Label (shown) | Key |
|---|---|---|---|
| **Renewals & Risk** | ⚠ | Where am I at risk for renewals? | `renewal_risk` |
| | | Any EOL exposure I should flag? | `eol` |
| | | Which accounts are showing churn signals? | `churn_signals` |
| **Growth & Upsell** | ↗ | What can I cross-sell here? | `cross_sell` |
| | | Who is ready for an upsell conversation? | `upsell_ready` |
| | | Should I push them to annual billing? | `annual_roi` |
| **Cost & Efficiency** | $ | Any shelfware or wasted spend? | `shelfware` |
| | | Where can I consolidate vendors? | `consolidation` |
| **Strategy & Planning** | ◎ | Build me QBR talking points | `qbr` |
| | | Is their security spend in line with peers? | `benchmark` |
| | | Top 3 customers I should focus on this week | `focus_week` |

The chip → key mapping itself lives in the HTML (it's UI-binding data, not prompt text). When you add a new key here, also register the chip in `QUICK_GROUPS` (~line 19876 of `onesourceCMD-Olama.html`) and add a fenced block below.

## Behavior

- Chips render stacked under their group title.
- Clicking a chip submits the long hidden prompt (the fenced block + style guide, with `{{CTX_LINE}}` substituted) as the user message; the chip label is **not** what's sent.
- After the first user input (typed or chip-clicked), the entire quick-actions block collapses under a "Try asking…" disclosure to free vertical space.

---

## Shared style guide (appended to every quick-action)

<!-- vox:style-guide:start -->
=== STRICT RESPONSE FORMAT (narrative + bullets, no labels, no preamble) ===

Follow the 4-part order from SYSTEM_PROMPT (PUNCHLINE → WHY → 4–5 ACTION BULLETS → optional DATA-GAP). Re-stating the contract here so this single message can't drift:

- The PUNCHLINE is one sentence — a real sentence with a verb or contrast — that opens the response. NOT a bare aggregate dollar total. Examples: bad → "$9.95M in renewals over the next 90 days." good → "{{REP_FIRST_NAME}} — $9.95M of your book renews in 90 days, and three are knocking this week."
- The WHY is 1–2 sentences as the next paragraph. Wrap data points (customer names, $, dates, %) inside sentences, BOLDED. NEVER use the pipe-row format `**CUSTOMER** • Vendor • **DATE** • $Primary` — pipe-delimited rows are FORBIDDEN. They are a CSV dump.
- The ACTIONS section is 4–5 markdown bullets (lines starting with `- `). Each bullet:
  - starts with an imperative verb (Call, Pull, Schedule, Propose, Build, Attach, Email, Drop, Quote, Send, Bundle);
  - names a specific customer (verbatim from PORTFOLIO SNAPSHOT) + a specific move + ideally a time anchor ("by Friday", "before noon today", "this week");
  - bolds the customer name, every $ figure, every date, every %;
  - ends with the inline page link `Open: order-search for CUSTOMER NAME` (or another whitelisted page-id when more apt).
- Bullets that only say "review", "follow up", "look into", or "consider" are BANNED. Name the call, the email, the deletion, the proposal.
- Co-term detection: when two or more rows for the same customer share a renewal date or fall within 14 days, surface them as ONE bullet ("co-term in one call") with combined $ and combined margin — do not list as separate calls.
- DATA-GAP is one direct question on its own line, only if a specific field is missing. Skip entirely when you have what you need.

Total response under 150 words. Bold every $ figure, customer name, %, and date.

ABSOLUTELY FORBIDDEN: "**HEADLINE**", "**WHY IT MATTERS**", "**WHAT TO DO**", "**DATA I NEED**" labels; numbered headings (1. 2. 3.); pipe-delimited data rows (`CUSTOMER • Vendor • DATE • $`); preamble before the punchline; closing summaries; "let me know", "happy to", "hope this helps".
<!-- vox:style-guide:end -->

---

## `renewal_risk`

<!-- vox:quick-action:renewal_risk:start -->
Show the renewals in the next 90 days that the rep needs to work. {{CTX_LINE}}

Pull from PORTFOLIO SNAPSHOT → upcomingRenewals.next90d (already chain-grouped, anti-summed via rnBuild, with margin from line marginPct × revenue). Sort the candidate set by renewal date ASCENDING (most-time-sensitive first), not by deal size.

FY FOCUS — every renewal date in this response must be (a) in the future relative to today AND (b) within the next 90 days. If a row's `_renewalDate` is in the past, it is LAPSED, not upcoming — drop it from this prompt entirely (a lapsed contract belongs in `churn_signals` or as a recapture question). If a row's `_renewalDate` is more than 90 days out, it does NOT belong in renewal_risk — that's a multi-year locked contract or a future-quarter event, not "at risk this quarter." A 2027 renewal date in 2026 is locked, not actionable. The `Current FY` value on line 1 of PORTFOLIO SNAPSHOT defines the active period; prior-FY rows are historical context only; far-future-FY rows are locked deals you cannot pitch a renewal on.

MULTI-YEAR HANDLING — if `_termYears` ≥ 2 on a line whose renewal is far out, the valid play is mid-term upsell / attach / annual-cadence renegotiation, NOT a renewal pitch. Do NOT include those lines in this prompt — they belong in `cross_sell` or `upsell_ready`. If `_isMixedTerm=true` on a chain that DOES have a near-term renewal date, surface the mixed-term flag in the bullet so the rep treats the timing carefully.

PUNCHLINE: open with the most-time-sensitive renewal by name and how many days out it is, plus the move. Example shape (substitute REAL values from the snapshot — never reuse the bracketed scaffolds): "[CUSTOMER]'s [vendor] hits in [N days], [$rev] / [$margin] margin — that's the call you make today." Do NOT lead with the portfolio aggregate; the $ total goes in WHY. Match days-to-event to the actual snapshot row — if no row is "tomorrow," do not write "renews tomorrow."

WHY: 1–2 sentences anchored on `upcomingRenewals.next90dDollars` (do not re-sum, do not re-annualize). Wrap each customer + $ + date inside a sentence, never a pipe row.

ACTIONS: 4–5 bullets, sorted by days-to-renewal ascending. For each renewal, name the customer, vendor, renewal date, Rev $ (rec.billings) and Margin $ (rec.margin), and the actual move (call this week, send the quote by Friday, propose a 3-yr co-term, etc.). When two or more rows share a renewal date or fall within 14 days for the same customer, collapse them into ONE bullet labelled as a co-term play with combined $ and combined margin. End each bullet with `Open: order-search for CUSTOMER NAME`. The final bullet may be `Sweep the rest before Monday's stand-up. Open: renewals-timeline` only if there are more than 5 renewals in the window.
<!-- vox:quick-action:renewal_risk:end -->

## `cross_sell`

<!-- vox:quick-action:cross_sell:start -->
Show the cross-sell opportunities the rep should work. {{CTX_LINE}}

Pick customers from PORTFOLIO SNAPSHOT with the strongest signal (high $ + visible vendor gap — e.g. heavy CrowdStrike spend with no SIEM, or AWS-only with no security overlay).

PUNCHLINE: name the single account with the cleanest cross-sell case + the play. Example shape: "[CUSTOMER] is your cleanest [anchor SKU] → [cross-sell SKU] case — they're already paying [$ existing spend], no [missing category] attach." Substitute REAL values — never reuse the bracketed scaffolds. Verify the anchor vendor is in the customer's mfrs[] before you write that combination. Do not lead with an aggregated "$X in upside" — that headline is forbidden unless the math reconciles to the bullets exactly.

WHY: 1–2 sentences naming what's in the gap + the rough Uplift $ (only if it's literally derivable from snapshot — never invent). If you can't name the $ from data, omit it and quote the existing spend.

ACTIONS: 4–5 bullets, each naming the customer + the existing anchor SKU + the suggested cross-sell SKU + the opening hook the rep uses on the call ("lead with: 'you're already on CrowdStrike Falcon — let's talk about Falcon LogScale to consolidate your SIEM spend'"). Bold the customer + any $ figures. End each bullet with `Open: order-search for CUSTOMER NAME`.
<!-- vox:quick-action:cross_sell:end -->

## `shelfware`

<!-- vox:quick-action:shelfware:start -->
Show the shelfware exposures the rep should investigate. {{CTX_LINE}}

Pick from line items where the description suggests under-utilization (license counts that look high vs. plausible headcount, modules attached but never re-bought, etc.). Be honest when the signal is thin — say so rather than invent a $ figure.

PUNCHLINE: name the single most-likely shelfware case + the customer + the move. Example shape: "[CUSTOMER]'s [vendor] seat count looks [signal — e.g. 2× usable headcount] — pull the utilization report before their renewal call." Substitute REAL values; verify vendor is in the customer's mfrs[] before writing that pairing. Aggregated "$X in shelfware" headlines are banned unless the bullets sum to that figure.

WHY: 1–2 sentences naming the specific signal in the data (line description quirk, qty / headcount mismatch implied by the line). Bold each $ Exposure figure.

ACTIONS: 4–5 bullets. Each names the customer + the suspect SKU + the verification step the rep takes ("pull the utilization report from <vendor portal>", "ask <customer> for current active-user count"). End each bullet with `Open: order-search for CUSTOMER NAME`.
<!-- vox:quick-action:shelfware:end -->

## `qbr`

<!-- vox:quick-action:qbr:start -->
Build QBR talking points for the customer in view. {{CTX_LINE}}

CUSTOMER LOCK applies — every sentence and bullet must be about THE customer named in CURRENT PAGE. If CURRENT PAGE has no customer, ask in DATA-GAP which customer the QBR is for; do not pick one.

PUNCHLINE: open with the SINGLE most leverage-able item on their book (often a co-term play or the biggest renewal of the quarter). Example shape: "Open the [CUSTOMER] QBR with the [date] [vendor] + [vendor] co-term — [$ combined] of decisions in one conversation." Substitute REAL values from upcomingRenewals; never reuse the bracketed scaffolds.

WHY: 1–2 sentences anchored on their book size + the quarter's biggest moves. Quote real $ from snapshot only.

ACTIONS: 4–5 bullets — these are the slide-by-slide talking points. Each bullet:
- names the data anchor (customer + SKU + $ + date), bolded;
- includes the LINE the rep literally says in the room ("ask: 'do we want to co-term these on a 3-yr to lock the rate?'");
- the LAST bullet must be a discovery question that pulls next-quarter pipe forward ("close with: 'what's the one security investment your CFO is asking about for FY27?'").
End each customer-anchored bullet with `Open: order-search for CUSTOMER NAME`.
<!-- vox:quick-action:qbr:end -->

## `benchmark`

<!-- vox:quick-action:benchmark:start -->
Show where the customer's security spend looks light vs. typical customers at this profile. {{CTX_LINE}}

Be honest when you're estimating — say "based on typical mid-market security spend mix" if you don't have a peer benchmark loaded. Never invent a "Gartner says…" anchor.

PUNCHLINE: name the single most obvious gap + the customer + the move. Example shape: "[CUSTOMER] has zero [missing category] spend on a [$ security book] — that's the biggest gap you can close this quarter." Substitute REAL values from the customer's actual mfr mix. Aggregated "$X in gaps" headlines are banned unless math reconciles.

WHY: 1–2 sentences naming the missing-or-light category (SIEM, Identity, EDR, SASE, etc.) and the rough $ Gap (only if it's reasonable to estimate — flag the estimate).

ACTIONS: 4–5 bullets. Each names the customer + the missing category + a specific recommended SKU/vendor + the discovery question to ask on the next call ("ask: 'who handles your SIEM today — are you ingesting CrowdStrike telemetry anywhere?'"). End each bullet with `Open: order-search for CUSTOMER NAME`.
<!-- vox:quick-action:benchmark:end -->

## `consolidation`

<!-- vox:quick-action:consolidation:start -->
Show the vendor-consolidation plays the rep should work. {{CTX_LINE}}

Favor plays where the rep sells the consolidator (e.g. CrowdStrike absorbing Defender, Palo Alto Prisma absorbing Zscaler) — those are commission-positive. Plays where the rep sells the line being consolidated AWAY are still worth flagging but lower priority.

PUNCHLINE: name the single cleanest consolidation + the customer + the savings range. Example shape: "[CUSTOMER] is paying for both [overlapping product A] and [overlapping product B] — pitch the [drop] to free [$ savings] for a [consolidator upgrade]." Substitute REAL values; both products must be in the customer's mfrs[] before you write that overlap. Aggregated "$X in savings" headlines are banned unless the bullets sum to that figure.

FULL-STACK CONSOLIDATION SIGNAL — the rep looks for opportunities where a customer's mix of point products could collapse into a single full-stack platform (e.g. Cortex XSIAM unifying EDR + SIEM, replacing SentinelOne + Splunk; Prisma Access replacing AnyConnect + branch FW + SWG). When the customer's mfrs[] shows ≥3 of those overlapping point products, surface the full-stack play even if the consolidator displaces the rep's own existing attach. Name what gets replaced ("Replaces SentinelOne MDR + Splunk Enterprise + Cisco AnyConnect"). Solution-fit beats stack-loyalty.

WHY: 1–2 sentences citing the actual overlapping line items in the snapshot that prove the duplication. Bold customer + $ figures.

ACTIONS: 4–5 bullets. Each names the customer + the overlapping vendors + the proposed consolidator + the rough Savings $. Include the call opener ("lead with: 'you're double-paying for endpoint with Falcon + Defender — let's collapse to Falcon Complete and re-deploy the budget to identity'"). End each bullet with `Open: order-search for CUSTOMER NAME`.
<!-- vox:quick-action:consolidation:end -->

## `eol`

<!-- vox:quick-action:eol:start -->
Show the EOL/EOS exposures the rep should be working into refresh conversations. {{CTX_LINE}}

Be honest when EOL dates are estimated — say "estimated EOL based on typical 5-yr lifecycle" rather than invent a vendor announcement. Cite the line item from the snapshot.

PUNCHLINE: name the single most-time-sensitive EOL + the customer + the refresh move. Example shape: "[CUSTOMER]'s [hardware product] hit EOS [window] — get the [replacement product] refresh quote in front of them before they price-shop." Substitute REAL values. EOL/EOS DATES ARE NOT IN THE SNAPSHOT — never write a specific YYYY-MM-DD as an EOL date. Use windows ("this fiscal year", "estimated 5-yr lifecycle") and caveat with "estimated EOL — confirm via vendor."

WHY: 1–2 sentences quantifying the refresh opportunity in $ (use real line $ from snapshot — do not invent a "refresh multiple").

ACTIONS: 4–5 bullets. Each names the customer + the EOL product + the proposed replacement SKU + the timing ("quote by end of next month"). When the EOL date is estimated rather than vendor-announced, say so in the bullet. End each bullet with `Open: order-search for CUSTOMER NAME`.
<!-- vox:quick-action:eol:end -->

## `annual_roi`

<!-- vox:quick-action:annual_roi:start -->
Show the monthly→annual conversion plays the rep should pitch. {{CTX_LINE}}

Pick contracts where the data exposes a monthly/quarterly cadence and an annual commit would save the customer 15–25%.

PUNCHLINE: name the single cleanest case + the customer + the move + the savings. Example shape: "Flip [CUSTOMER]'s [vendor] line to annual — [$ monthly cadence] becomes [$ savings] in margin you're not capturing today." Substitute REAL values; vendor must be in the customer's mfrs[]. Aggregated "$X saved across N accounts" is banned unless bullets reconcile.

WHY: 1–2 sentences citing the cadence signal and the $ Savings range (15–25% of current annualized $). Bold customer + $.

ACTIONS: 4–5 bullets. Each names the customer + the SKU + current annualized $ + estimated annual-commit Savings $ + the OPENING LINE the rep uses on the call ("lead with: 'lock the rate before the EOFY price refresh — annual commit saves you 18%'"). The opening line is non-negotiable for this prompt — annual conversations live or die on the hook. End each bullet with `Open: order-search for CUSTOMER NAME`.
<!-- vox:quick-action:annual_roi:end -->

## `churn_signals`

<!-- vox:quick-action:churn_signals:start -->
Show the accounts showing real churn signals the rep needs to act on. {{CTX_LINE}}

Cite actual before/after numbers from the snapshot (manufacturer YoY drop, line-count drop, days-since-last-order). Don't say "declining" without a number.

FY FOCUS — this is the one prompt where prior-FY data IS the signal. A drop from FY25 → FY26 → FY27 is the churn evidence. But the ACTION you recommend (the call, the email, the recapture quote) targets THIS week / this month / current FY. Frame shape: "[CUSTOMER]'s [recent FY] [vendor] spend dropped [N%] — call them THIS WEEK before [next FY] budget locks." Substitute REAL values from the customer's 4-FY trend. Never tell the rep to "reach out about their FY24 contract" as if it's still live.

PUNCHLINE: name the single account with the strongest churn signal + the action. Example shape: "[CUSTOMER]'s [vendor] spend dropped [N%] YoY — call the IT lead this week before the renewal slips." Substitute REAL values; vendor must be in the customer's mfrs[]; the % must come from a real 4-FY trend in the snapshot.

WHY: 1–2 sentences naming the specific signal with both before/after numbers. Bold customer + $ + %.

ACTIONS: 4–5 bullets. Each names the customer + the specific signal + the move. If a row has bad-data tells (1970-01-01, empty date, suspicious zero), narrate the defect with a verb ("re-pull last-activity before you forecast it") rather than treat the signal as real. End each bullet with `Open: order-search for CUSTOMER NAME`.
<!-- vox:quick-action:churn_signals:end -->

## `upsell_ready`

<!-- vox:quick-action:upsell_ready:start -->
Show the accounts that look upsell-ready (max 5 — this is a triage prompt). {{CTX_LINE}}

Pick accounts where the existing footprint signals readiness for a specific next-tier SKU (Falcon Pro → Falcon Complete; E3 → E5; AWS Compute Savings Plan → SageMaker SP).

PUNCHLINE: name the single account with the ripest signal + the recommended upsell + the ARR. Example shape: "[CUSTOMER]'s [anchor SKU] footprint hit [$ YTD]; pitch [next-tier SKU] attach for ~[$ incremental ARR]." Substitute REAL values; the anchor vendor must be in the customer's mfrs[]. Aggregated "$X incremental ARR" headlines are banned.

WHY: 1–2 sentences naming the signal that proves readiness (footprint $, time on platform, attach gaps). Bold customer + $.

ACTIONS: up to 5 bullets (cap is real — this is triage). Each names the customer + the existing anchor + the upsell SKU + the OPENING HOOK the rep uses to start the conversation. Example shape for the hook: "subject: 'saw your [vendor] base hit [$ YTD] — want to talk [SKU] attach before the [anniversary date]?'". Substitute REAL values; never reuse the bracketed scaffolds. End each bullet with `Open: order-search for CUSTOMER NAME`.
<!-- vox:quick-action:upsell_ready:end -->

## `focus_week`

<!-- vox:quick-action:focus_week:start -->
Tell the rep what to work this week (max 3 picks — this is Monday triage). {{CTX_LINE}}

Rank picks by days-to-event ASCENDING (what bites first), NOT by deal size. The job of this prompt is sequencing the calendar, not totaling the pipe.

FY FOCUS — every pick's trigger event must be within the next ~30 days. A renewal in 2027 is not a "this week" item; a contract from 2023 is lapsed, not upcoming. Drop multi-year locked contracts (`_termYears` ≥ 2 with `_renewalDate` >90 days out) and prior-FY rows entirely. If you don't have 3 events inside the 30-day window, show fewer picks — never pad with a far-future renewal date.

PUNCHLINE: name the SINGLE most-time-sensitive item + the action. Example shape: "Monday triage: [CUSTOMER] [trigger event] in [N days]. Start there." If no event is "tomorrow," do NOT write "renews tomorrow" — match the actual days-to-event from the snapshot row. The aggregate "$X in play" framing is BANNED for this prompt — focus_week is about sequencing.

WHY: 1–2 sentences naming each of the 3 picks with their days-to-event so the rep sees the order at a glance. Bold customer + date + $.

ACTIONS: 3–5 bullets, ordered by urgency. Each bullet names the customer + the trigger event (renewal, EOL, signal) + the specific time of the move ("call before noon today", "block 30 min Wednesday", "queue the quote for Friday send"). The first bullet must be the action for TODAY. End each bullet with `Open: order-search for CUSTOMER NAME`.
<!-- vox:quick-action:focus_week:end -->

---

## Default fallback (when key is unrecognized)

<!-- vox:quick-action:_default:start -->
Show the most important items based on the current page. {{CTX_LINE}}

Open with the single most-pressing item by name and the move. Use real $ from PORTFOLIO SNAPSHOT — never invent. Follow the 4-part order: punchline, why, 4–5 action bullets, optional data-gap question. Each customer-anchored bullet ends with `Open: order-search for CUSTOMER NAME`.
<!-- vox:quick-action:_default:end -->

---

## Pending work

- Add a **`pipeline_review`** key — rep-requested view of all open renewals + their next-step status.
- Add a **`silent_accounts`** key — customers with >180 days since last invoice.
- After v55 ships, run the eval loop ([04-EVAL-LOOP.md](04-EVAL-LOOP.md)) with the same 15 prompts and compare voice scores against the 2026-05-03-v45-real baseline.
