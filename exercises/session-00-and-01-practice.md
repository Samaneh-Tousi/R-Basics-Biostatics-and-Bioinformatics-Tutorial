# Practice Questions for sessions 00 (installation) and 01 (R basics)

Use the R console or an R script to complete the following exercises.

## 1. Checking and Installing a Package

Suppose you want to use the package `ggplot2`.

Write R code that:

1. Checks whether `ggplot2` is already installed.
2. Installs the package only if it is not already installed.
3. Loads the package after checking the installation.

**Hint:** You may find `%in%`, `installed.packages()`, `install.packages()`, and `library()` useful.

---

## 2. Create a Categorical Vector

Create a vector called `treatment_group` containing six samples.

The samples should belong to two categories:

- `"control"`
- `"treated"`

Use three control samples and three treated samples.

Then:

1. Convert the vector to a factor.
2. Use `str()` to inspect its structure.
3. Use `levels()` to check its categories.

---

## 3. Create a Small Data Frame

Create a data frame called `glucose_data` with **6 rows and 2 columns**.

The columns should be:

- `status`: a categorical variable containing `"control"` and `"treated"`
- `glucose_level`: a numerical variable containing glucose measurements

For example, your data might represent:

| status | glucose_level |
|---|---:|
| control | ? |
| control | ? |
| control | ? |
| treated | ? |
| treated | ? |
| treated | ? |

Choose your own numerical glucose values.

After creating the data frame:

1. Display the data frame.
2. Use `str()` to inspect its structure.
3. Use `dim()` to check its dimensions.
4. Use `colnames()` to check the column names.

---

## 4. Create a Gene Expression Matrix

Imagine that you measured the expression of **6 genes in 6 samples**.

Create a numeric matrix called `gene_expression` with:

- 6 rows
- 6 columns
- numerical values only

Each row should represent a gene and each column should represent a sample.

Then:

1. Display the matrix.
2. Use `dim()` to check its dimensions.
3. Use `str()` to inspect the object.
4. Extract the value from **row 2, column 4**.

---

## 5. Identify the R Objects

After completing the exercises above, determine the object type of each of the following:

- `treatment_group`
- `glucose_data`
- `gene_expression`

Use an R function to check your answers.

Think about the difference between:

- a vector
- a factor
- a data frame
- a matrix