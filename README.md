# Railroader 0.6 Map Modding Guide

How to build on the Railroader 0.6 base map with the game's own Map Editor and mod files: new terrain, levelled
ground, track and industries, released as a mod that adds them to the base map.

The base map isn't officially open for building yet, and the editor and file formats may still change during
Experimental. This guide will change with them. Replace `yourname/...` with your own ids throughout.

## Contents

1. [Two mods: editing and release](#1-two-mods-editing-and-release)
2. [Set up your editing mod](#2-set-up-your-editing-mod)
3. [Add terrain tiles](#3-add-terrain-tiles) (optional)
4. [Shape the ground with terrain masks](#4-shape-the-ground-with-terrain-masks)
5. [Add industries](#5-add-industries)
6. [Save and check your changes](#6-save-and-check-your-changes)
7. [Make the release mod](#7-make-the-release-mod)
8. [Install and enable](#8-install-and-enable)
9. [Test](#9-test)

---

## 1. Two mods: editing and release

Building on the base map uses **two mods**. Keep them apart:

| | Editing mod | Release mod |
|---|---|---|
| **Where** | `Documents\Railroader\ModDev\` | `Documents\Railroader\Mods\` |
| **What it's for** | Working in the Map Editor. Only for you. | What you and other players install and play with. |
| **Made by** | The game's **+ Create Mod** button | You, by hand, from the editing mod's files |
| **Holds** | Your Map definition and the changes the editor saves | Only your changes, plus a small script that adds them to the base map |

If you launch the game with `-homedir`, both folders are inside the folder you gave it instead of
`Documents\Railroader`.

### Don't copy the game's map

The base map is the game's own pack, `rr/nc-bryson-terrain`, in the game's install folder
(`Railroader_Data\StreamingAssets\AssetPacksCore\nc-bryson-terrain`). Don't copy it into ModDev or edit it there:

- The copy has the same id as the game's pack, which confuses the game.
- Saving writes the **whole** base map with your changes mixed in, not just your changes. That file can't be released
  as it is: it would put back your copy of everything on the map, undoing later game updates and other mods' changes.
- Changes made in the install folder are lost when Steam updates the game.

Your own editing mod is layered **over** the base map instead. The base map still shows in the editor, but Save only
writes what you changed.

If you've already edited a copy, its `Data\Map.json` holds your changes inside the whole map. Set up your own editing
mod as below and make the changes again there, then remove the copy from ModDev.

---

## 2. Set up your editing mod

Your map has to have a tile set before it opens, or it fails to load with "Error loading server game". So **don't
click Edit Map until step 2.7**: steps 2.1 to 2.6 only set up files. You need the tile set even if you won't add any
terrain.

### 2.1 Create the mod

1. Start the game and open the **Definition Editor**.
2. Click **+ Create Mod**, at the bottom of the mod list.
3. Fill in:
   - **Identifier:** `yourname/my-mod-dev`. Use your own name before the `/`. Lowercase, no spaces.
   - **Display Name:** e.g. `My Mod (dev)`.
   - **Author:** your name.
4. Click **Create**.

The game creates `Documents\Railroader\ModDev\my-mod-dev\` (the part of the id after the `/`) with an `Info.json` and
an empty `Definitions.json`, and selects the new mod in the list. A ModDev mod doesn't need an entry in
`Preferences.json`.

### 2.2 Create the Map definition

With your mod selected, click **+ New**, choose **Map**, enter an id and click **Create & Edit**. The game saves it to
your mod's `Definitions.json` straight away and shows "Map Definition Saved".

**Don't use Create Terrain:** it sets the origin to the coordinates you type, which can't be layered with the base
map's terrain.

### 2.3 Add the map's layers

Do this either in the editor or in the file. Keep your tile set first, because terrain is read-only if another mod's set
comes first.

**In the editor:**

- **a.** Click **Edit** on your Map. This is the small button beside **Edit Map**, not Edit Map itself.
- **b.** Under **Object Definition** on the left, select **Map**.
- **c.** Click **+** on each list to make a row, then type the id. Press Enter or click away after each one:

  | List | Row | Id |
  |---|---|---|
  | **TerrainTileSets** | `[0]` | `yourname/my-terrain` |
  | **TerrainTileSets** | `[1]` | `rr/nc-bryson` |
  | **MapLayers** | `[0]` | `rr/bushnell-whittier` |

  **-** deletes the last row. You'll type your tile set's id before it exists, which is fine: the editor only checks
  the id's format.

- **d.** Click **Save**, at the right-hand end of the bottom row of buttons. The **Log** window can cover it, so move
  that if you need to. **Save Component** at the top is a different button.

**Or in the file:** close the game, then in your mod's `Definitions.json`, find your new Map definition and add these
fields to its `definition`:

```json
"mapLayers": ["rr/bushnell-whittier"],
"terrainTileSets": ["yourname/my-terrain", "rr/nc-bryson"]
```

### 2.4 Close the game

Close the game if it's still open, and go to your mod's folder in `ModDev`.

### 2.5 Add the tile set

The **+ New** list has no tile set type, so this part is a file edit. In `Definitions.json`, add the tile set to the
`objects` list:

```json
{
  "identifier": "yourname/my-terrain",
  "metadata": { "name": "My Terrain", "description": "", "tags": [], "credits": "" },
  "definition": { "kind": "TerrainTileSet", "path": "Terrain",
                  "attribution": "Elevation: Mapterhorn (mapterhorn.com/attribution)" }
}
```

### 2.6 Create `Terrain.json`

Create `Terrain\Terrain.json` with exactly this. The origin must match the base map's exactly, so copy the digits:

```json
{"origin":{"latitude":35.38261413574219,"longitude":-83.49540710449219},"tileDimension":500,"heightmapResolution":513,"tiles":[]}
```

The empty `tiles` list is correct: the map has no tiles of its own yet.

### 2.7 Open the map

Start the game, open the Definition Editor, select your mod in the list and click **Edit Map**. The base map's terrain,
track, areas and industries all show. You can't change the base map itself, but your map saves your changes and
additions on top of it.

Your editing mod now looks like this:

```
Documents\Railroader\ModDev\my-mod-dev\
  Info.json
  Definitions.json      your Map and tile set definitions
  Terrain\Terrain.json  your terrain tiles (none yet)
  Data\Map.json         your changes to the base map, written by Save (section 6)
```

---

## 3. Add terrain tiles

Only needed if you want to build beyond the edge of the base map. To change the ground **on** the base map, use
masks ([section 4](#4-shape-the-ground-with-terrain-masks)).

> **Tested so far:** fetching, checking `Terrain.json` and the seams in the game on a test map. The release steps were
> tested with other tiles, not with tiles fetched here.

### 3.1 Choose your tiles

- **Tile size and position:** each tile is 500 x 500 m. Tile `(x, y)` starts at world `(x * 500, y * 500)`, with X
  running east and Z running north.
- **Tiles already in use:** check
  `Railroader_Data\StreamingAssets\AssetPacksCore\nc-bryson-terrain\Terrain\Terrain.json`. Pick tiles that aren't listed
  and that touch the existing edge of the map.
- **Finding the real-world location:** the same file gives the map's origin, about 35.3826, -83.4954. These formulas
  are close enough for choosing an area, but they drift slightly:
  - latitude = `35.3826 + Z / 111320`
  - longitude = `-83.4954 + X / 90760`

### 3.2 Fetch the terrain

1. In the Map Editor, select **Terrain**, set **Mode** to **Tiles** and click **Fetch Terrain...**. The Tiles mode
   shows "Coverage: 0 tiles" at first.
2. Fly to the area. The fetch is centred on the camera itself, not on the point it's looking at, so check **Center
   Tile** in the window before you start.
3. Set **Radius** and click **Fetch**. Radius is in tiles: 0 is one tile, 1 is 3 x 3, 2 is 5 x 5.

**Keep the fetch area off the base map.** Fetched tiles replace base tiles at the same position. If you fetch one by
accident, close the game, delete its `tile_XXX_YYY.riff` and remove its entry from `Terrain.json`.

Tiles are written to `Terrain\` as soon as they're fetched. Reopen the map with **Edit Map** to check them.

### 3.3 Check `Terrain.json`

Open `Terrain\Terrain.json` and check that:

- `origin` still has the digits from step 2.6
- `tiles` has the positions you expected, with none from the base map's list
- `tileDimension` is 500 and `heightmapResolution` is 513

### 3.4 Check the seams

Neighbouring tiles need matching heights along their shared edges, or you'll see steps in the ground.

In our test, fetched tiles met the base map with no step at the 14 points measured along one 1.5 km edge. That's only
one edge, so check yours.

- **Inspect:** look along every edge between your tiles and the base map.
- **Fix:** use the editor's flatten and smooth brushes on your side of the edge, then save.
- **Limitation:** brushes only get the edges close, not identical, and the base tiles can't be edited. To shape the
  ground on the base map's side, use masks ([section 4](#4-shape-the-ground-with-terrain-masks)).

Terrain doesn't add any track, so lay that in the editor as well if you want to run trains there.

---

## 4. Shape the ground with terrain masks

The base map's terrain tiles belong to the game, so the terrain brushes can't change them. If you try, the editor says
"This map's terrain comes from another pack's tile set, which this session cannot write to." Brushes only work on
tiles your own mod fetched ([section 3](#3-add-terrain-tiles)).

To level ground on the base map, use **terrain masks**. A mask is a map object, like a prop: it's saved in your map
file with your track and industries and goes in the same release file. The base map shapes its own yards and industry
sites the same way, with 144 masks.

### 4.1 Do you need one?

Track already shapes its own roadbed: the ground is cut or filled to the track along a narrow strip, blending into the
terrain over 20 m on each side. A single track on a gentle slope usually needs nothing more.

Use a mask where the roadbed isn't enough:

- a yard or several tracks side by side
- a level pad for an industry, a loading area or a building
- a road or parking area
- a wider or smoother cut or fill than the roadbed makes

### 4.2 Add a mask

In the Map Editor, open **Create** below the outline, then **Terrain Masks**, and choose a shape:

| Shape | Footprint | Use it for |
|---|---|---|
| **Circle** | **Radius** around its position | small pads, around a building |
| **Rectangle** | **Size X** by **Size Z**, plus **Radius** around it | yards, industry pads, parking areas |
| **Curve** | **Radius** each side of a curve from end A to end B | along track or a road, cuts and fills, a graded slope |

The mask is placed on the ground where the camera is looking. Move and turn it like any other object; a curve's two ends
can also be moved on their own.

### 4.3 Set it up

Select the mask and set these in the inspector. The editor shows the same descriptions as tooltips.

- **Flatten Heightmap:** turn this **on**. This is what levels the ground. Without it, the mask doesn't change the
  height.
- **Height:** the ground is levelled to the mask's own height, the Y of its **Position**. For a curve, the height runs
  along the curve from end A to end B, so ends at different heights make an even slope.
- **Radius:** the footprint, in metres. On a rectangle it's added around the size and rounds the corners. On a curve,
  **Size A** and **Size B** scale it at each end.
- **Falloff:** how far beyond the footprint the levelled ground blends back into the terrain, in metres. Use more for a
  gentler bank: the base map uses anything from 0 to 50 m.
- **Order:** which masks apply first where they overlap; higher applies later. Track roadbed applies at order 1, so a
  mask at 0 or below (most of the base map's are -1 or 0) still has the track's roadbed cut on top of it.
- **Biome:** optional ground cover stamped inside the footprint, such as gravel for a yard. Leave it empty to keep the
  ground as it is.
- **Mask Targets:** optional terrain channels painted inside the footprint. The base map uses **Cut Trees** (value 1) to
  clear trees from a site.

A mask levels the ground under its whole footprint, so check that it doesn't bury or undercut nearby track, roads or
buildings.

When you save, masks go in `Data\Map.json` under `objects`:

```json
{ "objects": {
  "yourname-log-pad": {
    "kind": "RectangleMask", "sizeX": 40.0, "sizeZ": 120.0,
    "order": 0, "radius": 2.0, "falloff": 15.0, "flattenHeightmap": true,
    "position": [17900.0, 577.5, -380.0], "rotation": [0.0, 30.0, 0.0], "scale": [1.0, 1.0, 1.0]
  }
}}
```

---

## 5. Add industries

An industry is made of four parts, created in this order:

- **Area:** a named place on the map, such as Bryson or Whittier. Industries belong to an area.
- **Industry:** the customer players see, such as a mill or a team track.
- **Track spans:** stretches of track where the industry's cars are spotted.
- **Components:** what the industry does on its spans, such as loading, unloading or repairing cars.

### 5.1 Lay the track first

If the industry needs new track, lay it before you add the industry, because spans are set by clicking track. Level the
ground for it with masks if needed ([section 4](#4-shape-the-ground-with-terrain-masks)).

### 5.2 Create or choose the area

- **New area:** open the **Create** menu and choose **Area**. It's placed on the ground where the camera points.
- **Existing area:** to add an industry to a base-game area such as Bryson, select that area in the outline instead.

In the area's inspector, set:

| Field | What it does |
|---|---|
| **Id** | The area's key. Keep it short, lowercase, no spaces. |
| **Name** | The name players see. |
| **Radius** | The area's size in metres. |
| **Tag Color** | The colour of the area's car tags. |
| **Position** | The area's centre. Use **Move** to drag it. |

### 5.3 Create the industry

In the area's inspector, click **Create Industry**. In the industry's inspector, set:

| Field | What it does |
|---|---|
| **Id** | The industry's key. Start it with your name (e.g. `yourname-sawmill`) so it can't clash with the game's or another mod's industries. |
| **Name** | The name players see. |
| **Position** | Where the industry's label sits on the map. |
| **Uses Contract** | Whether the industry works under a contract that players sign, as most base-game customers do. |

### 5.4 Add the track spans

In the industry's inspector, click **Create Span** once for each stretch of track. For each span:

1. Click **Set Lower**, then click the track where the span starts. Right-click cancels.
2. Click **Set Upper**, then click where it ends.
3. Check the **Length** shown in the inspector. A red line under an end means it doesn't point at any track.

You can also type an end as `segmentId|end|distance`, for example `S4xl|a|12.5`: the distance in metres from the
segment's `a` or `b` end. **Flip** swaps the two ends.

Give each span a clear **Id** (e.g. `spot1`, `coal`, `shed`). Components refer to spans by these ids, and
**Referenced By** shows which components use each one.

### 5.5 Add the components

In the industry's inspector, open the **Create Component** dropdown and pick a kind:

| Kind | Used for |
|---|---|
| **IndustryLoader** | Produces one load and loads it into cars. |
| **IndustryUnloader** | Unloads one load from cars into storage. |
| **FormulaicIndustryComponent** | Turns input loads into output loads at set rates per day. |
| **StorageLoader** / **StorageUnloader** | Load or unload several kinds of load from one store. |
| **TeleportLoadingIndustry** | Swaps empty cars on its input spans for loaded ones on its output spans. |
| **IndustrySeller** | Sells a load into cars. |
| **RepairTrack** | Repairs cars on its spans, using a repair parts load. |
| **TeamTrack** | A public team track. |
| **PassengerStop** | A passenger station. |
| **Interchange**, **InterchangedIndustryLoader**, **ProgressionIndustryComponent** | Interchanges, and industries tied to progression. |

**The easy way to learn a kind:** select a base-game industry that does what you want in the outline and look at its
components' fields. Do this especially for the less common kinds.

Then fill in the component's fields:

- **Name:** the name players see.
- **Track Spans:** pick from the industry's spans.
- **Load:** pick from the loads the game knows. A load from another mod needs that mod installed, and listed as a
  dependency of your release mod ([section 7](#7-make-the-release-mod)).
- **Rates and storage:** for example **Production Rate** and **Max Storage**, in the load's own units per day. Hover
  over a field for its tooltip. See [5.6](#56-rates-and-storage).
- **Car Type Filter:** which cars may be used, e.g. `GB*`. Leave it empty for the component's usual cars.
- **Shared Storage:** whether the component shares stored loads with the industry's other components.

Renaming an id warns you if something still uses the old one. Every change can be undone with **Undo**.

### 5.6 Rates and storage

**Units.** All amounts are in the load's own units: pounds for most freight (logs, coal, lumber, pulpwood), gallons
for liquids, or a count for loads that come in pieces. All rates are **per game day**; the game handles the time speed
for you.

**Car loads.** To think in cars, divide by what one car holds. The base log car holds 49,500 lb of logs, so:

- 15 car loads of logs = 15 × 49,500 = 742,500 lb
- 48 car loads = 48 × 49,500 = 2,376,000 lb

Look up the capacity of the cars you expect to be used in their definitions (`loadSlots`, `maximumCapacity`).

**Contracts.** With **Uses Contract** on, every rate below is multiplied by the contract tier the player signed:

| Tier | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Multiplier | 12.5% | 25% | 50% | 75% | 100% |

With no contract, the industry does nothing at all. With **Uses Contract** off, it always runs at 100%. So set the rates
for tier 5, and a tier 1 contract gets one eighth of them.

#### IndustryLoader

| Field | What it means |
|---|---|
| **Production Rate** | How much the industry makes per day. It goes into the industry's storage. Set it to 0 when a **FormulaicIndustryComponent** makes this load instead. |
| **Max Storage** | The most the industry holds. When storage is full, production stops until cars take some away. |
| **Car Load Rate** | How fast cars are filled, per day. Cars are only filled from what's in storage. |

The **Car Load Rate** is for the whole component, shared by all its cars and spans, not per car or per track.

The industry panel's "Production: *n* cars/day" is the smaller of Production Rate and Car Load Rate, divided by the
average capacity of the player's own cars that fit.

How long it takes to fill one car is its capacity divided by the Car Load Rate. With the base log spurs' 3,000,000 lb
per day, a 49,500 lb log car takes 49,500 / 3,000,000 = 0.0165 days, about 24 game minutes.

A good starting point is a Max Storage of about one day's production. Then a spur served once a day is full when the
train arrives, and nothing is lost to a full store.

With **Order Empties** on, the industry orders its own empty cars: see [5.7](#57-how-an-industry-orders-cars).

#### IndustryUnloader

| Field | What it means |
|---|---|
| **Car Unload Rate** | How fast cars are emptied, per day. Unloading stops while storage is full. |
| **Max Storage** | The most the industry holds. |
| **Storage Consumption Rate** | How much the industry uses up from storage per day. Set it to 0 when a **FormulaicIndustryComponent** uses this load instead. |

The panel's "Consumes: *n* cars/day" is the consumption rate (or the formula's input) divided by the average capacity
of the player's cars. With **Order Loads** on, the industry orders its own loaded cars: see
[5.7](#57-how-an-industry-orders-cars).

#### FormulaicIndustryComponent

A formula turns its **Input Terms** into its **Output Terms**, each in units per day. It takes its inputs from the
industry's storage and puts its outputs back there, so the industry also needs:

- an **IndustryUnloader** for each input, with **Storage Consumption Rate** 0
- an **IndustryLoader** for each output, with **Production Rate** 0

Leave **Shared Storage** on for these, so they use the same storage as the formula.

The formula runs at full speed while it has a full rate of every input and room for every output. It slows down when one
of them runs short, and it stops ("Production Stopped") when any input is empty or any output's storage is full. Its
storage limits are the **Max Storage** of the loader or unloader that has the same load.

#### Example: Whittier Saw Mill

| Component | Load | Settings |
|---|---|---|
| Unloader | Logs | Car Unload Rate 72,000,000; Max Storage 2,376,000; Storage Consumption Rate 0 |
| Formula | Logs → Lumber + Woodchips | Input 2,376,000 logs; outputs 1,864,800 lumber, 480,000 woodchips |
| Loader | Lumber | Car Load Rate 2,797,200; Max Storage 3,729,600; Production Rate 0 |
| Loader | Woodchips | Car Load Rate 720,000; Max Storage 960,000; Production Rate 0 |

At tier 5 the mill uses 2,376,000 lb of logs a day: 48 log cars. Its log storage holds exactly one day of that. A log car
is emptied in about one game minute (49,500 / 72,000,000 days). The lumber store holds two days of output and the
woodchip store two days. At a tier 1 contract, everything is one eighth: 6 log cars a day.

### 5.7 How an industry orders cars

Industries order cars when the interchanges are served: once a day at the interchange service hour, and again for an
extra service. Each loader with **Order Empties** on and each unloader with **Order Loads** on works out an amount of
load, not a number of cars. With **Uses Contract** on, it only orders while the contract is active.

**How far ahead it orders.** The industry keeps a running average of how many days its cars take from being waybilled
to being delivered. It starts at 1 day and moves 20% of the way towards each new delivery's time, never above 3 days.
The industry orders for that many days ahead, but at least until the next daily interchange service.

**A loader (Order Empties)** orders empty cars for:

> what's in storage + Production Rate × days ahead − the free space in empties already ordered or on their way

So with 1 day ahead, 15 car loads of production a day and nothing in storage or on the way, it orders space for 15 car
loads. If 5 empties are already on their way, it orders 10.

**An unloader (Order Loads)** orders loaded cars for:

> Storage Consumption Rate × days ahead − (what's in storage + loads already ordered or on their way)

If the unloader uses no storage itself (rate 0), it uses the formula's input rate for that load instead. Only loads the
game can import from off the map can be ordered this way. Logs, for example, can't, so a log unloader like the saw
mill's has **Order Loads** off and gets its logs from the player's own log trains.

Both rates are multiplied by the contract tier, as above.

**From an amount to cars.** The order goes to an interchange that can reach the industry by track. When it's filled,
the interchange picks a random car type that matches the component's **Car Type Filter**, and takes away that car's
full capacity from the order. It keeps adding cars until the order is used up, so a part car load left over still gets
a whole car.

**Interchange space.** An interchange only takes as many cars as fit: 70% of its track length in 50 ft (15.24 m) car
lengths, minus the cars already there. When it's full, the player gets a message that the interchange is at capacity.
What wasn't delivered isn't kept: the industries work out their orders again at the next service.

### 5.8 How deliveries are paid

Each waybill's payment is set when the waybill is written:

> payment = fee + 0.12 × gross tons × √miles, rounded to whole dollars

- **Gross tons:** the car's empty weight plus its load, divided by 2,000.
- **Miles:** the shortest route along the track from origin to destination, not the straight-line distance. With no
  route, the car can't be priced, and the game logs "No route from ... Freight from here cannot be priced until the
  track connects."
- **Fee:** $60 when the move starts or ends at an industry, $30 for a move from one interchange to another.

Distance pays with diminishing returns: 4 miles pays twice the line haul of 1 mile, not four times. Weight counts in
full, so heavy loads pay much more than empties. Example: a 40,000 lb log car with 49,500 lb of logs is 45 tons; over
10 miles it pays 60 + 0.12 × 45 × √10 = $77.

When the car is delivered, the payment is adjusted:

- **Timely bonus:** only if the industry it's delivered to has an active contract. A share of the payment by contract
  tier and by whole days since the waybill was written:

  | Tier | Same day | 1 day | 2 days | Later |
  |---|---|---|---|---|
  | 1 | 0% | 0% | 0% | 0% |
  | 2 | 4% | 2% | 1% | 0% |
  | 3 | 6% | 3% | 1.5% | 0% |
  | 4 | 8% | 4% | 2% | 0% |
  | 5 | 10% | 5% | 2.5% | 0% |

- **Damage fine:** none at 95% condition or better, rising evenly to 75% of the payment at 0% condition.
- **Grace days:** routes over 20 miles get 1 grace day, over 40 miles 2. They don't change the payment; they only count
  towards the industry's delivery record.

**Loads moved between industries on the map**, such as from a loader to another industry's unloader, are paid by the
unloader once a day, in proportion to what it unloaded. It uses the same formula on the route from the load's origin
industry, and also pays the empty car's trip back if the unloader is set to pay both legs.

So an industry's rates and storage decide how many cars move each day; what each car pays depends on its weight and
how far it travels.

### 5.9 Example: a log loading spur

The base game's log spurs (Walker Lower and Upper Logs, Connelly Creek L1 to L3) are each one **IndustryLoader** with
its own span or spans. Set up yours like this:

| Field | Setting |
|---|---|
| **Load** | Logs (`rr/logs`) |
| **Car Load Rate** | 3,000,000 (the base spurs' value) |
| **Production Rate** / **Max Storage** | about 742,500 for each track (the base spurs' values): 15 log cars a day, one day of storage |
| **Order Empties** | off |
| **Shared Storage** | off, so each spur keeps its own logs, like the base spurs |

On the industry, turn **Uses Contract** off, as the base log camps do.

### 5.10 Example: changing a base-game industry

Base-game industries are edited the same way as your own. This example uses Barker's Creek Pulpwood Spur (area
`barkers`, industry `bc-pulpwood`). In the base map it has one span, `barker's-creek-pulpwood-spur`, used by one
component, `loader`.

**To lengthen or move an existing span:**

1. Select the span in the outline, under the industry.
2. Click **Set Lower** or **Set Upper** and click the track where that end should be.
3. Save.

The saved changes hold only the span you changed:

```json
{ "ops": { "areas": { "barkers": { "industries": { "bc-pulpwood": {
  "trackSpans": {
    "barker's-creek-pulpwood-spur": ["Spgl|b|0.234", "Sl75|a|80.000"]
  }
}}}}}}
```

**To add a second track to it:**

1. Select the industry in the outline and click **Create Span**.
2. Give the span an **Id** (e.g. `yourname-track2`) and set both ends on the new track.
3. Select the `loader` component and add the new span to its **Track Spans**, keeping the old one.
4. Save.

The saved changes hold the new span and the component's whole span list. Ids such as `S1abc` stand for your track's own
segment ids:

```json
{ "ops": { "areas": { "barkers": { "industries": { "bc-pulpwood": {
  "trackSpans": {
    "yourname-track2": ["S1abc|a|1.000", "S2def|b|1.000"]
  },
  "components": {
    "loader": {
      "trackSpans": ["barker's-creek-pulpwood-spur", "yourname-track2"]
    }
  }
}}}}}}
```

Lists are replaced whole when merged, so the component's list must keep the base game's span too. The editor saves it
that way.

**To give the new track its own load instead**, such as logs beside the pulpwood: don't add the span to `loader`.
Create a new **IndustryLoader** component on the same industry, set up as in [5.9](#59-example-a-log-loading-spur), and
pick only the new span.

### 5.11 When a loader doesn't show under "When empty"

A car's **When Empty** waybill list offers every loader that can fill it. If your loader doesn't show there, check:

- **Component:** it's an **IndustryLoader** (not an unloader or a team track) with a **Load** set.
- **Order Empties:** it's off. With it on, the industry orders its own empties and isn't offered in the list.
- **Track Spans:** at least one span is picked.
- **Car Type Filter:** it matches the car. An empty filter matches all cars.
- **Progression:** if the industry is tied to a progression milestone, that milestone has to be built.

---

## 6. Save and check your changes

Click **Save** at the top of the Map Editor. Your map is layered over the base map, so the game saves only what differs
from it. The file is in your editing mod's `Data` folder, named by your Map definition's `mapData` (usually
`Data\Map.json`). It holds:

- your new track, masks, props, areas, industries, spans and components
- the fields you changed on the base map's own ones
- anything you deleted from the base map, marked as `null`

Open the file and check that it holds only the changes you meant to make. If you moved something by accident, undo that
in the editor and save again.

---

## 7. Make the release mod

The release mod is a **new folder**, with its own id (e.g. `yourname/my-mod`, without `-dev`). Don't move or rename the
editing mod: keep it for your next changes.

```
My Mod/
  Info.json
  map-patch.json        copy of the editing mod's Data\Map.json (track, masks, industries)
  Scripts/Data.rl
  Definitions.json      only if you added terrain tiles
  Terrain/              only if you added terrain tiles: copy of the editing mod's Terrain folder
```

These files don't need an ADK build. Every time you change something in the editor, copy the new `Data\Map.json` (and
`Terrain\`) into the release mod again and raise its version.

### `Info.json`

```json
{
  "identifier": "yourname/my-mod",
  "version": "0.1.0",
  "name": "My Mod",
  "author": "Your Name",
  "dependencies": { "required": ["rr/game", "rr/nc-bryson-terrain"] }
}
```

`rr/nc-bryson-terrain` is the game's own pack holding the base map. Requiring it makes your mod load after it, so your
script can find the base map. Without it, mods load in alphabetical order of their ids, and an id sorting before `rr/`
(such as `abc/...`) fails with "Definition rr/bushnell-whittier not found". Also add any mod whose loads, cars or track
your changes use.

### `Definitions.json` (terrain tiles only)

```json
{ "objects": [{
  "identifier": "yourname/my-terrain",
  "metadata": { "name": "My Terrain", "description": "...", "tags": [], "credits": "" },
  "definition": { "kind": "TerrainTileSet", "path": "Terrain",
                  "attribution": "Your elevation source" }
}]}
```

### `Scripts/Data.rl`

Keep the parts you need: `modifyDefinitions` for terrain tiles, `modifyMap` for everything in `map-patch.json`.

```
import json

// Terrain tiles: adds your tile set to the base map.
func modifyDefinitions(context) {
    context.modify("rr/bushnell-whittier", (obj) => {
        obj["definition"]["terrainTileSets"] = ["rr/nc-bryson", "yourname/my-terrain"]
    })
}

// Track, masks and industries: only on the base map, the map that has this node.
func modifyMap(map) {
    var isBase = false
    map.modify((data) => {
        isBase = data["graph"]?["nodes"]?["Ntky0ixdk"] != null
    })
    if isBase {
        map.merge(json.load("map-patch.json"))
    }
}
```

**Terrain tiles:** the list is replaced whole, so `rr/nc-bryson` must stay in it. Two mods doing this will probably
conflict, and the docs don't define which one wins. Any mod that needs both must list both tile sets.

**The map check:** `modifyMap` runs for every map the game loads, so the check keeps your changes off other maps.
`Ntky0ixdk` is a node at the end of the Sylva Interchange track. Any node id that only the base map has will do.

**How `map.merge` combines them:**

- **Objects** (areas, industries, components, spans, map objects) merge key by key. New keys are added and changed
  fields replace the old ones.
- **Lists** (for example a component's `trackSpans`) are replaced whole, not added to.
- **`null`** deletes that key.

So, if two mods change the same base-game industry, the one that loads last wins on each field it sets.

---

## 8. Install and enable

With the game closed:

1. Copy the release mod's folder to `Documents\Railroader\Mods` (or the `Mods` folder in the folder you gave with
   `-homedir`, if you launch the game with it).
2. Add the mod to `Documents\Railroader\Preferences.json` with `"enabled": true, "scope": 0`.
3. When you release a new version, update the version there too, or startup hangs on a dialog.

---

## 9. Test

Load the base map (not the editor) and check that:

- **Terrain:** your tiles render, with no gaps or steps at the seams.
- **Ground:** masked areas are level, and nearby track, roads and buildings aren't buried or left in the air.
- **Track:** your track is there and connects.
- **Industries:** they and their names show on the map, their spans sit on the right track, and cars are loaded,
  unloaded or repaired as you set them up.
- **Log:** `Player.log` has no errors about your terrain, industries, spans or loads.

---

## Versions

Written for Railroader 0.6 (the 2026 Experimental update) and its built-in mod loader. Nothing here applies to Legacy
(2025.1 and earlier) or its loaders.

## Credits

By Tetz95, from building map mods for Railroader 0.6 (Tetz's Beta Yard and others) and reading the game's code along
the way. Thanks to the modders whose questions and feedback shaped this guide.

## License

This guide is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/);
see [LICENSE](LICENSE). You may share and adapt it, including in your own mods' docs, as long as you credit Tetz95
and link back here.

---

Corrections and suggestions are welcome as [issues or pull requests](https://github.com/Tetz95/Railroader-Map-Modding-Guides).
