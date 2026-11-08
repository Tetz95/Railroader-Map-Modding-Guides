# Your editing mod and your release mod

Building on the base map uses **two mods**. Keep them apart:

| | Editing mod | Release mod |
|---|---|---|
| **Where** | `Documents\Railroader\ModDev\` | `Documents\Railroader\Mods\` |
| **What it's for** | Working in the Map Editor. Only for you. | What you and other players install and play with. |
| **Made by** | The game's **+ Create Mod** button | You, by hand, from the editing mod's files |
| **Holds** | Your Map definition and the changes the editor saves | Only your changes, plus a small script that adds them to the base map |

If you launch the game with `-homedir`, both folders are inside the folder you gave it instead of
`Documents\Railroader`.

## Don't copy the game's map

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

## 1. Create the mod

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

## 2. Add a Map layered over the base map

Follow steps 2.1 to 2.6 of [Adding terrain tiles](terrain-tiles.md#2-fetch-the-terrain-in-the-map-editor), with your
new mod selected:

- **2.1:** create your Map definition (**+ New**, **Map**).
- **2.2:** give it the base map as its layer (`rr/bushnell-whittier`) and two tile sets: your own first, then
  `rr/nc-bryson`.
- **2.4 and 2.5:** add your own tile set and an empty `Terrain/Terrain.json`. You need these even if you don't add
  terrain, or the map won't open.
- **2.6:** open it with **Edit Map**.

## 3. What's in the folder

After you've clicked **Save** in the Map Editor, your editing mod looks like this:

```
Documents\Railroader\ModDev\my-mod-dev\
  Info.json
  Definitions.json      your Map and tile set definitions
  Terrain\Terrain.json  your terrain tiles (none, if you didn't fetch any)
  Data\Map.json         your changes to the base map, written by Save
```

`Data\Map.json` is only what you changed: new track, areas and industries, the fields you changed on the base map's,
and anything you deleted (as `null`). That's the file your release mod uses.

## 4. Make the release mod

The release mod is a **new folder** in `Documents\Railroader\Mods`, with its own id (e.g. `yourname/my-mod`, without
`-dev`). Don't move or rename the editing mod: keep it for your next changes.

- Industries and track: [Adding industries, section 7](industries.md#7-set-up-the-release-pack).
- Terrain: [Adding terrain tiles, section 4](terrain-tiles.md#4-set-up-the-release-pack).

Every time you change something in the editor, copy the new `Data\Map.json` (or `Terrain\`) into the release mod again and raise its version.
