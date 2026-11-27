---
type: Decision Record
title: Culina is a recipe app with history, ideas and bases to build on
description: Decided by the owner on 2026-11-22. Culina is a recipe app in which the user keeps their recipes and their history and gets ideas what to cook; shipped bases (sauces, doughs, frames such as tacos or pizza) are starting points for the user's own creations, and the user imports the recipes they like. Replaces the purpose of the helper decision; its sourced teaching stays as help.
tags: [decision, scope, purpose]
generated: { by: claude-code/2.1.326, at: 2026-11-22 }
sources:
  - '[^owner]: the owner in the session of 2026-11-22, in German, translated here'
---

# Culina is a recipe app with history, ideas and bases to build on

**Note of 2026-11-27:** the purpose of point 1 and the goal of point 3
to cover every common base are replaced by the
[daily driver decision](daily-driver.md): Culina is a daily driver to
build dishes, with sourced pairing ratings and a taste profile. Versions,
tasting notes, the import and the bases built so far stand.

## Context

The [helper decision](mentor.md) of 2026-11-18 made Culina a helper that
teaches until it is no longer needed; shipped classics, 12 per country,
were a source of ideas. On 2026-11-22 the agent argued against 100
regional German classics with that purpose. The owner answered that the
scope was meant to change, and that the change had never been recorded:

> Culina is a recipe app in which one tracks one's history, among other
> things, and gets ideas what one can make. For that I thought of
> putting all classics in, because from there one can make one's own
> creations. Perhaps only basics such as sauces (mayo, hollandaise) and
> basics such as tacos, burritos, pizza and so on, and from there one
> completes one's creations, and also imports recipes one likes to
> make, such as pancakes.[^owner]

The agent proposed bases over a full set of classics; the owner agreed
("then let us rebuild it that way").[^owner]

## Decision

1. **Purpose.** Culina is the user's recipe app: their recipes with
   versions and tasting notes, their history, and ideas what to cook.
   It is measured by whether the user finds something to cook and
   builds their own dishes, no longer by whether they need it less.
2. **Bases ship, as starting points.** What ships is mainly bases the
   user builds on: sauces (mayonnaise, hollandaise, béchamel,
   vinaigrette, tomato sauce), doughs (pizza, pasta, shortcrust, yeast
   dough, pancakes) and frames (tacos, burritos, bowl, curry, risotto,
   stir-fry, soup). A base is components with slots and alternatives,
   so a creation swaps a component instead of rewriting a recipe. Each
   base follows a named source, amounts as facts, the method in our own
   words, as before. Bases are international; the cuisine stays a
   filter.
3. **Classics become bases or an import pack** (owner, 2026-11-22: "yes,
   become basics, and add more, all possible ones there are"). The
   shipped classics that are a base (sauces such as mayonnaise, alioli,
   hollandaise, tomato sauce, béchamel; doughs and batters such as egg
   noodles, pancakes, yeast dough, gnocchi; frames such as risotto,
   soup, omelette, custard) become bases with slots and alternatives.
   The others leave the shipped list and are offered as an optional
   import pack, so their work is kept. The set of bases is to be broad,
   covering every common base, built in batches. Regional classics
   (issue 38) are on hold; only the region field may come earlier.
4. **The user's own recipes come in by import** (own JSON, Cooklang;
   [decision](cooklang.md)) and by the editor; an import is version 1.
5. **Teaching stays as help, no longer as the purpose.** The sourced
   technique notes, substitutes and handling rules of the helper
   decision stay and keep their source rules (no language model, every
   statement sourced). Marking a skill as learned stays. The goal of
   "free of the app" and the four stages of point 9 there no longer
   steer the work.

## Open, for the owner

- **History.** Versions and tasting notes exist. Whether history also
  means a log of when a dish was cooked is not decided; such a log
  touches the rule "no food diary" in `AGENTS.md`, which stays until
  the owner changes it.
- **Ingredients of bases.** The catalogue comes from USDA SR Legacy and
  lacks some products bases need (for example quark); a German source
  for them is not chosen.

## Consequences

- `AGENTS.md` of Culina, the root `AGENTS.md`, the README and the
  [roadmap](../roadmap.md) describe Culina as a recipe app.
- The [helper decision](mentor.md) carries a note that its purpose is
  replaced; its rules on sources and teaching stand.
- How a base and the pack are marked in the data:
  [bases and the pack](bases-and-pack.md) (2026-11-22).
- New issue for the bases; issue 38 on hold; issue 33 (Davidis 1849)
  stands, since the German classics stay.
