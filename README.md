# Railroader 0.6 Map Modding Guide

How to build on the Railroader 0.6 base map with the game's own Map Editor and mod files: new terrain, levelled
ground, track and industries, released as a mod that adds them to the base map.

The editor and file formats may still change during Experimental, and this guide will change with them. It's written
for **0.6.3**, which added map patches (see [Versions](#versions)). The game's own documentation covers the same
ground in less detail: [Map Modding](https://docs.railroadergame.com/modding/map-modding/). Replace `yourname/...`
with your own ids throughout.

## Contents

1. [One mod: a map patch](#1-one-mod-a-map-patch)
2. [Set up your mod](#2-set-up-your-mod)
3. [Add terrain tiles](#3-add-terrain-tiles) (optional)
4. [Shape the ground with terrain masks](#4-shape-the-ground-with-terrain-masks)
5. [Add industries](#5-add-industries)
6. [Save and check your changes](#6-save-and-check-your-changes)
7. [Make the release mod](#7-make-the-release-mod)
8. [Install and enable](#8-install-and-enable)
9. [Test](#9-test)
10. [Build on another map mod](#10-build-on-another-map-mod)
11. [The script way (Data.rl)](#11-the-script-way-datarl)

---

## 1. One mod: a map patch

A mod changes the base map with a **map patch**: a Map definition whose `role` is `patch`. The patch holds only your
changes (new track and industries, changed fields, removals), and the game applies it on top of the base map while
your mod is enabled. Players still pick the base map in New Game, and railroads started before they installed your mod
get your changes too.

You work on the patch in the Map Editor and release a copy of it:

| | Editing copy | Release copy |
|---|---|---|
| **Where** | `Documents\Railroader\ModDev\` | `Documents\Railroader\Mods\` |
| **What it's for** | Working in the Map Editor. Only for you. | What you and other players install and play with. |
| **Made by** | The game's **+ Create Mod** button | You, by copying the editing copy's files |
| **Holds** | Your patch and the changes the editor saves | The same files, under the release mod's own id |

If you launch the game with `-homedir`, both folders are inside the folder you gave it instead of
`Documents\Railroader`.

Before 0.6.3 there were no patches, and a release mod added its changes with a small script. That still works, and is
still needed in a few cases: see [section 11](#11-the-script-way-datarl).

### Don't copy the game's map

The base map is the game's own pack, `rr/nc-bryson-terrain`, in the game's install folder
(`Railroader_Data\StreamingAssets\AssetPacksCore\nc-bryson-terrain`). Don't copy it into ModDev or edit it there:

- The copy has the same id as the game's pack, which confuses the game.
- Saving writes the **whole** base map with your changes mixed in, not just your changes. That file can't be released
  as it is: it would put back your copy of everything on the map, undoing later game updates and other mods' changes.
- Changes made in the install folder are lost when Steam updates the game.

Your patch is applied **over** the base map instead. The base map shows in the editor, but Save only writes what you
changed.

If you've already edited a copy, its `Data\Map.json` holds your changes inside the whole map. Set up your own mod as
below and make the changes again there, then remove the copy from ModDev.

---

## 2. Set up your mod

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

With your mod selected, click **+ New**, choose **Map**, enter an id (e.g. `yourname/my-mod-dev-map`) and click
**Create & Edit**. The game saves it to your mod's `Definitions.json` straight away and shows "Map Definition Saved".

**Don't use Create Terrain:** it sets the origin to the coordinates you type, which can't be layered with the base
map's terrain.

### 2.3 Make it a patch of the base map

Close the game, then in your mod's `Definitions.json`, find your new Map definition and add these fields to its
`definition`:

```json
"role": "patch",
"mapLayers": ["rr/bushnell-whittier"]
```

- **`role`:** `patch` makes it a patch. A patch doesn't show in New Game.
- **`mapLayers`:** the map it patches, `rr/bushnell-whittier` for the base map. It must be the **last** entry; patches
  of other mods you build on go before it ([section 10](#10-build-on-another-map-mod)).
- **No tile set needed:** a patch uses the base map's terrain. You only add one for new terrain
  ([section 3](#3-add-terrain-tiles)).

The game's own docs set the same fields in the Definition Editor instead, in the form that **Edit** opens (the small
button beside **Edit Map**). This guide was tested with the file edit above.

Your Map definition then looks like this:

```json
{
  "identifier": "yourname/my-mod-dev-map",
  "metadata": { "name": "My Mod (dev)", "description": "", "tags": [], "credits": "" },
  "definition": { "kind": "Map", "role": "patch", "mapData": "./Map.json", "mapLayers": ["rr/bushnell-whittier"] }
}
```

### 2.4 Open the map

Start the game, open the Definition Editor, select your mod in the list and click **Edit Map**. The base map's terrain,
track, areas and industries all show. You can't change the base map itself, but your patch saves your changes and
additions on top of it.

The terrain is read-only in a patch: the editor says "Patch terrain editing is not yet supported." Shape the ground
with masks ([section 4](#4-shape-the-ground-with-terrain-masks)), and make new terrain tiles as in
[section 3](#3-add-terrain-tiles).

Your mod now looks like this:

```
Documents\Railroader\ModDev\my-mod-dev\
  Info.json
  Definitions.json      your Map definition (the patch)
  Data\Map.json         your changes to the base map, written by Save (section 6)
```

---

## 3. Add terrain tiles

Only needed if you want to build beyond the edge of the base map. To change the ground **on** the base map, use
masks ([section 4](#4-shape-the-ground-with-terrain-masks)).

A patch can't edit terrain, so the tiles are made in a second ModDev mod with an ordinary map (not a patch), and then
added to your patch.

> **Tested so far:** fetching, checking `Terrain.json` and the seams in the game on a test map, and a patch showing
> its tile set in front of the base map's terrain, in the editor and in play.

### 3.1 Make a terrain mod

Your terrain map has to have a tile set before it opens, or it fails to load with "Error loading server game". So
**don't click Edit Map until step 3.1.5**: the steps before only set up files.

1. **Create the mod:** as in [2.1](#21-create-the-mod), with the id `yourname/my-terrain-dev`, and a Map definition as
   in [2.2](#22-create-the-map-definition). Don't make this one a patch.
2. **Add the map's layers:** close the game, then add these fields to the Map definition's `definition`. Keep your tile
   set first, because terrain is read-only if another mod's set comes first:

   ```json
   "mapLayers": ["rr/bushnell-whittier"],
   "terrainTileSets": ["yourname/my-terrain", "rr/nc-bryson"]
   ```

3. **Add the tile set:** the **+ New** list has no tile set type, so add it to the `objects` list in
   `Definitions.json`:

   ```json
   {
     "identifier": "yourname/my-terrain",
     "metadata": { "name": "My Terrain", "description": "", "tags": [], "credits": "" },
     "definition": { "kind": "TerrainTileSet", "path": "Terrain",
                     "attribution": "Elevation: Mapterhorn (mapterhorn.com/attribution)" }
   }
   ```

4. **Create `Terrain\Terrain.json`** with exactly this. The origin must match the base map's exactly, so copy the
   digits:

   ```json
   {"origin":{"latitude":35.38261413574219,"longitude":-83.49540710449219},"tileDimension":500,"heightmapResolution":513,"tiles":[]}
   ```

   The empty `tiles` list is correct: the map has no tiles of its own yet.

5. **Open the map:** start the game, select the terrain mod in the Definition Editor and click **Edit Map**.

### 3.2 Choose your tiles

- **Tile size and position:** each tile is 500 x 500 m. Tile `(x, y)` starts at world `(x * 500, y * 500)`, with X
  running east and Z running north.
- **Tiles already in use:** check
  `Railroader_Data\StreamingAssets\AssetPacksCore\nc-bryson-terrain\Terrain\Terrain.json`. Pick tiles that aren't listed
  and that touch the existing edge of the map.
- **Finding the real-world location:** the same file gives the map's origin, about 35.3826, -83.4954. These formulas
  are close enough for choosing an area, but they drift slightly:
  - latitude = `35.3826 + Z / 111320`
  - longitude = `-83.4954 + X / 90760`

### 3.3 Fetch the terrain

1. In the Map Editor, select **Terrain**, set **Mode** to **Tiles** and click **Fetch Terrain...**. The Tiles mode
   shows "Coverage: 0 tiles" at first.
2. Fly to the area. The fetch is centred on the camera itself, not on the point it's looking at, so check **Center
   Tile** in the window before you start.
3. Set **Radius** and click **Fetch**. Radius is in tiles: 0 is one tile, 1 is 3 x 3, 2 is 5 x 5.

**Keep the fetch area off the base map.** Fetched tiles replace base tiles at the same position. If you fetch one by
accident, close the game, delete its `tile_XXX_YYY.riff` and remove its entry from `Terrain.json`.

Tiles are written to `Terrain\` as soon as they're fetched. Reopen the map with **Edit Map** to check them.

### 3.4 Check `Terrain.json`

Open `Terrain\Terrain.json` and check that:

- `origin` still has the digits from step 3.1
- `tiles` has the positions you expected, with none from the base map's list
- `tileDimension` is 500 and `heightmapResolution` is 513

### 3.5 Check the seams

Neighbouring tiles need matching heights along their shared edges, or you'll see steps in the ground.

In our test, fetched tiles met the base map with no step at the 14 points measured along one 1.5 km edge. That's only
one edge, so check yours.

- **Inspect:** look along every edge between your tiles and the base map.
- **Fix:** use the editor's flatten and smooth brushes on your side of the edge, then save.
- **Limitation:** brushes only get the edges close, not identical, and the base tiles can't be edited. To shape the
  ground on the base map's side, use masks ([section 4](#4-shape-the-ground-with-terrain-masks)).

### 3.6 Add the tiles to your patch

With the game closed:

1. Copy the terrain mod's `Terrain` folder into your patch mod (`ModDev\my-mod-dev\Terrain`).
2. Copy the tile set definition (from step 3.1.3) into your patch mod's `Definitions.json`.
3. Add your tile set to the patch's Map definition:

   ```json
   "terrainTileSets": ["yourname/my-terrain"]
   ```

   A patch lists only its own tile sets, not `rr/nc-bryson`. They go in front of the base map's terrain: where both
   have a tile, yours is used. They must share the base map's origin, tile size and resolution, which tiles made as
   above do.

Reopen your patch with **Edit Map**: the new terrain shows (read-only), and you can lay track on it. When you change
the terrain later, do it in the terrain mod and copy the `Terrain` folder again.

---

## 4. Shape the ground with terrain masks

The terrain brushes don't work in a patch ("Patch terrain editing is not yet supported"), and the base map's tiles
belong to the game anyway. Brushes only work in a terrain mod, on the tiles it fetched
([section 3](#3-add-terrain-tiles)).

To level ground on the base map, use **terrain masks**. A mask is a map object, like a prop: it's saved in your patch
with your track and industries and goes in the same release file. The base map shapes its own yards and industry
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
- **Tilt (rectangle and circle):** since 0.6.3, tilting the mask (an X or Z value in its **Rotation**, not just Y)
  levels the ground to a sloped plane through its position instead of a flat one, up to 45°. A 10° tilt falls about
  1.76 m every 10 m. Check which way it falls in the editor: in our test, `"rotation": [10, 0, 0]` fell towards +Z.
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
| **Id** | The industry's key. Start it with your name (e.g. `yourname-sawmill`) so it can't clash with the game's or another mod's industries. The editor refuses an id it can't use, such as one already used by another industry or one containing `/`. |
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
| **PassengerStop** | A passenger station. Since 0.6.3, stations and lines are made under **Ops: Passenger** (see the game's [Ops: Passenger](https://docs.railroadergame.com/modding/map-modding/ops-passenger/) page); this guide doesn't cover them yet. |
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

**Interchange space.** An interchange only takes as many cars as fit: a share of its track length in 50 ft (15.24 m)
car lengths, minus the cars already there. The share is the game's **Interchange Fill To** setting (30% to 100%,
default 70%; before 0.6.3 it was always 70%). When it's full, the player gets a message that the interchange is at capacity.
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

Click **Save** at the top of the Map Editor. **Save** is always available and turns orange when there are unsaved
changes. Your map is a patch, so the game saves only what differs from the base map. The file is in your mod's `Data`
folder, named by your Map definition's `mapData` (usually `Data\Map.json`). It holds:

- your new track, masks, props, areas, industries, spans and components
- the fields you changed on the base map's own ones
- anything you deleted from the base map, marked as `null`

Open the file and check that it holds only the changes you meant to make. If you moved something by accident, undo that
in the editor and save again.

---

## 7. Make the release mod

The release mod is a **copy** of your editing mod in `Mods`, with its own ids. Don't move or rename the editing mod:
keep it for your next changes.

```
Documents\Railroader\Mods\My Mod\
  Info.json
  Definitions.json      your patch (and your tile set, if you added terrain)
  Data\Map.json         copy of the editing mod's Data\Map.json (track, masks, industries)
  Terrain\              only if you added terrain tiles
```

1. Copy the editing mod's folder into `Mods` under a new name, e.g. `My Mod`.
2. In its `Info.json`, give it its own id (e.g. `yourname/my-mod`, without `-dev`), name and version.
3. In its `Definitions.json`, give the Map definition its own id too (e.g. `yourname/my-mod-map`), so it can't clash
   with the editing copy's.

There's no script and no ADK build. Every time you change something in the editor, copy the new `Data\Map.json` (and
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

`rr/nc-bryson-terrain` is the game's own pack holding the base map. Requiring it makes your mod load after it. Also add
any mod whose loads, cars or track your changes use.

### What the editor shows

The editor shows the base map with only the patches your patch lists in `mapLayers`. Other map mods you have enabled
in `Mods`, your own release copy included, don't show there: they're applied only in play. So you can keep your release
mod enabled while you edit. To work next to another mod's changes, build on it as in
[section 10](#10-build-on-another-map-mod).

---

## 8. Install and enable

With the game closed:

1. Copy the release mod's folder to `Documents\Railroader\Mods` (or the `Mods` folder in the folder you gave with
   `-homedir`, if you launch the game with it).
2. Add the mod to `Documents\Railroader\Preferences.json` with `"enabled": true, "scope": 0`. Scope 0 is the default,
   unscoped: a patch only works in an unscoped mod, so every player builds the same map.
3. When you release a new version, update the version there too, or startup hangs on a dialog.

Players' existing railroads get the patch too, from the next time they load them with your mod enabled.

---

## 9. Test

Load the base map (not the editor) and check that:

- **Terrain:** your tiles render, with no gaps or steps at the seams.
- **Ground:** masked areas are level (or sloped as you set them), and nearby track, roads and buildings aren't buried
  or left in the air.
- **Track:** your track is there and connects.
- **Industries:** they and their names show on the map, their spans sit on the right track, and cars are loaded,
  unloaded or repaired as you set them up.
- **Log:** `Player.log` has no errors about your patch, terrain, industries, spans or loads. A patch the game can't use
  is skipped with a message saying why, and the map loads without it.

---

## 10. Build on another map mod

To add to another mod's track or terrain (a branch off someone's extension, more industries in their yard), follow
sections 2 to 9 as for the base map, with the changes below. "The other mod" is the map mod you're building on.

### 10.1 Install the other mod

Install and enable the other mod in `Mods` ([section 8](#8-install-and-enable)) before you start, and add it to your
release mod's `Info.json`:

```json
"dependencies": { "required": ["rr/game", "rr/nc-bryson-terrain", "theirname/their-mod"] }
```

### 10.2 If the other mod is a patch

Look in the other mod's `Definitions.json` for a definition of kind `Map` with `"role": "patch"`. If there is one,
build on it directly: list its id in your patch's `mapLayers`, **before** the base map:

```json
"mapLayers": ["theirname/their-map", "rr/bushnell-whittier"]
```

- The editor opens the base map with their patch on it, and Save writes only your changes.
- In play, your patch is applied after theirs, so a field you change on one of their industries or tracks overrides
  theirs, and a `null` in your patch deletes something they added.
- Your mod must require theirs ([10.1](#101-install-the-other-mod)). Otherwise the game skips your patch with a message
  that your mod doesn't require theirs.
- To build on several patches, list them all, topmost first, with the base map last.

### 10.3 If the other mod uses a script

If the other mod adds its changes with a `Scripts\Data.rl` instead (as mods made before 0.6.3 do), the editor doesn't
show them: it doesn't run scripts ("Data.rl doesn't run in the editor" in `Player.log`). Give your editing mod a copy
of their changes as a patch to build on:

1. Open the other mod's `Scripts\Data.rl` and find the file its `modifyMap` merges: `map.merge(json.load("..."))`.
   If the script picks between files (for example one for use with a third mod), take the one that matches the mods
   you play with.
2. Copy that file into your editing mod's `Data` folder under a new name, e.g. `Data\other-mod.json`.
3. Add a patch for it to your editing mod's `Definitions.json`:

   ```json
   {
     "identifier": "yourname/other-mod-layer",
     "metadata": { "name": "Other Mod (layer, don't edit)", "description": "", "tags": [], "credits": "" },
     "definition": { "kind": "Map", "role": "patch", "mapData": "./other-mod.json", "mapLayers": ["rr/bushnell-whittier"] }
   }
   ```

4. Build your own patch on it: `"mapLayers": ["yourname/other-mod-layer", "rr/bushnell-whittier"]`.

Your editing mod now has two maps, each with its own **Edit Map** button. Always use the one on **your own map's** row:
Edit Map on the layer opens the copy of the other mod's file, and saving there changes the copy. When the other mod
releases a new version, copy its file again (step 2), then open your map and check that your track still joins theirs.

The copy is only for editing. Release your mod the script way ([section 11](#11-the-script-way-datarl)), so your
changes go in after theirs, checking for one of the nodes **their** mod adds. Your changes then go in only when their
track is there, and stay off the base map if someone removes their mod.

If the other mod's script builds its changes in code instead of loading a file, there's no file to copy. Ask its
author for one, or to release it as a patch.

---

## 11. The script way (Data.rl)

Before 0.6.3, a release mod added its changes with a script instead of a patch. Use this way when:

- you build on a mod that uses a script ([10.3](#103-if-the-other-mod-uses-a-script))
- your changes depend on which other mods are installed (a patch is always applied as it is)
- you support players still on 0.6.2 or earlier

Edit as in sections 2 to 6. The release mod then holds your saved changes under another name, and a script:

```
My Mod/
  Info.json
  map-patch.json        copy of the editing mod's Data\Map.json (track, masks, industries)
  Scripts/Data.rl
  Definitions.json      only if you added terrain tiles: just the TerrainTileSet, not the Map definition
  Terrain/              only if you added terrain tiles
```

`Info.json` is as in [section 7](#7-make-the-release-mod). Requiring `rr/nc-bryson-terrain` matters more here: without
it, mods load in alphabetical order of their ids, and an id sorting before `rr/` (such as `abc/...`) fails with
"Definition rr/bushnell-whittier not found".

### `Scripts/Data.rl`

Keep the parts you need: `modifyDefinitions` for terrain tiles, `modifyMap` for everything in `map-patch.json`.

```
import json

// Terrain tiles: adds your tile set to the base map's list, keeping the tile sets already in it.
func modifyDefinitions(context) {
    context.modify("rr/bushnell-whittier", (obj) => {
        var sets = []
        var old = obj["definition"]["terrainTileSets"]
        if old == null {
            sets.add("rr/nc-bryson")
        } else {
            for s in old {
                sets.add(s)
            }
        }
        sets.add("yourname/my-terrain")
        obj["definition"]["terrainTileSets"] = sets
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

**Terrain tiles:** the script adds your tile set to the list instead of setting the list. Setting it (for example
`= ["rr/nc-bryson", "yourname/my-terrain"]`) would drop the tile sets of map mods that loaded before yours, and their
terrain would disappear.

**The map check:** `modifyMap` runs for every map the game loads, so the check keeps your changes off other maps.
`Ntky0ixdk` is a node at the end of the Sylva Interchange track. Any node id that only the base map has will do, but
choose one the game is unlikely to remove: a game update that reshapes track can delete nodes, and then the check
quietly fails and your changes stop going in, with no error. 0.6.3 reshaped the Sylva switch and removed a node
there that at least one mod's check used. A node your
own track joins is a good choice, since your track needs it anyway. After every game update, load the map and check
that your changes are still there.

**How `map.merge` combines them:**

- **Objects** (areas, industries, components, spans, map objects) merge key by key. New keys are added and changed
  fields replace the old ones.
- **Lists** (for example a component's `trackSpans`) are replaced whole, not added to.
- **`null`** deletes that key.

So, if two mods change the same base-game industry, the one that loads last wins on each field it sets.

**Other scripts:** since 0.6.3, functions the game marks as internal start with `_`, and the game says mods shouldn't
rely on them. Scripts that looked definitions up with `getDefinition` stop with "Undefined method 'getDefinition'"
on 0.6.3: it's now `_getDefinition`, which works, but may change in a later update.

---

## Versions

- **0.6.3 (2026-11-17):** map patches (`"role": "patch"`) are the main way to release changes; the script way is
  section 11. Patch terrain is read-only, so terrain tiles are made in a separate terrain mod. Rectangle and circle
  masks can be tilted. The editor refuses industry ids it can't use. Tested in the game: editing a patch with and
  without terrain tiles (Save writes only the patch's changes), a released patch on a railroad started before it was
  installed, and a tilted mask.
- **0.6.0 to 0.6.2:** two mods, the release mod adding its changes with a script (section 11).

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
