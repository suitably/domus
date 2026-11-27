---
type: Plan
title: What "what do I feel like" still needs
description: Written on 2026-11-21 after the decision on one dish list. What the question "what do I feel like" still needs once own and shipped dishes share one filter, as an ordered list (course and cuisine coverage, the filter with the stock, something like this dish, time and effort, taste effects, seasonal produce, favourites and history), each with the data or model change it needs, what it costs and which rule or decision it touches; and the recommendation. No code.
tags: [plan, finder, characters, filter, dishes, time, season]
generated: { by: claude-code, at: 2026-11-21 }
sources:
  - id: decision
    title: 'Decision: one list of dishes, whatever their origin'
    url: decisions/one-dish-list.md
  - id: characters
    title: 'Dish characters computed from the recipe'
    url: dish-characters.md
  - id: classics
    title: 'The 48 shipped dishes, `packages/app/data/classics/`, counted by a script on 2026-11-21'
---

# What "what do I feel like" still needs

The owner's question is "what do I cook, what do I feel like?"
([decision](decisions/one-dish-list.md)). After the unification, one
filter narrows every dish by origin, character, course, cuisine, "Without
meat and fish" and "Only with every amount given". This plan lists what
the question still needs beyond that. Nothing here is decided or built.

**Cost** is a rough size: **small** is a change in the app with no new
data; **medium** is new data that the project can fill itself, or a new
stored field; **large** is a data set that needs a source not yet found,
or rereading the sources of the shipped dishes.

## The list, in the recommended order

### 1. Course and cuisine on every dish

- **Now**: the shipped dishes have both. Own dishes get both as hand-set
  fields; this is being built on 2026-11-21. An own dish saved before has
  neither, and components have neither.
- **Needs**: no model change beyond the two fields. The editor could
  propose a course from the slots (a dish with a slot "sauce" alone is a
  sauce); that is a judgement and stays a proposal.
- **Cost**: small.
- **Touches**: nothing. A cuisine the user types is kept as given, and
  the filter offers it beside the four country codes (`offeredBy` in
  `src/browse/filter.ts` already sorts unknown values after the known).
- **Open**: whether a cuisine is a country or a style ("Levantine", "home
  cooking"). The four shipped ones are countries; the user's need not be.

### 2. What I feel like that I can cook now

- **Now**: the finder judges own dishes and shipped dishes against the
  stock, and with the unification it takes the one filter (being built on
  2026-11-21). The random pick can already keep to what is in stock.
- **Needs**: no model change. What remains is the way in: one choice on
  the start screen, "with what is there", that hands the filter to the
  finder instead of the list, and the random pick drawing from that
  result.
- **Cost**: small.
- **Touches**: the [stock in spaces](decisions/stock-in-spaces.md)
  decision only as a reader: stock is presence, so "can cook" means every
  ingredient is ticked, not that enough is there.

### 3. Something like this dish

- **Now**: every dish has its characters, computed when it is shown
  ([analysis](dish-characters.md)).
- **Needs**: a pure function that ranks other dishes by the characters
  they share with this one (the share of common characters among those
  either has), with course and cuisine as tie-breakers. A link "Dishes
  like this" on the dish screen opens the list with that order. No stored
  field, no new data.
- **Cost**: small to medium (the function, its vectors, one link).
- **Touches**: the rule that a judgement says so. The ranking rests on
  the character table, whose every row is a judgement, so the screen says
  what it compared ("shares: tomatoey, braised"). A dish with one or no
  character finds nothing useful; the screen then says nothing rather
  than a poor match.

### 4. Time and effort ("quick")

- **Now**: a step may carry a `time` as its source gives it, one value or
  a range in seconds, minutes or hours ([data model](data-model.md#steps)).
  Of the 447 steps of the 48 shipped dishes, 27 carry a time, in 22
  dishes. No dish has a total. On the user's own steps a time is
  optional; the step proposal can suggest one from a sentence
  ([engine rules](engine-rules.md#proposing-a-step-from-a-sentence)).
- **What a total would need**:
  - a time on every step, or a total stated for the dish;
  - a split into active and waiting time: a sauerbraten marinates for
    days and takes little work, so "quick" and "little work" are two
    questions;
  - steps that run at the same time (the pasta water heats while the
    sauce cooks), which a sum of the steps overstates;
  - a range rather than one number where the source gives a range.
- **Two ways**: derive a total only where every step has a time, and say
  "no total" otherwise (no new data, but few dishes qualify); or a stored
  `time` on the dish, `{ active, total }`, set by hand on own dishes and
  settled by us on shipped ones. Effort could be derived from the number
  of steps and of techniques the user has not marked as learned; that is
  a judgement and personal.
- **Cost**: large for the shipped dishes (the 19th-century sources rarely
  give times, so most totals would be settled by us, dish by dish); medium
  for own dishes (one stored field, the editor, the filter).
- **Touches**: the [classic format](classic-format.md) rule that a value
  the source leaves vague is marked as settled by us, and the
  [helper decision](decisions/mentor.md) that nothing is taken from
  memory. A stored total also breaks "derived, not stored" for the one
  field where deriving does not work.

### 5. Taste effects: heat, sourness and others

- **Now**: the character table has "spicy" (a hot chili present, or
  cayenne above a share) and "lemony" (lemon or lime from 2 percent of the
  mass). How hot or how sour is not known: the catalogue has no measure of
  either ([analysis](dish-characters.md#what-a-rule-can-read)).
- **Needs**: a table of taste effects per ingredient or group (heat,
  sourness, bitterness, salt, umami), each with a level and a source, and
  rules in the character table that read it.
- **Cost**: large. No data set with a usable licence has been found or
  checked for this.
- **Touches**: the rule "every data set names its source and licence" and
  FlavorDB, allowed since 2026-11-27 ([daily driver](decisions/daily-driver.md)). A table filled
  by the project is a judgement and says so in each row, as the character
  table does.

### 6. Seasonal produce

- **Now**: the catalogue has no season
  ([ingredient catalogue](ingredient-catalogue.md)). The affinity in the
  [data model](data-model.md) has a season as context, with no data and
  no code.
- **Needs**: a season per produce and region (months), a region for the
  user, and a rule "in season" over a dish's ingredients. The app has no
  clock in the engine, so the month comes in from the app.
- **Cost**: large. Seasons differ by region and by source, and no source
  with a licence has been read for it.
- **Touches**: sources and licences as in item 5. The device's language
  does not give the region; asking for one is a new setting.

### 7. Favourites and "not cooked for long"

- **Now**: the app keeps no record of liking or of cooking. Two dates come
  close: a version has a `date`, and a tasting note has the `date` of the
  tasting ([data model](data-model.md#tasting-note)). "Last tasted" can be
  derived from the notes the user wrote, with no new state; it says nothing
  about a dish cooked without a note.
- **Needs**: a favourite is one stored mark per dish family, or per
  shipped dish id (a shipped dish is read-only, so the mark lives beside
  it in the user's store). "Not cooked for long" needs a date each time a
  dish is cooked, for example when the cook screen reaches its last step.
- **Cost**: medium (stored state, its place in export and in the planned
  library or profile sync, [sync](decisions/sync.md)).
- **Touches**: the rule "no meal plans and no food diary" in
  [AGENTS.md](../AGENTS.md). A favourite mark is not a diary. A date per
  cooking is a record of what was eaten when, which is close to one. Both
  are **the owner's to decide** before anything is built.

## Recommendation

1. **Items 1, 2 and 3 next.** They need no new data and touch no rule,
   and each makes the one filter answer more of the question at once.
   Item 3 gives "what do I feel like" an answer that starts from a dish
   rather than from a list of choices.
2. **Item 4 in its small form**: a total only where every step has a
   time, shown and not filtered, so that the gap is visible on real
   dishes. Whether a stored total with active and waiting time is wanted,
   and whether the shipped dishes get settled totals, is a question for
   the owner, because it is the costliest item and breaks "derived, not
   stored".
3. **Item 7 waits for the owner's word** on the food diary rule. "Last
   tasted" from existing notes may be offered meanwhile, since it adds no
   state.
4. **Items 5 and 6 wait for a source.** Without a data set with a licence,
   they would be the project's judgement on every ingredient, which is a
   large and visible risk to trust for a small gain.

Not verified: how many own dishes in a real notebook have characters,
times or course; every count above is over the shipped dishes only.
