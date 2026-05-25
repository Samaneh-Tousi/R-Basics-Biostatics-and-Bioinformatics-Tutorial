# Data Handling & Cleaning in R with dplyr and tidyr

## Introduction

Data handling and cleaning are essential skills in data science, statistics, and biomedical research.

Real-world biomedical datasets are often:

- incomplete
- duplicated
- inconsistent
- improperly formatted
- difficult to analyze without cleaning

The `dplyr` and `tidyr` packages from the **tidyverse** provide powerful tools for cleaning and transforming data in R.

In this tutorial, you will learn how to:

- import biomedical datasets
- inspect data structures
- manipulate data using `dplyr`
- reshape data using `tidyr`
- handle missing values
- remove duplicates
- build reproducible cleaning workflows

---

# Learning Objectives

By the end of this tutorial, you will be able to:

- use `dplyr` for data manipulation
- use `tidyr` for reshaping data
- clean messy biomedical datasets
- handle missing values
- remove duplicated rows
- export cleaned datasets

---

# Installing and Loading Packages

Install the package once:

```r
install.packages("tidyverse")
```

> You only need to install packages once.  
> After installation, simply load them using `library()`.

Load the package:

```r
library(tidyverse)
```

---

# Understanding the Tidyverse

The **tidyverse** is a collection of packages designed for data science in R.

Important packages include:

| Package | Purpose |
|---|---|
| `dplyr` | Data manipulation |
| `tidyr` | Data reshaping |
| `ggplot2` | Data visualization |
| `readr` | Importing files |
| `stringr` | String handling |

---

# Creating an Example Biomedical Dataset

We will use a small biomedical dataset representing simplified gene expression measurements from RNA-seq samples.

Create a file called:

```text
data/messy_gene_expression.csv
```

with the following content:

```csv
Sample_ID,Patient ID,Condition,Tissue,Gene,Expression_Rep1,Expression_Rep2,Collection Date
S01,P001,Control,Blood,TP53,12.5,13.1,2024-01-10
S02,P002,Treated,Blood,TP53,18.2,19.0,2024-01-12
S03,P003,Control,Liver,BRCA1,8.7,,2024-01-15
S04,P004,Treated,Liver,BRCA1,,15.4,2024-01-18
S05,P005,Control,Blood,MYC,22.1,21.8,2024-01-20
S05,P005,Control,Blood,MYC,22.1,21.8,2024-01-20
S06,P006,Treated,Blood,MYC,30.5,31.2,2024-01-22
```

This dataset contains:

- missing expression values
- duplicated rows
- inconsistent column names
- sample metadata
- gene expression measurements

---

# Importing Data

Use `read_csv()` to import CSV files.

```r
library(tidyverse)

gene_expr <- read_csv("data/messy_gene_expression.csv")
```

---

# Inspecting Data

## View the first rows

```r
head(gene_expr)
```

---

## View dataset structure

```r
glimpse(gene_expr)
```

---

## View dimensions

```r
dim(gene_expr)
```

---

## View column names

```r
colnames(gene_expr)
```

---

## Generate summary statistics

```r
summary(gene_expr)
```

---

# Introduction to dplyr

The `dplyr` package is used for data manipulation.

Main functions include:

| Function | Purpose |
|---|---|
| `select()` | Choose columns |
| `filter()` | Choose rows |
| `mutate()` | Create variables |
| `arrange()` | Sort rows |
| `summarise()` | Generate summaries |
| `group_by()` | Group data |

---

# Selecting Columns

Use `select()` to keep specific columns.

```r
gene_expr %>%
  select(Sample_ID, `Patient ID`, Condition, Gene)
```

---

# Filtering Rows

Use `filter()` to keep rows that satisfy conditions.

```r
gene_expr %>%
  filter(Condition == "Treated")
```

Filter using multiple conditions:

```r
gene_expr %>%
  filter(
    Condition == "Treated",
    Tissue == "Blood"
  )
```

---

# Creating New Variables

Use `mutate()` to create new variables.

```r
gene_expr %>%
  mutate(
    mean_expression = (Expression_Rep1 + Expression_Rep2) / 2
  )
```

---

# Arranging Rows

Sort rows using `arrange()`.

Ascending order:

```r
gene_expr %>%
  arrange(Expression_Rep2)
```

Descending order:

```r
gene_expr %>%
  arrange(desc(Expression_Rep2))
```

---

# Renaming Columns

Use `rename()` to improve column names.

```r
gene_expr %>%
  rename(
    sample_id = Sample_ID,
    patient_id = `Patient ID`,
    collection_date = `Collection Date`
  )
```

---

# Grouping and Summarizing Data

Use `group_by()` with `summarise()`.

```r
gene_expr %>%
  group_by(Condition, Gene) %>%
  summarise(
    mean_expression_rep2 = mean(Expression_Rep2, na.rm = TRUE),
    .groups = "drop"
  )
```

---

# Counting Observations

Count samples by condition.

```r
gene_expr %>%
  count(Condition)
```

Count genes by condition.

```r
gene_expr %>%
  count(Gene, Condition)
```

---

# Using Pipes

The pipe operator `%>%` passes output from one step to the next.

```r
gene_expr %>%
  filter(Condition == "Treated") %>%
  select(Sample_ID, `Patient ID`, Tissue, Gene, Expression_Rep2) %>%
  arrange(desc(Expression_Rep2))
```

---

# Introduction to tidyr

The `tidyr` package helps reshape datasets.

Important functions:

| Function | Purpose |
|---|---|
| `pivot_longer()` | Wide to long format |
| `pivot_wider()` | Long to wide format |
| `separate()` | Split columns |
| `unite()` | Merge columns |
| `drop_na()` | Remove missing values |
| `replace_na()` | Replace missing values |

---

# Wide vs Long Format

## Wide format

| Sample_ID | Gene | Expression_Rep1 | Expression_Rep2 |
|---|---|---|---|
| S01 | TP53 | 12.5 | 13.1 |

---

## Long format

| Sample_ID | Gene | Replicate | Expression |
|---|---|---|---|
| S01 | TP53 | Expression_Rep1 | 12.5 |
| S01 | TP53 | Expression_Rep2 | 13.1 |

---

# Converting Wide to Long Format

Use `pivot_longer()`.

```r
gene_expr_long <- gene_expr %>%
  pivot_longer(
    cols = c(Expression_Rep1, Expression_Rep2),
    names_to = "replicate",
    values_to = "expression"
  )

gene_expr_long
```

---

# Converting Long to Wide Format

Use `pivot_wider()`.

```r
gene_expr_wide <- gene_expr_long %>%
  pivot_wider(
    names_from = replicate,
    values_from = expression
  )

gene_expr_wide
```

---

# Separating Columns

Use `separate()` to split one column into multiple columns.

```r
sample_info <- tibble(
  sample_label = c("Control_Blood", "Treated_Liver")
)

sample_info %>%
  separate(
    sample_label,
    into = c("condition", "tissue"),
    sep = "_"
  )
```

---

# Combining Columns

Use `unite()` to combine multiple columns into one column.

```r
sample_parts <- tibble(
  condition = c("Control", "Treated"),
  tissue = c("Blood", "Liver")
)

sample_parts %>%
  unite(
    "sample_group",
    condition,
    tissue,
    sep = "_"
  )
```

---

# Understanding Missing Values

Missing values in R are represented by:

```r
NA
```

Missing values are common in biomedical datasets because of:

- failed measurements
- low-quality samples
- incomplete metadata
- technical problems during data collection

---

# Identifying Missing Values

```r
is.na(gene_expr)
```

Count missing values in each column:

```r
colSums(is.na(gene_expr))
```

---

# Removing Missing Values

Use `drop_na()`.

```r
gene_expr %>%
  drop_na(Expression_Rep1, Expression_Rep2)
```

---

# Replacing Missing Values

Use `replace_na()`.

```r
gene_expr %>%
  mutate(
    Expression_Rep1 = replace_na(Expression_Rep1, 0),
    Expression_Rep2 = replace_na(Expression_Rep2, 0)
  )
```

---

# Removing Duplicate Rows

Use `distinct()`.

```r
gene_expr %>%
  distinct()
```

Remove duplicates based on selected columns:

```r
gene_expr %>%
  distinct(Sample_ID, Gene, .keep_all = TRUE)
```

---

# Converting Data Types

Convert text to dates:

```r
gene_expr %>%
  mutate(
    `Collection Date` = as.Date(`Collection Date`)
  )
```

Convert expression values to numeric:

```r
gene_expr %>%
  mutate(
    Expression_Rep1 = as.numeric(Expression_Rep1),
    Expression_Rep2 = as.numeric(Expression_Rep2)
  )
```

---

# Complete Biomedical Data Cleaning Workflow

Below is a complete reproducible cleaning workflow.

```r
library(tidyverse)

gene_expr <- read_csv("data/messy_gene_expression.csv")

clean_gene_expr <- gene_expr %>%
  
  # Rename columns
  rename(
    sample_id = Sample_ID,
    patient_id = `Patient ID`,
    condition = Condition,
    tissue = Tissue,
    gene = Gene,
    expression_rep1 = Expression_Rep1,
    expression_rep2 = Expression_Rep2,
    collection_date = `Collection Date`
  ) %>%
  
  # Remove duplicated rows
  distinct() %>%
  
  # Replace missing expression values
  mutate(
    expression_rep1 = replace_na(expression_rep1, 0),
    expression_rep2 = replace_na(expression_rep2, 0)
  ) %>%
  
  # Convert collection date to Date format
  mutate(
    collection_date = as.Date(collection_date)
  ) %>%
  
  # Create mean expression variable
  mutate(
    mean_expression = (expression_rep1 + expression_rep2) / 2
  )

clean_gene_expr
```

---

# Checking the Cleaned Dataset

View the cleaned dataset:

```r
head(clean_gene_expr)
```

Check its structure:

```r
glimpse(clean_gene_expr)
```

Count missing values again:

```r
colSums(is.na(clean_gene_expr))
```

---

# Summarizing Cleaned Biomedical Data

Calculate mean expression by condition and gene.

```r
clean_gene_expr %>%
  group_by(condition, gene) %>%
  summarise(
    average_expression = mean(mean_expression),
    .groups = "drop"
  )
```

Calculate mean expression by tissue.

```r
clean_gene_expr %>%
  group_by(tissue) %>%
  summarise(
    average_expression = mean(mean_expression),
    .groups = "drop"
  )
```

---

# Exporting Cleaned Data

Save cleaned data:

```r
write_csv(
  clean_gene_expr,
  "data/clean_gene_expression.csv"
)
```

---

# Practice Exercises

## Exercise 1

Select only:

- sample ID
- patient ID
- condition
- gene

---

## Exercise 2

Filter rows where:

```r
Condition == "Control"
```

---

## Exercise 3

Filter rows where:

```r
Tissue == "Blood"
```

---

## Exercise 4

Create a new variable called:

```r
mean_expression
```

equal to:

```r
(Expression_Rep1 + Expression_Rep2) / 2
```

---

## Exercise 5

Calculate the average expression for each gene.

---

## Exercise 6

Convert the dataset from wide format to long format.

---

## Exercise 7

Replace missing expression values with 0.

---

## Exercise 8

Remove duplicated rows.

---

# Practice Solutions

## Solution 1

```r
gene_expr %>%
  select(
    Sample_ID,
    `Patient ID`,
    Condition,
    Gene
  )
```

---

## Solution 2

```r
gene_expr %>%
  filter(Condition == "Control")
```

---

## Solution 3

```r
gene_expr %>%
  filter(Tissue == "Blood")
```

---

## Solution 4

```r
gene_expr %>%
  mutate(
    mean_expression =
      (Expression_Rep1 + Expression_Rep2) / 2
  )
```

---

## Solution 5

```r
gene_expr %>%
  group_by(Gene) %>%
  summarise(
    average_rep1 = mean(Expression_Rep1, na.rm = TRUE),
    average_rep2 = mean(Expression_Rep2, na.rm = TRUE),
    .groups = "drop"
  )
```

---

## Solution 6

```r
gene_expr %>%
  pivot_longer(
    cols = c(Expression_Rep1, Expression_Rep2),
    names_to = "replicate",
    values_to = "expression"
  )
```

---

## Solution 7

```r
gene_expr %>%
  mutate(
    Expression_Rep1 = replace_na(Expression_Rep1, 0),
    Expression_Rep2 = replace_na(Expression_Rep2, 0)
  )
```

---

## Solution 8

```r
gene_expr %>%
  distinct()
```

---

# Summary

In this tutorial, you learned how to:

- import biomedical datasets
- inspect data structures
- manipulate data using `dplyr`
- reshape data using `tidyr`
- handle missing values
- remove duplicated rows
- create reproducible cleaning workflows
- export cleaned data

These skills form the foundation of biomedical data analysis, transcriptomics, and RNA-seq workflows in R.

---

# Recommended Next Topics

After mastering data handling and cleaning, consider learning:

- data visualization with `ggplot2`
- exploratory data analysis
- statistical analysis in R
- reproducible reports with Quarto
- transcriptomics data analysis
- RNA-seq workflows
- differential gene expression analysis

---

# References

- Wickham H. et al. *R for Data Science*
- tidyverse documentation
- dplyr documentation
- tidyr documentation