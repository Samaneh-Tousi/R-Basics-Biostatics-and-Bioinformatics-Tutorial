# Basic Biostatistics in R

## Introduction

Biostatistics helps us describe biomedical data and answer research questions using statistical methods.

In practice, statistical analysis usually follows a simple workflow:

1. Understand the research question.
2. Identify the type of variables.
3. Explore and summarize the data.
4. Visualize the data.
5. Choose an appropriate statistical test or model.
6. Check assumptions.
7. Interpret the effect size, confidence interval, and p-value.
8. Report the result in the biological or clinical context.

In this tutorial, we will focus on statistical methods that are commonly used in biomedical research.

You will learn how to:

- summarize continuous and categorical variables
- understand standard deviation, confidence intervals, and p-values
- compare two groups
- compare more than two groups
- analyze paired measurements
- use non-parametric alternatives
- analyze categorical variables
- perform correlation analysis
- fit linear regression models
- fit logistic regression models
- perform basic survival analysis
- calculate statistical power and sample size

---

# 1. Install and Load Packages

Install packages only once:

```r
install.packages(c(
  "tidyverse",
  "rstatix",
  "survival",
  "survminer"
))
```

Load them when starting a new R session:

```r
library(tidyverse)
library(rstatix)
library(survival)
library(survminer)
```

---

# 2. Create an Example Biomedical Dataset

We will use a simulated patient dataset.

```r
set.seed(123)

patient_data <- tibble(
  patient_id = 1:120,
  age = round(rnorm(120, mean = 52, sd = 12)),
  sex = sample(c("Female", "Male"), 120, replace = TRUE),
  treatment_group = rep(c("Control", "Treatment"), each = 60),
  bmi = round(rnorm(120, mean = 27, sd = 4), 1),
  glucose = c(
    rnorm(60, mean = 110, sd = 18),
    rnorm(60, mean = 101, sd = 18)
  ),
  cholesterol = round(rnorm(120, mean = 190, sd = 35), 1),
  crp = round(rlnorm(120, meanlog = 1.2, sdlog = 0.6), 2),
  disease_status = sample(
    c("Healthy", "Disease"),
    120,
    replace = TRUE
  )
)

head(patient_data)
```

The dataset contains both **continuous variables** and **categorical variables**.

Continuous variables include:

- age
- BMI
- glucose
- cholesterol
- CRP

Categorical variables include:

- sex
- treatment group
- disease status

Knowing the type of variable is important because it helps determine which statistical method should be used.

---

# 3. Inspect the Dataset

Before calculating statistics, always inspect the data.

```r
str(patient_data)
```

Check the dimensions:

```r
dim(patient_data)
```

Check the variable names:

```r
names(patient_data)
```

View a general summary:

```r
summary(patient_data)
```

These functions help answer basic questions such as:

- How many observations are there?
- Which variables are available?
- Are variables stored as numeric or categorical?
- Are there unusual values?

---

# 4. Descriptive Statistics

Descriptive statistics summarize the data before statistical testing.

For a continuous variable, commonly reported statistics include:

- mean
- median
- standard deviation
- minimum and maximum
- quartiles
- interquartile range

---

## 4.1 Mean and Median

The **mean** is the arithmetic average.

```r
mean(patient_data$glucose)
```

The **median** is the middle value after sorting the observations.

```r
median(patient_data$glucose)
```

The mean is useful for approximately symmetric data.

The median is often more informative when the data are strongly skewed or contain extreme values.

For example, CRP is often right-skewed:

```r
mean(patient_data$crp)

median(patient_data$crp)
```

---

## 4.2 Standard Deviation and Variance

The **standard deviation** describes how much observations vary around the mean.

```r
sd(patient_data$glucose)
```

The **variance** is the squared standard deviation:

```r
var(patient_data$glucose)
```

A larger standard deviation means that the observations are more spread out.

---

## 4.3 Minimum, Maximum, and Range

```r
min(patient_data$glucose)

max(patient_data$glucose)

range(patient_data$glucose)
```

`range()` returns the minimum and maximum values.

---

## 4.4 Quartiles and Interquartile Range

Quartiles divide ordered data into four parts.

```r
quantile(
  patient_data$glucose,
  probs = c(0.25, 0.50, 0.75)
)
```

These values correspond to:

- Q1: 25th percentile
- Q2: median
- Q3: 75th percentile

The **interquartile range (IQR)** describes the middle 50% of the data.

```r
IQR(patient_data$glucose)
```

Median and IQR are commonly reported for skewed variables.

For example:

```r
median(patient_data$crp)

IQR(patient_data$crp)
```

---

## 4.5 Summarize Several Statistics Together

```r
patient_data %>%
  summarise(
    n = n(),
    mean_glucose = mean(glucose),
    sd_glucose = sd(glucose),
    median_glucose = median(glucose),
    q1 = quantile(glucose, 0.25),
    q3 = quantile(glucose, 0.75),
    min_glucose = min(glucose),
    max_glucose = max(glucose)
  )
```

---

# 5. Summarizing Categorical Variables

For categorical variables, we usually report:

- number of observations
- proportion
- percentage

Create a frequency table:

```r
table(patient_data$sex)
```

Calculate proportions:

```r
prop.table(
  table(patient_data$sex)
)
```

Calculate percentages:

```r
prop.table(
  table(patient_data$sex)
) * 100
```

Using `dplyr`:

```r
patient_data %>%
  count(sex) %>%
  mutate(
    percentage = n / sum(n) * 100
  )
```

---

# 6. Always Visualize the Data

Statistical tests should not replace looking at the data.

For a continuous variable, a histogram can show the shape of the distribution.

```r
ggplot(patient_data, aes(x = glucose)) +
  geom_histogram(
    bins = 25,
    color = "black",
    fill = "lightblue"
  ) +
  theme_minimal()
```

A boxplot is useful for comparing groups.

```r
ggplot(
  patient_data,
  aes(
    x = treatment_group,
    y = glucose,
    fill = treatment_group
  )
) +
  geom_boxplot() +
  geom_jitter(
    width = 0.15,
    alpha = 0.6
  ) +
  theme_minimal()
```

Before running a statistical test, ask:

> Does the plot support the assumptions I am about to make?

---

# 7. Understanding Confidence Intervals and p-Values

Before learning statistical tests, it is useful to understand two important concepts.

## Confidence Interval

A **confidence interval** gives a range of plausible values for a population parameter.

For example:

```text
Mean difference = 8.2

95% CI = 2.1 to 14.3
```

The estimated difference is 8.2, but there is uncertainty around that estimate.

In biomedical research, confidence intervals are important because they show both:

- the size of the estimated effect
- the uncertainty around the estimate

---

## p-Value

A p-value is calculated under the assumption that the **null hypothesis** is true.

A commonly used threshold is:

```text
0.05
```

A simple interpretation is:

- `p < 0.05`: evidence against the null hypothesis
- `p >= 0.05`: insufficient evidence to reject the null hypothesis

However:

> A small p-value does not automatically mean that an effect is biologically or clinically important.

Always consider the estimated effect and its confidence interval.

---

# 8. Confidence Interval for a Mean

Suppose we want a 95% confidence interval for the mean glucose level.

```r
t.test(patient_data$glucose)
```

The output contains:

- estimated mean
- 95% confidence interval
- t statistic
- p-value

For a different confidence level:

```r
t.test(
  patient_data$glucose,
  conf.level = 0.99
)
```

---

# 9. Checking the Distribution

Several statistical tests work best when the data within groups are reasonably compatible with a normal distribution.

We can investigate this using plots.

---

## 9.1 Histogram

```r
ggplot(patient_data, aes(x = glucose)) +
  geom_histogram(
    bins = 25,
    color = "black"
  ) +
  theme_minimal()
```

---

## 9.2 Q-Q Plot

```r
ggplot(
  patient_data,
  aes(sample = glucose)
) +
  stat_qq() +
  stat_qq_line() +
  theme_minimal()
```

If the points approximately follow the line, the distribution is reasonably compatible with normality.

---

## 9.3 Shapiro-Wilk Test

```r
shapiro.test(patient_data$glucose)
```

A common interpretation is:

- `p > 0.05`: no strong evidence against normality
- `p <= 0.05`: evidence that the distribution differs from normality

Do not use the Shapiro-Wilk test alone.

Always also inspect the distribution visually.

---

# 10. Comparing Two Independent Groups

One of the most common biomedical questions is:

> Is a continuous measurement different between two groups?

For example:

> Is glucose different between the Control and Treatment groups?

---

## 10.1 Independent Samples t-Test

Use a t-test when comparing the means of two **independent groups**.

```r
t.test(
  glucose ~ treatment_group,
  data = patient_data
)
```

By default, R performs **Welch's t-test**, which does not require equal variances.

Important information in the output includes:

- mean in each group
- estimated difference
- confidence interval
- p-value

Visualize the comparison:

```r
ggplot(
  patient_data,
  aes(
    x = treatment_group,
    y = glucose,
    fill = treatment_group
  )
) +
  geom_boxplot() +
  geom_jitter(
    width = 0.15,
    alpha = 0.6
  ) +
  theme_minimal()
```

---

## 10.2 Effect Size

A statistically significant difference may still be very small.

Effect size helps describe the magnitude of the difference.

For a two-group comparison, Cohen's d can be calculated using:

```r
patient_data %>%
  cohens_d(
    glucose ~ treatment_group
  )
```

Effect size should be interpreted together with the confidence interval and clinical context.

---

# 11. Paired t-Test

A **paired test** is used when the two measurements come from the same individuals.

Examples include:

- blood pressure before and after treatment
- gene expression before and after stimulation
- measurements from matched patients

Create example data:

```r
before <- c(
  125, 138, 142, 130, 150,
  145, 135, 128, 160, 140
)

after <- c(
  118, 130, 137, 122, 143,
  139, 130, 120, 150, 135
)
```

Run the paired t-test:

```r
t.test(
  after,
  before,
  paired = TRUE
)
```

The important point is that observation 1 in `before` belongs to the same person as observation 1 in `after`.

Do not use an independent t-test for paired measurements.

---

# 12. Non-Parametric Alternative for Two Groups

Sometimes the data are strongly skewed, contain influential outliers, or are not suitable for a mean-based comparison.

For two independent groups, a common alternative is the **Wilcoxon rank-sum test**.

```r
wilcox.test(
  crp ~ treatment_group,
  data = patient_data
)
```

For paired data:

```r
wilcox.test(
  after,
  before,
  paired = TRUE
)
```

A useful simplified guide is:

| Situation | Common Test |
|---|---|
| Two independent groups, approximately normal continuous outcome | Independent t-test |
| Two paired measurements, approximately normal differences | Paired t-test |
| Two independent groups, strongly non-normal/ordinal outcome | Wilcoxon rank-sum |
| Two paired measurements, strongly non-normal differences | Wilcoxon signed-rank |

---

# 13. Comparing More Than Two Groups

Suppose we want to compare a continuous variable among three or more groups.

First create age groups:

```r
patient_data <- patient_data %>%
  mutate(
    age_group = case_when(
      age < 40 ~ "Young",
      age < 60 ~ "Middle-aged",
      TRUE ~ "Older"
    )
  )

table(patient_data$age_group)
```

---

## 13.1 One-Way ANOVA

ANOVA tests whether the group means are all equal.

Example:

> Does cholesterol differ among age groups?

```r
anova_model <- aov(
  cholesterol ~ age_group,
  data = patient_data
)

summary(anova_model)
```

The ANOVA p-value answers:

> Is there evidence that at least one group mean differs?

It does **not** tell us which groups differ.

---

## 13.2 Post-Hoc Test

If the ANOVA is significant, we can compare pairs of groups.

```r
TukeyHSD(anova_model)
```

Tukey's method adjusts for performing multiple comparisons.

---

## 13.3 Kruskal-Wallis Test

For strongly non-normal or ordinal outcomes, a common alternative is:

```r
kruskal.test(
  cholesterol ~ age_group,
  data = patient_data
)
```

---

# 14. Association Between Categorical Variables

Suppose both variables are categorical.

Example:

> Is disease status associated with treatment group?

Create a contingency table:

```r
disease_table <- table(
  patient_data$disease_status,
  patient_data$treatment_group
)

disease_table
```

---

## 14.1 Chi-Square Test

```r
chi_result <- chisq.test(
  disease_table
)

chi_result
```

The null hypothesis is that the two categorical variables are independent.

Check the expected cell counts:

```r
chi_result$expected
```

---

## 14.2 Fisher's Exact Test

If some expected cell counts are very small, Fisher's exact test is often preferable.

```r
fisher.test(
  disease_table
)
```

A practical rule is:

- adequate expected counts → chi-square test
- small/sparse tables → Fisher's exact test

---

# 15. Comparing Two Proportions

Suppose:

- 40 of 100 patients in group A experienced an event
- 25 of 100 patients in group B experienced an event

Use:

```r
prop.test(
  x = c(40, 25),
  n = c(100, 100)
)
```

This compares the two population proportions.

---

# 16. Correlation

Correlation measures the strength of association between two variables.

Example:

> Is BMI associated with glucose?

Always visualize the relationship first.

```r
ggplot(
  patient_data,
  aes(
    x = bmi,
    y = glucose
  )
) +
  geom_point() +
  geom_smooth(
    method = "lm",
    se = TRUE
  ) +
  theme_minimal()
```

---

## 16.1 Pearson Correlation

Pearson correlation measures **linear association**.

```r
cor.test(
  patient_data$bmi,
  patient_data$glucose,
  method = "pearson"
)
```

The correlation coefficient `r` ranges between:

```text
-1 and +1
```

Approximately:

- positive `r` → variables increase together
- negative `r` → one increases while the other decreases
- `r` close to 0 → little linear association

---

## 16.2 Spearman Correlation

Spearman correlation is based on ranks and is useful for monotonic relationships or variables with strong skewness/outliers.

```r
cor.test(
  patient_data$bmi,
  patient_data$crp,
  method = "spearman"
)
```

Important:

> Correlation does not demonstrate causation.

---

# 17. Linear Regression

Correlation describes association.

**Linear regression** goes further by modeling how a continuous outcome changes with one or more predictors.

Example:

> How does glucose change with BMI?

---

## 17.1 Simple Linear Regression

```r
model_1 <- lm(
  glucose ~ bmi,
  data = patient_data
)

summary(model_1)
```

The model can be written conceptually as:

```text
glucose = intercept + slope × BMI
```

The important coefficient is the estimate for `bmi`.

For example, an estimate of:

```text
2.1
```

would mean that glucose is estimated to increase by approximately 2.1 units for every one-unit increase in BMI.

---

## 17.2 Confidence Intervals

```r
confint(model_1)
```

Confidence intervals help show how precisely the regression coefficients are estimated.

---

## 17.3 Multiple Linear Regression

Biomedical outcomes usually depend on more than one variable.

For example:

```r
model_2 <- lm(
  glucose ~ bmi + age + sex,
  data = patient_data
)

summary(model_2)
```

Each coefficient is interpreted while **holding the other predictors constant**.

This is one reason regression models are widely used in biomedical research.

---

## 17.4 Regression Assumptions

Important assumptions include:

- observations are independent
- the relationship is reasonably linear
- residual variability is approximately constant
- residuals are reasonably normally distributed

R provides useful diagnostic plots:

```r
par(mfrow = c(2, 2))

plot(model_2)

par(mfrow = c(1, 1))
```

Pay particular attention to:

- Residuals vs Fitted
- Normal Q-Q

---

# 18. Logistic Regression

Linear regression is used when the outcome is continuous.

**Logistic regression** is commonly used when the outcome has two categories.

Examples include:

- disease / healthy
- dead / alive
- responder / non-responder
- positive / negative

---

## 18.1 Prepare a Binary Outcome

```r
patient_data <- patient_data %>%
  mutate(
    disease_binary =
      ifelse(
        disease_status == "Disease",
        1,
        0
      )
  )
```

Check the coding:

```r
table(
  patient_data$disease_binary
)
```

Here:

```text
0 = Healthy
1 = Disease
```

---

## 18.2 Fit the Logistic Regression Model

```r
logistic_model <- glm(
  disease_binary ~ age + bmi + glucose,
  data = patient_data,
  family = binomial
)

summary(logistic_model)
```

The raw regression coefficients are expressed in **log-odds**, which are difficult to interpret directly.

---

## 18.3 Odds Ratios

Convert the coefficients to odds ratios:

```r
exp(
  coef(logistic_model)
)
```

Confidence intervals for the odds ratios:

```r
exp(
  confint(logistic_model)
)
```

A simplified interpretation is:

- OR > 1 → higher odds of the outcome
- OR < 1 → lower odds of the outcome
- OR = 1 → no association with the odds

For example:

```text
OR = 1.20
```

means that the odds are multiplied by `1.20` for a one-unit increase in that predictor, assuming the other predictors remain constant.

---

# 19. Survival Analysis

Some biomedical outcomes are not simply yes/no.

Instead, we are interested in:

> How long does it take until an event occurs?

Examples include:

- time until death
- time until relapse
- time until disease progression
- time until treatment failure

This is called **time-to-event** or **survival analysis**.

---

## 19.1 What Is Censoring?

Suppose a patient is still alive when the study finishes.

We know that the patient survived at least until the end of follow-up, but we do not know their exact future survival time.

This observation is called **censored**.

Survival analysis can use both:

- patients who experienced the event
- patients who were censored

---

## 19.2 Create Example Survival Data

```r
survival_data <- data.frame(
  time = c(
    5, 8, 12, 15, 18,
    22, 25, 30, 35, 40,
    8, 12, 18, 22, 28,
    32, 38, 42, 48, 52
  ),
  status = c(
    1, 1, 1, 0, 1,
    1, 0, 1, 0, 1,
    1, 0, 1, 0, 1,
    0, 1, 0, 0, 0
  ),
  treatment = rep(
    c("Control", "Treatment"),
    each = 10
  )
)
```

Here:

```text
status = 1 → event occurred
status = 0 → censored
```

---

## 19.3 Kaplan-Meier Curve

First create a survival object:

```r
survival_object <- Surv(
  time = survival_data$time,
  event = survival_data$status
)
```

Fit Kaplan-Meier curves:

```r
km_fit <- survfit(
  survival_object ~ treatment,
  data = survival_data
)
```

Plot:

```r
ggsurvplot(
  km_fit,
  data = survival_data,
  conf.int = TRUE,
  risk.table = TRUE,
  pval = TRUE
)
```

The Kaplan-Meier curve estimates the probability of remaining event-free over time.

---

## 19.4 Log-Rank Test

To formally compare survival curves:

```r
survdiff(
  Surv(time, status) ~ treatment,
  data = survival_data
)
```

The null hypothesis is that the survival experience is the same between groups.

---

## 19.5 Cox Regression

Cox regression allows us to study survival while including predictors.

```r
cox_model <- coxph(
  Surv(time, status) ~ treatment,
  data = survival_data
)

summary(cox_model)
```

The most important effect measure is the **hazard ratio (HR)**.

A simplified interpretation is:

- HR > 1 → higher event hazard
- HR < 1 → lower event hazard
- HR = 1 → similar event hazard

As with an odds ratio, interpretation depends on the reference group.

---

# 20. Statistical Power and Sample Size

Before starting a study, researchers should ask:

> How many participants are needed?

A study with too few participants may have insufficient power to detect an important effect.

**Statistical power** is the probability of detecting an effect when the effect truly exists.

A commonly used target is:

```text
80% power
```

or:

```text
power = 0.80
```

Power depends on:

- sample size
- expected effect size
- variability
- significance level

---

## 20.1 Sample Size for Comparing Two Means

Suppose we want to detect a difference of:

```text
5 units
```

with an expected standard deviation of:

```text
10 units
```

using:

```text
alpha = 0.05
power = 0.80
```

Use:

```r
power.t.test(
  n = NULL,
  delta = 5,
  sd = 10,
  sig.level = 0.05,
  power = 0.80,
  type = "two.sample"
)
```

`n = NULL` tells R:

> Calculate the required sample size.

For a two-sample test, the returned `n` is approximately the required number of participants **per group**.

---

## 20.2 Calculate Power When Sample Size Is Known

Suppose we already plan to include 50 participants per group.

```r
power.t.test(
  n = 50,
  delta = 5,
  sd = 10,
  sig.level = 0.05,
  power = NULL,
  type = "two.sample"
)
```

`power = NULL` tells R to calculate the expected power.

---

## 20.3 Sample Size for Comparing Two Proportions

Suppose we expect:

```text
Group 1: 40% response
Group 2: 60% response
```

Use:

```r
power.prop.test(
  n = NULL,
  p1 = 0.40,
  p2 = 0.60,
  sig.level = 0.05,
  power = 0.80
)
```

Again, the returned `n` is approximately the sample size required per group.

---

## 20.4 Sample Size for ANOVA

For three or more groups, R provides:

```r
power.anova.test()
```

For example:

```r
power.anova.test(
  groups = 3,
  n = NULL,
  between.var = 25,
  within.var = 100,
  sig.level = 0.05,
  power = 0.80
)
```

In real studies, expected effect sizes and variability should ideally come from:

- previous studies
- pilot data
- clinically meaningful differences

---

# 21. Multiple Testing

In biomedical and especially genomic research, researchers may perform many statistical tests.

The more tests we perform, the greater the chance of obtaining false-positive results.

For example:

```r
p_values <- c(
  0.001,
  0.01,
  0.03,
  0.08,
  0.20
)
```

Bonferroni adjustment:

```r
p.adjust(
  p_values,
  method = "bonferroni"
)
```

Benjamini-Hochberg adjustment:

```r
p.adjust(
  p_values,
  method = "BH"
)
```

The Benjamini-Hochberg method controls the **false discovery rate** and is especially important in analyses such as transcriptomics where thousands of genes may be tested.

---

# 22. Which Statistical Test Should I Use?

A useful first question is:

> What type of outcome variable do I have?

| Research Question | Outcome | Common Method |
|---|---|---|
| Describe a continuous variable | Continuous | Mean/SD or Median/IQR |
| Compare two independent groups | Continuous | Independent t-test |
| Compare paired measurements | Continuous | Paired t-test |
| Compare two non-normal independent groups | Continuous/ordinal | Wilcoxon rank-sum |
| Compare 3+ groups | Continuous | ANOVA |
| Compare 3+ non-normal groups | Continuous/ordinal | Kruskal-Wallis |
| Compare two proportions | Categorical | `prop.test()` |
| Association between categorical variables | Categorical | Chi-square |
| Small categorical table | Categorical | Fisher's exact |
| Association between two continuous variables | Continuous | Pearson/Spearman correlation |
| Predict a continuous outcome | Continuous | Linear regression |
| Predict a binary outcome | Binary | Logistic regression |
| Analyze time until an event | Time-to-event | Kaplan-Meier / Cox regression |

This table is a starting point.

The final choice also depends on:

- study design
- independence or pairing
- sample size
- data distribution
- statistical assumptions
- research question

---

# 23. A Practical Statistical Workflow

For most biomedical analyses, a good workflow is:

### Step 1: Define the question

For example:

```text
Is glucose different between treatment groups?
```

### Step 2: Identify the variables

```text
Outcome: glucose → continuous

Predictor: treatment group → categorical with 2 groups
```

### Step 3: Explore the data

```r
summary(patient_data$glucose)
```

Visualize:

```r
ggplot(
  patient_data,
  aes(
    x = treatment_group,
    y = glucose
  )
) +
  geom_boxplot() +
  geom_jitter(
    width = 0.15
  )
```

### Step 4: Choose the test

Two independent groups with a continuous outcome:

```r
t.test(
  glucose ~ treatment_group,
  data = patient_data
)
```

### Step 5: Interpret more than the p-value

Look at:

- group means
- difference between groups
- confidence interval
- p-value
- effect size

### Step 6: Interpret biologically

Ask:

> Is the observed difference large enough to matter biologically or clinically?

---

# 24. Practice Exercises

## Exercise 1

Calculate the following for `glucose`:

- mean
- median
- standard deviation
- first quartile
- third quartile
- IQR

```r
mean(patient_data$glucose)

median(patient_data$glucose)

sd(patient_data$glucose)

quantile(
  patient_data$glucose,
  c(0.25, 0.75)
)

IQR(patient_data$glucose)
```

---

## Exercise 2

Compare glucose between the Control and Treatment groups.

First visualize the data:

```r
ggplot(
  patient_data,
  aes(
    x = treatment_group,
    y = glucose
  )
) +
  geom_boxplot() +
  geom_jitter(
    width = 0.15
  )
```

Then perform the test:

```r
t.test(
  glucose ~ treatment_group,
  data = patient_data
)
```

---

## Exercise 3

Compare CRP between treatment groups using a non-parametric test.

```r
wilcox.test(
  crp ~ treatment_group,
  data = patient_data
)
```

---

## Exercise 4

Test whether disease status is associated with sex.

```r
disease_sex_table <- table(
  patient_data$disease_status,
  patient_data$sex
)

chisq.test(
  disease_sex_table
)
```

Check expected counts:

```r
chisq.test(
  disease_sex_table
)$expected
```

---

## Exercise 5

Test the correlation between BMI and glucose.

```r
cor.test(
  patient_data$bmi,
  patient_data$glucose,
  method = "pearson"
)
```

---

## Exercise 6

Fit a multiple linear regression model predicting glucose from:

- BMI
- age
- sex

```r
glucose_model <- lm(
  glucose ~ bmi + age + sex,
  data = patient_data
)

summary(glucose_model)
```

---

## Exercise 7

Fit a logistic regression model predicting disease status from age and BMI.

```r
disease_model <- glm(
  disease_binary ~ age + bmi,
  data = patient_data,
  family = binomial
)

summary(disease_model)
```

Calculate odds ratios:

```r
exp(
  coef(disease_model)
)
```

---

## Exercise 8

Calculate the sample size required to detect a difference of 5 units between two groups when:

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
  type = "two.sample"
)
```

---

# 25. Interpreting Statistical Results

When reading statistical output, do not search only for:

```text
p < 0.05
```

Instead, ask:

1. What is the estimated effect?
2. In which direction is the effect?
3. How large is the effect?
4. What is the confidence interval?
5. Is the result statistically convincing?
6. Is it biologically or clinically meaningful?

For regression models, pay attention to the effect measure:

| Model | Common Effect Measure |
|---|---|
| Linear regression | Regression coefficient |
| Logistic regression | Odds ratio |
| Cox regression | Hazard ratio |

---

# 26. Good Statistical Practice

When performing biostatistical analysis:

- define the research question before running a test
- identify the outcome and predictor variables
- distinguish independent from paired observations
- explore and visualize the data first
- check important assumptions
- report descriptive statistics
- report effect sizes and confidence intervals when possible
- do not interpret statistical significance as clinical importance
- use multiple-testing correction when performing many tests
- calculate sample size before collecting data when possible
- interpret the result in the context of the study design

---

# 27. Session Information

At the end of an analysis, record the R environment:

```r
sessionInfo()
```

This helps make the analysis reproducible.

---

# Summary

In this tutorial, you learned the statistical methods most commonly encountered in biomedical research.

You learned how to:

- describe continuous data using mean, median, SD, quartiles, and IQR
- summarize categorical data using counts and percentages
- visualize data before statistical testing
- understand confidence intervals and p-values
- compare two independent groups
- analyze paired measurements
- use Wilcoxon tests for non-parametric comparisons
- compare three or more groups using ANOVA or Kruskal-Wallis
- analyze categorical variables using chi-square and Fisher's exact tests
- compare proportions
- calculate Pearson and Spearman correlations
- fit simple and multiple linear regression models
- fit logistic regression models and interpret odds ratios
- analyze time-to-event data using Kaplan-Meier curves and Cox regression
- understand censoring and hazard ratios
- calculate basic statistical power and sample size
- correct p-values when performing multiple statistical tests

The most important skill is not memorizing R commands.

It is learning to connect:

```text
Research question
        ↓
Variable types
        ↓
Study design
        ↓
Appropriate statistical method
        ↓
Interpretation
```

This approach provides a strong foundation for biomedical, clinical, and transcriptomics analyses in R.