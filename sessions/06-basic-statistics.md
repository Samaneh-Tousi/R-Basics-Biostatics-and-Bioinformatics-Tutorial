# Basic Statistics in R

## Overview

This tutorial introduces basic statistical analysis in **R** using biomedical-style example data.

You will learn how to:

- create and inspect a dataset
- calculate descriptive statistics
- summarize continuous and categorical variables
- visualize data distributions
- check normality
- compare groups
- perform correlation analysis
- run simple linear regression
- export statistical results

---

## 1. Prerequisites

Before starting, make sure R and RStudio are installed.

You only need to install packages **once** on your computer.  
After installation, you only need to load them using `library()`.

Reinstalling already installed packages repeatedly is not necessary and may sometimes cause version or dependency issues.

---

## 2. Install and Load Required Packages

```r
# Install packages only once
install.packages(c(
  "tidyverse",
  "ggpubr",
  "rstatix",
  "skimr",
  "janitor"
))
```

```r
# Load packages every time you start a new R session
library(tidyverse)
library(ggpubr)
library(rstatix)
library(skimr)
library(janitor)
```

---

## 3. Create an Example Biomedical Dataset

In this tutorial, we will use a simulated biomedical dataset.

The dataset contains information about patients, including:

- age
- sex
- treatment group
- body mass index
- glucose level
- cholesterol level
- inflammatory marker level
- disease status

```r
# Set seed for reproducibility
set.seed(123)

# Create simulated biomedical dataset
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

# View the first rows
head(patient_data)
```

---

## 4. Inspect the Dataset

```r
# View dataset structure
str(patient_data)

# View column names
names(patient_data)

# View dimensions: rows and columns
dim(patient_data)

# Summary of the dataset
summary(patient_data)
```

---

## 5. Clean Column Names

The `janitor` package can clean column names and make them consistent.

```r
patient_data <- patient_data %>%
  clean_names()

names(patient_data)
```

---

## 6. Check for Missing Values

```r
# Count missing values in each column
colSums(is.na(patient_data))

# Check total number of missing values
sum(is.na(patient_data))
```

For demonstration, we will intentionally add some missing values.

```r
# Add artificial missing values
patient_data$glucose[c(5, 12, 30)] <- NA
patient_data$bmi[c(8, 18)] <- NA

# Check missing values again
colSums(is.na(patient_data))
```

---

## 7. Handle Missing Values

There are several ways to handle missing values.

For this basic tutorial, we will remove rows with missing values.

```r
patient_data_clean <- patient_data %>%
  drop_na()

dim(patient_data)
dim(patient_data_clean)
```

---

## 8. Descriptive Statistics

Descriptive statistics summarize the main features of a dataset.

Common descriptive statistics include:

- mean
- median
- standard deviation
- minimum
- maximum
- interquartile range

---

## 9. Summary of Continuous Variables

```r
patient_data_clean %>%
  summarise(
    mean_age = mean(age),
    median_age = median(age),
    sd_age = sd(age),
    min_age = min(age),
    max_age = max(age),
    
    mean_bmi = mean(bmi),
    median_bmi = median(bmi),
    sd_bmi = sd(bmi),
    
    mean_glucose = mean(glucose),
    median_glucose = median(glucose),
    sd_glucose = sd(glucose),
    
    mean_cholesterol = mean(cholesterol),
    median_cholesterol = median(cholesterol),
    sd_cholesterol = sd(cholesterol),
    
    mean_crp = mean(crp),
    median_crp = median(crp),
    sd_crp = sd(crp)
  )
```

---

## 10. Summary Using `skimr`

The `skimr` package gives a detailed summary of the dataset.

```r
skim(patient_data_clean)
```

---

## 11. Frequency Tables for Categorical Variables

```r
# Frequency table for sex
table(patient_data_clean$sex)

# Frequency table for treatment group
table(patient_data_clean$treatment_group)

# Frequency table for disease status
table(patient_data_clean$disease_status)
```

Using `dplyr`:

```r
patient_data_clean %>%
  count(sex)

patient_data_clean %>%
  count(treatment_group)

patient_data_clean %>%
  count(disease_status)
```

---

## 12. Proportions for Categorical Variables

```r
# Proportion of sex categories
prop.table(table(patient_data_clean$sex))

# Proportion of treatment groups
prop.table(table(patient_data_clean$treatment_group))

# Proportion of disease status
prop.table(table(patient_data_clean$disease_status))
```

Convert proportions to percentages:

```r
patient_data_clean %>%
  count(sex) %>%
  mutate(percent = n / sum(n) * 100)

patient_data_clean %>%
  count(treatment_group) %>%
  mutate(percent = n / sum(n) * 100)

patient_data_clean %>%
  count(disease_status) %>%
  mutate(percent = n / sum(n) * 100)
```

---

# Data Visualization for Basic Statistics

## 13. Histogram

A histogram shows the distribution of a continuous variable.

```r
ggplot(patient_data_clean, aes(x = glucose)) +
  geom_histogram(bins = 30, color = "black", fill = "skyblue") +
  labs(
    title = "Distribution of Glucose Levels",
    x = "Glucose Level",
    y = "Count"
  ) +
  theme_minimal()
```

---

## 14. Density Plot

```r
ggplot(patient_data_clean, aes(x = glucose)) +
  geom_density(fill = "lightgreen", alpha = 0.6) +
  labs(
    title = "Density Plot of Glucose Levels",
    x = "Glucose Level",
    y = "Density"
  ) +
  theme_minimal()
```

---

## 15. Boxplot

A boxplot shows the median, spread, and possible outliers.

```r
ggplot(patient_data_clean, aes(y = glucose)) +
  geom_boxplot(fill = "orange", color = "black") +
  labs(
    title = "Boxplot of Glucose Levels",
    y = "Glucose Level"
  ) +
  theme_minimal()
```

---

## 16. Boxplot by Group

```r
ggplot(patient_data_clean, aes(x = treatment_group, y = glucose, fill = treatment_group)) +
  geom_boxplot() +
  labs(
    title = "Glucose Levels by Treatment Group",
    x = "Treatment Group",
    y = "Glucose Level"
  ) +
  theme_minimal()
```

---

## 17. Bar Plot for Categorical Variables

```r
ggplot(patient_data_clean, aes(x = disease_status, fill = disease_status)) +
  geom_bar() +
  labs(
    title = "Disease Status Distribution",
    x = "Disease Status",
    y = "Count"
  ) +
  theme_minimal()
```

---

# Normality Testing

## 18. Why Check Normality?

Many statistical tests assume that the data are approximately normally distributed.

Examples include:

- t-test
- ANOVA
- linear regression

Normality can be checked using:

- histogram
- Q-Q plot
- Shapiro-Wilk test

---

## 19. Q-Q Plot

```r
ggqqplot(patient_data_clean$glucose)
```

Using `ggplot2`:

```r
ggplot(patient_data_clean, aes(sample = glucose)) +
  stat_qq() +
  stat_qq_line() +
  labs(
    title = "Q-Q Plot of Glucose Levels"
  ) +
  theme_minimal()
```

---

## 20. Shapiro-Wilk Normality Test

```r
shapiro.test(patient_data_clean$glucose)
```

Interpretation:

- p-value > 0.05: data are approximately normally distributed
- p-value ≤ 0.05: data significantly deviate from normality

---

## 21. Normality Test by Group

```r
patient_data_clean %>%
  group_by(treatment_group) %>%
  shapiro_test(glucose)
```

---

# Comparing Two Groups

## 22. Independent Samples t-test

Use an independent samples t-test when comparing the means of two independent groups.

Example question:

> Is the mean glucose level different between the Control and Treatment groups?

```r
t.test(glucose ~ treatment_group, data = patient_data_clean)
```

---

## 23. t-test Using `rstatix`

```r
patient_data_clean %>%
  t_test(glucose ~ treatment_group)
```

Add effect size:

```r
patient_data_clean %>%
  cohens_d(glucose ~ treatment_group)
```

---

## 24. Visualize t-test Results

```r
ggboxplot(
  patient_data_clean,
  x = "treatment_group",
  y = "glucose",
  fill = "treatment_group",
  palette = "jco",
  add = "jitter"
) +
  stat_compare_means(method = "t.test") +
  labs(
    title = "Comparison of Glucose Levels Between Groups",
    x = "Treatment Group",
    y = "Glucose Level"
  )
```

---

## 25. Non-parametric Alternative: Wilcoxon Test

If data are not normally distributed, use the Wilcoxon rank-sum test.

```r
wilcox.test(glucose ~ treatment_group, data = patient_data_clean)
```

Using `rstatix`:

```r
patient_data_clean %>%
  wilcox_test(glucose ~ treatment_group)
```

---

# Comparing More Than Two Groups

## 26. Create a Three-Group Variable

For ANOVA demonstration, we will create an age group variable.

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

## 27. One-way ANOVA

ANOVA compares the means of more than two groups.

Example question:

> Are cholesterol levels different among age groups?

```r
anova_result <- aov(cholesterol ~ age_group, data = patient_data_clean)

summary(anova_result)
```

---

## 28. ANOVA Using `rstatix`

```r
patient_data_clean %>%
  anova_test(cholesterol ~ age_group)
```

---

## 29. Post-hoc Test

If ANOVA is significant, use a post-hoc test to identify which groups differ.

```r
TukeyHSD(anova_result)
```

Using `rstatix`:

```r
patient_data_clean %>%
  tukey_hsd(cholesterol ~ age_group)
```

---

## 30. Visualize ANOVA Results

```r
ggboxplot(
  patient_data_clean,
  x = "age_group",
  y = "cholesterol",
  fill = "age_group",
  palette = "jco",
  add = "jitter"
) +
  stat_compare_means(method = "anova") +
  labs(
    title = "Cholesterol Levels Across Age Groups",
    x = "Age Group",
    y = "Cholesterol Level"
  )
```

---

## 31. Non-parametric Alternative: Kruskal-Wallis Test

If ANOVA assumptions are not met, use the Kruskal-Wallis test.

```r
kruskal.test(cholesterol ~ age_group, data = patient_data_clean)
```

Using `rstatix`:

```r
patient_data_clean %>%
  kruskal_test(cholesterol ~ age_group)
```

---

# Categorical Data Analysis

## 32. Contingency Table

A contingency table shows the relationship between two categorical variables.

Example question:

> Is disease status associated with treatment group?

```r
disease_treatment_table <- table(
  patient_data_clean$disease_status,
  patient_data_clean$treatment_group
)

disease_treatment_table
```

---

## 33. Chi-square Test

The Chi-square test checks whether two categorical variables are associated.

```r
chisq.test(disease_treatment_table)
```

---

## 34. Chi-square Test Using `rstatix`

```r
patient_data_clean %>%
  chisq_test(disease_status, treatment_group)
```

---

## 35. Fisher's Exact Test

Fisher's exact test is useful when sample sizes are small.

```r
fisher.test(disease_treatment_table)
```

---

## 36. Visualize Categorical Data

```r
ggplot(patient_data_clean, aes(x = treatment_group, fill = disease_status)) +
  geom_bar(position = "fill") +
  labs(
    title = "Disease Status by Treatment Group",
    x = "Treatment Group",
    y = "Proportion",
    fill = "Disease Status"
  ) +
  theme_minimal()
```

---

# Correlation Analysis

## 37. Pearson Correlation

Pearson correlation measures the linear relationship between two continuous variables.

Example question:

> Is BMI associated with glucose level?

```r
cor.test(
  patient_data_clean$bmi,
  patient_data_clean$glucose,
  method = "pearson"
)
```

---

## 38. Spearman Correlation

Spearman correlation is used when data are not normally distributed or the relationship is monotonic but not linear.

```r
cor.test(
  patient_data_clean$bmi,
  patient_data_clean$glucose,
  method = "spearman"
)
```

---

## 39. Correlation Matrix

```r
continuous_vars <- patient_data_clean %>%
  select(age, bmi, glucose, cholesterol, crp)

cor_matrix <- cor(continuous_vars)

cor_matrix
```

---

## 40. Correlation Matrix with Missing Values Handled

```r
cor_matrix <- cor(
  continuous_vars,
  use = "complete.obs",
  method = "pearson"
)

cor_matrix
```

---

## 41. Visualize Correlation

```r
ggplot(patient_data_clean, aes(x = bmi, y = glucose)) +
  geom_point(alpha = 0.7) +
  geom_smooth(method = "lm", se = TRUE) +
  labs(
    title = "Relationship Between BMI and Glucose",
    x = "BMI",
    y = "Glucose Level"
  ) +
  theme_minimal()
```

---

# Linear Regression

## 42. Simple Linear Regression

Linear regression is used to model the relationship between a continuous outcome and one or more predictors.

Example question:

> Does BMI predict glucose level?

```r
model_1 <- lm(glucose ~ bmi, data = patient_data_clean)

summary(model_1)
```

---

## 43. Multiple Linear Regression

Example question:

> Do BMI, age, and cholesterol predict glucose level?

```r
model_2 <- lm(glucose ~ bmi + age + cholesterol, data = patient_data_clean)

summary(model_2)
```

---

## 44. Regression with a Categorical Predictor

```r
model_3 <- lm(glucose ~ treatment_group, data = patient_data_clean)

summary(model_3)
```

---

## 45. Regression with Continuous and Categorical Predictors

```r
model_4 <- lm(
  glucose ~ bmi + age + cholesterol + treatment_group + sex,
  data = patient_data_clean
)

summary(model_4)
```

---

## 46. Diagnostic Plots for Linear Regression

```r
par(mfrow = c(2, 2))
plot(model_4)
par(mfrow = c(1, 1))
```

---

## 47. Extract Regression Results

```r
regression_results <- summary(model_4)

regression_results
```

Extract coefficients:

```r
coef(model_4)
```

Extract confidence intervals:

```r
confint(model_4)
```

---

# Logistic Regression

## 48. Prepare Binary Outcome

Logistic regression is used when the outcome variable is binary.

We will create a binary disease outcome:

- 0 = Healthy
- 1 = Disease

```r
patient_data_clean <- patient_data_clean %>%
  mutate(
    disease_binary = ifelse(disease_status == "Disease", 1, 0)
  )

table(patient_data_clean$disease_binary)
```

---

## 49. Logistic Regression Model

Example question:

> Are age, BMI, glucose, and cholesterol associated with disease status?

```r
logistic_model <- glm(
  disease_binary ~ age + bmi + glucose + cholesterol,
  data = patient_data_clean,
  family = binomial
)

summary(logistic_model)
```

---

## 50. Odds Ratios

The coefficients from logistic regression are in log-odds.  
To convert them to odds ratios, use `exp()`.

```r
odds_ratios <- exp(coef(logistic_model))

odds_ratios
```

Confidence intervals for odds ratios:

```r
odds_ratio_ci <- exp(confint(logistic_model))

odds_ratio_ci
```

---

# Reporting Statistical Results

## 51. Reporting a t-test

Example format:

> An independent samples t-test was used to compare glucose levels between the Control and Treatment groups. The result showed whether there was a statistically significant difference between groups.

```r
t_test_result <- t.test(glucose ~ treatment_group, data = patient_data_clean)

t_test_result
```

---

## 52. Reporting an ANOVA

Example format:

> A one-way ANOVA was performed to compare cholesterol levels among age groups.

```r
anova_result <- aov(cholesterol ~ age_group, data = patient_data_clean)

summary(anova_result)
```

---

## 53. Reporting a Correlation

Example format:

> Pearson correlation was used to assess the relationship between BMI and glucose level.

```r
correlation_result <- cor.test(
  patient_data_clean$bmi,
  patient_data_clean$glucose,
  method = "pearson"
)

correlation_result
```

---

## 54. Reporting a Linear Regression

Example format:

> A multiple linear regression model was used to evaluate whether age, BMI, cholesterol, treatment group, and sex predicted glucose level.

```r
linear_model_result <- summary(model_4)

linear_model_result
```

---

# Exporting Results

## 55. Export Clean Dataset

```r
write.csv(
  patient_data_clean,
  "patient_data_clean.csv",
  row.names = FALSE
)
```

---

## 56. Export Summary Statistics

```r
summary_statistics <- patient_data_clean %>%
  summarise(
    mean_age = mean(age),
    sd_age = sd(age),
    mean_bmi = mean(bmi),
    sd_bmi = sd(bmi),
    mean_glucose = mean(glucose),
    sd_glucose = sd(glucose),
    mean_cholesterol = mean(cholesterol),
    sd_cholesterol = sd(cholesterol),
    median_crp = median(crp),
    iqr_crp = IQR(crp)
  )

write.csv(
  summary_statistics,
  "summary_statistics.csv",
  row.names = FALSE
)
```

---

## 57. Export Group Summary

```r
group_summary <- patient_data_clean %>%
  group_by(treatment_group) %>%
  summarise(
    n = n(),
    mean_glucose = mean(glucose),
    sd_glucose = sd(glucose),
    median_glucose = median(glucose),
    iqr_glucose = IQR(glucose)
  )

write.csv(
  group_summary,
  "group_summary.csv",
  row.names = FALSE
)
```

---

# Complete Practice Exercises

## Exercise 1

Calculate the mean, median, and standard deviation of `bmi`.

```r
patient_data_clean %>%
  summarise(
    mean_bmi = mean(bmi),
    median_bmi = median(bmi),
    sd_bmi = sd(bmi)
  )
```

---

## Exercise 2

Create a histogram of cholesterol levels.

```r
ggplot(patient_data_clean, aes(x = cholesterol)) +
  geom_histogram(bins = 30, color = "black", fill = "lightblue") +
  labs(
    title = "Distribution of Cholesterol Levels",
    x = "Cholesterol",
    y = "Count"
  ) +
  theme_minimal()
```

---

## Exercise 3

Compare BMI between males and females using a t-test.

```r
t.test(bmi ~ sex, data = patient_data_clean)
```

---

## Exercise 4

Test whether disease status is associated with sex.

```r
sex_disease_table <- table(
  patient_data_clean$sex,
  patient_data_clean$disease_status
)

chisq.test(sex_disease_table)
```

---

## Exercise 5

Test the correlation between age and cholesterol.

```r
cor.test(
  patient_data_clean$age,
  patient_data_clean$cholesterol,
  method = "pearson"
)
```

---

## Exercise 6

Fit a linear regression model predicting cholesterol from age and BMI.

```r
cholesterol_model <- lm(cholesterol ~ age + bmi, data = patient_data_clean)

summary(cholesterol_model)
```

---

# Common Statistical Tests in R

| Research Question | Variable Type | Test |
|---|---|---|
| Compare mean between two independent groups | Continuous outcome, categorical predictor with 2 groups | Independent t-test |
| Compare median between two independent groups | Non-normal continuous outcome, categorical predictor with 2 groups | Wilcoxon rank-sum test |
| Compare mean among more than two groups | Continuous outcome, categorical predictor with 3+ groups | ANOVA |
| Compare medians among more than two groups | Non-normal continuous outcome, categorical predictor with 3+ groups | Kruskal-Wallis test |
| Test association between two categorical variables | Two categorical variables | Chi-square test |
| Test relationship between two continuous variables | Two continuous variables | Pearson or Spearman correlation |
| Predict continuous outcome | Continuous outcome | Linear regression |
| Predict binary outcome | Binary outcome | Logistic regression |

---

# Interpretation of p-values

A p-value helps evaluate statistical evidence against the null hypothesis.

Common interpretation:

- p-value < 0.05: statistically significant
- p-value ≥ 0.05: not statistically significant

Important note:

A statistically significant result does not always mean the result is biologically or clinically important. Always interpret statistical results in the context of the research question, study design, sample size, and effect size.

---

# Good Statistical Practice

When performing statistical analysis, always:

- define your research question clearly
- inspect and clean your data
- check assumptions before choosing a test
- report effect sizes when possible
- visualize your data
- avoid relying only on p-values
- interpret results in biological or clinical context
- document your code for reproducibility

---

# Session Information

It is good practice to include session information at the end of your analysis.

```r
sessionInfo()
```

---

# Summary

In this tutorial, you learned how to perform basic statistics in R using a biomedical-style dataset.

You covered:

- descriptive statistics
- frequency tables
- data visualization
- normality testing
- t-tests
- ANOVA
- non-parametric tests
- chi-square tests
- correlation
- linear regression
- logistic regression
- exporting results

This tutorial provides a foundation for statistical analysis in R and prepares you for more advanced biomedical and transcriptomics data analysis.
