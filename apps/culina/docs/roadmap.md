---
type: Plan
title: Roadmap
description: Goals and their order for Culina; the shared model comes from materia; work items are GitHub issues.
tags: [plan, roadmap]
generated: { by: claude-code/2.1.326, at: 2026-11-16 }
---

# How the plan is tracked

This file keeps the goals and their order; work items are
[issues](https://github.com/nexumia-de/culina/issues). The reasons are
in [how chefs work](chef-practice.md) and
[existing tools](existing-tools.md).

# Goals

0. **Wait for Mixtura's engine** ([decision](decisions/after-mixtura.md)).
   Done: the engine exists and its general model is the package
   materia ([decision](decisions/shared-model.md)).
1. **Foundations.** The general model moves into shared code; Culina's
   data model as a document; the numbers in
   [how chefs work](chef-practice.md) checked against their sources;
   the decision on Cooklang as recipe format.
2. **Ratios and components.** Ratio cards with their variations,
   baker's percentages, components with yields, scaling by weight.
3. **Notebook.** Own recipes and components with versions and tasting
   notes, export and import.
4. **Stock.** What is there with state and use-by date; what it allows,
   ranked by urgency and role.
5. **Helper.** Built on 2026-11-19 after the
   [helper build plan](helper-plan.md), state at the top of the
   [log](log.md); the correction checklist and suggestions from a
   tasting check are still open. Since 2026-11-22 the helper is help
   inside a recipe app ([decision](decisions/scope-recipe-app.md)), not
   its purpose ([helper decision](decisions/mentor.md)): skills with scripted questions
   and cautions from research and textbooks, a learned skill no longer
   taught; substitutes with their effect and handling rules per
   ingredient (when it goes in and why); the correction checklist and suggestions from a tasting
   check; known dishes shipped as a source of ideas, matched to the
   catalogue by role.
6. **Sync.** The same mechanism as Mixtura in three units (place,
   library, profile), encrypted, over storage the user configured
   ([decision](decisions/sync.md)). Partly done on 2026-11-20: **stock**
   is the shared collection `stock` of a space, synced by the host
   package with the household's other apps
   ([decision](decisions/stock-in-spaces.md)); places are spaces now.
   Not done: the library (components, dishes, own ingredients, tasting
   notes) and the profile.
7. **One list of dishes.** Decided on 2026-11-21
   ([decision](decisions/one-dish-list.md)): own and shipped dishes in
   one list with one filter (origin, character, course, cuisine, without
   meat and fish, only with every amount), used by the lists, the start
   screen, the finder and the random pick; one dish and one cook screen;
   course and cuisine on own dishes. Built on 2026-11-21 (pull request
   30), and the user's correction of a character on 2026-11-22. Open after it, in the order of the
   [plan for "what do I feel like"](feel-like-plan.md): something like
   this dish, time and effort, favourites (the owner's to decide under the
   rule "no food diary"), taste effects and seasonal produce (both wait
   for a source).
8. **Bases.** Sauces, doughs and frames (tacos, pizza, curry, …) as
   components with slots, each from a named source, for the user's own
   creations ([decision](decisions/scope-recipe-app.md)). Part 1 done
   on 2026-11-23. Since 2026-11-27 only as far as building needs them,
   no longer every common base ([decision](decisions/daily-driver.md)).
   Regional classics are on hold.
9. **Daily driver: pairing and taste profile** (decided 2026-11-27,
   [decision](decisions/daily-driver.md)). In this order: pairing data
   with source and licence (FlavorDB or a cited alternative, into the
   existing affinity); pairing ratings with their reason while building
   a dish; a taste profile of the dish (sweet, salty, fat from the
   USDA catalogue; sour, bitter, umami, heat from a sourced table, else
   unknown). This goal comes before goals 6 and 8.
10. **Later.** Curated affinities with sources, cost, prep lists,
    allergens.
