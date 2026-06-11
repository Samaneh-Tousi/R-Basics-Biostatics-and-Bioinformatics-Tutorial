#Programming Basics in R: Functions, Conditions, and Loops"

# Learning objectives

By the end of this session, you will be able to:

- Understand why functions, conditions, and loops are useful in R
- Write simple R functions
- Use `if`, `else if`, and `else` for decision-making
- Use `for` loops to repeat tasks
- Understand the basic idea of `while` loops
- Apply functions, conditions, and loops to biomedical examples
- Understand when vectorized R code is better than loops

---

# 1. Why do we need programming basics?

As your R analysis becomes longer, you will often repeat similar steps.

For example, you may want to:

- calculate the mean expression of several genes
- classify samples based on expression level
- repeat the same analysis for multiple variables
- apply quality-control rules to many samples or cells
- avoid copying and pasting the same code many times

Programming basics help you write code that is:

- reusable
- cleaner
- easier to debug
- easier to share
- less repetitive

In this session, we focus on three important programming concepts:

- functions
- conditions
- loops

These concepts are useful in almost every type of R analysis, including basic statistics, RNA-seq, and single-cell RNA-seq workflows.

---

# 2. Setup

For this session, we use base R and the `tidyverse`.

```{r}
# Install only once if needed:
# install.packages("tidyverse")

library(tidyverse)
```

---

# 3. Example biomedical dataset

We will create a small example dataset representing gene expression values from different samples.

```{r}
gene_expression <- tibble(
  sample_id = paste0("Sample_", 1:8),
  condition = c(
    "Control", "Control", "Control", "Control",
    "Treatment", "Treatment", "Treatment", "Treatment"
  ),
  TP53 = c(8.2, 7.9, 8.5, 8.1, 12.4, 11.9, 12.8, 13.1),
  MYC = c(5.1, 5.4, 5.2, 5.0, 9.8, 10.2, 9.5, 10.0),
  GAPDH = c(15.2, 15.5, 15.1, 15.3, 15.4, 15.6, 15.2, 15.5)
)

gene_expression
```

This dataset contains:

- `sample_id`: sample name
- `condition`: experimental group
- `TP53`, `MYC`, and `GAPDH`: example gene expression values

---

# 4. Functions in R

## 4.1 What is a function?

A function is a reusable block of code that performs a specific task.

You have already used many built-in R functions, such as:

```{r}
mean(c(1, 2, 3, 4, 5))

sum(c(1, 2, 3, 4, 5))

log2(8)

head(gene_expression)
```

Each function takes an input, does something, and usually returns an output.

For example:

```{r}
mean(c(1, 2, 3))
```

The input is:

```r
c(1, 2, 3)
```

The output is:

```text
2
```

---

## 4.2 Why write your own functions?

Writing your own functions is useful when you repeat the same code multiple times.

For example:

```{r}
mean(gene_expression$TP53)

mean(gene_expression$MYC)

mean(gene_expression$GAPDH)
```

This works, but if we have many genes, repeating the same code becomes inefficient.

A function allows us to write the logic once and reuse it.

---

## 4.3 Basic structure of a function

The basic structure of a function is:

```r
function_name <- function(argument) {
  code_to_run
}
```

Example:

```{r}
calculate_mean <- function(x) {
  mean(x)
}

calculate_mean(gene_expression$TP53)
```

Here:

- `calculate_mean` is the function name
- `x` is the input argument
- `mean(x)` is the code inside the function
- the function returns the mean of `x`

---

## 4.4 Function with one argument

Let us write a function that calculates the mean expression of a gene.

```{r}
mean_expression <- function(expression_values) {
  mean(expression_values)
}

mean_expression(gene_expression$TP53)

mean_expression(gene_expression$MYC)

mean_expression(gene_expression$GAPDH)
```

This function can be reused for any numeric vector.

---

## 4.5 Function with multiple arguments

Functions can have more than one argument.

For example, let us write a function that calculates fold change between two values.

```{r}
calculate_fold_change <- function(treatment_mean, control_mean) {
  treatment_mean / control_mean
}

calculate_fold_change(
  treatment_mean = 12.55,
  control_mean = 8.18
)
```

---

## 4.6 Function for log2 fold change

In RNA-seq analysis, we often use log2 fold change.

The formula is:

```r
log2(treatment_mean / control_mean)
```

Now we can write a function:

```{r}
calculate_log2fc <- function(treatment_mean, control_mean) {
  log2(treatment_mean / control_mean)
}

calculate_log2fc(
  treatment_mean = 12.55,
  control_mean = 8.18
)
```

---

## 4.7 Applying a function to the example data

First, calculate the mean expression of `TP53` in each condition.

```{r}
tp53_summary <- gene_expression |>
  group_by(condition) |>
  summarise(
    mean_TP53 = mean(TP53)
  )

tp53_summary
```

Now calculate log2 fold change using the function.

```{r}
tp53_control_mean <- tp53_summary |>
  filter(condition == "Control") |>
  pull(mean_TP53)

tp53_treatment_mean <- tp53_summary |>
  filter(condition == "Treatment") |>
  pull(mean_TP53)

calculate_log2fc(
  treatment_mean = tp53_treatment_mean,
  control_mean = tp53_control_mean
)
```

---

## 4.8 Returning values from a function

In R, the last line of a function is returned automatically.

```{r}
add_two_numbers <- function(a, b) {
  a + b
}

add_two_numbers(5, 3)
```

You can also use `return()` explicitly.

```{r}
add_two_numbers_explicit <- function(a, b) {
  return(a + b)
}

add_two_numbers_explicit(5, 3)
```

Both approaches work.

In simple R functions, many R users prefer returning the last line automatically.

---

# 5. Conditional statements: if, else if, and else

## 5.1 What is a condition?

A condition allows R to make a decision.

For example:

```text
If expression is greater than 10, classify it as highly expressed.
Otherwise, classify it as lowly expressed.
```

In R, this can be written using `if` and `else`.

---

## 5.2 Basic if statement

```{r}
expression_value <- 12

if (expression_value > 10) {
  print("Highly expressed")
}
```

The message is printed because `expression_value > 10` is `TRUE`.

Now try a lower value:

```{r}
expression_value <- 5

if (expression_value > 10) {
  print("Highly expressed")
}
```

Nothing is printed because the condition is `FALSE`.

---

## 5.3 if and else

Use `else` when you want something to happen if the condition is not true.

```{r}
expression_value <- 5

if (expression_value > 10) {
  print("Highly expressed")
} else {
  print("Lowly expressed")
}
```

---

## 5.4 if, else if, and else

Use `else if` when you have more than two possible decisions.

```{r}
expression_value <- 8

if (expression_value > 10) {
  print("Highly expressed")
} else if (expression_value >= 5) {
  print("Moderately expressed")
} else {
  print("Lowly expressed")
}
```

This gives three possible categories:

- highly expressed
- moderately expressed
- lowly expressed

---

## 5.5 A function using if and else

Now we can combine functions and conditions.

```{r}
classify_expression <- function(expression_value) {
  if (expression_value > 10) {
    "Highly expressed"
  } else if (expression_value >= 5) {
    "Moderately expressed"
  } else {
    "Lowly expressed"
  }
}

classify_expression(12)

classify_expression(8)

classify_expression(3)
```

This function takes one expression value and returns a category.

---

## 5.6 Applying the function to a data frame

We can use our function with `mutate()`.

```{r}
gene_expression_classified <- gene_expression |>
  mutate(
    TP53_category = map_chr(TP53, classify_expression),
    MYC_category = map_chr(MYC, classify_expression),
    GAPDH_category = map_chr(GAPDH, classify_expression)
  )

gene_expression_classified
```

Here, `map_chr()` applies the function to each value and returns character output.

---

## 5.7 Using if_else() in dplyr

For simple two-level decisions, `dplyr::if_else()` is often easier.

```{r}
gene_expression |>
  mutate(
    TP53_status = if_else(TP53 > 10, "High", "Not high")
  )
```

Use `if_else()` inside data frames when you want to create a new column based on a condition.

---

## 5.8 Using case_when() for multiple conditions

For more than two categories, `case_when()` is often clearer than many nested `if_else()` statements.

```{r}
gene_expression |>
  mutate(
    TP53_category = case_when(
      TP53 > 10 ~ "Highly expressed",
      TP53 >= 5 ~ "Moderately expressed",
      TRUE ~ "Lowly expressed"
    )
  )
```

The `TRUE ~` line means:

```text
If none of the previous conditions are true, use this value.
```

---

# 6. Loops in R

## 6.1 What is a loop?

A loop repeats code several times.

For example, you may want to print the names of several genes:

```{r}
genes <- c("TP53", "MYC", "GAPDH")

for (gene in genes) {
  print(gene)
}
```

This loop says:

```text
For each gene in genes, print the gene name.
```

---

## 6.2 Basic for loop

The structure of a `for` loop is:

```r
for (item in collection) {
  code_to_repeat
}
```

Example:

```{r}
numbers <- c(1, 2, 3, 4, 5)

for (number in numbers) {
  print(number)
}
```

---

## 6.3 For loop with calculation

```{r}
for (number in numbers) {
  squared_number <- number^2
  print(squared_number)
}
```

---

## 6.4 Looping over genes

Let us calculate the mean expression of each gene.

```{r}
genes <- c("TP53", "MYC", "GAPDH")

for (gene in genes) {
  gene_mean <- mean(gene_expression[[gene]])
  print(paste("Mean expression of", gene, "is", round(gene_mean, 2)))
}
```

Important:

```r
gene_expression[[gene]]
```

is used to select a column when the column name is stored as a character value.

For example:

```{r}
gene <- "TP53"

gene_expression[[gene]]
```

---

## 6.5 Saving loop results

Printing results is useful, but often we want to save the results.

```{r}
mean_results <- tibble(
  gene = character(),
  mean_expression = numeric()
)

for (gene in genes) {
  gene_mean <- mean(gene_expression[[gene]])
  
  mean_results <- bind_rows(
    mean_results,
    tibble(
      gene = gene,
      mean_expression = gene_mean
    )
  )
}

mean_results
```

This creates an empty results table and adds one row during each loop.

---

## 6.6 Better approach: using summarise()

For many data frame operations, `dplyr` can be cleaner than a loop.

```{r}
gene_expression |>
  summarise(
    across(
      .cols = c(TP53, MYC, GAPDH),
      .fns = mean
    )
  )
```

This calculates the mean for multiple columns.

---

## 6.7 Looping by condition

Now let us calculate the mean expression of each gene in each condition.

```{r}
condition_gene_means <- tibble(
  condition = character(),
  gene = character(),
  mean_expression = numeric()
)

conditions <- unique(gene_expression$condition)

for (condition_name in conditions) {
  for (gene in genes) {
    
    subset_data <- gene_expression |>
      filter(condition == condition_name)
    
    gene_mean <- mean(subset_data[[gene]])
    
    condition_gene_means <- bind_rows(
      condition_gene_means,
      tibble(
        condition = condition_name,
        gene = gene,
        mean_expression = gene_mean
      )
    )
  }
}

condition_gene_means
```

This is an example of a nested loop.

A nested loop means one loop inside another loop.

---

## 6.8 Cleaner tidyverse alternative

The same result can be produced with `pivot_longer()` and `summarise()`.

```{r}
condition_gene_means_tidy <- gene_expression |>
  pivot_longer(
    cols = c(TP53, MYC, GAPDH),
    names_to = "gene",
    values_to = "expression"
  ) |>
  group_by(condition, gene) |>
  summarise(
    mean_expression = mean(expression),
    .groups = "drop"
  )

condition_gene_means_tidy
```

In many real analyses, this tidyverse approach is easier to read than nested loops.

However, learning loops is still important because they help you understand programming logic.

---

# 7. Combining functions, conditions, and loops

Now we will combine all three concepts.

Goal:

```text
For each gene, calculate mean expression.
Then classify the gene as high, moderate, or low expression.
```

First, write a function.

```{r}
classify_mean_expression <- function(mean_value) {
  if (mean_value > 10) {
    "High mean expression"
  } else if (mean_value >= 5) {
    "Moderate mean expression"
  } else {
    "Low mean expression"
  }
}
```

Now use the function inside a loop.

```{r}
gene_summary <- tibble(
  gene = character(),
  mean_expression = numeric(),
  expression_category = character()
)

for (gene in genes) {
  gene_mean <- mean(gene_expression[[gene]])
  gene_category <- classify_mean_expression(gene_mean)
  
  gene_summary <- bind_rows(
    gene_summary,
    tibble(
      gene = gene,
      mean_expression = gene_mean,
      expression_category = gene_category
    )
  )
}

gene_summary
```

This example uses:

- a function
- an if/else condition
- a for loop
- a results table

---

# 8. while loops

## 8.1 What is a while loop?

A `while` loop repeats code as long as a condition is true.

The structure is:

```r
while (condition_is_true) {
  code_to_repeat
}
```

Example:

```{r}
counter <- 1

while (counter <= 5) {
  print(counter)
  counter <- counter + 1
}
```

This loop continues while `counter <= 5`.

---

## 8.2 Be careful with while loops

A `while` loop can accidentally run forever if the condition never becomes false.

Do not run this:

```r
counter <- 1

while (counter <= 5) {
  print(counter)
}
```

In this example, `counter` never changes, so the loop never stops.

Always make sure something inside the loop changes the condition.

---

## 8.3 Example: simple quality-control threshold

Imagine we want to increase a threshold until enough samples pass.

```{r}
qc_values <- c(1000, 1500, 2200, 3000, 4500, 5000)

threshold <- 1000
samples_passing <- 0

while (samples_passing < 4) {
  samples_passing <- sum(qc_values >= threshold)
  
  print(paste(
    "Threshold:",
    threshold,
    "- Samples passing:",
    samples_passing
  ))
  
  threshold <- threshold + 500
}
```

This is a simple example to show how `while` loops work.

In real quality-control workflows, thresholds should be chosen carefully based on the biology, data distribution, and analysis goal.

---

# 9. Vectorization in R

## 9.1 What is vectorization?

R is designed to work with vectors.

Many operations can be applied to an entire vector without writing a loop.

Example:

```{r}
values <- c(1, 2, 3, 4, 5)

values * 2
```

This multiplies every value by 2.

You do not need to write:

```{r}
for (value in values) {
  print(value * 2)
}
```

Both work, but vectorized code is usually shorter and easier to read.

---

## 9.2 Vectorized log transformation

In transcriptomics, expression values are often log-transformed.

```{r}
counts <- c(0, 10, 50, 100, 500, 1000)

log2(counts + 1)
```

We add 1 because `log2(0)` is undefined.

---

## 9.3 Vectorized classification with case_when()

```{r}
gene_expression |>
  mutate(
    TP53_category = case_when(
      TP53 > 10 ~ "High",
      TP53 >= 5 ~ "Moderate",
      TRUE ~ "Low"
    )
  )
```

This is often preferred to writing a loop for simple column-wise classification.

---

# 10. Common mistakes

## 10.1 Forgetting parentheses in function definitions

Incorrect:

```r
my_function <- function x {
  mean(x)
}
```

Correct:

```r
my_function <- function(x) {
  mean(x)
}
```

---

## 10.2 Forgetting curly brackets

Incorrect:

```r
if (x > 10)
  print("High")
else
  print("Low")
```

This may work for one-line expressions, but it is not recommended for beginners.

Better:

```r
if (x > 10) {
  print("High")
} else {
  print("Low")
}
```

Curly brackets make your code clearer and safer.

---

## 10.3 Using `=` instead of `==` in conditions

Incorrect:

```r
if (condition = "Control") {
  print("Control sample")
}
```

Correct:

```r
condition <- "Control"

if (condition == "Control") {
  print("Control sample")
}
```

Remember:

- `<-` assigns a value
- `=` can also assign values in some cases, especially inside function arguments
- `==` tests equality

---

## 10.4 Forgetting to update the counter in a while loop

Incorrect:

```r
counter <- 1

while (counter <= 5) {
  print(counter)
}
```

Correct:

```r
counter <- 1

while (counter <= 5) {
  print(counter)
  counter <- counter + 1
}
```

---

## 10.5 Using a loop when a vectorized solution is easier

This works:

```{r}
values <- c(1, 2, 3, 4, 5)

doubled_values <- c()

for (value in values) {
  doubled_values <- c(doubled_values, value * 2)
}

doubled_values
```

But this is simpler:

```{r}
values * 2
```

---

# 11. Mini project: classify genes by expression change

In this mini project, we will:

1. calculate mean expression for each gene in each condition
2. calculate log2 fold change
3. classify each gene as upregulated, downregulated, or unchanged

---

## 11.1 Prepare data in long format

```{r}
expression_long <- gene_expression |>
  pivot_longer(
    cols = c(TP53, MYC, GAPDH),
    names_to = "gene",
    values_to = "expression"
  )

expression_long
```

---

## 11.2 Calculate mean expression by condition and gene

```{r}
mean_expression_by_condition <- expression_long |>
  group_by(condition, gene) |>
  summarise(
    mean_expression = mean(expression),
    .groups = "drop"
  )

mean_expression_by_condition
```

---

## 11.3 Convert to wide format

```{r}
mean_expression_wide <- mean_expression_by_condition |>
  pivot_wider(
    names_from = condition,
    values_from = mean_expression
  )

mean_expression_wide
```

---

## 11.4 Write a function to calculate log2 fold change

```{r}
calculate_log2fc <- function(treatment, control) {
  log2(treatment / control)
}
```

---

## 11.5 Write a function to classify gene regulation

```{r}
classify_regulation <- function(log2fc) {
  if (log2fc > 1) {
    "Upregulated"
  } else if (log2fc < -1) {
    "Downregulated"
  } else {
    "Unchanged"
  }
}
```

---

## 11.6 Apply the functions

```{r}
gene_regulation_results <- mean_expression_wide |>
  mutate(
    log2FC = calculate_log2fc(
      treatment = Treatment,
      control = Control
    ),
    regulation = map_chr(log2FC, classify_regulation)
  )

gene_regulation_results
```

---

## 11.7 Plot the result

```{r}
ggplot(gene_regulation_results, aes(x = gene, y = log2FC, fill = regulation)) +
  geom_col() +
  geom_hline(yintercept = 0, linetype = "dashed") +
  labs(
    title = "Example gene regulation based on log2 fold change",
    x = "Gene",
    y = "log2 fold change",
    fill = "Regulation"
  ) +
  theme_minimal()
```

---

# 12. Practice exercises

## Exercise 1

Create a function called `calculate_range()` that calculates the difference between the maximum and minimum value of a numeric vector.

Example input:

```r
c(4, 8, 10, 15)
```

Expected output:

```text
11
```

Hint:

```r
max(x) - min(x)
```

---

## Exercise 2

Create a function called `classify_p_value()`.

The function should:

- return `"Significant"` if the p-value is less than 0.05
- return `"Not significant"` otherwise

Test your function with:

```r
0.01
0.2
0.049
0.05
```

---

## Exercise 3

Use a `for` loop to print the mean expression of each gene:

```r
TP53
MYC
GAPDH
```

---

## Exercise 4

Create a function called `classify_cell_quality()`.

The function should classify cells based on detected gene number:

- fewer than 200 genes: `"Low quality"`
- 200 to 2500 genes: `"Good quality"`
- more than 2500 genes: `"Possible doublet"`

Test it with:

```r
c(100, 500, 1800, 3000)
```

---

## Exercise 5

Create a small data frame:

```r
cell_qc <- tibble(
  cell_id = paste0("Cell_", 1:6),
  detected_genes = c(150, 300, 1200, 2400, 2800, 5000)
)
```

Use your `classify_cell_quality()` function to add a new column called `qc_status`.

---

# 13. Exercise solutions

## Solution 1

```{r}
calculate_range <- function(x) {
  max(x) - min(x)
}

calculate_range(c(4, 8, 10, 15))
```

---

## Solution 2

```{r}
classify_p_value <- function(p_value) {
  if (p_value < 0.05) {
    "Significant"
  } else {
    "Not significant"
  }
}

classify_p_value(0.01)

classify_p_value(0.2)

classify_p_value(0.049)

classify_p_value(0.05)
```

Note that a p-value equal to `0.05` is classified as `"Not significant"` because the condition is:

```r
p_value < 0.05
```

not:

```r
p_value <= 0.05
```

---

## Solution 3

```{r}
genes <- c("TP53", "MYC", "GAPDH")

for (gene in genes) {
  gene_mean <- mean(gene_expression[[gene]])
  print(paste("Mean expression of", gene, "is", round(gene_mean, 2)))
}
```

---

## Solution 4

```{r}
classify_cell_quality <- function(detected_genes) {
  if (detected_genes < 200) {
    "Low quality"
  } else if (detected_genes <= 2500) {
    "Good quality"
  } else {
    "Possible doublet"
  }
}

classify_cell_quality(100)

classify_cell_quality(500)

classify_cell_quality(1800)

classify_cell_quality(3000)
```

---

## Solution 5

```{r}
cell_qc <- tibble(
  cell_id = paste0("Cell_", 1:6),
  detected_genes = c(150, 300, 1200, 2400, 2800, 5000)
)

cell_qc <- cell_qc |>
  mutate(
    qc_status = map_chr(detected_genes, classify_cell_quality)
  )

cell_qc
```

---

# 14. Summary

In this session, we learned that:

- Functions help us write reusable code
- Conditions allow R to make decisions
- `if`, `else if`, and `else` are used for conditional logic
- Loops repeat code several times
- `for` loops are useful when you know what you want to iterate over
- `while` loops repeat code while a condition remains true
- R is vectorized, so many operations do not require loops
- `if_else()` and `case_when()` are useful for conditional operations inside data frames
- Functions, conditions, and loops are useful for biomedical and transcriptomics workflows

---

# 15. Key take-home message

Programming basics help you move from running individual R commands to writing reusable and flexible analysis workflows.

In bioinformatics, this is important because we often repeat similar steps across:

- many genes
- many samples
- many cell types
- many conditions
- many comparisons

Learning functions, conditions, and loops will make your R code more powerful, cleaner, and easier to maintain.