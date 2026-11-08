# Adding terrain tiles to the Railroader 0.6 base map

Building on the base map is "not officially supported yet", and the tile format may still change during Experimental.
Replace `yourname/...` with your own ids.

> **Tested so far:** sections 2, 3 and 6 in the game on a test map, including both ways of adding the map's layers in
> step 2.2. Sections 4, 5, 7 and 8 were tested with other tiles, not with tiles fetched here.

## 1. Choose your tiles

- **Tile size and position:** each tile is 500 x 500 m. Tile `(x, y)` starts at world `(x * 500, y * 500)`, with X
  running east and Z running north.
- **Tiles already in use:** check
  `Railroader_Data/StreamingAssets/AssetPacksCore/nc-bryson-terrain/Terrain/Terrain.json`. Pick tiles that aren't listed
  and that touch the existing edge of the map.
- **Finding the real-world location:** the same file gives the map's origin, about 35.3826, -83.4954. These formulas
  are close enough for choosing an area, but they drift slightly:
  - latitude = `35.3826 + Z / 111320`
  - longitude = `-83.4954 + X / 90760`

## 2. Fetch the terrain in the Map Editor

Your tile set has to exist before the map opens, or the map fails to load with "Error loading server game". So **don't
click Edit Map until step 2.6**: steps 2.1 to 2.5 only set up files, and once the Map definition exists (step 2.1), the
order of the other file edits doesn't matter. **Don't use Create Terrain:** it sets the origin to the coordinates you
type, which can't be layered with the base map's terrain.

### 2.1 Create the Map definition

In the Definition Editor, select your editing mod in the list. If you don't have one yet, click **+ Create Mod** at the
bottom of the list (see [Your editing mod](editing-mod.md#1-create-the-mod)); don't copy the game's own map pack into
ModDev. Click **+ New**, choose **Map**, enter
an id and click **Create & Edit**. The game saves it to your mod's `Definitions.json` straight away and shows "Map
Definition Saved".

### 2.2 Add the map's layers

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

**Or in the file:** close the game, then in `ModDev`, in your mod's `Definitions.json`, find your new Map definition and
add these fields to its `definition`:

```json
"mapLayers": ["rr/bushnell-whittier"],
"terrainTileSets": ["yourname/my-terrain", "rr/nc-bryson"]
```

### 2.3 Close the game

Close the game if it's still open, and go to your mod's folder in `ModDev`.

### 2.4 Add the tile set

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

### 2.5 Create `Terrain.json`

Create `Terrain/Terrain.json` with exactly this. The origin must match the base map's exactly, so copy the digits:

```json
{"origin":{"latitude":35.38261413574219,"longitude":-83.49540710449219},"tileDimension":500,"heightmapResolution":513,"tiles":[]}
```

The empty `tiles` list is correct: the map has no tiles of its own yet. You don't need to create a tile first.

### 2.6 Open the map

Start the game, open the Definition Editor, select your mod in the list and click **Edit Map**.

### 2.7 Open the fetch window

Select **Terrain**, set **Mode** to **Tiles** and click **Fetch Terrain...**. The Tiles mode shows "Coverage: 0
tiles" at first.

### 2.8 Move the camera

Fly to the area. The fetch is centred on the camera itself, not on the point it's looking at, so check **Center Tile**
in the window before you start.

### 2.9 Fetch

Set **Radius** and click **Fetch**. Radius is in tiles: 0 is one tile, 1 is 3 x 3, 2 is 5 x 5.

**Keep the fetch area off the base map.** Fetched tiles replace base tiles at the same position. If you fetch one by
accident, close the game, delete its `tile_XXX_YYY.riff` and remove its entry from `Terrain.json`.

Tiles are written to `Terrain/` as soon as they're fetched. Reopen the map with **Edit Map** to check them.

## 3. Check `Terrain.json`

Open `Terrain/Terrain.json` and check that:

- `origin` still has the digits from step 2.5
- `tiles` has the positions you expected, with none from the base map's list
- `tileDimension` is 500 and `heightmapResolution` is 513

## 4. Set up the release pack

Create this layout, copying `Terrain/` from your ModDev mod:

```
My Terrain/
  Info.json
  Definitions.json
  Terrain/
  Scripts/Data.rl
```

Terrain files and JSON don't need an ADK build.

**`Info.json`**

```json
{
  "identifier": "yourname/my-terrain-pack",
  "version": "0.1.0",
  "name": "My Terrain",
  "author": "Your Name",
  "dependencies": { "required": ["rr/game", "rr/nc-bryson-terrain"] }
}
```

`rr/nc-bryson-terrain` is the game's own pack holding the base map. Requiring it makes your pack load after it, so your
`Data.rl` (section 5) can find `rr/bushnell-whittier`. Without it, packs load in alphabetical order of their ids, and an id
sorting before `rr/` (such as `abc/...`) fails with "Definition rr/bushnell-whittier not found".

**`Definitions.json`**

```json
{ "objects": [{
  "identifier": "yourname/my-terrain",
  "metadata": { "name": "My Terrain", "description": "...", "tags": [], "credits": "" },
  "definition": { "kind": "TerrainTileSet", "path": "Terrain",
                  "attribution": "Your elevation source" }
}]}
```

## 5. Attach the tiles to the base map

The separate map from section 2 is only for editing. To add the tiles to the real base map, use **`Scripts/Data.rl`**:

```
func modifyDefinitions(context) {
    context.modify("rr/bushnell-whittier", (obj) => {
        obj["definition"]["terrainTileSets"] = ["rr/nc-bryson", "yourname/my-terrain"]
    })
}
```

This replaces the whole list, so `rr/nc-bryson` must stay in it. Two mods doing this will probably conflict, and the
docs don't define which one wins here. Any mod that needs both must list both tile sets.

## 6. Check the seams

Neighbouring tiles need matching heights along their shared edges, or you'll see steps in the ground.

In our test, fetched tiles met the base map with no step at the 14 points measured along one 1.5 km edge. That's only
one edge, so check yours.

- **Inspect:** look along every edge between your tiles and the base map.
- **Fix:** use the editor's flatten and smooth brushes on your side of the edge, then save and copy `Terrain/` into the
  pack again.
- **Limitation:** brushes only get the edges close, not identical, and the base tiles can't be edited.

## 7. Install and enable (game closed)

1. Copy the pack folder to `Documents\Railroader\Mods` (or the `Mods` folder in the folder you gave with
   `-homedir`, if you launch the game with it).
2. Add the mod to `Documents\Railroader\Preferences.json` with `"enabled": true, "scope": 0`.
3. When you release a new version, update the version there too, or startup hangs on a dialog.

## 8. Test

Start the base map and travel to the new area. Check that the terrain renders, there are no gaps or steps at the
seams, and `Player.log` has no terrain errors.

Terrain doesn't add any track, so you'll need to add that separately if you want to run trains there.
