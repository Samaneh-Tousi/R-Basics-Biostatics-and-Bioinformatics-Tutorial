# R Markdown Tutorial

## Introduction

**R Markdown** is a file format that allows you to combine:

- text
- R code
- code output
- tables
- figures
- explanations

in one document.

R Markdown is widely used for:

- data analysis reports
- reproducible research
- tutorials
- assignments
- scientific documents
- GitHub documentation

---

## What Is R Markdown?

R Markdown files usually have the extension:

```text
.Rmd
```

An R Markdown document contains three main parts:

1. A YAML header
2. Markdown text
3. R code chunks

Example:

```yaml
---
title: "My R Markdown Document"
author: "Your Name"
date: "2026-05-26"
output: html_document
---
```

---

## Why Use R Markdown?

R Markdown helps you create reproducible documents.

This means that your code, results, and explanations are stored together in one file.

Instead of writing code in one place and copying results into another document, R Markdown allows you to generate everything automatically.

---

## Installing Required Packages

You only need to install packages **one time**.

After installation, you only need to load them with `library()`.

Repeatedly reinstalling packages that already exist may sometimes cause unnecessary errors or version conflicts.

```r
install.packages("rmarkdown")
install.packages("knitr")
install.packages("tidyverse")
```

---

## Loading Packages

After packages are installed, load them using:

```r
library(rmarkdown)
library(knitr)
library(tidyverse)
```

---

## Creating an R Markdown File in RStudio

To create an R Markdown file in RStudio:

1. Open RStudio
2. Click **File**
3. Click **New File**
4. Click **R Markdown**
5. Add a title and author
6. Choose an output format
7. Click **OK**

RStudio will create a new `.Rmd` file.

---

## Basic Structure of an R Markdown File

A simple R Markdown file looks like this:

````rmd
---
title: "My First R Markdown Report"
author: "Your Name"
date: "2026-05-26"
output: html_document
---

# Introduction

This is my first R Markdown document.

## R Code Example

```{r}
x <- 10
y <- 20
x + y
```
````

---

## The YAML Header

The YAML header appears at the top of an R Markdown file.

It is placed between three dashes:

```yaml
---
title: "My Report"
author: "Your Name"
date: "2026-05-26"
output: html_document
---
```

The YAML header controls document information and output format.

Common output formats include:

```yaml
output: html_document
```

```yaml
output: pdf_document
```

```yaml
output: word_document
```

---

## Markdown Text Formatting

You can write normal text in R Markdown using Markdown syntax.

---

## Headings

Use `#` symbols to create headings.

```markdown
# Main Title

## Section Title

### Subsection Title

#### Smaller Heading
```

---

## Bold and Italic Text

```markdown
**This text is bold**

*This text is italic*

***This text is bold and italic***
```

Output:

**This text is bold**

*This text is italic*

***This text is bold and italic***

---

## Lists

### Bullet List

```markdown
- R
- RStudio
- R Markdown
- GitHub
```

Output:

- R
- RStudio
- R Markdown
- GitHub

---

### Numbered List

```markdown
1. Install R
2. Install RStudio
3. Install packages
4. Create an R Markdown file
```

Output:

1. Install R
2. Install RStudio
3. Install packages
4. Create an R Markdown file

---

## Links

```markdown
[R Project](https://www.r-project.org/)
```

Output:

[R Project](https://www.r-project.org/)

---

## Images

You can add images using Markdown:

```markdown
![RStudio Interface](../assets/rstudio-interface.png)
```

You can also control the image size using HTML:

```html
<p align="center">
  <img src="../assets/rstudio-interface.png" width="700">
</p>

```

---

## R Code Chunks

R code chunks allow you to run R code inside an R Markdown document.

A code chunk looks like this:

````rmd
```{r}
x <- 5
x * 2
```
````

When the document is knitted, R runs the code and shows the output.

---

## Naming Code Chunks

You can give each chunk a name.

````rmd
```{r calculate-mean}
scores <- c(85, 90, 78, 92, 88)
mean(scores)
```
````

Chunk names should be:

- short
- descriptive
- unique
- written without spaces

Good examples:

```text
load-packages
import-data
clean-data
plot-results
```

---

## Code Chunk Options

R Markdown allows you to control how code and output appear.

Common chunk options include:

| Option | Meaning |
|---|---|
| `echo = TRUE` | Show the code |
| `echo = FALSE` | Hide the code |
| `eval = TRUE` | Run the code |
| `eval = FALSE` | Do not run the code |
| `message = FALSE` | Hide messages |
| `warning = FALSE` | Hide warnings |
| `include = FALSE` | Run code but hide code and output |

---

## Example: Show Code and Output

````rmd
```{r}
x <- c(1, 2, 3, 4, 5)
mean(x)
```
````

---

## Example: Hide Code but Show Output

````rmd
```{r, echo=FALSE}
x <- c(1, 2, 3, 4, 5)
mean(x)
```
````

---

## Example: Hide Messages and Warnings

````rmd
```{r, message=FALSE, warning=FALSE}
library(tidyverse)
```
````

---

## Setup Chunk

Most R Markdown documents include a setup chunk near the beginning.

````rmd
```{r setup, include=FALSE}
knitr::opts_chunk$set(
  echo = TRUE,
  message = FALSE,
  warning = FALSE
)
```
````

This sets default options for all code chunks.

---

# Complete R Markdown Example

The following is a complete R Markdown document.

You can save it as:

```text
example-report.Rmd
```

````rmd
---
title: "Example R Markdown Report"
author: "Your Name"
date: "2026-05-26"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(
  echo = TRUE,
  message = FALSE,
  warning = FALSE
)
```

# Introduction

This report demonstrates the basic use of R Markdown.

We will create a small biomedical dataset, summarize it, and visualize it.

# Load Packages

```{r load-packages}
library(tidyverse)
library(knitr)
```

# Create Example Dataset

```{r create-data}
patients <- data.frame(
  patient_id = paste0("P", 1:12),
  group = rep(c("Control", "Treatment"), each = 6),
  age = c(34, 45, 29, 52, 41, 38, 36, 47, 31, 55, 43, 39),
  gene_expression = c(5.2, 4.8, 5.5, 5.1, 4.9, 5.3, 7.1, 6.8, 7.5, 7.2, 6.9, 7.4)
)

patients
```

# View the Data

```{r view-data}
kable(patients)
```

# Summary Statistics

```{r summary-statistics}
patients %>%
  group_by(group) %>%
  summarise(
    mean_age = mean(age),
    sd_age = sd(age),
    mean_gene_expression = mean(gene_expression),
    sd_gene_expression = sd(gene_expression)
  )
```

# Data Visualization

```{r plot-expression}
ggplot(patients, aes(x = group, y = gene_expression)) +
  geom_boxplot() +
  geom_jitter(width = 0.1, size = 2) +
  labs(
    title = "Gene Expression by Group",
    x = "Group",
    y = "Gene Expression"
  ) +
  theme_minimal()
```

# Statistical Test

```{r statistical-test}
t.test(gene_expression ~ group, data = patients)
```

# Interpretation

The treatment group appears to have higher gene expression values compared with the control group.

The t-test can be used to evaluate whether the difference between the two groups is statistically significant.

# Conclusion

This document shows how R Markdown can combine text, code, tables, figures, and statistical results in one reproducible report.
````

---

# Working With Tables

You can create simple tables in Markdown.

```markdown
| Variable | Description |
|---|---|
| patient_id | Unique patient identifier |
| group | Experimental group |
| age | Patient age |
| gene_expression | Expression value of a selected gene |
```

Output:

| Variable | Description |
|---|---|
| patient_id | Unique patient identifier |
| group | Experimental group |
| age | Patient age |
| gene_expression | Expression value of a selected gene |

---

## Creating Tables From R Data

Use `knitr::kable()` to display a clean table.

```r
library(knitr)

patients <- data.frame(
  patient_id = paste0("P", 1:5),
  group = c("Control", "Control", "Treatment", "Treatment", "Treatment"),
  expression = c(5.1, 5.3, 7.2, 6.9, 7.4)
)

kable(patients)
```

---

# Inline R Code

Inline R code allows you to include R results inside a sentence.

Example:

````markdown
The mean value is `r mean(c(1, 2, 3, 4, 5))`.
````

When knitted, this becomes:

```text
The mean value is 3.
```

Inline R code is useful for reporting statistics automatically.

---

# Adding Plots

You can create plots directly inside R Markdown.

````rmd
```{r}
library(ggplot2)

ggplot(mtcars, aes(x = wt, y = mpg)) +
  geom_point() +
  labs(
    title = "Relationship Between Car Weight and MPG",
    x = "Weight",
    y = "Miles per Gallon"
  ) +
  theme_minimal()
```
````

---

# Saving Output Files

R Markdown can generate several output types.

## HTML Output

```yaml
output: html_document
```

HTML is useful for:

- websites
- GitHub pages
- interactive reports

---

## Word Output

```yaml
output: word_document
```

Word output is useful for:

- reports
- assignments
- documents that need editing

---

## PDF Output

```yaml
output: pdf_document
```

PDF output is useful for:

- formal reports
- publications
- printable documents

To create PDF files, you may need a LaTeX installation.

You can install TinyTeX from R:

```r
install.packages("tinytex")
tinytex::install_tinytex()
```

---

# Knitting an R Markdown File

To knit an R Markdown file in RStudio:

1. Open the `.Rmd` file
2. Click the **Knit** button
3. Choose the output format
4. Wait for the document to render

RStudio will run all code chunks from top to bottom.

---

# Common R Markdown Errors

## Package Not Installed

Error:

```text
there is no package called 'tidyverse'
```

Solution:

```r
install.packages("tidyverse")
library(tidyverse)
```

Remember: install the package once, then load it each time you start a new R session.

---

## Object Not Found

Error:

```text
object 'patients' not found
```

This means the object was not created before it was used.

Make sure the code chunk that creates the object appears before the code chunk that uses it.

---

## Missing Comma or Parenthesis

Error:

```text
unexpected symbol
```

Check your code for:

- missing commas
- missing closing parentheses
- missing quotation marks
- incomplete code

---

## File Path Error

Error:

```text
cannot open file
```

This usually means R cannot find the file.

Check:

- the file name
- the folder location
- spelling
- capitalization
- your working directory

---

# Best Practices

## 1. Use Clear Section Headings

Good headings make your document easier to read.

```markdown
# Introduction
# Methods
# Results
# Discussion
# Conclusion
```

---

## 2. Name Your Code Chunks

Use descriptive chunk names.

````rmd
```{r load-packages}
library(tidyverse)
```
````

---

## 3. Keep Code Organized

Place your code in a logical order:

1. Load packages
2. Import data
3. Clean data
4. Analyze data
5. Visualize results
6. Interpret results

---

## 4. Avoid Reinstalling Packages in R Markdown

Do not put `install.packages()` inside your final R Markdown report unless the document is specifically about installation.

Instead, install packages once in the console:

```r
install.packages("tidyverse")
```

Then load them in the document:

```r
library(tidyverse)
```

---

## 5. Use Relative File Paths

For GitHub projects, use relative paths instead of absolute paths.

Good:

```r
read.csv("data/patients.csv")
```

Avoid:

```r
read.csv("C:/Users/YourName/Desktop/project/data/patients.csv")
```

Relative paths make your project easier to share.

---

# Example Biomedical Analysis in R Markdown

Below is another example using a biomedical-style dataset.

````rmd
---
title: "Biomedical Data Analysis Example"
author: "Your Name"
date: "2026-05-26"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(
  echo = TRUE,
  message = FALSE,
  warning = FALSE
)
```

# Introduction

This document demonstrates a simple biomedical data analysis using R Markdown.

The dataset contains patients from two groups: control and treatment.

# Load Packages

```{r load-packages}
library(tidyverse)
library(knitr)
```

# Create Dataset

```{r create-dataset}
biomedical_data <- data.frame(
  patient_id = paste0("Patient_", 1:20),
  group = rep(c("Control", "Treatment"), each = 10),
  age = c(45, 50, 38, 60, 55, 42, 49, 53, 47, 51,
          44, 48, 39, 57, 52, 41, 46, 54, 43, 50),
  biomarker_level = c(12.4, 11.8, 13.1, 12.9, 11.7,
                      12.2, 12.8, 13.0, 11.9, 12.5,
                      16.3, 15.9, 17.1, 16.8, 15.7,
                      16.1, 16.5, 17.0, 15.8, 16.4)
)

kable(biomedical_data)
```

# Summary Statistics

```{r summarize-data}
summary_table <- biomedical_data %>%
  group_by(group) %>%
  summarise(
    n = n(),
    mean_age = mean(age),
    sd_age = sd(age),
    mean_biomarker = mean(biomarker_level),
    sd_biomarker = sd(biomarker_level)
  )

kable(summary_table)
```

# Visualization

```{r biomarker-plot}
ggplot(biomedical_data, aes(x = group, y = biomarker_level)) +
  geom_boxplot() +
  geom_jitter(width = 0.15, size = 2) +
  labs(
    title = "Biomarker Levels by Group",
    x = "Group",
    y = "Biomarker Level"
  ) +
  theme_minimal()
```

# Statistical Analysis

```{r t-test}
t_test_result <- t.test(biomarker_level ~ group, data = biomedical_data)

t_test_result
```

# Interpretation

The treatment group has higher biomarker levels than the control group.

The t-test evaluates whether the observed difference in biomarker levels between the two groups is statistically significant.

# Conclusion

R Markdown is useful for writing reproducible biomedical analysis reports because it keeps code, results, plots, and explanations together in one document.
````

---

# Useful Keyboard Shortcuts in RStudio

| Action | Windows/Linux | Mac |
|---|---|---|
| Insert code chunk | Ctrl + Alt + I | Cmd + Option + I |
| Run current line | Ctrl + Enter | Cmd + Enter |
| Knit document | Ctrl + Shift + K | Cmd + Shift + K |
| Comment code | Ctrl + Shift + C | Cmd + Shift + C |

---

# Summary

In this tutorial, you learned:

- what R Markdown is
- how to create an R Markdown file
- how to write Markdown text
- how to use R code chunks
- how to create tables and plots
- how to use inline R code
- how to knit documents
- how to avoid common errors
- how to organize R Markdown files in a GitHub repository

R Markdown is an important tool for reproducible data analysis and scientific reporting.

---

# Practice Exercises

## Exercise 1

Create a new R Markdown file called:

```text
my-first-report.Rmd
```

Add a title, author, and date.

---

## Exercise 2

Create a vector of five numeric values and calculate the mean.

```r
values <- c(10, 15, 20, 25, 30)
mean(values)
```

---

## Exercise 3

Create a small data frame with patient IDs, treatment groups, and biomarker values.

```r
patients <- data.frame(
  patient_id = paste0("P", 1:6),
  group = c("Control", "Control", "Control", "Treatment", "Treatment", "Treatment"),
  biomarker = c(11.2, 12.1, 11.8, 15.4, 16.1, 15.8)
)

patients
```

---

## Exercise 4

Create a boxplot comparing biomarker values between groups.

```r
library(ggplot2)

ggplot(patients, aes(x = group, y = biomarker)) +
  geom_boxplot() +
  geom_jitter(width = 0.1) +
  theme_minimal()
```

---

## Exercise 5

Write a short interpretation of your results below the plot.

Example:

```markdown
The treatment group appears to have higher biomarker values than the control group.
```

---

