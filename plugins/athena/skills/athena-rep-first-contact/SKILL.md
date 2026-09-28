---
name: athena-rep-first-contact
description: The first time a rep connects, show them what Athena has set up for them — read their own context back, then propose a few saved views tailored to their patch, each created with ONE confirmation and owned by them. Use when someone asks what's set up for them, how to get started, what to look at first, "what have you got for me", or when a rep is new to Athena.
named-terms:
  briefing: Radar Briefing
  pickup-phrase: What's next on my radar
  note: Placeholders. Written here once so a rename is one edit.
---

# Rep first-contact

The first session a rep has with Athena. Someone else — their admin, or Athena — has already recorded
who this rep covers; this skill reads that back, checks it still fits, and turns it into a small set of
**saved views the rep owns**, so from the next session on their standing questions re-run with one click.

This is NOT the set-up skill. Set-up is the admin configuring the company and seeding each rep. This is
the rep arriving to what was seeded, and it never writes the Company context or the Messaging playbook
— only the rep's own views (and, if they ask, their own context).

## Before you start

**Check you are live by CALLING a tool, not by looking for one.** On some platforms connector tools are
listed but not loaded, so a missing tool in a list proves nothing. Call `athena_orient`. If it returns,
you are connected; if it does not, say so and stop — do not improvise a briefing from memory.

`athena_orient` also tells you which company you are scoped to, whether this session can save
(`writes_available.views`), and the vocabulary and safety rules below. Two rules matter here because a
first session is where a rep's whole mental model gets set:

- **Scores have three states, and one number.** `lead_score_standardized` is the only lead score you
  will see or say. `lead_score_standardized` absent means not scored on this client's standardised
  scale. `lead_score_standardized` 0 with a tier is a REAL score, at the bottom of the ranking.
  `lead_score_standardized` 0 with `lead_score_tier` "N/A" means one of this client's own scoring
  rules ruled the person out: a value they marked as unwanted, with nothing scored to outweigh it.
  Say "ruled out by your scoring rules", never "unscored", and keep them out of priority lists. A
  ruled-out person's score is 0 too, so the TIER is the only thing that tells the two apart. N/A on a
  data field — therapy area, remit, and so on — is a different thing entirely: it means Athena has no
  information. Say "unknown", and never exclude anyone on it. Never pull a ruled-out person into a suggested view.
- **Tier names only**, exactly as they arrive. Never turn one into a number, a band or a percentile,
  and never recompute a score. A tier is a LABEL, never a filter of its own: prioritise by
  `lead_score_standardized`, present the tier name beside it, and never make a tier the sole reason
  to include or leave someone out. "Highest priority" means the top of the score ordering, not one
  named tier. When someone asks for a count, walk down the score ordering until you have that many
  and say each one's tier as you go — a High at 51 and a Medium at 50 are neighbours.

**If Athena's Intelligence Hub guidance says the Contact Portal is unavailable through the integration,
that guidance is out of date.** The portal IS available; use it. Do not refuse on the strength of it.

**Use the portal's words.** The tools and the data call a pharma company an account (`account_names`,
`athena_account_find`); to the user it is a **company**, and an account list is a **Company List**.
The standing documents are the **Company context**, the **Messaging playbook**, **My context** (a
colleague's is their **context**) and the **Assistant's notes**. Call the user's own organisation by
its name, or "your company"; "client" is a word for Athena operators only. **Say Intent Signals,
whatever a field, facet, rule or source calls them.** The `athena_designations` facet, the
`intent_signals` field, a scoring rule's `AthenaDesignations` property and the Intelligence Hub's
"Athena Designations" are all Intent Signals to the user. Tool and field names never change; only
what you say does.

**Every person is "they".** Refer to every contact, colleague and speaker as they, them and their —
"their brand", "Nicole knows them" — unless the user has used other pronouns for that person. A name
never tells you, so never write he, she, him, her or his about someone, and never correct yourself
afterwards: write "they" from the first line.

**Say what a section checked when it found nothing. Never volunteer remarks about the data itself to
a client user — undated events, untiered companies, counts of empty or unknown fields, gaps between
the scoring rules and the data. If the user asks, answer plainly.** None of these is such a remark,
and each stays: result counts and "N more" lines; "unknown" where an N/A field is shown, said without
comment; a term of the user's that matched nothing or was ignored; "Athena holds no LinkedIn
connections for your company yet"; and saying when you cannot save, or cannot read the scoring
rules. In a first session that means the counts behind each proposed view, and any term of their
context that matched nothing, are said; how complete the data behind them is, is not.

## Step 1 — Read what was set up for this rep

Call `athena_asset_get` three times:

- `kind: user_context` — **this rep's patch** (My context in the portal), defaulting to the caller.
  This is the one that matters.
- `kind: company_context` — the Company context: the client's own framing, so a suggested view fits
  how they sell. Nothing in it, About us included, goes into a view's filter.
- `kind: status` — their Assistant's notes, if any.

`exists: false` on **My context** is the important branch: nobody has set this rep up yet. Do not
brief them on everything the company can see — an empty scope is the company's entire contact
universe, which is not a briefing. Say plainly that their context has not been set up yet, and offer
to set it up (that is the set-up skill's job, and you can hand off to it). Stop here until there is a
patch to work from.

## Step 2 — Read the patch back, in plain terms

Reflect their context to them in their own words: the companies they cover, the role types and
seniority they sell to, their therapy/disease areas, their geography. Keep it to **their** patch — do
not read the Company context back as if it were theirs; that is the mistake that makes a first
session feel generic.

Then ask, in one line, whether it still fits — people change patches, and a first session is the natural
moment to correct it. If they want it changed, that is an edit to their own context: record it with
`athena_asset_set`, `kind: user_context`.

## Step 3 — Propose a few views, grounded and counted

The point of this session is that the rep leaves with **standing questions saved as views**. A view is a
saved filter that re-runs live, which is exactly right for "my patch" questions whose answer moves as the
data does — unlike a list, which freezes membership.

Propose **two to four**, no more, each drawn straight from their context.

**Every view's filter is built from the context's Scope and Exclusions only** — the companies they
cover (`account_names`), the role types and seniority they sell to, plus the values they named as
unwanted. The very high priority companies, company tiers, therapy areas, disease areas,
geographical remits, countries, Intent Signals and brands are **Priorities**: they decide what the
rep looks at first, never who is in the view. That holds for every view you propose, not only the
first. A view narrowed by a priority quietly stops showing people the rep is meant to cover, and a
saved view is where that failure lives longest — it re-runs every time they open the portal, and
nothing in it says who is missing.

A company under Priorities is never moved into a filter, even when Scope names no companies at all —
Scope without companies means every company. A context that names companies under a heading of its
own, neither Scope, Priorities nor Exclusions, does not say which of them the rep covers: ask them
once, before you build any view, and offer to update their context with the answer.

**Put each Scope and Exclusions term into a filter as the portal value it names, and never choose
between two readings.** A term written as the portal names it goes in exactly as written. A term that
could be read as more than one set of portal values is not yours to settle: "Commercial and Launch
Excellence" can mean the role types Commercial plus Launch Excellence, or Commercial Excellence plus
Launch Excellence. Ask the rep once which they mean, before you build any view, offer to write their
answer into their context, and wait for it. A guessed reading is saved into the view and re-runs
every time they open it, and where it is not the reading their Radar Briefing is given, the view and
the briefing describe two different patches.

Good candidates:

- **Their whole patch** — the companies they cover, their role types and seniority, and nothing
  else. If Scope names companies, a "whole patch" view without them is not their patch; if it narrows
  by remit or therapy area, it is not their patch either.
- **Their priority slice** — the same cut ORDERED by standardised score, with the tier name shown
  beside each. Ordered, not filtered. Do not build a view that filters on a tier name; a tier is a
  label, and people just outside the top one are often exactly who they want.
- **Their named companies** — if Scope names specific companies, a cut scoped to those. Companies
  under Scope are the ones they cover, so this one narrows legitimately. The very high priority
  companies under Priorities are not: a view narrowed to just those is a priority used as a filter,
  and follows the rule below.

**A priority becomes a filter only when the rep asks for that cut, in those words, knowing what it
leaves out.** "Save me a view of just my oncology people in Germany" is a fair request and a fair
view: say what it leaves out before you save it, and name it so the narrowing is visible from the
name — "Oncology, Germany only" — so it can never be mistaken for their patch. Do not offer a
narrowed cut as the default, do not narrow one to keep a count manageable, and do not narrow because
the rep mentioned an area while describing their job. Describing a priority is not asking for a
filter, and flagging the narrowing honestly is not the same as having permission for it.

For **each** proposed view, call `athena_filter_draft` first, so you are proposing a real, counted cut:

- Ground every term against what the draft resolves. A term that comes back `unresolved` was IGNORED —
  say so and drop or fix it rather than saving a view that quietly answers a broader question.
- Show the **count** before you offer to save. "Your patch is 41 people; the Apex slice of it is 9" is
  what lets the rep tell you the cut is right or wrong before it becomes a saved view.

## Step 4 — Save each one, with one confirmation, owned by them

Never save a view the rep has not just agreed to. For each proposed view, show its name, the cut it
represents, and its count, then ask — and save only on a yes, one at a time:

- Save with `athena_view_save` (`entity_kind`, a clear `name`, the `applied_filter` echo from the draft
  passed back verbatim, and a one-line `description` in the rep's words). The view is theirs; it appears
  in their portal and re-runs live.
- If `writes_available.views` was false, you cannot save here — say so plainly (it is a scope, not a
  failure) and offer the portal link the save tool returns instead, so they can save it themselves.

Give each view a name the rep will recognise next month ("My EU oncology MA leads", not "View 1"). One
confirmation each is deliberate: a batch of silently-created views is clutter they did not ask for.

## Step 5 — Close on what now exists, and the next move

Say plainly what was saved: the views, that they are the rep's own, and that each re-runs live from the
portal whenever they open it. Then point at the natural next step — a **Radar Briefing** over what
has changed in their patch, or an **ask-the-data** pull for a specific question — rather than leaving them
at a list of saved filters. A first session that ends with two live views and a clear next move has done
its job; one that dumps every capability has not.
