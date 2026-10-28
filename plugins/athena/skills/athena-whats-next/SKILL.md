---
name: athena-whats-next
description: Produce the Radar Briefing from Athena — four sections served one at a time (multi-signal leads, Job Change Updates, Pipeline News Updates, Conference Attendee / Speaker), each grouped into the user's connections, their Watchlist and top prospects, with drill-down. Use when someone asks what's new, what's changed, what they should do next, who has moved, asks for multi-signal leads, says "next section", says "what's next on my radar", calls on the assistant as Aidrian ("Hey Aidrian, what's new?"), or when a recurring briefing run fires.
named-terms:
  briefing: Radar Briefing
  pickup-phrase: What's next on my radar
  note: Placeholders. Written here once so a rename is one edit.
---

# Radar Briefing

A briefing, not a data dump. Four sections, served one at a time — Multi-signal leads, Job Change
Updates, Pipeline News Updates and Conference Attendee / Speaker — each short enough to read in a
minute, and each about people the user could act on this week.

The engine is `athena_changes`, called once for each section. Athena chooses the people, groups them,
orders them, numbers them and pages them. This skill picks the news stories, says it all in plain
words, and keeps the Radar Briefing markers in the Assistant's notes, so a briefing can be picked up
where it stopped, in this conversation or another.

## Before you start

**Check you are live by CALLING a tool, not by looking for one.** On some platforms connector tools
are listed but not loaded, so "I can see the tools" proves nothing either way. Call `athena_orient`.
If it returns, you are connected. If it does not, say so and stop — never produce a briefing from
memory or from an earlier conversation.

**This briefing needs Athena's four-section briefing.** `athena_orient` names the briefing it serves
as `briefing_contract`, and every section response from `athena_changes` carries `contract`; both
say `"sections-1"` for this one. If `athena_orient` has no such `briefing_contract`, or if
`athena_changes` comes back without a `section` and that `contract`, the Athena connector is older
than this briefing: say so in one line, give no briefing and save no marker. Do the same when Athena
refuses a section call with `sections_disabled`, which means the four-section briefing is switched
off: one line — "Athena can't serve the Radar Briefing right now, so I'm not running one; please try
again later." — no briefing and no marker. When Athena refuses with `job_change_list_not_found`, no
Job Change list has been published for their company yet: say so in one line — "Athena hasn't
published a Job Change list for your company yet, so there is no Radar Briefing to run." — with no
briefing, no marker and no retry. Never call `athena_changes` without `section` to fill the gap: that is an older kind
of briefing, and its answers do not fit this one.

**The Intelligence Hub gives the briefing its news.** You read this month's News Digest from the Hub,
and Athena uses it for Pipeline News Updates and for the news half of multi-signal leads. Athena reads
the conferences and their speakers itself. The Hub's `whoami`, called once before the first Hub read,
is the check. If the Hub does not answer, brief from the portal anyway and say, in Pipeline News
Updates and above multi-signal leads, that the news could not be checked because the Intelligence Hub
is not connected. An unchecked part must never read as one that found nothing.

**Check the Hub is signed in to the same client before you use anything it says.** Compare the
`hub_client_id` of the company you are working in — on `athena_orient`'s `company`, or on the row
you chose from `athena_company_list` — with the `companyId` the Hub's `whoami` returns, never with its
`clientId`, which names the app that connected, not a client. If they match,
carry on: there is nothing to say, and the rule on scores below holds all the same. If they differ,
say once, plainly: "The Intelligence Hub is signed in to a different organisation from Northwind, so
I'm using only what it publishes for everyone: conference dates and agendas, Likely Attendee lists
and pipeline news." If either id is missing you cannot tell, so say the same thing in other words:
"I can't confirm the Intelligence Hub is signed in to Northwind, so I'm using only what it publishes
for everyone." Either way, from then on use only those.

**Whichever of those two sentences applies has one place in your reply, and is said there exactly
once.** In a briefing it is the first line of the briefing, before the opening line and everything
else, and later sections do not repeat it. In a pick-up it comes directly after the pick-up's opening
line, before anything else. Nowhere else: not in a progress line while you work, not a second time
further down, and never only in your own reasoning — a sentence you only thought has not been said.
A one-line refusal (the paragraph above on Athena's four-section briefing) is the whole reply, with
no Hub sentence beside it.

**Whether or not the Hub matches, every score, tier and "who knows them" comes from the contact
portal.** Never use the Hub's speaker scores, tiers or connections — the `lead_score`,
`lead_score_tier` and `connection` on any Hub record, a speaker, a roster or a prompt list — not
even when the Hub is signed in to this same client: they are the Hub's own, and a person given one
score by the Hub and another by the portal has two. Name a Hub speaker with their company and
conference, and the LinkedIn link their Hub record carries — a public profile link, not the Hub
client's data — and nothing else, unless you have found them among this client's contacts: then
their score, tier and "who knows them" are that contact's, read from the portal, and since the match
is made on name and employer, say they look like the same person.

**A Hub call that fails or times out is retried at most once, and the retry counts against the call
limit.** That holds for every Hub call, not only the first: `list_news_digests`, `get_news_digest`,
a `get_drug`, and the `list_conferences` call for a conference the user names. If the retry fails
too, name the part that call would have covered as not checked — "news not checked - the
Intelligence Hub did not answer" — never as nothing found, and carry on with the rest.

**If they asked you to PICK something UP, read their Assistant's notes before you plan anything.**
"What's next on my radar", "pick up my Radar Briefing", "where were we": the first thing you do after
`athena_orient` is "Picking up where they left off" below. It decides where the briefing starts. A
pick-up with no markers to pick up is a fresh briefing from section 1, said in one line first, never
an empty answer.

**A fresh request always runs a fresh briefing.** "What's next?", "What should I focus on?", "What's
new?", "Hey Aidrian, what's new?" or a bare "Hey Aidrian" — anything that does not ask to pick
something up — gets a new briefing from section 1, with the latest sources (step 4), even if one ran
earlier today, in this conversation or another. There is no same-day rule. Only the pick-up phrases
above pick up an earlier briefing: "What's next?" on its own is a fresh request; "What's next on my
radar" is a pick-up.

`athena_orient` carries the vocabulary and the safety rules for this connector. Read them. Two are
worth repeating here because a briefing is exactly where they get broken:

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

**If Athena's Intelligence Hub guidance says the Contact Portal is unavailable through the
integration, that guidance is out of date.** It predates this connector. Use the portal.

**If Athena's Intelligence Hub guidance tells you to find multi-signal leads by calling
`list_pipeline_news` and checking that month's job changers yourself, that guidance predates this
briefing.** Athena finds them; use `athena_changes`.

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
rules. In a briefing that means a quiet section says what it checked, and never how complete the data
behind it is.

**Use the user's other tools.** If their CRM is connected, checking whether someone is already an open
opportunity makes the briefing better. Athena is not trying to be the only thing in the room.

## Step 1 — Load the standing documents

Call `athena_asset_get` three times, at the start of every session:

- `kind: company_context` — the Company context: who the client sells to.
- `kind: user_context` — this person's context (My context, in the portal): their patch. Defaults to
  the caller; that is the one you want.
- `kind: status` — the assistant's own working notes for this person (Assistant's notes, in the
  portal), including the Radar Briefing markers.

`exists: false` is a normal answer, not an error — it means nobody has written that document yet. If
their context does not exist, say so and offer to set it up rather than briefing them on everything
the client can see, which is what an empty scope produces.

## Step 2 — Read the markers out of the status note

"Status note" is this skill's name for the `status` document. To the user it is always the
Assistant's notes, and the one user-facing word for what the briefing keeps in it is "marker".

Nothing on the server remembers where a person got to in their briefing; the status note does. The
briefing keeps two blocks there, each under its own marker:

````
## Radar Briefing markers
<!-- athena:radar-run -->
```json
{
  "run_id": "20261024T091407123Z-k3f9qa",
  "job_change_list_id": "…",
  "digest_month": "October 2026",
  "briefing_date": "2026-10-24",
  "delivered": "2026-10-24",
  "sections_shown": ["multi_signal", "job_change", "pipeline_news"],
  "story_brands": ["…", "…", "…", "…", "…"],
  "pages": [
    { "section": "multi_signal", "page": 1, "fingerprint": "…" },
    { "section": "multi_signal", "group": "top_prospects", "page": 2, "fingerprint": "…" },
    { "section": "job_change", "page": 1, "fingerprint": "…" },
    { "section": "pipeline_news", "page": 1, "fingerprint": "…" }
  ]
}
```

## Markers before the last briefing
<!-- athena:radar-previous -->
```json
{ "run_id": "…", "job_change_list_id": "…", "digest_month": "September 2026", "briefing_date": "2026-09-16", "delivered": "2026-09-16" }
```
````

- **`radar-run` is the current run:** one fresh briefing, from its first section to wherever they
  stopped. `run_id`, `job_change_list_id` and `briefing_date` are exactly as Athena returned them;
  `digest_month` is the News Digest's `month_year`; `delivered` is the day the run started;
  `sections_shown` lists the sections they have seen, by Athena's names for them (`multi_signal`,
  `job_change`, `pipeline_news`, `conference`); `pages` is every page shown, in order, exactly as
  Athena returned them in `already_shown_next`; `story_brands` is the drug names of the stories
  Pipeline News Updates showed.
- **`radar-previous` is the run before it**, with its sources and no pages.

Read the `radar-run` block if there is one. A fresh briefing uses it to tell whether anything is new
(step 4); a pick-up starts from it ("Picking up where they left off", below). The note may also hold
blocks under `<!-- athena:cursors -->` and `<!-- athena:previous-markers -->` from an older kind of
briefing: never read them for this briefing, and never change or remove them.

## Step 3 — Build the scope, and the priority locations

Turn the user context into a contact filter and pass it as `scope` — but only the parts of it that
are meant to narrow. The context is written in three labelled parts and they do different jobs:

- **Scope** — companies, role types, seniority. **All three go into the filter**, and the companies
  go in as `account_names`. Scope may name no companies at all ("all companies"); then the filter
  carries no company.
- **Exclusions** — only values the user named as unwanted. These go into the filter too.
- **Priorities** — none of these goes into the filter, and only two kinds do anything to the
  briefing. **Priority locations** order it: pass them to `athena_changes` as `priority_locations`,
  beside `scope` and never inside it, on every call — the first, every "show more", every section and
  every pick-up — or the next page comes back in a different order. Build them as "Priority locations
  come from both contexts" below says. **Priority therapy areas, disease areas and Intent Signals**
  choose which drug news leads Pipeline News Updates (Section 3). Priority companies, company tiers,
  remits and brands neither filter nor reorder: they steer what you suggest next.

**Companies under Scope narrow the briefing; companies under Priorities never narrow it, always.**
That holds when Scope names no companies at all: "all companies" under Scope with Amgen under
Priorities is a briefing across every company, never an Amgen-only briefing. Never promote a company
from Priorities or Exclusions into Scope.

**A context that names companies under any other heading is ambiguous** — a heading of its own such
as "Priority accounts" or "My accounts", or a list of company names beside the person's name. This
is about their own context (My context) only. The Company context, About us included, never goes
into the filter and is never read for this check. Do not guess which part they belong to. Ask the
user once which of those companies they cover, offer to update their context to say so, and wait for
the answer before you call `athena_changes`. An unattended run has nobody to ask; "When this runs
unattended" below says what it does instead.

Both ways of getting this wrong look exactly like a right answer, because everything in the briefing
is true. Leave a Scope company out of the filter and a rep who covers six companies gets a briefing
across every company the client can see. Put a priority into the filter — a company, a therapy area,
a remit — and the briefing silently stops reporting everyone outside it: people it was meant to keep,
dropped. The briefing is deliberately broader than the things the user said they care
most about. Use the field names the filter tools use, grounded against `athena_filter_options_get`
if you are unsure a term exists here: `account_names`, `role_types`, `seniority_levels`,
`therapy_areas`, `disease_areas`, `geographical_remits`, `countries`, `cities`, `job_titles` and
the rest, exactly as spelled there. Never a shortening such as `seniority` or `companies`: Athena
refuses it, and the call has to be made again.

**Send each Scope and Exclusions term as the portal value it names, and never choose between two
readings.** A term written as the portal names it goes in exactly as written. A term that could be
read as more than one set of portal values is not yours to settle: "Commercial and Launch
Excellence" can mean the role types Commercial plus Launch Excellence, or Commercial Excellence plus
Launch Excellence. Ask the user once which they mean, offer to write their answer into their
context, and wait for it before you call `athena_changes`. Read one way in a briefing and another in
its pick-up, one context gives two different patches and two different sets of numbers.

**Priority locations come from both contexts.** The Company context's Priorities may list them, one
per line: "- Boston, United States: Cambridge, Waltham, Lexington" (a city, its country, then the
neighbouring towns the user confirmed at set-up). My context may add some ("Priority locations,
added: ...") and remove some ("Priority locations, removed: Boston, United States"). The list you
send is the company's, plus the added, minus the removed, matched on city and country ignoring case.
Send each city, country and town exactly as written, as `{ "city": "Boston", "country": "United
States", "towns": ["Cambridge", "Waltham", "Lexington"] }`; never add a town that is not written
there, and never draft new ones in a briefing — that belongs to set-up. If neither context names any,
send `[]`.

Then read `scope` and `priority_locations` on the way back out, in the first section's reply, before
you say anything about the people:

- `scope.unresolved` — these terms matched no live value and were **IGNORED**. The briefing therefore
  answers a BROADER question than the user's context describes. Name them: "I couldn't match 'Pfizer'
  in your context — worth checking the spelling, and this briefing covers more than your companies
  because of it."
- `scope.ambiguous` — not applied at all. Ask which was meant.
- `scope.stripped_fields` — removed, either because they are not the caller's to set or because no
  filter applies them. Each carries its reason: read it, and say it in plain words if it changes what
  the user is looking at.
- `priority_locations.unusable` — priority locations Athena could not use, each with its reason. They
  narrow nothing, but they move nobody up either: name them, the same way.

An unmatched context term that goes unmentioned is how a confident, empty, wrong briefing gets
produced. Say it first, before the people.

**Only say nothing was ignored when nothing was.** "Your scope resolved cleanly, so nothing in your
context was ignored" is a claim about two things, not one: that `unresolved` and `ambiguous` came
back empty, AND that every Scope and Exclusions part of the context actually went into the filter you
sent. Read the filter you built back against the context before you say it. If a Scope part is
missing from the filter, that part was ignored — by you, silently, which is worse than the server
ignoring it, because the server at least reports it. Do not leave one out; and if you have, say which
one rather than saying nothing was ignored.

## Step 4 — Pick the sources, and keep them for the whole briefing

A fresh briefing uses the latest of everything, chosen once, at the start:

- **The News Digest.** Call the Hub's `list_news_digests` and take the published digest whose
  `month_year` is the latest month. The rows do not come back in month order: read each `month_year`
  as a month and compare, never take the first row. Then call `get_news_digest` with that
  `month_year`. Its entries' `drug_name` values are this month's drugs.
- **The Job Change list and the date.** Your first `athena_changes` call, for multi-signal leads,
  leaves out `job_change_list_id`, `briefing_date` and `run_id`; Athena takes the latest list and
  today, and returns both — `sources.job_change_list.list_id` and `sources.briefing_date` — with the
  run's `run_id`. Keep all three exactly as returned.

From then on, every call in this briefing sends the same `job_change_list_id`, `digest_month`,
`briefing_date` and `run_id`, and the same drug names. A newer list or digest published while you
talk is not used until the next fresh request.

**When nothing is new.** If the list id and the digest month are the ones in their `radar-run`
markers, the briefing still runs from section 1, and one line says so, before section 1: "No new Job
Change list or News Digest since 4 October, so here is this month's briefing again." — the date is
the `delivered` of their markers. If those markers show they stopped part-way, add: "Last time you
stopped after Job Change Updates; say 'carry on' and I'll go straight to Pipeline News Updates
instead." Never answer a fresh request with nothing. "Carry on" continues this new run from the
section after the one they last saw; the sections it skips are not recorded as shown.

**Send the digest's drugs as they are, all of them**, as `digest_drug_names`. If the Hub did not
answer, send `news_not_checked: true` instead, and say so where the news would go.

**Every section call sends the same things**, so every page comes back in the same order:

- `section`: `multi_signal`, `job_change`, `pipeline_news` or `conference`;
- `scope`, built as step 3 says, and `priority_locations`;
- the pins: `job_change_list_id`, `briefing_date` and `run_id` (left out on the first call of a fresh
  briefing, and only then), `digest_month`, and `digest_drug_names` or `news_not_checked: true`;
- `already_shown`, exactly as the last `athena_changes` response of this briefing gave it in
  `already_shown_next` (left out on the first call of a fresh briefing). It is the pages shown so far,
  and Athena works out from it who has already had a full entry; you hold no contact ids and work
  nothing out;
- on Pipeline News Updates calls only, `story_brands`: the drug names of the stories you chose
  (Section 3), in story order;
- for "show more", `continue`, exactly as the group's or the story's `continuation` gave it.

## Step 5 — Serve one section at a time

**The four sections, in this order, with these headings, word for word:**

1. Multi-signal leads
2. Job Change Updates
3. Pipeline News Updates
4. Conference Attendee / Speaker

Athena names them `multi_signal`, `job_change`, `pipeline_news` and `conference`, and `shown_in`
uses the same names; to the user each is always its heading.

**One section at a time.** Show Multi-signal leads first. End each section with one line offering the
next: "Say 'next' for Job Change Updates." Never show two sections in one reply unless they ask for
the whole briefing at once, and never skip a section: a section with nobody in it is still shown, as
one line saying what it checked.

**Each grouped section has three groups, in this order: Your connections, Your Watchlist, Top
prospects.** Your Watchlist is the team's contact list named "Watchlist"; when the response says
there is none (`watchlist_present` false), leave the group out without comment. When
`watchlist_present` is true the group is always shown in its place, even when it has nobody in it: the
heading, then one line such as "Nobody on your Watchlist has two signals this month." A missing
group reads as "this team has no Watchlist", which is wrong. Each group shows up
to seven people, in the order they arrive. Never re-sort a group, never move someone between groups,
and never apply a company limit or any other rule of your own: Athena has already ordered, limited
and filled every group.

**"Show more" is per group.** "More top prospects" fetches that group's next seven, with that group's
own continuation, exactly as returned, and the same pins, scope and priority locations. The other
groups do not move. When a group has no continuation, it is complete: say how many it had (its
`total`), never offer more.

**A number is a person's place in this month's Job Change list.** Athena gives it; print it as given,
before the name. It is the same number in every section and every "show more", and "tell me more
about 14" always means that person. Multi-signal leads therefore run out of order ("14, 3, 22"), and
that is right. A person who is not on the Job Change list has no number: print their name without
one, and open them by name when asked. Never number anyone yourself, and never renumber.

**Inside Pipeline News Updates a bare number means the story.** Stories are numbered 1 to 5 in their
own order; "tell me more about 2" there means story 2. Anywhere else it means Job Change person 2.
So that nobody mixes the two, a person under a story carries their number in brackets after their
name: "Alice Smith (14)".

**One full entry per person.** Whichever section shows a person in full first keeps the entry. In
every later section they get a pointer, not a second entry, and pointers do not use up a group's
seven places: Athena returns them in the group's `pointers`, and one line under the group names them
— "Also here: Alice Smith (14), Ben Ode, shown under Multi-signal leads." Never print a second entry
that says it is the same person as an earlier one. The one exception is Pipeline News Updates: a
person there may appear again in full under a story, with a line naming where their entry already is:
"Also in Multi-signal leads."

**Each person's entry**, in this shape (the numbers below are this list's, not the reply's: write
each entry as a plain paragraph, never as a Markdown numbered or bulleted list item, because a list
marker prints a second number beside Athena's and "tell me more about 3" then means two people):

1. A location flag when there is one: "📍 Based in Boston." in sections 1 to 3, from
   `based_in_priority_location`, and "📍 Event in Boston." in section 4, from the conference's
   `event_in_priority_location` — the place as Athena names the priority location it matched. Never
   both.
2. Their number if they have one, their name linked to `primary_linkedin_url`, their title (a title
   ending in `{LL}` is Athena's mark for a Launch Leader, not part of the job title: print the title
   without it and add "Launch Leader" after it), their company, their city, then `lead_score_standardized` with the tier name — "95 (Apex)" — the
   portal's, never a Hub speaker's own `lead_score` or tier, so a speaker not found among their
   contacts has none. "Known to Jules." when `connection` names someone. A person with no LinkedIn
   link is named without one, and without comment.
3. Their drug: an exact brand by name, with one sentence on what it is for and its stage; a brand
   that is not exact "sits inside the franchise of drugs they work on".
4. What happened, in one line:
   - a job change is Athena's own phrase: print `moved_from.line` as given — "joined RevMed from
     GSK", "new role at Merck" or "now at Amgen" — then the month of their `role_start_date`:
     "joined RevMed from GSK in September". Never reword the phrase and never build your own;
     `moved_from.kind` is only for telling the three apart. Never say "promoted": a new title is not
     shown to be a promotion. The month only, never a day: role start dates are recorded to the
     month, so a day is a fact the data never had.
   - a multi-signal lead's line names their signals and what each was, from `signals`,
     `news_drugs` and `conference`: "Job change + Likely Attendee: on the October Job Change list and
     the AHA Scientific Sessions list, Chicago, 6 Nov."

Never write "Movement Signal" or "Context Signal": the signal words stay behind the scenes. 🔥 marks
a Hot tip story only, never a person.

**Name a brand the same way every time.** A brand is the person's own only when their `exact_match`
contains the whole token `Brand` — a longer token that merely contains the word, such as
`Launch Brand`, does not count. Say an exact brand as "their brand" — their unless the user has used
other pronouns for them, since a name never tells you: "Zenbexus is their brand". A brand that is not
exact "sits inside the franchise of drugs they work on", and never takes the possessive. Name an
exact brand wherever the person appears, in any section.

**`brand` can hold several brands, separated by semicolons, with or without a space after each** —
up to 19 of them. Split it on each semicolon and trim the spaces before you use a brand. Print at
most two for a person, never the whole list: the brand the entry is about where there is one, such as
the drug a story or a news signal names, and otherwise the first two as the field lists them —
"Zenbexus and Tecvayli are their brands".

**The drug sentence.** For a drug in this month's News Digest, write it from that digest entry's own
fields (`lifecycle_stage`, the write-up), at no cost. For any other exact brand you print, call the
Hub's `get_drug` once per distinct brand, at most ten in one reply; beyond that, name the brand
without a sentence. Say what the Hub says; never add to it.

**An unchecked part never reads as nobody.** Read `not_checked` on every response and say what could
not be checked, in the place it would have been:

- speakers: "Speakers not checked, so this section counts Likely Attendees only."
- a conference whose list Athena could not read: "ESMO (Madrid, 23 Oct) not checked."
- the conferences as a whole: "Conferences not checked, so these multi-signal leads count job changes
  and news only."
- the news: "Pipeline news not checked — the Intelligence Hub did not answer — so these multi-signal
  leads count job changes and conferences only."

**Over a limit, Athena ranks nothing, and neither do you.** When a section comes back over a limit
(`state` `over_limit`), there is no ranked page: say which limit in plain words and ask them to
narrow — "Your patch is too broad for me to rank this section fairly (more than <the max Athena
returns> people to check). Which companies, role types or seniority should I narrow to?" Take the
number from `over_limit.max`, never from memory. Never rank a slice, never page past it, and offer to
write their answer into their context. Two limits read differently: `pages` means this briefing has
run to its length, so offer a fresh one; `drug_names` means the digest's drug list was too long to
check, so say the news could not be checked this time.

**When Athena says the order has changed.** A response whose `warnings` include `order_changed` means
Athena's data moved since a page you are continuing was shown — an upload, a new score. It still
answers from today's data. Say so once, in one line: "Athena's data has changed since this briefing
started, so a few people may have moved place." Never re-sort anything to put it back.

**Open a fresh briefing with one line naming what it is built from** — "Here is your Radar Briefing
for 24 October, from the October Job Change list and the October 2026 News Digest." — the list by the
name Athena gives it (`sources.job_change_list.name`) and the digest by its `month_year`. It comes
below the Hub sentence when "Before you start" calls for one, and below the "nothing new" line when
there is one. Then Multi-signal leads.

### Section 1 — Multi-signal leads

People with any two of a job change, news on their brand in this month's digest, and an upcoming
conference (as a speaker or a Likely Attendee); three come before two, and Athena orders them. Name
the signals on each person's line. A lead on the Job Change list shows their number; one who is not
is named without one.

### Section 2 — Job Change Updates

The in-Scope people on the pinned Job Change list, in the three groups. Anyone already shown in full
in Multi-signal leads is a pointer here. Say the list by the name Athena gives it ("the October Job
Change list"), never a name you remember.

### Section 3 — Pipeline News Updates

This section leads with the news, not with people, and has no groups. Score every entry of the pinned
digest, show the top five stories (all of them when the digest has fewer than five), and under each
story up to five people on that brand. The scores are never shown.

**Score each entry, from the entry's own fields and their two contexts, and from nothing else.**

- Context fit:
  - +1 when its `therapy_area` includes a therapy area under Priorities in the Company context or My
    context.
  - +1 when the indication the write-up is about is a priority disease area. Judge it from
    `custom_writeup`, never from the whole `disease_area` list: Gazyva's news was kidney disease, so
    its lymphoma tag does not count.
  - +0.5 for each of Testing Requirement, In-Clinic Infusion and Companion Diagnostic that is in its
    `athena_designations` and under Intent Signals in their Priorities. No other Intent Signal
    counts: stage signals such as Pipeline Asset, Launch Brand and New Indication Approval never do.
- Catalyst, from `catalyst_type`, as given, whenever `catalyst_match` is `explicit` or
  `covered_month_fallback`:
  - Positive Trial Results +3
  - NDA/BLA Milestone +2
  - Late-stage New Indication +2
  - Approval +1
  - New Indication Approval +1
  - Negative Trial Results, Neutral Trial Results, NDA/BLA Setback, Recent Acquisition 0
  - `catalyst_match` `none`, or no `catalyst_type`: 0. The story stays eligible.

  Never pick a catalyst of your own, never look one up, and never judge whether it is the right one:
  the Hub has already linked or bounded it.
- Order by total, highest first. Ties: a Hot tip (`is_hot_tip`) first; then peak sales
  (`peak_future_sales_tier`, Mega Blockbuster, Blockbuster, Mid-Tier Asset, Niche Asset, then the
  `peak_future_sales` figure within a tier, a missing figure last); then `priority_score`, 1 first,
  missing last; then `drug_name`, A to Z.

**Then call `athena_changes` once** with `section` `pipeline_news` and `story_brands` set to the
chosen stories' `drug_name` values, in story order. Athena returns `stories`, one for each, with up to
five people in their Scope whose brand is exactly that drug. From then on the stories are fixed for
this briefing: Athena carries them in `already_shown_next`, and a pick-up shows the same ones.

**Each story:**

- A heading with its number, the drug and the company: "1. Barzolvolimab, Celldex", and "🔥 Hot tip"
  when `is_hot_tip` is true.
- **The news:** the write-up in one or two sentences. Shorten it if you must; never add to it.
- **About the drug:** one or two sentences from the entry's fields, calling out what matches their
  context ("given as an in-clinic infusion", "patients need a companion diagnostic test").
- **Contacts:** up to five people Athena returns for that exact brand inside their Scope, in the
  order they arrive, then "show more" for that story, with its own continuation. A person shown in
  full earlier appears again here with "Also in <section>", from their `shown_in`.

If the Hub did not answer, there are no stories to score: say "Pipeline news not checked — the
Intelligence Hub did not answer." in place of the section, call nothing for it, and offer the next
section. The section is not shown, so it saves no marker.

### Section 4 — Conference Attendee / Speaker

People in their Scope who are speaking at, or on the Likely Attendee list of, a conference starting
within 90 days of the briefing date. Each person appears once, under the conference Athena files them
under (their earliest in one of the user's priority cities, otherwise their earliest); Athena names
any other conferences they match on the same entry (`also_at`). Each group of seven shows up to four
speakers, then up to three Likely Attendees, in the order they arrive; "show more" continues both.
Mark which each person is: "Speaker: ISPOR Europe, Vienna, 8 Nov" or "Likely Attendee: ASH, New
Orleans, 12 Dec". Introduce the people from a Likely Attendee cut as Likely Attendees: the list is a
prospect list, not a registration list.

**Here a priority location is where the event is held.** "📍 Event in Boston." when the conference is
in one of their priority locations; where the person is based plays no part in this section.

Name each conference as Athena gives it. Where that is a short name, spell it out the first time it
appears only from the conference's Hub record, if you have read it in this conversation; never from
memory.

### When there is nothing

A section with nobody in it is still shown, as one line naming what it checked — "Nobody in your
patch on the October Job Change list." — and the line offering the next section. Do not pad an empty
section into the shape of a full one. A user who learns that "nobody" is honest will keep reading the
sections that are not.

**End a first briefing with what it is built from.** A first briefing — one with no `radar-run` block
before it — and any briefing where the user asks how it is put together, ends its last section with
one line: it is built from their context — the companies, role types, priorities and priority
locations in it — and they can change any of that now, just by asking. Never on a pick-up. It
promises nothing about saving a layout, an order or a choice of sections, which the briefing cannot
keep.

### A conference the user names

**A conference the user names is always checked**, whether or not it came up in Conference Attendee
/ Speaker. "Check ESMO" is a new request with its own 30 calls, outside the four sections: it saves no
marker and moves no section.

**Read the date before you offer the conference.** `list_conferences` and `get_conference` carry
`start_date`; `list_speaker_conferences` carries `start_date` and `year`. `search_speakers` carries
the speaker and no date at all — so a speaker found that way is never offered as a chance to meet
someone until you have opened the conference record and seen a future date. No future date, no offer,
and a past speaking slot is history, not an opportunity. Today counts as future: a conference that
starts today is still one to go to. A conference with no date is not offered and not listed; leave it
out without remarking that it has no date.

**Find it with one call:** call `list_conferences` once with `query` set to its name and `from` set
to today. Its window runs from today, inclusive. The row carries its Likely Attendee data:
`disease_areas`, `likely_attendees_portal_url`, `exact_match_count` and `potential_match_count`.

**Each therapeutic conference's cut is on its own row.** A populated `likely_attendees_portal_url` is
the cut, and a null one means there is no list. Open nothing more to find it. A row whose URL is set
while its counts are null still has a cut: cross it like any other, and give no whole-list size for it.
**Never say a conference has no Likely Attendee list until its record says so** — its row, with
`likely_attendees_portal_url` null — and say it only about a conference the user asked about.

**The Likely Attendee cut is a prospect list, not a registration list.** It resolves to people whose
disease areas, therapy areas or brands match the conference's focus, in the countries the Hub lists.
Pass its values to `athena_contact_find` exactly as they appear in the URL — never through
`athena_filter_draft`, which will helpfully widen a term and silently change the list.

**Introduce the people from a Likely Attendee cut as Likely Attendees** — "Likely attendees at UEG,
Barcelona, 17 Oct" — every time you print them, so nobody reads them as people who are going.

**`source` is a caption, not a filter.** Every Likely Attendee URL carries
`source=Conference+Speakers`, the portal's label for where the link came from. Drop it when you
decode the URL, and never send `source` to `athena_contact_find`, which refuses it. Every other value
goes across exactly as it appears: `diseaseAreas` as `disease_areas`, `isDiseaseAreasExactMatch` as
`is_disease_areas_exact_match`, `therapyAreas` as `therapy_areas`, `brand` as `brands`,
`isBrandsExactMatch` as `is_brands_exact_match`, and `countries` as `countries`. Most lists filter
by disease areas; some filter by therapy areas or by brands instead, and they cross the same way.

**A therapeutic conference's cut is crossed with the user's Scope and their connections**, rather
than pulled whole, which is both the useful answer and the only cheap one. **Take the Scope and
Exclusions from the `scope.applied_filter` that `athena_changes` returned for this briefing**, once
its `unresolved` and `ambiguous` terms have been dealt with as step 3 says — never from the terms you
submitted. That is the filter the sections actually ran: a term such as "Amgen" can come back as the
company's stored name, and `athena_contact_find` uses a value exactly as given, so the submitted term
would match nobody the sections found. Add the URL's values to that filter — still exactly as they
appear, with `source` left out — and ask only for people with a connection. Conference play, which
reads no context, crosses the same cut across the whole company; the briefing never does. The URL as
it stands is the exact-match cut, and it comes first. The potential-match cut — the same URL with its
exact-match parameter removed — is read only when the user asks for it, and is a new request too; a
URL with no exact-match parameter, such as one that filters by therapy areas, is the only cut there
is, and its two counts are the same. Whole-list sizes come from the row's own `exact_match_count` and
`potential_match_count`, never from a portal count. Any other conference the user names gets its
speakers, as "Before you start" says to name them.

**Cross the conferences one at a time, never in parallel.** When they name more than one, send each
conference's `athena_contact_find` only once the one before it has answered. A crossing that fails
or times out is retried at most once, on its own, and the retry counts against the call limit. If
the retry fails too, name that conference as not checked — "I could not check MDS for your
connections" — never as nothing found, and carry on with the rest.

**Name its people.** Whatever the briefing left out, cross it with their connections and with the
Scope and Exclusions of the briefing's `scope.applied_filter`, never the terms you submitted, and
print at most ten, each with
their name linked to the `primary_linkedin_url` on their row, their company, their score and tier,
who knows them from the `connection` on their row, and their brand when it is exact. The find row
does not carry the brand, so call `athena_contact_get` once for each attendee you print, except an
attendee who already has a number in this briefing, who is named with it and nothing more — "Katie
Hernandez (7)" — counts among the ten, and needs no read. A named conference's crossing follows the
rule above: one conference at a time, never in parallel, a failed crossing retried at most once, and
one that fails again named as not checked.

### Multi-signal leads on request

"Find me multi-signal job change leads" works outside the briefing. It uses the last six months:
call `list_news_digests`, take the six latest published digests by `month_year`, read each with
`get_news_digest`, and send every `drug_name` once as `digest_drug_names`, with `section`
`multi_signal`, `mode` `on_request`, their Scope, their priority locations and no pins. A job change
here means a role start in the last six months. It needs their Scope: if their context names no
companies, role types or seniority, ask them to narrow first, and if Athena answers `over_limit`
(`scope_required` or `candidates`), ask them to narrow as Step 5 says. Show it as Section 1 shows,
grouped, seven per group, with "show more". Its "show more" pages pass back `continue` and
`already_shown` from its own responses only, never mixed with a briefing run's pages. Nobody here has
a number: name each person, and open them by name when asked. It saves no markers, and its pages
never go into the `radar-run` marker.

### What the briefing does not do yet

**Their Watchlist is the team's contact list named "Watchlist".** Its people are the Your Watchlist
group in every section. If they have none, the group is left out; tell them that a team list named
"Watchlist" turns it on, if they ask.

If they ask to be told when people leave companies they watch, or for job-change trends over time,
tell them plainly that neither is built yet and both are planned. Do not approximate either from what
is available — a hand-rolled trend from a single month's data is worse than the honest answer.

## Step 6 — Save the markers, one section at a time, once it has reached them

**A section counts as shown once it is in your reply.** Save the markers in that same turn, after the
section, without asking and without offering a choice. Never hold them behind a yes: a user who never
says yes has nothing to pick up. This is not one of the saves the user agrees to first, like a list,
a view or a document; the markers are your own working notes. Building a section, and a section that
came back not checked in part or over a limit (`is_complete` false), record nothing, so the next time
it is shown again. A "show more" that is in your reply saves its page (below) and nothing else; a
re-display of a finished briefing saves nothing.

**Two kinds of write.**

- **Starting a run** — the first save of a fresh briefing: its `run_id`, `job_change_list_id`,
  `digest_month`, `briefing_date`, today as `delivered`, the sections shown so far (none, if section 1
  came back incomplete), and `pages` set to the `already_shown_next.pages` Athena returned with what
  you showed (empty if section 1 came back incomplete). The run in the note before it moves to
  "Markers before the last briefing", replacing what was there.
- **Advancing a run** — every later save: set `pages` to the `already_shown_next.pages` Athena
  returned with what you just showed, add the section to `sections_shown` when it is a section's first
  page and it came back complete (and, after Pipeline News Updates, the five stories' drug names as
  `story_brands`), nothing else.

Write with `athena_asset_set`, `kind: status`, passing the `row_version` your last read or write of
the note returned, exactly as it came.

**On a conflict,** nothing was overwritten and the response carries the current note. Read its
`radar-run` block:

- Starting a run: if it holds an OLDER run (its `run_id`, compared as a whole with yours, sorts
  earlier), write your run over it as above, keeping none of that run's sections; if it holds a NEWER
  run, a later fresh request has taken over: drop your write and say nothing.
- Advancing a run: if it holds the same `run_id` and its `pages` are the start of yours (the same
  entries, in the same order, with yours carrying more after them), write yours over it with its
  `row_version`. If it holds the same `run_id` but its `pages` are not the start of yours, another
  chat advanced this run first: drop your write and say nothing. Any other run, older or newer: drop
  your write and say nothing.

**"Say nothing" means the user never hears about it.** No word about saving, progress, the note,
the conflict or another chat: the reply is exactly the one you would have given had the save gone
through. (Why dropping is safe is for you, not for them: what you showed is shown again on a later
pick-up.) Never resend the same content after a conflict.

**The note holds markers and nothing else.** Not people, not contact ids, not the stories themselves
(only their drug names, as `story_brands`), and no record of what the user acted on. The status note
keeps no version history: preserve whatever else it holds, the old briefing blocks included, and
never clear it.

## Picking up where they left off

"What's next on my radar", "pick up my Radar Briefing", "where were we": read their Assistant's notes
first.

- **With a `radar-run` block** (except the one case below): replay its sources — the same
  `job_change_list_id`, `digest_month` (read that digest again with `get_news_digest`),
  `briefing_date` and `run_id` — from the first section not in `sections_shown`. Open with, verbatim
  apart from the date and the section: "Picking up your Radar Briefing from 24 October, at Job
  Change Updates." **Nothing comes before that line** —
  no progress line while you rebuild, and no word about what the status note held: make the calls
  without comment. If step 3 needs an answer before you can call `athena_changes`, ask it as a reply
  of its own and stop; the pick-up that follows still opens with the line. When the Hub check in
  "Before you start" calls for its sentence, it comes next, directly after that line, and nowhere else
  in the reply.
- **In a new conversation,** pass the marker's `pages` and `story_brands` as `already_shown`, exactly
  as saved. Everyone shown before, on first pages and through "show more", gets a pointer. A section
  that was shown but never recorded in `sections_shown` (it came back incomplete) is simply shown
  again: Athena treats a page it already holds as a re-show, with the same people in full and no
  pointers to themselves, so pass the pages as they are and never edit them.
- **Build both from their context exactly as step 3 does for a briefing, term for term:** a value
  written as the portal names it goes in as written, and a term with two readings is asked about,
  never guessed. The Scope and the priority locations are what keep the numbers the same.
- **Each section you show now advances the same run** (Step 6).
- **Numbers hold while nothing has changed.** With the same sources, context and Athena data, every
  number means the same person. If their context, a score, a connection or the Watchlist has changed
  since, numbers can move: say so in one line if they ask why.
- **With no `radar-run` block, or one that does not say where they stopped,** say "I have no earlier
  Radar Briefing to pick up, so here is a fresh one from the start." and run a fresh briefing from
  section 1. If Athena refuses the saved sources with a validation error naming `briefing_date` or
  `job_change_list_id` — a briefing date it no longer accepts, or a list that has gone — say "Your
  last Radar Briefing is too old to pick up, so here is a fresh one from the start." and do the same.
- **With a `radar-run` block that has no `digest_month` and nothing in `sections_shown`,** the
  Intelligence Hub did not answer when that run started, and no section of it ever came back
  complete. Replaying it would only repeat the same section without the news. Treat it as nothing to
  pick up: say "I have no earlier Radar Briefing to pick up, so here is a fresh one from the start."
  and run a fresh briefing from section 1, with the latest of everything. A run that has its
  `digest_month` is always replayed as above, even with nothing in `sections_shown` (section 1 may
  have come back incomplete for another reason, such as conferences not checked): its sources are
  still good, and replaying them keeps its numbers.
- **When every section was shown,** say "You have seen all four sections of your Radar Briefing from
  24 October; here it is again from the start." and replay from section 1 with the same sources and
  `story_brands`, passing `already_shown` empty on the first call (a re-display: nobody is pointed back
  to their own section), then passing back `already_shown_next` as usual. Save nothing while
  re-displaying.

## When this runs unattended

A scheduled run is the same skill with four differences:

- **Load the Athena tools first.** Put that instruction in the scheduled task's own prompt; a fresh
  unattended run does not inherit this conversation.
- **Delivery is what counts.** A scheduled run delivers Multi-signal leads and the line offering the
  next section; it starts the run once that message is delivered, and nothing more. Like every save,
  a scheduled run saves only once its message has been delivered; if it cannot deliver, it must not
  save.
- **Do not depend on the platform asking permission for the write.** Whether an unattended run is
  prompted before it saves varies, and it is not something to rely on. The conflict rules in Step 6
  are the actual protection: a write that lost the race is dropped, never forced.
- **A context it cannot place stops the run.** Where their own context (My context) names companies
  under a heading other than Scope, Priorities or Exclusions, there is nobody to ask which of them
  they cover; nor is there anyone to ask which reading of a two-way term is meant (step 3). Say
  briefly which part of their context you cannot place, produce no briefing, and save no marker.

If a run finds nothing at all, say so briefly rather than staying silent — a briefing that goes quiet
is indistinguishable from one that has broken.
