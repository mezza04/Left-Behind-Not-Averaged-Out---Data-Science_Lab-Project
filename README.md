# Left Behind, Not Averaged Out

**A Poset-Based Identification of Multidimensional Youth Deprivation in 24 European Countries**

Which European countries are *unambiguously* leaving their young people behind? This project applies **partial order set (poset) theory**, a non-compensatory alternative to composite indices, to five youth-deprivation indicators from OECD, Eurostat and PISA data, and compares the result with a conventional equal-weight composite index.

> Project for the **Data Science Lab** course – University of Milano-Bicocca.
> Authors: Ilaria Anna Cesaratto, Chiara Mezzanzanica, Nicolò Nucci, Andrea Porro.

📄 **Report:** [`DSLab_project-Left_Behind__Not_Averaged_Out.pdf`](./DSLab_project-Left_Behind__Not_Averaged_Out.pdf)
💻 **Full pipeline (R Markdown):** [`Left_Behind__Not_Averaged_Out.Rmd`](./Left_Behind__Not_Averaged_Out.Rmd)

---

## Motivation

Composite indices average several indicators, so a good score on one dimension can compensate for a severe weakness on another. Two countries with very different youth problems (for example high unemployment but low poverty, versus mediocre results on both) can end up with the same score. A poset only ranks a country as more deprived than another if it is at least as deprived on **every** dimension, and leaves the remaining profiles **incomparable** instead of forcing them into a single number.

## Research questions

1. Which countries are unambiguously "left behind", and which are not, regardless of how the five dimensions are combined?
2. Which countries are classified substantially differently by a composite index and by the poset, and why?
3. How robust are the results to methodological choices (indicators, thresholds, number of grade levels), and which single dimension weighs most for the hard-to-classify countries?

## Data

| Indicator | Source | Notes |
|-----------|--------|-------|
| Youth monetary poverty (ages 18–25) | OECD SDG database (Goal 1); Eurostat `ilc_li02` as fallback | |
| NEET rate | OECD SDG database (Goal 8); Eurostat `edat_lfse_20` as fallback | Ages 15–24 |
| Youth unemployment | OECD SDG database (Goal 8); Eurostat `une_rt_a` as fallback | |
| Low math skills | PISA 2022 (via OECD SDG database, Goal 4) | |
| Low upper-secondary completion | OECD SDG database (Goal 4); Eurostat `edat_lfse_14` as fallback | Proxy for early leaving from education |
| ~~Low reading skills (PISA)~~ | OECD SDG database | Dropped: Spearman ρ = 0.67 with math skills |
| ~~Lack of a bank account (Global Findex 2021)~~ | World Bank | Dropped: nearly saturated across Europe, does not discriminate |

- **Sample:** 24 European OECD countries.
- **Access:** data are downloaded automatically by the pipeline from the OECD SDMX API (rate-limited, with retries), with Eurostat used only for permanent OECD gaps. Only four cells needed the Eurostat fallback, all on the completion indicator (Czechia, Ireland, Netherlands, Slovakia). No value is estimated or interpolated.
- **Known limitations:** reference years are not perfectly aligned (labour data 2023–24, PISA 2022, poverty 2021–23), and the completion indicator is a proxy for the EU "early leavers" measure.
- All indicators are oriented so that higher values always mean more deprivation.

## Methodology

1. **Cleaning and orientation** – three-step missing-value search (OECD 2019–2024, OECD 2015–2024, Eurostat), pre-registered and post-hoc indicator removals clearly labelled as such.
2. **Ordinal discretisation** into three grades (0 = low, 1 = medium, 2 = high deprivation). Every cut-point is labelled as an *official EU target*, an *empirical anchor* or a *normative convention*.
   - **Schema A** (policy-anchored, main analysis), **Schema B** (terciles) and **Schema C** (quintiles) are compared.
3. **Poset construction** – country profiles are ordered by component-wise dominance; a threshold profile τ = (1, 1, 1, 1, 1) ("medium" on every dimension) is added as an extra node.
4. **Identification degree and severity** – the identification degree is the share of linear extensions in which a country ranks above τ; severity is the mean normalised rank gap above τ. Both are estimated by uniformly sampling 20,000 linear extensions (Bubley–Dyer chain).
5. **Cross-validation of two implementations** – a from-scratch R implementation and the reference [`parsec`](https://cran.r-project.org/package=parsec) package, both validated first on a toy poset with hand-computable results.
6. **Composite-index benchmark** – equal-weight average of the five min–max normalised indicators, compared with the poset average rank via Spearman correlation.
7. **Robustness checks** – indicator jackknife, country jackknife, alternative discretisation schemes, permutation test (1,000 replications), and a "blocking dimension" analysis.

## Key results

- **Five countries are certainly deprived** (identification degree = 1): Spain, Italy, Austria, Lithuania, United Kingdom.
- **Nine are certainly not deprived** (identification degree = 0): Belgium, Czechia, France, Ireland, Latvia, Netherlands, Poland, Slovenia, Switzerland.
- **Ten remain ambiguous**, from Estonia (0.474) to Norway (0.947): a definitive classification is possible for only 14 of 24 countries.
- **Severity separates the certain group:** Spain (0.858) and Italy (0.771) are far more severely deprived than Austria, Lithuania and the UK (0.385).
- **Poset vs composite index:** rank correlation ρ = 0.85. Nine countries shift by at least three positions; for example Austria and the UK are penalised by the poset (one severe NEET weakness averaged away), while France is ranked mid-table by the composite index but is certainly not deprived under the poset.
- **Implementations agree closely:** Spearman ρ = 0.9861 between the from-scratch and `parsec` results (maximum absolute difference 0.055).
- **Robustness:** the extreme groups are stable; the intermediate group is sensitive to which indicators are included. NEET is the dimension that most often keeps a country below certain classification (13 of 19 non-certain countries).
- **Structure of the poset:** 56.2% of pairs of distinct profiles are mutually incomparable, markedly fewer than expected under independent indicators (permutation test, p ≈ 0.003).

## Repository structure

```
.
├── README.md
├── Left_Behind__Not_Averaged_Out.Rmd                     # complete, executable pipeline
└── DSLab_project-Left_Behind__Not_Averaged_Out.pdf       # final report
```

Running the pipeline also generates the following files in the working directory (they are not stored in this repository): `idea5_matrix_oriented_R.csv`, `idea5_grades_A_R.csv`, `idea5_grades_B_R.csv`, `idea5_jackknife_idn_R.csv` and `idea5_summary_R.json`.

## How to reproduce

1. Clone the repository:
   ```bash
   git clone https://github.com/mezza04/Left-Behind-Not-Averaged-Out---Data-Science_Lab-Project.git
   cd Left-Behind-Not-Averaged-Out---Data-Science_Lab-Project
   ```
2. Install the required R packages:
   ```r
   install.packages(c("httr", "readr", "dplyr", "tidyr", "purrr",
                      "corrplot", "pheatmap", "ggplot2",
                      "parsec", "jsonlite", "eurostat", "rmarkdown"))
   ```
3. Knit the pipeline (an internet connection is required to download the data):
   ```r
   rmarkdown::render("Left_Behind__Not_Averaged_Out.Rmd")
   ```

**Notes**
- The pipeline downloads the data live from the OECD and Eurostat APIs, so it needs no local data files. The OECD API is rate-limited (20 requests per minute); the code includes retries and pauses.
- Both sources may revise their series over time, so later runs can differ slightly from the figures reported here. Sampling uses fixed seeds for reproducibility.
- The generated CSV and JSON files are written to the working directory.

## Tools

R · parsec · tidyverse (readr, dplyr, tidyr, purrr, ggplot2) · corrplot · pheatmap · httr · jsonlite · eurostat.
AI tools were used to support code development; all methodological choices, data checks and final content were reviewed by, and are the responsibility of, the authors.

## Data sources and licences

- OECD SDG database (SDMX API), Eurostat (`edat_lfse_14`, `edat_lfse_20`, `une_rt_a`, `ilc_li02`) and PISA 2022 are used according to their respective terms of reuse; please cite the original providers.
- See the report for the full reference list.

## References

- Fattore, M. (2016). *Partially ordered sets and the measurement of multidimensional ordinal deprivation.* Social Indicators Research, 128, 835–858.
- Fattore, M., & Arcagni, A. (2014). *PARSEC: An R package for poset-based evaluation of multidimensional poverty.*
- Bubley, R., & Dyer, M. (1999). *Faster random generation of linear extensions.* Discrete Mathematics, 201, 81–88.
