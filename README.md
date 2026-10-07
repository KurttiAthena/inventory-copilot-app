# Inventory Copilot (Concept App)

## Premise (abstract) 
App which allows you to ask questions about your game's inventory in natural language. App sorts / organizes inventory externally and presents the "sorted" items based on user's original prompt.

### Example questions: 
- Which weapons can I sell without losing my best option in any class?
  - _Partition -> rank -> preserve maxima -> complement_
- Keep one weapon of every type and optimise the rest for maximum sale value.
  - _Set-cover-like constraint + optimization_
- Which pets should I release if I need 20 spaces while retaining roughly equal species representation?
  - _Constrained optimization / balancing_
- Do I own anything rare that I'm carrying duplicates of?
  - _Filter + group + aggregation_

### Notes:
- AI itself does not manipulate inventory or write code / functions
  - it translates what the user says into a safe structured query (like a recipe) that the program then executes, see App Workflow below
  - it will not sell/release/delete/dismantle on its own or take control of the game to do such actions
- Eventual desktop integration of the app if possible
- Biggest limitation: there would be no universal way to "sync" the inventory to any game
  - either application can be made as a mod that sends inventory to the program -> that way official game modding support can be used
  - or screenshots could be uploaded to the application and computer vision / OCR used to scan the inventory
- Launches whenever the game launches -> either through user configuration and / or monitoring desktop processes until game is booted up and app gets booted up with it
  - for games that allow modding, this would help with auto-syncing the inventory / refreshing inventory database
- Idea is for people to be able to branch from this project to make adapters for the games THEY like, so instead of me making an adapter for every game, I just try for a few to give examples of how it should roughly be structured (i.e. Minecraft, Voidpet Garden, Cyberpunk 2077, Zelda)

## App Workflow: 
1. User prompt, e.g. "Keep the two best pistols of each type and sell duplicates, but never sell Iconics."
2. Local AI / LLM prompt parser creates the structured rule, e.g. "Each gun category -> identify 2 'best' corresponding to highest attack and shots stats, recommend keep; recommend all others to sell; if one in Other == Iconic then recommend keep."
3. The _Universal Engine_ handles filtering, ranking, grouping, optimization of the inventory and explanations (short "reason" why it flagged that item over another one, e.g. "strongest / highest stats in X category" or "slowest reload capacity," etc.)
4. _Universal Engine_ then normalizes the inventory format -> requires passing through individual game adapter for categories to make "sense," e.g. "type" of gun vs "rarity" of gun cannot be considered equal, must have a "defined" way to sort against multiple identification categories
5. Local LLM reads output info of the sort and returns recommended actions based on adapter context and engine recommendation -> the AI itself doesn't "decide" or "recommend" what is best, only returns the natural-language response of the output of the engine after it's filtered / optimized / sorted the inventory in the desired way

A more precise example: user prompts "Get rid of low-stat Anxietys and Angers, but keep any vivid ones and always keep my strongest three of each."
LLM then generates something like:

{
  "action": "recommend_remove",
  "filter": {
    "species": ["Anxiety", "Anger"]
  },
  "protect": {
    "vivid": true
  },
  "group_by": ["species"],
  "keep": {
    "count": 3,
    "sort_by": "overall_stats",
    "order": "descending"
  }
}

Then _Universal Engine_ code executes it. The above code snippet example could be formatted properly and called something like Inventory Query Language, IQL. 

A small LLM would already be able to go from natural language to structured JSON without reinforcement learning / a training dataset. In this way having a local AI within the application would be the most practical -> game inventory would never need to leave user's machine. Another advantage would be that a small LLM could have memory of user preferences, such that it could generate "rules" for specific game inventories that would make its suggestions better (i.e. never suggesting to remove certain types of items, favoring certain items over others, etc.) that the user would be able to review and change at any time. 

The LLM can also be programmed to have "rules" such as "always outputting what it understood of the user input BEFORE creating the IQL function that would sort the inventory," so users have a chance to correct its reasoning if wrong. 
For example, it outputting even something simple such as:

"I understood this as:

✓ Consider pistols only
✓ Never sell Iconic items
✓ Group pistols by subtype
✓ Keep the top 2 in each subtype
✓ Recommend selling remaining items

Confirm sorting operation ? / Run analysis ?"


Thus 2 parts to the entire system: 
1. Inventory sorting and acquisition -> game adapter handles the "context" of the items, what gets ordered and how / to what respect with other categories -> different for each game, supplies the "schema" (_see below_) to the AI
2. Inventory reasoning -> handled by Universal Engine, specifically "what items get flagged under user's prompt ?" (e.g. "which ones should I remove ?")

## Normalized Item Schema ? 

Create a sort of "normal item identification card" / structure e.g. 

class InventoryItem:
    id: str
    game_id: str

    name: str
    category: str | None
    subtype: str | None

    quantity: int

    rarity: float | None
    level: float | None
    value: float | None

    stats: dict[str, float | str | bool]

    tags: set[str]

    locked: bool
    equipped: bool
    favorite: bool

where _game-specific_ data goes into "stats" -> the query engine doesn't need to know what they mean, game adapter describes them and then sort / ordering can be applied

## InventoryItem things to include
- unique ID (to be able to differentiate duplicate items)
- game ID (if available)
- name of object
- category
- subtype
- quantity
- rarity
- level
- monetary value
- arbitrary stats (will vary by game)
- arbitrary tags
- item state (locked / equipped / favorite / starred / etc.)

### CyberPunk 2077 example:
Adapter first sorts things into the following categories: 
category:
  weapon
  clothing
  consumable
  ...

weapon.subtype:
  pistol
  revolver
  shotgun
  ...

attributes:
  damage
  attack_speed
  reload_time
  magazine_size
  tier
  sell_value

tags:
  iconic
  crafted
  quest_item

protected_states:
  equipped
  favorite
  iconic
  quest_item

THEN ALSO SUPPLIES THE CONTEXT TO THOSE CATEGORIES, as the sorting engine won't know whether certain values being low or high are better / worse. 

For example, each item could include a GameSchema alongside the inventory:
AttributeDefinition(
    key="damage",
    type="number",
    direction="higher_is_better",
    comparable_within=["weapon.subtype"],
)

AttributeDefinition(
    key="reload_time",
    type="number",
    direction="lower_is_better",
    unit="seconds",
)


## The problem behind optimization
Relevant for games which have an inventory capacity, users who want equally-distributed amounts of categories of inventory, favoring of certain qualities / stats over others, "choice" priority of certain inventory item types, etc. -> solved algorithmically, e.g. 

maximise:
    total collection quality

subject to:
    inventory_count <= 100
    representation(species) ≈ equal
    keep(vivid) = true

Potential libraries for this: 
- scipy.optimize
- OR-Tools
- PuLP

## Repo Structure 
- software architecture
- optimisation code
- LLM structured output (it shouldn't be able to have conversation beyond discussion of the items)
- local AI itself
- data modelling
- game modding
- desktop development / configuration instructions

### Python ?
- inventory model
- query engine
- game adapters
- LLM integration
- optimiser
- tests

Consider Rust, PySide6, TypeScript, Lua, JSON / Websocket...

### Things to include specifically on GitHub
inventory-copilot/
│
├── core/
│   ├── inventory.py
│   ├── queries.py
│   ├── optimiser.py
│   └── rules.py
│
├── ai/
│   ├── parser.py
│   ├── schema.py
│   └── prompts/
│
├── adapters/
│   ├── demo/
│   ├── cyberpunk2077/
│   └── voidpet/
│
├── desktop/
│
├── tests/
│
├── examples/
│
├── docs/
│
├── plugin-sdk/
│
└── README.md

Allowing GitHub Actions to eventually build Windows executable, package + publish GitHub release

## Stages / Versions to build 
0. Specification of an item (system architecture) -> InventoryItem definition / development, GameSchema structure, IQL, and adapter interfaces 
1. Universal inventory schema + manually loaded JSON/CSV (example inventory to use for tests) -> fake inventory datasets that are similar to the datasets from various games (CyberPunk-like, Voidpets-like, RPG-like)...
2. Deterministic filters/rank/group queries -> develop the logic behind concepts i.e. filter, group, sort, aggregate, protect, keep/retain, compare, etc.
3. Explanation engine ("reason" concatenator) -> deterministic reason codes that then get output as strings in separate format, based on definitions provided in game-specific adapter engine (basic POC / prototype structure)
4. Optimization engine -> builds mathematical / logical functions that represent conceptual data i.e. capacity, diversity, minimum representation, weighted utility
5. Natural language -> structured query, potentially unique language (Inventory Query Language, IQL) -> initially API/model agnostic
6. Small local LLM (prototype, run on Streamlit ?) -> constrained and structured output
7. Introduce first _real_ game adapter for a game allowing mods (i.e. CyberPunk 2077) -> demo UI, importation of data types (JSON / CSV / game-specific files), receiving user request / question, inspection of interpreted rule, inspection of results
8. Live inventory watching / synchronization
9. Desktop shell / tray app + game startup detection
10. Second game adapter for a closed-mod game (i.e. Voidpet), radically different sorting tasks for reference 
11. Plug-in SDK
12. Polished Windows release / community adapters

## Software architecture
1. Natural language
2. IQL compiler
3. Schema validation
4. Semantic validation against GameSchema
5. Query planner
6. Execution

## Types of query 
1. **Deterministic** -> just asking to "show" items within a sorting category (e.g. "all legendary pistols," "duplicates worth >500," "lowest damage weapon per category") -> no optimization needed
2. **Preference / ranking** -> "best" type / category isn't objective; adapter can either have a "default" definition of each category (e.g. "default shotgun ranking has X units damage, Y fire rate, Z reload speed, A units range, F-level handling") and then the adapter organizes "best" as anything above those values; OR user defines "best" (e.g. "prefer damage values to reload speed values") and adapter then gives higher ranking priority to weapons which do more damage than those which do less but are faster to reload
3. **Constraint optimization** -> dealt by OR-Tools or PuLP, for questions such as "Recommend which pets to release to free 20 slots while keeping at least two of every species, all vivids, and maximizing average stats."

## Adapter capabilities
Some games would allow for direct live syncing due to mods. Others might be so closed that only screenshots of the game / manual input could allow for sorting. Several adapter capabilities could be defined as the following (example):
- IMPORT_FILE
- MANUAL_ENTRY
- MOD_API
- SAVE_FILE
- LIVE_MEMORY
- SCREENSHOT
- LIVE_SCREEN

## Screenshot ingestion
Probably best as being external, e.g. adapter separate from OCR reader, so that screenshot extractor could be created or updated or improved without needing to fully change any specific game adapters themselves. 

Separate project idea...? Hm...

## Initial / desired games 
Having a few different "types" with different "inventory categories" to ensure the app fits a variety of use cases.

### Cyberpunk 2077
Contains lots of:
- stats
- equipment
- values
- rarity / tier
- weapon categories
- special protected items

Would be very good for ranking by price, capacity, etc. This "frame" of architecture would also cover a wide variety of shooting, open-world, RPG, and simulation-type games. 

### Voidpet Garden
Focus would be on
- collection management
- variants
- duplicates
- individual pet stats
- species representation / diversity and weighing
- limited slots

Benefit targeted for games that would require optimization / collection-ordering / personal rules for user preferences and thus opinionated decision-reasoning. 

### Minecraft
Includes concepts such as 
- space management
- stacks (stackable vs non-stackable items) -> stack-aware optimization
- storage hierarchy
- resource allocation
- player-defined "junk" / preference items
- resources with multiple downstream uses

Introduces a more complex idea than "quantity," as maximum stack size changes "item amount" consideration and usefulness for crafting favors certain item types / basic items that are preferred by the user.

### Zelda: Tears of the Kingdom
This would primarily prioritize:
- limited weapon / bow / shield slots
- durability and expendable equipment
- Fuse combinations between weapons and materials
- contextual usefulness rather than simple "highest stat = best"
- elemental / situational utility
- scarce or rare materials with competing uses
- loadout diversity across weapon types
- opportunity cost between selling, cooking, upgrading, and fusing materials

Benefit targeted toward games where inventory decisions depend heavily on context, scarcity, item combinations, and preserving versatility, rather than simply ranking individual items by a fixed set of stats.


## Key concepts
- LLM structured outputs
- formal schemas / DSL
- constraint optimization
- desktop software
- plugin architecture and guidance
- game modding
- data modelling
- computer vision
- software safety / validation
- user testing / community adapters 
