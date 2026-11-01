# Sensor Selection for Sparse PM₂.₅ Monitoring

## Balancing Forecast Accuracy and Warning Reliability

A STATS 402 Interdisciplinary Data Analysis project on choosing a small set of live air-quality monitors while preserving useful forecasts and pollution warnings across locations.

**Status:** Proposal and implementation planning. This repository currently documents the study design; implementation, reproducible experiment commands, and results will be added as the project progresses. No experimental performance claims are made at this stage.

**Repository:** [SeanWan514/pm25-sensor-selection](https://github.com/SeanWan514/pm25-sensor-selection)

## Research motivation

In this **Data + X** project, **X is environmental monitoring**: designing air-quality monitoring networks that support warnings about unusually high pollution. Limited monitoring capacity creates a practical question: which stations should provide live readings, and how reliably can forecasts serve the other locations?

Low average forecast error can conceal poor warning performance at individual stations. We will compare networks chosen for average accuracy with networks chosen to improve the lowest station-level detection rate, under explicit limits on false positives and accuracy loss.

The data-science contribution combines supervised time-series forecasting, exhaustive sensor selection, and uncertainty analysis. Our focus is the empirical trade-off between accuracy and warning reliability, rather than a claim to a new forecasting algorithm.

## Research questions

1. Do networks selected for low average forecast error also detect high-pollution hours reliably at every station?
2. Can maximizing minimum station-level recall improve the weakest warnings within a 5% allowance for mean absolute error and a common false-positive cap?
3. Do performance differences between the selected networks persist in a later held-out year and across seasons?

## Study setting and scope

We simulate retaining live PM₂.₅ measurements at **three of 12 Beijing stations** and forecasting concentrations **six hours ahead at all 12 stations**. Historical labels from every target station are available for development. At prediction time, only the selected stations supply current and lagged measurements.

This setting studies sparse live monitoring after historical calibration. It does not evaluate transfer to entirely unseen locations without historical training labels.

| Design choice | Planned specification |
| --- | --- |
| Live monitoring budget | Three stations |
| Forecast targets | All 12 historically observed stations |
| Pollutant and horizon | PM₂.₅, six hours ahead |
| Candidate networks | All 220 three-station subsets |
| Main model | One ridge-regression forecast per target station and candidate network |
| Inputs | Selected stations' readings at lags 0, 1, 3, 6, 12, and 24 hours; missingness indicators; cyclic hour and month features |
| Core comparison | Average-error selection versus selection for minimum station-level recall |
| Compute | CPU-based; GPU availability is not required |

Additional cities, pollutants, forecast horizons, and deep-learning architectures are outside the required experiment.

## Data

The primary source is the [UCI Beijing Multi-Site Air Quality dataset](https://doi.org/10.24432/C5RK5G), distributed under **CC BY 4.0**. The catalogue describes 420,768 station-hour records from 12 stations between March 1, 2013 and February 28, 2017. Records include timestamps, station identifiers, six pollutant measurements, and six meteorological variables. Missing values are recorded as `NA`; PM₂.₅ concentrations are measured in µg/m³.

The core experiment uses PM₂.₅ and calendar features. Data preparation will check timestamp alignment, duplicate records, missingness, units, and high-pollution counts. Missing targets will remain unfilled and will be excluded from scoring. At each target station, methods will be evaluated on the same observed outcomes.

[EPA AirData](https://aqs.epa.gov/aqsweb/airdata/download_files.html) and the [OpenAQ archive](https://docs.openaq.org/aws/about) are contingency sources. Substitution would require checking overlapping years, hourly coverage, instrument comparability, provider licenses, and co-located duplicates. They are not additional required datasets.

## Planned experimental protocol

### Chronological partitions

The following dates refer to dataset time, independently of the course-week schedule below. Each partition spans a full seasonal cycle.

| Purpose | Dataset interval |
| --- | --- |
| Model fitting and internal forward validation | March 2013–February 2014 |
| Warning-threshold calibration | March 2014–February 2015 |
| Network selection | March 2015–February 2016 |
| Final held-out evaluation | March 2016–February 2017 |

Forecast origins whose six-hour targets cross a partition boundary will be excluded from the earlier partition. Imputation, scaling, and regularization selection will use the appropriate training folds only. We will avoid future-aware interpolation and inputs from inactive stations.

### Warning definition

A high-pollution hour at each station is an observed concentration above that station's fitting-period **90th percentile**. For each network and target station, the warning threshold is the calibration-period 90th percentile of predicted concentrations among non-event hours; warnings are issued strictly above that threshold.

This targets a **10% calibration false-positive rate**. Later false-positive rates will be measured, and any violations will be reported. Station-relative percentiles describe local concentration extremes; they are not regulatory or medical safety limits.

### Selection objectives

For each network, we will calculate:

- **Mean MAE:** the unweighted mean of the 12 station-specific mean absolute errors.
- **Minimum recall:** the lowest station-specific fraction of high-pollution hours receiving warnings.
- **Maximum FPR:** the highest station-specific fraction of non-event hours receiving warnings.

Both primary selection rules use networks whose **maximum selection-period FPR is at most 10%**:

1. **Accuracy-oriented selection:** choose the feasible network with the lowest mean MAE.
2. **Reliability-oriented selection:** choose the feasible network with the highest minimum recall, allowing at most **5% greater mean MAE** than the accuracy-oriented choice.

The cap and allowance are prespecified operating choices. Ties will be resolved by lower mean MAE and then a fixed station ordering. If no network satisfies the FPR cap, we will report infeasibility and the full trade-off without relaxing the cap after examining test results.

Comparators will include the unconstrained MAE-optimal network, 20 random triples fixed in advance, a historical hour-of-day/month climatology, and a 12-station model as an information-rich reference.

### Evaluation and uncertainty

Models, warning thresholds, selected networks, and evaluation rules will be frozen before final test evaluation. We will report aggregate and station-level errors, recall, false-positive rates, event counts, and seasonal summaries.

For dependence-aware uncertainty analysis, we plan to adapt the moving-block bootstrap: resample seven-day blocks within seasons, use identical time blocks across methods and stations, and calculate approximate paired 95% intervals from 1,000 replicates. Sensitivity checks will use one- and fourteen-day blocks. These intervals are conditional on the fitted models and selections; they do not provide a general validity guarantee under distribution shift.

## Course schedule

Week 1 covers preparation. The six-week implementation period runs from **Week 2 through Week 7**, with approximately **10 combined person-hours per week**, totaling **60 implementation hours** including analysis, writing, and presentation.

| Course week | Milestone |
| --- | --- |
| Week 1 — preparation | Set the topic, review literature, prepare the proposal, establish the repository, and outline the analysis protocol. |
| Week 2 | Acquire and audit data, conduct preliminary analysis, and freeze temporal splits and evaluation rules. |
| Week 3 | Build features, validate ridge regularization, fit candidate networks, and calibrate warnings. |
| Week 4 | Apply selection objectives and baselines, assess feasibility and trade-offs, and freeze models and test-evaluation code. |
| Week 5 | Evaluate the held-out year and calculate station-level and seasonal results and paired uncertainty intervals. |
| Week 6 | Complete prespecified sensitivity checks, interpret limitations, finalize figures and tables, and draft the report. |
| Week 7 | Finalize the report and presentation, verify reproducibility, and upload code, configurations, and result summaries. |

## Repository maintenance and reproducibility

We will maintain this repository throughout the project and upload code and documentation as they become available. Planned deliverables include:

- Data acquisition and preprocessing code, with source and license attribution.
- Explicit temporal split definitions, feature settings, random seeds, and model configurations.
- Scripts for forecasting, calibration, network selection, evaluation, and uncertainty analysis.
- Reproducible environment specifications and executable instructions.
- Generated result summaries, figures, and documented limitations.

Analysis decisions will be recorded before final test evaluation, and changes to the protocol will be documented. Raw-data provenance and download instructions will be retained so that the analysis can be reproduced from the original sources.

To obtain the repository:

```bash
git clone https://github.com/SeanWan514/pm25-sensor-selection.git
cd pm25-sensor-selection
```

The planned analysis stack is Python, pandas, NumPy, and scikit-learn. Training commands and dependency versions will be added with the implementation.

## Interpretation limits

The study uses one city, one historical period, and one main forecasting family. Missingness, nonlinear relationships, temporal drift, and selection overfitting may affect the findings. Equal weighting of monitoring stations does not establish demographic or population-level fairness. A retrospective improvement in warning metrics would not demonstrate health benefits or readiness for operational deployment. Null findings and infeasible constraints will be reported alongside positive results.

## Selected references

- Chen, S. (2017). *Beijing Multi-Site Air Quality*. UCI Machine Learning Repository. [Dataset and DOI](https://doi.org/10.24432/C5RK5G).
- Hoerl, A. E., and Kennard, R. W. (1970). Ridge regression: Biased estimation for nonorthogonal problems. *Technometrics*, 12(1), 55–67. [DOI](https://doi.org/10.1080/00401706.1970.10488634).
- Krause, A., Singh, A., and Guestrin, C. (2008). Near-optimal sensor placements in Gaussian processes: Theory, efficient algorithms and empirical studies. *Journal of Machine Learning Research*, 9, 235–284. [Paper](https://jmlr.org/papers/v9/krause08a.html).
- Kelp, M. M., et al. (2023). Data-driven placement of PM2.5 air quality sensors in the United States: An approach to target urban environmental injustice. *GeoHealth*, 7(9), e2023GH000834. [DOI](https://doi.org/10.1029/2023GH000834).
- Alvarado-Alcon, F.-J., et al. (2026). Citywide air quality forecasting over sparse sensor networks: Cross-location generalization and deep learning reliability under missing data. *Journal of Sensor and Actuator Networks*, 15(4), 52. [DOI](https://doi.org/10.3390/jsan15040052).
- Tashman, L. J. (2000). Out-of-sample tests of forecasting accuracy: An analysis and review. *International Journal of Forecasting*, 16(4), 437–450. [DOI](https://doi.org/10.1016/S0169-2070(00)00065-0).
- Künsch, H. R. (1989). The jackknife and the bootstrap for general stationary observations. *The Annals of Statistics*, 17(3), 1217–1241. [DOI](https://doi.org/10.1214/aos/1176347265).
