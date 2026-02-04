# Second Revision (Updated): exercise_v2.ipynb vs Assignment Requirements

This document evaluates `13 - Processing/exercise_v2.ipynb` against every requirement in `11 - Assigment/New_Street_Research_Assignment.md`.


## Summary
The v2 notebook correctly implements all algorithmic and deliverable requirements from the assignment except for modular function organization. This is the sole remaining item for v3.
---

## Part 1: Passed Requirements
### 1. Technology Classification
**Requirement (line 62-84):** Classify technologies into Slow Copper, Fast Copper, Slow Cable, Fast Cable, Fiber, or Other based on technology code and download speed.
**Status: PASS** (cell_05). Uses `np.select` with all six conditions matching the spec exactly. Column named `technology_category`.
### 2. Gigabit-Capable Provider Definition
**Requirement (line 86-94):** Fiber (tech 50) OR Cable with download speed >= 500 Mbps.
**Status: PASS** (cell_07). Vectorized boolean expression correctly identifies gigabit-capable providers.
### 3. Market Structure Constraints
**Requirement (line 96-111):** Max 2 gigabit-capable providers per location. Available Spots = 2 - current gigabit count. Grandfathered locations (>2) remain unchanged.
**Status: PASS** (cell_20 in simulation, cell_15 in exploratory). Competition-related columns computed only where needed; simulation recalculates from scratch each year with correct capping at 0 for grandfathered locations.
### 4. Competition Score
**Requirement (line 159-163):** Competition Score = 1 - (Number of Existing Gigabit Providers / 2).
**Status: PASS** (cell_20 in simulation). Formula `(2 - current_gigabit_count).clip(lower=0) / 2` is algebraically equivalent to `1 - (gb_providers / 2)`, both capped at [0, 1].
### 5. Gigabit Coverage Score
**Requirement (line 165-171):** Provider's Gigabit Locations in Block / Total Locations in Block. Normalize to 0-1 across the dataset.
**Status: PASS** (cell_20 in simulation). Computed via merge from `provider_gigabit_by_block` DataFrame, then min-max normalized.
### 6. Market Size Score
**Requirement (line 173-178):** Total Unique Locations in Census Block. Normalize to 0-1 across the dataset.
**Status: PASS** (cell_20 in simulation). Block location counts are mapped and min-max normalized.
### 7. Composite Score
**Requirement (line 180-183):** Final Score = (Competition + Normalized GB Coverage + Normalized Market Size) / 3. Rank descending.
**Status: PASS** (cell_20). All three components summed and divided by 3. Locations ranked by descending composite score.
### 8. Upgrade Contender Identification
**Requirement (line 116-123):** Providers with Slow Copper or Slow Cable at a location can upgrade to Fiber or Fast Cable.
**Status: PASS** (cell_09, cell_20). Filters for `technology_category in ['Slow Copper', 'Slow Cable']`.
### 9. Edge-Out Near-Net Contender Identification
**Requirement (line 125-128):** Provider has no presence at the location but does have presence in the same census block.
**Status: PASS** (cell_10, cell_20). Cartesian product of (providers in block x locations in block) minus existing (location, provider) pairs.
### 10. Upgrades Prioritized Over Edge-Out
**Requirement (line 139):** Upgrades always have priority over Edge-Out Near-Net builds.
**Status: PASS** (cell_20). `type_priority` maps Upgrade=0, Edge-Out=1. Within each location, contenders are sorted with `type_priority` ascending (Upgrade first).
### 11. Within-Type Tie-Breaking
**Requirement (line 141-144):** Within each growth type, rank by: (1) gigabit-capable locations in block descending, (2) total locations in block descending.
**Status: PASS** (cell_20). Location contenders pre-sorted by `['type_priority', 'provider_gigabit_count_in_block', 'provider_total_locations_in_block']` with ascending `[True, False, False]`.
### 12. Provider Annual Capacity
**Requirement (line 190-200):** floor(0.04 x provider's existing unique footprint). Recalculated at start of each year. Footprint grows with builds.
**Status: PASS** (cell_20). `provider_footprints` recalculated from `market_state` at the start of each year. `.astype(int)` truncates correctly. Edge-out rows added to `market_state` grow the footprint for subsequent years.
### 13. State Annual Capacity
**Requirement (line 202-211):** floor(0.02 x total unique locations). Constant year-over-year.
**Status: PASS** (cell_19 + cell_20). Calculated once as `int(0.02 * total_unique_locations)`. Reset each year to the same fixed value. Total locations remain constant since edge-outs target existing locations.
### 14. Multi-Year Simulation
**Requirement (line 249, 316):** Simulate 5-10 years or until no more builds are possible.
**Status: PASS** (cell_20). Loop spans `range(2026, 2036)` (10 years) with early break when no builds remain or no candidates exist.
### 15. Provider Footprint Grows YoY
**Requirement (line 238-240, 251-252):** Provider footprints increase based on completed builds. Capacity compounds over time.
**Status: PASS** (cell_20). Upgrade rows updated in-place; edge-out rows bulk-inserted into `market_state`. Next year's `provider_footprints` is recalculated from the expanded `market_state`.
### 16. Scores Recalculated Each Year
**Requirement (line 243-248):** Recalculate capacities and scores at start of each year based on updated footprints and market state.
**Status: PASS** (cell_20). All lookup structures (`location_gigabit_counts`, `provider_gigabit_by_block`, `provider_total_locations_by_block`) and all candidate scores are rebuilt from `market_state` at the start of each year.
### 17. Location-by-Location Allocation With Contender Fallback
**Requirement (line 221-235):** Iterate locations by score. For each location, try highest-priority contender. If at capacity, try next contender. If state cap hit, stop.
**Status: PASS** (cell_20). `location_processing_order` iterates locations by max composite score. For each location, pre-sorted contenders are tried in priority order. If a contender's provider is at capacity, `continue` to the next contender. State cap checked before each location.
### 18. Year-Over-Year Summary Table
**Requirement (line 266-278):** Table with columns: year, provider_id, upgrade_builds, edge_out_builds, total_builds.
**Status: PASS** (cell_22). Correct structure with all required columns. Multi-year data produced (one row per provider per year).
### 19. Analysis Report
**Requirement (line 280-288):** Brief written summary addressing: (1) fastest growing providers and why, (2) growth type dominance early vs late, (3) capacity constraint trends.
**Status: PASS** (cell_24). All three questions addressed with actual simulation output: specific provider names and build counts, exact crossover year (2027), and state cap binding analysis (100% utilization all 10 years).
### 20. CSV Export
**Requirement (line 325):** Year-over-year summary table in CSV format.
**Status: PASS** (cell_23). Exports to `14 - Outputs/yoy_summary.csv` with automatic directory creation.
### 21. README With Run Instructions
**Requirement (line 326):** "A README with instructions on how to run your code."
**Status: PASS.** `README.md` in the project root contains project description, requirements, installation, running instructions, project structure, and output description.



## Part 2: Failed Requirements

### F1. Modular Functions With Docstrings
**Requirement (line 262-264):** "Well-commented, production-quality Python code... a Jupyter notebook with functions to organize your code. Modular functions with clear docstrings."
**Status: FAIL.** All logic is inline procedural code. No functions are defined in the notebook. The assignment explicitly asks for modular functions as a code organization strategy, and this is listed under the "Code Quality" evaluation criterion (line 296-297).





