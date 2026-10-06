# Antimicrobial Peptides × Biophysical Determinants × Protein Language Models

Code and data for the paper
**"Protein Language Models Encode the Order-Dependent Biophysical Determinants of Antimicrobial Activity: Interpretable Probing and Mechanism-Aware Peptide Design."**

*Status: under review at the Journal of Computational Chemistry (first revision).*

The analysis is a **single script** (`amino.py`), or the same steps as a notebook (`amino.ipynb`). It runs end to end: collects peptides from UniProtKB/Swiss-Prot, builds physicochemical and membrane-interaction features, trains classifiers, probes frozen ESM-2 embeddings layer by layer, performs a homology-aware evaluation, and optimizes new peptide sequences under a multi-oracle objective.

## How to run

**Google Colab (recommended — free GPU):**
1. Open `amino.ipynb` in Colab
2. `Runtime → Change runtime type → T4 GPU`
3. `Runtime → Run all`

**Locally:**
```bash
pip install -r requirements.txt
python amino.py
```

The script installs `transformers` and `cd-hit` by itself. A GPU is recommended (the ESM-2 step is slow on CPU), and `cd-hit` needs Linux.

### Reproducibility — read this first

**The notebook is self-contained.** The published benchmark — 1,724 peptides, 862 AMP and 862 non-AMP — is embedded inside `amino.ipynb` itself, compressed and checksummed. Upload the single notebook to Colab, run all cells, and you reproduce the paper: nothing is downloaded, no data file has to be present, and there is no way for a stale copy to be picked up. The first cell verifies an MD5 checksum and stops if the embedded data is corrupted. (`amp_dataset.csv` is also shipped here for convenience and is written out by that cell, but the notebook does not depend on it.)

The benchmark was originally built by querying UniProtKB/Swiss-Prot live, and that code is preserved in the two cells after the embedded one, switched off by `REBUILD_FROM_UNIPROT = False`. Setting it to `True` rebuilds from the live API — but Swiss-Prot keeps growing, so a rebuild today returns a larger set (1,744 peptides as of October 2026) and every downstream value shifts: ROC-AUC, probe R², cluster counts and the designed sequences. The conclusions hold either way, but the numbers no longer match the paper.

The clustering step is checked the same way: it prints whether CD-HIT reproduced the 351 clusters the paper reports.

**What reproduces exactly, and what drifts.** A clean run on Colab (October 2026) reproduced the dataset, the clustering (351 clusters), every descriptor-based result, the full layer-wise probing profile (all 31 values identical), the leakage gap, and every designed peptide including all of Table 2. The only quantities that moved are the eight derived from the mean-pooled final ESM-2 layer, which depend on the `transformers` release:

| Quantity | Paper | Oct 2026 run |
|---|---|---|
| ESM-2 + logistic regression (ROC-AUC) | 0.948 | 0.951 |
| ESM-2 + random forest (ROC-AUC) | 0.960 | 0.962 |
| Probe R², hydrophobic moment | 0.720 | 0.726 |
| Probe R², net charge | 0.958 | 0.967 |
| Probe R², cation-π | 0.760 | 0.783 |
| Probe R², GRAVY / WW-interface / H-bond donors | 0.988 / 0.977 / 0.990 | 0.991 / 0.981 / 0.992 |

The shifts are at most 0.023 and all in the same direction; the composition baselines are unchanged, so the central contrast (ESM-2 0.72 vs composition 0.33 on the hydrophobic moment) is if anything slightly wider. The layer-wise values are untouched because they are read from `hidden_states` rather than `last_hidden_state`. The notebook prints the versions of torch, transformers, scikit-learn and numpy so the environment of any run is on the record.

## Main results

| | |
|---|---|
| Benchmark | 1,724 Swiss-Prot peptides (862 AMP / 862 non-AMP), 351 CD-HIT clusters at 40 % identity |
| Classification | ROC-AUC 0.962 (random split) → 0.934 (homology-aware) — leakage gap **0.027** |
| ESM-2 probing | hydrophobic moment: R² **0.720** (ESM-2) vs **0.330** (composition) |
| Layer-wise | amphipathicity peaks at **layer 6 of 30** (R² 0.766) |
| Sequence optimization | inter-oracle agreement **0.145 → 0.545**; **89** robust novel candidates |
| Homology ≠ biophysics | 58.8 % of multi-member clusters hold both AMP and non-AMP; 58.7 % of amphipathicity variance sits *within* clusters |

## What the code produces

Running the notebook top to bottom reproduces every figure and table of the paper:

- **Figures 1–5** — homology-aware benchmark, probing, layer-wise emergence, class separation, multi-oracle optimization
- **Figures S1–S5** — length and descriptor distributions, held-out ROC and confusion matrix, feature importances, helical wheels
- **Figure S6, Table S3** — cluster-level test of whether 40 % sequence identity implies biophysical similarity *(added in revision)*
- **Figure S7, Table S4** — why two individually accurate oracles agree only moderately on optimized sequences *(added in revision)*
- `amp_dataset.csv`, `seqs.fasta`, `clust.clstr`, `robust_designs.csv`

The last three cells of the notebook are the revision analyses; they need only NumPy, pandas, scikit-learn and Matplotlib, so they run on CPU in under a minute.

## Files

```
amino.py             full pipeline as one script
amino.ipynb          same pipeline as a notebook, with outputs and figures
amp_dataset.csv      curated benchmark (also regenerated by the code)
seqs.fasta           sequences in FASTA form, input to CD-HIT
clust.clstr          CD-HIT clustering at 40 % identity
robust_designs.csv   designed peptides endorsed by both oracles
requirements.txt     Python dependencies
figures/             all figures at 300 dpi (PNG + PDF)
```

## Scope and limitations

Class labels come from **UniProt keyword annotation**, not from measured activities, so absolute performance is illustrative of the methodology rather than competitive with activity-trained predictors. All designs are validated computationally only. A definitive study needs activity-annotated data (DBAASP, DRAMP 3.0) with matched negatives, alignment-based partitioning (GraphPart) plus an external test set, end-to-end deep baselines on identical splits, and ultimately wet-lab MIC and hemolysis assays.

## Citation

Khairbek, A. A.; Alzahrani, A. Y. A.; Ben Moussa, S.; Thomas, R. *Protein Language Models Encode the Order-Dependent Biophysical Determinants of Antimicrobial Activity: Interpretable Probing and Mechanism-Aware Peptide Design.* Under review, J. Comput. Chem., 2026.

## License

MIT
