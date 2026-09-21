# Practice Questions for sessions 00 (installation) and 01 (R basics)

Use the R console or an R script to complete the following exercises.

## 1. Checking and Installing a Package

Suppose you want to use the package `tidyverse`.

Write R code that:

1. Checks whether `tidyverse` is already installed.
2. Installs the package only if it is not already installed.
3. Loads the package after checking the installation.

---

## 2. Create a Categorical Vector

Create a vector called `treatment_group` containing six samples.

The samples should belong to two categories:

- `"control"`
- `"treated"`

Use three control samples and three treated samples.

Then:

1. Use `str()` to inspect its structure. 
2. Convert the vector to a factor.
3. re-use `str()` to re-inspect its structure.
4. Use `levels()` to check its categories.

---

## 3. Create a Small Data Frame

Create a data frame called `glucose_data` with **6 rows and 3 columns**.

The columns should be:

- `sample`: a categorical variable containing sample IDs like `"sample_1"` and `"sample_2"`, etc
- `status`: a categorical variable containing `"control"` and `"treated"`
- `glucose_level`: a numerical variable containing glucose measurements

After creating the data frame:

1. Print the data frame.
2. Use `str()` to inspect its structure.
3. Use `dim()` to check its dimensions.
4. Use `colnames()` to check the column names.
5. check class(glucose_data$glucose_level), if its not numeric convert it to numeric
6. Choose your one or more of the numerical glucose values to print out in your console by defining a treshold value.
7. Convert the sample IDs as rownames of the dataframe and then remove the `sample` column

---

## 4. Create a Gene Expression Matrix

Imagine that you measured the expression of **6 genes in 6 samples**.

Create a numeric matrix called `gene_expression` with:

- 6 rows
- 6 columns
- numerical values only
- values filled **column by column** rather than row by row

Each **row** should represent a gene, and each **column** should represent a sample.

Then complete the following tasks:

1. Display the matrix.

2. Use `dim()` to check its dimensions.

3. Use `str()` to inspect the structure of the matrix.

4. Rename the rows as:

   - `"gene_1"`
   - `"gene_2"`
   - `"gene_3"`
   - ...
   - `"gene_6"`

   Rename the columns as:

   - `"sample_1"`
   - `"sample_2"`
   - `"sample_3"`
   - ...
   - `"sample_6"`

5. Extract the value located at **row 2, column 4**.

6. Create a smaller matrix from `gene_expression` containing:

   - the **first 4 genes**
   - the **first 3 samples**

7. Replace the expression values in the smaller matrix with **random integer values between 50 and 200**.

8. Convert the smaller matrix into a **data frame**.

9. Use `str()` to compare the structure of the smaller matrix before and after converting it to a data frame.

-----
