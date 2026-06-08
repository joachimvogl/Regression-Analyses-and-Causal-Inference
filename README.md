# Regression Analysis - Credit Comparison Tests

This analysis uses cross-sectional data on 1,319 credit card applicants (Greene, 2003) from the `AER` package in R. The goal is to examine whether self-employment status and number of dependents have a statistically significant effect on average monthly credit card expenditure.

Two models are estimated:
- **Simple regression:** expenditure ~ self-employment status
- **Multiple regression:** expenditure ~ self-employment status + number of dependents

## Key Findings

- Self-employment status has a **negative but statistically insignificant** effect on monthly credit card expenditure at conventional significance levels
- Number of dependents has a **small positive and marginally significant** effect ($11.84 per additional dependent, p = 0.049)
- Both models have **very low R²**, indicating that self-employment status and number of dependents alone are weak predictors of credit card spending — other variables such as income, age, and credit history would likely improve explanatory power substantially
- The 95% confidence interval for the self-employment coefficient is (-$96.21, $19.69), which contains zero, consistent with failing to reject the null hypothesis

## Tools
R / RStudio · `AER` · `ggplot2`
