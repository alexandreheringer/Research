# ISP Year-Over-Year Growth Analysis

Multi-year expansion forecast model for internet service providers (ISPs) in a simplified single-state market. Determines which locations each provider should prioritize for network upgrades or new builds, subject to provider-level and state-level capacity constraints.

## Requirements

- Python 3.10+
- pandas
- numpy

## Installation

# Install dependencies
uv sync

## Project Structure

```
Research/
├── 11 - Assigment/
│   ├── New Street Research - Take-Home Assignment.pdf
├── 12 - Inputs/
│   ├── sample_data.csv                         # Input dataset (121k rows)
│   └── data_generation.ipynb                   # Data generation notebook
├── 13 - Processing/
│   ├── exercise_v2.ipynb                       # Inline implementation (v2)
│   └── exercise_v3.ipynb                       # Modular implementation (v3)
├── 14 - Outputs/
│   ├── yoy_summary.csv                         # Generated summary table
│   ├── second_revision.md                      # v2 evaluation report
│   └── third_revision.md                       # v3 evaluation report
├── pyproject.toml
├── uv.lock
└── README.md
```

## Running

1. Ensure the input CSV is at `12 - Inputs/sample_data.csv`
   (or generate it using the `generate_sample_data()` function from the assignment)
2. Open and run all cells in `13 - Processing/exercise_v3.ipynb`
3. The year-over-year summary CSV will be written to `14 - Outputs/yoy_summary.csv`

## Output

- **`14 - Outputs/yoy_summary.csv`**: Year-over-year summary table with columns:
  `year`, `provider_id`, `upgrade_builds`, `edge_out_builds`, `total_builds`
- **Analysis Report**: Embedded in the final markdown cell of the notebook, addressing fastest-growing providers, growth type trends, and capacity constraint dynamics.

## Notebook Versions

Both `exercise_v2.ipynb` and `exercise_v3.ipynb` produce identical simulation results (12,190 total builds over 10 years, same per-provider and per-year breakdown). The only difference is code organization:

- **v2** keeps all logic inline within notebook cells.
- **v3** wraps the core business logic into six modular functions with docstrings (`classify_technology`, `identify_gigabit_capable`, `find_upgrade_candidates`, `find_edge_out_candidates`, `score_candidates`, `allocate_builds_for_year`), as requested by the assignment's "Modular functions with clear docstrings" requirement.
