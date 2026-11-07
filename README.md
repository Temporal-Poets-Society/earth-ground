# Earth ground data (30″)

Ground data at 30″ (1/120°, about 0.9 km) for the Vintage Story mod **Earth**: soil groups, depth to bedrock and
permafrost probability, which Earth's ground columns read to choose the soil, the loose cover and its thickness, and
frozen ground. Earth reads them from `VintagestoryData/ModData/earth/ground`. Without them it uses the coarser 2′ sets
in the Earth Ground Data pack (`earthgrounddata`).

They are here, not on the Vintage Story mod database, because the three files are 172 MB together and an archive
there may hold 40 MB.

## Files

The data is in the release [`egts-v1`](../../releases/tag/egts-v1): three tile sets, a manifest and the attribution.

| File | Bytes | Tiles | Field | Source | Licence | SHA-256 |
|---|---|---|---|---|---|---|
| `global-soilgroup-30s.egts` | 20,979,888 | 296 | `soil_group` | SoilGrids 2.0 WRB group | CC BY 4.0 | `b1b1f22d7c1d46a142a1ebdb86f802d4956213a4b26d61c033d0d214828089a8` |
| `global-shangguan-30s.egts` | 134,774,489 | 365 | `quaternary_m` | Shangguan et al. 2017 depth to bedrock | CC BY 4.0 | `d4c165f7b346aed9ba91f39778dda2f5c47962a857628dd81be8ee98282bc40d` |
| `global-permafrost-30s.egts` | 16,496,922 | 153 | `permafrost_pct` | Obu et al. 2019 permafrost probability | CC BY 3.0 | `ff9117624a5bdb6ce415832fd9634deeae1ce21ccaab2ed6dc0036dbae741739` |
| `manifest.json` | | | | | | in this repository too |
| `ATTRIBUTION.txt` | | | | | | in this repository too |

To install by hand, put the three `.egts` files in `VintagestoryData/ModData/earth/ground/` on the server (or the
single-player machine) before creating a world. Chunks generated with and without them differ, so add or remove them
only between worlds.

### Version

`egts-v1` names this bake. Earth versions that list these file names use them as they are. A new bake or a new format
gets a new tag and new files; this release stays.

### manifest.json

```json
{ "format": "egts", "formatVersion": 1, "tileFormat": "EGT1", "release": "egts-v1",
  "installTo": "VintagestoryData/ModData/earth/ground",
  "sets": [ { "name": "global-soilgroup-30s.egts", "path": "ground/global-soilgroup-30s.egts", "bytes": 20979888,
              "sha256": "…", "tiles": 296, "field": "soil_group", "unit": "…", "source": "…", "licence": "CC BY 4.0" }, … ] }
```

## Format

**Tile set (EGTS, version 1).** `"EGTS"`, the tile count (uint32, little-endian), then for each tile its length
(uint32) and the tile. Earth unpacks a tile only when it generates a column inside it.

**Tile (EGT1).** `"EGT1"`, the header's length (uint32), a UTF-8 JSON header, then the field planes compressed with
zlib as one block. The header gives `name`, `source`, `licence`, `west` and `north` (the tile's corner in degrees),
`step` (1/120°), `nx` and `ny` (1,200 each: a tile is 10° square) and `fields`. Each field has `name`, `type` (all
three are `u8`), `scale`, `offset`, `nodata` (255), `interp` (`linear` or `category`), for categories a `legend`, and
optionally a `curve`. Rows run north to south; cell (r, c) covers latitude north − (r + 1) / 120 to north − r / 120
and longitude west + c / 120 to west + (c + 1) / 120. A stored value v means `scale × v + offset`; with `"curve":
"log1p"` the value is `exp(scale × v + offset) − 1`. Categories store an index into `legend`. A cell holding `nodata`
has no value, and Earth asks the next tile below it (the 2′ sets).

- `soil_group`: the index of one of 30 WRB reference soil groups, named in the legend (Acrisols … Vertisols).
- `quaternary_m`: depth to bedrock in metres, log-coded: v = round(40 ln(1 + m)), so m = exp(v / 40) − 1, about 2.5 %
  steps, 254 = 570 m.
- `permafrost_pct`: permafrost probability in percent, 1–100; below 1 % is stored as no data.

## Sources and changes

**Soil groups.** SoilGrids 2.0, ISRIC – World Soil Information: WRB most probable reference soil group
(`MostProbable`), from the ISRIC web coverage service at its full resolution (fetched as 7.5″ cells). Poggio, L., de
Sousa, L. M., Batjes, N. H., Heuvelink, G. B. M., Kempen, B., Ribeiro, E. and Rossiter, D. (2021): SoilGrids 2.0:
producing soil information for the globe with quantified spatial uncertainty. SOIL 7, 217–240,
https://doi.org/10.5194/soil-7-217-2021. Licence CC BY 4.0. Changes: each 30″ cell takes the most common class of its
16 full-resolution cells; the service writes 0 for both Acrisols and no data, so a 0 counts as Acrisols only where the
Acrisols probability says so; gaps of up to 12 cells (towns, lakes, rivers) take the nearest mapped class.

**Depth to bedrock.** Shangguan, W., Hengl, T., Mendes de Jesus, J., Yuan, H. and Dai, Y. (2017): Mapping the global
depth to bedrock for land surface modeling. Journal of Advances in Modeling Earth Systems 9, 65–88,
https://doi.org/10.1002/2016MS000686. Data: SoilGrids250m 2017-03-10, absolute depth to bedrock `BDTICM_M`, the 1 km
aggregate `BDTICM_M_1km_ll.tif` (ISRIC). Licence CC BY 4.0. Changes: centimetres to metres, log-coded in one byte as
above; the grid is the source's.

**Permafrost.** Obu, J., Westermann, S., Kääb, A. and Bartsch, A. (2018): Ground Temperature Map, 2000-2016, Northern
Hemisphere Permafrost. PANGAEA, https://doi.org/10.1594/PANGAEA.888600 (Obu et al. 2019, Earth-Science Reviews 193,
299–316, https://doi.org/10.1016/j.earscirev.2019.04.023). Licence CC BY 3.0. Changes: the permafrost probability
(`PERPROB`) of its 1 km polar stereographic grid is read at each 30″ cell centre from 25° N northward and stored in
whole percent.

## Licence and notices

See [LICENSE.md](LICENSE.md) and [ATTRIBUTION.txt](ATTRIBUTION.txt). This repository, Earth and Temporal Poets
Society are not endorsed by ISRIC, the authors of the data, or PANGAEA.
