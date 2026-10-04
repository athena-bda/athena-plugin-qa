---
name: athena-set-up
description: Set up or tune Athena — record what your company sells and who it sells to, which companies each person covers, and the Messaging playbook you draft from, so every later Athena conversation is already scoped. Use when someone says "set me up", "set up Athena", "tune my context", "set up a colleague's context", "help me build my messaging playbook", or when a briefing comes back too broad because nobody has said what the user actually cares about.
named-terms:
  briefing: Radar Briefing
  pickup-phrase: What's next on my radar
  note: Placeholders. Written here once so a rename is one edit.
---

# Set up / tune

Athena works from four standing documents. This skill writes them, and these are the names the
portal gives them:

- **Company context** — who this client is, what they sell and who they sell to. It opens with
  **About us**. Everyone at the client reads it.
- **My context** (a colleague's is their **context**) — one person's patch: the companies they cover,
  the role types they sell to, what they are working on.
- **Messaging playbook** — the messaging outreach is drafted against.
- **Assistant's notes** — an assistant's own working notes for one person, including where their last
  briefing got to.

The tools call them `company_context`, `user_context`, `playbook` and `status`.

Get these right once and every later conversation starts scoped. Get them wrong and every briefing
answers a broader question than anyone asked.

## Before you start

**Check you are live by CALLING a tool, not by looking for one.** On some platforms connector tools
are listed but not loaded, so "I can see the tools" is not evidence and "I cannot see them" is not a
failure. Call `athena_orient`. If it returns, you are connected. If it does not, say so plainly and
stop — do not improvise from memory.

`athena_orient` also tells you three things this skill needs: which company you are in, whether this
session can save anything (`writes_available`), and the vocabulary and safety rules that apply to
everything below. Read them; they are not repeated here.

Two rules are worth repeating here, because a set-up conversation is where they get built into
everything that follows:

- **Scores have three states, and one number.** `lead_score_standardized` is the only lead score you
  will see or say. `lead_score_standardized` absent means not scored on this client's standardised
  scale. `lead_score_standardized` 0 with a tier is a REAL score, at the bottom of the ranking.
  `lead_score_standardized` 0 with `lead_score_tier` "N/A" means one of this client's own scoring
  rules ruled the person out: a value they marked as unwanted, with nothing scored to outweigh it.
  Say "ruled out by your scoring rules", never "unscored", and keep them out of priority lists. A
  ruled-out person's score is 0 too, so the TIER is the only thing that tells the two apart. N/A on a
  data field — therapy area, remit, and so on — is a different thing entirely: it means Athena has no
  information. Say "unknown", and never exclude anyone on it.
- **Tier names only**, exactly as they arrive. Never turn one into a number, a band or a percentile,
  and never recompute a score. A tier is a LABEL, never a filter of its own: prioritise by
  `lead_score_standardized`, present the tier name beside it, and never make a tier the sole reason
  to include or leave someone out. "Highest priority" means the top of the score ordering, not one
  named tier. When someone asks for a count, walk down the score ordering until you have that many
  and say each one's tier as you go — a High at 80 and a Medium at 79 are neighbours.

**Use the portal's words.** The tools and the data call a pharma company an account (`account_names`,
`athena_account_find`); to the user it is a **company**, and an account list is a **Company List**.
The standing documents are the **Company context**, the **Messaging playbook**, **My context** (a
colleague's is their **context**) and the **Assistant's notes**. Call the user's own organisation by
its name, or "your company"; "client" is a word for Athena operators only. **Say Intent Signals,
whatever a field, facet, rule or source calls them.** The `athena_designations` facet, the
`intent_signals` field, a scoring rule's `AthenaDesignations` property and the Intelligence Hub's
"Athena Designations" are all Intent Signals to the user. Tool and field names never change; only
what you say does. **Pipeline Blockbuster is the retired name of the Intent Signal Pipeline Asset.**
Propose and write Pipeline Asset, the name the contacts carry, even where the scoring rules still say
Pipeline Blockbuster (that rule's `option_label` carries the current name), and never both.

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
rules. In this interview that also means never telling the user how many values a facet holds, which
companies have no tier, or where the scoring rules and the data disagree — Step 3 has the one
exception, for Athena operators.

**Use the user's other tools.** If they have their CRM, their drive or a research tool connected, use
them. Athena is one source among several and works better beside the rest.

## Step 1 — Work out which of the three set-ups this is

Read `you` and `writes_available` from `athena_orient`.

- **`is_auto_scoped` is true** — the user belongs to one company and everything scopes to it. This is
  a client admin setting their own company up, or doing it on a call with Athena. Do not ask for a
  company id; they have one and it is already applied.
- **`is_auto_scoped` is false** — the user works across companies. They are an Athena operator, or a
  client user who belongs to more than one company. Call `athena_company_list` and ask which one this
  session is for, by name, from that list: an operator is choosing a client ("Northwind or
  Halcyon?"); a client user is choosing between their own workspaces ("Brightwater, Athena BDA or
  Halcyon?"). Never ask a client user which "company" — to them a company is a pharma company. If
  they have already named one ("in Northwind"), take it from the list rather than asking again. Pass
  that `company_id` on **every** subsequent call, `athena_team_find` included. Do not guess an id and
  do not carry one over from an earlier conversation.
- **Whether you can save the documents** — the four documents are published with `athena_asset_set`,
  so the write family that matters here is **`writes_available.assets`**, not `lists` or `views`. Read
  it as three cases, and do not collapse them:
  - **`is_determined` is false** — you have not scoped a company yet (you are an operator, or a
    multi-company user). This is **not** a refusal. Choose a company first (the bullet above), and let
    the company-scoped tools answer: `athena_asset_get` returns `can_edit` per document. Do not tell
    the user you cannot save on the strength of an undetermined answer.
  - **`is_determined` is true and `assets` is false** — this user cannot publish the Company context
    or anyone else's context from this connection (their own context they can still change). Say so
    at the START, before the interview, not after it. Run the interview anyway if the user wants — the
    answers are still useful — but tell them the documents will need publishing by someone who can,
    and do not end the session implying anything was recorded. If they want a colleague's context,
    still call `athena_team_find` (Step 6): its refusal names the people who can publish it.
  - **`is_determined` is true and `assets` is true** — you can publish. Proceed.

**Note whether this is an Athena operator session** — `athena_orient`'s `you` says when the caller
is acting as an Athena operator. Two lines below are for operators only: one about scoring rules that
match nothing in the data (Step 3), and one about the Key Companies list (Step 4). Everything else is
identical when someone from Athena runs this for a client: the documents land in the client's
company, and the people in that company read them from their next session.

## Step 2 — Ground the vocabulary before you ask about it

Call `athena_filter_options_get` for `contact`, and again for `account` — the companies topic needs
the account `tiers` facet.

This is not a formality. The values are live and differ per company: a therapy area or tier that
exists for one client may not exist for another, and a term the user says confidently may match
nothing here. Read each facet's flags as well as its values — a facet reporting `is_empty` is a
question not worth asking, and a facet reporting `is_truncated` is a sample you must not treat as the
whole vocabulary. The flags steer what you ask; they are not something to tell the user.

Two facets are worth reading before the interview even starts:

- `lead_score_tiers` — if this client's scoring is already set up, the tier names tell you so.
- `connections` — if it is empty, this client has no LinkedIn connection data, and any plan that
  leans on "who do we already know" has nothing behind it. Find that out now, not in a briefing.

## Step 3 — Start from the client's own scoring rules

Call `athena_scoring_rules_get` before you ask anything. It returns the scoring rules set up for this
client: which values of which properties earn points, and which values rule a contact out entirely.
Each row carries the `property`, its `property_label` — the portal's own name for it — and the
`option`. Those are decisions the client has already made, and starting from them turns the
interview into confirmation and correction — which people are far better at than invention.

**If it returns rules, use them to propose, never to recite.** Topic by topic, turn the values the
rules score into a short, obvious shortlist the user can accept, reject or tweak: "From your lead
score criteria, these look like the role types you care about: Market Access, Medical Affairs,
Procurement, Launch Excellence. Is that right, any to add?" **Name each value on its own, as the
portal names it** — as a list, or separated by commas alone, never joined by an "and" that lets two
values share a word. "Health Economics, Commercial and Launch Excellence" can be heard as Commercial
plus Launch Excellence or as Commercial Excellence plus Launch Excellence, and the user confirms a
reading you did not mean: it is the fold step 4 forbids under Scope, said aloud. The same holds
whenever you say the values back. **At most five suggestions a topic** — the
values the rules score highest, the obvious ones — never every value the rules score; the user can
add the rest. Say the `property_label` (Role Type, Intent Signals), never the `property`, and quote
each `option` verbatim. In the interview, never read out a
score, never list the values the rules mark as unwanted, and never remark to a client user on gaps
between the rules and the data.

**Propose only values that exist in the data.** Before a scored value goes into a suggestion, check it
against the facets from Step 2, or with `athena_filter_draft`. A value the rules score but no contact
here carries is left out of the suggestion, without comment.

**If the user asks what the rules say**, that is a different question, and it gets a plain answer:
read every row back grouped by `property_label`, each `option` quoted verbatim with what the rule does
to it — its score, or that it rules a person out — and no tier thresholds.

**In an Athena operator session**, you may add one line for Athena about scoring rules that match
nothing in the data: "For Athena: the rules score R&D and Clinical, but no role type here carries
either, so those points never land." Once, and never in a client user's session.

**A ruled-out value is not an exclusion you may apply on their behalf.** An exclusion comes from what
the user says they do not want, never from a rule row — so ask topic 4's exclusion question on its
own, whatever the rules mark as unwanted.

**If it returns no rules**, or says it could not read them in full, say so in one line and run the
interview cold, from topic 1. That is a normal state, not a fault. It never means the client has not
been scored.

Once a topic is confirmed, a count is still worth putting in front of them: draft it with
`athena_filter_draft` so the shape of the population is visible before anything is written down.

Two limits to be straight about either way:

- The rules do not usually carry companies or brands. Those topics are real questions every time.
- You can read what the scoring decided; you cannot change how it is calculated in this conversation.
  If the user wants the weighting changed, use these words: **"these are the scoring rules set up for
  you, which we can change — ask Athena"**. Do not name the place the rules are authored and do not
  send the user to it. It is not a surface a client can reach, so a route to it is a door that does
  not open, and an instruction to go there reads as the user's job left undone. This holds in the
  conversation AND in anything you publish: never write the name of an internal surface into a
  context document, where it outlives the conversation and the whole company reads it.

Do not ask for, infer, or state a tier threshold. Where a tier boundary sits is not something this
conversation needs and not something you are given.

## Step 4 — The interview

**Open by saying what this is for**, in one or two lines, before the first question: it records what
they sell and who they sell to, so every later briefing and draft starts from that; and it writes
their **Company context** first — the one everyone at their company shares — then each person's own
**context**. Use their company's name. Something like: "This sets Athena up for Northwind, so every
briefing and draft from here on starts from what you sell and who you sell to. I'll write Northwind's
Company context first, then each person's own context." Whenever you move from one document to the
next, say which one you are gathering now ("Now your own context").

Cover all seven, in this order, every time. Ask each one; do not wait for the user to notice a topic
was skipped. Where the scoring rules already answer a topic, propose the shortlist from them (Step 3)
rather than asking cold. If they answer several topics at once, take the lot and confirm each
remaining one in a line rather than making them repeat themselves.

1. **Companies — the handful that matter most, and which company tiers.** Ask for a handful of very
   high priority companies to track closely, not a long list, grounded as below. Then, inside the
   same topic: "Which company tiers matter most — Big Pharma, Mid Pharma, Small Pharma, Biotech?",
   offering the values exactly as the account `tiers` facet returns them. The scoring rules do not
   usually carry companies, so this one is always a real question.
2. **Role types and seniority.** Who they sell to inside those companies.
3. **Intent Signals.** The triggers worth acting on — the `athena_designations` facet.
4. **Geographical remit priorities, and anything to leave out.** Ask which remits are priorities, and
   say more than one can be chosen. Offer the values exactly as the facet returns them — Europe and
   European Region are two different values with two different meanings, and merging them loses the
   distinction. Then ask separately whether there is any remit they definitely do not want. Only that
   second answer becomes an exclusion.
5. **Therapy areas.** Which ones matter most.
6. **Disease areas that are very high priority.** This steers suggestions and drafting; it never
   narrows the briefing. Do not turn it into a filter that leaves anyone out.
7. **Brands.** "Are there any very high priority brands you'd like to track?"

Two rules run across all seven:

- **Priorities are not exclusions.** Everything above steers what Athena suggests and drafts, and
  the handful of very high priority companies also leads the briefing. Nothing above removes anyone
  from view unless the user named a specific value they do not want — for example an African or a
  Local affiliate remit. Ask both questions and keep the answers apart.
- **Never exclude on an unknown.** A contact whose remit, therapy area or disease area is N/A is a
  contact Athena has no information about. They stay in.

**Ground the companies topic in Athena's Key Companies.** Athena's monthly tracking covers a list of
Key Companies. Find it with `athena_list_find`, `entity_kind` `account`, `search` "Athena BDA Key
Companies", and keep only a list whose name is exactly that.

- **Exactly one such list is visible** — ground the topic in it. Any company you suggest comes from
  its members. Check each company the user names with `athena_account_find`, filtering on `list_ids`
  (that list's id) with a `search` for the name; if their short name finds nothing, try the name
  Athena holds it under (Johnson & Johnson for J&J) before you decide it is not a member.
  - A member: take it.
  - Not a member: warn once, and keep it if they want it: "Athena's monthly tracking covers its Key
    Companies, and Dyne isn't one of them, so it will show up less fully in briefings. Keep it
    anyway?" If a member is plainly part of the same group — J&J for Janssen, which Athena holds as a
    separate company — say so, once you have checked that it is a member. On a yes, keep it, under
    the name Athena holds it by.
  - Say a company's tier, or anything else about it, only from its member row, and say nothing about
    a company that has no tier.
  - Call a company a Key Company only when you checked it in this turn and the check found it, and
    name the companies you mean. Never extend a membership statement to a company you did not check
    in this turn: "All six companies are Key Companies" said after checking three of them is a guess
    about the other three. A company you have already warned about is not warned about again, and is
    not called a Key Company either. This holds wherever you check, a person's context included.
- **No such list, or more than one** — run the topic without it, grounding each name against the
  contact `account_names` facet, and never quote how many companies that facet holds. In an operator
  session, say once, for Athena, that the Key Companies list is not visible to this client; in a
  client user's session, say nothing about it.

The list grounds this conversation and nothing else. It is never a filter: do not write it into any
context, and do not narrow anything to it.

**Company tiers are a priority, never a filter.** Write the answer as one sentence under Priorities
("Small Pharma and Biotech matter most"), and as you record it, tell the user it guides your
suggestions and your drafting — briefings do not sort by company tier. Do not promise a search or a
briefing that uses it.

Do not ask which lead score tier the briefings should cover. A lead score tier is a label on what the
client's own scoring produced, not a question for the user — unlike the company tiers in topic 1.

**About us.** After the seven topics, gather the Company context's opening: an **About us** of three
to five sentences — what they do, who they serve, and what sets them apart. Offer to draft it from
their website: "Shall I draft a short About us from your website?" If you do not have its address,
ask for it. Draft it only from the page itself, read in this conversation. **A web tool that is
listed but not yet loaded is a tool you have:** load it — through the tool search, where the session
has one — and try the page before you ever say you cannot open it. Say you cannot open their website
only once a real attempt has failed, or when this session has no web tool at all, loaded or not;
then ask them to paste the About text from their website or to tell you in a few sentences. Never
write it from what you remember about the company, and never fill a gap with a guess. It is shown
with the rest of the Company context before anything is published.

Write what you learn into the context in three labelled parts, so a later skill can tell them apart:
**Scope** (the companies covered, role types, seniority), **Priorities** (the very high priority
companies, company tiers, therapy areas, disease areas, remits, Intent Signals, brands) and
**Exclusions** (only values the user named as unwanted). Briefings and views filter on Scope and
Exclusions, and nothing under Priorities filters.

**Under Scope and Exclusions, write each filter value exactly as the portal names it, as an item of
its own** — the value the tools returned, and the one you drafted the count with. Never fold values
into shared words: "Commercial and Launch Excellence" can be read as the role types Commercial plus
Launch Excellence, or Commercial Excellence plus Launch Excellence — all four are real — and two
readings brief two different patches. Write "Role types: Commercial, Launch Excellence."

**Only the very high priority companies change a briefing's order.** Their people come straight
after connections. Everything else under Priorities — therapy areas, disease areas, remits, Intent
Signals, brands — steers your suggestions and drafting, as company tiers do, and never the order.
When you tell the user what a context does, say exactly that: "Amgen and J&J lead your briefings;
Oncology, Europe and the United States steer what I suggest and how I draft." Never say a therapy
area, disease area, remit, Intent Signal or brand puts anyone at the top or moves them up a list.

**Every context takes those three headings — the company's, and every person's.** Use the three
words themselves as headings in every context this interview writes. The Company context is About us
first, then the three headings — Scope, Priorities, Exclusions, in that order; a person's context has
the three headings alone. A context written as prose, or under headings of your own, gives the skills
that read it back nothing to tell apart, and they are the skills that decide who is in a briefing.
Nothing reads About us as Scope, and nothing filters on it.

**Companies under a heading of their own make the briefing stop and ask.** If the user writes, or
asks you to publish, a context that names companies under a heading of its own — "My companies",
"Priority accounts" — tell them what it will do before you publish it: the briefing cannot tell
whether those are companies they cover or companies that matter most, so it will stop and ask which
of them they cover, and a scheduled briefing will produce nothing until the context says which.
Never tell them such a heading covers every company, or that it is ignored. Offer to put those companies under
Scope, Priorities or both, and publish their own wording only if they still want it.

**Covered companies go under Scope; very high priority companies go under Priorities.** The handful
from topic 1 goes under the Company context's Priorities. Its Scope names companies only if the user
says their whole company sells to a fixed set of them. In a person's context, every company they
actually cover goes under Scope — including the ones they call their priorities — because Scope is
what narrows their briefing, and a covered company missing from it drops out of everything they see.
Each of the company's very high priority companies that the person covers, and any other company they
cover that they say matters most, goes under their Priorities as well, where it only ranks: it leads
their briefing and never narrows it. A company under Priorities is never read as one they cover, so
never file a covered company under Priorities alone, and never under a heading of your own such as
"My companies". Covered companies narrow. Always.

A Company context, in full — About us first, the handful under Priorities:

```
## About us
Three to five sentences from their website or their own words: what they do, who they serve, and
what sets them apart.

## Scope
Role types: Market Access, Medical Affairs.
Seniority: C-Suite, VP, Head, Director.

## Priorities
Very high priority companies: Pfizer and Novartis.
Company tiers: Small Pharma and Biotech matter most.
Therapy areas: Oncology, Immunology.
Intent Signals: the ones this client's own facet actually returns.
Brands: only the very high priority ones they named.

## Exclusions
Geographical remits: Local Affiliate — named by them as unwanted.
```

Then a set of things that are **not portal filters** but belong in the Company context as prose,
because they steer drafting and the intelligence side rather than a search: drug lifecycle stage,
route of administration and sales tier, alongside the company tiers above. Write them down as
sentences. Do not invent filter fields for them and do not promise a search that uses them.

While you go, check each answer with `athena_filter_draft`. A term that comes back in `unresolved`
was IGNORED — say so and offer the suggestions rather than writing a context term that will never
match anything. A term in `ambiguous` needs one question answered before it means anything.

## Step 5 — Propose with counts, then save

Never save a document the user has not seen. Show them:

- the Company context you intend to write, in full, About us first;
- the number of people it describes, from `athena_filter_draft`, with any caveat the draft reported
  before the number rather than after it;
- for each person you are seeding, the context you intend to write for them and its count.

**Say what publishing means, once, before the first save.** In one line, in their words: this goes
live for everyone at their company straight away, every version is kept with who wrote it and when,
and it can be rolled back from the portal. The Assistant's notes are the exception — those are your
own working notes for that person, they overwrite in place and keep no history. Say it once, at the
first save, not every time.

Then save with `athena_asset_set`. Three things to know about saving:

- **It goes live immediately.** There is no draft and no review step. That is why the confirmation
  above is the control.
- **Pass `row_version` exactly as `athena_asset_get` returned it.** If someone changed the document
  while you were talking, you get a conflict, nothing is overwritten, and the response carries the
  current live document. Merge your change into that and call again with ITS `row_version`. Never
  retry a conflict by sending the same content again.
- **Check `can_edit` before you offer.** If it is false, the user cannot publish this document, and
  `editable_by` names who can — tell them who to ask, in plain words, instead of proposing an edit
  the server will refuse. `editable_by` is filled in ONLY when `can_edit` is false, so an empty one
  beside `can_edit: true` is the normal state and never a fault. Do not narrate it, and never read a
  field name out to the user.

After each save, say it is live and give its version from the response.

Documents are capped at 64 KiB of text. Over that the save is refused with the exact numbers and
nothing is truncated — cut it down rather than hoping.

## Step 6 — Seed each person

Seeded contexts are live as soon as they are written, and nobody has to accept them for the system
to work. Publishing a context does not itself trigger a read-back or any other follow-up, so promise
only what is true. Say, when you seed: "Sam's context is live now. They can see it under My context in
the portal and change it whenever they want, or ask their assistant to." Never say their assistant
will read it back to them, check it with them or offer to tune it because it was published: nothing
follows from publishing, and a promise nobody keeps is one the user will notice.

**Call a colleague "they"** — "the companies they cover", "their context" — unless the user has used
other pronouns for them. Never infer pronouns from a name.

**To seed a colleague, find them by name or email with `athena_team_find`. Never ask the user for an
id.**

- Search once with what the user called them: `search` set to the name or email, `offset` 0, `limit`
  200 — and the client's `company_id` in an operator session.
- A person is found only when that single response is untruncated — no `result_truncated` — and holds
  exactly one row that is not the user themselves (`is_you` false). Then confirm the full name before
  you write anything: "Sam Patel?" The search matches any part of a name or email, so
  "Sam" alone is not yet a person.
- Several such rows: ask which one they mean, by full name, and by email where two share a name.
- A truncated response: ask for more of the name, or their email, and search again from offset 0.
- Never identify anyone from a second or later page.
- Nobody else matches: say they may not have a portal login yet — Athena adds users — and carry on
  with the rest of the set-up.
- Pass that row's `user_id` as `user_id` to `athena_asset_get` and `athena_asset_set`: only a
  `user_id` this tool returned, never one you made up. The user's own context needs no id; it
  defaults to them.

**If `athena_team_find` is refused** (`permission_denied`), this user cannot see the team list. Say so
plainly: only people who can set up your company's documents can see it. Name who can, as the
refusal names them — people at their company by name, Athena's operators as "Athena" — and offer to
draft the colleague's context here for one of them to publish. No error text, and never a pretend
success.

**If `athena_team_find` does not exist on this connection** — you call it and are told there is no
such tool — you cannot find colleagues from here. Say so plainly, draft each colleague's context here
as above, and tell the user it can be published from the portal instead: the Team section of the
Assets page, by someone who can set up your company's documents.

Ask which companies a person covers when you write their context, not in the companies topic — topic
1 gathers only the company's handful — and put every one of them under their Scope. If they cover
every company, say so under Scope in words ("All companies") and name none there; the company's
handful still goes under their Priorities. If you check the companies they cover against the Key
Companies, say what step 4 allows about membership and no more: only the companies checked in this
turn, and nothing new about a company already warned about.

Write each person's context in the same three labelled parts as the company's — **Scope**,
**Priorities**, **Exclusions** — and use step 4's mapping field for field, not just its headings:
**Scope is the companies this person covers, role types and seniority; Priorities is the very high
priority companies among them, therapy areas, disease areas, geographical remits, Intent Signals and
brands; Exclusions is only the values this person named as unwanted. Company tiers stay in the
Company context only: they are the company's answer, not this person's.** Nothing about seeding
changes that mapping, or step 4's rule that each value under Scope and Exclusions is written exactly
as the portal names it, as an item of its own. It is easier to break here than in the Company context, because the person you are
writing about is usually not in the room to notice that their territory has been filed in the wrong
part.

**A remit under Scope silently narrows everything they will ever see.** The skills that read a
person's context FILTER on Scope and Exclusions, and never on Priorities. Write "Territory. Remits
Europe, European Region and Global." under Scope and every briefing and every saved view that person
gets is cut down to those three values — no error, no warning, and everyone outside them quietly
stops appearing. A remit is a priority. So are therapy areas, disease areas, Intent Signals and
brands: each one under Scope removes people instead of steering, and a person's context is the
one that decides whose briefing they get. Written as a paragraph, or with the companies they cover
under Priorities alone, it hands them a briefing across every company, labelled as their own patch.

A seeded person's context, in full — every company they cover a plain list under Scope, the company's
very high priority ones among them under Priorities too, remits under Priorities:

```
## Scope
- Pfizer
- Novartis
- AstraZeneca

Role types: Medical Affairs, Market Access.
Seniority: C-Suite, VP, Head, Director.

## Priorities
Very high priority companies: Pfizer and Novartis.
Geographical remits: Europe, European Region, Global.
Therapy areas: Oncology, Immunology.
Disease areas: non-small cell lung cancer.
Intent Signals: the ones this client's own facet actually returns.
Brands: only the very high priority ones they named.

## Exclusions
Geographical remits: Local Affiliate — named by them as unwanted.
```

Europe and European Region are two values, not one, and both are listed under Priorities exactly as
the facet returns them. Pfizer and Novartis are under Scope because this person covers them, and under
Priorities because they are the company's very high priority companies. The Exclusions line is there
because this person named it, not because a remit was left out of the priorities.

People can always edit their OWN context and their own Assistant's notes. They cannot read each
other's — that is by design, not a permission that can be granted, so do not offer it.

## Step 7 — The Messaging playbook

**This one is optional, and it is the last thing we do — we can do it now or later.** Say that, and
mean it. A client who wants to stop after their contexts are published has a working set-up.

The Messaging playbook is what outreach gets drafted from, and it is paired with **Athena's email
writing guide** on the Intelligence Hub connector. If the Hub is connected, call
`get_email_writing_guide` before seeding the playbook and let it steer the interview; if it is not,
build the playbook anyway and say the conditional messaging is worth revisiting with the guide to
hand.

Lead with what it buys them, not with how it is structured: the more approved messaging they supply,
the more every draft can be tailored to the person it is going to; without it drafts stay generic and
say the same thing to everyone. Never say "Tier 1", "Tier 2" or "blocks" to the user — those are the
guide's internal vocabulary and mean nothing to the person answering.

Then make it concrete. Read the client's own `athena_designations` facet, tell them which Intent
Signals actually appear across their contacts, and for each of the common ones ask what they would
want said when it is present. Record which signals were deliberately left uncovered, and say plainly
that a signal with nothing written for it is a hook their drafts will silently never use — so the
choice is visible rather than an accident. (Read the guide's own handling of those signals from the
guide each time; it is Athena's to change, not this skill's to remember.)

**Ask for their material, and read it here.** The single most useful thing this step can do is work
from what they already have: invite them to paste or attach approved messaging, case studies and
positioning into the chat, and draft the conditional messaging from those rather than from a blank
page. Show what you took from each and what you left out, so nothing is quietly dropped.

Seed the playbook with the structure Athena's template uses, and fill what the conversation gives
you:

1. **Company overview and baseline messaging**, written under three sub-headings: **Company
   overview** (what the company does, its flagship products and what sets it apart; the Company
   context's About us is the starting point, so build on it rather than asking for it twice),
   **Case studies** (two to four client stories from their own material: the context, the problem,
   what they did and the outcome) and **Baseline email examples** (two or three emails they are
   happy with, with their subject lines).
2. **Conditional messaging by data point** — one piece of conditional messaging per therapy area,
   disease area, role type, Intent Signal or brand where they want the framing to change.
3. **Job change and conference guidance** — what to say to someone who has just moved, been promoted,
   or is speaking somewhere.
4. **Company-specific guidance** — one piece per company they sell to where the framing should
   change: an existing relationship, a prior pilot, something to avoid mentioning.

Fill it from what they tell you and from the material they give you here — approved messaging, case
studies, positioning, past proposals, their website. Read those in this conversation, or through
**their** tools: their drive, their document store, or a file they attach. For their website, the
About us rule on web tools in step 4 holds here too: try the page before you say you cannot open it.
Show what you took from each and what you left out. Athena never stores their source documents; only
the playbook you write lands here, so do not offer to keep the files. If you cannot reach something
the playbook needs, ask for it. Do not write a case study from memory and do not invent a metric.

A playbook with two honest pieces beats one with twelve invented ones. Say that if they stall.

## Step 8 — Offer the recurring briefing

The point of set-up is that the briefing then arrives without anyone asking for it.

**Offer a scheduled run only when this session has a tool that creates one** — a tool you can see
and could call now, not a feature the platform may have somewhere else. **The test is that you can
name the tool you would call.** If you cannot name it, there is no such tool.

With no such tool, do not offer to set one up, and do not list it among the things you can do next.
Say so plainly and leave it: they will need to ask each month, or use their platform's own
scheduling. An assistant that promises a recurring run it cannot create is worse than one that says
"you will need to ask me each month."

With one, offer to create it: "shall I set up a monthly run that does the Radar Briefing and shows
you the result?" Monthly, timed for just after Athena publishes its curated editions, is the right
default — that is the cadence the underlying data moves at, and a weekly run mostly reports that
nothing has happened. Conference work moves faster and is better run on its own, when a conference
is actually coming up.

Two things not to rely on:

- **Do not lean on the platform's own approval prompts to protect the writes.** Whether a scheduled
  run is asked to confirm an action varies by platform and by surface, and it is not dependable. The
  protection is that the briefing's only write is idempotent and it advances the baseline only after
  the briefing has actually reached the user.
- **Do not assume an unattended run has your tools loaded.** Write the instruction to load the Athena
  tools into the scheduled task's own prompt.

## Step 9 — Confirm what is live

Close by saying, in plain terms, what now exists: the Company context and its version, whose contexts
were seeded, whether the Messaging playbook is started or finished, and whether a recurring run was
created. Every document keeps its full history with an author and a timestamp against each version,
and anything can be rolled back from the Athena portal — worth saying once, because it is what makes
direct publishing safe.

**Whenever you list what you can do next — here, after a save, anywhere in this set-up — a
scheduled run is on the list only when you can name the tool that would create it.** When you
cannot, leave it off and say instead that they will need to ask for their Radar Briefing each month.

One exception, and it needs saying: **the Assistant's notes keep no history.** Anything cleared from
them is gone. If you are ever about to clear them, say "this cannot be undone" and get an explicit yes
first.
