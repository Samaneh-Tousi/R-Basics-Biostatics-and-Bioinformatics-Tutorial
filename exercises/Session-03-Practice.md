# Session 03 Practice: Data Visualization with ggplot2

In Session 02, you cleaned and organized the human health dataset.  
In this session, you will use that cleaned dataset to explore the data visually and create figures that could be used in a scientific report.

> [!WARNING]
> **Important Note:** This is a **small example** dataset created for practice purposes. Any patterns or conclusions observed here **should not** be generalized to larger populations or interpreted as evidence from a population-level study.

Use your saved cleaned file at your assets folder within the r tutorial project that you made at the previous session at your own computer:

```text
assets/clean_human_health_data.csv
```

Import it as:

```text
health_data
```

Before plotting, briefly inspect the dataset and make sure the variables needed for each figure are available.

---

## Practice 1. Who Is in Our Study Population?

Create a figure showing the **number of samples in each smoking-status category** and use `sex` to make the composition of each smoking group visible.

Your figure should clearly show:

- `Never`
- `Former`
- `Current`
- `Unknown`

Add an informative title and clear axis labels.

From the figure, determine which smoking-status group contains the largest number of participants.

---

## Practice 2. Explore the Distribution of BMI

Create a figure showing the **distribution of BMI values** in the study population.

Then separate the visualization by `sex` so that the BMI distributions can be compared between the groups.

Choose an appropriate number or width of intervals so that the distribution is easy to interpret.

Add:

- an informative title
- a clear x-axis label
- a clear y-axis label
- a clean plotting theme

Consider whether the BMI distributions appear similar or different between the groups.

---

## Practice 3. Compare LDL Levels Between Smoking Groups

Create a figure that allows you to compare the **distribution of LDL levels** among the different `smoking_status` groups.

The figure should show both:

- the overall distribution within each smoking group
- the individual participants

Use `sex` as an additional visual characteristic if appropriate.

Make sure that the smoking-status categories and LDL axis are clearly labelled.

Which smoking group appears to have the highest LDL levels in the **sample small** dataset? 

---

## Practice 4. Explore the Relationship Between Blood Sugar and LDL

Investigate whether participants with higher **blood sugar** also tend to have higher **LDL levels**.

Create a figure in which:

- blood sugar is shown on one axis
- LDL level is shown on the other axis
- smoking status is visually distinguishable

Make the individual observations easy to see and add appropriate axis labels and a title.

Based only on the visualization, describe in one or two sentences whether you notice a relationship between blood sugar and LDL level.

---

## Practice 5. Prepare a Figure for a Scientific Report

Choose **two of the figures** created above and improve them so that they would be suitable for presentation in a report or research meeting.

Make sure that both plots have:

- meaningful titles
- informative axis labels
- readable text
- an appropriate theme
- consistent visual styling

Combine the two plots into **one figure** and save the final figure in your project as:

```text
figures/session03_health_summary.png
```

Before finishing, check that the saved figure can be opened and that all labels remain readable.

---
