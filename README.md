# heat-threshold-transfer-abc — Proof of Concept

AI-augmented Approximate Bayesian Computation for transferring health-based
heat-health thresholds from California (data-rich) to data-sparse regions.

## What this is

A three-phase pipeline that transfers community-specific heat-health
thresholds from the **1,686 California ZIPs** covered by the source CA
index to ZIPs in other regions. The proposal mechanism inside ABC is a
**Large Language Model (LLM) used as a stochastic proposal distribution**,
calibrated on the relationship between community profiles and observed
California heat-health thresholds — so the LLM proposes California
analogs that empirically predict thresholds, rather than analogs that
merely "look similar".

The approach is **LLM-agnostic**. The proof-of-concept artifacts in this
repository were generated using a current commercial LLM as the proposal
mechanism; in the funded project, we will port the pipeline to **Google
Gemini and Gemma** and use Google.org accelerator support for prompt
engineering, fine-tuning, and scaling on Google Cloud. The validation
methodology, the calibrated CA threshold index, and the ABC framework are
all model-independent.


## Headline results

### Within-California validation — LA County (n = 289 ZIPs)

A leave-one-out style evaluation: predict each LA County ZIP's threshold
from the rest of California's calibrated index, then compare to the actual
California threshold.

| Metric                            | Value             |
|-----------------------------------|-------------------|
| Mean MAE vs CA ground truth       | 1.12 °F           |
| Median MAE                        | 0.75 °F           |
| Statewide-baseline MAE            | 3.74 °F           |
| Median improvement vs baseline    | 4.15×             |
| ZIPs beating statewide baseline   | 260 / 289 (90 %)  |
| ZIPs with MAE < 1 °F              | 180 / 289 (62 %)  |
| ZIPs with MAE < 0.5 °F            | 89 / 289 (31 %)   |


### Cross-state transfer — Portland vs. Las Vegas

Two metros chosen for contrasting climate / AC-penetration regimes test
whether the ABC transfer recovers the SES-mediated adaptive-capacity
mechanism documented in California.

|                                            | **Portland** (n = 67) | **Las Vegas** (n = 67) |
|--------------------------------------------|-----------------------|-------------------------|
| Cat 1 threshold (mean ± SD)                | 75.5 ± 2.3 °F         | 91.0 ± 2.4 °F           |
| Cat 4 threshold (mean ± SD)                | 84.4 ± 2.3 °F         | 98.8 ± 2.4 °F           |
| July avg daily max temp                    | 80.4 ± 2.9 °F         | 106.5 ± 1.6 °F          |
| AC penetration (across ZIPs)               | 45–85 %               | ~95–99 %                |
| Poverty × Cat 1, controlling for July max  | **−0.31\***           | +0.08 (n.s.)            |
| Poverty × Cat 4, controlling for July max  | **−0.42\*\*\***       | +0.06 (n.s.)            |

(\* p < 0.05; \*\*\* p < 0.001)

**Portland** recovers a significant SES gradient that **strengthens** with
heat-category — exactly what the California calibration predicts when AC is
the rate-limiting adaptation. **Las Vegas** shows a flat null across all
categories: when nearly everyone has AC, household income carries no
independent threshold signal. The category gradient in Portland is the
methodological evidence that the ABC transfer is recovering a real
mechanism, not noise.


## Files in this repository

Files are numerically prefixed so the GitHub file listing displays
them in narrative order: app screenshots first (California reference
ZIPs, then the two transfer-region ZIPs), then the threshold maps,
then the underlying numeric outputs.

| Path                               | Purpose                                                          |
|------------------------------------|------------------------------------------------------------------|
| `01_App_UCLA.png`                  | Heat-warning mobile-app view for a UCLA-area ZIP (California reference) |
| `02_App_SouthCentral.png`          | Mobile-app view for a South Central LA ZIP (California reference) |
| `03_App_Portland.png`              | Mobile-app view for a Portland ZIP (cross-state transfer)        |
| `04_App_LasVegas.png`              | Mobile-app view for a Las Vegas ZIP (cross-state transfer)       |
| `05_Map_Portland_thresholds.png`   | Per-ZIP Cat 1–4 threshold maps for the Portland transfer         |
| `06_Map_LasVegas_thresholds.png`   | Per-ZIP Cat 1–4 threshold maps for the Las Vegas transfer        |
| `07_Thresholds.csv`                | Predicted Cat 1–4 thresholds + posterior SD + 95 % CI for each Portland and Las Vegas ZIP (139 rows) |
| `LICENSE`                          | MIT license                                                      |
| `README.md`                        | This overview                                                    |

The full calibrated CA ZIP index, per-ZIP narrative profiles, ABC draw
logs, and the LA County within-CA validation outputs are not included in
this proof-of-concept snapshot but are available on request.

## Method in one paragraph

Phase 0 builds a **calibrated index**: the LLM writes free-form profiles
for all 1,686 California ZIPs from its prior knowledge, sees the actual
California heat-health thresholds, identifies which of its own narrative
features co-vary with thresholds, then rewrites all profiles to emphasize
the predictive features. Phase 1 generates an anonymized profile for a
target ZIP. Phase 2 runs **N independent ABC draws**: each draw is a
fresh-context LLM subagent that proposes a small set of California
analogs for the target. Pooling across draws yields a **posterior over
California heat-health thresholds** for the target ZIP. The LLM is the
proposal distribution; calibration on observed CA thresholds is what
makes the proposals discriminating.

## Reproducibility caveat

The pipeline uses an LLM as a stochastic proposal mechanism, so outputs
are not bit-identical across reruns — rerunning produces correlated but
not identical posteriors. The committed CSV and PNG artifacts in this
repository are the canonical numbers reported above.
