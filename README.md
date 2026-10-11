# ALS Causal Knowledge Graph: Community Structure and Multi-Biomarker Hypothesis Testing

Analysis code for the paper *Automated Causal Knowledge Graph Construction for ALS Biomarker
Panel Design: A Community Detection Framework on CSF Proteomics Literature*.

The notebook `ALS_CKG_Hypothesis_Testing.ipynb` loads a pre-built ALS Causal Knowledge Graph (CKG)
and reproduces the community-detection analysis and the four hypothesis tests reported in the paper.
It consumes only the final graph. It does not include or depend on the upstream extraction and
validation pipeline (GROBID, the language models, or the TCVS validator).

## What the notebook tests

| Notebook | Paper | Question |
|----------|-------|----------|
| Hypothesis I   | H1 | Are biomarker communities more densely interconnected with each other than with genetic communities? (cross-community edge density) |
| Hypothesis II  | H2 | Do neurofilament (NfL) nodes occupy central, hub-like positions? (betweenness centrality) |
| Hypothesis III | H3 | Is the Louvain community structure stable across random seeds and resolution values? (Normalized Mutual Information) |
| Hypothesis IV  | H4 | Do the top-degree CSF-community nodes correspond to clinically established ALS biomarkers? (precision at top-K) |

The mapping is one to one with the paper. An earlier exploratory hypothesis about the progression
community is not part of the paper and is not included here.

## Repository contents

```
ALS_CKG_Hypothesis_Testing.ipynb   the analysis notebook
requirements.txt                   pinned Python dependencies
prepare_release_graph.py           OPTIONAL helper to build a minimal, shareable graph pickle
README.md                          this file
ckg_louvain.pkl                    the graph input (you add this; see "Data")
```

## Data

### The one file you need to provide

The notebook needs exactly **one** pickle file: the final CKG as a NetworkX graph. By default it
looks for `ckg_louvain.pkl` in the working directory. Edit `PKL_GRAPH` in Step 1 if your file name or
location differs.

You do **not** need to release any intermediate graphs (for example earlier "raw" versions at
different edge thresholds). Only the final graph used for the reported analysis is required.

### What the graph pickle must contain

A pickled `networkx` graph (`Graph` or `DiGraph`) where:

- **Nodes** are the term strings (for example `NfL`, `C9orf72`, `APOE`).
- **Node attribute `community`** (integer) is recommended. If present, the notebook uses it directly
  and reproduces the exact communities. If absent, the notebook re-runs Louvain (seed 42, resolution
  1.0) to recover them.
- **Node attribute `category`** is optional and not required.
- **Edge attribute `weight`** (float) is used for weighting. The notebook also accepts any of
  `w_norm`, `edge_confidence`, `confidence`, or `tcvs` and auto-detects which is present.

Nothing else is read by the analysis.

### Protecting your other work (recommended)

The analysis never uses node embeddings, source provenance, raw text, or the TCVS component scores.
You can strip all of those before publishing, which keeps the file small and keeps that work private.
`prepare_release_graph.py` does this for you: it loads your full graph, keeps only node names, the
`community` (and optional `category`) attribute, and a single numeric `weight` per edge, and writes a
clean `ckg_louvain.pkl`. Run it once on the machine that holds your full graph:

```bash
python prepare_release_graph.py
```

Tip: add your full graph and the generated figures and CSVs to `.gitignore` so only the minimal,
sanitized pickle is committed.

## Environment setup

The notebook was developed on Python 3.10 and runs entirely on CPU. Use either option below.

### Option A: conda

```bash
conda create -n als-ckg python=3.10 -y
conda activate als-ckg
pip install -r requirements.txt
```

### Option B: venv + pip

```bash
python3.10 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Notes:

- The community-detection package installs as `python-louvain` but imports as `community`. This is
  already handled in `requirements.txt`. Do not install an unrelated PyPI package also named
  `community`.
- `matplotlib` is pinned below 3.9 because the notebook uses `get_cmap(name, N)`. The notebook also
  includes a fallback for matplotlib 3.9 and later, so it will still work if you use a newer version.
- `numpy` is pinned below 2.0 for compatibility with the pinned data-science stack.

## How to run

1. Place your graph pickle in the project folder (or run `prepare_release_graph.py` to create it).
2. Launch Jupyter:
   ```bash
   jupyter notebook ALS_CKG_Hypothesis_Testing.ipynb
   ```
3. In **Step 0**, either run the install cell or skip it if you already installed `requirements.txt`.
4. In **Step 1**, set `PKL_GRAPH` to your file path.
5. Run all cells top to bottom (Cell menu, Run All).

Runtime is a few minutes on a typical laptop CPU. Betweenness centrality uses a 500-sample
approximation for graphs larger than 2,000 nodes, which keeps it fast.

### Outputs (written to the working directory)

Figures: `fig_community_graph.png`, `fig_h1_cross_community.png`, `fig_h2_betweenness.png`,
`fig_h3_stability.png`, `fig_h4_csf_precision.png`, `fig_overview_panel.png`.

CSVs: `centrality_results.csv`, `cross_community_edges.csv`, `csf_community_nodes.csv`,
`stability_seed_results.csv`, `stability_resolution_results.csv`.

## Reproducibility notes

- Reference partition: seed 42, resolution 1.0.
- Seed stability: seeds 0, 1, 7, 13, 99 at resolution 1.0.
- Resolution stability: resolutions 0.6, 0.8, 1.0, 1.2, 1.5 at seed 42.
- Louvain uses the undirected projection of the graph with edge weight as the weighting attribute.
- Community labels in the notebook are auto-derived from each community's top-degree nodes. To match
  the exact labels in the paper's Table A.1, fill in the `COMMUNITY_LABELS` dictionary in Step 1e.
- Community-to-supertype grouping (biomarker, genetic, clinical, other), used in Hypothesis I, is
  assigned from label keywords. Adjust the keyword lists or `COMMUNITY_LABELS` if your terms differ.

## Caveats

The CKG was built from 35 ALS CSF proteomics papers. This is a pilot-scale corpus. Community sizes are
small and several tests have limited statistical power, so results should be read as directional and
hypothesis-generating rather than confirmatory. See the paper for the full discussion.

## Citation

If you use this code, please cite the paper. (Add the final citation and DOI here once available.)

## License

Add a license of your choice. A common pairing is MIT for the code and CC BY 4.0 for the released
graph data. State clearly which license applies to the notebook and which applies to `ckg_louvain.pkl`.
