---
type: Specification
title: Data model
description: Every entity Culina stores (ingredient, ratio template, component, dish, stock item, affinity, tasting note) and the export file, with fields, required flags, units, ids, what is stored versus derived, which part is the general model shared with Mixtura and which is kitchen-specific, how it stays compatible with both Cooklang outcomes, and the open questions with proposals.
tags:
  [
    data-model,
    schema,
    json-schema,
    units,
    ids,
    export,
    storage,
    migrations,
    ratios,
    components,
    yield,
  ]
generated: { by: claude-code/2.1.326, at: 2026-11-17 }
sources:
  - id: chef
    title: 'How chefs work, section "Data model that follows"'
    url: chef-practice.md
  - id: ratios
    title: Ratios and salt percentages checked against sources
    url: ratios.md
  - id: vectors
    title: Test vector format
    url: vectors.md
  - id: cooklang
    title: Whether Culina reads and writes Cooklang files (decision record, needs decision)
    url: decisions/cooklang.md
  - id: tools
    title: Existing tools and data
    url: existing-tools.md
  - id: mixmodel
    title: 'Mixtura: Data model (read for structure only; nothing copied)'
    url: ../../mixtura/docs/data-model.md
  - id: mixshared
    title: 'Mixtura: Shared model inventory (which parts are general)'
    url: ../../mixtura/docs/shared-model.md
  - id: ka
    title: "King Arthur Baking: What is baker's math (total formula, preferment, final mix; read 2026-11-17)"
    url: https://www.kingarthurbaking.com/blog/2022/04/28/what-is-bakers-math-and-how-can-i-use-it-in-my-everyday-bread-baking
  - id: busby
    title: "Busby's Bakery: Bakers percentages and bakers formulas (pre-fermented flour; search excerpt only, host blocked)"
    url: https://busbysbakery.com/bakers-percentages/
  - id: yield
    title: 'Basic Kitchen and Food Service Management (BC Cook Articulation Committee, BCcampus; CC BY 4.0), chapter 3 "Food Costing", section 3.2 "Yield Testing" (AP, EP, yield percentage, factor; read 2026-11-18 on workforce.libretexts.org, the same book on opentextbc.ca answers 403)'
    url: https://workforce.libretexts.org/Bookshelves/Food_Production_Service_and_Culinary_Arts/Basic_Kitchen_and_Food_Service_Management_(BC_Campus)/03%3A_Food_Costing/3.02%3A_Yield_Testing
  - id: eu
    title: 'Regulation (EU) 1169/2011, Annex II, substances causing allergies or intolerances (read 2026-11-17)'
    url: https://www.legislation.gov.uk/eur/2011/1169/annex/II
  - id: cooklangspec
    title: 'Cooklang specification README (ingredients @name{qty%unit}, recipe references, no percentage notation; read 2026-11-17)'
    url: https://raw.githubusercontent.com/cooklang/spec/main/README.md
  - id: orf
    title: 'Open Recipe Format, documentation start page (YAML based, accurate and flexible storage of recipes; read 2026-11-18; licence not stated on the pages read)'
    url: https://openrecipeformat.readthedocs.io/latest/
  - id: orfref
    title: 'Open Recipe Format Reference (ingredients, amounts, processing, yields, steps; read 2026-11-18)'
    url: https://openrecipeformat.readthedocs.io/latest/topics/reference/orf.html
  - id: mealie
    title: 'Mealie: recipe schema (recipe.py; yield as quantity and text, ingredients as a list; ingredient type not read)'
    url: https://raw.githubusercontent.com/hay-kot/mealie/mealie-next/mealie/schema/recipe/recipe.py
  - id: schemaorg
    title: 'schema.org/Recipe, recipeIngredient and recipeYield (development version; CC BY-SA 3.0 per schema.org/docs/terms.html; read 2026-11-18)'
    url: https://schema.org/Recipe
  - id: jsonschema
    title: 'JSON Schema: Structuring a complex schema ($id, $defs, $ref, $schema, draft 2020-12; read 2026-11-17)'
    url: https://json-schema.org/understanding-json-schema/structuring
  - id: ajv
    title: 'Ajv 8.17.1 (MIT), licence read from its package.json; 2020-12 class ajv/dist/2020'
    url: https://www.npmjs.com/package/ajv/v/8.17.1
---

# Data model

What Culina stores, what it derives, and the JSON Schemas that pin the
stored shapes. It is written before any engine or store code, so the
engine (issues 8, 9, 10) and the store (3.1) start from one agreed shape.
The machine-readable form is in
[`packages/engine/schemas/`](../packages/engine/schemas/); a test fails
when a field in the tables below is missing in a schema or the other way
round, or when a "Required" cell differs from the schema.

Principle: **derive, don't store** ([AGENTS.md](../AGENTS.md)). Only
what the user entered, and seed data, is stored. Yields, losses,
percentages, cost, "what can I make" and rankings are computed every
time from stored amounts ([stored versus derived](#stored-versus-derived)).
No kitchen number is defined in this document: a field that needs a
figure (yield, density, usual range) has no default and carries its
source.

# General model and kitchen-specific parts

[Mixtura's shared model inventory](../../mixtura/docs/shared-model.md)[^mixshared]
lists what is general and will move into shared code; where that code
lives is still an owner decision (issue 2), so Culina holds no engine
code and **copies nothing**: the schemas here are written for Culina's
model. Where the shared code exists, the "general" columns below are
replaced by a reference to it.

| Entity                  | General model (Mixtura inventory module)                  | Kitchen-specific here                                                                      |
| ----------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Amounts and units       | `amounts/`: number plus unit, conversion, scaling         | Kitchen unit set; mass is canonical; density and unit weight per ingredient                |
| Ingredient              | `ingredients/` tree, inheritance, shelf life              | Yield per state (as purchased to edible), season, allergens, density, unit weight          |
| Ratio template          | `templates/` roles, slots, matching, variants             | Parts by mass with an anchor role, ranges from sourced rows, derivatives as changes        |
| Component               | `ingredients/prepared.ts`: nesting, cycles, yield         | Yield by weight, percentage lines, baker's percentage base, method steps                   |
| Dish                    | `templates/slots.ts`: roles filled by things              | Dish roles (main, sauce, crunch, acid, garnish, starch) and portions                       |
| Version                 | `specs/`: versions, family, diff                          | Diff fields are lines, yield and method                                                    |
| Tasting note            | none                                                      | A note with a date on one version of a component or dish                                   |
| Stock item              | `stock/` and `shelf/`: what is held, dates, fresh or past | Amount and state on stock, date prepared, use by; the shared collection `stock` of a space |
| Affinity                | `affinity.schema.json` exists in Mixtura                  | Context is cuisine, season, technique; later (issue 27 and the pairing work), no code yet  |
| Export file             | `exchange/`: envelope, version check, migrations          | Culina's entity kinds                                                                      |
| Nutrition, salt figures | none (balance is drink-only)                              | Kitchen rules, derived; see [nutrition](nutrition-evaluation.md)                           |

The shared model's seams matter for the shapes: roles are **data** (an id,
not a closed union), the measure of a template is a parameter (here
always mass), and units are a table, not code.

# Ids

- **Seed ids** are lower-case slugs (`^[a-z0-9]+(-[a-z0-9]+)*$`, at most
  80 characters), readable and stable: `wheat-flour`, `bread-dough`. They
  are never reused for another thing.
- **User ids** are `u-` plus a lower-case UUID, generated by the host
  (the engine has no randomness), so two devices never collide and an
  import cannot overwrite a seed entity.
- One `id` pattern in `common.schema.json` accepts both; "user entities
  use the `u-` form" is a rule of the store, not of the schema.
- **Renaming** changes `name` only; ids never change. Seed entities are
  read-only: to change one, the user makes a new entity with the seed
  one as parent.
- A reference to nothing is not repaired silently: the engine reports
  the line as unresolved and everything derived from it is marked
  incomplete.
- A version is a separate record with its own id (see
  [Version](#version)); `family` is the id of the first version.

# Units and dates

| Quantity         | Stored as                                                                                    | Note                                                                                     |
| ---------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Mass             | number plus `g`, `kg`, `oz`, `lb`                                                            | As entered. Grams are canonical for every computation; conversion is derived             |
| Volume           | number plus `ml`, `l`, `tsp`, `tbsp`, `cup`, `imp-floz`, `imp-gill`, `imp-pint`, `imp-quart` | Only convertible to mass with the ingredient's `densityGPerMl`; otherwise an error       |
| Count            | number plus `count`                                                                          | Only convertible with the ingredient's `unitWeightG` (an egg, a clove)                   |
| Percent of base  | number plus `percent`                                                                        | Component lines only: percent of the summed mass of the `percentBase` role (flour = 100) |
| Yield of an item | percent of the weight as purchased                                                           | Above 100 is possible (cooked rice, pasta)                                               |
| Shelf life       | whole days                                                                                   | Per state and storage                                                                    |
| Date             | `YYYY-MM-DD`                                                                                 | Entered by the user or supplied by the host                                              |
| Instant          | ISO 8601 UTC with `Z`                                                                        | Only `exportedAt`, supplied by the host                                                  |

Rules:

- **Amount** is always a number and a unit id, never a string such as
  `"1 1/2 cups"`; the number is greater than zero.
- **Mass first.** Professional kitchens and bakers work by weight; volume
  and count are conversions that need data about the ingredient, and
  where that data is missing the engine answers "cannot convert", never
  a guess ([engine rules](engine-rules.md)). The unit table (factors, US
  customary definitions of `tsp`, `tbsp` and `cup`) is
  `packages/engine/data/units.json`; the schema only fixes the ids.
- **Dates without a clock.** The engine has no clock. Every function that
  needs "today" takes it as an input `YYYY-MM-DD`; days left are whole
  calendar days between two such dates. The store or the app supplies
  today. Stored dates are those the user entered (`preparedOn`, `useBy`,
  version `date`, tasting `date`).
- **Rounding** happens only at the explicit output step; stored numbers
  keep what was entered.

# Entities

Every table lists the top-level fields of the schema of the same name.
"Required" is `yes` or `no`. All entity schemas set
`additionalProperties: false` and allow vendor fields that start with
`x-`. Nested objects are described under each table.

## Ingredient

A node of the ingredient tree: a category (alliums) or a variety (red
onion). Properties left out are inherited from the parent; a child
overrides. Schema: `ingredient.schema.json`; valid and invalid
examples in `schemas/examples/`.

| Field           | Type                 | Required | Meaning                                                                                  |
| --------------- | -------------------- | -------- | ---------------------------------------------------------------------------------------- |
| `id`            | id                   | yes      | See [ids](#ids)                                                                          |
| `name`          | string               | yes      | Visible name                                                                             |
| `parentId`      | id                   | no       | Parent in the tree; absent for a root                                                    |
| `names`         | object               | no       | Names in other languages, keyed by language tag; see below                               |
| `roles`         | array of role id     | no       | Roles the ingredient can play (acid, fat, aromatic, crunch, starch, umami, flour, water) |
| `season`        | object               | no       | `months` (1 to 12), optional `region`, `source`                                          |
| `allergens`     | array of allergen id | no       | Reserved for issue 27: the 14 of Annex II[^eu]; declared, never derived                  |
| `yields`        | array of object      | no       | `state`, `percent`, `source`: weight in that state as percent of purchased weight        |
| `shelfLife`     | array of object      | no       | `state`, `storage`, `days`, `source`                                                     |
| `densityGPerMl` | sourced number       | no       | Grams per millilitre; needed for volume to mass                                          |
| `unitWeightG`   | sourced number       | no       | Grams per counted piece; needed for count to mass                                        |
| `x-nutrition`   | object               | no       | Nutrients per 100 g (`per100g`) read from the catalogue's dataset; see below             |
| `note`          | string               | no       | Free text                                                                                |

- **Nutrition.** `x-nutrition` is `{ per100g, source? }`. `per100g` has
  `kcal`, `kJ`, `proteinG`, `fatG` and `carbohydrateG` (all five required, at
  least 0) and optionally `fibreG` and `sodiumMg`; a nutrient that is not there
  is unknown, not zero. The catalogue generator reads the values from the
  dataset's own food ([ingredient catalogue](ingredient-catalogue.md#nutrition));
  none is typed by hand. A food row has no `source` (its `x-fdc` and the
  envelope's `dataset` say where the figures come from); a generated parent
  has one that names the rule. Salt is not stored: it is sodium times 2.5
  ([engine rules](engine-rules.md#nutrition)).
- **Names in other languages.** `name` is the canonical English name and
  stays so. `names` is an optional object keyed by language tag (`de`,
  `pt-BR`; two or three lower-case letters and optional subtags), each value
  `{ name, aliases?, source }`: the name in that language, further names it
  is known by (no duplicates), and the `source` (the word `user` or a
  citation, as for every sourced value). They are searched with `name` and
  shown beside it when the match came through them
  ([picker](behaviour.md#ingredient-picker)); the display language
  otherwise stays English, and no rule of the engine reads them. A record
  adopted from the catalogue keeps its `names` unchanged. Example: `"names":
{ "de": { "name": "Rosenkohl", "aliases": ["Sprossenkohl"], "source":
"Wikidata Q150463, label de and aliases, NDB 11098, snapshot 2026-11-18" } }`.
- **Level is derived.** Category or variety is the position in the tree,
  not a field.
- **The tree is the substitution class.** A recipe line that names a
  parent ingredient ("butter") is satisfied by any descendant in stock; a
  line that names a leaf ingredient may, unless the rule is switched off,
  be satisfied by a sibling under the same parent, and the finder reports
  that as a substitution naming the substitute. A sibling never changes
  the amount of the line ([engine rules](engine-rules.md#what-the-stock-allows)).
  What replaces an ingredient beyond its siblings, with what changes, is a
  shipped [substitute effect](#substitute-effect).
  A line cannot name a class that is not a node of the tree; that would
  need a new field on lines and slots and is open.
- **As purchased versus edible portion.** `yields` records, per state,
  the weight that remains as a percentage of the weight as bought;
  yield % = EP weight / AP weight x 100[^yield] (the book's term for EP is
  "edible product"; "edible portion" is Culina's name for it). Purchased to edible
  and back is derived ([engine rules](engine-rules.md#edible-portion)). An ingredient with no recorded yield for a state
  is "unknown", never 100 %.
- **Sourced numbers.** Yield, shelf life, density and unit weight must
  name a source: the word `user` for the user's own measurement,
  otherwise a citation. Nothing is invented.
- **Allergens.** The ids are `gluten`, `crustaceans`, `eggs`, `fish`,
  `peanuts`, `soybeans`, `milk`, `nuts`, `celery`, `mustard`, `sesame`,
  `sulphites`, `lupin`, `molluscs`, one per Annex II entry[^eu]. The field
  exists so that adding the data later changes no schema; Culina's
  engine does not read it before issue 27. Exceptions that Annex II
  lists for some entries, and "may contain", are part of that issue.

## Ratio template

A named formula by mass: parts per role, an anchor role, optional usual
ranges, and derivatives that change a parent. Schema:
`template.schema.json`.

| Field        | Type            | Required | Meaning                                                                                                                                           |
| ------------ | --------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`         | id              | yes      | See [ids](#ids)                                                                                                                                   |
| `name`       | string          | yes      | Visible name                                                                                                                                      |
| `source`     | source          | yes      | Row of [ratios](ratios.md)[^ratios] or `user`: a pointer into the repository, never shown                                                         |
| `references` | array of object | no       | What the ratio card shows as its source: `author`, `title`, optional `page`                                                                       |
| `parentId`   | id              | no       | The template this one derives from; with `changes`, instead of `parts`                                                                            |
| `anchor`     | role id         | no       | Role whose parts are 100 percent in baker's terms; absent means no percentages                                                                    |
| `parts`      | array of object | no       | Roots only: `role`, `parts` (above 0), optional `range` (`min`, `max`)                                                                            |
| `changes`    | array of object | no       | Derivatives only: `add` (role, parts, optional `range`), `set` (role, parts, optional `range`), `remove` (role), `swap` (role, `ingredientClass`) |
| `note`       | string          | no       | Free text                                                                                                                                         |

- **Root or derivative, not both**: a template has `parts` and no
  `parentId`, or `parentId` and `changes` and no `parts` (checked by the
  schema). The family tree is the `parentId` links; children, ancestors
  and siblings are derived. Cycles cannot be excluded by a schema and
  are rejected by the engine with the error `cycle` and the path of
  ids ([engine rules](engine-rules.md#ratio-templates)); `validateModel`
  reports them together with unknown parents ([cross-record
  validation](engine-rules.md#cross-record-validation)).
- **Changes.** `add` puts a new role last, with parts and an optional
  range. `set` gives an existing role other parts and an optional range;
  it keeps the role's `ingredientClass` and drops an inherited range
  unless it gives one, since a range belongs to the parts it was
  sourced for. `remove` and `swap` as before. The rules and errors are in
  [engine rules](engine-rules.md#resolving-a-derivative).
- **Ratios and baker's percentages are one thing.** A ratio 5 flour : 3
  water has `parts` 5 and 3; a baker's formula has the anchor at 100
  (flour 100, water 56, salt 3) and its numbers are percentages. The
  percentage of a role is parts / parts of anchor x 100, derived. This
  is the same mechanism as Mixtura's slot ratios, measured by mass
  instead of volume[^mixshared].
- **Ranges** are in the same parts as `parts`, only from rows of
  [ratios](ratios.md) that give a range (salt in Neapolitan dough 2.8 to
  3.1 %). Where a source gives one value, the template has no range and
  the check reports the deviation only. That `min` is not above `max` is
  checked by the engine (`invalid-range`, also in `validateModel`). The range check says `below`, `within`,
  `above`, never "wrong" ([vectors](vectors.md)[^vectors]).
- Roles are the shared id space of [roles](#roles). `ingredientClass` is
  a node of the ingredient tree.

## Component

A made item with its own output weight: a sauce, a dough, a stock, a
preferment. It is the recipe: amounts by weight or percentage, roles,
nesting. Schema: `component.schema.json`. One stored record is one
[version](#version).

| Field         | Type            | Required | Meaning                                                                     |
| ------------- | --------------- | -------- | --------------------------------------------------------------------------- |
| `id`          | id              | yes      | See [ids](#ids)                                                             |
| `name`        | string          | yes      | Visible name                                                                |
| `role`        | role id         | no       | The dish role it usually fills (sauce, crunch, starch)                      |
| `templateId`  | id              | no       | The ratio template it follows; the range check compares against it          |
| `percentBase` | role id         | no       | Role whose summed mass is 100 percent for lines given in `percent`          |
| `lines`       | array of object | yes      | `ref` (ingredient or component), optional `amount`, `role`, `state`, `note` |
| `yield`       | object          | no       | `amount` (mass) and `source`: the output weight as weighed                  |
| `method`      | array of string | no       | Steps as text                                                               |
| `keeps`       | array of object | no       | How long the made component keeps, per storage place; see below             |
| `steps`       | array of object | no       | Steps by technique or plain action, lines used, note; see [Steps](#steps)   |
| `version`     | object          | yes      | See [Version](#version)                                                     |

- **Keeps (since 2026-11-20, for meal prep).** `keeps` says how long the
  made component keeps and how: one entry per storage place, each
  `{ storage, days, source, note? }` with `storage` `pantry`, `fridge` or
  `freezer`, `days` a whole number, and `note` the user's words for how
  ("in portions, airtight"). Every entry has a `source`; the user's own
  entry is `user`. **A shelf life is a food-safety figure:** a shipped
  component gets one only from an authority that was read, and none is
  shipped today. The stock rules read these entries as a shelf life
  without a state ([stock item](#stock-item)).
- **Lines.** A line may carry a `key`, a short slug that is unique in the component, so that a step can name it (see [Steps](#steps)). An amount is mass, volume or count, or `percent` of the
  `percentBase` mass. A line without an amount ("salt to taste") is kept
  and reported as unmeasured by anything derived.
- **Nesting.** A line may refer to another component by mass of its
  output. Needing 200 g of a sauce whose `yield` is 800 g takes a quarter
  of its recipe; a component without `yield` cannot be used by mass, and
  the engine says so (`no-yield`). Cycles are rejected by the engine
  (`cycle`); see [engine rules](engine-rules.md#components).
  `validateModel` reports unknown components, templates and ingredients
  and nesting cycles by ids alone ([cross-record
  validation](engine-rules.md#cross-record-validation)).
- **Preferments and the two formulas.** A preferment (poolish, biga,
  levain) is a component with its own lines (flour, water). The **final
  dough** formula is the dough's own lines, the preferment as one line;
  the **total formula** counts the preferment's flour and water into the
  dough's flour and water. Professional bakers keep both: the total
  formula is every ingredient, the final mix is the total minus the
  preferment[^ka], and the pre-fermented flour is the share of total
  flour that is in the preferment[^busby]. Culina stores only the nested
  structure; both formulas, the pre-fermented flour and hydration with
  or without the preferment come from expanding it.
- **Yield.** Output weight is entered because cooking loses water and
  only the cook can weigh it; total input and loss are derived. Yield
  percent is not stored.
- **Standardised recipe.** A component with lines, yield and portions
  (on the dish) is the standardised recipe of a professional kitchen:
  quantities, yield, portion size and sub-recipes in one record. Only the
  yield part is in the source: "Professional recipes should always state
  a yield"[^yield].

### Steps

`steps` is the method for the helper: an ordered array, and `method`
(plain text) stays valid beside it for the user's own recipes. A step is
`{ text?, technique?, action?, uses?, time?, temperature?, note? }`. It
says what is done in at least one of four ways, and the schema refuses a
step that has none of them:

- `text` is [a text in both languages](#text-in-two-languages). A shipped
  dish always has it (its schema requires it, and so does the engine,
  `text-incomplete`), and a copy of a shipped dish keeps it. A step the
  user builds has none.
- `technique` is the id of a [technique](#technique), which the helper
  explains once ([engine rules](engine-rules.md#techniques-steps-and-classics)).
- `action` is the id of a **plain action** (since 2026-11-20): a word for
  a step that teaches nothing, such as set aside, drain or serve. The 42
  plain actions are shipped in `data/actions.json` with a name in both
  languages (`ACTIONS`, `actionById`). **Why two lists:** a technique
  needs what to watch, why, and a source; a plain action has none of
  them, and it does not show among the skills that can be marked as
  learned. A plain action becomes a technique when a source says
  something about it: the id moves from one list to the other, and a
  test keeps the two lists free of a shared id. A step names a technique
  or an action, not both (`technique-and-action`); `stepWord` gives the
  one it has. Every step of a shipped dish names one of the two (since
  2026-11-20, [how to choose](classic-format.md#steps)); 12 of the 42
  actions were added for them (cook, pour, sprinkle, crush, grease, trim,
  prepare, turn, take out, turn out, take off the heat, let stand), and
  `validateClassic` refuses an action that is not shipped
  (`unknown-action`).
- `note` is the user's own remark in one language, plain text: what the
  structure cannot say ("until golden", "half of the butter").
- `uses` lists the `key`s of the lines the step uses. **Why keys, not line
  numbers:** the plan says "the lines the step uses", and an index shifts
  as soon as a line is added or moved in the editor, so a step would point
  at the wrong ingredient without any error. A key stays with its line. A
  line without a key can only be unused by steps.
- `time` is `{ value, to?, unit }` with `unit` `s`, `min` or `h`, and
  `temperature` is `{ value, to?, unit }` with `unit` `C` or `F`: one
  number or a range from `value` to `to`, **as the source gives it**,
  never converted. A word ("moderate oven") or another scale goes into
  the text.

## Dish

Role slots filled by components or ingredients. Schema: `dish.schema.json`.
One stored record is one [version](#version).

| Field        | Type            | Required | Meaning                                                                       |
| ------------ | --------------- | -------- | ----------------------------------------------------------------------------- |
| `id`         | id              | yes      | See [ids](#ids)                                                               |
| `name`       | string          | yes      | Visible name                                                                  |
| `portions`   | integer         | no       | Number of portions the dish makes                                             |
| `course`     | string          | no       | Where it sits in a meal: the course list of a [classic](#classic)             |
| `cuisine`    | string          | no       | Whose kitchen it is: `de`, `it`, `fr` or `es`                                 |
| `characters` | object          | no       | The cook's correction of the computed characters: `add`, `remove`; see below  |
| `slots`      | array of object | yes      | `role`, `ref` (ingredient or component), optional `amount`, `optional`, `key` |
| `steps`      | array of object | no       | The assembly: what is done with the fillings; see below                       |
| `version`    | object          | yes      | See [Version](#version)                                                       |
| `note`       | string          | no       | Free text                                                                     |

Weight per portion, the roles covered ("no crunch") and what is missing
from stock are derived. A slot's amount is mass, volume or count, not
percent.

**Course and cuisine (since 2026-11-21).** The two things a shipped
classic states, so that one filter works over every dish
([behaviour](behaviour.md#one-filter-over-every-dish)). Both are
optional and belong to the version like the other fields: a change is a
new version, and the history names it. `course` is one of the classic's
courses (`$defs/course` of `common.schema.json`, the engine's
`COURSE_IDS`); `cuisine` is a country code of `$defs/country`, the same
four a classic's `country` takes (`COUNTRY_CODES`). A copy of a classic
keeps its course, and its country as the cuisine.
**Why only the four codes:** they are the countries the app has a name
for in both languages; a free code would show as a bare `jp` in the
list and the filter. Allowing more later is additive (a wider enum,
and names from the browser's `Intl.DisplayNames`); a narrower list
later would not be. An export with another code is refused by the
schema like any other value it does not know.

**The correction of the characters (since 2026-11-22).** The
[characters](behaviour.md#characters) of a dish are computed and never
stored. What is stored is only the cook's correction, where there is
one: `characters.add` lists ids of the character table the dish shows
although no rule gives them, `characters.remove` computed ones it does
not show. Each list has at least one id, each id once; an object without
a list is not written. The shown set is the computed one without
`remove`, with `add`, in the order of the table
([engine rules](engine-rules.md#the-cooks-correction)); an id the table no
longer knows is ignored, so a row can leave the table without a
migration. It belongs to the version like the other fields.
**Why a difference and not the whole set:** a stored set would freeze
the dish; with a difference a changed recipe still changes the
characters the cook did not touch.

**Assembly (since 2026-11-20).** `steps` are [steps](#steps) of the same
shape as a component's, for what is done once the fillings are made
("stretch the dough, top it, bake"). Where a component's step uses lines,
an assembly step uses **slots**, by the slot's `key`: a short slug that is
unique in the dish, given when a step first uses the slot (`s1`, `s2`,
...). A dish whose fillings only lie side by side on the plate needs no
assembly, or one step. Cooking a dish runs through its components in the
order of its slots and then through these steps
([behaviour](behaviour.md#dishes)).

## Version

Not an entity of its own: the `version` object in a component or dish.
Fields of `version`: `family` (id of the first
version), `date`, optional `parent`, `change`, `reason`.

- A new version is a new record, so every version stays complete and a
  diff is derived. `parent` in the same family is the previous version;
  `parent` in another family makes the record the first of a **variation**
  in a new family, which is how "my version of that" is kept.
- The head of a family is its latest version by date and parent chain;
  the version number is derived.
- **Append-only.** A stored version record is never written again. Save
  adds a record (new id, same family, `parent` = the version edited, `date`
  from the host's clock, optional `change` and `reason` as typed); Restore
  adds a record with the content of an old version and `parent` = the
  current head; a variation is a record in a new family whose `parent` is
  the version it came from. Deleting a component or dish tombstones every
  version of the family and the notes on them together, and Undo restores
  them together; that is the user's delete of the whole thing, not a change
  of history. The app refuses a put of a stored version id
  ([behaviour](behaviour.md#versions-history-and-variations)).
- **Head and number.** The head is the version furthest along the `parent`
  chain in the family; the number is the position in that chain, the first
  being 1 ([engine rules](engine-rules.md#versions-and-variations)).
- **What a reference names.** The model leaves open whether the `ref` of a
  line or slot to a component names a version id or the family. Culina
  stores the id the editor gives it, the **family** id, and treats any
  version id as a reference to the head of that record's family when
  computing, so a sub-component follows the current version. A reference
  never needs rewriting when a new version is saved. `validateModel`
  accepts any record id as a reference; a variation whose parent was deleted
  reports `unknown-parent` there, which the app shows as "parent removed".
- Each version is its own record with its own `id`; that id is what a
  [tasting note](#tasting-note) points at. A note belongs to the version
  it was made on and may be added later; adding one never changes the
  version, so versions stay immutable ([sync](decisions/sync.md)).

## Tasting note

A note on one version of a component or dish, a record of its own
([decision](decisions/sync.md), point 4). Schema: `tasting.schema.json`.
It replaces the `tastings` list that components and dishes had: with sync,
the merge is per record, so two devices or two people adding a note at the
same time never lose one.

| Field  | Type   | Required | Meaning                                                                                        |
| ------ | ------ | -------- | ---------------------------------------------------------------------------------------------- |
| `id`   | id     | yes      | See [ids](#ids)                                                                                |
| `ref`  | object | yes      | `kind` (`component` or `dish`) and `id`: the id of the **version record** the note was made on |
| `date` | date   | yes      | The day of the tasting, as entered                                                             |
| `text` | string | yes      | The note                                                                                       |

- `ref.id` is the id of one stored version, not of the family: a note on
  the second version of a sauce points at that record, so the note stays
  with the version that was tasted. The family and the head of a family
  are found through the version's `version.family`.
- Several notes may point at one version. `validateModel` reports a `ref`
  that is no record of the kind it names (`unknown-component`,
  `unknown-dish`) and a repeated `id` (`duplicate-id`); see [engine
  rules](engine-rules.md#cross-record-validation).
- A note is never part of the version it refers to, so it is not copied
  when a new version is made.

## Stock item

What the user has. Stock is not Culina's own data: it is the collection
`stock` of a **space** in the database `domus` that all apps on the
address share, the same collection as Mixtura's, synced with the space
([stock in spaces](decisions/stock-in-spaces.md),
[spaces on the host](../../../packages/host/docs/spaces.md)). Culina's own
database keeps no stock; a device without a space gets one on this device
only. Schema of the item as Culina shows it: `stock.schema.json`. An item
can be nothing but "this ingredient (or prepared component) is there":
ticking writes exactly that. The kitchen's detail stays optional and can
be added to any item at any time. Days left are derived.

**The item in memory** (what the screens and the engine read):

| Field        | Type    | Required | Meaning                                                                      |
| ------------ | ------- | -------- | ---------------------------------------------------------------------------- |
| `id`         | id      | yes      | See [ids](#ids)                                                              |
| `ref`        | ref     | yes      | An ingredient or a component (a prepared item the user made)                 |
| `amount`     | amount  | no       | Mass, volume or count; not percent                                           |
| `state`      | state   | no       | `raw`, `cooked` or `prepared`                                                |
| `always`     | boolean | no       | The user marks it as always there; it counts as present until it is unticked |
| `preparedOn` | date    | no       | When it was cooked or made                                                   |
| `useBy`      | date    | no       | The date the user entered                                                    |
| `storage`    | string  | no       | Where it lies: `pantry`, `fridge` or `freezer`                               |
| `note`       | string  | no       | Free text                                                                    |

`id` is not stored: it is the record's key for the first item and
`key~n` for the n-th further batch.

**The record in the collection** (one per thing; translated in one file,
`packages/app/src/shared-stock/value.ts`):

- **Key**: the ingredient's id, or for a component the id of its family
  (the first version), whichever version was ticked. Culina shows a
  record whose key is an ingredient of the notebook, else a component
  family of the notebook; any other record is left untouched.
- **Value**: `present` (boolean, required; `false` means not there), then
  the first item's fields under the shared names where the meaning is the
  same: `openedOn` for `preparedOn`, `bestBefore` for `useBy`, `note`; and
  Culina's own optional fields under their names: `amount`, `state`,
  `always`, `storage`. Further items of the same thing (batches kept
  apart) are objects with the same fields in `batches`. A value without a
  boolean `present` is not shown and left untouched; a field that does
  not fit is not shown and stays stored.
- **Fields of other apps** (Mixtura's `moreBatches`, any other) are kept:
  every write is worked out inside one atomic change from the value as
  stored then, replacing only `present`, `amount`, `state`, `always`,
  `storage`, `openedOn`, `bestBefore`, `note` and `batches`.
- **Unticking** writes `present: false` without Culina's fields; the
  record stays, so a merge can tell "out" from "never entered".

Stored as, for presence only and with two batches:

```json
{ "key": "carrot", "updatedAt": "2026-11-18T08:00:00.000Z", "value": { "present": true } }
```

```json
{
  "key": "u-77b2…",
  "updatedAt": "2026-11-18T08:00:00.000Z",
  "value": {
    "present": true,
    "amount": { "value": 400, "unit": "g" },
    "state": "prepared",
    "openedOn": "2026-11-16",
    "bestBefore": "2026-11-20",
    "note": "second shelf",
    "batches": [{ "state": "prepared", "openedOn": "2026-11-18", "storage": "freezer" }]
  }
}
```

Items from before 2026-11-20 (Culina's own kind `stock`, `{ id, ref, ... }`)
are moved into the used space once at the first start after the update
and then cleared from Culina's database
([decision](decisions/stock-in-spaces.md#the-one-move-of-old-items),
[orderings](stock-orderings.md)).

Days left, "use soon" and "past", the limiting shelf life of a dish and
"what can I make, ranked by what must go first" are derived. If `useBy`
is absent, the engine may derive a date from `preparedOn` plus a recorded
shelf life, only when exactly one value applies; it never writes it back
and invents no shelf life. A recorded shelf life applies when its state
is the item's and its place is the item's (`shelfLifeApplies`): an
ingredient records it per state and place, a component per place only
([`keeps`](#component)), so a component's entry applies in any state.
`storage` on the item says where it lies (since 2026-11-20): with it,
only the shelf life of that place counts, which is what makes a date
derivable for a component that keeps 3 days in the fridge and 90 in the
freezer. Without it, and with several places, the screen offers the
dates as suggestions and the user chooses.

## Affinity

Later extension, schema only: two ingredients that go together. Schema:
`affinity.schema.json`. No code and no data before the pairing work;
data would need a source and licence of its own (non-commercial data such as
FlavorDB is allowed since 2026-11-27, [daily driver](decisions/daily-driver.md)).

| Field        | Type    | Required | Meaning                                                   |
| ------------ | ------- | -------- | --------------------------------------------------------- |
| `id`         | id      | yes      | See [ids](#ids)                                           |
| `a`          | id      | yes      | First ingredient                                          |
| `b`          | id      | yes      | Second ingredient; the pair is unordered                  |
| `strength`   | integer | yes      | 1 to 3                                                    |
| `context`    | object  | no       | Optional `cuisine`, `season` (month 1 to 12), `technique` |
| `source`     | source  | yes      | Citation or `user`                                        |
| `confidence` | enum    | yes      | `low`, `medium`, `high`                                   |

## Roles

Roles are data, not a closed list: an id in one shared space for
ingredient roles (acid, fat, aromatic, crunch, starch, umami, and the
ratio roles flour, water, egg, sugar, oil, liquid) and dish roles (main,
sauce, crunch, acid, garnish, starch). The seed list comes with the seed
data; a role id nobody defined is reported as unknown by the engine, not
refused by the schema. There is no role entity yet (open question 2).

**The role table** (`packages/app/data/ingredient-roles.json`, read by
`src/catalogue/roles.ts`) says which catalogue ingredients or groups fill a role
of a ratio card (`flour`, `egg`, `fat`, `sugar`, `oil`, `acid`, `liquid`,
`salt`) or of a dish (`crunch`, `garnish`). The ratio templates already carry
what is needed on their side (each part has a `role`, and the finder matches an
ingredient whose roles include it), and the finder's tree already lets a role
on a group cover every food below it; what was missing is that the USDA data
has no roles, so no catalogue ingredient could fill one. So no second table of
slots was added: the table is only the ingredient side, 23 entries (a group or
a food, its roles, and `basis`: every one is marked as a judgement, with the
ratio card that uses the role). The app adds the roles when it reads the
catalogue, so a copy in the notebook has them (an entry that already has roles
keeps its own, and a copy made before has none). Water has no catalogue entry
and no row; the finder treats the ratio role water as the kitchen's
([engine rules](engine-rules.md#what-the-stock-allows)). A test checks that
every id is in the catalogue and every role is one a ratio card or a dish
knows. The dish lines of the classics carry no role table of their own: a
line's `role` is the slot's, as before.

**The parts table** (`packages/app/data/ingredient-parts.json`, format
`culina-ingredient-parts` 1, read by `src/cook/parts.ts`, 2026-11-21) says
which catalogue foods are parts of a whole, for the finder's part rule
([engine rules](engine-rules.md#what-the-stock-allows)): each entry has
`whole` (an id), `parts` (ids) and `basis` (a judgement with its source). One
entry: the whole egg (`fdc-group-egg-whole`) has the yolk and the white
(`fdc-group-egg-yolk`, `fdc-group-egg-white`). The USDA data says nothing of
parts; nothing is derived from a name. A test checks that every id is in the
catalogue and every entry has a basis.

## Technique

A kitchen technique with the short note the helper shows beside a step.
Shipped read-only, one JSON file per technique in
`packages/engine/data/techniques/<id>.json`, never stored. Schema:
`technique.schema.json`; the guide for writing one is
[classic format](classic-format.md#a-technique-file).

| Field     | Type            | Required | Meaning                                                                             |
| --------- | --------------- | -------- | ----------------------------------------------------------------------------------- |
| `id`      | id              | yes      | See [ids](#ids); equal to the file name                                             |
| `name`    | text in 2 langs | yes      | Visible name                                                                        |
| `watch`   | text in 2 langs | yes      | The short note: what to watch                                                       |
| `why`     | text in 2 langs | yes      | Why it matters                                                                      |
| `level`   | string          | yes      | One of the five [evidence levels](#evidence-levels): how the statement is known     |
| `sources` | array of object | yes      | At least one: `title`, `place`, a `url` or an `isbn`, `licence`, and the `sentence` |

- **Page or book.** A source read on the web has a `url` and a `licence`
  (an SPDX identifier). A printed book has the `isbn` of the edition read
  (13 digits, no hyphens) instead: the app shows it as text without a link,
  and it has no `licence`, because the book is cited as fact and nothing of
  it is reused. The schema refuses a source with neither. Since 2026-11-20.
- **Level of a source.** A source may carry its own `level` when it knows
  the statement in another way than the note says, for example a kitchen
  test (`tested`) cited beside a textbook; the app then labels that source
  with its own level. Since 2026-11-20.
- **Sentence.** Every source carries the sentence the note rests on, at most
  25 words, for the reviewer ([helper plan](helper-plan.md#rules-for-content)).
  The engine counts the words (`sentence-too-long`).
- **Licence.** The `licence` of every shipped source is an SPDX identifier
  (`CC-BY-NC-SA-4.0`, `CC-BY-4.0`, `CC-BY-NC-4.0`), never free text; the
  schemas refuse a value with spaces, and `test/source-licences.test.ts` holds
  the shipped data to it and to the counts of
  [the decision](decisions/sources-licence.md). A later build without what is
  non-commercial reads it.

## Evidence levels

The `level` of a technique note, a handling rule and a substitute says how the
statement is known, in five words (`TECHNIQUE_LEVELS` in the engine; the three
schemas carry the same list and a test keeps them equal; the validators name
any other word, `unknown-level`). The app shows the level as a short label in
front of the source ("Textbook · title, place"):

| Level       | English      | German        | Meaning                                                                                   |
| ----------- | ------------ | ------------- | ----------------------------------------------------------------------------------------- |
| `studied`   | Study        | Studie        | A study measured it; only where a paper was read                                          |
| `textbook`  | Textbook     | Lehrbuch      | A textbook states it                                                                      |
| `tested`    | Kitchen test | Küchenversuch | The author ran an experiment and reports it, for example a tested-kitchen book            |
| `authority` | Authority    | Behörde       | An official body states it (food safety), cited from the authority of the user's language |
| `tradition` | Tradition    | Überlieferung | Kitchens' sources state it, no study                                                      |

`tested` and `authority` were added on 2026-11-19. No shipped note was
re-labelled: all 64 are `textbook`. A level above `textbook` needs a source that
someone read in the book or on the page ([source books](source-books.md)).

## Handling rule

When an ingredient goes into a dish and how it is treated, with the reason
([mentor](decisions/mentor.md), point 8). Shipped read-only, one JSON file per
rule in `packages/engine/data/handling/<id>.json`, never stored. Schema:
`handling.schema.json`; the guide for writing one is
[classic format](classic-format.md#a-handling-rule-file).

| Field       | Type            | Required | Meaning                                                                                  |
| ----------- | --------------- | -------- | ---------------------------------------------------------------------------------------- |
| `id`        | id              | yes      | See [ids](#ids); equal to the file name                                                  |
| `target`    | string          | yes      | A catalogue ingredient id, or a catalogue group id (applies to everything under it)      |
| `text`      | text in 2 langs | yes      | The rule in one or two short sentences: when it goes in, how it is treated               |
| `why`       | text in 2 langs | yes      | The reason                                                                               |
| `technique` | id              | yes      | The skill the rule belongs to; the rule is not shown once that technique is learned      |
| `level`     | string          | yes      | One of the five [evidence levels](#evidence-levels), as for a technique                  |
| `sources`   | array of object | yes      | At least one: `title`, `place`, `url`, optional `licence`, and the `sentence` (25 words) |

- **Group targets.** A rule on a group shows for every ingredient below it in
  the catalogue tree (`parentId`), so it shows on every shipped or own dish that
  uses one. An ingredient the catalogue does not know matches only a rule that
  names it.
- **No state.** What a user has learned is the [skill state](#skill-state); a
  rule has none of its own.

## Substitute effect

What replaces an ingredient and what changes: reference knowledge, always
offered and never faded by the skills the user has learned
([decision](decisions/mentor.md)). Shipped read-only, one JSON file per
substitute in `packages/engine/data/substitutes/<id>.json`, never stored.
Schema: `substitute.schema.json`; the guide for writing one is
[classic format](classic-format.md#a-substitute-file). The finder extends its
sibling rule with them ([engine rules](engine-rules.md#substitute-effects)).

| Field     | Type            | Required | Meaning                                                                                |
| --------- | --------------- | -------- | -------------------------------------------------------------------------------------- |
| `id`      | id              | yes      | See [ids](#ids); equal to the file name                                                |
| `from`    | string          | yes      | The catalogue ingredient or group id that is replaced; a group covers all below it     |
| `to`      | string          | yes      | The catalogue ingredient or group id that replaces it; a group covers all below it     |
| `factor`  | number          | no       | Amount of `to` per amount of `from`, in the same unit; only where the source gives one |
| `changes` | text in 2 langs | yes      | What changes (taste, texture, amount, handling) in one or two short sentences          |
| `level`   | string          | yes      | One of the five [evidence levels](#evidence-levels), as for a technique                |
| `sources` | array of object | yes      | At least one: `title`, `place`, `url`, optional `licence`, and the `sentence`          |

- **One direction per file.** A replacement that the source states both ways
  is two files (`flour-to-cornstarch`, `cornstarch-to-flour`); a reverse is
  written only from what the source says, with a `factor` only if it names one.
- **No factor, no scaling.** Without `factor` the amount of the line stays as
  it is and `changes` says what the source says about amounts.
- **Sentence.** As for a technique: at most 25 words per source, checked by the
  engine; a level `studied` only where a study measured it.
- **Catalogue ids.** `from` and `to` are ids of the catalogue; the app's tests
  check that every shipped one is in it.

## Classic

A shipped dish of one named source, one JSON file per dish in
`packages/app/data/classics/<country>/<id>.json` (`de`, `it`, `fr`, `es`),
never written to the store. Schema: `classic.schema.json`; the field-by-field
guide is [classic format](classic-format.md).

| Field          | Type            | Required | Meaning                                                                                  |
| -------------- | --------------- | -------- | ---------------------------------------------------------------------------------------- |
| `id`           | id              | yes      | See [ids](#ids); equal to the file name                                                  |
| `kind`         | string          | no       | `base` (a base to build on) or `dish`; without it a dish                                 |
| `name`         | text in 2 langs | yes      | Visible name                                                                             |
| `originalName` | string          | yes      | The name in the source's language                                                        |
| `country`      | string          | yes      | `de`, `it`, `fr` or `es`; the folder                                                     |
| `vegetarian`   | boolean         | yes      | Without meat and fish                                                                    |
| `course`       | string          | yes      | `starter`, `soup`, `main`, `side`, `dessert`, `baking`, `sauce`, `breakfast` or `snack`  |
| `source`       | object          | yes      | `author`, `title`, `year`, `place`, `url`, optional `licence`: the one source it follows |
| `persons`      | object          | yes      | `value` (the persons the amounts are for) and `basis` (`stated` or `settled`)            |
| `units`        | object          | no       | Source unit words that are not in the unit table: `kind`, `factor`, `read`               |
| `components`   | array of object | no       | `id`, `name`, optional `role`, `lines`, optional `steps`                                 |
| `slots`        | array of object | yes      | The dish's role slots, as in [Dish](#dish) but with the source's amounts                 |
| `settled`      | array of object | yes      | What we settled: `what`, `value`, `reason`, optional `target`; may be empty              |
| `check`        | object          | yes      | The second check: `method` (`page-image`, `second-edition`, `second-reader`) and `note`  |
| `note`         | text in 2 langs | no       | A short note on the dish                                                                 |

- **Amounts as the source gives them.** A line's `amount` is
  `{ value, max?, unit, original? }`: a number, or a range from `value` to
  `max`; `unit` is an id of the unit table or a word of the classic's `units`;
  `original` is the short original wording ("1 Pfund"). **Why not the
  stored amount shape:** the stored `amount` allows only table units and a
  single number, which cannot carry "2 to 3" or a word the table lacks.
  The engine derives the stored shape ([engine rules](engine-rules.md#techniques-steps-and-classics)).
- **A unit word of the source** that is not in `data/units.json` is defined
  in the classic's `units`: `kind` (`mass`, `volume`, `count`), `factor`
  (grams, millilitres or pieces in one unit) and `read`, where that equivalence was read.
  Nothing historical is in the engine's table.
- **Lines** have `ref` (an ingredient of the catalogue, or a component of the
  same dish) **or** `wanted` (`{ en, de }`, while the catalogue lacks the
  ingredient; a later package brings these to zero), an optional `amount`
  (none: the line shows its name alone), `amountWord` (only on a line
  without `amount`, and only where the source says it: `to-taste` or
  `as-needed`), `role`, `state`, `note` (cooking help, a text in 2 langs),
  `sourceNote` (a remark about the source, a text in 2 langs, shown only in
  the dish's source section), `key` and `settled`. A step may have a
  `sourceNote` too.
- **Settled.** A value we settled where the source is vague is an entry in
  `settled`. A line or slot whose amount we settled has `settled: true` and a
  `key`, and the entry's `target` is `<component id>/<key>` (`slots/<key>` for
  a slot); the line or slot may have no `amount`, and the entry then records a
  choice (its `value` is free text) instead of a number; settled persons (`basis: settled`) have an entry with target
  `persons`. The engine checks that both sides exist.
- **A base** (`kind: base`, [decision](decisions/bases-and-pack.md)) has
  a slot filled by a component; a component no slot reaches whose `role`
  is that of such a slot is an alternative for it. No list of
  alternatives is stored. The import pack is dish files of the same shape
  in `packages/app/data/pack/<country>/`, never a base.
- **No `version`, no `yield`.** A classic is not a stored record, and the
  output weight is something only the cook who weighs it can know.

## Skill state

Whether the user marked one technique as learned: a stored personal record in
the store kind `skills`. Schema: `skill.schema.json`.

| Field       | Type    | Required | Meaning                                           |
| ----------- | ------- | -------- | ------------------------------------------------- |
| `id`        | id      | yes      | `skill-` plus the technique id                    |
| `technique` | id      | yes      | The technique                                     |
| `learned`   | boolean | yes      | True when the user marked it; false takes it back |

- **One record per technique.** The id is derived from the technique, so two
  devices that mark the same technique write the same row and sync merges by
  `updatedAt`, instead of creating two records that disagree. Taking the mark
  back writes `learned: false` and keeps the record.
- Nothing else is stored; which notes show follows from it
  ([engine rules](engine-rules.md#techniques-steps-and-classics)).
- **In the export file since format version 3** (2026-11-21,
  [export file](#export-file)): merged by id like every kind (a mark that
  differs is a conflict the user decides), replaced by "replace everything"
  with a version 3 file, left as it is by an older file, and counted by the
  backup reminder.

## Text in two languages

Shipped helper content is written in English (canonical) and German as
`{ "en": "...", "de": "..." }` (`i18nText` in `common.schema.json`). Both are
required and not empty; the engine also refuses a text of white space only.
A user's own recipe keeps plain strings; only the `text` of a step is in
two languages, because a derived or copied classic keeps its steps as
they are. A step the user builds has no text: it is shown from the name
of its technique or action, which the app has in both languages.

# Stored versus derived

| Stored (user entry or seed)                              | Derived every time                                                      |
| -------------------------------------------------------- | ----------------------------------------------------------------------- |
| Amounts as entered (value and unit)                      | Grams, conversions, scaled amounts                                      |
| Template parts and ranges                                | Percentages, shares, position against the range, family tree            |
| Component lines, `percentBase`                           | Raw ingredients through any nesting depth, total and final formula      |
| Component `yield` (weighed output)                       | Loss, yield percent, portions weight                                    |
| Ingredient `yields`                                      | Purchased to edible and back; unknown stays unknown                     |
| Ingredient density and unit weight                       | Volume and count to mass                                                |
| `version`                                                | Version number, head of a family, diff, variation tree                  |
| Tasting notes (own records)                              | The notes of a version, the latest note of a family                     |
| Stock `always`, `amount`, `state`, `preparedOn`, `useBy` | Days left, use soon, past, makeable, what must go first, what to buy    |
| Ingredient `shelfLife`                                   | Use-by date from `preparedOn`; a dish's limiting shelf life             |
| Allergens as data (reserved)                             | Allergens of a dish: union over its expanded ingredients (issue 27)     |
| (nothing)                                                | Cost per edible unit, nutrition per portion, hydration, salt percentage |

A schema test refuses `totalMassG`, `lossPercent`, `daysLeft`, a stored
`percent` on a template part and a stored ingredient `level`.

# Export

## Ingredient catalogue file

The catalogue shipped with the app is not user data and is not part of the
export. One JSON file, generated by `tools/catalogue/build.mjs`
([ingredient catalogue](ingredient-catalogue.md)) and validated by
`catalogue.schema.json`; the app serves it as `catalogue.json`.

| Field             | Type     | Required | Meaning                                                                                              |
| ----------------- | -------- | -------- | ---------------------------------------------------------------------------------------------------- |
| `format`          | constant | yes      | `culina-ingredient-catalogue`                                                                        |
| `formatVersion`   | constant | yes      | `1`                                                                                                  |
| `dataset`         | object   | yes      | `name`, `release`, `file`, `sha256` of the dataset file read, `licence`                              |
| `refuseDataset`   | object   | no       | `name`, `file`, `sha256`, `licence` of the file the yields' refuse percentages come from             |
| `namesDataset`    | object   | no       | `name`, `file`, `sha256`, `retrieved`, `licence` of the Wikidata snapshot the German names come from |
| `ownNamesDataset` | object   | no       | the same five fields for the project's own German names (`tools/catalogue/names-de-own.json`)        |
| `generatedBy`     | string   | yes      | `tools/catalogue/build.mjs`                                                                          |
| `ingredients`     | array    | yes      | Ingredient records, sorted by `id`, each valid against `ingredient.schema.json`                      |

- **An entry is an ordinary ingredient record**: `id` (`fdc-<FDC id>`, a
  seed-style slug), `name`, optionally `densityGPerMl` and `unitWeightG`
  with a `source` that names dataset, release, FDC id and the exact portion
  text, optionally `yields` (one entry, state `raw`, percent 100 minus the
  refuse percentage of the source, `source` naming file, NDB number, FDC id
  and refuse text), optionally `parentId`, optionally `names` (German name
  and aliases from Wikidata, `source` naming the item, NDB number and
  snapshot date: [German names](ingredient-catalogue.md#german-names)),
  optionally `x-nutrition` (above), and an extension field `x-fdc` (`id`,
  `description`, `category`) with the
  source's full description and USDA's food category
  ([food category](ingredient-catalogue.md#food-category)). No other field
  is present.
- **The catalogue is a tree.** Parents are entries of their own, generated
  from the shared leading segments of the descriptions
  ([grouping rule](ingredient-catalogue.md#grouping-rule)): id
  `fdc-group-<slug of the name>`, `name`, optionally `parentId`, a
  `densityGPerMl` or `unitWeightG` only where every food row below agrees
  (its `source` names the rule and the FDC ids), `x-nutrition` only where
  the rows below agree on energy and the three macronutrients, and the extension field
  `x-group` (`rows`, the number of food rows below, `category` where all
  of them share USDA's food category, and `usual: { id, source }` where
  `tools/catalogue/usual.json` names the row a recipe naming no kind means,
  [usual member](ingredient-catalogue.md#usual-member)), and `names` only where all the foods
  below have the same German name. A food row may be the
  parent of variants whose names extend its own. The `parentId` links form
  a forest without cycles and without unknown parents (a test runs
  `validateModel` and `buildTree` over the file). A variant's `name` stays
  the full USDA description, so a record is meaningful without its parent.
- **Picking an entry copies it unchanged** into the notebook through the
  ordinary save path; the stored record is the catalogue entry, `x-fdc`
  included, and from then on it is the user's own record, editable like
  any other. The id stays `fdc-<FDC id>`, so picking the same entry again
  finds the record that is already there instead of writing a second one.
  The catalogue itself is never written to. The entries above the picked
  one (its parent chain, root first) are copied the same way, only those not
  in the notebook yet, so the notebook's ingredient tree is whole; a parent
  that is picked saves itself and its own parents.

## Export file

One JSON file with everything the user entered, validated by
`export.schema.json`.

| Field           | Type     | Required | Meaning                    |
| --------------- | -------- | -------- | -------------------------- |
| `format`        | constant | yes      | `culina-export`            |
| `formatVersion` | integer  | yes      | `3`                        |
| `seedVersion`   | string   | no       | Seed data the ids refer to |
| `exportedAt`    | instant  | no       | Supplied by the host       |
| `ingredients`   | array    | no       | Ingredient records         |
| `templates`     | array    | no       | Template records           |
| `components`    | array    | no       | Component records          |
| `dishes`        | array    | no       | Dish records               |
| `affinities`    | array    | no       | Affinity records           |
| `tastings`      | array    | no       | Tasting note records       |
| `skills`        | array    | no       | Skill state records        |

```json
{
  "format": "culina-export",
  "formatVersion": 3,
  "seedVersion": "0",
  "exportedAt": "2026-11-17T12:00:00Z",
  "components": [],
  "tastings": []
}
```

- `format` is the constant `culina-export`; `formatVersion` is an
  integer (the same name Mixtura uses; the issue called it
  `schemaVersion`). `seedVersion` names the seed data that ids refer to.
- Arrays `ingredients`, `templates`, `components`, `dishes`,
  `affinities`, `tastings`, `skills` are optional; the export holds user entities, seed entities
  stay in the seed.
- **Forward compatibility.** A reader checks `format` and
  `formatVersion` **first** and refuses a higher version with a message
  that says to update; it never validates a newer file against an older
  schema. Within one `formatVersion` the schemas are closed
  (`additionalProperties: false`), except for fields that start with `x-`,
  which are kept and written back untouched. Any other new field, or a new
  enum value, raises `formatVersion` and comes with a pure migration
  from the previous version to the next. Older files are read by
  migrating step by step.
- **Format version.** Moving tasting notes out of components and dishes
  into `tastings` (issue 50) changes the format, and `formatVersion`
  stays 1 on purpose: nothing is released, so no export file of the old
  shape exists outside development and no migration is written.
- **Version 2 (2026-11-20): no stock.** Stock is the shared collection
  `stock` of a space, not Culina's data
  ([stock in spaces](decisions/stock-in-spaces.md)): a replace import
  would otherwise delete what another app or person put into the space.
  A version 1 file is still read; the migration 1 to 2 leaves its `stock`
  out, `parseExport` counts what it left out (`stockLeftOut`), and the
  import preview says so.
- **Version 3 (2026-11-21): with `skills`.** The [skill state](#skill-state)
  (which techniques the user marked as learned, and marks taken back) is
  the user's own and is now in the file. The migration 2 to 3 changes
  nothing (a version 2 file has no skills); a version 2 file that holds
  `skills` is refused (`unknown-field`), and an app that reads up to
  version 2 refuses a version 3 file as newer (`newer-version`). The
  reader returns the file's own version (`fileVersion`) so that "replace
  everything" with an older file leaves the skill state as it is.
- Identity: importing never overwrites a seed entity, and a user id that
  exists is merged or replaced by the user's choice. Built in issue 18 in
  the engine (`buildExport`, `parseExport`, `planMerge`; [exchange](exchange.md)),
  not with materia's document helpers: the envelope above is Culina's own.
  The file holds live records only (no drafts, no tombstones); `exportedAt`
  is the app clock's instant. The merge rule by id (conflicts listed, never
  "newer wins", version records immutable) is in
  [behaviour](behaviour.md#the-import-screen).
- Cross-record rules a schema cannot state (references exist, no
  cycles, `min` not above `max`, an anchor that exists is a role) are the
  engine's job (`validateModel`) and its findings are named, not silent.
  The anchor is optional, so "one anchor" is not required.

## Recipe import file

The answer a language model of the user's choice writes from a recipe
([decision](decisions/ai-prompt-import.md), [exchange](exchange.md#the-recipe-import-format)),
validated by `recipe-import.schema.json`. It is read, never stored as it
stands: names and amounts as written, no ids; Culina matches the names
with the user's confirmation and writes ordinary components and dishes.

| Field           | Type     | Required | Meaning                                    |
| --------------- | -------- | -------- | ------------------------------------------ |
| `format`        | constant | yes      | `culina-recipes`                           |
| `formatVersion` | constant | yes      | `1`                                        |
| `recipes`       | array    | yes      | 1 to 20 recipes (title, components, slots) |

A recipe has `title`, `persons` or `yield` (written, "1 kg"), `source`
(`title`, `author`, `url`, `where`; at least one), `components` (each
`name`, `role`, `lines`, `steps`), `slots` (each `role` and a `component`
or an `ingredient` by name, `amount` for an ingredient, `guessed`,
`optional`, `alternatives`: names of other components of the recipe) and
`steps` (the assembly). A line is `ingredient`, `amount` ("200 g",
"1/2 TL"), `guessed` (only with an amount), `role` and `note` (the
preparation). Slot roles are the dish roles (`main`, `sauce`, `starch`,
`crunch`, `acid`, `garnish`), line roles those of the ratio cards and
dishes; both are closed lists in this format so that every role the
prompt names is one the app offers.

# Cooklang

[Cooklang](decisions/cooklang.md)[^cooklang] is decided (issue 6) and built
(issue 18, [exchange](exchange.md#cooklang)): own JSON is lossless, Cooklang
is exported per component and imported through a completion step.

- **Own JSON as the lossless format**: the recommendation of the decision
  record. Nothing in this model depends on Cooklang.
- **Cooklang as a one-way export**: an ingredient line maps to
  `@name{qty%unit}` and a line referring to a component maps to a recipe
  reference such as `@./sauces/Hollandaise{150%g}`[^cooklangspec]. Cooklang has no
  percentage notation, so `percent` lines are converted to mass first
  (derived from `percentBase`). Version, tasting notes, roles, ranges and
  yield per state have no Cooklang place and are lost or go into
  metadata.
- **Import (built)**: `amount` is optional on a line and `method`
  holds steps as text, so a recipe without quantities or with units the
  model does not know can still be kept (the unit goes into `note`, the
  line stays unmeasured). Cooklang metadata maps to `source` and `note`;
  an imported file is a component with a new family. Volumes stay
  volumes until a density is known.
- Because both outcomes only add or drop information at the edge, the
  decision does not change a schema.

# Practice behind the choices

How professional kitchens model the same things, and where open formats
fall short for ratio and component thinking.

- **Baker's percentages.** Flour is 100 %, every other ingredient is a
  percentage of the flour weight; formulas are written as total formula,
  preferment and final mix[^ka]. Culina's anchor and `percentBase` are
  that idea for any template.
- **As purchased versus edible portion.** Kitchens cost and plan on the
  edible portion: yield % = EP / AP x 100 and cost factor = 100 / yield
  %[^yield]. The same section says the yield percentage tells "how much
  raw product to actually order" but gives no formula; the amount to
  order as the needed EP divided by the yield is Culina's derivation from
  the first formula. All of this is derived from `yields`; the cost half
  waits for issue 5.3.
- **Standardised recipes and sub-recipes.** The book says "Professional
  recipes should always state a yield"[^yield]; that section names no
  portion size or sub-recipe references, so those are Culina's own
  design, not taken from it: component plus dish.
- **schema.org/Recipe.** `recipeIngredient` takes `Text`, `ItemList` or
  `PropertyValue`: "The ingredients can be represented as free text or
  more structured values", and the markup example on the Recipe page
  uses `PropertyValue` with `value`, `unitCode` and `name`. `recipeYield`
  is `QuantitativeValue` or `Text`, "the quantity produced by the recipe
  (for example, number of people served, number of servings, etc)"[^schemaorg].
  Recipe sites mostly publish free text, but the vocabulary allows an
  amount and a unit. The property definitions read name no percentage,
  no nesting of recipes in ingredients and no yield per ingredient
  state.
- **Cooklang.** Typed quantities with units, recipe references for
  sub-recipes, YAML metadata; no percentages, ratios, yield per state or
  versions[^cooklangspec].
- **Open Recipe Format.** YAML based, designed for "accurate and
  flexible storage of recipes"[^orf]. Each ingredient is a dict with a
  list `amounts` of `amount` and `unit` (several amounts when one recipe
  is stored at several yields), `processing` tags such as "whole",
  "minced" or "steamed", `usda_num`, `notes` and `substitutions`; the
  recipe has `yields` (amount and unit), `steps`, `oven_*` and source
  fields[^orfref]. Typed amounts and units, like Cooklang; no percentages,
  ratios, nesting of recipes or yield per state in the field reference.
  Its licence is not stated on the pages read and stays unread.
- **Mealie and Tandoor.** Mealie's recipe model has a yield as quantity
  and text, an ingredient list, steps and notes[^mealie]; its ingredient
  type was not read. Tandoor's model could not be confirmed from its
  documentation. Both are recipe managers ([existing
  tools](existing-tools.md)[^tools]).
- **Where they fall short** for Culina: none treats a ratio or template
  as the primary thing, none carries a baker's percentage base, a
  yield per state, a sub-component's yield used for scaling, or
  tasting-note versions in the format itself.

# Storage and migrations

Built by issue #15 in `packages/app/src/store/` (the app, not the engine:
the engine stays pure). Only what the user entered is stored; everything
in [Stored versus derived](#stored-versus-derived) is computed.

**Database.** One IndexedDB database `culina`. One object store per entity
kind, named like the collections of the [export file](#export-file):
`ingredients`, `templates`, `components`, `dishes`, `stock`, `affinities`,
`tastings`, and `skills` (technique state, see [Skill state](#skill-state)), each with the key path `key`. The
object store `stock` holds nothing since 2026-11-20: stock is the shared
collection of a space in the database `domus`, and `stock` only held the
items of the version before until their one move
([stock item](#stock-item)); nothing writes it, and `clearKind('stock')`
empties it after the move. Two more: `meta` (`key` to `value`; today
only `schemaVersion`, an integer that starts at 1) and `backup` (empty
except during a migration). No device id yet; sync adds it to `meta`
without a migration.

**Row shape.** A row is materia's `SyncRecord` from the start, so sync
later needs no data migration and `mergeRecords` works on the rows
unchanged:

```json
{ "key": "u-3f1c...", "updatedAt": "2026-11-18T08:00:00.000Z", "value": { "id": "u-3f1c...", "name": "Onion" } }
{ "key": "u-3f1c...", "updatedAt": "2026-11-18T08:05:00.000Z", "deleted": true }
```

- `key` is the record's `id`; `value` is the record, valid against its
  schema; `updatedAt` is an ISO 8601 UTC instant with `Z`.
- **Why `updatedAt` and `deleted` are outside the record:** the schemas
  are closed and the export file holds records, not rows; keeping the
  metadata in the row leaves every record valid against its schema
  unchanged and the export format independent of the store.
- **Removal writes a tombstone** (`deleted: true`, no `value`) and never
  deletes the row, so a later merge knows the record was removed
  ([decision](decisions/shared-model.md)). `get` and `list` do not return
  tombstones; `listRows` returns every row for sync and export. Writing a
  record under a tombstoned id revives it with a newer `updatedAt`.
- The unit test `test/store.test.ts` checks the row shape structurally.
  The engine does not re-export materia's `normaliseRecord` and
  `mergeRecords` yet; it will when sync is built, and the store imports
  nothing from materia directly.

**The clock** is injected: `openStore({ now, indexedDB?, migrations? })`;
the app passes `() => new Date().toISOString()`. **Rule:** a write gets
the later of `now()` and the stored `updatedAt` plus 1 ms, so every write
is strictly newer than the row it replaces, also when the device clock
went backwards. Tested in `test/store.test.ts`.

**Repository.** `get(kind, id)`, `list(kind)`, `put(kind, record)`,
`remove(kind, id)`, `listRows(kind)`, all async, returning
`{ ok: true, ... }` or `{ ok: false, error: { code, ... } }`; they do not
throw for expected failures.

| Code               | When                                                                                    | Carries                      |
| ------------------ | --------------------------------------------------------------------------------------- | ---------------------------- |
| `invalid-record`   | `put` of a record that does not meet its schema (also: no `id`)                         | `issues`: paths and keywords |
| `not-found`        | `get` or `remove` of an id that is not there or was removed                             |                              |
| `quota-exceeded`   | the browser's `QuotaExceededError`; nothing was written                                 |                              |
| `unavailable`      | no IndexedDB, or opening is refused or blocked (private mode), or another storage error | `reason`: the error name     |
| `newer-schema`     | the database was written by a newer app version; it is not opened and not written to    | `found`, `supported`         |
| `migration-failed` | a migration step failed; the old data is untouched and readable                         | `step` (`1->2`), `reason`    |

**Validation.** Every `put` is validated against the JSON Schemas of
`packages/engine/schemas/` before anything is stored. The app's Content
Security Policy forbids `eval`, so the validators are compiled at build
time: `packages/app/compile-schemas.mjs` generates
`src/store/validators.generated.js` with Ajv's standalone code (see
[development](development.md#app-build-service-worker-and-tests)).
Cross-record rules (references, cycles) are **not** checked on a write,
since a reference may arrive later; `checkStore(store, seed?)` loads all
kinds and returns the findings of the engine's `validateModel`.

**Schema version and migrations.** The data schema version is `schemaVersion`
in `meta`, not IndexedDB's own version: IndexedDB's version cannot be
rolled back when a data migration fails, so it only serves to create the
object stores (`onupgradeneeded`), which creates every store that is
missing and touches no row. IndexedDB's version is 1 for the first six
kinds plus `meta` and `backup` and **2** since `tastings` was added
(issue 50), **3** since `drafts` was added (issue 16, see [drafts](#drafts)) and **4** since `skills` was added (issue 87): a browser that already holds version 1 of the database gets the
new store on the next open and keeps its rows (`test/store-tastings.test.ts`
builds the old layout and opens it). Migrations are an ordered list
`{ from: n, to: n + 1, migrate(rows) }` of pure functions over plain data
(`src/store/migrations/`); the schema version is 1 plus the number of
migrations. There is none yet, and none was written for tasting notes or for skills (a new store changes no row):
the data schema version stays 1 because nothing is released. A record stored by the commit before issue 50 could still hold
a `tastings` list inside a component or dish, since the old schema allowed
it; such a record is neither migrated nor validated on read (only `put`
validates), it is rejected when it is written again, and the list is
ignored by everything. This is accepted for development data only; once
a version is released, a change like this needs a migration step.
Opening a database:

1. No `schemaVersion` (new database): write the current one.
2. Newer than the app knows: `newer-schema`, nothing is opened for writing;
   the app tells the user to update.
3. Older: the runner reads all rows, copies them to `backup` with the old
   version, runs the migrations in order in memory, validates every
   migrated record against its schema (and that `key` equals `id`), then
   writes all rows and the new `schemaVersion` in **one readwrite
   transaction** over all stores, clearing `backup` in the same
   transaction.
4. On any failure (a migration throws, a result breaks a schema, the write
   fails) the transaction aborts, so the old rows and the old version
   stay, `backup` is cleared afterwards, and `openStore` returns
   `migration-failed` with the step and a short reason plus a read-only
   handle to the old data. A crash during a migration leaves the old data
   as it was; a leftover `backup` is cleared by the next open.

A reason is the error name and message of the migration code, cut to 120
characters; a migration must not put record text in its messages. Each
migration comes with a before and an after fixture in
`packages/app/test/fixtures/store/` and a test; `test/store-migrate.test.ts`
proves the runner with test-only migrations (a rename of a field, one that
throws).

**Persistence.** `requestPersistence()` wraps `navigator.storage.persist()`,
`persisted()` and `estimate()` and answers `persistent`, `best-effort` or
`unsupported`. It is to be called when the user first saves something, not
at start; Settings shows the current state read with `persisted()`
([behaviour](behaviour.md#local-store)).

**What is logged.** `storeLog(event, { kind, count, ids })`: event names,
kinds, counts, schema versions and record ids, never names, notes or other
text (ids are a branded type that only accepts id-shaped strings). The log
is a no-op unless `localStorage['culina.debug']` is `1`.
`test/store-log.test.ts` puts a distinctive note through put, get, remove
and a failing migration and checks that the log does not contain it.

## Drafts

An editor keeps what the user has typed but not saved in its own object
store `drafts` (IndexedDB layout version 3; created on the next open, rows
of an older database kept, `test/store-drafts.test.ts` builds the version 2
layout). A row has the shape of every row, `{ key, updatedAt, value }`, but
the value is the form state and is **not validated**: an unfinished form
may break every schema. The key names the draft: `component:new`,
`component:<id>`, `dish:new`, `dish:<id>`. Drafts are not records: they are
not in `list`, `listRows`, the export file, the backup or any sync, and the
log never carries their content. A removed draft is deleted outright, since
nothing needs a tombstone. The data schema version stays 1. API: `store.drafts`
with `get`, `put`, `remove` and `list`.

# Schemas and tests

- `common`, `ingredient`, `template`, `component`, `dish`, `stock`,
  `affinity`, `tasting`, `export`, `catalogue`, `technique`, `substitute`, `classic`, `skill` in `packages/engine/schemas/`, JSON Schema draft
  2020-12[^jsonschema]. Each has an absolute `$id`
  (`https://nexumia-de.github.io/culina/schemas/<name>.schema.json`; an
  identifier, not necessarily a place to download), shared types live in
  `common.schema.json` under `$defs` and are referenced with `$ref`, and
  the export schema refers to the entity schemas.
- Every entity schema is closed with `additionalProperties: false` and
  allows `x-` fields through `patternProperties`.
- `packages/engine/schemas/examples/<name>.valid.json` and
  `<name>.invalid.json`: one of each per schema.
  `packages/engine/test/schemas.test.ts` validates them with Ajv[^ajv]
  (draft 2020-12, strict mode), adds the invalid cases the issue names
  (missing unit, negative amount, unknown state), refuses derived fields
  and checks that these tables and the schemas agree.
- The examples are test fixtures with the user's own made-up entries
  (`source` is `user`), not seed data; the ratios and ranges in the
  template example are rows of [ratios](ratios.md).

# Open questions

For the owner; each has a proposal.

1. **Where the general schemas live** (issue 2). _Proposal:_ keep
   `common.schema.json` here until the shared package exists; then replace
   the general types by a reference to the shared schema and keep the
   kitchen fields; the table-agreement test then reads both.
2. **Roles: id space and a role entity.** _Proposal:_ open ids with a
   seed list; no role entity until a screen needs names and descriptions.
   The ratio roles (flour, water, egg) and dish roles share one space, so
   "crunch" can be both.
3. **Preferment as a component, not a flag.** _Proposal:_ as written: a
   preferment is a component; total formula, pre-fermented flour and
   hydration come from expansion. A sourdough starter's own hydration is
   then simply the starter's lines; counting or not counting it in
   overall hydration is a choice of the function (issue 8), because
   sources differ[^busby].
4. **States.** `raw`, `cooked`, `prepared` are a closed enum; peeled,
   trimmed, roasted or frozen are not states. _Proposal:_ keep three; a
   fourth is a format version bump. Trim loss is already inside
   `yields` from raw to the edible state.
5. **Cup, tablespoon, teaspoon.** _Decided (issue 8):_ US customary, in
   the unit table with exact factors (cup = 231 · 2.54³ / 16 ml =
   236.5882365 ml, tbsp = cup / 16, tsp = tbsp / 3); no metric cup. See
   [engine rules](engine-rules.md#units-and-factors).
6. **Allergens before issue 27.** _Proposal:_ keep the reserved field
   (data may be entered, the engine ignores it), and let issue 27 decide
   "may contain" and the Annex II exceptions.
7. **One export file for everything or per kind.** _Proposal:_ one file,
   as written; a single recipe handed to someone else is a file with one
   component and what it references.
8. **Name of the version field**: `formatVersion` instead of the issue's
   `schemaVersion`. _Proposal:_ `formatVersion`, equal to Mixtura, so a
   shared exchange module needs no mapping.
9. **Cross-record validation.** _Decided and implemented (issue 36):_ a
   pure `validateModel` in the engine with named findings for unknown
   references, cycles, anchors that are no role, `min` above `max` and
   duplicate ids; see [engine rules](engine-rules.md#cross-record-validation).
10. **Device id for sync.** _Decided (issue #15):_ not stored now; the rows
    already are sync records, and the id is added to `meta` when sync is
    built. See [Storage and migrations](#storage-and-migrations).
11. **Tasting notes: a list or records.** _Decided (issue 50, from the
    [sync decision](decisions/sync.md)):_ records of their own, pointing at
    the version they were made on; `formatVersion` stays 1 because nothing
    is released. See [Tasting note](#tasting-note).
