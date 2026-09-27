---
name: athena-whats-next
description: Produce the Radar Briefing from Athena — what has changed since the user last looked, as a short pointer line and a few numbered sections of things worth acting on, with drill-down. Use when someone asks what's new, what's changed, what they should do next, who has moved, says "what's next on my radar", or when a recurring briefing run fires.
named-terms:
  briefing: Radar Briefing
  pickup-phrase: What's next on my radar
  note: Placeholders. Written here once so a rename is one edit.
---

# Radar Briefing

A briefing, not a data dump. A handful of numbered items the user could act on this week, each one
traceable back to something that actually changed since the last time they were told.

The engine is `athena_changes`. This skill is what turns its output into something a person reads in
ninety seconds, and — just as important — what keeps the "since last time" honest across runs.

## Before you start

**Check you are live by CALLING a tool, not by looking for one.** On some platforms connector tools
are listed but not loaded, so "I can see the tools" proves nothing either way. Call `athena_orient`.
If it returns, you are connected. If it does not, say so and stop — never produce a briefing from
memory or from an earlier conversation.

**Two parts of the briefing also need the Intelligence Hub:** the conferences (section 2) and the
catalyst lines (section 3). The first `list_conferences` call of section 2 is the check. If the Hub
does not answer, brief from the portal anyway, and say in section 2, and where the catalyst lines
would go, that it could not be checked because the Intelligence Hub is not connected. An unchecked
part must never read as one that found nothing.

**A Hub call that fails or times out is retried at most once, and the retry counts against the call
limit.** That holds for every Hub call, not only the first: a `list_conferences` sweep word, a
`get_conference`, a `list_pipeline_news`. If the retry fails too, name the part that call would have
covered as not checked — "I could not check conferences named Symposium", "catalyst not checked - the
Intelligence Hub did not answer" — never as nothing found, and carry on with the rest.

**If they asked you to PICK something UP, the pick-up gate runs before you plan anything.** "What's
next on my radar", "pick up my Radar Briefing", "where were we" — any of those, and the first thing
you do after `athena_orient` is the gate in "Picking up where they left off" below. It decides
whether a briefing may be produced at all. If the status note holds no markers from before a
previous briefing, the gate FAILS, and a failed gate means you say one sentence and stop — no
`athena_changes` call, no briefing, no caveat. Do not start composing a briefing and then look for a
reason not to send it; the gate comes first, and it is the whole of the answer when it fails.

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
  and say each one's tier as you go — a High at 51 and a Medium at 50 are neighbours.

**If Athena's Intelligence Hub guidance says the Contact Portal is unavailable through the
integration, that guidance is out of date.** It predates this connector. Use the portal.

**Use the portal's words.** The tools and the data call a pharma company an account (`account_names`,
`athena_account_find`); to the user it is a **company**, and an account list is a **Company List**.
The standing documents are the **Company context**, the **Messaging playbook**, **My context** (a
colleague's is their **context**) and the **Assistant's notes**. Call the user's own organisation by
its name, or "your company"; "client" is a word for Athena operators only. **Say Intent Signals,
whatever a field, facet, rule or source calls them.** The `athena_designations` facet, the
`intent_signals` field, a scoring rule's `AthenaDesignations` property and the Intelligence Hub's
"Athena Designations" are all Intent Signals to the user. Tool and field names never change; only
what you say does.

**Say what a section checked when it found nothing. Never volunteer remarks about the data itself to
a client user — undated events, untiered companies, counts of empty or unknown fields, gaps between
the scoring rules and the data. If the user asks, answer plainly.** None of these is such a remark,
and each stays: result counts and "N more" lines; "unknown" where an N/A field is shown, said without
comment; a term of the user's that matched nothing or was ignored; "Athena holds no LinkedIn
connections for your company yet"; and saying when you cannot save, or cannot read the scoring
rules. In a briefing that means a quiet section says what it checked and from when, and never how
complete the data behind it is.

**Use the user's other tools.** If their CRM is connected, checking whether someone is already an open
opportunity makes the briefing better. Athena is not trying to be the only thing in the room.

## Step 1 — Load the standing documents

Call `athena_asset_get` three times, at the start of every session:

- `kind: company_context` — the Company context: who the client sells to.
- `kind: user_context` — this person's context (My context, in the portal): their patch. Defaults to
  the caller; that is the one you want.
- `kind: status` — the assistant's own working notes for this person (Assistant's notes, in the
  portal), including the baseline.

`exists: false` is a normal answer, not an error — it means nobody has written that document yet. If
their context does not exist, say so and offer to set it up rather than briefing them on everything
the client can see, which is what an empty scope produces.

## Step 2 — Read the baseline out of the status note

"Status note" is this skill's name for the `status` document. To the user it is always the
Assistant's notes, and the one user-facing word for the baseline inside it is "marker".

The baseline is a set of **cursors** — one per stream — and it lives in the status note, because
nothing on the server remembers where a person got to.

Keep it in a fenced block with a marker so it survives alongside whatever else the note holds:

````
## Briefing baseline
<!-- athena:cursors -->
```json
[
  {
    "stream_key": "new_arrivals",
    "last_edition_id": null,
    "last_edition_order_value": "2026-08-01T09:14:00Z",
    "label": "briefing of 1 August"
  },
  {
    "stream_key": "job_change_updates",
    "last_edition_id": "…",
    "last_edition_order_value": "2026-08-04T00:00:00Z",
    "label": "August edition"
  }
]
```
````

Read that array and pass it as the `cursors` parameter. If there is no block, omit `cursors`
entirely — the engine runs a first-run default and says so in every group. Do not invent a date to
stand in for a missing baseline: a guessed baseline either buries the user in history or silently
hides things.

**Never rename or invent a `stream_key`.** They are permanent. `new_arrivals`, `role_changes` and
`employer_moves` are fixed; the series keys come from what the tool returns. A key that does not
match reads as a first run, and the user is told everything is new.

## Step 3 — Build the scope

Turn the user context into a contact filter and pass it as `scope` — but only the parts of it that
are meant to narrow. The context is written in three labelled parts and they do different jobs:

- **Scope** — companies, role types, seniority. **All three go into the filter**, and the companies
  go in as `account_names`. Scope may name no companies at all ("all companies"); then the filter
  carries no company.
- **Exclusions** — only values the user named as unwanted. These go into the filter too.
- **Priorities** — therapy areas, disease areas, geographical remits, Intent Signals, brands, and the
  handful of companies that matter most. None of these goes into the filter. Only the companies among
  them order the briefing: people at those companies lead the pointer line and come straight after
  connections in every section, under the one order in step 5. The rest of Priorities neither
  filters nor reorders.

**Companies under Scope narrow the briefing; companies under Priorities only rank it, always.** That
holds when Scope names no companies at all: "all companies" under Scope with Amgen under Priorities
is a briefing across every company with Amgen's people leading, never an Amgen-only briefing. Never
promote a company from Priorities or Exclusions into Scope.

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
if you are unsure a term exists here.

Then read `scope` on the way back out, before you say anything about the report:

- `scope.unresolved` — these terms matched no live value and were **IGNORED**. The briefing therefore
  answers a BROADER question than the user's context describes. Name them: "I couldn't match 'Pfizer'
  in your context — worth checking the spelling, and this briefing covers more than your companies
  because of it."
- `scope.ambiguous` — not applied at all. Ask which was meant.
- `scope.stripped_fields` — removed because they are not the caller's to set.

An unmatched context term that goes unmentioned is how a confident, empty, wrong briefing gets
produced. Say it first, before the items.

**Only say nothing was ignored when nothing was.** "Your scope resolved cleanly, so nothing in your
context was ignored" is a claim about two things, not one: that `unresolved` and `ambiguous` came
back empty, AND that every Scope and Exclusions part of the context actually went into the filter you
sent. Read the filter you built back against the context before you say it. If a Scope part is
missing from the filter, that part was ignored — by you, silently, which is worse than the server
ignoring it, because the server at least reports it. Do not leave one out; and if you have, say which
one rather than saying nothing was ignored.

## Step 4 — Read each group's state before you describe it

An empty group means four different things and only one of them is good news. Read `state`:

- **`has_items`** — the ordinary case.
- **`nothing_new`** — the stream ran and genuinely found nothing. Say so in one line.
- **`no_newer_edition`** — no new edition of that series has been published. Say which edition is
  still the latest, naming it as the group names it — the `title`, and the edition named in the
  group's `summary`, rather than a series name you are carrying from somewhere else. If the user was
  expecting one, add that the published name may have drifted and it is worth flagging to Athena.
  **Never substitute an older edition and present it as new.**
- **`not_computed`** — no longer produced for the three change streams; the server examines a patch
  of any size now. If you still see it, you are talking to an older server: treat it as "unexamined,
  not empty", offer to narrow the scope, and say the connector is behind. Never report it as nothing;
  nothing and unexamined are not the same answer.

Groups carry a `note` when there is something you need in order to read them correctly. Pass one on
only when it changes how the user should read the section — the first-run window, a month-boundary
caveat — and say the first-run window once for the whole briefing, not once per section.

## Step 5 — Render it: one pointer line, then the sections

Open with a single pointer line that says what is in front of them, how much has changed — the count
of numbered items; the people under conferences are not counted — and **which items to start with —
named, in the pointer line itself**: "Here is your Radar Briefing: 11 things since 1 August, and the
two to start with are Omar Mensah's move to Zenas and Anna Weber at EADV." **Name at most three items,
and NAME them.** "The first three are worth your morning" without saying which three is not a pointer
line, and neither is one whose three are revealed at the foot under a closing "where I would start" —
a reader who stops after the first screen has to leave with the pointer. The recommendation goes at
the top, not in a trailer. Where their context names priority companies, the items you name lead with
items at those companies. If the whole briefing is three items or fewer, the people under conferences
included, skip the pointer line and go straight to them. Then the sections, in this order, every
time. A section with nothing in it still appears, as one line saying so and what was checked; an
absent section reads as a system that forgot rather than a patch that was quiet.

1. **Connections who changed role or employer.** First, always. These are people someone at the
   client already knows, and they are the highest-value flags the briefing makes. Read the
   `connection` marker off the item rather than guessing from a name, and say who knows them — it
   carries their names. If the client holds no connection data at all, say that — "Athena holds no
   LinkedIn connections for your company yet" — rather than a line that reads like a quiet month.
2. **Conferences in the next 90 days.** For each therapeutic conference, the people on Athena's
   Likely Attendee cut who are already connections, NAMED — "EADV: Anna Weber, Head of Medical
   Affairs at Galderma, 82 (High), known to Sam Patel" — and for the other conferences,
   speakers in their patch. They are listed without numbers, as the numbering rule below says. How
   to find and read them is under "Section 2" below.
3. **The pipeline edition**, headed by the newest reported edition's own `title`: the people in this
   user's patch that it carries — and any older edition reported with it — each with their brand and
   a catalyst line ("Section 3" below).
4. **Recent job changes.** Everyone else who changed job: every job-change edition reported, role
   changes and employer moves together, one entry per person ("Sections 1 and 4" below). Say where an employer
   move came FROM; that is often the most useful half of the flag.
5. **New arrivals.** The lowest-value flag, and last for that reason. Under its heading, every time
   the section appears — pick-ups included — say this line: "New contacts added to the Athena
   database that you should be aware of."

**Four headings are fixed text; one comes off the wire.** Print "Connections who changed role or
employer", "Conferences in the next 90 days", "Recent job changes" and "New arrivals" exactly as they
stand above. Head section 3 with the `title` of the newest pipeline edition group in the response,
copied character for character.

**Find the edition groups by `stream_key`, never by title or position.** The pipeline edition groups
are every group whose `stream_key` is `pipeline_launch_brand_prompts`; the job-change edition groups,
which go into sections 1 and 4, are every group whose `stream_key` is `job_change_updates`. Stream
keys are permanent; titles are server configuration and can change.

**A series can return several edition groups: collect every one.** The engine returns one group for
each edition it reports, all with the series' `stream_key` — two when the user missed a month, up to
six after a long absence. Stopping at the first drops a whole edition's people and its continuation.
A series' groups arrive newest edition first: that order is how you tell which edition is newer,
never a date read out of a title. Where a group carries
`result_truncated.edition_coverage.newest_reported`, it names the newest and should agree; the order
does not depend on it.

**Never carry a series name in your own words.** The series, and the editions they publish, are
server configuration rather than something you know: a phrase you remember is a phrase that can be
renamed underneath you, and a section headed by a name the data has stopped using is one the user
cannot match to anything. Read section 3's heading off its group every time, even when you are sure
you know it.

However a heading was arrived at, it does not move afterwards. Do not append a qualifier ("… matched
to your patch"), do not re-word one for the second telling, and give a pick-up the same five headings
as the briefing it reproduces. A pick-up whose sections are named differently cannot be checked
against the briefing it claims to be reproducing, which is the only thing the user has to go on.

**Number the headline items. Always.** The headline items are the engine's: everything in the
connections section, the pipeline edition section, "Recent job changes" and "New arrivals". Number
them straight through those four sections in section order, 1 upward, so "tell me more about 2"
resolves to one thing — there is no way for the user to click one. Keep that numbering for the rest
of the conversation. Each item carries an `item_id` if you need to be exact about which one you mean.

**The conference section's people are named, never numbered.** List them under their conference
without a number, and open one by name when asked — "tell me more about Anna Weber". That section is
read live from the Intelligence Hub, so on a pick-up it can come back different, shorter or not at
all; because none of it carries a number, none of that can move an engine item's number.

**One order, in every section.** Connections first; then people at the companies under Priorities
in their context; then `lead_score_standardized`, highest first; then date, most recent first, with
undated items last; then `contact_id`. Inside the connection and priority-company groupings the score
does the ranking, and the tier stays a label beside it. In the numbered sections the date is the
item's own, from `athena_changes`, never a catalyst's: nothing the Hub returns may move a numbered
item. Apply the order to everything you have loaded and not yet printed, not only to the first page,
so the first telling, "the next ten" and a pick-up — which reloads the same items — all number the
same way.

For each item:

- what changed, in one line, in the user's own vocabulary;
- who it is about and where they work, with their name linked to their LinkedIn profile when the
  record carries one — `primary_linkedin_url` on a portal item or row, the LinkedIn URL on a Hub
  speaker. A person with neither is named without a link, and without comment;
- their `lead_score_standardized`, with their `lead_score_tier` name beside it;
- their brand, when it is exact (below);
- why it is worth their time this week.

**Name a brand the same way every time.** A brand is the person's own only when their `exact_match`
contains the whole token `Brand` — a longer token that merely contains the word, such as
`Launch Brand`, does not count. Say an exact brand as "her brand" — his or their, whichever fits the
person; their when you cannot tell: "Zenbexus is her brand". A brand that is not exact "sits inside
the franchise of drugs she works on", and never takes the possessive. Name an exact brand wherever
the person appears, in any section. Name a brand that is not exact only in the pipeline edition
section. Briefing items carry `brand` and `exact_match`; a Likely Attendee comes from a find row,
which does not, so section 2 reads them for the attendees it prints.

**`brand` can hold several brands, separated by `; `** — a semicolon and a space, up to 19 of them.
Split it on that separator before you use a brand. Print at most two for a person, never the whole
list: the brand the item is about where there is one, such as the one its catalyst line names, and
otherwise the first two as the field lists them — "Zenbexus and Tecvayli are her brands".

**Say the date the data supports, and no more.** Each kind of change carries its own date field, and
they are not equally precise:

- **Role changes are month-granular.** `role_start_date` arrives carrying a day, and that day is an
  artefact of the storage: role start dates are RECORDED to the month, and the comparison that
  selected the change ran at month boundaries. Say the month and nothing finer — "now Chief Medical
  Officer at Zenas, started in September". Never "on 8 September", never "from 23 September". A day
  the source cannot support is a fact the user may act on, and it is one the data never had.
- **Employer moves carry a real date, so give it.** `changed_at` is when the move was recorded.
  Render a move as **"moved from X to Y, now <title>, on <date>"** — "Rohan Reyes moved from Intellia
  to 89bio, now Director, Patient Marketing (HAE), on 14 September". The from, the to, the title and
  the date, every time. Dropping the date leaves the user unable to tell last week's move from last
  quarter's, and the field is on the wire whether or not you print it.
- **New arrivals carry `arrived_at`**, the day the contact appeared. A day is right here.

**Print at most five items in a section, but keep the rest.** The tool returns up to ten per group.
When you print five, hold the others in this conversation and say how many more — "five more, say
the word". In a section built from one group, "N more" is counted from what the group says is
available, minus what you have printed; a section built from several groups — sections 1 and 4
always, section 3 when it covers more than one edition — counts as "Sections 1 and 4" below says.
Never characterise people you have not been given.

**Keep track of what you have PRINTED, and never print it twice.** Two lists, both held in this
conversation and neither written anywhere: the items you have already printed, and the items the tool
returned that you have not. When they ask for more — "say the word", "the next ten by score", or
anything of that shape — serve the RETURNED-BUT-UNPRINTED queue first, in the one order above, and
fetch more only once that queue is empty. Otherwise people six to ten are skipped, because the
continuation starts after the ten the tool has already given you.

Number on from the last number you used: item 11 follows item 10, and the numbering keeps climbing
for the rest of the conversation. An item you printed earlier in this conversation is never one of
the next ones — not renumbered, and not with a note saying it is a repeat. A repeat is not "the next
ten"; it is a shorter answer than the one they asked for. If the queue and the continuation together
run out before you reach the number they asked for, print what there is and say that is all of it.

**Fetch more only where there is more.** A group whose `result_truncated` carries a populated
`continuation` has more behind it: fetch the rest when asked with `continue_group_id` and
`continue_offset`, taken exactly from that continuation, rather than re-running the whole report.
`result_truncated` on its own does not mean there is more — an edition group carries it to describe
the edition's backlog even when nothing further can be fetched. When the queue and every continuation
are exhausted, say that is everyone.

For the pipeline edition, both numbers are worth saying: how many of the edition's people are in
this user's patch, and how big the edition was. "Three of the 180 people in the August list are
yours" reads correctly; a bare "three" makes the edition sound tiny. When the section covers more
than one edition, give each edition its own pair, an older one's in the line that names it.

**End a first briefing with what it is built from.** A first briefing — one run with no saved
baseline — and any briefing where the user asks how it is put together, ends with one line: it is
built from their context — the companies, role types and priorities in it — and they can change any
of that now, just by asking. Never on a pick-up. It promises nothing about saving a layout, an order
or a choice of sections, which the briefing cannot keep.

### Sections 1 and 4: one job change, one entry

Both sections are built from the same three sources: the job-change edition — every group whose
`stream_key` is `job_change_updates`, one for each edition reported — then `role_changes` and
`employer_moves`. One person can be in several of those groups, and each group gives them a
different `item_id`, so:

- **One person, one entry.** Merge every group from the three sources on `contact_id`. When a person
  is in more than one, the employer move wins, then the role change, then the edition; between two
  editions, the newer edition's item wins. The winning item keeps its own `item_id` and its own
  group, for drill-down and for fetching more.
- **Who goes where.** A person who carries a `connection` goes in section 1; everyone else goes in
  section 4. Nobody appears in both.
- **How each one reads.** An employer move and a role change read as the date rules above say — the
  move with its from, to, title and date, the role change with its month. An edition-only person
  reads "now <title> at <company>": the edition carries no date and no previous employer, so give
  neither.
- **How many more.** Say "at least N more", N being the distinct people you have loaded and not
  printed — never the groups' totals added together, because one person can sit in several groups.
  Offer to fetch beyond them only when some group from the three sources carries a populated
  `continuation` in its `result_truncated`.
- **The next ten.** Serve the loaded queue first. Then fetch every group whose `continuation` is
  populated — each edition group, role changes, employer moves — at exactly the group and offset it
  returned; merge what comes back with the same winner, leave out everyone already printed, order it
  the one way above and number on. When the queue and every continuation are exhausted, say that is
  everyone.
- **Printed once.** A person printed from an edition who turns up later as an employer move, or in
  another edition, is not printed again; where it is a move, say where they moved from when they are
  next opened.
- If the job-change series reports `no_newer_edition`, say so in section 4, as step 4 says.

### Section 2: conferences in the next 90 days

**Read the date before you offer the conference.** `list_conferences` and `get_conference` carry
`start_date`; `list_speaker_conferences` carries `start_date` and `year`. `search_speakers` carries
the speaker and no date at all — so a speaker found that way is never offered as a chance to meet
someone until you have opened the conference record and seen a future date. No future date, no offer,
and a past speaking slot is history, not an opportunity. A conference with no date is not offered and
not listed; leave it out without remarking that it has no date.

**Find the conferences.** How depends on what `list_conferences` offers, so read its parameters
rather than assuming:

- If it takes a date range or an "upcoming" parameter, ask it for the next 90 days.
- Otherwise, call it once for the current year with `limit` 100. It returns that year's conferences
  in date order from January, so if its last row starts after the window's end, that one call covers
  the window. If it does not — in the autumn, when more than 100 conferences come first — call it
  again once for each of the words Congress, Meeting, Summit, Week, Symposium, Sessions and
  Conference (`query`, the current year, `limit` 100) and merge the rows on `id`. When the window
  runs past the year end, add one plain call for the next year, `limit` 100.
- Check every call the same way: it covers its part of the window when its last row starts after the
  window's end, or it returned fewer than 100 rows. If a call stops short and there is no sweep for
  it, say so — "I could only see conferences up to <date>" — rather than implying you checked
  everything.

Keep the conferences whose `start_date` falls in the next 90 days.

**Therapeutic conferences are read for their Likely Attendee cut; the rest keep their speakers.** A
conference whose `event_type` includes Therapeutic Conference is crossed with the user's
connections. Any other conference gets speakers in their patch, from at most three
`list_conference_speakers` calls across the section.

**Where each therapeutic conference's cut comes from:**

- If the list rows carry the `likely_attendees_portal_url` key, the row is enough: a populated value
  is the cut, and an explicit null means there is no list. Open nothing more.
- If the rows do not carry the key, open `get_conference` for the therapeutic conferences only,
  soonest first, at most six. The ones after the sixth are not opened.

**Never say a conference has no Likely Attendee list until its record says so** — the list row that
carries the key, or the conference record you opened — and say it only about a conference the user
asked about. In the briefing, a conference without a cut is simply not mentioned in the Likely
Attendee part.

**The Likely Attendee cut is a prospect list, not a registration list.** It resolves to people whose
disease areas match the conference's focus, in the countries the Hub lists. Pass its values to
`athena_contact_find` exactly as they appear in the URL — never through `athena_filter_draft`, which
will helpfully widen a term and silently change the list — and cross it with the user's connections
rather than pulling the whole cut, which is both the useful answer and the only cheap one. The URL as
it stands is the exact-match cut, and it comes first. The potential-match cut — the same URL with the
disease-area exact-match parameter removed — is read only when the user asks for it. Whole-list sizes
come from the Hub's own `exact_match_count` and `potential_match_count`, never from a portal count.

**Name the people rather than counting them.** Print at most five people in this section, Likely
Attendees and speakers together, in the one order above — where a person's date is their
conference's, soonest first — each under their conference and without a number, and any more you
are asked for the same way. For each attendee you print: their name, linked to the
`primary_linkedin_url` on their row; their company; their score and tier; who at the client knows
them, from the `connection` on their row; and their brand when it is exact. The find row does not
carry the brand, so call `athena_contact_get` once for each attendee you PRINT, and for nobody else.

**Say what the section checked.** Name the conferences you read — "I checked EADV, EURETINA, MDS,
NACFC, ACG and ECNP" — and, where the cap left some unopened, name them with their dates: "Not
opened yet: ESMO 23 Oct, ASH 12 Dec - ask and I'll check them."

**The section stops at 30 calls.** Every call made for this section counts, Hub and portal together,
and 5 of the 30 are kept for the attendees' `athena_contact_get` reads. Stop reading conferences once
25 calls are used, print the attendees already crossed with their brands, and name what you did not
check, with dates, the same way as the unopened ones.

**A conference the user names is always opened.** "Check ESMO" is a new request with its own 30
calls: open it whatever the cap, cross it with their connections and name its attendees the same way —
print at most ten, each with its `athena_contact_get` read. A potential-match cut they ask for is a
new request too.

### Section 3: the pipeline edition's brands and catalysts

**Section 3 draws on every `pipeline_launch_brand_prompts` group in the response.** Head it with the
newest reported edition's `title`. Merge the groups on `contact_id`; where a person is in more than
one edition, the newest edition's item wins and keeps its own `item_id` and group. Under the heading,
say one plain line naming each older edition it also covers, from that group's own `title`: "This
also covers the August edition, which you had not seen." Order, "at least N more", the next ten and
printing once then work as in sections 1 and 4, every group continued at exactly its own group and
offset.

Each person printed in section 3 gets their brand — exact or not, in the shapes above — and, where
the Hub has one, a single catalyst line: "Zenbexus is her brand - FDA approved Zenbexus for multiple
myeloma on 13 August."

- **One call per brand, five at most.** Call `list_pipeline_news` with `drug_name` set to a single
  brand and `limit` 10, and WITHOUT `since`: `since` silently drops every item with no `article_date`,
  and undated items are common. A single brand, because the whole `brand` field matches nothing once
  it holds several. Take each printed person's first brand — the text before the first `; ` — in
  print order, skipping one already called. Where a person carries several brands and none has given a catalyst yet, call their
  others while calls remain, and name the brand that has one — otherwise their first.
- **Choose the item.** From the rows returned, take the most recent item dated within 60 days of the
  day you produce the briefing. If there is none, take the first undated item in the order the Hub
  returned, show it without a date and do not call it recent. Otherwise — the call answered and
  nothing in it qualifies — there is no catalyst line for that brand; say nothing in its place. A
  call that did not answer is the Hub-failure rule under "Before you start", never this case.
- **Say what the Hub says.** Shorten its summary if you need to; never add to it. No web research.

### When there is nothing

Say it in one line, name what was checked and from when, and offer the standing alternatives — asking
the data directly, or looking at an upcoming conference. An empty Radar Briefing is still a Radar
Briefing; say which it is so the next one is recognisable. Do not pad an empty result into the shape of
a briefing. A user who learns that "no news" is honest will keep reading the ones that are not.

### Two sections that do not exist yet

If the user asks for their favourites, or for job-change trends over time, tell them plainly that
neither is built yet and that both are planned sections of this briefing. Do not approximate either
from what is available — a hand-rolled trend from a single month's data is worse than the honest
answer.

## Step 6 — Save the baseline, and only after the user has it

`next_cursors` in the response is what the baseline SHOULD become. Three rules govern writing it back,
and all three exist because getting this wrong loses a briefing nobody ever saw:

1. **Advance only on confirmed delivery.** Write the cursors after the briefing has actually reached
   the user — not when the report was generated. A run that fails, errors, or is cut off before the
   user sees anything must leave the baseline exactly where it was.
2. **Merge stream by stream.** Update only the streams present in `next_cursors`, leaving every other
   entry in the note untouched. A scheduled run and a conversation happening the same morning must not
   overwrite each other's progress, and a wholesale replacement is how one of them does.
3. **Retrying is safe; re-reporting is not a failure.** If you are unsure whether the last run's
   cursors were saved, run again with the baseline you have. The same edition reported twice is a
   small annoyance. An edition marked seen and never shown is gone.

Write it back with `athena_asset_set`, `kind: status`, passing the `row_version` from the read at step
1 exactly as it came. On a conflict, nothing was overwritten and the response carries the current
document — merge your cursor block into THAT and call again with its `row_version`. Never resend the
same content after a conflict.

**The status note keeps no version history.** Anything you remove from it is gone. Preserve whatever
else the note holds when you write the cursors back, and never clear the note as part of a briefing.

If `next_cursors` came back with `unrecognised_cursor_keys`, an old key is sitting in the note. Drop
those entries when you write back and mention it once, calling the note their Assistant's notes.

**Keep the markers you just moved off.** Alongside the new cursors, record the `baseline_used` values
each group reported for this briefing, and when you delivered it. Those are the markers as they stood
BEFORE this briefing, and they are what a pick-up replays from. Save what `baseline_used` says, not
the clock: the clock slides, and a replay from a sliding window is a different briefing.

**The note holds markers and nothing else.** Not items, not contact ids, not summaries of what you
said, and no record of what the user acted on. It is a set of cursors and two timestamps. Anything
more is a small database inside a text file, and it will drift from the truth the moment the data
moves.

## Picking up where they left off

"What's next on my radar" — or anything that means "let's pick up on my Radar Briefing" — is NOT a
request for a new briefing. Work steps P1 and P2 below IN ORDER. P1 is a gate, not advice: until it
passes you may not call `athena_changes`, and you may not write a briefing.

### Step P1 — The gate: is there a previous briefing to pick up?

Do these five things, in this order, before anything else:

1. **Read the status note first.** Call `athena_asset_get`, `kind: status`, and read it before you
   plan a briefing, build a scope or call `athena_changes`. A pick-up starts by finding out whether
   there is anything to pick up.
2. **Look for the markers from before the last briefing.** That is the block step 6 writes when a
   briefing is delivered: the `baseline_used` values each group reported, with the date you delivered
   it. The current cursor block (`<!-- athena:cursors -->`) is a DIFFERENT thing and finding it is
   not a pass — current markers on their own mean a briefing was delivered by something that did not
   record what it replayed from, so there is still nothing to replay.
3. **If that block is absent, the gate has FAILED. Say exactly this, and nothing else:** "I have no
   previous Radar Briefing to pick up. I can run a fresh one now - it will look back 30 days. Shall
   I?" Then END THE TURN and wait for their answer.
4. **A failed gate is the whole of your answer.** No `athena_changes` call, no scope, no pointer
   line, no sections, no items, and no caveat around a briefing you delivered anyway. Delivering one
   and explaining the gap afterwards is the failure this gate exists to stop: they asked to pick
   something up, so a briefing that arrives reads as the one they were picking up, and nothing you
   write around it tells them otherwise. Offering and stopping is not an unhelpful answer — it is
   the correct one.
5. **Only if that block is present** do you carry on to step P2. If they answer your offer with a
   yes, that is a NEW briefing: go to step 1 at the top of this skill and run it as a first briefing,
   not as a pick-up — no pick-up opening line, and the baseline advances on delivery as normal.

**If there are no earlier markers, offer a briefing — do not deliver one.** Do not infer the
first-run window and produce a full briefing unasked, however well you explain it afterwards.

### Step P2 — Rebuild the last briefing and work it with them

- **Open with this line, in these words.** The first thing they see is, verbatim, carrying the date
  of the briefing you are picking up: "Picking up your Radar Briefing from 16 September - I have not
  moved your marker, so nothing here counts as read". Then the invitation, also verbatim: "tell me
  what is done and I will leave it out for the rest of this conversation". Conveying the same two
  facts in your own words is not enough. "Nothing here counts as read" is the half that explains why
  the same items are in front of them again, and "marker" is the one user-facing word for the
  baseline. The date is the only part that changes.
- **Do not advance the baseline.** Nothing new is being reported, so nothing new has been delivered.
  The cursors stay exactly where they are, and say so: the marker has not moved.
- **Same sections, same headings, same numbers.** Head the sections as step 5 says — the four fixed
  headings, and the newest reported pipeline edition's own `title` off this re-run, with the same
  line naming any older edition the section covers — and give the engine's items the numbers the
  briefing gave them. Renaming a section, or renumbering, makes a pick-up impossible to check against
  the briefing it is reproducing. The conference section's people are named, never numbered, so a
  conference part that comes back different, shorter or not at all moves no number. The New arrivals
  line stays, because it belongs to that section; the closing line about their context does not,
  because it belongs to a first briefing.
- **Rebuild it, do not recall it.** Call `athena_changes` again with the markers you saved before the
  last briefing and the same scope. The engine is stateless, so the same inputs give the same engine
  items and the same numbers. Present it as a refresh rather than a guaranteed replay:
  anything published or changed since will show — the conference crossings and catalyst lines are
  read live from the Hub, so they can differ — and a one-off cut you made in conversation cannot be
  recovered; say so if they ask for one.
- Work down the items they have not dealt with, and offer the next tranche in the one order step 5
  sets when they ask for more, using the continuation the re-run returns — the same printed /
  unprinted discipline as step 5, and the same numbering carried on.
- **Be honest about the limit.** Athena cannot know what they acted on. Unless the outreach happened
  through this assistant, nothing recorded it. Ask rather than assume, and never present a guess as
  memory — it is an invitation to tell you, not an apology.

## When this runs unattended

A scheduled run is the same skill with four differences:

- **Load the Athena tools first.** Put that instruction in the scheduled task's own prompt; a fresh
  unattended run does not inherit this conversation.
- **Delivery is what counts.** The cursors advance only if the briefing actually lands in front of
  the user. If the run cannot deliver, it must not save.
- **Do not depend on the platform asking permission for the write.** Whether an unattended run is
  prompted before it saves varies, and it is not something to rely on. The status-note write is safe
  to repeat, which is the actual protection.
- **A context it cannot place stops the run.** Where their own context (My context) names companies
  under a heading other than Scope, Priorities or Exclusions, there is nobody to ask which of them
  they cover. Say briefly that their context names companies under a heading you cannot place,
  produce no briefing, and advance no marker.

If a run finds nothing at all, say so briefly rather than staying silent — a briefing that goes quiet
is indistinguishable from one that has broken.
