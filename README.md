# 🧬 SCRIBE Gene Regulatory Network Inference Pipeline (Nextflow)

This project implements **SCRIBE**-style gene regulatory network (GRN) inference for single-cell RNA-seq data using a **Nextflow DSL2 pipeline**.  
It accepts `.h5ad` as input and produces a ranked list of regulator→target edges plus a normalized adjacency.

The pipeline follows the spirit of the [Beeline GRN Benchmarking Suite](https://github.com/Murali-group/Beeline) and provides:

- a native **R backend** (if `Rscript` is available), and
- a built-in **Python fallback** (no R required).

---

## 📁 Project Structure

```

scribe\_nextflow\_v4/
├── main.nf                         # Main Nextflow pipeline (R backend + Python fallback)
├── nextflow\.config                 # Resources (memory/CPUs) and runtime settings
├── README.md                       # This file
├── modules/
│   └── grn/
│       └── scribe/
│           ├── h5ad\_to\_scribe\_inputs.py  # Read .h5ad -> expr.csv / genes.txt / pseudotime.csv
│           ├── run\_scribe.R              # SCRIBE core (preferred if Rscript exists)
│           ├── averageA.R                # Average/normalize edge weights
│           └── aggregate\_edges.py        # Beeline-style final ranking
└── filtered\_placeholder.h5ad        # (example input; optional)

```

---

## 🔧 Setup Instructions

### 🧬 Prerequisites

- **Nextflow** ≥ 25.x
- **Python 3.9+** with:
  - `anndata`, `h5py`, `numpy`, `pandas`, `scipy`
- **(Optional) R** with `Rscript` (the pipeline auto-falls back to Python if missing)

> 💡 On WSL, keep the Nextflow work directory on Linux FS for reliability, e.g. `-w "$HOME/nf_work"`.

---

## 🚀 Run the SCRIBE Pipeline

From the project root:

```bash
# If your .h5ad is in this folder:
nextflow run main.nf \
  --in_h5ad "$PWD/filtered_placeholder.h5ad" \
  --outdir results_scribe \
  -w "$HOME/nf_work"
```

Or with an absolute path to any `.h5ad`:

```bash
nextflow run main.nf \
  --in_h5ad /full/path/to/your_data.h5ad \
  --outdir results_scribe \
  -w "$HOME/nf_work"
```

**Result folders:**

```
results_scribe/
├─ inputs/         # expr.csv, genes.txt, pseudotime.csv
├─ results/        # edges.tsv, A_averaged.tsv
└─ final/          # scribe_ranked_edges.tsv
```

---

## 📤 Output Files

- **`results/edges.tsv`**
  Raw SCRIBE edges with columns:
  `regulator  target  score`
  Higher `score` ⇒ stronger inferred regulation.

- **`results/A_averaged.tsv`**
  Normalized per-pair weights with columns:
  `regulator  target  weight` (0–1)
  Useful as a final adjacency for network tools (e.g., Cytoscape).

- **`final/scribe_ranked_edges.tsv`**
  Beeline-style final ranked edges aggregated for downstream benchmarking.

> 🔎 Rule of thumb:
> Use `edges.tsv` for full ranking/analysis; use `A_averaged.tsv` for a clean, one-score-per-edge network.

---

## ⚙️ Configuration (resources)

All resource settings are in **`nextflow.config`**. A safe default for your machine:

```groovy
process {
  executor = 'local'
  errorStrategy = 'retry'
  maxRetries = 1

  withName:run_scribe            { time = '24h'; memory = '13 GB' }  // tuned to fit 15.5 GB available
  withName:convert_h5ad_to_scribe{ memory = '4 GB' }
  withName:aggregate_edges       { memory = '2 GB' }
}
```

You can override at runtime if needed:

```bash
nextflow run main.nf --in_h5ad ... -process.memory '13 GB'
```

---

## 🧪 Backend Behavior

- If **`Rscript` is available**, the pipeline runs:

  - `modules/grn/scribe/run_scribe.R`
  - `modules/grn/scribe/averageA.R`

- If **`Rscript` is NOT found**, the pipeline uses an **inline Python fallback** that computes SCRIBE-like scores and writes the same two files:

  - `results/edges.tsv`
  - `results/A_averaged.tsv`

This keeps the pipeline runnable even on minimal environments.

---

## 📊 (Optional) Quick Visualization

Load `A_averaged.tsv` into Cytoscape / Gephi / igraph for GRN visualization.
Edge attribute to use: **`weight`**.

---

## 🛠 Troubleshooting

- **“No .h5ad found”**
  Ensure `--in_h5ad` points to a real file. Prefer absolute paths or `"$PWD/…"`.

- **R not installed**
  No action needed — the pipeline will **automatically** use the Python fallback.

- **Memory exceeded**
  Lower memory in `nextflow.config` for `withName:run_scribe` (e.g., `'13 GB'`).

- **WSL symlink quirks**
  We force **copy** staging for inputs and recommend `-w "$HOME/nf_work"` to avoid `/mnt/c` symlink issues.

---

## 📚 References

- **Qiu, Ke, et al.**
  _Inferring Causal Gene Regulatory Networks from Time-Stamped Single-Cell Transcriptome Data._
  (SCRIBE methodology)

- **Beeline Benchmark Suite**
  [https://github.com/Murali-group/Beeline](https://github.com/Murali-group/Beeline)

---

## 👨‍💻 Author

**Prosenjit Chowdhury**
M.Sc. Artificial Intelligence – FAU Erlangen-Nürnberg
Working Student @ SAP ERP PCX
🌍 Erlangen, Germany
🔗 GitHub: [@prosenjit-chowdhury](https://github.com/prosenjit-chowdhury)

---

## 🧠 License

This project is licensed under the MIT License. See [`LICENSE`](LICENSE) for details.

````

---

### Suggested `.gitignore` (recap)

```bash
cat > .gitignore << 'EOF'
# Nextflow runtime
work/
.nextflow*
.cache/
*timeline*.html
*trace*.txt
*report*.html
dag.png

# Outputs (code-only repo)
results*/
**/inputs/
**/results/
**/final/

# Data files
*.h5ad
*.h5
*.loom
*.mtx
*.npz
*.npy
*.rds

# Python
__pycache__/
*.py[cod]
*.egg-info/
.venv/
venv/
.ipynb_checkpoints/

# R
.Rhistory
.Rproj.user/
.RData
.Ruserdata

# OS / Editor
.DS_Store
Thumbs.db
*.swp
*.swo
.vscode/
.idea/

# Archives
*.zip
*.tar
*.tar.gz
*.tgz
EOF
````

Want me to also generate a tiny `LICENSE` (MIT) and a `CHANGELOG.md` skeleton to round out the repo?
