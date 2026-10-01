````markdown
# Session 02 Practice: Data Handling and Cleaning

In this exercise, you will work with a deliberately messy human health dataset and prepare it for further analysis.

The goal is to decide **which R functions and steps are appropriate yourself**, based on what you learned in Session 02.

---

## Getting the Dataset

The dataset is available in the `assets` folder of this GitHub repository.

1. Open the course repository on GitHub.
2. Open the **`assets`** folder.
3. Find the file:

```text
messy_human_health_data.csv
```

4. Download the file and save it inside the `assets` folder of your own R project.

Your project structure should look similar to:

```text
your-r-project/
├── assets/
│   └── messy_human_health_data.csv
├── sessions/
└── ...
```

You can now import the CSV file into R and start the exercises below.

---

## Practice 1 — Import and Explore the Dataset

Import `messy_human_health_data.csv` as an object called `health_data`, inspect its dimensions, column names, variable types, summary statistics, and missing values, and confirm that the original dataset contains **30 rows and 10 columns**.

---

## Practice 2 — Organize the Variables

Create a new data frame called `health_selected` with consistently named columns and keep only the following variables:

- `sample_id`
- `age`
- `sex`
- `bmi`
- `blood_sugar`
- `ldl_level`
- `smoking_status`

Your resulting data frame should contain **30 rows and 7 columns**.

---

## Practice 3 — Clean the Dataset

Starting from the full dataset, create a new data frame called `health_clean` in which completely duplicated records are removed and missing categorical information in variables such as `sex`, `smoking_status`, and `alcohol_usage` is recorded as `"Unknown"` rather than left missing.

After removing the duplicated records, your cleaned dataset should contain **27 unique samples**.

---

## Practice 4 — Create a Specific Subgroup

From `health_clean`, create a new data frame called `high_risk_group` containing only samples that meet **all** of the following conditions:

- age is **50 years or older**
- BMI is **25 or higher**
- blood sugar is **120 or higher**
- LDL level is **150 or higher**

Keep only the following variables:

```text
sample_id
age
sex
bmi
blood_sugar
ldl_level
smoking_status
```

Arrange the samples from the **highest to the lowest LDL level**.

If your filtering is correct, the resulting data frame should contain **7 samples**.

---

## Practice 5 — Create a Group Summary

Using `health_clean`, create a summary data frame with **one row for each smoking-status category**:

- `Never`
- `Former`
- `Current`
- `Unknown`

For each group, report:

- the number of samples
- the mean BMI
- the mean blood sugar
- the mean LDL level

Make sure that missing numerical measurements do not prevent the calculation of the group means.

---

## Practice 6 — Reshape and Export the Data

Create a long-format data frame containing:

- `sample_id`
- BMI
- blood sugar
- blood pressure
- LDL level

The four health measurements should be reorganized so that:

- their names appear in one column
- their numerical values appear in another column

After removing the duplicated records, the long-format data should contain **108 rows**:

```text
27 samples × 4 measurements = 108 rows
```

Finally, export your cleaned dataset to the `assets` folder as:

```text
clean_human_health_data.csv
```

---

