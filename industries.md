# Adding industries to the Railroader 0.6 base map

You build industries in the Map Editor on a map of your own that's layered over the base map. Then a small script adds
them to the real base map. Building on the base map is "not officially supported yet", so details may change during
Experimental. Replace `yourname/...` with your own ids.

An industry is made of four parts, created in this order:

- **Area:** a named place on the map, such as Bryson or Whittier. Industries belong to an area.
- **Industry:** the customer players see, such as a mill or a team track.
- **Track spans:** stretches of track where the industry's cars are spotted.
- **Components:** what the industry does on its spans, such as loading, unloading or repairing cars.

## 1. Set up an editing map

Follow [Your editing mod and your release mod](editing-mod.md), sections 1 and 2: create your own editing mod with
**+ Create Mod**, then a Map layered over the base map. **Don't copy the game's own map pack into ModDev:** saving
would then write the whole base map, not just your changes. You don't need to fetch any terrain unless your industry
also needs new ground.

The base map's track, areas and industries all show in the editor. You can't change the base map itself, but your map
saves your changes and additions on top of it.

**Track first:** if the industry needs new track, lay it before you add the industry, because spans are set by clicking
track.

## 2. Create or choose the area

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

## 3. Create the industry

In the area's inspector, click **Create Industry**. In the industry's inspector, set:

| Field | What it does |
|---|---|
| **Id** | The industry's key. Start it with your name (e.g. `yourname-sawmill`) so it can't clash with the game's or another mod's industries. |
| **Name** | The name players see. |
| **Position** | Where the industry's label sits on the map. |
| **Uses Contract** | Whether the industry works under a contract that players sign, as most base-game customers do. |

## 4. Add the track spans

In the industry's inspector, click **Create Span** once for each stretch of track. For each span:

1. Click **Set Lower**, then click the track where the span starts. Right-click cancels.
2. Click **Set Upper**, then click where it ends.
3. Check the **Length** shown in the inspector. A red line under an end means it doesn't point at any track.

You can also type an end as `segmentId|end|distance`, for example `S4xl|a|12.5`: the distance in metres from the
segment's `a` or `b` end. **Flip** swaps the two ends.

Give each span a clear **Id** (e.g. `spot1`, `coal`, `shed`). Components refer to spans by these ids, and
**Referenced By** shows which components use each one.

The terrain brushes can't change the base map's ground. To level a site for new track or an industry, use
[terrain masks](terrain-masks.md).

## 5. Add the components

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
  dependency in section 7.
- **Rates and storage:** for example **Production Rate** and **Max Storage**, in the load's own units per day. Hover
  over a field for its tooltip.
- **Car Type Filter:** which cars may be used, e.g. `GB*`. Leave it empty for the component's usual cars.
- **Shared Storage:** whether the component shares stored loads with the industry's other components.

Renaming an id warns you if something still uses the old one. Every change can be undone with **Undo**.

### The maths: rates and storage

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

The industry panel's "Production: *n* cars/day" is the smaller of Production Rate and Car Load Rate, divided by the
average capacity of the player's own cars that fit.

How long it takes to fill one car is its capacity divided by the Car Load Rate. With the base log spurs' 3,000,000 lb
per day, a 49,500 lb log car takes 49,500 / 3,000,000 = 0.0165 days, about 24 game minutes.

A good starting point is a Max Storage of about one day's production. Then a spur served once a day is full when the
train arrives, and nothing is lost to a full store.

With **Order Empties** on, the industry orders its own empty cars: see *How an industry orders cars* below.

#### IndustryUnloader

| Field | What it means |
|---|---|
| **Car Unload Rate** | How fast cars are emptied, per day. Unloading stops while storage is full. |
| **Max Storage** | The most the industry holds. |
| **Storage Consumption Rate** | How much the industry uses up from storage per day. Set it to 0 when a **FormulaicIndustryComponent** uses this load instead. |

The panel's "Consumes: *n* cars/day" is the consumption rate (or the formula's input) divided by the average capacity
of the player's cars. With **Order Loads** on, the industry orders its own loaded cars: see *How an industry orders
cars* below.

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

#### How an industry orders cars

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
the interchange picks a random car type that matches the component's **Car Type Filter**, and takes away that car's full capacity from the order. It keeps adding cars until the order
is used up, so a part car load left over still gets a whole car.

**Interchange space.** An interchange only takes as many cars as fit: 70% of its track length in 50 ft (15.24 m) car
lengths, minus the cars already there. When it's full, the player gets a message that the interchange is at capacity. What
wasn't delivered isn't kept: the industries work out their orders again at the next service.

### Example: a log loading spur

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

### When a loader doesn't show under "When empty"

A car's **When Empty** waybill list offers every loader that can fill it. If your loader doesn't show there, check:

- **Component:** it's an **IndustryLoader** (not an unloader or a team track) with a **Load** set.
- **Order Empties:** it's off. With it on, the industry orders its own empties and isn't offered in the list.
- **Track Spans:** at least one span is picked.
- **Car Type Filter:** it matches the car. An empty filter matches all cars.
- **Uses Contract:** if it's on for the industry, the loader only shows while the player has an active contract there.
- **Track:** the industry's spans are connected by track to the rest of the railroad.
- **Progression:** if the industry is tied to a progression milestone, that milestone has to be built.

## 6. Save

Click **Save** at the top of the Map Editor. On a layered map, the game saves only what differs from the base map, as a
patch. This file is in your mod's folder in `Documents\Railroader\ModDev`, in its `Data` folder, named by your Map
definition's `mapData` (usually `Data\Map.json`). The patch holds:

- your new areas, industries, spans and components
- the fields you changed on the base map's own ones
- anything you deleted from the base map, marked as `null`

Open the file and check that it holds only the changes you meant to make. If you moved something by accident, undo that
in the editor and save again.

### Example: changing a base-game industry

Base-game industries are edited the same way as your own. This example uses Barker's Creek Pulpwood Spur (area
`barkers`, industry `bc-pulpwood`). In the base map it has one span, `barker's-creek-pulpwood-spur`, used by one
component, `loader`.

**To lengthen or move an existing span:**

1. Select the span in the outline, under the industry.
2. Click **Set Lower** or **Set Upper** and click the track where that end should be.
3. Save.

The patch holds only the span you changed:

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

The patch holds the new span and the component's whole span list. Ids such as `S1abc` stand for your track's own
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
Create a new **IndustryLoader** component on the same industry, set up as in *Example: a log loading spur* in section 5,
and pick only the new span.

## 7. Set up the release pack

```
My Industries/
  Info.json
  industries.json
  Scripts/Data.rl
```

`industries.json` is a copy of the saved map file from section 6.

**`Info.json`**

```json
{
  "identifier": "yourname/my-industries",
  "version": "0.1.0",
  "name": "My Industries",
  "author": "Your Name",
  "dependencies": { "required": ["rr/game", "rr/nc-bryson-terrain"] }
}
```

`rr/nc-bryson-terrain` is the game's own pack holding the base map. Add any mod whose loads or track your industries use.

**`Scripts/Data.rl`**

```
import json

// Only on the base map: it's the map that has this node.
func modifyMap(map) {
    var isBase = false
    map.modify((data) => {
        isBase = data["graph"]?["nodes"]?["Ntky0ixdk"] != null
    })
    if isBase {
        map.merge(json.load("industries.json"))
    }
}
```

`modifyMap` runs for every map the game loads, so the check keeps your industries off other maps. `Ntky0ixdk` is a node
at the end of the Sylva Interchange track. Any node id that only the base map has will do.

**How `map.merge` combines them:**

- **Objects** (areas, industries, components, spans) merge key by key. New keys are added and changed fields replace
  the old ones.
- **Lists** (for example a component's `trackSpans`) are replaced whole, not added to.
- **`null`** deletes that key.

So, if two mods change the same base-game industry, the one that loads last wins on each field it sets.

## 8. Install and enable (game closed)

1. Copy the pack folder to `Documents\Railroader\Mods` (or the `Mods` folder in the folder you gave with
   `-homedir`, if you launch the game with it).
2. Add the mod to `Documents\Railroader\Preferences.json` with `"enabled": true, "scope": 0`.
3. When you release a new version, update the version there too, or startup hangs on a dialog.

## 9. Test

Load the base map and check that:

- the industry and its name show on the map
- its spans sit on the right track
- cars are loaded, unloaded or repaired as you set them up
- `Player.log` has no errors about your industry, its spans or its loads

---

Corrections and suggestions are welcome as [issues or pull requests](https://github.com/Tetz95/Railroader-Map-Modding-Guides).
