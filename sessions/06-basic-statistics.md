# Basic Statistics in R

## Overview

This tutorial introduces statistical analysis in **R** using biomedical-style example data.

The aim is not only to run statistical tests, but also to understand:

- which test to use
- which R function performs the test
- how to inspect the output
- how to interpret the result

You will learn how to:

- calculate descriptive statistics
- work with probability distributions
- calculate confidence intervals
- perform hypothesis tests
- compare two or more groups
- use parametric and non-parametric tests
- analyze categorical variables
- perform correlation analysis
- fit linear and generalized linear models
- check model assumptions
- perform survival analysis
- calculate statistical power and sample size
- export statistical results

---

# 1. Prerequisites

Before starting, make sure R and RStudio are installed.

You only need to install packages **once** on your computer.

After installation, packages can be loaded using:

```r
library(package_name)
```

---

# 2. Install and Load Required Packages

Install the packages once:

```r
install.packages(c(
  "tidyverse",
  "ggpubr",
  "rstatix",
  "skimr",
  "janitor",
  "BSDA",
  "car",
  "lmtest",
  "gamlr",
  "survival",
  "survminer",
  "pwr",
  "epiR"
))
```

Load the packages when starting a new R session:

```r
library(tidyverse)
library(ggpubr)
library(rstatix)
library(skimr)
library(janitor)
```

Other packages will be loaded when needed later in this tutorial.

---

# 3. Create an Example Biomedical Dataset

In this tutorial, we will use a simulated biomedical dataset.

The dataset contains:

- age
- sex
- treatment group
- body mass index
- glucose level
- cholesterol level
- inflammatory marker level
- disease status

```r
set.seed(123)

patient_data <- tibble(
  patient_id = 1:120,
  age = round(rnorm(120, mean = 52, sd = 12)),
  sex = sample(c("Female", "Male"), 120, replace = TRUE),
  treatment_group = sample(c("Control", "Treatment"), 120, replace = TRUE),
  bmi = round(rnorm(120, mean = 27, sd = 4), 1),
  glucose = round(rnorm(120, mean = 105, sd = 20), 1),
  cholesterol = round(rnorm(120, mean = 190, sd = 35), 1),
  crp = round(rlnorm(120, meanlog = 1.2, sdlog = 0.6), 2),
  disease_status = sample(c("Healthy", "Disease"), 120, replace = TRUE)
)

head(patient_data)
```

---

# 4. Inspect the Dataset

Before performing statistical analyses, always inspect the dataset.

```r
str(patient_data)

names(patient_data)

dim(patient_data)

summary(patient_data)
```

---

# 5. Clean Column Names

The `janitor` package can clean column names and make them consistent.

```r
patient_data <- patient_data %>%
  clean_names()

names(patient_data)
```

---

# 6. Check for Missing Values

```r
colSums(is.na(patient_data))

sum(is.na(patient_data))
```

For demonstration, we add some missing values:

```r
patient_data$glucose[c(5, 12, 30)] <- NA
patient_data$bmi[c(8, 18)] <- NA

colSums(is.na(patient_data))
```

---

# 7. Handle Missing Values

There are several ways to handle missing data.

For this tutorial, we remove rows containing missing values.

```r
patient_data_clean <- patient_data %>%
  drop_na()

dim(patient_data)

dim(patient_data_clean)
```

The appropriate method for handling missing values depends on the study and why the data are missing.

---

# Descriptive Statistics

# 8. Descriptive Statistics

Descriptive statistics summarize the main characteristics of a dataset.

Important measures include:

- mean
- median
- variance
- standard deviation
- minimum
- maximum
- range
- quartiles
- percentiles
- interquartile range

---

# 9. Mean and Median

Calculate the mean:

```r
mean(patient_data_clean$glucose)
```

Calculate the median:

```r
median(patient_data_clean$glucose)
```

The **mean** is the arithmetic average.

The **median** is the middle observation after sorting the data.

The median is often useful for skewed variables or variables containing extreme values.

---

# 10. Variance and Standard Deviation

The variance describes how much the observations vary around the mean.

```r
var(patient_data_clean$glucose)
```

The standard deviation is:

```r
sd(patient_data_clean$glucose)
```

A larger standard deviation indicates that the observations are more spread out.

---

# 11. Minimum, Maximum, and Range

Minimum:

```r
min(patient_data_clean$glucose)
```

Maximum:

```r
max(patient_data_clean$glucose)
```

Both values together:

```r
range(patient_data_clean$glucose)
```

The statistical range is:

```r
max(patient_data_clean$glucose) -
  min(patient_data_clean$glucose)
```

---

# 12. Quartiles and Percentiles

Quartiles divide ordered observations into four parts.

The first quartile is the 25th percentile:

```r
quantile(
  patient_data_clean$glucose,
  probs = 0.25,
  type = 6
)
```

The third quartile is the 75th percentile:

```r
quantile(
  patient_data_clean$glucose,
  probs = 0.75,
  type = 6
)
```

Calculate multiple quartiles at once:

```r
quantile(
  patient_data_clean$glucose,
  probs = c(0.25, 0.50, 0.75),
  type = 6
)
```

Other percentiles can be calculated by changing `probs`.

For example:

```r
quantile(
  patient_data_clean$glucose,
  probs = c(0.10, 0.90),
  type = 6
)
```

---

# 13. Interquartile Range

The **interquartile range (IQR)** is the difference between the third and first quartiles.

```r
IQR(
  patient_data_clean$glucose,
  type = 6
)
```

The IQR describes the spread of the middle 50% of the data.

For skewed variables such as `crp`, the median and IQR are often useful summaries.

```r
median(patient_data_clean$crp)

IQR(
  patient_data_clean$crp,
  type = 6
)
```

---

# 14. Summary of Continuous Variables

```r
patient_data_clean %>%
  summarise(
    mean_age = mean(age),
    median_age = median(age),
    sd_age = sd(age),

    mean_bmi = mean(bmi),
    median_bmi = median(bmi),
    sd_bmi = sd(bmi),

    mean_glucose = mean(glucose),
    median_glucose = median(glucose),
    variance_glucose = var(glucose),
    sd_glucose = sd(glucose),
    min_glucose = min(glucose),
    max_glucose = max(glucose),
    q1_glucose = quantile(glucose, 0.25, type = 6),
    q3_glucose = quantile(glucose, 0.75, type = 6),
    iqr_glucose = IQR(glucose, type = 6),

    mean_cholesterol = mean(cholesterol),
    median_cholesterol = median(cholesterol),
    sd_cholesterol = sd(cholesterol),

    mean_crp = mean(crp),
    median_crp = median(crp),
    sd_crp = sd(crp)
  )
```

---

# 15. Summary Using `summary()`

```r
summary(patient_data_clean$glucose)
```

For a complete dataset:

```r
summary(patient_data_clean)
```

---

# 16. Summary Using `skimr`

```r
skim(patient_data_clean)
```

---

# 17. Frequency Tables

Use `table()` for categorical variables.

```r
table(patient_data_clean$sex)

table(patient_data_clean$treatment_group)

table(patient_data_clean$disease_status)
```

Using `dplyr`:

```r
patient_data_clean %>%
  count(sex)
```

---

# 18. Proportions

```r
prop.table(
  table(patient_data_clean$sex)
)
```

Convert to percentages:

```r
prop.table(
  table(patient_data_clean$sex)
) * 100
```

Using `dplyr`:

```r
patient_data_clean %>%
  count(sex) %>%
  mutate(
    percent = n / sum(n) * 100
  )
```

---

# Probability Distributions in R

R provides functions for working with probability distributions.

A common naming system is:

```text
d = density or probability
p = cumulative probability
q = quantile
r = generate random values
```

For example:

```text
dbinom()
pbinom()
qbinom()
rbinom()
```

---

# 19. Binomial Distribution

Suppose:

```text
X ~ Binomial(n = 4, p = 0.2)
```

Calculate:

```text
P(X = 2)
```

using:

```r
dbinom(
  2,
  size = 4,
  prob = 0.2
)
```

Calculate:

```text
P(X <= 2)
```

using:

```r
pbinom(
  2,
  size = 4,
  prob = 0.2
)
```

Calculate:

```text
P(X > 2)
```

using:

```r
pbinom(
  2,
  size = 4,
  prob = 0.2,
  lower.tail = FALSE
)
```

---

# 20. Multinomial Distribution

Suppose observations fall into four categories.

```r
dmultinom(
  c(12, 10, 2, 1),
  size = 25,
  prob = c(0.46, 0.42, 0.09, 0.03)
)
```

This calculates the probability of observing exactly:

```text
12, 10, 2, and 1
```

observations in the four categories.

---

# 21. Normal Distribution

Suppose:

```text
X ~ Normal(mean = 170, variance = 100)
```

The standard deviation is:

```r
sqrt(100)
```

Calculate:

```text
P(X <= 150)
```

using:

```r
pnorm(
  150,
  mean = 170,
  sd = sqrt(100)
)
```

Calculate:

```text
P(X > 150)
```

using:

```r
pnorm(
  150,
  mean = 170,
  sd = sqrt(100),
  lower.tail = FALSE
)
```

Calculate:

```text
P(150 <= X <= 160)
```

using:

```r
pnorm(
  160,
  mean = 170,
  sd = sqrt(100)
) -
  pnorm(
    150,
    mean = 170,
    sd = sqrt(100)
  )
```

For continuous distributions, the probability of observing exactly one value is zero.

Therefore `dnorm()` gives a **density**, not the probability of observing exactly one value.

---

# 22. Chi-Square Distribution

For a chi-square distribution with 2 degrees of freedom:

```r
pchisq(
  3,
  df = 2
)
```

For:

```text
P(X > 3)
```

use:

```r
pchisq(
  3,
  df = 2,
  lower.tail = FALSE
)
```

---

# 23. t Distribution

For a t distribution with 2 degrees of freedom:

```r
pt(
  3,
  df = 2
)
```

For:

```text
P(X > 3)
```

use:

```r
pt(
  3,
  df = 2,
  lower.tail = FALSE
)
```

---

# Confidence Intervals

A **confidence interval** provides a range of plausible values for a population parameter.

A common confidence level is:

```text
95%
```

---

# 24. Confidence Interval for One Mean

Use `t.test()`:

```r
t.test(
  patient_data_clean$glucose
)
```

The output includes:

- sample mean
- t statistic
- p-value
- confidence interval

For a 99% confidence interval:

```r
t.test(
  patient_data_clean$glucose,
  conf.level = 0.99
)
```

---

# 25. Confidence Interval for One Proportion

Suppose 63 of 136 patients are frequent smokers.

```r
prop.test(
  x = 63,
  n = 136,
  conf.level = 0.95
)
```

Here:

- `x` = number of successes
- `n` = total number of observations

For a binary variable containing 0 and 1:

```r
data01 <- c(
  0, 0, 0, 1, 0, 1, 0, 1, 1,
  0, 1, 0, 1, 0, 0, 1, 0, 1
)
```

Then:

```r
prop.test(
  x = sum(data01),
  n = length(data01)
)
```

---

# 26. Exact Confidence Interval for One Proportion

When the sample is small, an exact binomial method can be used.

```r
binom.test(
  x = 4,
  n = 10,
  conf.level = 0.90
)
```

---

# Hypothesis Testing

A hypothesis test usually compares:

```text
H0 = null hypothesis
HA = alternative hypothesis
```

The p-value measures how compatible the observed data are with the null hypothesis.

A commonly used significance level is:

```text
alpha = 0.05
```

---

# 27. One-Sample t-Test

Suppose we want to test whether mean glucose equals 100.

```r
t.test(
  patient_data_clean$glucose,
  mu = 100,
  alternative = "two.sided"
)
```

Alternative hypotheses can be:

```text
"two.sided"
"less"
"greater"
```

For example:

```r
t.test(
  patient_data_clean$glucose,
  mu = 100,
  alternative = "greater"
)
```

tests whether the population mean is greater than 100.

---

# 28. One-Proportion Test

Suppose we want to test:

```text
H0: p = 0.50
```

and observe 20 successes among 50 observations.

```r
prop.test(
  x = 20,
  n = 50,
  p = 0.50,
  alternative = "two.sided"
)
```

---

# 29. Exact One-Proportion Test

For small samples:

```r
binom.test(
  x = 4,
  n = 10,
  p = 0.45,
  alternative = "less"
)
```

---

# Normality Testing

# 30. Why Check Normality?

Several statistical procedures assume approximately normally distributed observations or residuals.

Examples include:

- t-tests
- ANOVA
- linear regression

Normality can be investigated using:

- histograms
- Q-Q plots
- Shapiro-Wilk tests

---

# 31. Histogram

```r
ggplot(
  patient_data_clean,
  aes(x = glucose)
) +
  geom_histogram(
    bins = 30,
    color = "black",
    fill = "skyblue"
  ) +
  labs(
    title = "Distribution of Glucose Levels",
    x = "Glucose Level",
    y = "Count"
  ) +
  theme_minimal()
```

---

# 32. Q-Q Plot

```r
ggqqplot(
  patient_data_clean$glucose
)
```

Using `ggplot2`:

```r
ggplot(
  patient_data_clean,
  aes(sample = glucose)
) +
  stat_qq() +
  stat_qq_line() +
  labs(
    title = "Q-Q Plot of Glucose Levels"
  ) +
  theme_minimal()
```

---

# 33. Shapiro-Wilk Test

```r
shapiro.test(
  patient_data_clean$glucose
)
```

A common interpretation is:

- p-value > 0.05: there is no strong statistical evidence against normality
- p-value <= 0.05: the data significantly deviate from normality

Always combine formal testing with plots and knowledge of the data.

---

# 34. Normality by Group

```r
patient_data_clean %>%
  group_by(treatment_group) %>%
  shapiro_test(glucose)
```

---

# Comparing Two Groups

# 35. Independent Samples t-Test

Use an independent samples t-test when the two groups contain different individuals.

Example:

> Is mean glucose different between Control and Treatment groups?

```r
t.test(
  glucose ~ treatment_group,
  data = patient_data_clean
)
```

By default, R uses Welch's t-test, which does not assume equal variances.

---

# 36. Comparing Variances

Use `var.test()` to compare two variances.

```r
control_glucose <- patient_data_clean %>%
  filter(treatment_group == "Control") %>%
  pull(glucose)

treatment_glucose <- patient_data_clean %>%
  filter(treatment_group == "Treatment") %>%
  pull(glucose)
```

Then:

```r
var.test(
  control_glucose,
  treatment_glucose
)
```

If you specifically want the pooled-variance version of the t-test:

```r
t.test(
  control_glucose,
  treatment_glucose,
  var.equal = TRUE
)
```

---

# 37. Paired t-Test

A paired t-test is used when two measurements come from the same individuals.

Example:

```r
before <- c(
  115, 112, 107, 119, 115,
  138, 126, 105, 104, 115
)

after <- c(
  128, 115, 106, 128, 122,
  145, 132, 109, 102, 117
)
```

Run:

```r
t.test(
  after,
  before,
  paired = TRUE
)
```

The pairing is important because each value in `before` corresponds to the same individual in `after`.

---

# 38. t-Test Using `rstatix`

```r
patient_data_clean %>%
  t_test(
    glucose ~ treatment_group
  )
```

Effect size:

```r
patient_data_clean %>%
  cohens_d(
    glucose ~ treatment_group
  )
```

---

# 39. Wilcoxon Rank-Sum Test

For two independent groups, a common non-parametric alternative is the Wilcoxon rank-sum test.

```r
wilcox.test(
  glucose ~ treatment_group,
  data = patient_data_clean,
  paired = FALSE
)
```

Using `rstatix`:

```r
patient_data_clean %>%
  wilcox_test(
    glucose ~ treatment_group
  )
```

---

# 40. Wilcoxon Signed-Rank Test

For paired measurements:

```r
wilcox.test(
  after,
  before,
  paired = TRUE
)
```

For a one-sample signed-rank test:

```r
wilcox.test(
  patient_data_clean$glucose,
  mu = 100
)
```

---

# 41. Sign Test

The sign test is available through the `BSDA` package.

```r
library(BSDA)
```

For paired data:

```r
SIGN.test(
  after,
  before,
  md = 0,
  alternative = "two.sided"
)
```

The sign test uses only the direction of the paired differences.

---

# Comparing More Than Two Groups

# 42. Create a Three-Group Variable

```r
patient_data_clean <- patient_data_clean %>%
  mutate(
    age_group = case_when(
      age < 40 ~ "Young",
      age >= 40 & age < 60 ~ "Middle-aged",
      age >= 60 ~ "Older"
    )
  )

patient_data_clean %>%
  count(age_group)
```

---

# 43. One-Way ANOVA

ANOVA compares means across three or more groups.

```r
anova_result <- aov(
  cholesterol ~ age_group,
  data = patient_data_clean
)

summary(anova_result)
```

---

# 44. ANOVA Using `rstatix`

```r
patient_data_clean %>%
  anova_test(
    cholesterol ~ age_group
  )
```

---

# 45. Post-Hoc Comparisons

If the overall ANOVA is significant, post-hoc comparisons can identify which groups differ.

```r
TukeyHSD(anova_result)
```

Using `rstatix`:

```r
patient_data_clean %>%
  tukey_hsd(
    cholesterol ~ age_group
  )
```

---

# 46. Kruskal-Wallis Test

A common non-parametric alternative to one-way ANOVA is:

```r
kruskal.test(
  cholesterol ~ age_group,
  data = patient_data_clean
)
```

Using `rstatix`:

```r
patient_data_clean %>%
  kruskal_test(
    cholesterol ~ age_group
  )
```

---

# Categorical Data Analysis

# 47. Contingency Tables

A contingency table summarizes two categorical variables.

```r
disease_treatment_table <- table(
  patient_data_clean$disease_status,
  patient_data_clean$treatment_group
)

disease_treatment_table
```

Add row and column totals:

```r
addmargins(
  disease_treatment_table
)
```

Convert frequencies to proportions:

```r
prop.table(
  disease_treatment_table
)
```

---

# 48. Chi-Square Test of Independence

```r
chisq.test(
  disease_treatment_table
)
```

The test evaluates whether the two categorical variables are independent.

---

# 49. Expected Cell Counts

The chi-square approximation works best when expected cell counts are sufficiently large.

Check expected counts:

```r
chisq.test(
  disease_treatment_table
)$expected
```

If expected counts are too small, Fisher's exact test may be more appropriate.

---

# 50. Continuity Correction

For a 2 x 2 table, R applies a continuity correction by default.

Disable it using:

```r
chisq.test(
  disease_treatment_table,
  correct = FALSE
)
```

---

# 51. Fisher's Exact Test

```r
fisher.test(
  disease_treatment_table
)
```

Fisher's exact test is especially useful for small cell counts.

---

# 52. Two-Proportion Test

Suppose we observe:

```text
Group A: 78 successes from 279
Group B: 40 successes from 211
```

Run:

```r
prop.test(
  x = c(78, 40),
  n = c(279, 211),
  alternative = "two.sided",
  correct = FALSE
)
```

The output contains:

- estimated proportions
- test statistic
- p-value
- confidence interval for the difference in proportions

---

# Correlation Analysis

# 53. Pearson Correlation

Pearson correlation measures linear association between two continuous variables.

```r
cor.test(
  patient_data_clean$bmi,
  patient_data_clean$glucose,
  method = "pearson"
)
```

The output includes:

- estimated correlation coefficient
- confidence interval
- p-value

Correlation does not imply causation.

---

# 54. Spearman Correlation

Spearman correlation is based on ranks.

```r
cor.test(
  patient_data_clean$bmi,
  patient_data_clean$glucose,
  method = "spearman"
)
```

It is useful when the relationship is monotonic but not necessarily linear.

---

# 55. Correlation Matrix

```r
continuous_vars <- patient_data_clean %>%
  select(
    age,
    bmi,
    glucose,
    cholesterol,
    crp
  )

cor_matrix <- cor(
  continuous_vars
)

cor_matrix
```

If missing values are present:

```r
cor_matrix <- cor(
  continuous_vars,
  use = "complete.obs",
  method = "pearson"
)

cor_matrix
```

---

# Linear Regression

# 56. Simple Linear Regression

Linear regression models a continuous outcome as a function of one or more predictors.

Example:

> Does BMI predict glucose?

```r
model_1 <- lm(
  glucose ~ bmi,
  data = patient_data_clean
)

summary(model_1)
```

The model has the form:

```text
glucose = intercept + slope * BMI
```

Important parts of `summary()` include:

- `Estimate`: estimated regression coefficients
- `Std. Error`: uncertainty of the estimates
- `t value`: test statistic
- `Pr(>|t|)`: p-value
- `R-squared`: proportion of variation explained by the model
- `F-statistic`: overall model test

---

# 57. Confidence Intervals for Regression Coefficients

```r
confint(
  model_1
)
```

---

# 58. Regression Plot

```r
ggplot(
  patient_data_clean,
  aes(
    x = bmi,
    y = glucose
  )
) +
  geom_point(
    alpha = 0.7
  ) +
  geom_smooth(
    method = "lm",
    se = TRUE
  ) +
  labs(
    title = "Relationship Between BMI and Glucose",
    x = "BMI",
    y = "Glucose Level"
  ) +
  theme_minimal()
```

---

# 59. Prediction from a Linear Model

Predict glucose for a patient with BMI = 25:

```r
predict(
  model_1,
  newdata = data.frame(
    bmi = 25
  )
)
```

Confidence interval for the **mean response**:

```r
predict(
  model_1,
  newdata = data.frame(
    bmi = 25
  ),
  interval = "confidence"
)
```

Prediction interval for an **individual observation**:

```r
predict(
  model_1,
  newdata = data.frame(
    bmi = 25
  ),
  interval = "prediction"
)
```

Prediction intervals are wider because they include both uncertainty in the estimated mean and individual variability.

---

# 60. Multiple Linear Regression

```r
model_2 <- lm(
  glucose ~ bmi + age + cholesterol,
  data = patient_data_clean
)

summary(model_2)
```

In multiple regression, each coefficient describes the association with the outcome while the other predictors are held constant.

---

# 61. Regression with Categorical Predictors

```r
model_3 <- lm(
  glucose ~ treatment_group,
  data = patient_data_clean
)

summary(model_3)
```

R automatically creates indicator variables for categorical predictors.

One category becomes the **reference category**.

It is often useful to explicitly convert categorical variables to factors:

```r
patient_data_clean <- patient_data_clean %>%
  mutate(
    treatment_group = factor(treatment_group),
    sex = factor(sex)
  )
```

---

# 62. Continuous and Categorical Predictors Together

```r
model_4 <- lm(
  glucose ~
    bmi +
    age +
    cholesterol +
    treatment_group +
    sex,
  data = patient_data_clean
)

summary(model_4)
```

---

# 63. Linear Regression Assumptions

Important assumptions include:

- independence
- linearity
- constant variance of residuals
- approximately normal residuals

---

## Residuals vs Fitted Values

```r
plot(
  model_4,
  which = 1
)
```

Look for:

- no strong curved pattern
- approximately equal vertical spread

---

## Q-Q Plot of Residuals

```r
plot(
  model_4,
  which = 2
)
```

---

## Shapiro-Wilk Test of Residuals

```r
shapiro.test(
  residuals(model_4)
)
```

Model assumptions should not be evaluated using only one formal test.

---

# 64. Diagnostic Plots

```r
par(
  mfrow = c(2, 2)
)

plot(model_4)

par(
  mfrow = c(1, 1)
)
```

---

# 65. Type II ANOVA for Regression Models

When models contain categorical predictors, it can be useful to test an entire predictor rather than individual dummy coefficients.

Load:

```r
library(car)
```

Then:

```r
Anova(
  model_4,
  type = 2
)
```

---

# 66. Global F-Test

The F-statistic shown by:

```r
summary(model_4)
```

tests the overall hypothesis that all non-intercept regression coefficients are zero.

The corresponding ANOVA table can be viewed using:

```r
anova(model_4)
```

---

# 67. Pairwise Comparisons

Suppose we compare cholesterol between the three age groups.

```r
pairwise.t.test(
  patient_data_clean$cholesterol,
  patient_data_clean$age_group,
  p.adjust.method = "none"
)
```

Bonferroni adjustment:

```r
pairwise.t.test(
  patient_data_clean$cholesterol,
  patient_data_clean$age_group,
  p.adjust.method = "bonferroni"
)
```

Benjamini-Hochberg adjustment:

```r
pairwise.t.test(
  patient_data_clean$cholesterol,
  patient_data_clean$age_group,
  p.adjust.method = "BH"
)
```

Multiple-testing adjustments help account for the fact that several comparisons are being performed.

---

# 68. Interaction Effects

An interaction means that the association between one predictor and the outcome depends on another predictor.

Use `*`:

```r
interaction_model <- lm(
  glucose ~ bmi * treatment_group,
  data = patient_data_clean
)

summary(
  interaction_model
)
```

This is equivalent to including:

```text
bmi
treatment_group
bmi:treatment_group
```

The interaction term tests whether the slope for BMI differs between treatment groups.

---

# 69. Log-Likelihood

```r
logLik(
  model_4
)
```

The likelihood describes how well a statistical model explains the observed data.

---

# 70. Akaike Information Criterion

Calculate AIC:

```r
AIC(
  model_4
)
```

AIC can be used to compare models fitted to the same outcome and observations.

Lower AIC values generally indicate a better balance between model fit and complexity.

For corrected AIC:

```r
library(gamlr)

AICc(
  model_4
)
```

---

# 71. Likelihood Ratio Test

Likelihood-ratio tests compare nested models.

Load:

```r
library(lmtest)
```

Create two models:

```r
reduced_model <- lm(
  glucose ~ bmi + age,
  data = patient_data_clean
)

full_model <- lm(
  glucose ~ bmi + age + cholesterol,
  data = patient_data_clean
)
```

Compare them:

```r
lrtest(
  reduced_model,
  full_model
)
```

---

# Generalized Linear Models

Generalized linear models extend ordinary linear regression to different types of outcomes.

The general structure is:

```r
glm(
  formula,
  family = distribution(link = "link_function"),
  data = dataset
)
```

Common families include:

```text
gaussian   -> continuous outcome
binomial   -> binary outcome
poisson    -> count outcome
```

---

# 72. Logistic Regression

Logistic regression is used when the outcome is binary.

Create:

```r
patient_data_clean <- patient_data_clean %>%
  mutate(
    disease_binary =
      ifelse(
        disease_status == "Disease",
        1,
        0
      )
  )

table(
  patient_data_clean$disease_binary
)
```

Fit a logistic model:

```r
logistic_model <- glm(
  disease_binary ~
    age +
    bmi +
    glucose +
    cholesterol,
  data = patient_data_clean,
  family = binomial(
    link = "logit"
  )
)

summary(
  logistic_model
)
```

---

# 73. Odds Ratios

Logistic regression coefficients are expressed in log-odds.

Convert them to odds ratios:

```r
exp(
  coef(logistic_model)
)
```

Confidence intervals:

```r
exp(
  confint(logistic_model)
)
```

A simple interpretation is:

```text
OR > 1  -> higher odds
OR < 1  -> lower odds
OR = 1  -> no difference in odds
```

Interpretation must always consider the reference group and how the predictor is coded.

---

# 74. Logistic Regression Prediction

Predict the probability of disease for a new patient:

```r
new_patient <- data.frame(
  age = 60,
  bmi = 28,
  glucose = 110,
  cholesterol = 200
)
```

Then:

```r
predict(
  logistic_model,
  newdata = new_patient,
  type = "response"
)
```

Using:

```r
type = "response"
```

returns the predicted probability rather than the logit.

---

# 75. Poisson Regression

Poisson regression is commonly used for count outcomes.

Create a simple simulated count variable:

```r
set.seed(321)

patient_data_clean <- patient_data_clean %>%
  mutate(
    hospital_visits = rpois(
      n(),
      lambda = 2
    )
  )
```

Fit:

```r
poisson_model <- glm(
  hospital_visits ~ age + disease_status,
  data = patient_data_clean,
  family = poisson(
    link = "log"
  )
)

summary(
  poisson_model
)
```

Predict expected counts:

```r
predict(
  poisson_model,
  newdata = data.frame(
    age = 60,
    disease_status = "Disease"
  ),
  type = "response"
)
```

---

# Survival Analysis

Survival analysis is used when the outcome is the **time until an event occurs**.

Examples include:

- time until death
- time until relapse
- time until disease progression
- time until treatment failure

An important feature of survival data is **censoring**.

A censored observation occurs when the exact event time is not observed, for example because the patient is still alive when the study ends.

---

# 76. Load Survival Packages

```r
library(survival)

library(survminer)
```

---

# 77. Create Example Survival Data

```r
survival_data <- data.frame(
  patient_id = 1:20,
  time = c(
    5, 8, 12, 15, 18,
    22, 25, 30, 35, 40,
    6, 10, 14, 20, 23,
    28, 32, 36, 42, 48
  ),
  status = c(
    1, 1, 0, 1, 1,
    0, 1, 0, 1, 0,
    1, 0, 1, 1, 0,
    1, 0, 1, 0, 0
  ),
  treatment = rep(
    c("Control", "Treatment"),
    each = 10
  )
)

survival_data
```

Here:

- `time` is the follow-up time
- `status = 1` means that the event occurred
- `status = 0` means that the observation was censored

---

# 78. Create a Survival Object

Use `Surv()`:

```r
survival_object <- Surv(
  survival_data$time,
  survival_data$status
)

survival_object
```

The survival object combines:

- event time
- censoring information

---

# 79. Kaplan-Meier Curve

Fit one overall survival curve:

```r
km_overall <- survfit(
  survival_object ~ 1,
  data = survival_data
)

km_overall
```

Plot:

```r
ggsurvplot(
  km_overall,
  data = survival_data,
  title = "Kaplan-Meier Survival Curve",
  xlab = "Time",
  ylab = "Survival Probability",
  conf.int = TRUE
)
```

The Kaplan-Meier curve estimates the probability of remaining event-free over time.

---

# 80. Kaplan-Meier Curves by Group

```r
km_group <- survfit(
  Surv(time, status) ~ treatment,
  data = survival_data
)
```

Plot:

```r
ggsurvplot(
  km_group,
  data = survival_data,
  title = "Kaplan-Meier Curves by Treatment",
  xlab = "Time",
  ylab = "Survival Probability",
  pval = TRUE,
  conf.int = TRUE
)
```

The displayed p-value corresponds to a test comparing the survival curves.

---

# 81. Log-Rank Test

The log-rank test compares survival distributions between groups.

```r
survdiff(
  Surv(time, status) ~ treatment,
  data = survival_data,
  rho = 0
)
```

---

# 82. Wilcoxon Survival Test

A weighted Wilcoxon-type survival test can be performed using:

```r
survdiff(
  Surv(time, status) ~ treatment,
  data = survival_data,
  rho = 1
)
```

---

# 83. Cox Proportional Hazards Regression

Cox regression evaluates the association between predictors and the event hazard.

```r
cox_model <- coxph(
  Surv(time, status) ~ treatment,
  data = survival_data
)

summary(
  cox_model
)
```

In the output:

```text
coef
```

is the log hazard ratio.

```text
exp(coef)
```

is the hazard ratio.

A simplified interpretation is:

```text
HR > 1  -> higher hazard
HR < 1  -> lower hazard
HR = 1  -> similar hazard
```

---

# 84. Cox Model with Multiple Predictors

Create an age variable for demonstration:

```r
set.seed(456)

survival_data$age <- round(
  rnorm(
    nrow(survival_data),
    mean = 60,
    sd = 10
  )
)
```

Fit:

```r
cox_model_2 <- coxph(
  Surv(time, status) ~
    treatment +
    age,
  data = survival_data
)

summary(
  cox_model_2
)
```

Each coefficient describes the association with the hazard while accounting for the other variables in the model.

---

# Power and Sample-Size Calculations

Statistical **power** is the probability that a statistical test detects an effect when a real effect exists.

Power depends on:

- sample size
- effect size
- variability
- significance level
- type of statistical test

A commonly used target is:

```text
80% power
```

or:

```text
power = 0.80
```

---

# 85. Power for One Mean

Use `power.t.test()`.

Suppose we want to detect a difference of 3 units with standard deviation 9 using 100 observations:

```r
power.t.test(
  n = 100,
  delta = 3,
  sd = 9,
  sig.level = 0.05,
  power = NULL,
  type = "one.sample",
  alternative = "one.sided"
)
```

Setting:

```r
power = NULL
```

means that R calculates the power.

---

# 86. Sample Size for One Mean

If we want to calculate the required sample size:

```r
power.t.test(
  n = NULL,
  delta = 3,
  sd = 9,
  sig.level = 0.05,
  power = 0.80,
  type = "one.sample",
  alternative = "two.sided"
)
```

Setting:

```r
n = NULL
```

means that R calculates the required sample size.

---

# 87. Sample Size for Two Means

Suppose we want to detect a mean difference of 3 with standard deviation 17.1:

```r
power.t.test(
  n = NULL,
  delta = 3,
  sd = 17.1,
  sig.level = 0.05,
  power = 0.80,
  type = "two.sample",
  alternative = "two.sided"
)
```

For a two-sample test, `n` represents the required sample size **per group**.

---

# 88. Paired-Sample Power

For paired measurements:

```r
power.t.test(
  n = NULL,
  delta = 3,
  sd = 8,
  sig.level = 0.05,
  power = 0.80,
  type = "paired",
  alternative = "two.sided"
)
```

---

# 89. Power for ANOVA

Use `power.anova.test()` when comparing three or more means.

Suppose expected group means are:

```r
group_means <- c(
  150,
  170,
  160,
  185
)
```

Calculate the variance between group means:

```r
var(
  group_means
)
```

Then:

```r
power.anova.test(
  groups = 4,
  n = NULL,
  between.var = var(group_means),
  within.var = 15^2,
  sig.level = 0.05,
  power = 0.80
)
```

The resulting `n` is the approximate number of observations required **per group**.

---

# 90. Power for One Proportion

Load:

```r
library(pwr)
```

Suppose the null proportion is:

```text
0.85
```

and the alternative is:

```text
0.75
```

Calculate Cohen's `h`:

```r
h <- ES.h(
  0.75,
  0.85
)

h
```

Then:

```r
pwr.p.test(
  h = h,
  n = NULL,
  sig.level = 0.05,
  power = 0.90,
  alternative = "less"
)
```

---

# 91. Power for Two Proportions

Suppose:

```text
p1 = 0.60
p2 = 0.40
```

Calculate:

```r
h <- ES.h(
  0.60,
  0.40
)
```

Then calculate power for two groups of 100:

```r
pwr.2p2n.test(
  h = h,
  n1 = 100,
  n2 = 100,
  sig.level = 0.05,
  power = NULL,
  alternative = "two.sided"
)
```

---

# 92. Sample Size for an Odds Ratio

For case-control studies, sample size can also be planned using an expected odds ratio.

Load:

```r
library(epiR)
```

Suppose:

```text
Exposure prevalence among controls = 0.30
Exposure prevalence among cases = 0.46
```

Calculate the expected odds ratio:

```r
p1 <- 0.46

p0 <- 0.30

expected_or <-
  (p1 / (1 - p1)) /
  (p0 / (1 - p0))

expected_or
```

Calculate sample size:

```r
epi.sscc(
  N = NA,
  OR = expected_or,
  p1 = p1,
  p0 = p0,
  n = NA,
  power = 0.80,
  r = 2,
  sided.test = 2,
  conf.level = 0.95
)
```

Here:

- `OR` is the expected odds ratio
- `p1` is expected exposure prevalence among cases
- `p0` is expected exposure prevalence among controls
- `r` is the ratio of controls to cases
- `power` is the desired statistical power

---

# Reporting Statistical Results

# 93. Reporting a t-Test

Example:

> An independent samples t-test was used to compare mean glucose levels between the Control and Treatment groups.

```r
t_test_result <- t.test(
  glucose ~ treatment_group,
  data = patient_data_clean
)

t_test_result
```

When reporting results, include:

- test used
- groups compared
- estimated difference
- confidence interval when relevant
- p-value

---

# 94. Reporting ANOVA

```r
anova_result <- aov(
  cholesterol ~ age_group,
  data = patient_data_clean
)

summary(
  anova_result
)
```

Example:

> A one-way ANOVA was used to compare mean cholesterol levels among age groups.

---

# 95. Reporting Correlation

```r
correlation_result <- cor.test(
  patient_data_clean$bmi,
  patient_data_clean$glucose,
  method = "pearson"
)

correlation_result
```

Example:

> Pearson correlation was used to evaluate the linear association between BMI and glucose.

---

# 96. Reporting Linear Regression

```r
linear_model_result <- summary(
  model_4
)

linear_model_result
```

Example:

> Multiple linear regression was used to evaluate whether age, BMI, cholesterol, treatment group, and sex were associated with glucose level.

---

# 97. Reporting Logistic Regression

```r
summary(
  logistic_model
)

exp(
  coef(logistic_model)
)
```

For logistic regression, effect estimates are commonly reported as **odds ratios**.

---

# 98. Reporting Cox Regression

```r
summary(
  cox_model_2
)
```

For Cox regression, effects are commonly reported as **hazard ratios**.

---

# Exporting Results

# 99. Export Clean Dataset

```r
write.csv(
  patient_data_clean,
  "patient_data_clean.csv",
  row.names = FALSE
)
```

---

# 100. Export Summary Statistics

```r
summary_statistics <- patient_data_clean %>%
  summarise(
    mean_age = mean(age),
    sd_age = sd(age),

    mean_bmi = mean(bmi),
    sd_bmi = sd(bmi),

    mean_glucose = mean(glucose),
    variance_glucose = var(glucose),
    sd_glucose = sd(glucose),
    median_glucose = median(glucose),
    q1_glucose = quantile(
      glucose,
      0.25,
      type = 6
    ),
    q3_glucose = quantile(
      glucose,
      0.75,
      type = 6
    ),
    iqr_glucose = IQR(
      glucose,
      type = 6
    ),

    mean_cholesterol = mean(cholesterol),
    sd_cholesterol = sd(cholesterol),

    median_crp = median(crp),
    iqr_crp = IQR(
      crp,
      type = 6
    )
  )

write.csv(
  summary_statistics,
  "summary_statistics.csv",
  row.names = FALSE
)
```

---

# 101. Export Group Summary

```r
group_summary <- patient_data_clean %>%
  group_by(treatment_group) %>%
  summarise(
    n = n(),
    mean_glucose = mean(glucose),
    sd_glucose = sd(glucose),
    median_glucose = median(glucose),
    iqr_glucose = IQR(glucose),
    .groups = "drop"
  )

write.csv(
  group_summary,
  "group_summary.csv",
  row.names = FALSE
)
```

---

# Practice Exercises

# 102. Descriptive Statistics

Calculate the following for `bmi`:

- mean
- median
- variance
- standard deviation
- minimum
- maximum
- first quartile
- third quartile
- IQR

```r
mean(
  patient_data_clean$bmi
)

median(
  patient_data_clean$bmi
)

var(
  patient_data_clean$bmi
)

sd(
  patient_data_clean$bmi
)

range(
  patient_data_clean$bmi
)

quantile(
  patient_data_clean$bmi,
  probs = c(
    0.25,
    0.75
  ),
  type = 6
)

IQR(
  patient_data_clean$bmi,
  type = 6
)
```

---

# 103. One-Sample t-Test

Test whether the mean cholesterol level differs from 190.

```r
t.test(
  patient_data_clean$cholesterol,
  mu = 190
)
```

---

# 104. Independent t-Test

Compare BMI between males and females.

```r
t.test(
  bmi ~ sex,
  data = patient_data_clean
)
```

---

# 105. Paired t-Test

Create paired measurements:

```r
baseline <- c(
  120, 130, 125, 118,
  140, 135, 128, 132
)

followup <- c(
  115, 124, 121, 116,
  133, 130, 125, 127
)
```

Compare them:

```r
t.test(
  followup,
  baseline,
  paired = TRUE
)
```

---

# 106. Categorical Association

Test whether disease status is associated with sex.

```r
sex_disease_table <- table(
  patient_data_clean$sex,
  patient_data_clean$disease_status
)

chisq.test(
  sex_disease_table
)
```

Check expected counts:

```r
chisq.test(
  sex_disease_table
)$expected
```

---

# 107. Correlation

Test the correlation between age and cholesterol.

```r
cor.test(
  patient_data_clean$age,
  patient_data_clean$cholesterol,
  method = "pearson"
)
```

---

# 108. Linear Regression

Predict cholesterol from age and BMI.

```r
cholesterol_model <- lm(
  cholesterol ~ age + bmi,
  data = patient_data_clean
)

summary(
  cholesterol_model
)
```

Confidence intervals:

```r
confint(
  cholesterol_model
)
```

---

# 109. Logistic Regression

Fit a logistic regression model predicting disease status from age and BMI.

```r
disease_model <- glm(
  disease_binary ~ age + bmi,
  data = patient_data_clean,
  family = binomial
)

summary(
  disease_model
)
```

Odds ratios:

```r
exp(
  coef(disease_model)
)
```

---

# 110. Survival Analysis

Fit Kaplan-Meier curves by treatment.

```r
survival_fit <- survfit(
  Surv(time, status) ~ treatment,
  data = survival_data
)

ggsurvplot(
  survival_fit,
  data = survival_data,
  pval = TRUE
)
```

---

# 111. Cox Regression

```r
cox_exercise <- coxph(
  Surv(time, status) ~ treatment + age,
  data = survival_data
)

summary(
  cox_exercise
)
```

---

# 112. Power Calculation

Calculate the sample size required to detect a mean difference of 5 when:

```text
SD = 10
Power = 80%
Alpha = 0.05
```

```r
power.t.test(
  n = NULL,
  delta = 5,
  sd = 10,
  sig.level = 0.05,
  power = 0.80,
  type = "two.sample",
  alternative = "two.sided"
)
```

---

# Common Statistical Tests in R

| Research Question | Common Test | R Function |
|---|---|---|
| Describe a continuous variable | Mean, median, SD, IQR | `mean()`, `median()`, `sd()`, `IQR()` |
| Confidence interval for one mean | One-sample t procedure | `t.test()` |
| Test one population mean | One-sample t-test | `t.test()` |
| Confidence interval/test for one proportion | Approximate proportion test | `prop.test()` |
| Exact test for one proportion | Exact binomial test | `binom.test()` |
| Compare two independent means | Independent t-test | `t.test()` |
| Compare two paired means | Paired t-test | `t.test(..., paired = TRUE)` |
| Compare two variances | F-test | `var.test()` |
| Compare two independent non-normal groups | Wilcoxon rank-sum | `wilcox.test()` |
| Compare paired non-normal measurements | Wilcoxon signed-rank | `wilcox.test(..., paired = TRUE)` |
| Sign-based paired comparison | Sign test | `SIGN.test()` |
| Compare 3+ means | ANOVA | `aov()` |
| Compare 3+ non-normal groups | Kruskal-Wallis | `kruskal.test()` |
| Compare two proportions | Two-sample proportion test | `prop.test()` |
| Test two categorical variables | Chi-square | `chisq.test()` |
| Small categorical table | Fisher's exact test | `fisher.test()` |
| Test normality | Shapiro-Wilk | `shapiro.test()` |
| Linear association | Pearson correlation | `cor.test()` |
| Rank-based association | Spearman correlation | `cor.test(method = "spearman")` |
| Predict continuous outcome | Linear regression | `lm()` |
| Predict binary outcome | Logistic regression | `glm(family = binomial)` |
| Model count outcome | Poisson regression | `glm(family = poisson)` |
| Kaplan-Meier survival estimate | Kaplan-Meier | `survfit()` |
| Compare survival curves | Log-rank test | `survdiff()` |
| Model survival outcome | Cox regression | `coxph()` |
| Power/sample size for t-test | Power analysis | `power.t.test()` |
| Power/sample size for ANOVA | ANOVA power | `power.anova.test()` |
| Power for proportions | Proportion power | `pwr.p.test()`, `pwr.2p2n.test()` |
| Case-control sample size | Odds-ratio-based calculation | `epi.sscc()` |

---

# Choosing Between Common Tests

A simple decision process is:

```text
What type of outcome do you have?
```

For a **continuous outcome**:

```text
One group
    |
    +-- Mean -> one-sample t-test
    |
    +-- Non-parametric -> signed-rank or sign test
```

```text
Two groups
    |
    +-- Independent -> independent t-test
    |
    +-- Paired -> paired t-test
    |
    +-- Non-parametric independent -> Wilcoxon rank-sum
    |
    +-- Non-parametric paired -> Wilcoxon signed-rank
```

```text
Three or more groups
    |
    +-- Parametric -> ANOVA
    |
    +-- Non-parametric -> Kruskal-Wallis
```

For a **categorical outcome**:

```text
One proportion
    |
    +-- Large sample -> prop.test()
    |
    +-- Small sample -> binom.test()
```

```text
Two categorical variables
    |
    +-- Adequate expected counts -> chisq.test()
    |
    +-- Small expected counts -> fisher.test()
```

For a **continuous relationship**:

```text
Correlation -> cor.test()

Prediction -> lm()
```

For a **binary outcome**:

```text
Logistic regression -> glm(..., family = binomial)
```

For **count data**:

```text
Poisson regression -> glm(..., family = poisson)
```

For **time-to-event data**:

```text
Kaplan-Meier -> survfit()

Compare curves -> survdiff()

Multiple predictors -> coxph()
```

---

# Interpretation of p-Values

A p-value measures evidence against the null hypothesis.

A commonly used threshold is:

```text
0.05
```

A simple interpretation is:

- p-value < 0.05: statistically significant at the 5% level
- p-value >= 0.05: not statistically significant at the 5% level

However, statistical significance does not automatically mean that the result is:

- biologically important
- clinically important
- large in magnitude

Always consider:

- effect size
- confidence interval
- sample size
- study design
- biological or clinical context

---

# Confidence Intervals vs p-Values

A p-value answers a question about statistical evidence.

A confidence interval gives information about:

- the estimated effect
- the direction of the effect
- the uncertainty around the effect

For example:

```text
Estimated difference = 5

95% CI = 1 to 9
```

contains more information than simply reporting:

```text
p < 0.05
```

Whenever possible, report both effect estimates and confidence intervals.

---

# Multiple Testing

When many statistical tests are performed, the probability of false-positive results increases.

Common adjustment methods include:

```r
p.adjust(
  c(
    0.001,
    0.02,
    0.04,
    0.20
  ),
  method = "bonferroni"
)
```

and:

```r
p.adjust(
  c(
    0.001,
    0.02,
    0.04,
    0.20
  ),
  method = "BH"
)
```

This topic becomes especially important in genomic and transcriptomic analyses where thousands of statistical tests may be performed simultaneously.

---

# Good Statistical Practice

When performing statistical analysis:

- define the research question before choosing the test
- identify the type of outcome and predictor variables
- inspect and clean the data
- visualize the data
- check statistical assumptions
- distinguish paired and independent observations
- use exact methods when sample sizes are too small for approximations
- report effect sizes when possible
- report confidence intervals
- do not rely only on p-values
- adjust for multiple testing when appropriate
- interpret results in biological or clinical context
- document the complete analysis for reproducibility

---

# Session Information

It is good practice to record information about the R environment used for the analysis.

```r
sessionInfo()
```

This shows information such as:

- R version
- operating system
- loaded packages
- package versions

This is useful for reproducibility.

---

# Summary

In this tutorial, you learned how to perform a broad range of statistical analyses in R.

You covered:

- descriptive statistics
- mean and median
- variance and standard deviation
- minimum, maximum, and range
- quartiles and percentiles
- interquartile range
- frequency tables and proportions
- binomial, multinomial, normal, chi-square, and t distributions
- confidence intervals for means and proportions
- one-sample hypothesis tests
- independent and paired t-tests
- tests for variances
- one- and two-proportion tests
- sign tests
- Wilcoxon signed-rank and rank-sum tests
- ANOVA
- Kruskal-Wallis tests
- normality testing
- contingency tables
- chi-square tests
- Fisher's exact tests
- Pearson and Spearman correlation
- simple linear regression
- multiple linear regression
- categorical predictors
- regression confidence intervals
- prediction intervals
- regression assumptions
- Type II ANOVA
- global F-tests
- pairwise comparisons
- interaction effects
- log-likelihood
- AIC and AICc
- likelihood-ratio tests
- logistic regression
- Poisson regression
- Kaplan-Meier survival analysis
- log-rank tests
- Cox regression
- statistical power
- sample-size calculations for means
- sample-size calculations for ANOVA
- power calculations for proportions
- case-control sample-size calculations based on odds ratios

This session provides a statistical foundation for later biomedical, genomics, and transcriptomics analyses in R.
