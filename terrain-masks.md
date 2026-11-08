# Shaping the base map's ground with terrain masks

The base map's terrain tiles belong to the game, so the Map Editor's terrain brushes can't change them. If you try, it
says "This map's terrain comes from another pack's tile set, which this session cannot write to." Brushes only work on
tiles your own mod fetched ([Adding terrain tiles](terrain-tiles.md)).

To level ground on the base map, use **terrain masks**. A mask is a map object, like a prop: it's saved in your map
file with your track and industries and goes in the same release patch. The base map shapes its own yards and
industry sites the same way, with 144 masks.

## Do you need one?

Track already shapes its own roadbed: the ground is cut or filled to the track along a narrow strip, blending into the
terrain over 20 m on each side. A single track on a gentle slope usually needs nothing more.

Use a mask where the roadbed isn't enough:

- a yard or several tracks side by side
- a level pad for an industry, a loading area or a building
- a road or parking area
- a wider or smoother cut or fill than the roadbed makes

## 1. Add a mask

In the Map Editor, open **Create** below the outline, then **Terrain Masks**, and choose a shape:

| Shape | Footprint | Use it for |
|---|---|---|
| **Circle** | **Radius** around its position | small pads, around a building |
| **Rectangle** | **Size X** by **Size Z**, plus **Radius** around it | yards, industry pads, parking areas |
| **Curve** | **Radius** each side of a curve from end A to end B | along track or a road, cuts and fills, a graded slope |

The mask is placed on the ground where the camera is looking. Move and turn it like any other object; a curve's two ends
can also be moved on their own.

## 2. Set it up

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

## 3. Save and release

Click **Save**. Masks are saved in `Data\Map.json` with your other changes, under `objects`:

```json
{ "objects": {
  "yourname-log-pad": {
    "kind": "RectangleMask", "sizeX": 40.0, "sizeZ": 120.0,
    "order": 0, "radius": 2.0, "falloff": 15.0, "flattenHeightmap": true,
    "position": [17900.0, 577.5, -380.0], "rotation": [0.0, 30.0, 0.0], "scale": [1.0, 1.0, 1.0]
  }
}}
```

The release mod needs nothing extra. Copy the new `Data\Map.json` into it again as in
[Adding industries, section 7](industries.md#7-set-up-the-release-pack), and raise its version.

---

Corrections and suggestions are welcome as [issues or pull requests](https://github.com/Tetz95/Railroader-Map-Modding-Guides).
