# pathogenicity-gates CLI Reference

The `pathogenicity-gates` command provides access to the Gate & Channel
pathogenicity prediction framework from the command line.

## Installation

```bash
pip install -e .
# Verify installation
pathogenicity-gates --version
```

## Subcommands

### `predict` — Single variant prediction

Predict pathogenicity for a single missense variant.

```bash
# Basic
pathogenicity-gates predict --protein p53 --mutation R175H

# Alternative mutation notation (space-separated)
pathogenicity-gates predict --protein p53 --mutation 175 R H

# Custom annotation
pathogenicity-gates predict --annotation ./my_protein.yaml --mutation G12V

# JSON output (pipeline-friendly)
pathogenicity-gates predict --protein p53 --mutation R175H --format json
```

Options:

| Flag | Description |
|------|-------------|
| `--protein NAME` | Bundled protein: p53, kras, tdp43, brca1 |
| `--annotation PATH` | Custom annotation.yaml |
| `--mutation MUT` / `-m` | Variant (e.g. `R175H` or `175 R H`) |
| `--format FMT` | Output: `compact` (default), `table`, `json` |
| `--mode MODE` | Prediction mode: `channels` (default), `legacy` |

### `predict-batch` — Batch prediction from file

```bash
# CSV input, JSON output
pathogenicity-gates predict-batch --protein p53 \
    --input variants.csv --output results.json

# JSON input, CSV output
pathogenicity-gates predict-batch --protein p53 \
    --input variants.json --output results.csv --format csv

# From stdin
cat variants.csv | pathogenicity-gates predict-batch --protein p53 --input -
```

Input CSV format (header row required):
```csv
position,wt,mt
175,R,H
248,R,W
```

Input JSON format:
```json
[
  {"position": 175, "wt": "R", "mt": "H"},
  {"position": 248, "wt": "R", "mt": "W"}
]
```

Column aliases supported: `pos`/`position`/`residue`, `wt`/`wildtype`/`ref`,
`mt`/`mutant`/`alt`.

Output formats: `json` (default), `table`, `compact`, `csv`.

### `explain` — Per-channel diagnostics

```bash
pathogenicity-gates explain --protein p53 --mutation R175H
```

Shows:
- Prediction + regime
- Per-channel states with plain-English descriptions
- Residue context (burial, SS, neighbors)

Flags:
- `--verbose`: Dump full residue context as JSON
- `--channels-only`: Suppress residue context section

### `list-proteins` — Show bundled protein annotations

```bash
pathogenicity-gates list-proteins
pathogenicity-gates list-proteins --format json     # scripting
pathogenicity-gates list-proteins --format compact  # names only
```

## Exit codes

| Code | Meaning |
|------|---------|
| 0    | Success |
| 1    | Runtime error (prediction/load failure) |
| 2    | Invalid arguments (parse error, missing flag) |

## Examples

### Example 1: p53 hotspot screening

```bash
cat > hotspots.csv <<EOF
position,wt,mt
175,R,H
176,C,F
179,H,R
248,R,W
273,R,H
EOF

pathogenicity-gates predict-batch \
    --protein p53 --input hotspots.csv --format table
```

### Example 2: CI pipeline

```bash
#!/bin/bash
result=$(pathogenicity-gates predict --protein p53 --mutation R175H --format json)
n_closed=$(echo "$result" | python -c "import sys,json;print(json.load(sys.stdin)['n_closed'])")
if [ "$n_closed" -ge 1 ]; then
    echo "Pathogenic variant detected"
    exit 1
fi
```

### Example 3: Debugging a prediction

```bash
pathogenicity-gates explain --protein p53 --mutation P47S --verbose
```
