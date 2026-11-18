# Elden Ring data (DuckDB / Parquet)

Game-data tables and SQL macros for Elden Ring PvE build theorycrafting: weapon attack rating, motion values,
frame data, stance (poise) damage, status buildup, enemy defenses and thresholds, spells, Ashes of War, buffs, armor.

- `data/*.parquet`: 77 tables (zstd Parquet, ~2 MB total)
- `sql/schema.sql`: 41 macros and views (`ar`, `catalyst_scaling`, `enemy_hit_damage`, `hits_for_procs`, `enemy_threshold`, `class_rl`, …), in dependency order
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
| Armor, talisman and weapon passives (`gear_passives`, `skill_subcats`) | 1.12 | resident SpEffects of EquipParamProtector / Accessory / Weapon; boosted attacks from ATK_SUB_CATEGORY |
| "This boosts that" links (`synergy`, `spell_school`) | 1.12 | derived from `gear_passives` and `buffs`, plus a few hand-entered rows (see `origin`, `evidence`) |
| Item regions (`item_region`, `region_tier`) | community | Region fields transcribed from [game-checklists.com](https://game-checklists.com) (834 weapons, talismans, spells, Ashes of War); tier = the game's area-scaling tier for that region |

`ar()` and `catalyst_scaling()` match an independent engine ([Raed-Jani/soulslike-build-planner](https://github.com/Raed-Jani/soulslike-build-planner))
with 0 mismatches over thousands of damage, status and spell-scaling checks. That verifies the math against community calculators, not against the game.

Not included: item and dialogue text, wiki-derived text, translation sheets, personal notes.

## Notes

- Quote stance damage from `attack_poise_pve.poise_pve_total` (MV × weapon base poise / 100), not poise MVs.
- Multi-hit attacks are treated as one packet for enemy defense (slight overestimate).
- `enemies.res_madness` is unreliable.

ELDEN RING and its game data belong to FromSoftware / Bandai Namco. Fan-made, non-commercial, not affiliated.

### gear_passives

One row per armor piece, talisman or weapon whose resident effect changes attack power or stats.
PvE multiplier per damage type = AttackRate x AttackPowerRate x atkEnemyDmgCorrectRate.
`applies` is the kind of attack it boosts: `any`, `charged`, `jump`, `guard_counter`, `chain_end`, `roar`, `skill`, `two_hand`,
`roll_backstep`, `running`, `critical`, `mounted`, `skill_subcat` (specific skills; join `skill_subcats` on the sub-category),
`spell` (one spell school: staves, seals, some armor) or `other` (arrows, pots, thrown weapons). Sub-category meanings were
checked against talisman descriptions; Fire Knight skills (130) are named from item pages because their DLC rows are unnamed.


### item_region / region_tier

Where an item is found and roughly when. `item_region(kind, item, region, tier, min_rl, origin)`, kind = weapon, talisman, spell or ash.
`tier` is the game's own area-scaling tier for the region. `min_rl` is a **judgment** of the rune level at which players
typically reach that region, not game data. Roundtable Hold trades use the boss's region where known; 11 rows are unresolved (null tier).
Armour is not covered. Region names are credited to game-checklists.com.

### synergy / spell_school

`synergy`: one row per "source boosts target" link (source item or buff, slot, relation `boosts` or `grants_stat`, target kind
attack / damage_type / skill / stat / spell_school / spell_type, magnitude, trigger, condition, scope). `spell_school` maps
123 spells to the school that gear boosts.

### Lore map (lore_region, lore_item, lore_mention, lore_link)

Tags, not text. `lore_item`: 2,638 described items with the region they are found in (from `item_region`; armour, key items
and spirit ashes have none). `lore_mention`: which places, figures, factions and powers each description names, matched by
hand-listed English and Japanese terms (`lang` = en, ja or both; `ja` means only the Japanese text names it). `lore_region`:
22 map regions with a schematic x,y layout (not game coordinates). `lore_link`: item pairs that share rare Japanese terms
(`hidden` = the English text does not show the link). The descriptions themselves are not included.

### roar_heavies

The strong attacks that War Cry, Barbaric Roar and Braggart's Roar swap in, per weapon class and grip (regulation 1.12
AtkParam_Pc): motion value, poise and stamina multipliers, and `roar_tag` (ATK_SUB_CATEGORY 106, what Roar Medallion and
Highland Axe boost). Which rows belong to which skill is inferred from the row layout, not named in the game files.

### spell_subcats

Every sub-category Magic.param (1.12) gives each player spell. `spell_school` now carries a spell's second school where the game gives one (12 spells), and a Magma school (sub-category 123).
