# How this database was made / 作り方

English first, 日本語は下にあります。

## Sources

| Source | Retrieved | Used for |
|---|---|---|
| OpenStreetMap (Overpass API), © OpenStreetMap contributors, ODbL 1.0 | 2026-09-26 (Niigata City area, plus two edge strips); airport runways/taxiways/aprons (`aeroway=*`) 2026-09-30 | road centre lines and classes, lanes, one-way, bridges/tunnels; the road graph; railways and platforms; water and coastline; land use outside PLATEAU coverage; parks, parking areas, cemeteries; positions and categories of points of interest (shops, shrines, stations …); buildings where PLATEAU has none; ward boundaries; place names used only to make fictional branch names |
| 3D都市モデル（Project PLATEAU）新潟市（2023年度）, 国土交通省, PDL1.0 | 2026-09 | LOD1 building footprints, measured heights and ground elevations; DEM (resampled to a 16 m height grid); land use (都市計画基礎調査, rasterised at 4 m) |
| 国土地理院 地理院タイル（全国最新写真（シームレス）） | 2026-09-29 | viewed only to read the position, direction and count of white parking-stall lines at 8 parking lots (`parking_lots.json`). No image pixels are in this database |

## Processing steps (the game's offline build)

1. **Extract.** PLATEAU CityGML buildings, DEM and land use were read for the game's bounding box. OSM data was fetched with Overpass queries for the same box.
2. **Convert to a local frame.** Latitude/longitude were projected to metres in a local frame (x east, z south, y above sea level).
3. **Merge and clean.** Roads were buffered into surfaces by class/lane count; sidewalks were planned from the clearance to buildings; buildings crossed by drivable roads were dropped or trimmed at the road edge; OSM buildings fill gaps in PLATEAU; trees, vending machines, traffic lights and bus stops were placed procedurally (not from any source); house yards (lots) were generated procedurally around detached houses.
4. **Map compression ("warp", since dataRev 6).** Landmark areas keep their real size and shape and are only moved; the land between them is shrunk. Buildings keep their real footprint and height and are moved/rotated as a whole; overlapping ones are thinned out. Distances between places are therefore shorter than in reality. The per-area offsets are listed in `data/index.json` → `warp`.
5. **Names.** Shops, shrines, police boxes and stations, hospitals, funeral halls, post offices and other organisations were given **fictional** names. All real building and road names were removed except six railway-station names. No OSM `name`/`brand` of a shop is stored.
6. **Removed before publication.** OSM element ids of points of interest and all latitude/longitude values (the coordinates are game-frame metres only).
7. **Encode.** Data was written into 512 m cells grouped into region packs (`r_<rx>_<rz>.bin.gz`), a terrain grid, a road graph and `index.json`; two overview rasters (`overview.jpg`, `mapview.jpg`) were rendered from the land use, terrain, buildings and roads (no aerial photos). See `FORMAT.md` for the exact binary layout.
8. **Game-only additions (not from any source).** Gameplay places (`ume_home`, `tanaka_home`, `ume_paddy`, `mansion`, …), landmark "kits" in `sites.json` (fictional signs, parking stall rows, drive-through lanes) and the workplace table are invented for the game.

## Use in the game

- The browser version of the game reads the files in `data/` directly.
- The Unity version converts a part of the same files (around Niigata Station) into 3D meshes (glTF) and a small JSON of prop positions at build time. No other map data source is added in that step, so the database here is the source of everything map-related in both versions. (Photo-textured PLATEAU LOD2 building models, used for some buildings in central Niigata, are separate 3D models from PLATEAU, not from OpenStreetMap, and are not part of this database.)
- One game file is **not** included: a hand-made list of appearance overrides for a few buildings near Niigata Station (wall style, fictional sign text). It is game art direction, not derived from OpenStreetMap.

---

## 日本語

### 元にしたデータ
- **OpenStreetMap**（© OpenStreetMap contributors、ODbL 1.0）：2026-09-26 に Overpass API で取得（空港の滑走路などは 2026-09-30）。道路、経路のグラフ、鉄道、水域・海岸線、PLATEAU の範囲外の土地利用、公園・駐車場、POI（店・神社・駅など）の位置と種類、PLATEAU に無い建物、区の境界。
- **3D都市モデル（Project PLATEAU）新潟市（2023年度）**（国土交通省、PDL1.0、著作権者：新潟市）：建物（LOD1）の形・高さ、DEM（16 m 格子に再サンプリング）、土地利用（4 m 格子）。
- **国土地理院 地理院タイル（全国最新写真（シームレス））**：駐車場 8 か所の白線の位置・向き・数を見て読み取っただけ（`parking_lots.json`）。写真の画素は入っていません。

### 加工
緯度経度をゲームの座標（m）に変換 → 道路・建物・歩道などを統合・整理 → 地図を縮める（目印の場所は形と大きさを保って移動し、間の土地を縮める）→ 店・神社・交番・病院・郵便局などに**架空の名前**を付け、駅 6 つ以外の実在の名前を削除 → POI の OSM の ID と緯度経度をすべて削除 → `FORMAT.md` の形式で書き出し。木・自販機・信号・バス停・家の庭はプログラムで置いたもので、元データにはありません。ゲームの場所（ウメの家など）と `sites.json` の看板などはゲームのための創作です。

### ゲームでの使い方
ブラウザ版はこの `data/` をそのまま読みます。Unity 版は同じファイルの一部（新潟駅のまわり）をビルド時に 3D のメッシュに変換するだけで、ほかの地図データは足していません。新潟駅前の数棟の見た目の上書き（外壁の型・架空の看板の文字）を書いたゲームのファイル 1 つは、OSM から作ったものではないので入れていません。
