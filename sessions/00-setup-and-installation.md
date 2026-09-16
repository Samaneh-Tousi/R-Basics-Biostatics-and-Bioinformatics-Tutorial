# 1. Install R, RStudio, and some R packages

- For your <strong>UHasselt laptop/PC</strong>, please install R/RStudio via the <strong>UHasselt Software Center</strong>. Open the link in a new tab by right Click <a href="../assets/UHasselt%20Software%20Center.pdf" target="_blank" rel="noopener noreferrer">installation instructions</a>.

**Important!** If you still **cannot** see the applications **CRAN R** and **RStudio** in the **UHasselt Software Center** after following the instructions, **please submit a ticket to the UHasselt Service Desk**.

- For your **personal laptop/PC** you can go to the official R/RStudio websites:


<https://cran.r-project.org/>

Download and install the correct version for your operating system:

- Windows
- macOS
- Linux

After installation, go to the R installation folder, for example: **C:\Program Files\R\R-4.4.2\bin** (Replace R-4.4.2 with the version of R installed on your computer). Then double-click the R application file to check that R opens and works correctly. 
When R opens, you should see the R version information on the first line.

<p align="center">
  <img src="../assets/R_session.png" width="600">
</p>


# 2. Install RStudio

RStudio is an editor that makes it easier to write and run R code.

Download the free, non-commercial version of RStudio Desktop from:

<https://posit.co/download/rstudio-desktop/>

Choose the installation link that matches your operating system.

<p align="center">
  <img src="../assets/RStudio_download_links.png" width="800">
</p>


# 3. Create an RStudio project

After installation, open RStudio.

Using an RStudio project is recommended because it keeps your working directory organized.

For that, please take these steps:

```text
File > Save workspace image to ~/.RData> New Directory > New Project> write r_tutorial at Directory name > click on Browse and choose a directory to save the R project
```

This creates an RStudio project in the directory you selected.

Every time you want to practice the tutorial sessions, go to the folder where you saved the `r_tutorial` project and double-click the `r_tutorial.Rproj` file. This will open the project in RStudio.

For each session of the R tutorial, you can create a new R script by clicking the first icon in the RStudio toolbar.

When you save the script, it will be stored in your `r_tutorial` project folder. You can open it again and revise it at any time.


# 4. Install and load R packages

R contains many built-in functions, but additional functions are available through packages.

R packages are bundles of reusable R code and related materials that add capabilities to R.

A package can contain functions, datasets, documentation, and sometimes compiled code. Instead of writing everything yourself, you can install a package and use functions that someone else has already built and tested.

A package normally needs to be installed only once on your computer.

Packages are usually loaded near the beginning of an R script.

To avoid reinstalling a package that is already installed, you can first check whether the package exists:


```r
if (!requireNamespace("ggplot2", quietly = TRUE)) {
  install.packages("ggplot2")
}

library(ggplot2)
```

This code means:

If ggplot2 is not installed, install it. **quietly = TRUE** means “check for the package without printing unnecessary messages to the console, and suppresses the usual message/warning about the package not being available.”
Then load ggplot2 so it can be used in the current R session.

This is useful because reinstalling packages again and again can take time and may sometimes cause installation or version conflicts.

So:

**install.packages()** installs a package on your computer and normally needs to be run only once.

**library()** loads an installed package into the current R session and needs to be run again when you restart R.

-----

# **Further explanation about R packages sources:**

**CRAN**
**CRAN** stands for **Comprehensive R Archive Network**. It is the main online collection of **R** packages.
Most general-purpose packages, such as `ggplot2`, `dplyr`, and `readr`, are installed from **CRAN**.

Example:

`install.packages("ggplot2")`

**Bioconductor**
**Bioconductor** is another collection of R packages, but it focuses mainly on **bioinformatics** and **biological data analysis**.
It contains packages for tasks such as **RNA-seq analysis, genomics, gene expression analysis, and single-cell analysis**.

Examples of **Bioconductor** packages include `DESeq2`, `edgeR`, and `limma`.

**BiocManager**
**BiocManager** is an R package that helps **R** install and manage packages from **Bioconductor**.
In other words, **Bioconductor** is the collection of bioinformatics packages, while **BiocManager** is the tool used to install them.

First, install **BiocManager** from **CRAN**:

`install.packages("BiocManager")`

Then use it to install a **Bioconductor** package:

`BiocManager::install("DESeq2")`

means:

**BiocManager** → the package name
`::` → “use a function from this package”
install → the function name

So you can read it as:

Use the install() function from the BiocManager package.

In R, `::` means “use something from a package without loading the whole package.”


So, to summerize the R package sources:

**CRAN** → where many general R packages come from


**Bioconductor** → where many bioinformatics R packages come from


**BiocManager** → the tool used to install Bioconductor packages
