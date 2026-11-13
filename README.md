# Elden Ring data (DuckDB / Parquet)

Game-data tables and SQL macros for Elden Ring PvE build theorycrafting: weapon attack rating, motion values,
frame data, stance (poise) damage, status buildup, enemy defenses and thresholds, spells, Ashes of War, buffs, armor.

- `data/*.parquet`: 65 tables (zstd Parquet, ~2 MB total)
- `sql/schema.sql`: 39 macros and 2 views (`ar`, `catalyst_scaling`, `enemy_hit_damage`, `hits_for_procs`, `enemy_threshold`, `class_rl`, …), in dependency order
- `sql/schema_elden_ring_qualified.sql`: the same macros with every table and macro call qualified as `elden_ring.main.*`, so they work from any default database (needed for MotherDuck Dives)
- `sql/load_motherduck.sql`: builds the tables in MotherDuck straight from this repo's raw URLs

Exported from a private working database; regenerated with `scripts/export_public_data.py` there.

## Load

DuckDB or MotherDuck:

```sql
CREATE DATABASE elden_ring;  -- MotherDuck; or open a local .duckdb file
USE elden_ring;
-- run sql/load_motherduck.sql, then sql/schema.sql
SELECT * FROM ar('Keen Uchigatana', 18, 50, 9, 9, 9, 25, true);
```

## Data versions

| Data | Version | Source |
|---|---|---|
| Weapon AR, scaling, requirements | 1.17 | [ThomasJClark/elden-ring-weapon-calculator](https://github.com/ThomasJClark/elden-ring-weapon-calculator) (MIT) |
| Spells, enemies, buffability, Ashes of War, armor, equip load | 1.17.1 | regulation.bin, decoded with [soulsmods/Paramdex](https://github.com/soulsmods/Paramdex) |
| Weapon motion values, frames (30 fps), status/poise/stamina MVs | 1.16.1 | community MV workbook (cryptidtracker) |
| Bullets, other SpEffects | 1.12 | [garyttierney/elden-ring-param-archive](https://github.com/garyttierney/elden-ring-param-archive) |
| `bosses` (26 bosses) | community | Community DPS Calc spreadsheet |
| NG+ scaling (`enemy_ng`, `ng_cycle`) | 1.12 | regulation SpEffectParam (area + 400 rows) and ClearCountCorrectParam |

`ar()` and `catalyst_scaling()` match an independent engine ([Raed-Jani/soulslike-build-planner](https://github.com/Raed-Jani/soulslike-build-planner))
with 0 mismatches over thousands of damage, status and spell-scaling checks. That verifies the math against community calculators, not against the game.

Not included: item and dialogue text, wiki-derived text, translation sheets, personal notes.

## Notes

- Quote stance damage from `attack_poise_pve.poise_pve_total` (MV × weapon base poise / 100), not poise MVs.
- Multi-hit attacks are treated as one packet for enemy defense (slight overestimate).
- `enemies.res_madness` is unreliable.

ELDEN RING and its game data belong to FromSoftware / Bandai Namco. Fan-made, non-commercial, not affiliated.
