
# [0300] Wilcoxon Signed Rank Test - Design

---
- [Input Dataset](#input-dataset)
- [Define Numeric Agreement Criteria](#define-numeric-agreement-criteria)
- [Expectations for Each Supported Statistic](#expectations-for-eachs-supported-statistic)
- [Function Usage and Critical Argument](#function-usage-and-critical-argument)
- [Incompatibilities Across SAS, R and Python](#incompatibilities-across-sas-r-and-python)
- [Comparison protocol and metrics](#comparison-protocol-and-metrics)

---

# Input Dataset

To keep synchronous input dataset with CAMIS Project's, we still use anonymized data from 2-period, cross-over study comparing treatments A and B in patients with asthma and acute airway obstruction induced by repeated mannitol challenges. The dataset includes neither “0” differences nor ties.

Wilcoxon signed rank test was applied to analyse the time to return to baseline FEV1 post-mannitol challenge 2. Median difference, p value and 95% CI were provided using the Hodges-Lehmann estimate.

Please note: the dataset is simulated by Python and not pointed to any real clinical trial.

```python
# simulated initialized statistics
np.random.seed(100)
num_patients = 50
mean_time_A = 25
mean_time_B = 22
sd_time = 5

# To make sure there is no zero or ties in differences
# Generate differences with a minimum absolute difference threshold
min_diff = 0.1 

# Generate recovery times for treatment A
time_A = np.random.normal(loc=mean_time_A, scale=sd_time, size=num_patients).round(3)
# Generate differences with a normal distribution centered around (mean_time_B - mean_time_A)
diffs = np.random.normal(loc=(mean_time_B - mean_time_A), scale=sd_time/2, size=num_patients)
# Make sure differences are not too close to zero
diffs = np.where(np.abs(diffs) < min_diff, np.sign(diffs) * min_diff, diffs)
time_B = (time_A + diffs).round(3)

# Create patient IDs
patient_ids = [f"CAMIS-{i:03}" for i in range(1, num_patients + 1)]

# Build DataFrame
crossover_df = pd.DataFrame({
    "PATIENT": patient_ids,
    "TIME_TO_BASELINE_A": time_A,
    "TIME_TO_BASELINE_B": time_B
})

crossover_df.to_csv("Wilcoxon_signed_rank_dataset.csv", index=False)
```

# Define Numeric Agreement Criteria

- Test Statistic (abs diff < 0.000001)
- P-value (abs diff < 0.000001)
- Confidence Interval (abs diff < 0.000001)
- Median Estimate (abs diff < 0.000001)

# Expectations for Each Supported Statistic

The following statistics should have the consistent values and results across all the methods:
- Test Statistic (T+, T-, W, S)
- P-value
- Confidence Interval
- Median Estimate
- Whether reject hypothesis and conclusion

# Function Usage and Critical Argument

## SAS (PROC UNIVARIATE)
Use `PROC UNIVARIATE` with required args `DATA=`, `VAR=`, `m0=`, optional args `EXACT`, `CONFIDENCE`, `PAIRED`.

## R (asht)
Use `wsrTest` from `asht` package with required args `x=`, optional args `y=`, `mu=`, `alternative=`, `conf.int =`, `conf.level=`, `digits=`, `tieDigits`.

## R (stats)
Use `wilcox.test` from `stats` package with required args `x=`, optional args `y=`, `mu=`, `alternative=`, `conf.int =`, `conf.level=`, `paired=`, `digits.rank=`, `exact=`, `tol.root=`.

## Python (scipy.stats)
Use `wilcoxon` from `scipy.stats` package with required args `x=`, optional args `y=`, `zero_method=`, `correction=`, `alternative=`, `method=`, `axis=`, `nan_policy`, `keepdims=`.

# Incompatibilities Across SAS, R and Python

- Definition of test statistic
  - R returns `T+` (sum of positive ranks without ties), Python returns `W` statistic (the smaller one of the sum of ranks of positive differences and the sum of ranks of negative differences), SAS returns `S` statistic(modification of T+), so values differ
- Handling ties and zero-values
  - automatically handled or by changing method argument input
- Exact method or normal approximation
  - SAS automatically chooses whether to do exact or non-exact based on sample size n = 20
  - R and Python have a flexibility of choosing different methods to calculate p-value (exact, non-exact etc)


# Comparison protocol and metrics
The expected comparison metrics include a series of output statistic, like test statistic, p-value, CI, estimate across R, Python, and SAS, based on different conditions and methods (exact method, normal approximation, continuity correction, etc.). The acceptable difference between the output values will refer to the aforementioned Part 2. 

