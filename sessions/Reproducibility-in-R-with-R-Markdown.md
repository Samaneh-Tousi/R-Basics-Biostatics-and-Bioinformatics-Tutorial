# Reproducibility in R with R Markdown

## Overview

This tutorial introduces reproducible workflows in R using **R Markdown**.

You will learn how to:

- create an organized R project
- use relative paths
- create R Markdown documents
- write text and code together
- generate reproducible reports
- knit R Markdown documents
- organize reproducible analyses

---

## 1. What is Reproducibility?

Reproducibility means that another person can:

1. access your files
2. run your code
3. generate the same results

A reproducible workflow helps:

- reduce errors
- improve transparency
- organize analyses
- make projects easier to update
- regenerate figures and tables automatically

---

## 2. Why Reproducibility Matters

Without reproducibility:

- analyses become difficult to repeat
- results may not match the original analysis
- figures may not be reproducible
- files become disorganized
- reports may require manual editing

With reproducibility:

- analyses can be rerun easily
- reports update automatically
- figures are generated directly from code
- projects remain organized
- analyses become easier to share

---

## 3. Recommended Project Structure

A reproducible R Markdown project should have a clear folder structure.

Example:

```text
reproducibility-rmarkdown/
├── reproducibility-rmarkdown.Rproj
├── README.md
├── data/
│   └── patient_data.csv
├── scripts/
│   └── 01-create-data.R
├── reports/
│   └── analysis.Rmd
├── figures/
└── output/
```

---

## 4. Folder Explanation

| Folder | Purpose |
|---|---|
| `data/` | stores datasets |
| `scripts/` | stores reusable R scripts |
| `reports/` | stores R Markdown reports |
| `figures/` | stores saved figures |
| `output/` | stores exported results |

---

## 5. Why Use an R Project?

R Projects help:

- keep all files together
- improve organization
- maintain reproducibility
- avoid broken file paths
- simplify relative paths

---

## 6. Create an R Project

In RStudio:

1. Click **File**
2. Click **New Project**
3. Click **New Directory**
4. Click **New Project**
5. Use the project name:

```text
reproducibility-rmarkdown
```

6. Click **Create Project**

---

## 7. Install Required Packages

Install packages only once:

```r
install.packages(c(
  "rmarkdown",
  "tidyverse",
  "here"
))
```

Load packages every time you start a new R session:

```r
library(rmarkdown)
library(tidyverse)
library(here)
```

You only need to install packages one time.

Repeatedly reinstalling packages that already exist is unnecessary and may occasionally cause dependency or version issues.

---

## 8. Create Project Folders

Inside your R project, create the following folders:

```text
data
scripts
reports
figures
output
```

Your project should look like this:

```text
reproducibility-rmarkdown/
├── data/
├── scripts/
├── reports/
├── figures/
└── output/
```

---

## 9. Relative Paths

Relative paths are one of the most important parts of reproducible work.

A path tells R where a file is located.

---

## 10. Absolute Paths

Avoid absolute paths like this:

```r
read.csv("C:/Users/YourName/Desktop/project/data/patient_data.csv")
```

Absolute paths only work on your computer.

They often break when:

- you move the project folder
- you share the project with someone else
- you use a different computer
- you rename folders

---

## 11. Relative Paths

Use relative paths instead:

```r
read.csv("data/patient_data.csv")
```

Relative paths start from the project folder.

This makes your work easier to move, share, and reproduce.

---

## 12. Using the `here` Package

The `here` package helps create safe file paths inside an R project.

Example:

```r
library(here)

read.csv(here("data", "patient_data.csv"))
```

The `here()` function builds paths from the root of the project.

This helps avoid path errors.

---

## 13. Create an Example Dataset

In this tutorial, we will create a small biomedical-style dataset.

```r
set.seed(123)

patient_data <- tibble(
  patient_id = 1:100,
  age = round(rnorm(100, mean = 50, sd = 10)),
  sex = sample(c("Female", "Male"), 100, replace = TRUE),
  bmi = round(rnorm(100, mean = 27, sd = 4), 1),
  glucose = round(rnorm(100, mean = 105, sd = 20), 1)
)

head(patient_data)
```

---

## 14. Save the Dataset in the `data/` Folder

Save the dataset into the `data/` folder using `here()`.

```r
write.csv(
  patient_data,
  here("data", "patient_data.csv"),
  row.names = FALSE
)
```

---

## 15. Load the Dataset Using a Relative Path

Load the dataset from the `data/` folder.

```r
patient_data <- read.csv(
  here("data", "patient_data.csv")
)

head(patient_data)
```

---

## 16. What is R Markdown?

R Markdown allows you to combine:

- text
- R code
- tables
- figures
- results

inside a single document.

R Markdown files use the extension:

```text
.Rmd
```

R Markdown can produce reports such as:

- HTML
- Word
- PDF

---

## 17. Why Use R Markdown?

R Markdown improves reproducibility because:

- code and explanation are stored together
- reports update automatically
- tables are generated from code
- figures are generated from code
- results can be regenerated
- manual copy-paste errors are reduced

---

## 18. Structure of an R Markdown File

An R Markdown file usually contains:

1. YAML header
2. markdown text
3. R code chunks

---

## 19. YAML Header

The YAML header appears at the top of the R Markdown document.

Example:

```yaml
---
title: "Reproducibility Tutorial"
author: "Your Name"
date: "`r Sys.Date()`"
output: html_document
---
```

The YAML header controls:

| Field | Meaning |
|---|---|
| `title` | report title |
| `author` | report author |
| `date` | report date |
| `output` | output format |

---

## 20. Markdown Syntax

Markdown is used to write formatted text.

Examples:

```markdown
# Main Header

## Section Header

### Smaller Header

**Bold text**

*Italic text*

- Bullet point 1
- Bullet point 2

1. Numbered item
2. Numbered item
```

---

## 21. R Code Chunks

R code chunks allow you to run R code inside the document.

A code chunk looks like this:

```text
```{r}
summary(mtcars)
```
```

When the document is knitted:

- the code runs automatically
- the output appears in the report

---

## 22. Chunk Options

Chunk options control how code and output appear in the final report.

Example:

```text
```{r, echo=FALSE}
summary(mtcars)
```
```

Common chunk options:

| Option | Meaning |
|---|---|
| `echo=TRUE` | show code in the report |
| `echo=FALSE` | hide code but show output |
| `include=FALSE` | run code but hide code and output |
| `warning=FALSE` | hide warnings |
| `message=FALSE` | hide messages |

---

## 23. Create Your First R Markdown Document

Create a file named:

```text
reports/analysis.Rmd
```

Copy the following content into `analysis.Rmd`.

```text
---
title: "Biomedical Reproducibility Report"
author: "Your Name"
date: "`r Sys.Date()`"
output: html_document
---

```{r setup, include=FALSE}
library(tidyverse)
library(here)
```

# Introduction

This report demonstrates a reproducible workflow using R Markdown.

# Load Dataset

```{r}
patient_data <- read.csv(
  here("data", "patient_data.csv")
)

head(patient_data)
```

# Summary Statistics

```{r}
summary(patient_data)
```

# Mean Glucose Level

```{r}
mean(patient_data$glucose)
```

# Data Visualization

```{r}
ggplot(patient_data, aes(x = bmi, y = glucose)) +
  geom_point(color = "steelblue", size = 3) +
  labs(
    title = "Relationship Between BMI and Glucose",
    x = "BMI",
    y = "Glucose Level"
  ) +
  theme_minimal()
```

# Group Comparison

```{r}
patient_data %>%
  group_by(sex) %>%
  summarise(
    mean_glucose = mean(glucose),
    sd_glucose = sd(glucose)
  )
```

# Conclusion

This report combines text, code, results, and figures inside one reproducible document.
```

---

## 24. Knit the R Markdown Report

Knitting converts the `.Rmd` file into a final report.

In RStudio:

1. Open `reports/analysis.Rmd`
2. Click **Knit**
3. RStudio will create an HTML file

The generated output will usually be:

```text
analysis.html
```

---

## 25. What Happens During Knitting?

When knitting:

1. R executes all code chunks
2. results are generated automatically
3. plots are created
4. tables are inserted
5. a final report is produced

This makes the report reproducible because the output comes directly from the code.

---

## 26. Benefits of Knitting

Knitting helps:

- automate report generation
- avoid copy-paste errors
- regenerate updated results
- ensure figures match the code
- improve transparency

---

## 27. Render R Markdown from the Console

You can also render an R Markdown report directly from the R console.

```r
rmarkdown::render("reports/analysis.Rmd")
```

This is useful when you want to regenerate reports without clicking the Knit button.

---

## 28. Save Figures Automatically

You can save figures inside the `figures/` folder.

```r
plot_example <- ggplot(patient_data, aes(x = bmi, y = glucose)) +
  geom_point()

ggsave(
  filename = here("figures", "bmi_glucose_plot.png"),
  plot = plot_example,
  width = 6,
  height = 4
)
```

---

## 29. Export Results

You can save processed results into the `output/` folder.

```r
summary_statistics <- patient_data %>%
  summarise(
    mean_age = mean(age),
    mean_bmi = mean(bmi),
    mean_glucose = mean(glucose)
  )

write.csv(
  summary_statistics,
  here("output", "summary_statistics.csv"),
  row.names = FALSE
)
```

---

## 30. Reproducibility Best Practices

Always:

- use one project folder
- use relative paths
- keep raw data separate
- organize files clearly
- use R Markdown for reports
- avoid manual copy-paste
- generate figures directly from code
- document your workflow

---

## 31. Common Beginner Mistakes

| Mistake | Better Practice |
|---|---|
| using desktop files | use project folders |
| using absolute paths | use relative paths |
| manually editing results | regenerate reports |
| storing all files together | organize folders |
| copying plots manually | generate plots from code |

---

## 32. Complete Workflow Summary

A reproducible R Markdown workflow looks like this:

```text
Create R Project
Create folders
Install packages once
Load packages
Create or import data
Use relative paths
Write R Markdown report
Generate tables and plots
Knit report
Export outputs
```

---

## 33. Practice Exercise

Create a new R Markdown report that includes:

- title
- introduction
- dataset import
- summary statistics
- one figure
- one table
- conclusion

Suggested dataset:

```r
mtcars
```

Example analysis:

```r
summary(mtcars)

ggplot(mtcars, aes(x = wt, y = mpg)) +
  geom_point()
```

Then:

1. Knit the report
2. Modify the code
3. Knit again
4. Observe how the report updates automatically

---

## 34. Session Information

It is good reproducibility practice to include session information at the end of your analysis.

```r
sessionInfo()
```

Session information records:

- R version
- package versions
- operating system information

This helps others understand the computing environment used for the analysis.

---

## 35. Summary

In this tutorial, you learned how to create reproducible workflows in R using R Markdown.

You covered:

- project organization
- relative paths
- the `here` package
- R Markdown structure
- YAML headers
- markdown syntax
- code chunks
- chunk options
- knitting reports
- exporting outputs
- reproducibility best practices

R Markdown provides a powerful framework for combining code, text, figures, and results into a single reproducible analysis document.

---

## 36. Final Takeaway

A reproducible R Markdown workflow keeps the analysis, explanation, code, figures, and results together.

This makes your work easier to understand, rerun, update, and share.