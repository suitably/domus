---
okf_version: '0.2'
---

# Culina knowledge bundle

[Open Knowledge Format](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md)
(OKF v0.2) bundle. One file per concept, YAML frontmatter, plain markdown
links. Change history is in [log.md](log.md).

# Plan

- [Roadmap](roadmap.md) - Goals and their order; the helper (skills, checklist, known dishes) comes before sync.
- [Helper build plan](helper-plan.md) - The owner's answers, the four stages, the data that is added, the work packages with order and model, the checks and the end: classics of four countries with their techniques, cookable step by step.
- [What "what do I feel like" still needs](feel-like-plan.md) - After one dish list: course and cuisine coverage, the filter with the stock, something like this dish, time and effort, taste effects, seasonal produce, favourites and history, each with its data, cost and the rule it touches; the recommendation.
- [Deployment](deployment.md) - Deployed from main to culina.nexumia.de on Cloudflare Workers static assets by the CI job deploy; the host is an adapter, and how to serve the same folder elsewhere.

# Research

- [Existing tools and data](existing-tools.md) - Survey of FOSS cooking software and data sets with licences, and the gap Culina fills.
- [Evaluation of nutrition values and food tracking](nutrition-evaluation.md) - Nutrient databases and licences read at the source, per-portion figures by weight and their precision, hand-over to trackers, EU rules, and the recommendation: derived per-portion figures from bundled USDA data, dishes handed to an existing tracker, no diary, no backend; and what was built (issue 94).
- [How chefs work](chef-practice.md) - Ratios, components, seasoning, pairing, cooking from stock, operations; what a tool should do and the data model that follows.
- [Classic dishes by country](classic-dishes.md) - Survey for Germany, Italy, France and Spain: the public-domain cookbooks that open in full, 25 dishes per country with their place in the source and how far each was read, and the gap for dishes younger than the books.
- [Selection of the classics](classic-selection.md) - The 12 dishes chosen per country on 2026-11-19, each with the source its recipe was read in, whether it is without meat and fish, units, persons, vague amounts and the dishes dropped.
- [The German classics in Davidis 1849](davidis-1849.md) - Where the 12 German classics stand in the German edition of 1849 that replaces the Milwaukee editions, its units, the licence of the Deutsches Textarchiv text and the dish it lacks.
- [Four books read against the notes](book-findings.md) - Research of 2026-11-20: McGee, Der junge Koch and The Food Lab read by three agents against the 64 notes, how the texts were handled, what the books confirm, refine and contradict, the 24 changed and 14 new notes, where the books disagree, what was not shipped and what nobody verified.
- [Books as cited authorities](source-books.md) - Research of 2026-11-19 on books that can be cited for the skills (McGee, Der junge Koch, The Food Lab, Pauli, Hamelman and others) with evidence level, languages, editions, what was verified and what not, food-safety authorities and their differing numbers, and the owner's decisions of the same day: buying order, a citation needs a page looked up in the book, two new evidence levels, food-safety figures by the authority of the user's language.
- [Name check](name-check.md) - Research of 2026-11-22: Culina and Cook's Notebook in npm and Flathub (nothing of either name); the app stores and trademark registers were not reachable and are the owner's to check.
- [Dish characters computed from the recipe](dish-characters.md) - Analysis of 2026-11-21: whether tomatoey, creamy or fried can be computed from ingredients, amounts and techniques for a finder "what do I feel like"; measured on the 48 classics (presence works, share by mass rests on 44 percent of the lines, no meat or fish in the catalogue), three kinds of character and the recommendation, and what was built of it.
- [The German of the app](german-review.md) - Review of 2026-11-21 after a tester read the app in German: every changed text (app strings, shipped dishes, technique notes, handling rules, substitutes) with old and new wording, the date form, cuisine adjectives and citation words, what is left to the owner (the visible name, everyday ingredient names), done by a language model and not yet by a native speaker with kitchen training.
- [Ratios and salt percentages checked against sources](ratios.md) - Every ratio, baker's percentage and salt percentage with basis, status (confirmed, corrected, unverified) and source; NCHFP fermentation data.

# Specifications

- [Test vector format](vectors.md) - JSON shape of the engine's worked examples for ratio checks, baker's percentages, scaling by weight and salt, with five vectors computed by hand and the kinds the engine now runs, now including ratio templates, salt, components and cross-record validation.

- [Classic format](classic-format.md) - The guide a transcriber follows to write one dish file and one technique file, field by field, with an invented example, the units and settled values, the second check and the command that validates one file.

- [Engine rules](engine-rules.md) - Unit table with exact factors (US customary cup, tablespoon, teaspoon), conversion and its errors, the explicit rounding step, scaling by weight with time and salt cautions, baker's percentages and hydration, ratio templates with derivatives, family queries, the range check (numbers, no verdicts) and the seed templates, salt by basis, and components with nesting, yield and loss, edible portion and portions, and cross-record validation of the model (references, cycles, anchors, ranges), versions with head, variations and the diff of two versions, stock (days left, use-by suggestions, using up); vectors in `packages/engine/vectors/`.

- [Data model](data-model.md) - Every stored entity (tasting notes are records of their own) with fields, ids, units, stored versus derived, general versus kitchen-specific, export envelope, Cooklang compatibility and open questions; JSON Schemas in `packages/engine/schemas/`; storage and migrations of the local store (IndexedDB database, sync-ready rows with `updatedAt` and tombstones, the clock rule, migration procedure and failure behaviour, persistence, logging, the drafts store).

- [Behaviour](behaviour.md) - Screens, navigation (one back stack, phone first), the ratio list and cards with the calculator, number entry, cautions and the neutral range display, the component and dish screens (lines, yield, scaling, role slots, portions, the open-slot rule, example data) and the editor for components and dishes (weight or percent through the engine, picker, cycles, ratio card to recipe, drafts, delete with undo), stock (tick, list, detail), Cook (one list of every dish with "Only with what I have" and the order "Use up first", start from one ingredient with the finder's filters and empty states), versions with history, diff, restore, tasting notes and variations, interaction rules, empty, error and offline states, the service worker and the installability check of the PWA, the local store's Settings line and error texts, export, import (merge, replace, Undo, Cooklang step) and the backup reminder, each rule with the test that checks it.
- [Ingredient catalogue](ingredient-catalogue.md) - The basic ingredients shipped with the app, generated from USDA FoodData Central SR Legacy (CC0): why that dataset, the selection rule, how density and unit weight are derived from portion text, the file, and what it does not have.

- [Design](design.md) - Light and dark with a user override, the Beet accent, colour, type and spacing tokens with computed contrast ratios, the ratio, component, dish and editor screens, icon rules, the 720 px breakpoint and screenshots.

- [Stock orderings](stock-orderings.md) - What the stock does when things happen in an unlucky order (a change during a cycle, a stale screen, the move of old items interrupted, two tabs, two devices), with the test that pins each ordering or "not pinned".

- [Export, import, matching and Cooklang](exchange.md) - The engine's pure rules of the export file, merging by id with conflicts, remapping ingredient ids, matching names to ingredients, and the Cooklang reader and writer written from the specification, with what is supported and what is lost; vectors in `packages/engine/vectors/`.

# Decisions

- [App name](decisions/app-name.md) - Code name Culina, visible name Cook's Notebook.
- [After Mixtura](decisions/after-mixtura.md) - Culina is the second app on the engine Mixtura builds first, as a PWA with own local data.
- [Shared model](decisions/shared-model.md) - Accepted: the general model comes from nexumia-de/materia at an exact version tag; Culina's engine adds the kitchen rules.
- [Prompt import](decisions/ai-prompt-import.md) - Decided 2026-11-22: a prompt for the user's own language model writes recipes in a small JSON import format the user pastes; Culina checks and matches with confirmation, runs no model itself; Cooklang sections become components of a dish.
- [Cooklang](decisions/cooklang.md) - Decided: own JSON as the lossless format; Cooklang export and import, where on import the user supplies what the file cannot carry.
- [Recipe book](decisions/recipe-book.md) - Decided: the user's own recipe book with versions and a finder; no scraping, meal plans or shopping list (planned sister app emptio); stock ticked by default; English canonical, German supported, language follows the device.
- [Daily driver](decisions/daily-driver.md) - Decided 2026-11-27: Culina is a daily driver to get ideas fast, click dishes together, see sourced pairing ratings with their reason and a taste profile of the dish; teaching on the side; non-commercial sources such as FlavorDB allowed, RecipeNLG not; bases only as building needs them. Replaces the purpose of the recipe app decision.
- [Recipe app](decisions/scope-recipe-app.md) - Purpose replaced on 2026-11-27 by the daily driver decision. Decided 2026-11-22: Culina is a recipe app with history, ideas and shipped bases (sauces, doughs, frames) to build one's own dishes on; replaces the purpose of the helper decision.
- [Bases and the pack](decisions/bases-and-pack.md) - Decided 2026-11-22 by the lead under the recipe app decision: a shipped file is a base by `kind: base`; a component of a slot's role that no slot reaches is an alternative, no list stored; the other classics become an import pack in `data/pack/`, taken as shipped dishes, never copied into the store; whether it is taken at first start is the owner's.
- [Helper](decisions/mentor.md) - Decided: a scripted helper without a language model that teaches from research and textbooks until it is no longer needed; a learned skill is not taught again; known dishes ship by country as full recipes with their techniques, first Germany, Italy, France and Spain.
- [Licence of the sources](decisions/sources-licence.md) - Decided 2026-11-19: non-commercial sources (CC BY-NC-SA 4.0, CC BY-NC 4.0) fit; every source of a shipped note carries its licence as an SPDX identifier.
- [One Cook screen](decisions/cook-screen.md) - Decided 2026-11-22 (issue 44): Cook is one screen, the one dish list with the one filter, "Only with what I have" judged by the finder and "Use up first" as its order, the stock one tap away; Start and the finder screen went; three destinations (Cook, Notebook, Ratios); the Notebook's Dishes are own and shipped dishes in a plain list; the old addresses redirect.
- [One dish list](decisions/one-dish-list.md) - Decided 2026-11-21: own and shipped dishes in one list with one entry, one filter and one dish and cook screen; the origin follows from where a dish lives and is not stored; shipped dishes stay read-only in their file; rejected: a stored flag in the user's store, two tabs with the same filters.
- [Hosting](decisions/hosting.md) - Decided: the app is a static folder any host or own server can serve; Cloudflare is the first host and only an adapter in this repository, no shared deploy code.
- [Sync](decisions/sync.md) - Decided: three units as Mixtura (place, library, profile), a library shareable by invite, places shared with Mixtura, tasting notes as own records, built after stock; for stock replaced by stock in spaces.
- [Stock in spaces](decisions/stock-in-spaces.md) - Decided 2026-11-20: stock is the shared collection `stock` of a space, the same collection as Mixtura's with Culina's optional fields; the record, batches, unticking, the space used, sync, the one move of old items, the export without stock, how the catalogues will meet, what is lost.

# Development

- [Native Linux app](native-linux.md) - Cook's Notebook as a GTK 4 and libadwaita app on GJS (`packages/linux`): Cook, the one list of the shipped dishes under the dish filter with "Only with what I have" and the order "Use up first", a dish with amounts per portion, cooking step by step, ticking the stock of the shown space, joining a space by link; what it shares with the web app, what 0.1 leaves out, the alignment of the ingredient list, build, release, packaging and what nobody verified.
- [Development](development.md) - Setup, commands, layout, the engine's purity rule, the app build, generated validators, tests and service worker, CI (jobs `check` and `e2e`) and the network hosts.
- [Dependencies](dependencies.md) - Every dependency with exact version, licence and use.
