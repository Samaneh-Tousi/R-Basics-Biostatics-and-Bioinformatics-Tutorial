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

#### Assignment and Comparison Operators in R

In R, `<-` is called the **assignment operator**. It is used to store a value inside an object.

```r
x <- 5
```

This means that the value `5` is assigned to the object `x`.

The `<-` operator is preferred in R because it clearly shows the direction of assignment: the value on the right is stored in the object on the left. It is also the traditional and most commonly used style in R scripts, tutorials, and package documentation.

The `=` operator can also be used for assignment:

```r
x = 5
```

However, in R, `=` is more commonly used inside functions to specify arguments.

```r
mean(x = glucose_values, na.rm = TRUE)
```

In this example, `x = glucose_values` tells the function which data to use, and `na.rm = TRUE` tells R to remove missing values before calculating the mean.

The `==` operator is used for comparison. It checks whether two values are equal and returns `TRUE` or `FALSE`.

```r
x <- 5

x == 5
```

Output:

```r
[1] TRUE
```

Another example:

```r
x == 10
```

Output:

```r
[1] FALSE
```

| Operator | Name | Use | Example |
|---|---|---|---|
| `<-` | Assignment operator | Stores a value in an object | `x <- 5` |
| `=` | Assignment or argument setting | Often used inside functions | `mean(x = values)` |
| `==` | Equality operator | Tests whether two values are equal | `x == 5` |


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

* A variable is a named object that stores a value in R.
* A vector is a collection of values stored together in one object.


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

````R

temperature <- data.frame(city = c('Hasselt', 'Genk', 'Leuven'), temp = c(21, 22, 19))
ages <- c(9,13,16, 18, 22, 35, 42, 55, 63, 82)
ls()
class(ages)
class(temperature)
````

---

## Key Takeaways

- RStudio helps organize coding and analysis workflows.
- The Source pane is used to write scripts.
- The Console runs commands immediately.
- The Environment stores your objects.
- The bottom-right pane helps manage files, plots, and packages.

Keeping your environment organized makes coding easier and more reproducible.