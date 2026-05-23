# 1. Install R, RStudio, and some R packages

R is the programming language we will use.

Go to the official R website:

<https://cran.r-project.org/>

Download and install the correct version for your operating system:

- Windows
- macOS
- Linux

After installation, open R and check that it works.

```r
version
```

You should see information about your installed R version.

# 2. Install RStudio

RStudio is an editor that makes it easier to write and run R code.

Download RStudio Desktop from:

<https://posit.co/download/rstudio-desktop/>

Install it, then open RStudio.

When RStudio opens successfully, you should see four main panels:

1. Source panel
2. Console
3. Environment / History
4. Files / Plots / Packages / Help


# 3. Create an RStudio project

Open RStudio.

Then go to:

```text
File > New Project > New Directory > New Project
```

Choose a project name, for example:

```text
r-transcriptomics-tutorial
```

This creates an `.Rproj` file.

Using an RStudio project is recommended because it keeps your working directory organized.

# 4. Check your working directory

In R, run:

```r
getwd()
```

This shows your current working directory.

If you are using an RStudio project, the working directory should be the main project folder.

For example:

```text
/path/to/r-transcriptomics-tutorial
```

# 5. Install essential R packages

We will start by installing general-purpose packages.

```r
install.packages(c(
  "tidyverse",
  "here",
  "janitor",
  "readr",
  "readxl",
  "writexl",
  "ggplot2",
  "patchwork",
  "pheatmap",
  "RColorBrewer",
  "viridis",
  "knitr",
  "rmarkdown",
  "quarto"
))
```

# 6. Install Bioconductor

Many transcriptomics packages are installed through Bioconductor.

First, install `BiocManager`:

```r
install.packages("BiocManager")
```

Then check that it works:

```r
BiocManager::version()
```

# 7. Install bulk RNA-seq packages

These packages are commonly used for bulk RNA-seq and differential expression analysis.

```r
BiocManager::install(c(
  "DESeq2",
  "edgeR",
  "limma",
  "tximport",
  "biomaRt",
  "AnnotationDbi",
  "org.Hs.eg.db",
  "clusterProfiler",
  "EnhancedVolcano"
))
```

Notes:

- `DESeq2` is commonly used for differential expression analysis.
- `edgeR` and `limma` are also widely used for RNA-seq analysis.
- `tximport` helps import transcript-level quantification.
- `clusterProfiler` is used for functional enrichment analysis.
- `EnhancedVolcano` is used to make volcano plots.

# 8. Install single-cell RNA-seq packages

For single-cell RNA-seq analysis, we will mainly use `Seurat`.

```r
install.packages("Seurat")
```

Additional useful packages:

```r
install.packages(c(
  "hdf5r",
  "Matrix",
  "patchwork"
))
```

Some single-cell tools are available through Bioconductor:

```r
BiocManager::install(c(
  "SingleCellExperiment",
  "scater",
  "scran",
  "slingshot"
))
```

# 9. Load the main packages

After installation, test that the packages can be loaded.

```r
library(tidyverse)
library(here)
library(janitor)
library(ggplot2)
library(patchwork)
```

For Bioconductor packages:

```r
library(DESeq2)
library(edgeR)
library(limma)
library(clusterProfiler)
```

For single-cell analysis:

```r
library(Seurat)
library(SingleCellExperiment)
```

If no error message appears, the packages are installed correctly.

# Important note: install packages only once

You usually need to install an R package only one time on your computer. After the package is installed, you do not need to install it again every time you open R. Instead, you only need to load the package using library().


# 10. Check your R session information

It is good practice to record your R version and package versions.

```r
sessionInfo()
```

This helps make your analysis reproducible.

At the end of every analysis, you should save the session information.

# 11. Install optional helper packages

These packages are not required at the beginning, but they are useful for larger projects.

```r
install.packages(c(
  "renv",
  "usethis",
  "devtools",
  "glue",
  "fs",
  "rio",
  "skimr"
))
```

What these packages do:

- `renv`: records package versions for reproducibility.
- `usethis`: helps create project files.
- `devtools`: helps install packages from GitHub.
- `glue`: helps combine text and variables.
- `fs`: helps work with files and folders.
- `rio`: helps import and export many file types.
- `skimr`: gives useful summaries of data frames.

# 12. Optional: initialize renv

The `renv` package helps freeze package versions in your project.

This is useful when you want others to reproduce your analysis.

```r
install.packages("renv")
renv::init()
```

After installing packages, save the package versions:

```r
renv::snapshot()
```

This creates a file called:

```text
renv.lock
```

Later, another user can restore the same package environment using:

```r
renv::restore()
```

In the next session, we will learn the basics of R, including objects, vectors, data frames, scripts, and packages.