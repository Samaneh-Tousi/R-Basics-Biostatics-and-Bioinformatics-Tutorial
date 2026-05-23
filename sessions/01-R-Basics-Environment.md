# R Basics & Environment


<p align="center">
  <img src="../assets/r_environment.png" width="600">
</p>

<p align="center">
  <em>The R environment stores objects such as x, data, and scores.</em>
</p>

The R environment stores objects such as `x`, `data`, and `scores`.

---

## Understanding the R Environment

The **R Environment** is the workspace where R stores all the objects you create during your session.

These objects can include:

- variables
- vectors
- data frames
- functions

Think of the environment as a desk with labeled boxes that hold your data and results.

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

After running the code above, the objects will appear in your environment.

---

## Viewing Objects in the Environment

To see all objects currently stored in your environment:

```r
ls()
```

Example output:

```r
[1] "x" "data" "scores"
```

---

## Removing Objects

Remove one object:

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

## Why the Environment Matters

Understanding the environment helps you:

- track your variables
- organize your analysis
- avoid overwriting objects
- debug errors more easily

---

## Practice Exercise

Try the following:

1. Create an object called `temperature`.
2. Create a vector called `ages`.
3. Run `ls()` to view your objects.
4. Remove one object using `rm()`.

---

## Key Takeaway

The R environment acts like your working desk:

- every object you create is stored there
- you can inspect, modify, or remove objects anytime
- keeping the environment organized makes coding easier