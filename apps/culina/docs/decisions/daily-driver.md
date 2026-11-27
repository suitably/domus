---
type: Decision Record
title: Culina is a daily driver for building dishes, with pairing and a taste profile
description: Decided by the owner on 2026-11-27. Culina is the cook's daily tool to get ideas fast, click dishes together from components, see from sourced data what goes well together, and end with a taste profile of the dish; teaching happens on the side, as the reason given for a pairing. Bases ship only as far as building needs them. Non-commercial sources, FlavorDB among them, are allowed; RecipeNLG stays out. Replaces the purpose of the recipe app decision.
tags: [decision, scope, purpose, pairing, taste-profile]
generated: { by: claude-code, at: 2026-11-27 }
sources:
  - '[^owner]: the owner in the session of 2026-11-27, in German, translated here'
---

# Culina is a daily driver for building dishes, with pairing and a taste profile

## Context

The purpose of Culina was recorded twice: on 2026-11-18 as a helper
that teaches until it is no longer needed ([helper](mentor.md)), on
2026-11-22 as a recipe app with history and a broad set of shipped
bases ([recipe app](scope-recipe-app.md)), from which issue 39 planned
about a hundred bases. Shown that state on 2026-11-27, the owner said:

> But we want a cooking-learning app, don't we?! I just want a daily
> driver with which I get inspiration fast, click recipes together,
> get ratings of what goes well together, and perhaps at the end have
> a taste profile of a dish.[^owner]

Asked how much teaching, the owner chose "on the side": each rating
says briefly why something fits, with its source; technique notes stay
available; no course and no ticking off.[^owner] Asked where ratings
come from, the owner chose sourced data only (not co-occurrence in the
shipped recipes, not the user's own tasting marks).[^owner] On licences:
"the app will be non-commercial, Culina that is, so sources are
fine".[^owner]

## Decision

1. **Purpose.** Culina is a daily driver: open it, get an idea fast,
   click a dish together from components, see what goes well with what,
   and see the taste profile of the result. It is measured by how fast
   the user gets from opening the app to a dish they want to cook.
2. **Building is the centre.** A dish is components in slots with
   alternatives (the existing model); building means picking and
   swapping components and ingredients. The one Cook screen
   ([cook screen](cook-screen.md)) is the entry for ideas.
3. **Pairing ratings from sourced data.** While building, the app shows
   for an ingredient or component what goes well with it, and for the
   dish being built how well its parts go together. Every rating comes
   from a data set or a cited work, names it, and says in one sentence
   why (for example shared aroma compounds, or "named as a classic
   pairing in …"). The rating is a hint with its source and confidence,
   not a verdict: aroma-based pairing holds for Western cuisines and not
   for all others ([how chefs work](../chef-practice.md#pairing)), and
   the screen says so where it matters. No rating is computed from
   co-occurrence in the shipped recipes or from the user's tasting
   notes. The stored form is the existing affinity
   ([data model](../data-model.md#affinity)).
4. **Taste profile.** A dish ends with a profile over sweet, sour,
   salty, bitter, umami, fat and heat. What the catalogue can give is
   derived from it (sugars, sodium, fat from USDA SR Legacy); the rest
   comes from a per-ingredient table with a source per row, or is
   marked as a judgement. A dimension without data is shown as unknown,
   never guessed.
5. **Teaching on the side.** The reason of a rating, the technique
   notes, substitutes and handling rules stay, with their source rules
   ([helper](mentor.md)). There is no course, no progress and no
   suggestion what to practise next; marking a skill as learned stays
   as it is.
6. **Sources.** Culina is non-commercial
   ([sources licence](sources-licence.md)). Data under a non-commercial
   licence is allowed, FlavorDB among them; data derived from a
   share-alike source carries that licence in the data. RecipeNLG stays
   out: it is licensed for research and education only and is scraped
   from recipe sites. Every data set still names its source and
   licence.
7. **Bases only as far as building needs them.** The bases of
   [issue 39](https://github.com/suitably/domus/issues/39) part 1 stay.
   The goal of "every common base" is dropped: a base is added when
   building needs it, not to complete a list.
8. **Unchanged.** No language model in Culina (recipes from the user's
   own model come in by the [prompt import](ai-prompt-import.md)); no
   meal plan, no food diary, no shopping list (that is emptio); own
   local data, offline first.

## Open

- **Which pairing data first.** FlavorDB is downloadable per molecule
  only ([existing tools](../existing-tools.md)); whether its data can be
  gathered within its terms, and whether the data of Ahn et al. (2011)
  has a licence, are to be read at the source. Curated pairings from
  cited books (one fact per entry, [source books](../source-books.md))
  are the fallback.
- **Sour, bitter, umami, heat.** No source per ingredient is chosen.
- **Shared with Mixtura.** Mixtura shows balance figures for drinks; a
  taste profile both apps need belongs in materia
  ([rule](../../../../AGENTS.md)), once both need it.
- **Issue 39 parts 2 and 3** (the import pack, new bases by family) are
  for the owner to keep, shrink or close.

## Consequences

- `AGENTS.md` of Culina, the root `AGENTS.md`, the README and the
  [roadmap](../roadmap.md) describe Culina as a daily driver; the rule
  "No data from FlavorDB" is lifted.
- The [recipe app decision](scope-recipe-app.md) carries a note that its
  purpose and point 3 (every common base) are replaced; versions,
  tasting notes, import and bases built so far stand.
- New issues: pairing data, pairing in the builder, taste profile.
