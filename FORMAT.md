# Map database format — ババアセフトシニアカー Niigata（仮題）

This folder is a derivative database of OpenStreetMap (ODbL 1.0) and PLATEAU 新潟市 2023 (PDL1.0).
Licences and attribution: `LICENSE.txt`. This file describes every file so the data can be read without the game.
（このフォルダの地図データベースの形式の説明です。ライセンスと出典は `LICENSE.txt`。）

## Coordinates
- Local metres: `x` = east, `z` = south (north is −z), `y` = metres above sea level. Heading = `atan2(dx, dz)`.
- Cells are 512 m: cell `(tx, tz)` covers `x ∈ [tx·512, tx·512+512)`, `z ∈ [tz·512, tz·512+512)`.
  Regions are 8 × 8 cells: region `(rx, rz)` = `(floor(tx/8), floor(tz/8))`.
- All binaries are little-endian. `.gz` files are standard gzip.

## Files
| file | content |
|---|---|
| `index.json` | bounds, region list, terrain/nav headers, overview extent, racecourse geometry, railway line, points of interest, reserved sites, areas, workplaces, attribution |
| `r_<rx>_<rz>.bin.gz` | one region pack (up to 64 cells) |
| `terrain.bin.gz` | height field (16 m grid) |
| `nav.bin.gz` | drivable road graph |
| `overview.jpg` | whole-map raster (8 m per pixel) covering `index.overview.{x0,z0,x1,z1}` |
| `mapview.jpg` | dataRev 12: the big map / minimap base picture in a flat map style (8 m per pixel) covering `index.mapview.{x0,z0,x1,z1}` (play area box + 1.5 km). `overview.jpg` stays the far-terrain colour |
| `sites.json` | landmark kits: suburban chain stores at their real positions with fictional names (see below) |
| `LICENSE.txt` | licences, sources, method |

## `index.json`
- `version` 2, `dataRev` 11 (5 = yards, 6 = compressed map, 7 = low bridges levelled + level crossings, 8 = the Shinano and Agano rivers at
  their real width + a playable area with distant scenery outside it, 9 = a street band and yards just outside the playable area, the Agano
  mouth widened, water kept 10 m from river-bridge ends, store names as in rev 7, 10 = rev 9 merged with the other branch's dataRev 8:
  no nav edge (GPS, traffic) runs through a building — big/named buildings on a driving road are trimmed off it, dead ends stop in
  front of big buildings and the remaining roads inside kept buildings (stations, passages) are cut there; the 紫竹山 IC building stays
  removed; the binary layouts did not change; 11 = grade-separated junctions rebuilt with embankment ramps (ROADS flags bit 3), lots
  that were walled off above the road lowered/cut or given steps (LOTS flags bit 6), store POIs at their own brand's kit entrance),
  `cellSize` 512, `regionCells` 8, `bounds {x0, z0, x1, z1}` (m).
- `regions["rx,rz"] = {f, bytes, cells: ["tx,tz", …]}` — `bytes` = size of the `.gz` file. dataRev 12: cells farther than 2.6 km from `playArea` are not written (never streamed; the far terrain shows there), so some cells inside `bounds` are missing.
- `terrain {f, x0, z0, step, nx, nz}`, `nav {f, nodes, edges}`, `overview {f, x0, z0, x1, z1, w, h, mPerPx}`, `mapview {f, x0, z0, x1, z1, w, h, mPerPx, doc}` (dataRev 12).
- `racecourse {cx, cz, angle, half, r, standX, standZ, standFacing, …}` — stadium-shaped course centre line.
- `rail.shinetsu [[x, z, y], …]` (a railway line resampled every ≤ 20 m), `rail.stations {id: {s}}` (distance along it), `rail.length`.
- `pois [{id, kind, name, x, z, y, ry, …}]` — `ry` = heading the entrance faces. Kinds: station, store, konbini, shrine,
  police, hospital, funeral (funeral hall), place, home, racecourse, homecenter, shop, pachinko, area, bridge. Optional fields: `cat`/`sub` (store
  category), `b: [tx, tz, i]` (building the store sits in: index `i` in that cell's BUILDINGS section), `rect`
  (`[cx, cz, long, short, heading]` when no building), `ward` (shrines), `role` (police/hospital/funeral type), `s` (distance along
  `rail.shinetsu`). **Names of shops, shrines, police, hospitals, funeral halls and post offices are fictional** (not the OSM names);
  ids are sequential and are not OpenStreetMap ids.
- `reserved [{id, x0, z0, x1, z1}]` — sites cleared for the game.
- `areas [{name, sub, pri, poly: [[x, z], …]}]` — ward/district/landmark polygons (CCW from above); the highest `pri` wins.
- `work {kinds, cols, rows}` — workplace door points (`id, k, x, z, cap`).
- `crossings {cols, rows}` (dataRev 7) — level crossings (at-grade road × railway). Columns: `x, z` centre, `y` road surface,
  `h` road heading, `w` road width, `a` half length along the road from the centre to the far side of the outermost track (m),
  `g` 1 = gates and warning lights (roads cls ≤ 5), 0 = crossing sign only (tracks, cls 6), `rh` rail heading, `n` tracks crossed,
  `s` distance along `rail.shinetsu` (−1 on other lines), `p0a, p0l, p1a, p1l` warning-post position for the entrance from the −u side
  (`p0`) and the +u side (`p1`): distance from the centre along the road and from the road centre line sideways (m, magnitudes; the post
  stands on the entering driver's left, off every other carriageway, building, water and track; older data without these columns uses
  `a − 2.6 + 1.6 + 3.0` and `w / 2 + 0.75`). Where a road bridge leaves less than 4.5 m under its girder over a road (bridges of 300 m or more:
  2.3 m; over farm tracks: 2.3 m), where a road bridge sits almost on the rails (girder less than 1 m above the rail), or where a railway
  bridge is less than 2.5 m above the ground over a road, that part of the way is stored as an ordinary ground way (flags without bit 0);
  levelled roads share a node with every ground road they cross (nav junction). Road overpasses over railways are otherwise kept, and
  a railway viaduct whose normal slab would leave less than 4.5 m over a road is drawn with a thin girder (rail − 1.07 m) there.
- `playArea {ring, doc}` (dataRev 8) — the playable area: `ring` = `[[x, z], …]`, one closed outer ring (first point not repeated,
  counter-clockwise seen from above). Outside it the database holds distant scenery only: terrain, land use, rivers/lakes/carved canals,
  railway lines, trees, buildings (all within 1 km of the edge, farther out only non-house buildings 10 m or taller; no LOTS) and road classes 0–1 (ROADS and
  ROADLINES without sidewalks; not in the nav graph, no props, no level crossings). dataRev 9: within 300 m of the edge also road classes
  2–4 (ROADS, ROADLINES and sidewalks; still not in the nav graph, no level crossings), and houses within 150 m of the edge keep their LOTS.
  No POIs, workplaces or landmark kits lie outside (a store whose building straddles the edge is kept; its entrance is inside).
  The game keeps the player inside the ring (an invisible wall along the edge).
- `sidewalk {widths, bits}` — see ROADLINES flags. `stats` — build statistics (dataRev 8 adds `stats.playArea` and
  `stats.riversRealWidth`; dataRev 9 adds `stats.playArea.edgeBandKm`, `lotsKeptEdgeBand`, `shopsKeptAtEdge` and
  `stats.riversRealWidth.bridges.endPulledBack`; dataRev 10 adds `stats.roadBuildingFix.deadEndAtBig`, `trimmedBigOnDrive`,
  `droppedBigOnDrive` and `roadsCutInBuildings`; after the Rev 10 review, still dataRev 10: `keptWhole` (big buildings kept whole, the roads
  through them cut instead), `trimmedBigOffDeck` and `deckThroughBigLeft` (big buildings a bridge deck ran through: trimmed / left); dataRev 11 adds
  `stats.embankments` (`pieces`, `km`, `sideSlopeKm`, `sideWallKm`, `maxHeight`, `spans`), `stats.lowBridges.road_emb_*` and `stats.lots.high_*`, `cut`)). `attribution` — credit lines.

## Region pack `r_<rx>_<rz>.bin.gz`
```
"NGR2" | u16 version=2 | i16 rx | i16 rz | u16 cellCount
directory: cellCount × { i16 tx, i16 tz, u32 offset, u32 length }   (offset from file start)
cell blob: "NGC2" | i16 tx | i16 tz | u16 sectionCount | sections
section:   u8 type | u8 0 | u16 0 | u32 byteLength | payload
```
Local coordinates are **u16 centimetres from the cell's min corner** (multiples of 10) unless noted; heights **i16 decimetres**.
Sections are optional (LANDUSE is always present). Triangles are counter-clockwise seen from above (normal +y).

| type | name | payload |
|---|---|---|
| 1 | LANDUSE | `u16 w=128, u16 h=128`, then `w·h` u8 classes, row = z. Pixel (i, j) covers 4 × 4 m. Classes: 0 urban/bare, 1 paddy, 2 other farmland, 3 residential yard, 4 commercial paving, 5 industrial, 6 park/grass, 7 forest, 8 sand, 9 railway ballast, 10 parking, 11 water, 12 cemetery, 13 golf |
| 2 | WATER | records `u8 kind (0 river/sea/lake, 1 canal/ditch/pond), u8 flags, i16 levelDm, u32 vertCount`, then vertCount vertices. `flags` 0: vertex = `(u16 x, u16 z)`, the surface is flat at `levelDm / 10` m. `flags` bit 0 (dataRev 4): vertex = `(u16 x, u16 z, i16 y cm)`, a height per vertex (the level drops downstream and never rises above the banks); bit 1: a narrow ditch laid on the ground (not carved into the terrain); bits 4–7: the depth shown by the colour, in 0.1 m. Triangle lists |
| 3 | ROADS | records `u8 cls, u8 flags (bit0 bridge, bit1 tunnel, bit2 sidewalk strip, bit3 embankment (dataRev 11, always with bit0): a raised road on an earth embankment — the deck triangles and heights are stored like a bridge, but the space under it is solid ground: no girders, piers or railings, the side edges get slopes (1.5 : 1) or retaining walls and guard rails, and no ground road or track passes under it (where one crosses, that part is an ordinary bridge record without bit3)), i16 0, u32 vertCount, vertCount × (u16 x, u16 z, i16 yDm)` — triangle list. cls: 0 motorway/trunk, 1 primary, 2 secondary, 3 tertiary, 4 residential/unclassified, 5 service, 6 track, 7 pedestrian/footway, 8 sidewalk |
| 4 | ROADLINES | records `u32 osmWayId, u8 cls, u8 lanes, u8 oneway (0 no, 1 forward, 255 backward), u8 flags, u16 nameIdx (0xffff = none), u16 n, n × (u16 x, u16 z, i16 yDm)`. flags: bit0 bridge, bit1 tunnel, bit2 sidewalk on the left, bit3 on the right, bits 4–5 / 6–7 left / right width code into `index.sidewalk.widths` (left = left of the point order, seen from above) |
| 5 | BUILDINGS | `u32 count`, then per building `u8 kind, u8 roof, u16 heightDm, i16 baseDm, u16 nameIdx, u16 n, n × (i32 xCm, i32 zCm)` (outer ring, CCW, relative to the cell min corner; walls from `baseDm` to `baseDm + heightDm`). dataRev 5: `baseDm` = the levelled ground of the building (its pad, see LOTS): never below the drawn terrain under the footprint; the ground floor is about 0.4 m above it (foundation). Where the terrain around falls below the base, the runtime draws the foundation down to the terrain. kind: 0 house, 1 apartment, 2 commercial, 3 office, 4 industrial/warehouse, 5 farm shed, 6 shrine/temple, 7 school/public, 8 station, 10 other. roof: 0 flat, 1 gable, 2 hip |
| 6 | RAIL | records `u8 kind (0 track centre line, 1 platform outline), u8 flags (bit0 viaduct), u16 n, n × (u16 x, u16 z, i16 yDm)` |
| 7 | TREES | `u32 count`, `count × (u16 x, u16 z, u8 type, u8 scale·50)` — type 0 broadleaf, 1 pine, 2 persimmon, 3 cedar |
| 8 | STRINGS | `u16 count`, then `u16 byteLength + UTF-8` each. Only the names of six stations are kept; every other building and road name was removed (`nameIdx = 0xffff`) |
| 9 | PROPS | `u32 count`, `count × (u8 type, u8 0, u16 x, u16 z, u16 rotDeg)` — 2 vending machine, 3 traffic light, 4 bus stop |
| 10 | LOTS | dataRev 5: the yard (lot) of a detached house or farmhouse (tools/lots_build.py). `u32 count`, then per lot: `u16 bld` (index in this cell's BUILDINGS), `u8 flags` (bit0 farmhouse, bit1 raised: the pad is above the drawn terrain somewhere in the lot by > 0.12 m, bit2 parking, bit3 gate, bit4 gate has steps, bit5 the gate stays at road level: no room for steps, bit6 (dataRev 11) cut: the pad was lowered toward the road (cut and fill) and lies up to 0.8 m below the drawn terrain on the hill side of the house — the house and its sheds may sit that much below the drawn terrain there), `u8 n` (vertices), `i16 x0, i16 z0` (first vertex, 20 cm units from the cell min corner), `(n − 1) × (i8 dx, i8 dz)` (each next vertex relative to the previous one, 20 cm; CCW from above, may cross the cell border), `ceil(n / 2)` bytes of edge types, 4 bits each, low nibble first (edge i = vertex i → i+1): 0 open (fields, empty land), 1 road front, 2 boundary with a neighbour (this lot draws the fence), 3 boundary drawn by the neighbour's lot, 4 side of a sunken parking space or stairs (retaining wall only). If parking: `i16 x, i16 z` (20 cm, midpoint of its road-side edge, from the cell min corner), `u8 heading` (road → lot, 256 = full turn, heading = atan2(dx, dz)), `u8 widthDm, u8 depthDm`, `4 × i8` surface height at 0, 1/3, 2/3, 1 of the depth (4 cm units relative to the pad). If gate: `i16 x, i16 z` (20 cm), `u8 heading`, `u8 widthDm`, `i8` road-side height (4 cm units relative to the pad). The pad level itself is the building's `baseDm` (no separate field). |

## `sites.json` (landmark kits)
Built by `tools/sites/build_sites.py` after `tools/apply_names.py`; read by `js/map/sites.js`. Game coordinates only; the sign names are fictional.
- `sites[]`: `k` kit (`mall gms home super box drug family ffdt cafedt ff cafe konbini fuel pachinko`), `sign {t, bg, fg, key}` (text, background and text colour,
  placeholder key while a name is pending), `b [tx, tz, i]` (building index in that cell's BUILDINGS section) or `a` (index into `add`), `x, z` (building centroid, used to
  check the index), `bh` (body height; the building and its collider are lowered to this), `front [ax, az, bx, bz, nx, nz]`, `ent [[x, z, nx, nz, width]]`,
  `pole [[x, z, heading, type]]`, `park [[x, z, ux, uz, nx, nz, count, kind]]` (a row of 2.5 × 5 m stalls starting at (x, z) on the nose line, along u, cars facing n;
  kind 0 plain, 1 U-shaped lines, 2 on the roof), `roofPark`, `asph [[outer, hole…]]` (asphalt polygons), optional `dt` (drive-through lane polyline), `canopy`
  `[cx, cz, ux, uz, length, width]`, `garden`, `ramp`, `parts`, `carpark`, `ten` (tenant signs).
- `add[]`: buildings missing from PLATEAU 2023 but drawn in OSM (`tx, tz, kind, h, pts`). `drop[]`: `[tx, tz, i]` buildings replaced by them.
- Region packs rebuilt → building indices change → rebuild `sites.json` (the game ignores a site whose building centroid no longer matches).

## `terrain.bin.gz`
`"NGH2" | f32 x0 | f32 z0 | f32 step (16) | u32 nx | u32 nz | nx·nz × i16 decimetres` (row-major by z). Bilinear sampling.

## `nav.bin.gz`
`"NGN2" | u32 nodeCount | nodeCount × (i32 xCm, i32 zCm, i16 yDm) | u32 edgeCount | edgeCount × (u32 a, u32 b, u8 cls, u8 lanes, u8 oneway, u8 flags, u16 n, n × (i32 xCm, i32 zCm, i16 yDm) including both ends)`.
Coordinates here are absolute (not cell-relative). Flags as in ROADLINES.
dataRev 11: at the play-area edge, dead-end edges stop 10 m inside the edge ring (short stubs from a junction to the ring are left out), and a pair of one-way roads that end/start at the ring side by side (the two carriageways of a motorway) is joined by a short one-way U-turn edge (`lanes` 1, `flags` 0) that has no road surface of its own. Bridge and embankment edges keep their height to within 0.4 m.

## Reading it
The game's own decoder is `js/map/format.js` (no dependencies). In Python: `gzip.decompress` + `struct.unpack_from('<…')` with the layouts above.
