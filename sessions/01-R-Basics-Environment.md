# R Basics & Environment

## Introduction to RStudio

Before learning R programming, it is important to understand the **RStudio interface**.

RStudio is an integrated development environment (IDE) for R that helps you:

- write code
- run analyses
- visualize results
- manage files and packages

---

## The RStudio Interface

<p align="center">
  <img src="../assets/rstudio_interface.png" width="900">
</p>

<p align="center">
  <em>Overview of the main sections of the RStudio interface.</em>
</p>

---

## Main Parts of RStudio

### 1. Source Pane (Top Left)

The **Source Pane** is where you write and edit R scripts.

You can:

- save your code
- organize analyses
- run selected lines or entire scripts

Example:

```r
x <- 5
y <- 10

x + y
```

---

### 2. Console Pane (Bottom Left)

The **Console** is where R executes commands immediately.

You can:

- test small pieces of code
- see outputs
- view warnings and errors

Example:

```r
mean(c(1, 2, 3, 4, 5))
```

Output:

```r
[1] 3
```

---

### 3. Environment Pane (Top Right)

The **Environment Pane** stores all objects created during the current R session.

These objects may include:

- variables
- vectors
- data frames
- functions

Think of the environment as a desk with labeled boxes holding your data and results.

<p align="center">
  <img src="../assets/r_environment.png" width="600">
</p>

<p align="center">
  <em>The R environment stores objects such as x, data, and scores.</em>
</p>

---

## Creating Objects in R

You can create objects using the assignment operator `<-`.

```r
x <- 5

data <- data.frame(
  id = 1:3,
  score = c(88, 92, 79)
)

scores <- c(88, 92, 79, 95)
```

After running the code above, the objects will appear in your Environment pane.

---

## Viewing Objects in the Environment

To display all current objects:

```r
ls()
```

Example output:

```r
[1] "x" "data" "scores"
```

---

## Removing Objects

Remove a single object:

```r
rm(x)
```

Remove all objects:

```r
rm(list = ls())
```

---

## Checking Object Types

```r
class(x)
class(data)
class(scores)
```

Expected output:

```r
[1] "numeric"
[1] "data.frame"
[1] "numeric"
```

---

### 4. Files, Plots, Packages, and Help Pane (Bottom Right)

This pane contains several useful tabs:

| Tab | Purpose |
|---|---|
| Files | Manage project files |
| Plots | Display graphs and visualizations |
| Packages | Install and load packages |
| Help | Access R documentation |
| Viewer | Display HTML outputs |

---

### 5. Toolbar

The toolbar provides quick access to common actions such as:

- creating new scripts
- saving files
- running code
- installing packages

---

## Why the Environment Matters

Understanding the R environment helps you:

- track your variables
- organize analyses
- avoid overwriting objects
- debug errors more easily

---

## Practice Exercise

Try the following:

1. Create an object called `temperature`.
2. Create a vector called `ages`.
3. Run `ls()` to display your objects.
4. Remove one object using `rm()`.
5. Check the class of your objects using `class()`.

---

## Key Takeaways

- RStudio helps organize coding and analysis workflows.
- The Source pane is used to write scripts.
- The Console runs commands immediately.
- The Environment stores your objects.
- The bottom-right pane helps manage files, plots, and packages.

Keeping your environment organized makes coding easier and more reproducible.