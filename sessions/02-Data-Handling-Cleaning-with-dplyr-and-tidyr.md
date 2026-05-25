# Data Handling & Cleaning in R with dplyr and tidyr

## Introduction

Data handling and cleaning are essential skills in data science, statistics, and bioinformatics.

Real-world datasets are often:

- incomplete
- duplicated
- inconsistent
- improperly formatted

The `dplyr` and `tidyr` packages from the **tidyverse** provide powerful tools for cleaning and transforming data in R.

In this tutorial, you will learn how to:

- import datasets
- inspect data structures
- manipulate data using `dplyr`
- reshape data using `tidyr`
- handle missing values
- remove duplicates
- build reproducible workflows

---

# Learning Objectives

By the end of this tutorial, you will be able to:

- use `dplyr` for data manipulation
- use `tidyr` for reshaping data
- clean messy datasets
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

# Creating an Example Dataset

Create a file called:

```text
data/messy_sales.csv
```

with the following content:

```csv
Order_ID,Customer Name,Region,Product,Sales_2023,Sales_2024,Order Date
1,Alice,North,Book,120,150,2024-01-10
2,Bob,South,Pen,45,50,2024-01-12
3,Charlie,East,Notebook,78,,2024-01-15
4,Alice,North,Book,120,150,2024-01-10
5,Diana,West,Pencil,,35,2024-01-18
6,Eric,South,Pen,60,65,2024-01-20
```

This dataset contains:

- missing values
- duplicated rows
- inconsistent column names
- mixed formatting

---

# Importing Data

Use `read_csv()` to import CSV files.

```r
library(tidyverse)

sales <- read_csv("data/messy_sales.csv")
```

---

# Inspecting Data

## View the first rows

```r
head(sales)
```

---

## View dataset structure

```r
glimpse(sales)
```

---

## View dimensions

```r
dim(sales)
```

---

## View column names

```r
colnames(sales)
```

---

## Generate summary statistics

```r
summary(sales)
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
sales %>%
  select(Order_ID, Product, Region)
```

---

# Filtering Rows

Use `filter()` to keep rows that satisfy conditions.

```r
sales %>%
  filter(Region == "South")
```

Multiple conditions:

```r
sales %>%
  filter(
    Region == "South",
    Sales_2024 > 50
  )
```

---

# Creating New Variables

Use `mutate()` to create new variables.

```r
sales %>%
  mutate(
    total_sales = Sales_2023 + Sales_2024
  )
```

---

# Arranging Rows

Sort rows using `arrange()`.

Ascending order:

```r
sales %>%
  arrange(Sales_2024)
```

Descending order:

```r
sales %>%
  arrange(desc(Sales_2024))
```

---

# Renaming Columns

Use `rename()` to improve column names.

```r
sales %>%
  rename(
    customer_name = `Customer Name`,
    order_date = `Order Date`
  )
```

---

# Grouping and Summarizing Data

Use `group_by()` with `summarise()`.

```r
sales %>%
  group_by(Region) %>%
  summarise(
    average_sales = mean(Sales_2024, na.rm = TRUE)
  )
```

---

# Counting Observations

Count rows by category.

```r
sales %>%
  count(Region)
```

---

# Using Pipes

The pipe operator `%>%` passes output from one step to the next.

```r
sales %>%
  filter(Region == "South") %>%
  select(Customer = `Customer Name`, Sales_2024) %>%
  arrange(desc(Sales_2024))
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

| ID | Sales_2023 | Sales_2024 |
|---|---|---|
| 1 | 120 | 150 |

---

## Long format

| ID | Year | Sales |
|---|---|---|
| 1 | Sales_2023 | 120 |
| 1 | Sales_2024 | 150 |

---

# Converting Wide to Long Format

Use `pivot_longer()`.

```r
sales_long <- sales %>%
  pivot_longer(
    cols = c(Sales_2023, Sales_2024),
    names_to = "year",
    values_to = "sales"
  )

sales_long
```

---

# Converting Long to Wide Format

Use `pivot_wider()`.

```r
sales_wide <- sales_long %>%
  pivot_wider(
    names_from = year,
    values_from = sales
  )

sales_wide
```

---

# Separating Columns

Use `separate()`.

```r
names_df <- tibble(
  full_name = c("Alice_Smith", "Bob_Jones")
)

names_df %>%
  separate(
    full_name,
    into = c("first_name", "last_name"),
    sep = "_"
  )
```

---

# Combining Columns

Use `unite()`.

```r
name_parts <- tibble(
  first_name = c("Alice", "Bob"),
  last_name = c("Smith", "Jones")
)

name_parts %>%
  unite(
    "full_name",
    first_name,
    last_name,
    sep = " "
  )
```

---

# Understanding Missing Values

Missing values in R are represented by:

```r
NA
```

---

# Identifying Missing Values

```r
is.na(sales)
```

Count missing values:

```r
colSums(is.na(sales))
```

---

# Removing Missing Values

Use `drop_na()`.

```r
sales %>%
  drop_na(Sales_2024)
```

---

# Replacing Missing Values

Use `replace_na()`.

```r
sales %>%
  mutate(
    Sales_2023 = replace_na(Sales_2023, 0),
    Sales_2024 = replace_na(Sales_2024, 0)
  )
```

---

# Removing Duplicate Rows

Use `distinct()`.

```r
sales %>%
  distinct()
```

Remove duplicates based on specific columns:

```r
sales %>%
  distinct(Order_ID, .keep_all = TRUE)
```

---

# Converting Data Types

Convert text to dates:

```r
sales %>%
  mutate(
    `Order Date` = as.Date(`Order Date`)
  )
```

Convert to numeric:

```r
sales %>%
  mutate(
    Sales_2023 = as.numeric(Sales_2023)
  )
```

---

# Complete Data Cleaning Workflow

Below is a complete reproducible cleaning workflow.

```r
library(tidyverse)

sales <- read_csv("data/messy_sales.csv")

clean_sales <- sales %>%
  
  # Rename columns
  rename(
    order_id = Order_ID,
    customer_name = `Customer Name`,
    region = Region,
    product = Product,
    sales_2023 = Sales_2023,
    sales_2024 = Sales_2024,
    order_date = `Order Date`
  ) %>%
  
  # Remove duplicate rows
  distinct() %>%
  
  # Replace missing values
  mutate(
    sales_2023 = replace_na(sales_2023, 0),
    sales_2024 = replace_na(sales_2024, 0)
  ) %>%
  
  # Convert dates
  mutate(
    order_date = as.Date(order_date)
  ) %>%
  
  # Create total sales variable
  mutate(
    total_sales = sales_2023 + sales_2024
  )

clean_sales
```

---

# Exporting Cleaned Data

Save cleaned data:

```r
write_csv(
  clean_sales,
  "data/clean_sales.csv"
)
```

---

# Practice Exercises

## Exercise 1

Select only:

- customer name
- product
- sales_2024

---

## Exercise 2

Filter rows where:

```r
Region == "North"
```

---

## Exercise 3

Create a variable called:

```r
sales_difference
```

equal to:

```r
Sales_2024 - Sales_2023
```

---

## Exercise 4

Calculate average sales by region.

---

## Exercise 5

Convert the dataset from wide format to long format.

---

## Exercise 6

Replace missing values with 0.

---

## Exercise 7

Remove duplicated rows.

---

# Practice Solutions

## Solution 1

```r
sales %>%
  select(
    `Customer Name`,
    Product,
    Sales_2024
  )
```

---

## Solution 2

```r
sales %>%
  filter(Region == "North")
```

---

## Solution 3

```r
sales %>%
  mutate(
    sales_difference =
      Sales_2024 - Sales_2023
  )
```

---

## Solution 4

```r
sales %>%
  group_by(Region) %>%
  summarise(
    average_sales =
      mean(Sales_2024, na.rm = TRUE)
  )
```

---

## Solution 5

```r
sales %>%
  pivot_longer(
    cols = c(Sales_2023, Sales_2024),
    names_to = "year",
    values_to = "sales"
  )
```

---

## Solution 6

```r
sales %>%
  mutate(
    Sales_2023 = replace_na(Sales_2023, 0),
    Sales_2024 = replace_na(Sales_2024, 0)
  )
```

---

## Solution 7

```r
sales %>%
  distinct()
```

---

# Summary

In this tutorial, you learned how to:

- import datasets
- inspect data structures
- manipulate data using `dplyr`
- reshape data using `tidyr`
- handle missing values
- remove duplicated rows
- create reproducible cleaning workflows

These skills form the foundation of data science and transcriptomics analysis in R.

---

# Recommended Next Topics

After mastering data handling and cleaning, consider learning:

- data visualization with `ggplot2`
- exploratory data analysis
- statistical analysis in R
- reproducible reports with Quarto
- transcriptomics analysis
- RNA-seq workflows

---

# References

- Wickham H. et al. *R for Data Science*
- tidyverse documentation
- dplyr documentation
- tidyr documentation