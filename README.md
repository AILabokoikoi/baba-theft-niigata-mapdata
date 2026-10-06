# baba-theft-niigata-mapdata

ゲーム「ババアセフトシニアカー Niigata」の地図データベースです。
The map database of the game "ババアセフトシニアカー Niigata" (Baba Theft Senior Car Niigata).

**Map data © OpenStreetMap contributors** — https://www.openstreetmap.org/copyright
Licence: **Open Database License (ODbL) 1.0** — see [LICENSE](LICENSE).

---

## 日本語

### これは何か
新潟市（中央区・江南区・秋葉区のあたり）を舞台にしたゲームが使っている地図データベースです。
OpenStreetMap のデータ（と国土交通省の 3D都市モデル PLATEAU）を、**AI動画ラボコイコイ** がゲームのために加工しました。
ODbL 1.0 の決まり（派生データベースを提供すること）に従って公開しています。

- 道路、経路のグラフ、鉄道、川・海、土地利用、建物、地形の高さ、POI（店・神社・駅などの位置と種類）
- 座標はゲームの座標（m）です。**緯度経度は入っていません**。地図は「縮めて」あるので、実際の距離とは違います。
- 店・神社・交番・病院などの**名前は架空**です。実在の会社・店・施設とは関係ありません（駅名 6 つだけ実在）。
- ゲームのプログラム、3D モデル、テクスチャ、写真は入っていません。

### ライセンス
- データベース全体：**ODbL 1.0**（https://opendatacommons.org/licenses/odbl/1-0/ ）
- 中身（OpenStreetMap から作った部分）：**DbCL 1.0**（OpenStreetMap と同じ）
- 使うときは「© OpenStreetMap contributors」と ODbL を表示してください。このデータベースを変えて公開・利用するときは、その派生データベースも ODbL で提供する必要があります。

### 出典
- 地図データ © OpenStreetMap contributors（Open Database License 1.0） https://www.openstreetmap.org/copyright
- 出典：「3D都市モデル（Project PLATEAU）新潟市（2023年度）」（国土交通省）（https://www.geospatial.jp/ckan/dataset/plateau-15100-niigata-shi-2023 ）を加工して AI動画ラボコイコイ 作成。公共データ利用規約 第1.0版（PDL1.0） https://www.digital.go.jp/resources/open_data/public_data_license_v1.0 （著作権者：新潟市）
  — 建物の形と高さ、地形の高さ（DEM）、土地利用
- 出典：国土地理院 地理院タイル（全国最新写真（シームレス））を加工して作成（国土地理院コンテンツ利用規約 https://www.gsi.go.jp/kikakuchousei/kikakuchousei40182.html ）
  — `data/parking_lots.json` の駐車場の白線の並びだけ（写真を見て読み取った数字。画像は入っていません）

国土交通省、新潟市、国土地理院、OpenStreetMap Foundation は、このデータベースとゲームを作成・監修していません。

### ファイル
| ファイル | 中身 |
|---|---|
| `data/index.json` | 範囲、セルの一覧、地形・経路の見出し、POI、鉄道、区域、勤め先の表、地図の縮め方（`warp`） |
| `data/r_<rx>_<rz>.bin.gz`（32 個） | 4096 m 四方ごとの地図（道路・建物・水・土地利用・木・小物・庭など）。gzip のバイナリ |
| `data/terrain.bin.gz` | 地形の高さ（16 m 格子） |
| `data/nav.bin.gz` | 道路の経路グラフ |
| `data/sites.json` | 郊外の店などの配置（看板の名前は架空） |
| `data/parking_lots.json` | 駐車場 8 か所の白線の並び |
| `data/overview.jpg`, `data/mapview.jpg` | 上の データから描いた全体の地図の絵（8 m／画素。写真ではありません） |
| `FORMAT.md` | ファイル形式の説明（英語） |
| `METHOD.md` | 作り方 |
| `SHA256SUMS.txt` | 全ファイルのハッシュ |

`data/` のファイルは、ゲームが配っているものとバイト単位で同じです。

---

## English

### What this is
The map database used by a game set in Niigata City, Japan (around Chūō, Kōnan and Akiha wards).
It was built from OpenStreetMap data (and Japan's PLATEAU 3D city model) and processed for the game by **AI動画ラボコイコイ** (AI Video Lab Koikoi).
It is published to meet the ODbL 1.0 requirement to offer the derivative database.

- Roads, a routing graph, railways, rivers and sea, land use, buildings, terrain heights, points of interest (positions and categories of shops, shrines, stations …).
- Coordinates are game-frame metres. **No latitude/longitude is included.** The map is compressed ("warped"), so distances differ from reality.
- Names of shops, shrines, police, hospitals etc. are **fictional** and not related to any real company, shop or facility (only six railway-station names are real).
- No game code, 3D models, textures or photos are included.

### Licence
- The database as a whole: **ODbL 1.0** (https://opendatacommons.org/licenses/odbl/1-0/)
- Contents derived from OpenStreetMap: **DbCL 1.0** (as OpenStreetMap does)
- Credit "© OpenStreetMap contributors" and the ODbL when you use it. If you publicly use an adapted version, you must offer that derivative database under the ODbL too.

### Sources
- Map data © OpenStreetMap contributors (Open Database License 1.0) https://www.openstreetmap.org/copyright
- 3D City Model (Project PLATEAU) Niigata City (FY2023), Ministry of Land, Infrastructure, Transport and Tourism, https://www.geospatial.jp/ckan/dataset/plateau-15100-niigata-shi-2023 — processed by AI動画ラボコイコイ. Public Data License 1.0 (PDL1.0) https://www.digital.go.jp/resources/open_data/public_data_license_v1.0 (copyright holder: Niigata City). Used for building footprints and heights, terrain (DEM), land use.
- Geospatial Information Authority of Japan, GSI Tiles (Seamless Photo), processed — only the parking-stall line layout in `data/parking_lots.json` (numbers read off the photos; no image pixels). GSI content terms: https://www.gsi.go.jp/kikakuchousei/kikakuchousei40182.html

MLIT, Niigata City, GSI and the OpenStreetMap Foundation did not create, review or endorse this database or the game.

### Files
See the table above. The binary layout is documented in [FORMAT.md](FORMAT.md) and the processing in [METHOD.md](METHOD.md). The files in `data/` are byte-identical to the ones shipped with the game.

### Reading it
gzip + little-endian structs; e.g. in Python `gzip.decompress` and `struct.unpack_from('<…')` with the layouts in FORMAT.md.
