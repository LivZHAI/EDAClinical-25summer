
# [0300] Wilcoxon Signed Rank Test - Requirements Analysis

---
- [Background](#background)
  - [Hypotheses](#hypotheses)
  - [Assumptions](#assumptions)
  - [Test Statistic & P-value](#test-statistic--p-value)
- [Package Implementations](#package-implementations)
  - [SAS (PROC UNIVARIATE)](#sas-proc-univariate)
    - [Function/Procedure (SAS)](#functionprocedure-SAS)
    - [Inputs (SAS)](#inputs-SAS)
    - [Outputs (SAS)](#outputs-SAS)
    - [Sample Code (SAS)](#sample-code-SAS)
    - [Limitations/Notes (SAS)](#limitationsnotes-SAS)
  - [R (asht)](#r-asht)
    - [Function/Procedure (R)](#functionprocedure-R)
    - [Inputs (R)](#inputs-R)
    - [Outputs (R)](#outputs-R)
    - [Sample Code (R)](#sample-code-R)
    - [Limitations/Notes (R)](#limitationsnotes-R)
  - [R (stats)](#r-stats)
    - [Function/Procedure (R)](#functionprocedure-R)
    - [Inputs (R)](#inputs-R)
    - [Outputs (R)](#outputs-R)
    - [Sample Code (R)](#sample-code-R)
    - [Limitations/Notes (R)](#limitationsnotes-R)
  - [Python (scipy.stats)](#python-scipystats)
    - [Function/Procedure (Python)](#functionprocedure-Python)
    - [Inputs (Python)](#inputs-Python)
    - [Outputs (Python)](#outputs-python)
    - [Sample Code (Python)](#sample-code-python)
    - [Limitations/Notes (Python)](#limitationsnotes-python)
- [Summary](#summary)
- [References](#references)


---

# Background

The Wilcoxon signed-rank test is a non-parametric rank test for statistical hypothesis testing used either to test the location (median) of a population based on a sample of data, or to compare the locations (medians) of two populations using two matched samples. The one-sample version serves a purpose similar to that of the one-sample Student's t-test. For two matched samples, it is a paired difference test like the paired Student's t-test (also known as the "t-test for matched pairs" or "t-test for dependent samples"). The Wilcoxon signed rank test is a good alternative to the t-test when the normal distribution of the differences between paired individuals cannot be assumed. [Wikipedia]


## Hypotheses
**One-sample Test:**

- null hypothesis: the median of the population is equal to 0

- alternative hypothesis: the median of the population is not equal to 0

**Paired data test:**

- null hypothesis: the difference between the medians of the paired population is equal to 0

- alternative hypothesis: the difference between the medians of the paired population is not equal to 0


## Assumptions
- **Dependent Samples:** The test requires two sets of measurements that are related or paired. This typically involves observations taken from the same subjects under two different conditions (e.g., before and after an intervention), making them dependent.

- **Independence within Pairs:** the pairs of observations are independent of each other, ensuring that each pair contributes uniquely to the analysis, which means that the paired observations are randomly and independently drawn.

- **Symmetric Distribution of Differences:** The distribution of the differences between paired samples should be approximately symmetric around the median.

- **Ordinal or Continuous Dependent Variable:** The dependent variable should be measured at least at the ordinal level (e.g., rankings, Likert scales) or continuously (e.g., interval or ratio scales).

## Test Statistic & P-value
**Test statistic**: 

$$
T^+ = \sum_{\substack{1 \leq i \leq n \\ X_i > 0}} R_i,\quad
T^- = \sum_{\substack{1 \leq i \leq n \\ X_i < 0}} R_i.
$$

where 
- $X_i$ is the ith observation
- $R_i$ is the rank of the ith observation

Note
- If several observations have the same absolute value (tie), then assign each observation with the mean rank for these observations.

**P-value**:
- Exact Method: (Small sample sizes (typically n ≤ 20))
    - Two sided:  
    ![function](https://quicklatex.com/cache3/90/ql_7ca5768b3028e45863630b71f9e4bb90_l3.png)

    - One sided:
    ![function](https://quicklatex.com/cache3/be/ql_c4a3b22a9f318d6daaf99a9fde4b41be_l3.png)

- Normal Approximation:
    - No Ties:
    $$
    Z_T = \frac{\left| T_+ - \left[ \frac{n(n+1)}{4} \right] \right| - 0.5}{\sqrt{ \frac{n(n+1)(2n+1)}{24}}}
    $$

    - Ties:
    $$
    Z_T = \frac{\left| T_+ - \left[ \frac{n(n+1)}{4} \right] \right| - 0.5}{\sqrt{\frac{n(n+1)(2n+1)}{24} - \sum_{i=1}^{g} \frac{t_i^3 - t_i}{48}}}
    $$

where  
- $g$ = number of tied groups  
- $t_i$ = number of observations in the $i^{th}$ tied group


# Package Implementations

## SAS (PROC UNIVARIATE)

### Function/Procedure (SAS) 
Use `PROC UNIVARIATE` function.

### Inputs (SAS)
- **Required**: 
    - `DATA=` (name of dataset)
    - `VAR=` (variable name for Wilcoxon signed rank test)
    - `m0=` (null hypothesis: median = 0)
    
- **Optional**: 
    - `EXACT`: use Exact Method to calculate p-value, by default `EXACT=NONE`
    - `CONFIDENCE`: set confidence level
    - `PAIRED`: calculate paired data as variable
    - `ods select TestsForLocation`: only output test statistic and p-value regarding Wilcoxon test

### Outputs (SAS)
- Table of **Tests for Location**
    - P-value
    - test statistic 
- Table of **Moments**
    - Skewness  
    - Kurtosis
- **Basic statistical measures**
    - Mean 
    - Min & max
    - Std. Deviation
    - SE (std error) 

### Sample Code (SAS)
- one-sample code
```sas
proc univariate data=your_dataset;
   var your_variable;
   mu0 = 0;  /* H0: median = 0 */
   exact;           
   confidence 0.95;
run;
```
- paired test code
```sas
proc univariate data=paired_data;
   var before after;      
   paired before*after;    
   mu0 = 0;                
   exact;               
   confidence 0.95; 
run;
```

### Limitations/Notes (SAS)
- Only PROC UNIVARIATE can be used in SAS to perform Wilcoxon Signed-Rank test
- Provided p-value is based on S statistic (modification of a common T+), where 
$$
S = T^+ - \frac{n_T(n_T + 1)}{4}
$$
- All the 0 differences are disregarded 

---
## R (asht)

### Function/Procedure (R)
Use `wsrTest` from base R `asht` package. 

### Inputs (R)
- **Required**: 
    - `x` (numeric vector of data)

- **Optional**: 
    - `y` (second sample for paired test, default is NULL)
    - `mu` (null hypothesis value for the median, default is 0)
    - `alternative` (alternative hypothesis type, default is "two.sided")
    - `conf.int` (whether to compute the confidence interval, default is TRUE)
    - `conf.level` (confidence level, default is 0.95)
    - `digits` (number of digits to display, default is NULL)
    - `tieDigits` (number of digits for tied values, default is 8)

### Outputs (R)
- T+ statistic
- P-value 
- Confidence Interval 
- Estimated median difference
- Alternative (a character string describing the alternative hypothesis) 
- Rank details (if there are tied ranks, some additional information about how ties are handled and the rank computation may be included)

### Sample Code (R)
```r
wsrTest(x, y = NULL, conf.int = TRUE, conf.level = 0.95,
   mu = 0, alternative = c("two.sided", "less", "greater"),
   digits = NULL, tieDigits=8)
```
### Limitations/Notes (R)
- This function calculates the **exact** Wilcoxon signed rank test using the Pratt method if there are zeros. In other words, rank the differences equal to zero together with the absolute value of the differences, but then permute the signs of only the non-zero ranks. 

---
## R (stats)

### Function/Procedure (R)
Use `wilcox.test` from base R `stats` package. 

### Inputs (R)
- **Required**: 
    - `x` (numeric vector of data)

- **Optional**: 
    - `y` (second sample for paired test, default is NULL)
    - `mu` (null hypothesis value for the median, default is 0)
    - `alternative` (alternative hypothesis type, default is "two.sided")
    - `conf.int` (whether to compute the confidence interval, default is TRUE)
    - `conf.level` (confidence level, default is 0.95)
    - `paired = TRUE` (Logical value indicating whether the test is for paired data, used when y included)
    - `exact` (whether to use an exact method to compute the p-value)
    - `tol.root` (the tolerance for approximating the root in the computation of the test statistic)
    - `digits.rank` (the number of digits to which ranks are computed. Default is Inf, meaning full precision is used for ranks)

### Outputs (R)
- T+ statistic
- P-value 
- Confidence Interval 
- Estimated median difference
- Alternative (a character string describing the alternative hypothesis)

### Sample Code (R)
```r
## Default S3 method:
wilcox.test(x, y,
            alternative = c("two.sided", "less", "greater"),
            mu = 0, paired = TRUE, exact = NULL,
            conf.int = TRUE, conf.level = 0.95,
            tol.root = 1e-4, digits.rank = Inf, ...)
```

### Limitations/Notes (R)
- the ranks of 0 difference are eliminated by default
- `paired` should be set to `True` if `y` is included
- `exact=NULL` automatically adopts exact calculation or normal appromation based on sample size (n<=50)

---
## Python (scipy.stats)

### Function/Procedure (Python)
Use `scipy.stats.wilcoxon` from `scipy` package 

### Inputs (Python)
- **Required**: 
    - `x` (sample data array)

- **Optional**: 
    - `y` (second sample for paired test, default is None)
    - `zero_method` (Specifies how zero values are handled; <br>
                     'wilcox' (default): Zero values are excluded from the test<br>
                     'pratt': Zero values are included as tied ranks<br>
                     'zsplit': Zero values are treated as half the rank of the first non-zero value)
    - `correction` (A boolean (True or False) indicating whether a continuity correction should be applied to the test statistic)
    - `alternative` (`two-sided`, `less`, or `greater`, alternative hypothesis type, default is "two-sided")
    - `method` (`auto`, `exact`, or `asymptotic`, specifies the method used to compute the test statistic, default is "auto")
    - `axis` (specifies the axis along which to compute the test if the input is a multi-dimensional array)
    - `nan_policy` (determines how NaN values are handled;<br>
                    'propagate' (default): Returns NaN if there are any NaN values in the data<br>
                    'raise': Raises an error if NaN values are present<br>
                    'omit': Ignores NaN values and computes the test based on non-NaN values)
    - `keepdims` (If True, the result will be broadcasted to the same shape as the input)

### Outputs (Python)
- Test statistic 
- P-value 

### Sample Code (Python)
```python
import pandas as pd
from scipy import stats

wilcoxon(x, y=None, zero_method='wilcox', correction=False,
alternative='two-sided', method='auto', *, axis=0,
nan_policy='propagate', keepdims=False)
```

### Limitations/Notes (Python)
- Axis defaults to `0`
- Keepdims defaults to `False`

---

# Summary
- SAS is highly automated for smaller samples and has extensive options for configuring the test, including continuity corrections and confidence intervals.
- R provides a straightforward implementation with both paired and unpaired test options, along with a zero-difference feature.
- Python offers a flexible and efficient implementation of the Wilcoxon Signed-Rank Test with control over exact methods, continuity corrections, and the alternative hypothesis.

# References

- SAS:
    - Introduction to Wilcoxon Signed Rank Test: https://support.sas.com/documentation/onlinedoc/stat/142/intronpar.pdf
    - Using Proc UNIVARIATE to execute the method, the results are displayed in Tests for Location: https://documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/procstat/procstat_univariate_details17.htm#procstat_univariate015076

- R:
    - wsrTest{asht}：https://search.r-project.org/CRAN/refmans/asht/html/wsrTest.html
    - wilcox.test{stats}: https://search.r-project.org/R/refmans/stats/html/wilcox.test.html

- Python:
    - scipy.stats.wilcoxon(): https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.wilcoxon.html

- Notes: 
    - https://phuse.s3.eu-central-1.amazonaws.com/Archive/2024/Connect/EU/Strasbourg/PAP_AS04.pdf
    - https://blogs.sas.com/content/iml/2023/07/19/wilcoxon-signed-rank.html




