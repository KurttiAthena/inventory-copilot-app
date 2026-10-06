# Inventory Copilot (Concept App)

## Premise (abstract) 
App which allows you to ask questions about your game's inventory in natural language. App sorts / organizes inventory externally and presents the "sorted" items based on user's original prompt.

### Example questions: 
- Which weapons can I sell without losing my best option in any class?
- Keep one weapon of every type and optimise the rest for maximum sale value.
- Which pets should I release if I need 20 spaces while retaining roughly equal species representation?
- Do I own anything rare that I'm carrying duplicates of?

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
5. Local LLM reads output info of the sort and returns recommended actions based on adapter context

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
1. Universal inventory schema + manually loaded JSON/CSV (example inventory to use for tests)
2. Deterministic filters/rank/group queries
3. Natural language -> structured query
4. Small local LLM (prototype, run on Streamlit ?)
5. Introduce first _real_ game adapter for a game allowing mods (i.e. CyberPunk 2077)
6. Live inventory watching
7. Desktop tray app + game startup detection
8. Second game adapter for a closed-mod game (i.e. Voidpet)
9. Plug-in SDK
10. Polished release / community adapters

