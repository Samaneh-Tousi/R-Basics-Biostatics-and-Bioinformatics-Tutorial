# 1. Install R, RStudio, and some R packages

R is the programming language we will use in this tutorial.



- For your **UHasselt laptop/PC**, please install R/RStudio via the **UHasselt Software Center**. Please see the [installation instructions](../assets/UHasselt%20Software%20Center.pdf).



- For your **personal laptop/PC** you can go to the official R/R/RStudio websites:


<https://cran.r-project.org/>

Download and install the correct version for your operating system:

- Windows
- macOS
- Linux

After installation, right click on R at C:\Program Files\R\R-4.4.2\bin (replace the R version with your installed R version at the adress) and run as administrator to check whether it works.  

```r
version
```

You should see information about your installed R version.

# 2. Install RStudio

RStudio is an editor that makes it easier to write and run R code.

Download a non-commercial RStudio Desktop from:

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

# 5. Install and load R packages

R contains many built-in functions, but additional functions are available through packages.

A package normally needs to be installed only once on your computer.

For example:

```r
install.packages("ggplot2")
```

After installation, the package needs to be loaded before you can use it:

```r
library(ggplot2)
```

The difference is:

**install.packages()** installs a package on your computer and normally needs to be run only once.
**library()** loads an installed package into the current R session and needs to be run again when you restart R.

Packages are usually loaded near the beginning of an R script.


