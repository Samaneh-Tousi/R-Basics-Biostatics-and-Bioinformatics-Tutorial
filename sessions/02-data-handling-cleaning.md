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

- import CSV, text, and Excel files
- inspect imported datasets
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
| `readr` | Importing text and CSV files |
| `stringr` | String handling |

---

# Importing Data into R

Biomedical data are often stored in files such as:

- Excel files (`.xlsx`, `.xls`)
- CSV files (`.csv`)
- text files (`.txt`)

Before importing a dataset, it is useful to know:

- whether the first row contains column names
- which delimiter separates the columns
- which decimal symbol is used
- how missing values are represented

Common missing-value symbols include:

```text
NA
?
.
```

or empty cells.

---

## Importing CSV Files

The `readr` package is included in the tidyverse.

Use `read_csv()` to import a standard comma-separated file:

```r
gene_data <- read_csv("data/gene_expression.csv")
```

You can specify how missing values are represented:

```r
gene_data <- read_csv(
  "data/gene_expression.csv",
  na = c("", "NA", "?")
)
```

After importing the data, inspect it:

```r
head(gene_data)
glimpse(gene_data)
```

---

## Importing Semicolon-Separated CSV Files

In some European systems, a comma is used as the decimal symbol and a semicolon is used to separate columns.

For these files, `read_csv2()` can be useful:

```r
gene_data <- read_csv2("data/gene_expression.csv")
```

Always check the imported data to make sure the columns and numeric values were interpreted correctly.

---

## Importing Text Files

For text files, you can use `read_delim()` and specify the delimiter.

For a tab-separated file:

```r
gene_data <- read_delim(
  "data/gene_expression.txt",
  delim = "\t"
)
```

For another delimiter, replace `"\t"` with the appropriate character.

For example, a semicolon-separated text file can be imported using:

```r
gene_data <- read_delim(
  "data/gene_expression.txt",
  delim = ";"
)
```

---

## Importing Excel Files

Excel files can be imported using the `readxl` package.

Install it once:

```r
install.packages("readxl")
```

Load the package:

```r
library(readxl)
```

Import an Excel file:

```r
gene_data <- read_excel(
  "data/gene_expression.xlsx"
)
```

If the Excel file contains multiple sheets, you can select one:

```r
gene_data <- read_excel(
  "data/gene_expression.xlsx",
  sheet = "Sheet1"
)
```

After importing an Excel file, always inspect the result:

```r
head(gene_data)
str(gene_data)
```

---

# Creating an Example Biomedical Dataset

We will make a small dummy biomedical dataset representing simplified gene expression measurements from RNA-seq samples.

Create a data frame called `gene_expr`:

```r
gene_expr <- data.frame(
  Sample_ID = c("S01", "S02", "S03", "S04", "S05", "S05", "S06"),
  Patient_ID = c("P001", "P002", "P003", "P004", "P005", "P005", "P006"),
  Condition = c("Control", "Treated", "Control", "Treated", "Control", "Control", "Treated"),
  Tissue = c("Blood", "Blood", "Liver", "Liver", "Blood", "Blood", "Blood"),
  Gene = c("TP53", "TP53", "BRCA1", "BRCA1", "MYC", "MYC", "MYC"),
  Expression_Rep1 = c(12.5, 18.2, 8.7, NA, 22.1, 22.1, 30.5),
  Expression_Rep2 = c(13.1, 19.0, NA, 15.4, 21.8, 21.8, 31.2),
  Collection_Date = as.Date(c(
    "2024-01-10",
    "2024-01-12",
    "2024-01-15",
    "2024-01-18",
    "2024-01-20",
    "2024-01-20",
    "2024-01-22"
  ))
)
```

This dataset contains:

- missing expression values
- duplicated rows
- sample metadata
- gene expression measurements

---

# Inspecting Data

Before cleaning or analyzing a dataset, always inspect its structure and contents.

## View the dataset

```r
View(gene_expr)
```

`View()` opens the dataset in a spreadsheet-like window in RStudio.

For large datasets, it is often faster to inspect only the first rows.

---

## View the first rows

```r
head(gene_expr)
```

---

## View dataset structure

Using tidyverse:

```r
glimpse(gene_expr)
```

Using Base R:

```r
str(gene_expr)
```

Both functions help you check which variables are present and what data type each column contains.

---

## View dimensions

```r
dim(gene_expr)
```

The first number is the number of rows or observations.

The second number is the number of columns or variables.

---

## View column names

```r
colnames(gene_expr)
```

You can also use:

```r
names(gene_expr)
```

---

## Generate summary statistics

```r
summary(gene_expr)
```

`summary()` gives a quick overview of each column.

For numeric variables, it reports values such as the minimum, median, mean, and maximum.

For other data types, the output depends on the type of variable.

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
  select(Sample_ID, Patient_ID, Condition, Gene)
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
    patient_id = Patient_ID,
    collection_date = Collection_Date
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
  select(Sample_ID, Patient_ID, Tissue, Gene, Expression_Rep2) %>%
  arrange(desc(Expression_Rep2))
```

---

# Base R Alternatives for Working with Data Frames

The tidyverse provides convenient functions for data manipulation, but it is also useful to recognize common **Base R** syntax.

These approaches perform similar operations without `dplyr`.

---

## Selecting a Column

Using `$`:

```r
gene_expr$Condition
```

Using column indexing:

```r
gene_expr[, "Condition"]
```

---

## Selecting Rows and Columns

Data frames use the following structure:

```text
dataframe[row, column]
```

Select the first row:

```r
gene_expr[1, ]
```

Select the first column:

```r
gene_expr[, 1]
```

Select the first three rows and the first two columns:

```r
gene_expr[1:3, 1:2]
```

---

## Filtering Rows in Base R

Select treated samples:

```r
gene_expr[gene_expr$Condition == "Treated", ]
```

Select treated blood samples:

```r
gene_expr[
  gene_expr$Condition == "Treated" &
    gene_expr$Tissue == "Blood",
]
```

---

## Sorting Rows in Base R

The `order()` function can be used to sort data.

Ascending order:

```r
gene_expr[
  order(gene_expr$Expression_Rep2),
]
```

Descending order:

```r
gene_expr[
  order(gene_expr$Expression_Rep2, decreasing = TRUE),
]
```

---

## Adding Columns with cbind()

The `cbind()` function combines objects by columns.

For example:

```r
quality_score <- c(1, 2, 1, 2, 1, 1, 2)

gene_expr_with_quality <- cbind(
  gene_expr,
  quality_score
)
```

The new column must contain the same number of values as the data frame has rows.

---

## Adding Rows with rbind()

The `rbind()` function combines objects by rows.

For example:

```r
new_sample <- gene_expr[1, ]

gene_expr_extended <- rbind(
  gene_expr,
  new_sample
)
```

The new row must contain the same columns as the original data frame.

For most data-cleaning tasks in this tutorial, we will mainly use the tidyverse functions because they are easier to read in longer workflows.

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

> Replacing missing values with `0` is shown here only as an example of how `replace_na()` works.  
> In real biomedical analyses, whether a missing value should be replaced with zero depends on what the missing value represents.

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
    Collection_Date = as.Date(Collection_Date)
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

Convert a categorical variable into a factor:

```r
gene_expr %>%
  mutate(
    Condition = as.factor(Condition)
  )
```

Check the result:

```r
str(gene_expr$Condition)
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
    patient_id = Patient_ID,
    condition = Condition,
    tissue = Tissue,
    gene = Gene,
    expression_rep1 = Expression_Rep1,
    expression_rep2 = Expression_Rep2,
    collection_date = Collection_Date
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

Check its dimensions:

```r
dim(clean_gene_expr)
```

Check its column names:

```r
colnames(clean_gene_expr)
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

## Exercise 9

Use Base R to:

1. display the dimensions of `gene_expr`
2. display its column names
3. select only rows where `Condition == "Treated"`

---

## Exercise 10

Suppose a dataset is stored as:

```text
data/patient_data.xlsx
```

Write the commands required to:

1. load the `readxl` package
2. import the Excel file
3. inspect the first rows of the imported dataset

---

# Practice Solutions

## Solution 1

```r
gene_expr %>%
  select(
    Sample_ID,
    Patient_ID,
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

## Solution 9

```r
dim(gene_expr)

colnames(gene_expr)

gene_expr[
  gene_expr$Condition == "Treated",
]
```

---

## Solution 10

```r
library(readxl)

patient_data <- read_excel(
  "data/patient_data.xlsx"
)

head(patient_data)
```

---

# Summary

In this tutorial, you learned how to:

- import CSV, text, and Excel datasets
- check delimiters, missing values, and file structure before analysis
- inspect datasets using `head()`, `View()`, `str()`, `glimpse()`, `dim()`, and `colnames()`
- manipulate data using `dplyr`
- recognize common Base R approaches for working with data frames
- reshape data using `tidyr`
- handle missing values
- remove duplicated rows
- create reproducible cleaning workflows
- export cleaned data

---

# References

- Wickham H. et al. *R for Data Science*
- tidyverse documentation
- dplyr documentation
- tidyr documentation
- readr documentation
- readxl documentation
