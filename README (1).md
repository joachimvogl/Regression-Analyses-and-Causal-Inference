# Log & Quadratic Regression Models

**Joachim Vogl | Colorado State University**

## Overview

This analysis uses cross-sectional data on **1,319 credit card applicants** (Greene, 2003) from the `AER` package in R. The goal is to examine the relationship between **age** and **average monthly credit card expenditure** using **nonlinear functional forms**, while controlling for number of dependents and self-employment status.

Three models are estimated and compared:

- **Baseline linear:** expenditure ~ age + dependents + self-employment
- **Quadratic model:** expenditure ~ age + age² + dependents + self-employment
- **Linear-log model:** expenditure ~ log(age) + dependents + self-employment

All models are estimated with **heteroskedasticity-robust (HC1) standard errors**, since the residual variance is not constant across the sample.

## Data

| Variable | Description |
| --- | --- |
| `expenditure` | Average monthly credit card expenditure ($) |
| `age` | Age of applicant (years) |
| `dependents` | Number of dependents (children, adult dependents, etc.) |
| `selfemp` | Binary: 1 = self-employed, 0 = not self-employed |

## Code

```r
library(AER)
library(lmtest)
library(sandwich)
data(CreditCard)

# Baseline linear model (robust SE)
linear.model <- lm(expenditure ~ age + dependents + selfemp, data = CreditCard)
coeftest(linear.model, vcov = vcovHC(linear.model, type = "HC1"))

# Joint F-test: are dependents and age jointly zero?
linearHypothesis(linear.model,
                 c("dependents = 0", "age = 0"),
                 vcov. = vcovHC(linear.model, type = "HC1"))

# Quadratic model in age
quadratic.model <- lm(expenditure ~ age + I(age^2) + dependents + selfemp, data = CreditCard)
coeftest(quadratic.model, vcov = vcovHC(quadratic.model, type = "HC1"))

# Linear-log model in age
linearlog.model <- lm(expenditure ~ log(age) + dependents + selfemp, data = CreditCard)
coeftest(linearlog.model, vcov = vcovHC(linearlog.model, type = "HC1"))
```

## Results

### Baseline Linear: expenditure ~ age + dependents + selfemp

| Term | Estimate | Std. Error | t value | p-value |
| --- | ---: | ---: | ---: | ---: |
| (Intercept) | 169.676 | 23.147 | 7.330 | 3.997e-13 *** |
| dependents | 11.490 | 6.565 | 1.750 | 0.080 . |
| selfempyes | −41.469 | 27.376 | −1.515 | 0.130 |
| age | 0.205 | 0.725 | 0.283 | 0.777 |

- `dependents` is significant at the **10% level only** (p = 0.080); `selfemp` and `age` are not significant at any conventional level.
- In levels, `age` has essentially no measurable linear effect (p = 0.777) — motivating the nonlinear specifications below.

### Joint F-Test: dependents and age jointly zero

- F-statistic = **1.631**, p = **0.196** → **fail to reject** H₀. Together, `dependents` and `age` do not jointly explain variation in expenditure in the linear model.

### Quadratic Model: expenditure ~ age + age² + dependents + selfemp

| Term | Estimate | Std. Error | t value | p-value |
| --- | ---: | ---: | ---: | ---: |
| (Intercept) | 85.904 | 44.903 | 1.913 | 0.056 . |
| age | 5.155 | 2.451 | 2.103 | 0.036 * |
| I(age^2) | −0.065 | 0.031 | −2.131 | 0.033 * |
| dependents | 9.738 | 6.617 | 1.472 | 0.141 |
| selfempyes | −40.413 | 27.258 | −1.483 | 0.138 |

- Both `age` and `age²` are significant at the **5% level** (p = 0.036 and p = 0.033) → the nonlinearity is **genuine, not noise**.
- Positive linear term + negative squared term = a **concave** relationship: expenditure rises with age, then falls.

### Linear-Log Model: expenditure ~ log(age) + dependents + selfemp

| Term | Estimate | Std. Error | t value | p-value |
| --- | ---: | ---: | ---: | ---: |
| (Intercept) | 162.334 | 49.427 | 3.284 | 0.001 ** |
| log(age) | 4.069 | 14.807 | 0.275 | 0.783 |
| dependents | 11.612 | 6.555 | 1.772 | 0.077 . |
| selfempyes | −41.184 | 27.052 | −1.522 | 0.128 |

- `log(age)` is **not significant** (p = 0.783): forcing a monotonic log shape onto age misses the curvature the quadratic captured.
- Only `dependents` is significant here, and only at the 10% level (p = 0.077).

---

## Interpretation

**(a) Quadratic model — the age effect.** The positive coefficient on `age` (+5.15) paired with the negative coefficient on `age²` (−0.065) is the textbook signature of a variable that **increases at a decreasing rate**. Expenditure rises with age up to a turning point and then declines. Because the squared term is significant at 5%, we can be confident the curvature is real rather than a sampling artifact.

**(b) Linear-log model — the age effect.** The coefficient on `log(age)` is 4.07, implying that a 1% increase in age is associated with only about **β/100 ≈ $0.04** more in monthly expenditure — economically negligible. With p = 0.783 we **fail to reject** the null that age has no effect. The log form assumes a smooth, monotonic relationship, which does not describe the rise-then-fall pattern in the data.

**(c) Which functional form fits better?** The **quadratic specification is the better description** of how age relates to expenditure. It is the only form in which age enters significantly, and its concave shape matches the fitted curve below. The linear-log form is a poor fit precisely because it cannot bend back down — it can only flatten. This is a useful reminder that a log transform is not a universal fix for nonlinearity: it captures diminishing returns, but not a genuine peak-and-decline.

## Visualization

```r
# Scatterplot of expenditure vs. age with linear and quadratic fits
plot(CreditCard$age, CreditCard$expenditure,
     col = "steelblue", pch = 20,
     xlab = "Age (years)", ylab = "Expenditure (dollars)",
     main = "Estimated Linear and Quadratic Regression Functions")

lin_age  <- lm(expenditure ~ age, data = CreditCard)
abline(lin_age, col = "green", lwd = 2)

quad_age <- lm(expenditure ~ age + I(age^2), data = CreditCard)
age_order <- order(CreditCard$age)
lines(CreditCard$age[age_order], fitted(quad_age)[age_order], col = "red", lwd = 2)

legend("topright", legend = c("Linear", "Quadratic"),
       col = c("green", "red"), lwd = 2)
```

![Scatterplot of credit card expenditure against age, with a flat green linear fit and a concave red quadratic fit that rises then falls.](images/quadratic_plot.png)

*The green linear fit is essentially flat, while the red quadratic fit rises through the middle-age range and declines at the extremes — visual confirmation of the concave relationship.*

## Key Findings

- In a purely **linear** specification, age has **no statistically significant effect** on monthly expenditure (p = 0.777).
- Adding a **quadratic term** reveals a genuine **concave (rise-then-fall) relationship**: both age and age² are significant at the 5% level.
- The **linear-log** form fails to capture this pattern — `log(age)` is insignificant (p = 0.783) because a log curve cannot turn back downward.
- Across all specifications, `dependents` is the most consistent predictor (significant at ~10%), while `selfemp` is never significant.
- **Takeaway:** functional form matters. The right transformation is dictated by the true shape of the relationship, not applied by default.

## Tools

- R / RStudio
- Packages: `AER`, `lmtest`, `sandwich`
- Dataset: CreditCard (Greene, 2003) via the `AER` package
