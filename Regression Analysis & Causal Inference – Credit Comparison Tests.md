# Regression Analysis & Causal Inference – Credit Comparison Tests

**Joachim Vogl** | Colorado State University

---

## Overview

This analysis uses cross-sectional data on 1,319 credit card applicants (Greene, 2003) from the `AER` package in R. The goal is to examine whether self-employment status and number of dependents have a statistically significant effect on average monthly credit card expenditure.

Two models are estimated:
- **Simple regression:** expenditure ~ self-employment status
- **Multiple regression:** expenditure ~ self-employment status + number of dependents

---

## Data

| Variable | Description |
|---|---|
| `expenditure` | Average monthly credit card expenditure ($) |
| `selfemp` | Binary: 1 = self-employed, 0 = not self-employed |
| `dependents` | Number of dependents (children, adult dependents, etc.) |

---

## Code

```r
library(AER)
data(CreditCard)

# Correlation between expenditure and self-employment status
cor(CreditCard$expenditure, as.numeric(CreditCard$selfemp == "yes"))
# R = -0.036 — very weak negative correlation

# Simple linear regression
mod <- lm(expenditure ~ selfemp, data = CreditCard)

# Multiple linear regression
mult.mod <- lm(expenditure ~ selfemp + dependents, data = CreditCard)

# Results
summary(mod)
summary(mult.mod)

# 95% Confidence interval for selfemp coefficient
confint(mod)
```

---

## Results

### Simple Regression: expenditure ~ selfemp

| | Estimate | Std. Error | t value | p-value |
|---|---|---|---|---|
| Intercept | 187.697 | 7.766 | 24.168 | < 2e-16 *** |
| selfempyes | -38.264 | 29.567 | -1.294 | 0.196 |

- R² = 0.00127 — self-employment alone explains less than 0.2% of variation in expenditure
- Residual Standard Error = $272.10

### Multiple Regression: expenditure ~ selfemp + dependents

| | Estimate | Std. Error | t value | p-value |
|---|---|---|---|---|
| Intercept | 176.100 | 9.737 | 18.086 | < 2e-16 *** |
| selfempyes | -40.716 | 29.561 | -1.377 | 0.169 |
| dependents | 11.838 | 6.007 | 1.971 | 0.049 * |

- R² = 0.0042 — slight improvement, dependents is significant at 5%
- F-statistic: 2.781, p = 0.062 — model significant at 10% but not 5%

---

## Interpretation

**(a) Self-employment coefficient:**
Self-employed individuals spend on average **$38.26 less per month** on credit card expenditure compared to non-self-employed individuals, holding all else equal. The intercept of $187.70 represents the predicted monthly expenditure for a non-self-employed person with zero dependents.

**(b) Hypothesis test at 10% significance level:**
We **fail to reject** the null hypothesis that the coefficient on selfemp equals zero.
- p-value = 0.196 > 0.10 → fail to reject
- |t-statistic| = |-1.294| = 1.294 < 1.645 → fail to reject

There is insufficient evidence at the 10% level to conclude that self-employment status has a statistically significant effect on monthly expenditure.

**(c) 95% Confidence Interval for selfemp:**

```
CI = estimate ± (1.96 × standard error)
   = -38.264 ± (1.96 × 29.567)
   = (-96.21, 19.69)
```

The interval **contains zero**, confirming we fail to reject H₀ at the 5% significance level. This is consistent with the hypothesis test result in (b).

---

## Visualization

```r
library(ggplot2)

ggplot(CreditCard, aes(x = dependents, y = expenditure, color = selfemp)) +
  geom_point(alpha = 0.4, size = 2) +
  geom_smooth(method = "lm", se = TRUE) +
  scale_color_manual(values = c("no" = "coral", "yes" = "steelblue"),
                     labels = c("Not Self-Employed", "Self-Employed")) +
  labs(title = "Credit Card Expenditure by Self-Employment Status",
       x = "Number of Dependents",
       y = "Average Monthly Expenditure ($)",
       color = "Employment Status") +
  theme_minimal()
```
<img width="927" height="586" alt="image" src="https://github.com/user-attachments/assets/1f3eec1b-a56d-407c-a7ad-9bc36427371b" />

---

## Key Findings

- Self-employment status has a **negative but statistically insignificant** effect on monthly credit card expenditure at conventional significance levels
- Number of dependents has a **small positive and marginally significant** effect ($11.84 per additional dependent, p = 0.049)
- Both models have **very low R²**, indicating that self-employment status and number of dependents alone are weak predictors of credit card spending — other variables in the dataset (income, age, credit history) would likely improve explanatory power substantially
- The 95% confidence interval for the self-employment coefficient is (-$96.21, $19.69), which contains zero, consistent with failing to reject the null hypothesis

---

## Tools

- **R** / RStudio
- Packages: `AER`, `ggplot2`
- Dataset: CreditCard (Greene, 2003) via AER package
