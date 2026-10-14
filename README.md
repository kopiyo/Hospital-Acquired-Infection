# Environmental colonization pressure and prior antibiotic exposure as risk factors for hospital-acquired infection

Analysis code for a matched case-control study of the PhysioNet *Predictors of Hospital-Onset Infection* dataset. The study compares two routes of acquisition: exposure to **colonization pressure** from other patients on the ward (the environmental route), and **prior antibiotic exposure** in the patient themselves (the patient route).

> **Manuscript:** *Comparative Analysis of Environmental Colonization Pressure and Prior Antibiotic Exposure as Risk Factors for Hospital-Acquired Infections: A Matched Case-Control Study*
> Atieno Diana Opiyo and Geofrey Nyakairu. Under review at the *Journal of Hospital Infection*.

## What the study asks

1. Is colonization pressure from each of nine pathogen groups independently associated with acquisition, after adjusting for age, comorbidity and, where applicable, surgery?
2. Is prior exposure to each of 14 antibiotic classes independently associated with acquisition?
3. Once both exposures are placed on the same scale (per 1 SD), which association is larger, and does it hold up when cases and controls are observed for comparable lengths of time?

The outcome is **microbiological acquisition** (a positive culture or surveillance result for one of 18 target pathogen runs), not an adjudicated clinical infection.

## Data

| | |
|---|---|
| Source | PhysioNet, *Predictors of Hospital Onset Infection: A Matched Retrospective Cohort Dataset* (v1.0.0), derived from Mass General Brigham data (Kanjilal S, Sagers L, Wei Z, et al.) |
| File | `HO_infxn_analysis.csv` (about 36 MB) |
| Size | 120,279 records: 37,955 cases and 82,324 controls across 18 pathogen runs |

**The data are not included in this repository.** Download the file from PhysioNet and follow its data-use terms, then place it next to the notebook.

### Analysed samples

| Cohort | Matched sets | Cases | Controls | Records |
|---|---:|---:|---:|---:|
| Environmental (colonization pressure) | 17,895 | 17,895 | 35,041 | 52,936 |
| Patient (antibiotic exposure) | 20,055 | 20,055 | 47,273 | 67,328 |

Matching is variable-ratio (1 case to up to 3 controls). The patient cohort loses 10 records at the complete-case step and 5 more through cascade exclusion (a set that loses its case also loses its controls). The notebook reports every stage in its participant-flow table.

## Methods in brief

- **Conditional (matched) logistic regression** (`statsmodels` `ConditionalLogit`), with one stratum per matched set. Predictors with no within-set variation, such as matching variables, are dropped by the estimator and reported.
- **Colonization pressure** is expressed per 1 SD. **Antibiotics** are expressed per treatment course, and also per 1 SD for the cross-cohort comparison.
- **Observation-time bias:** models adjusted for matched observation window, plus a **landmark analysis** restricted to matched sets in which every member was observed for at least 3, 5 or 7 days.
- **Diagnostics:** variance inflation factors, a spline test of linearity for age, standardized mean differences (SMD) and a Love plot.
- **Interactions** of colonization pressure with age, surgery and antibiotic exposure (36 terms), reported with Bonferroni and Benjamini-Hochberg corrections.
- **Sensitivity analyses:** individual Elixhauser comorbidity indicators in place of the summary index, drug-resistant and drug-susceptible pathogen subsets, leave-one-pathogen-out, and pathogen-specific models.

## Main findings

- Seven of nine colonization-pressure measures were independently associated with acquisition (four inversely, three positively). Six of 14 antibiotic classes were associated with acquisition, all inversely.
- On a common per-SD scale, the strongest environmental association (drug-susceptible Enterobacterales colonization pressure, aOR 0.87, 95% CI 0.85 to 0.90) was larger than the strongest antibiotic association (fluoroquinolones, aOR 0.95, 95% CI 0.93 to 0.97).
- The environmental association was stable in landmark analyses at 3, 5 and 7 days. The antibiotic associations attenuated towards the null.
- Of 36 interaction terms, four reached nominal significance and none survived multiplicity correction.

The inverse associations should be read with care. They are pooled across pathogens, and the outcome is detection rather than clinical infection. The manuscript discusses this in detail.

## Repository contents

```
.
├── HAI_CP_vs_Antibiotic_REVISED.ipynb   # the full analysis, end to end
├── requirements.txt
├── README.md
└── hai_outputs/                         # created when the notebook runs
    ├── table*.csv                       # one CSV per manuscript table
    ├── fig*.png                         # figures (study flow, Love plot, forest plots)
    ├── findings.txt                     # plain-text summary of the fitted results
    └── software_versions.json           # exact package versions used
```

## Running it

```bash
git clone <this-repository>
cd <this-repository>
python -m venv .venv && source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
# place HO_infxn_analysis.csv in this folder, then:
jupyter notebook HAI_CP_vs_Antibiotic_REVISED.ipynb
```

To run it headless:

```bash
jupyter nbconvert --to notebook --execute HAI_CP_vs_Antibiotic_REVISED.ipynb \
    --output HAI_CP_vs_Antibiotic_executed.ipynb --ExecutePreprocessor.timeout=-1
```

The full run takes longer than a quick check because it fits many conditional models on about 120,000 records. **Set `FAST_MODE = True` in the configuration cell for a one to two minute test run** on a subsample of matched sets. It must be `False` for the reported results.

The notebook looks for the data in a few places: the working directory, `/mnt/data/`, and a Google Colab Drive path. Edit `DATA_PATH_CANDIDATES` in section 1 if yours is elsewhere.

### Configuration (section 1 of the notebook)

| Setting | Default | Meaning |
|---|---|---|
| `STANDARDIZE_CP` | `True` | Colonization-pressure odds ratios per 1 SD |
| `STANDARDIZE_ABX` | `True` | Antibiotic odds ratios per 1 SD, for the common-scale comparison |
| `LANDMARK_DAYS` | `[3, 5, 7]` | Landmark thresholds, in days |
| `FAST_MODE` | `False` | Subsample matched sets for a quick test |
| `RANDOM_SEED` | `42` | Seed used for the subsample |
| `OUTPUT_DIR` | `hai_outputs` | Where tables and figures are written |

## Reproducibility

Results in the manuscript were produced with Python 3.11, `numpy` 2.4, `pandas` 3.0, `statsmodels` 0.15, `scipy` 1.17, `matplotlib` 3.10 and `patsy` 1.0. The notebook writes the versions it actually used to `hai_outputs/software_versions.json`. Package versions are pinned loosely in `requirements.txt`, so exact values may differ slightly on other versions.

Every table in the manuscript was compared cell by cell against the CSVs this notebook produces.

## Citation

If you use this code, please cite the manuscript (above) and the dataset. A permanent archive of this code will be deposited on Zenodo:

> Opiyo AD, Nyakairu G. Code for: environmental colonization pressure and prior antibiotic exposure as risk factors for hospital-acquired infection. Zenodo. DOI: `10.5281/zenodo.PENDING`

## Licence

Code: [choose a licence, for example MIT]. The dataset has its own terms of use on PhysioNet and is not covered by this licence.

## Contact

Atieno Diana Opiyo (corresponding author), MS Data Science and Analytics, Grand Valley State University. opiyod@mail.gvsu.edu
