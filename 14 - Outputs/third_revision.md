# Third Revision: exercise_v3.ipynb vs Assignment Requirements

This document evaluates `13 - Processing/exercise_v3.ipynb` against every requirement in `11 - Assigment/New_Street_Research_Assignment.md`.

---
## Part 1: Passed Requirements

### 1. Technology Classification
**Requirement (line 62-84):** Classify technologies into Slow Copper, Fast Copper, Slow Cable, Fast Cable, Fiber, or Other based on technology code and download speed.
**Status: PASS** (function `classify_technology()`, called in cell_05). Uses `np.select` with all six conditions matching the spec exactly. Column named `technology_category`.

### 2. Gigabit-Capable Provider Definition
**Requirement (line 86-94):** Fiber (tech 50) OR Cable with download speed >= 500 Mbps.
**Status: PASS** (function `identify_gigabit_capable()`, called in cell_07). Vectorized boolean expression correctly identifies gigabit-capable providers.

### 3. Market Structure Constraints
**Requirement (line 96-111):** Max 2 gigabit-capable providers per location. Available Spots = 2 - current gigabit count. Grandfathered locations (>2) remain unchanged.
**Status: PASS** (function `score_candidates()` in simulation, cell_15 in exploratory). Competition-related columns computed only where needed; simulation recalculates from scratch each year with correct capping at 0 for grandfathered locations.

### 4. Competition Score
**Requirement (line 159-163):** Competition Score = 1 - (Number of Existing Gigabit Providers / 2).
**Status: PASS** (function `score_candidates()`). Formula `(2 - current_gigabit_count).clip(lower=0) / 2` is algebraically equivalent to `1 - (gb_providers / 2)`, both capped at [0, 1].

### 5. Gigabit Coverage Score
**Requirement (line 165-171):** Provider's Gigabit Locations in Block / Total Locations in Block. Normalize to 0-1 across the dataset.
**Status: PASS** (function `score_candidates()`). Computed via merge from `provider_gigabit_by_block_df` DataFrame, then min-max normalized.

### 6. Market Size Score
**Requirement (line 173-178):** Total Unique Locations in Census Block. Normalize to 0-1 across the dataset.
**Status: PASS** (function `score_candidates()`). Block location counts are mapped and min-max normalized.

### 7. Composite Score
**Requirement (line 180-183):** Final Score = (Competition + Normalized GB Coverage + Normalized Market Size) / 3. Rank descending.
**Status: PASS** (function `score_candidates()`). All three components summed and divided by 3. Locations ranked by descending composite score.

### 8. Upgrade Contender Identification
**Requirement (line 116-123):** Providers with Slow Copper or Slow Cable at a location can upgrade to Fiber or Fast Cable.
**Status: PASS** (function `find_upgrade_candidates()`, also inline in cell_09). Filters for `technology_category in ['Slow Copper', 'Slow Cable']`.

### 9. Edge-Out Near-Net Contender Identification
**Requirement (line 125-128):** Provider has no presence at the location but does have presence in the same census block.
**Status: PASS** (function `find_edge_out_candidates()`, also inline in cell_10). Cartesian product of (providers in block x locations in block) minus existing (location, provider) pairs.

### 10. Upgrades Prioritized Over Edge-Out
**Requirement (line 139):** Upgrades always have priority over Edge-Out Near-Net builds.
**Status: PASS** (function `allocate_builds_for_year()`). `type_priority` maps Upgrade=0, Edge-Out=1. Within each location, contenders are sorted with `type_priority` ascending (Upgrade first).

### 11. Within-Type Tie-Breaking
**Requirement (line 141-144):** Within each growth type, rank by: (1) gigabit-capable locations in block descending, (2) total locations in block descending.
**Status: PASS** (function `allocate_builds_for_year()`). Location contenders pre-sorted by `['type_priority', 'provider_gigabit_count_in_block', 'provider_total_locations_in_block']` with ascending `[True, False, False]`.

### 12. Provider Annual Capacity
**Requirement (line 190-200):** floor(0.04 x provider's existing unique footprint). Recalculated at start of each year. Footprint grows with builds.
**Status: PASS** (cell_20 simulation loop). `provider_footprints` recalculated from `market_state` at the start of each year. `.astype(int)` truncates correctly. Edge-out rows added to `market_state` grow the footprint for subsequent years.

### 13. State Annual Capacity
**Requirement (line 202-211):** floor(0.02 x total unique locations). Constant year-over-year.
**Status: PASS** (cell_19 + cell_20). Calculated once as `int(0.02 * total_unique_locations)`. Reset each year to the same fixed value.

### 14. Multi-Year Simulation
**Requirement (line 249, 316):** Simulate 5-10 years or until no more builds are possible.
**Status: PASS** (cell_20). Loop spans `range(2026, 2036)` (10 years) with early break when no builds remain or no candidates exist.

### 15. Provider Footprint Grows YoY
**Requirement (line 238-240, 251-252):** Provider footprints increase based on completed builds. Capacity compounds over time.
**Status: PASS** (cell_20 + function `allocate_builds_for_year()`). Upgrade rows updated in-place; edge-out rows bulk-inserted into `market_state`. Next year's `provider_footprints` is recalculated from the expanded `market_state`.

### 16. Scores Recalculated Each Year
**Requirement (line 243-248):** Recalculate capacities and scores at start of each year based on updated footprints and market state.
**Status: PASS** (cell_20). All lookup structures rebuilt from `market_state` at the start of each year. `score_candidates()` recomputes all scores fresh.

### 17. Location-by-Location Allocation With Contender Fallback
**Requirement (line 221-235):** Iterate locations by score. For each location, try highest-priority contender. If at capacity, try next contender. If state cap hit, stop.
**Status: PASS** (function `allocate_builds_for_year()`). `location_processing_order` iterates locations by max composite score. Pre-sorted contenders tried in priority order. State cap checked before each location.

### 18. Year-Over-Year Summary Table
**Requirement (line 266-278):** Table with columns: year, provider_id, upgrade_builds, edge_out_builds, total_builds.
**Status: PASS** (cell_22). Correct structure with all required columns. Multi-year data produced (one row per provider per year).

### 19. Analysis Report
**Requirement (line 280-288):** Brief written summary addressing: (1) fastest growing providers and why, (2) growth type dominance early vs late, (3) capacity constraint trends.
**Status: PASS** (cell_24). All three questions addressed with actual simulation output: PROV_005 (1,039 builds) leads; edge-outs overtake upgrades in Year 2 (2027); state cap at 100% utilization all 10 years.

### 20. CSV Export
**Requirement (line 325):** Year-over-year summary table in CSV format.
**Status: PASS** (cell_23). Exports to `14 - Outputs/yoy_summary.csv` with automatic directory creation.

### 21. README With Run Instructions
**Requirement (line 326):** "A README with instructions on how to run your code."
**Status: PASS.** `README.md` in the project root contains project description, requirements, installation, run instructions, project structure, and output description. Updated to reference exercise_v3.ipynb.

### 22. Modular Functions With Docstrings
**Requirement (line 262-264):** "Well-commented, production-quality Python code... a Jupyter notebook with functions to organize your code. Modular functions with clear docstrings."
**Status: PASS.** Six functions defined after imports, each with Google-style docstrings (Args/Returns). Functions encapsulate all core business logic:
- `classify_technology()` — technology classification
- `identify_gigabit_capable()` — gigabit flag
- `find_upgrade_candidates()` — upgrade contender identification
- `find_edge_out_candidates()` — edge-out contender identification
- `score_candidates()` — composite score calculation with all three components
- `allocate_builds_for_year()` — greedy location-by-location allocation

The simulation loop (cell_20) calls these functions, making the year-by-year flow readable at a glance while the function bodies contain the full implementation with original inline comments preserved.


## Part 2: Failed Requirements

None. All assignment requirements pass.


## Part 3: Verification

### Output Verification (v3 vs v2)
exercise_v3.ipynb produces **identical results** to exercise_v2.ipynb:

| Metric                  | v2                          | v3                          | Match |
| ----------------------- | --------------------------- | --------------------------- | ----- |
| Total builds (10 years) | 12,190                      | 12,190                      | Yes   |
| Year 2026               | 1,219 upgrades, 0 edge-outs | 1,219 upgrades, 0 edge-outs | Yes   |
| Year 2027               | 478 upgrades, 741 edge-outs | 478 upgrades, 741 edge-outs | Yes   |
| Top provider            | PROV_005 (1,039)            | PROV_005 (1,039)            | Yes   |
| State cap utilization   | 100% all years              | 100% all years              | Yes   |
| Edge-out crossover year | 2027                        | 2027                        | Yes   |

### Key Simulation Results
- **10 years simulated** (2026-2035), all hitting the state cap of 1,219 builds/year
- **Year 1 (2026):** 100% upgrades (1,219), establishing the priority rule
- **Year 2 (2027):** Edge-outs overtake (741 vs 478), the crossover point
- **Top 5 providers:** PROV_005 (1,039), PROV_001 (992), PROV_007 (979), PROV_008 (962), PROV_011 (937)
- **State cap is binding** every year; provider caps are never the system bottleneck



## Part 4: Changes from v2 to v3

### Added: Function definitions cell (after imports)
Six modular functions with docstrings were added. Each function body contains the same implementation logic and preserves the original inline comments from v2. The functions are:

1. `classify_technology(dataframe)` — wraps cell_05 logic
2. `identify_gigabit_capable(dataframe)` — wraps cell_07 logic
3. `find_upgrade_candidates(market_state)` — wraps upgrade identification from simulation loop
4. `find_edge_out_candidates(market_state, edge_out_universe)` — wraps edge-out identification from simulation loop
5. `score_candidates(candidates, ...)` — wraps scoring logic (sections 3.4) from simulation loop; uses merge instead of apply (R2)
6. `allocate_builds_for_year(candidates, ...)` — wraps allocation logic (section 3.5) from simulation loop; uses pre-grouped candidates (R3)

### Modified: cell_05 (Technology Classification)
Now calls `classify_technology(source_data)` instead of inline `np.select`.

### Modified: cell_07 (Gigabit Identification)
Now calls `identify_gigabit_capable(source_data)` instead of inline boolean expression.

### Modified: cell_20 (Simulation Loop)
Sections 3.3, 3.4, and 3.5 replaced with function calls:
- `find_upgrade_candidates(market_state)` + `find_edge_out_candidates(market_state, edge_out_universe)`
- `score_candidates(candidates, ...)`
- `allocate_builds_for_year(candidates, ...)`

Sections 3.1 (capacities) and 3.2 (fast trackers) remain inline since they are setup/data-preparation steps specific to the simulation loop context.

### Unchanged
All other cells (data loading, exploratory section, final report, CSV export, analysis report) remain identical to v2.



## Summary

| Category   | Count | Items                                                                                                                                                                                                                                                                                                                                                                                          |
| ---------- | ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Passed** | 22    | Technology classification, gigabit definition, market constraints, competition score, GB coverage score, market size score, composite score, upgrade ID, edge-out ID, upgrade priority, tie-breaking, provider capacity, state capacity, multi-year simulation, footprint growth, score recalculation, allocation logic, summary table, analysis report, CSV export, README, modular functions |
| **Failed** | 0     | —                                                                                                                                                                                                                                                                                                                                                                                              |

All assignment requirements are met. The v3 notebook produces identical results to v2 while adding the modular function organization requested by the assignment specification.
