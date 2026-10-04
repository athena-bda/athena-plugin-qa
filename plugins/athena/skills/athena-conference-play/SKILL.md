---
name: athena-conference-play
description: Turn a conference into a working list of people worth meeting — Athena's conference and speaker intelligence joined to the client's own contacts, as a saved list or view. Use when someone mentions a conference by name, asks who is speaking, asks who they should meet at an event, or asks what is coming up.
---

# Conference play

A conference is a rare thing in this business: a date, a place, and a few hundred named people who
will all be in the same building. This skill turns one into a short list of people this client should
try to meet, and leaves that list somewhere they can use it.

It spans **both** Athena connectors:

- **Intelligence Hub** — the conferences themselves, their agendas, pricing tiers, sponsors and full
  speaker rosters.
- **Contact portal** — this client's own contacts, their scores, and the lists and views the plan gets
  saved into.

## Before you start

**Check you are live by CALLING a tool on each connector, not by looking for one.** On some platforms
connector tools are listed but not loaded, so an absent tool proves nothing. Call `athena_orient` for
the contact portal, and the Hub's `whoami` for the Hub — it also tells you whose Hub it is (below).

If only one answers, say which half you have. A conference play with the Hub alone gives speakers but
no idea who this client already knows; with the portal alone it gives contacts but no conference. Both
are still useful; neither should be presented as the whole play.

**A Hub call that fails or times out is retried at most once, and the retry counts against the call
limit.** That holds for every Hub call, not only the first: the `list_conferences` call, a
`get_conference`, a `list_conference_speakers`. If the retry fails too, name the part that call would
have covered as not checked — "I could not check the conferences in the next 90 days", "I could not
check the speakers at BIO-Europe" — never as nothing found, and carry on with the rest.

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
once:** at the top of the answer, before the first conference or person. Nowhere else: not in a
progress line while you work, not a second time further down, and never only in your own
reasoning — a sentence you only thought has not been said.

**Whether or not the Hub matches, every score, tier and "who knows them" comes from the contact
portal.** Never use the Hub's speaker scores, tiers or connections — the `lead_score`,
`lead_score_tier` and `connection` on any Hub record, a speaker, a roster or a prompt list — not
even when the Hub is signed in to this same client: they are the Hub's own, and a person given one
score by the Hub and another by the portal has two. Name a Hub speaker with their company and
conference, and the LinkedIn link their Hub record carries — a public profile link, not the Hub
client's data — and nothing else, unless you have found them among this client's contacts: then
their score, tier and "who knows them" are that contact's, read from the portal, and since the match
is made on name and employer, say they look like the same person.

`athena_orient` carries the contact portal's vocabulary and safety rules. Two matter here:

- **Scores have three states, and one number.** `lead_score_standardized` is the only lead score you
  will see or say. `lead_score_standardized` absent means not scored on this client's standardised
  scale. `lead_score_standardized` 0 with a tier is a REAL score, at the bottom of the ranking.
  `lead_score_standardized` 0 with `lead_score_tier` "N/A" means one of this client's own scoring
  rules ruled the person out: a value they marked as unwanted, with nothing scored to outweigh it.
  Say "ruled out by your scoring rules", never "unscored", and keep them out of priority lists. A
  ruled-out person's score is 0 too, so the TIER is the only thing that tells the two apart. N/A on a
  data field — therapy area, remit, and so on — is a different thing entirely: it means Athena has no
  information. Say "unknown", and never exclude anyone on it. Never rank a ruled-out person back into a target list.
- **Tier names only**, exactly as they arrive. Never turn one into a number, a band or a percentile,
  and never recompute a score. A tier is a LABEL, never a filter of its own: prioritise by
  `lead_score_standardized`, present the tier name beside it, and never make a tier the sole reason
  to include or leave someone out. "Highest priority" means the top of the score ordering, not one
  named tier. When someone asks for a count, walk down the score ordering until you have that many
  and say each one's tier as you go — a High at 80 and a Medium at 79 are neighbours.

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
rules. In a conference play that means saying which conferences you checked, and never how complete
their records are.

## Step 1 — Find the conference

A conference the user names is found with one `list_conferences` call, `query` set to its name —
with `from` set to today when they mean the next one, or `year` when they name the year. For "what's
coming up", or any stretch of dates — "what was on in March 2026?" — call `list_conferences` once
for that window, with `from` and `to` set to its first and last days, both as YYYY-MM-DD, and
`limit` 100. "Coming up" means the next 90 days, from today, inclusive — `from` today and `to`
today plus 90 days; a conference that starts today is still coming up — and a window in the past is
answered as history, never as something to go to. That one call is the whole discovery: no call by
year and no name search to fill the window out. The response does not say whether there were more,
so if it returns all 100 rows, say so — "The Hub gave me its limit of 100 conferences for those
dates, so there may be more I could not see" — rather than implying you saw everything.

Every row carries the dates, the city and country, the disease-area tags and the conference's Likely
Attendee data (step 2), which is what to get before anything else: half the value of this play is
telling someone early enough that they can still book. Open `get_conference`, by the `id` on its row,
only for what the row does not carry: the overview, the sponsors, and the therapy-area tags step 3
needs when there is no published cut.

**Read the date before you offer the conference.** `list_conferences` and `get_conference` carry
`start_date`; `list_speaker_conferences` carries `start_date` and `year`. `search_speakers` carries
the speaker and no date at all — so a speaker found that way is never offered as a chance to meet
someone until you have opened the conference record and seen a future date. No future date, no offer,
and a past speaking slot is history, not an opportunity. Today counts as future: a conference that
starts today is still one to go to. A conference with no date is not offered and not listed; leave it
out without remarking that it has no date.

`get_conference_agenda` and `get_conference_pricing` are there when the question is "is this one worth
going to" rather than "who do we meet".

## Step 2 — Use Athena's own join if it exists

Athena's own join between a conference and the portal is **`likely_attendees_portal_url`**, and it
is on every `list_conferences` row, beside `exact_match_count` and `potential_match_count`: a
populated value is the cut, and a null one means there is none. Open nothing more to find it. A row
whose URL is set while its counts are null still has a cut: cross it like any other, and give no
whole-list size for it.

When it is populated, it is the single most valuable field Athena has for the conference: Athena's
own team has hand-authored the contact-portal cut for this conference, and it defines two audiences —

- **Exact-Match Prospects** — the URL as it stands. The narrow cut: people Athena has verified as
  working on the relevant disease areas or brands.
- **Potential-Match Prospects** — the same URL with its exact-match parameter removed. The broader
  cut: people who plausibly work in the area, inferred from their franchise.

A URL with no exact-match parameter, such as one that filters by therapy areas, is the only cut
there is: its two counts are the same, and there is no broader list to offer.

Prefer this over anything you construct. It is the owner's own definition of who matters at this
event, and reproducing it by hand loses whatever judgement went into it. Work the Exact-Match
Prospects first; read the Potential-Match Prospects only when the user asks for them.

**Decode it verbatim.** Pass the URL's values to `athena_contact_find` exactly as they appear, keeping
its exact-match flag as it stands. Do not route them through `athena_filter_draft`: it will widen a
term to a longer one it recognises — "Urticaria" becomes "Chronic Spontaneous Urticaria" — and
silently hand you a different list from the one Athena published.

**`source` is a caption, not a filter.** Every Likely Attendee URL carries
`source=Conference+Speakers`, the portal's label for where the link came from. Drop it when you
decode the URL, and never send `source` to `athena_contact_find`, which refuses it. Every other value
goes across exactly as it appears: `diseaseAreas` as `disease_areas`, `isDiseaseAreasExactMatch` as
`is_disease_areas_exact_match`, `therapyAreas` as `therapy_areas`, `brand` as `brands`,
`isBrandsExactMatch` as `is_brands_exact_match`, and `countries` as `countries`. Most lists filter
by disease areas; some filter by therapy areas or by brands instead, and they cross the same way.

**Cross it with the client's connections first.** The whole cut can be tens of thousands of people and
counting it is slow enough to time out; the same cut restricted to people the client already knows
comes back immediately and is the more useful answer anyway. When you need the size of the whole list,
take it from the row's own `exact_match_count` and `potential_match_count` rather than counting in
the portal.

**Cross the conferences one at a time, never in parallel.** Take them in date order, soonest first,
and send each conference's `athena_contact_find` only once the one before it has answered: crossings
sent together can fail where the same crossings sent one after another answer. A crossing that
fails or times out is retried at most once, on its own, and the retry counts against the call limit.
If the retry fails too, name that conference as not checked — "I could not check MDS for your connections" — never as
nothing found, and carry on with the rest.

**Introduce the people from a Likely Attendee cut as Likely Attendees** — "Likely attendees at UEG,
Barcelona, 17 Oct" — every time you print them, so nobody reads them as people who are going.

**Name the people rather than counting them.** For one conference, print at most ten attendees, by
`lead_score_standardized`, and hold the rest. For each one you print:

- their name, linked to the `primary_linkedin_url` on their row — a person without one is named
  without a link, and without comment;
- their company, and their score with the tier name beside it;
- who at the client knows them, from the `connection` on their row;
- their brand, when it is theirs. The find row does not carry the brand, so call
  `athena_contact_get` once for each attendee you PRINT, and for nobody else. A brand is theirs only
  when their `exact_match` contains the whole token `Brand` — a longer token that merely contains the
  word, such as `Launch Brand`, does not count. Say it as "their brand" — their unless the user has
  used other pronouns for them, since a name never tells you. Do not volunteer a brand that is not
  theirs; if the user asks, it "sits inside the franchise of drugs they work on", and never takes the
  possessive.
  `brand` can hold several brands, separated by semicolons, with or without a space after each — up
  to 19 of them. Split it on each semicolon and trim the spaces before you use a brand. Print at most
  two, never the whole list: the one the user asked about where there is one, and otherwise the first
  two as the field lists them.

The Radar Briefing uses this same cut for conferences in the next 90 days, so the two must agree: same
URL, same verbatim decode, same connection crossing, people named the same way. They differ in one
thing only, whose people: the briefing keeps the people inside the user's Scope, while conference
play reads no context and crosses the cut across the whole company.

**It is often null.** Conferences nobody has set it on return nothing, and that is normal. Never say a
conference has no Likely Attendee list until its record says so — its row, with
`likely_attendees_portal_url` null — and say it only about a conference the user asked about:
"Athena hasn't published a Likely Attendee list for this one, so here's what I can build". Do not
invent a URL, do not adapt one from a different conference, and do not present a cut you built as
though it were Athena's.

### Several conferences at once

"What's coming up, and who of ours will be there?" asks about many conferences, so it runs on a
budget:

- **Therapeutic conferences are read for their cut; the rest keep their speakers.** A conference
  whose `event_type` includes Therapeutic Conference is crossed with the client's connections, one
  conference at a time, as step 2 says. Any other conference gets its speakers (step 4), from at
  most three `list_conference_speakers` calls, and they are named in this answer: each speaker with
  their company and conference, linked where the Hub record carries a LinkedIn profile, at most five
  for each conference, saying how many more it has. Never hold them back behind "say the word" or
  "I haven't checked those yet". Speakers already read earlier in this conversation may be reused
  rather than read again, and are named here all the same. Matching a speaker to the client's
  contacts is step 4's approximate match, and a speaker is named whether or not you have made it.
- **Print at most five attendees across the answer**, by `lead_score_standardized`, each with their
  `athena_contact_get` read, and hold the rest. Say how many, and name the conferences whose people
  you held back, so they know which to ask about: "at least 6 more at EADV and MDS — ask about
  either and I'll name its people".
- **Say what you checked.** Name the conferences you crossed — "I checked EADV, EURETINA, MDS,
  NACFC, ACG and ECNP" — and then every therapeutic conference with a list that the call limit
  stopped you reaching, with its dates: "Not checked yet: ESMO 23 Oct, ASH 12 Dec - ask and I'll
  check them." A conference you did not reach never reads as one where nobody they know is going. A
  conference without a cut is simply not mentioned in the Likely Attendee part.
- **Stop at 30 calls.** Every call counts, Hub and portal together: the `whoami`, the one
  `list_conferences`, up to three `list_conference_speakers`, and every crossing and every retry.
  5 of the 30 are kept for the attendees' contact reads, so stop crossing once 25 calls are used —
  about twenty crossings when nothing fails — print the attendees already crossed with their brands,
  and name the therapeutic conferences you did not reach under "Not checked yet", with their dates.

**A conference the user names is always checked** — "Who from ESMO specifically?" is a new request
with its own 30 calls, whatever the cap left unchecked before, and so is a Potential-Match Prospects
cut they ask for. Take its row from the answer's `list_conferences` call, or find it by name as step
1 says.

## Step 3 — Build the cut yourself when there is no published one

For a conference the user asked about that has no published cut, reproduce the same shape through
the contact portal's own filters rather than inventing a new idea of relevance. Take its disease
areas from its row and, where the row has none, its therapy areas from `get_conference`. Call
`athena_filter_options_get` for `contact` first — the values are live and per company, and a disease
area that exists for one client may not exist for this one.

The narrow-then-broad pattern the published join uses translates directly, on the conference's
disease areas:

- **Narrow** — filter `disease_areas` on the conference's disease areas and set
  `is_disease_areas_exact_match` to `true`, so you get only people Athena has verified as working on
  them. The flag narrows only alongside `disease_areas`; on its own it changes nothing.
- **Broad** — the same filter without `is_disease_areas_exact_match`. More people, less certainty.
  Say which one the user is looking at, every time.

**Therapy areas have no verified cut.** There is no exact-match flag for `therapy_areas`. When the
conference carries therapy areas and no disease areas, filter `therapy_areas` on them: that is the
broad cut, and the only one there is. Say so — people who plausibly work in the area, not people
Athena has verified — and never present it as the narrow cut.

Where the conference is named on contact records, `conference_names` is a facet you can filter on
directly. Check it first: like every facet, it reports `is_empty` when this company holds nothing for
it, and filtering an empty facet returns nothing at all — which is the data, not a fault.

Run `athena_filter_draft` before quoting any number, and pass on what it says about the count before
the count itself: `unresolved` terms were IGNORED so the number answers a broader question,
`is_rolling` means it moves on its own, `stripped_fields` were removed, either because they are not
the caller's to set or because no filter applies them. Each carries its reason: read it, and say it in
plain words if it changes what the user is looking at.

## Step 4 — Cross the speakers against the client's contacts, honestly

`list_conference_speakers` gives the roster; `get_speaker` opens one; `search_speakers` finds people
across conferences. Name a speaker with a link to the LinkedIn profile the Hub record carries, where
it carries one. The `lead_score`, `lead_score_tier` and `connection` on those records are never
said, whether or not the Hub matches, as "Before you start" says: a speaker's score, tier and "who
knows them" come only from the contact you match them to below.

**There is no shared identifier between a Hub speaker and a portal contact.** The match is made on
name and employer, and it is approximate. Two people share a name; someone changed employer last
month; a company appears under two spellings. So:

- Say "this looks like the same person as X in your contacts" and let the user confirm. Never state
  it as fact and never silently merge the two records into one description.
- When you cannot find a speaker among this client's contacts, that means Athena does not hold them
  for this client — not that they do not exist.
- Never present a speaker's Hub details and a contact's portal scores as if they were one record. If
  the join is uncertain, the score is uncertain too.

## Step 5 — Check the connection angle before you offer it

"Who do we already know who will be there" is the strongest play at a conference — and for several
clients there is no data behind it at all.

Check the `connections` facet in `athena_filter_options_get`. If it reports `is_empty`, this company
holds no LinkedIn connection data and the whole angle is unavailable. Say so once, plainly, and move
on. Do not offer the play, run it, and report an empty result as though nobody at the client knows
anybody.

Where the data does exist, filtering on `connections` gives you the people someone at the client is
already connected to — a much better shortlist than tier alone. Each `athena_contact_find` row
carries `connection`, the names of the people at the client who know that person, so say who knows
them straight off the row; no per-person read is needed for that.

## Step 6 — Distance, and what Athena will not do for you

There is no proximity or distance filter. Contacts carry `city`, `state` and `country`, and that is
what you have.

You can still answer "who's near enough to make the trip worthwhile" from those fields — but reason
out loud and say what you did: "I've taken everyone in the state, which is a rough stand-in for
travelling distance." Do not present a hand-reasoned radius as a filter Athena applied.

Two other honest limits worth knowing:

- Athena holds **no registration list**. The Likely Attendee cut (step 2) is a prospect list — people
  whose work matches the conference — not a record of who is going, and a speaker roster is not an
  attendance list either. Describe neither as one.
- A conference cut ages. Say when the data was read, and offer to re-run nearer the date.

## Step 7 — Leave them something they can use

A conference plan that exists only in a chat window is a plan that will not survive the week. Offer to
save it, and be clear about which of the two you are offering:

- **A list** if they want the people they picked, fixed as they are now — `athena_list_create`, with
  each cut passed back as the filter draft returned it. Write a description saying what it was for and
  when: "SITC 2026 — melanoma exact-match prospects plus connections, built October."
- **A view** if they want the search to re-run live — `athena_view_save`. Better where the conference
  is still months away and the data will keep moving.

Show the name, the description and what each cut pulls in, and get their agreement before saving.
Every one of these appears in the portal where colleagues will see it.

If `athena_orient` reported that this session cannot write, say so before you offer — you can still
draft and count, and the save tool will hand back a portal link carrying the filter instead of an
error. Offer that.

## Use their other tools

If the client has their CRM connected, checking which of these people are already in an open
conversation changes the plan entirely. If they have a research tool, the speakers' recent work is
worth a look before a meeting. Athena's job here is the shortlist, not the whole preparation.
