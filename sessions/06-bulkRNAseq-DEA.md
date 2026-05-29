# Bulk RNA-seq Differential Expression and Pathway Enrichment Analysis in R/RStudio

## Download the input count matrix

Before starting the analysis, download the featureCounts gene-level count matrix:

[Download featureCounts_counts_matrix.tsv](https://github.com/Samaneh-Tousi/R-Basics--Biostatics-and-Bioinformatics-Tutorial/assets/featureCounts_counts_matrix.tsv)

If the download button does not work, open the `assets` folder in this repository and click:

```text
featureCounts_counts_matrix.tsv
```

---

# Differential Expression Analysis and Enrichment Analysis

Once we have a clean gene-level count matrix, the next step is **differential expression analysis**, also called **DEA**.

DEA tests which genes show statistically significant changes in expression between conditions, for example:

```text
MS vs Control
```

DEA tools work on **raw counts**, model the variability between biological replicates, apply normalization, and perform statistical testing.

The most widely used tools in bulk RNA-seq are:

- DESeq2
- edgeR
- limma-voom

They all aim to answer the same question:

> Which genes are differentially expressed?

However, they differ in how they normalize counts, model variance, and handle different experimental designs.

A key point is that **biological replicates are essential** for reliable statistics.  
Technical replicates are usually merged at the count level instead of being treated as independent samples.

---

# Common DEA Tools and When to Use Them

| Tool | Input | Normalization method | Pros | Cons | Best for |
|---|---|---|---|---|---|
| **DESeq2** | Raw counts | Median-of-ratios size factors | Robust, intuitive, excellent for typical RNA-seq designs, built-in fold-change shrinkage, strong documentation | Slower on very large datasets | Small to moderate sample sizes, simple or moderately complex designs |
| **edgeR** | Raw counts | TMM normalization | Flexible, handles complex designs, GLMs, low replicate numbers, strong dispersion modeling | More technical syntax; sensitive to filtering choices | Few replicates or complex designs with batch, interaction, or paired samples |
| **limma-voom** | Raw counts converted to logCPM with precision weights | Library-size scaling plus voom mean-variance modeling | Very fast, powerful for larger sample sizes and multi-factor designs | Less ideal for very low replicate numbers | Larger studies and multi-factor designs |

---

## Question

Imagine you have **10 biological replicates per condition**, collected at **two different time points**, and there are confounding variables across replicates, such as donor effects or batch differences.

Which DEA tool would be most appropriate: **DESeq2**, **edgeR**, or **limma-voom**, and why?

A good answer would be:

> **limma-voom** or **DESeq2** could both be appropriate, but **limma-voom** is especially attractive for larger multi-factor designs because it is fast and works well with linear models, covariates, batches, donor effects, and time points. DESeq2 can also model covariates and is very robust, especially for moderate sample sizes. edgeR is also valid but is often especially useful when replicate numbers are low or when flexible GLM-based modeling is needed.

---

# Common RNA-seq Normalization Methods

RNA-seq samples usually have different sequencing depths.  
For example, one sample may have 20 million reads while another has 35 million reads.

Normalization adjusts for these differences so that expression values can be compared fairly across samples.

Different RNA-seq tools use different normalization strategies.

---

## 1. CPM Normalization

**CPM** means **counts per million**.

It adjusts each sample based on its total library size.

The formula is:

```text
CPM = raw count / total mapped reads in the sample × 1,000,000
```

Example:

```text
Gene count = 500
Total sample counts = 10,000,000

CPM = 500 / 10,000,000 × 1,000,000
CPM = 50
```

This means the gene has 50 counts per million reads.

CPM is useful for:

- basic expression comparison
- exploratory plots
- filtering low-expression genes
- visualization

However, CPM alone does not fully correct for **composition bias**.

Composition bias happens when a few highly expressed genes take up a large fraction of reads in one sample.  
This can make the remaining genes look artificially lower, even if they did not biologically change.

In edgeR, CPM values can be calculated using:

```r
cpm_values <- cpm(dge)
```

Log-transformed CPM is often used for visualization:

```r
log_cpm_values <- cpm(dge, log = TRUE)
```

---

## 2. TMM Normalization

**TMM** means **Trimmed Mean of M values**.

TMM is used by **edgeR**.

TMM corrects for:

- different sequencing depths
- RNA composition bias between samples

For example, imagine two samples have similar total read numbers, but in one sample a small number of genes are extremely highly expressed.  
Those genes consume a large proportion of the sequencing reads.  
As a result, other genes may appear lower in that sample even when they did not truly decrease.

TMM solves this by comparing each sample to a reference sample and estimating a normalization factor.

The method works by:

1. Comparing gene expression ratios between samples
2. Removing genes with extreme expression differences
3. Removing genes with very high or very low expression
4. Calculating a trimmed mean of the remaining log-ratios
5. Using this trimmed mean as a normalization factor

In edgeR:

```r
dge <- calcNormFactors(dge, method = "TMM")
```

After TMM normalization, edgeR stores normalization factors in:

```r
dge$samples
```

TMM is especially useful when:

- RNA composition differs between samples
- there are strong expression differences
- only a subset of genes is highly expressed
- edgeR or limma-voom workflows are used

---

## 3. Median-of-Ratios Size Factor Normalization

**Median-of-ratios normalization** is used by **DESeq2**.

DESeq2 does not simply divide by total library size.  
Instead, it estimates a **size factor** for each sample.

The goal is to correct for sequencing depth while reducing the influence of highly expressed genes.

---

### Step 1: Create a Reference for Each Gene

For each gene, DESeq2 calculates the **geometric mean** of its counts across all samples.

Example:

```text
Gene A counts = 10, 20, 5

Geometric mean = cube root of 10 × 20 × 5
Geometric mean ≈ 10
```

This gives a typical reference value for each gene.

---

### Step 2: Calculate Ratios

For each sample, DESeq2 divides each gene count by the gene’s geometric mean.

```text
ratio = gene count in sample / geometric mean of that gene
```

Example for one sample:

```text
Gene A = 10  → ratio = 10 / 10 = 1
Gene B = 100 → ratio = 100 / 50 = 2
Gene C = 5   → ratio = 5 / 10 = 0.5
```

---

### Step 3: Take the Median Ratio

DESeq2 takes the median of all ratios in each sample.

Example:

```text
ratios = 1, 2, 0.5
sorted ratios = 0.5, 1, 2
median = 1
```

This median becomes the **size factor**.

```text
size factor > 1 → sample has more reads than average
size factor < 1 → sample has fewer reads than average
```

---

### Step 4: Normalize the Counts

Finally:

```text
normalized count = raw count / size factor
```

Example:

```text
Raw count = 200
Size factor = 2

Normalized count = 200 / 2 = 100
```

This method assumes that most genes are not differentially expressed.  
Because it uses the median instead of the total count, it is less affected by a few extremely highly expressed genes.

In DESeq2, size factors can be viewed using:

```r
sizeFactors(dds)
```

Normalized counts can be extracted using:

```r
normalized_counts <- counts(dds, normalized = TRUE)
```

---

# Comparison of CPM, TMM, and Median-of-Ratios

| Method | Used by | Main idea | Best used for |
|---|---|---|---|
| **CPM** | edgeR, general RNA-seq workflows | Divides counts by total library size and scales to one million | Simple expression comparison, filtering, exploratory plots |
| **TMM** | edgeR, limma-voom | Corrects library size and composition bias using trimmed log-ratios | Differential expression workflows with edgeR or limma-voom |
| **Median-of-ratios size factors** | DESeq2 | Uses median gene-level ratios to estimate sample-specific size factors | Differential expression workflows with DESeq2 |

---

## Important Note

For differential expression testing:

- **DESeq2 expects raw counts**
- **edgeR expects raw counts**
- **limma-voom expects raw counts**

Do not input already normalized CPM values into DESeq2 or edgeR for DEA testing.

Normalization is performed internally by the analysis method.

---

# Mini Diagram of DESeq2 DEA

```text
Raw counts
    │
    ▼
Normalization using size factors
    │
    ▼
Dispersion estimation
    │
    ▼
Model fitting
    │
    ▼
Statistical test
    │
    ▼
Significant difference?
    ├── Yes → DEG
    └── No  → Not a DEG
```

---

# 1. Install Required Packages

You only need to install packages **once**.

After installation, you only need to load them using `library()`.

Repeatedly reinstalling packages is unnecessary and may cause problems.

```r
# ============================================================
# Install CRAN packages only if missing
# ============================================================

cran_packages <- c(
  "tidyverse",
  "readr",
  "dplyr",
  "ggplot2",
  "ggrepel",
  "pheatmap",
  "RColorBrewer",
  "tibble",
  "enrichR"
)

for (pkg in cran_packages) {
  if (!requireNamespace(pkg, quietly = TRUE)) {
    install.packages(pkg)
  }
}


# ============================================================
# Install Bioconductor packages only if missing
# ============================================================

if (!requireNamespace("BiocManager", quietly = TRUE)) {
  install.packages("BiocManager")
}

bioc_packages <- c(
  "DESeq2",
  "edgeR",
  "limma",
  "biomaRt",
  "AnnotationDbi",
  "org.Hs.eg.db",
  "clusterProfiler",
  "enrichplot",
  "msigdbr",
  "pathview",
  "KEGGREST"
)

for (pkg in bioc_packages) {
  if (!requireNamespace(pkg, quietly = TRUE)) {
    BiocManager::install(pkg, ask = FALSE, update = FALSE)
  }
}
```

---

# 2. Load Packages

```r
# ============================================================
# Load libraries
# ============================================================

library(tidyverse)
library(readr)
library(dplyr)
library(ggplot2)
library(ggrepel)
library(tibble)

library(DESeq2)
library(edgeR)
library(limma)
library(biomaRt)
library(AnnotationDbi)
library(org.Hs.eg.db)

library(clusterProfiler)
library(enrichplot)
library(msigdbr)
library(enrichR)
library(pathview)
library(KEGGREST)
library(pheatmap)
library(RColorBrewer)
```

---

# 3. Define File Paths

```r
# ============================================================
# Define local paths
# ============================================================

assets_dir <- "C:/Users/saman/OneDrive/Desktop/UHasselt/BIOMED/Bioinformatics Group/R workshop/R-Basics--Biostatics-and-Bioinformatics-Tutorial/assets"

counts_path <- file.path(assets_dir, "featureCounts_counts_matrix.tsv")

out_dir <- file.path(assets_dir, "bulkRNAseq_results")

dir.create(out_dir, showWarnings = FALSE, recursive = TRUE)

counts_path
out_dir
```

---

# 4. Import the Count Matrix

```r
# ============================================================
# Import featureCounts count matrix
# ============================================================

counts_raw <- read_tsv(counts_path)

head(counts_raw)
dim(counts_raw)
colnames(counts_raw)
```

---

# 5. Prepare Count Matrix

The input count matrix should have genes as rows and samples as columns.

The gene column may be named:

```text
gene
Gene
Geneid
gene_id
ensembl_gene_id
```

```r
# ============================================================
# Prepare count matrix
# ============================================================

possible_gene_cols <- c("gene", "Gene", "Geneid", "gene_id", "ensembl_gene_id")

gene_col <- possible_gene_cols[possible_gene_cols %in% colnames(counts_raw)][1]

if (is.na(gene_col)) {
  stop("Could not detect gene column. Please rename the gene column to gene or Geneid.")
}

gene_ids <- counts_raw[[gene_col]]

annotation_cols <- c(
  "gene", "Gene", "Geneid", "gene_id", "ensembl_gene_id",
  "Chr", "Start", "End", "Strand", "Length"
)

cts <- counts_raw %>%
  dplyr::select(-dplyr::any_of(annotation_cols)) %>%
  as.data.frame()

rownames(cts) <- gene_ids

cts[] <- lapply(cts, function(x) as.numeric(as.character(x)))

cts[is.na(cts)] <- 0

cts <- round(cts)

head(cts)
dim(cts)
colSums(cts)
```

---

# 6. Reorder Samples

Edit this section based on your dataset.

For the original MS vs Control example:

```text
SRR6849240, SRR6849241, SRR6849242 = MS
SRR6849255, SRR6849256, SRR6849257 = Control
```

```r
# ============================================================
# Optional: reorder samples
# ============================================================

expected_samples <- c(
  "SRR6849240", "SRR6849241", "SRR6849242",
  "SRR6849255", "SRR6849256", "SRR6849257"
)

if (all(expected_samples %in% colnames(cts))) {
  cts <- cts[, expected_samples]
}

colnames(cts)
```

---

# 7. Create Sample Metadata

```r
# ============================================================
# Create metadata
# ============================================================

coldata <- data.frame(
  sample = colnames(cts),
  condition = c(
    "MS", "MS", "MS",
    "Control", "Control", "Control"
  )
)

coldata$condition <- factor(coldata$condition, levels = c("Control", "MS"))

rownames(coldata) <- coldata$sample

coldata
```

Check that count matrix and metadata match:

```r
# ============================================================
# Check sample matching
# ============================================================

all(colnames(cts) == rownames(coldata))

if (!all(colnames(cts) == rownames(coldata))) {
  stop("Column names of count matrix do not match row names of metadata.")
}
```

---

# 8. Library Size Quality Control

```r
# ============================================================
# Library size QC
# ============================================================

library_sizes <- data.frame(
  sample = colnames(cts),
  library_size = colSums(cts),
  condition = coldata$condition
)

p_library <- ggplot(library_sizes, aes(x = sample, y = library_size, fill = condition)) +
  geom_col() +
  theme_minimal(base_size = 12) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
  labs(
    title = "Library size per sample",
    x = "Sample",
    y = "Total assigned counts"
  )

p_library

ggsave(
  file.path(assets_dir, "bulkRNAseq_library_sizes.png"),
  p_library,
  width = 7,
  height = 5,
  dpi = 300
)
```

<p align="center">
  <img src="assets/bulkRNAseq_library_sizes.png" alt="Library size plot" width="700">
</p>

---

# 9. DESeq2 Differential Expression Analysis

```r
# ============================================================
# Build DESeq2 object
# ============================================================

dds <- DESeqDataSetFromMatrix(
  countData = cts,
  colData = coldata,
  design = ~ condition
)

dds
```

---

# 10. Filter Low-Count Genes

```r
# ============================================================
# Filter low-count genes
# ============================================================

dds <- dds[rowSums(counts(dds)) >= 10, ]

dds
```

---

# 11. Run DESeq2

```r
# ============================================================
# Run DESeq2
# ============================================================

dds <- DESeq(dds)

dds
```

---

# 12. Inspect DESeq2 Size Factors

```r
# ============================================================
# Inspect DESeq2 size factors
# ============================================================

sizeFactors(dds)
```

The size factors show how much each sample is scaled during DESeq2 normalization.

---

# 13. Dispersion Plot

```r
# ============================================================
# Dispersion plot
# ============================================================

png(file.path(assets_dir, "Disp_plot.png"), width = 800, height = 600)

plotDispEsts(dds)

dev.off()
```

<p align="center">
  <img src="assets/Disp_plot.png" alt="DESeq2 dispersion plot" width="800">
</p>

The **dispersion plot** shows:

- **Black dots**: raw observed dispersion for each gene
- **Red curve**: fitted dispersion trend
- **Blue circles**: final shrunken dispersions used in testing

Low-count genes usually have more variability.  
High-count genes usually have more stable expression estimates.

DESeq2 shrinks noisy gene-wise dispersion estimates toward the fitted trend.  
This stabilizes variance estimates and prevents unstable genes from dominating the analysis.

---

# 14. MA Plot

```r
# ============================================================
# MA plot
# ============================================================

png(file.path(assets_dir, "MAplot.png"), width = 800, height = 600)

plotMA(dds, ylim = c(-10, 10), main = "DESeq2 MA plot: MS vs Control")

dev.off()
```

<p align="center">
  <img src="assets/MAplot.png" alt="DESeq2 MA plot" width="800">
</p>

The **MA plot** shows:

- **M**: log2 fold change
- **A**: average expression
- **Grey points**: all genes
- **Blue points**: significantly differentially expressed genes

Genes with positive log2FC are upregulated in MS.  
Genes with negative log2FC are downregulated in MS.

Low-count genes often show a wider spread because they are noisier.  
High-count genes usually have more stable fold-change estimates.

---

# 15. VST Transformation and PCA Plot

```r
# ============================================================
# VST transformation and PCA
# ============================================================

vsd <- vst(dds, blind = FALSE)

p_pca <- plotPCA(vsd, intgroup = "condition") +
  theme_minimal(base_size = 12) +
  labs(title = "PCA plot of VST-normalized expression")

p_pca

ggsave(
  file.path(assets_dir, "PCAplot.png"),
  p_pca,
  width = 7,
  height = 5,
  dpi = 300
)
```

<p align="center">
  <img src="assets/PCAplot.png" alt="PCA plot" width="600">
</p>

The PCA plot shows how samples group based on overall gene-expression patterns after normalization and variance-stabilizing transformation.

Possible interpretations:

- Controls may cluster closely if they are homogeneous
- MS samples may show more heterogeneity
- Strong heterogeneity can reduce statistical power
- More biological replicates may improve statistical power

---

# 16. Sample Distance Heatmap

```r
# ============================================================
# Sample distance heatmap
# ============================================================

vst_matrix <- assay(vsd)

sample_dist <- dist(t(vst_matrix))

sample_dist_matrix <- as.matrix(sample_dist)

rownames(sample_dist_matrix) <- colnames(vst_matrix)
colnames(sample_dist_matrix) <- colnames(vst_matrix)

annotation_col <- data.frame(
  condition = coldata$condition
)

rownames(annotation_col) <- rownames(coldata)

png(file.path(assets_dir, "bulkRNAseq_sample_distance_heatmap.png"),
    width = 800,
    height = 700)

pheatmap(
  sample_dist_matrix,
  annotation_col = annotation_col,
  annotation_row = annotation_col,
  main = "Sample distance heatmap"
)

dev.off()
```

<p align="center">
  <img src="assets/bulkRNAseq_sample_distance_heatmap.png" alt="Sample distance heatmap" width="700">
</p>

---

# 17. Extract DESeq2 Results

```r
# ============================================================
# Extract DESeq2 results
# ============================================================

res <- results(
  dds,
  contrast = c("condition", "MS", "Control"),
  cooksCutoff = FALSE,
  independentFiltering = FALSE
)

summary(res)

res_tbl <- as.data.frame(res)

res_tbl$gene_id <- rownames(res_tbl)

head(res_tbl)
```

---

# 18. Shrink Log2 Fold Changes

```r
# ============================================================
# Shrink log2 fold changes
# ============================================================

res_shrunk <- lfcShrink(
  dds,
  contrast = c("condition", "MS", "Control"),
  res = res,
  type = "normal"
)

res_shrunk_tbl <- as.data.frame(res_shrunk)

res_shrunk_tbl$gene_id <- rownames(res_shrunk_tbl)

head(res_shrunk_tbl)
```

---

# 19. Gene Annotation

DESeq2 results often contain Ensembl IDs such as:

```text
ENSG00000141510.12
```

These IDs are accurate but not easy to interpret.

We remove the version suffix and map Ensembl IDs to:

- gene symbols
- Entrez IDs

```r
# ============================================================
# Annotate genes with org.Hs.eg.db
# ============================================================

res_shrunk_tbl$ensembl_gene_id <- gsub("\\..*", "", res_shrunk_tbl$gene_id)

res_shrunk_tbl$external_gene_name <- mapIds(
  org.Hs.eg.db,
  keys = res_shrunk_tbl$ensembl_gene_id,
  keytype = "ENSEMBL",
  column = "SYMBOL",
  multiVals = "first"
)

res_shrunk_tbl$ENTREZID <- mapIds(
  org.Hs.eg.db,
  keys = res_shrunk_tbl$ensembl_gene_id,
  keytype = "ENSEMBL",
  column = "ENTREZID",
  multiVals = "first"
)

res_annot <- res_shrunk_tbl %>%
  arrange(padj)

head(res_annot)

write_tsv(
  res_annot,
  file.path(out_dir, "deseq2_results_annotated.tsv")
)
```

---

# 20. Identify Differentially Expressed Genes

A differentially expressed gene should show:

1. A large enough expression change
2. Consistent differences across biological replicates
3. Statistical significance after multiple-testing correction

In this tutorial, we use:

```text
padj < 0.05
absolute log2FC > 0.5
```

```r
# ============================================================
# Define DEG thresholds
# ============================================================

lfc_thr <- 0.5
padj_thr <- 0.05

DEGs <- res_annot %>%
  filter(
    !is.na(padj),
    padj < padj_thr,
    abs(log2FoldChange) > lfc_thr
  ) %>%
  arrange(padj)

nrow(DEGs)

write_tsv(
  DEGs,
  file.path(
    out_dir,
    sprintf("DEGs_MS_vs_Control_padj%.2f_LFC%.2f.tsv", padj_thr, lfc_thr)
  )
)
```

---

# 21. p-value, Adjusted p-value, and FDR

In DEA, the **p-value** tells us how likely it is that an observed expression difference happened by chance, assuming no true difference exists.

However, RNA-seq tests thousands of genes at the same time.  
Some genes will appear significant by chance.  
This is called the **multiple testing problem**.

To correct this, we use **False Discovery Rate**, or **FDR**.

The **adjusted p-value**, also called **padj**, is the p-value after FDR correction, commonly using the Benjamini-Hochberg method.

In practice, we usually use:

```text
padj < 0.05
```

This means that among the genes called significant, about 5% are expected to be false discoveries.

---

## Question

A gene has:

```text
p-value = 0.002
padj = 0.10
```

What does this tell you?

A good answer would be:

> The gene looked significant before correction, but after correcting for thousands of tests, it is not significant at padj < 0.05. The difference occurs because the raw p-value does not account for multiple testing, while padj does.

---

# 22. Volcano Plot

```r
# ============================================================
# Volcano plot
# ============================================================

df <- res_annot

if (!"external_gene_name" %in% names(df)) {
  df$external_gene_name <- NA_character_
}

df$status <- "NotSig"

df$status[
  !is.na(df$padj) &
    df$padj < padj_thr &
    df$log2FoldChange > lfc_thr
] <- "Up"

df$status[
  !is.na(df$padj) &
    df$padj < padj_thr &
    df$log2FoldChange < -lfc_thr
] <- "Down"

df$mlog10padj <- -log10(df$padj)

df$mlog10padj[!is.finite(df$mlog10padj)] <- NA

sig <- df %>%
  filter(
    !is.na(padj),
    padj < padj_thr,
    abs(log2FoldChange) > lfc_thr
  )

top10 <- sig %>%
  arrange(padj, desc(abs(log2FoldChange))) %>%
  slice_head(n = 10)

p_volcano <- ggplot(df, aes(x = log2FoldChange, y = mlog10padj, color = status)) +
  geom_point(size = 1.3, alpha = 0.8, na.rm = TRUE) +
  geom_vline(xintercept = c(-lfc_thr, lfc_thr), linetype = "dashed") +
  geom_hline(yintercept = -log10(padj_thr), linetype = "dashed") +
  ggrepel::geom_text_repel(
    data = top10,
    aes(label = external_gene_name),
    size = 3,
    max.overlaps = 20
  ) +
  scale_color_manual(
    values = c(
      NotSig = "grey70",
      Up = "red",
      Down = "blue"
    )
  ) +
  labs(
    x = "log2 fold change",
    y = expression(-log[10]("adjusted p-value")),
    color = "Status",
    title = "MS vs Control — Volcano Plot"
  ) +
  theme_minimal(base_size = 12)

p_volcano

ggsave(
  file.path(assets_dir, "DEGs_Volcano.png"),
  p_volcano,
  width = 7,
  height = 5,
  dpi = 300
)
```

<p align="center">
  <img src="assets/DEGs_Volcano.png" alt="DEG volcano plot" width="600">
</p>

---

# 23. What log2 Fold Change Means

| log2FC | Meaning | Fold change |
|---|---|---|
| +1 | Upregulated | 2 times higher |
| +2 | Strongly upregulated | 4 times higher |
| -1 | Downregulated | 2 times lower |
| -2 | Strongly downregulated | 4 times lower |
| 0 | No change | Same expression |

A **DEG** is a gene that shows:

1. A large enough change in expression
2. Consistent differences across replicates
3. A statistically significant adjusted p-value

| Gene | log2FC | padj | Interpretation |
|---|---:|---:|---|
| GeneA | +1.5 | 0.001 | Strongly upregulated in MS |
| GeneB | -2.0 | 0.03 | Downregulated in MS |
| GeneC | 0.2 | 0.8 | No meaningful change |

---

# 24. Heatmap of Most Variable Genes

```r
# ============================================================
# Heatmap of top 50 most variable genes
# ============================================================

gene_variances <- apply(vst_matrix, 1, var)

top_variable_genes <- names(sort(gene_variances, decreasing = TRUE))[1:50]

top_variable_matrix <- vst_matrix[top_variable_genes, ]

top_variable_matrix_scaled <- t(scale(t(top_variable_matrix)))

png(file.path(assets_dir, "bulkRNAseq_top50_variable_genes_heatmap.png"),
    width = 900,
    height = 1000)

pheatmap(
  top_variable_matrix_scaled,
  annotation_col = annotation_col,
  show_rownames = FALSE,
  main = "Top 50 most variable genes"
)

dev.off()
```

<p align="center">
  <img src="assets/bulkRNAseq_top50_variable_genes_heatmap.png" alt="Top variable genes heatmap" width="700">
</p>

---

# 25. Heatmap of Top DEGs

```r
# ============================================================
# Heatmap of top 50 DEGs
# ============================================================

top_deg_ids <- DEGs %>%
  filter(gene_id %in% rownames(vst_matrix)) %>%
  arrange(padj) %>%
  slice_head(n = 50) %>%
  pull(gene_id)

top_deg_matrix <- vst_matrix[top_deg_ids, ]

top_deg_matrix_scaled <- t(scale(t(top_deg_matrix)))

deg_labels <- DEGs %>%
  filter(gene_id %in% top_deg_ids) %>%
  select(gene_id, external_gene_name)

rownames(top_deg_matrix_scaled) <- ifelse(
  is.na(deg_labels$external_gene_name) | deg_labels$external_gene_name == "",
  deg_labels$gene_id,
  deg_labels$external_gene_name
)

png(file.path(assets_dir, "bulkRNAseq_top50_DEGs_heatmap.png"),
    width = 900,
    height = 1000)

pheatmap(
  top_deg_matrix_scaled,
  annotation_col = annotation_col,
  show_rownames = TRUE,
  fontsize_row = 7,
  main = "Top 50 DEGs"
)

dev.off()
```

<p align="center">
  <img src="assets/bulkRNAseq_top50_DEGs_heatmap.png" alt="Top DEG heatmap" width="700">
</p>

---

# 26. edgeR Differential Expression Analysis

edgeR is another popular method for RNA-seq DEA.

It uses:

- TMM normalization
- negative binomial models
- dispersion estimation
- generalized linear models

```r
# ============================================================
# Create edgeR object
# ============================================================

group <- coldata$condition

dge <- DGEList(
  counts = cts,
  group = group
)

dge
```

---

# 27. Filter Low-Expression Genes in edgeR

```r
# ============================================================
# Filter genes using edgeR
# ============================================================

keep_edger <- filterByExpr(dge, group = group)

dge <- dge[keep_edger, , keep.lib.sizes = FALSE]

dge
```

---

# 28. TMM Normalization

```r
# ============================================================
# TMM normalization
# ============================================================

dge <- calcNormFactors(dge, method = "TMM")

dge$samples
```

---

# 29. CPM Values for Visualization

```r
# ============================================================
# CPM and logCPM values
# ============================================================

cpm_values <- cpm(dge)

log_cpm_values <- cpm(dge, log = TRUE)

head(cpm_values[, 1:3])
head(log_cpm_values[, 1:3])
```

---

# 30. edgeR MDS Plot

```r
# ============================================================
# edgeR MDS plot
# ============================================================

png(file.path(assets_dir, "bulkRNAseq_edgeR_MDS_plot.png"),
    width = 800,
    height = 600)

plotMDS(
  dge,
  labels = rownames(coldata),
  col = as.numeric(group),
  main = "edgeR MDS plot"
)

legend(
  "topright",
  legend = levels(group),
  col = seq_along(levels(group)),
  pch = 16
)

dev.off()
```

<p align="center">
  <img src="assets/bulkRNAseq_edgeR_MDS_plot.png" alt="edgeR MDS plot" width="700">
</p>

---

# 31. edgeR Design Matrix

```r
# ============================================================
# edgeR design matrix
# ============================================================

design <- model.matrix(~ group)

design
```

---

# 32. Estimate Dispersion with edgeR

```r
# ============================================================
# edgeR dispersion estimation
# ============================================================

dge <- estimateDisp(dge, design)

png(file.path(assets_dir, "bulkRNAseq_edgeR_BCV_plot.png"),
    width = 800,
    height = 600)

plotBCV(dge)

dev.off()
```

<p align="center">
  <img src="assets/bulkRNAseq_edgeR_BCV_plot.png" alt="edgeR BCV plot" width="700">
</p>

---

# 33. edgeR GLM Testing

```r
# ============================================================
# edgeR GLM analysis
# ============================================================

fit <- glmFit(dge, design)

lrt <- glmLRT(fit, coef = 2)

edger_results <- topTags(lrt, n = Inf)$table

edger_results$gene_id <- rownames(edger_results)

head(edger_results)
```

---

# 34. Annotate edgeR Results

```r
# ============================================================
# Annotate edgeR results
# ============================================================

edger_results$ensembl_gene_id <- gsub("\\..*", "", edger_results$gene_id)

edger_results$external_gene_name <- mapIds(
  org.Hs.eg.db,
  keys = edger_results$ensembl_gene_id,
  keytype = "ENSEMBL",
  column = "SYMBOL",
  multiVals = "first"
)

edger_results$ENTREZID <- mapIds(
  org.Hs.eg.db,
  keys = edger_results$ensembl_gene_id,
  keytype = "ENSEMBL",
  column = "ENTREZID",
  multiVals = "first"
)

edger_results <- edger_results %>%
  arrange(FDR)

write_tsv(
  edger_results,
  file.path(out_dir, "edgeR_MS_vs_Control_all_results.tsv")
)
```

---

# 35. Select edgeR DEGs

```r
# ============================================================
# Select edgeR DEGs
# ============================================================

edger_degs <- edger_results %>%
  filter(
    !is.na(FDR),
    FDR < padj_thr,
    abs(logFC) > lfc_thr
  ) %>%
  arrange(FDR)

nrow(edger_degs)

write_tsv(
  edger_degs,
  file.path(out_dir, "edgeR_MS_vs_Control_DEGs.tsv")
)
```

---

# 36. edgeR Volcano Plot

```r
# ============================================================
# edgeR volcano plot
# ============================================================

edger_volcano <- edger_results

edger_volcano$status <- "NotSig"

edger_volcano$status[
  !is.na(edger_volcano$FDR) &
    edger_volcano$FDR < padj_thr &
    edger_volcano$logFC > lfc_thr
] <- "Up"

edger_volcano$status[
  !is.na(edger_volcano$FDR) &
    edger_volcano$FDR < padj_thr &
    edger_volcano$logFC < -lfc_thr
] <- "Down"

edger_volcano$mlog10FDR <- -log10(edger_volcano$FDR)

edger_volcano$mlog10FDR[!is.finite(edger_volcano$mlog10FDR)] <- NA

top_edger <- edger_volcano %>%
  filter(
    !is.na(FDR),
    FDR < padj_thr,
    abs(logFC) > lfc_thr
  ) %>%
  arrange(FDR) %>%
  slice_head(n = 10)

p_edger_volcano <- ggplot(
  edger_volcano,
  aes(x = logFC, y = mlog10FDR, color = status)
) +
  geom_point(size = 1.3, alpha = 0.8, na.rm = TRUE) +
  geom_vline(xintercept = c(-lfc_thr, lfc_thr), linetype = "dashed") +
  geom_hline(yintercept = -log10(padj_thr), linetype = "dashed") +
  geom_text_repel(
    data = top_edger,
    aes(label = external_gene_name),
    size = 3,
    max.overlaps = 20
  ) +
  scale_color_manual(
    values = c(
      NotSig = "grey70",
      Up = "red",
      Down = "blue"
    )
  ) +
  theme_minimal(base_size = 12) +
  labs(
    title = "edgeR volcano plot: MS vs Control",
    x = "log2 fold change",
    y = "-log10 FDR",
    color = "Status"
  )

p_edger_volcano

ggsave(
  file.path(assets_dir, "bulkRNAseq_edgeR_volcano.png"),
  p_edger_volcano,
  width = 7,
  height = 5,
  dpi = 300
)
```

<p align="center">
  <img src="assets/bulkRNAseq_edgeR_volcano.png" alt="edgeR volcano plot" width="700">
</p>

---

# 37. Compare DESeq2 and edgeR

```r
# ============================================================
# Compare DESeq2 and edgeR DEGs
# ============================================================

deseq2_deg_genes <- DEGs$gene_id
edger_deg_genes <- edger_degs$gene_id

common_degs <- intersect(deseq2_deg_genes, edger_deg_genes)

comparison_summary <- data.frame(
  method = c("DESeq2", "edgeR", "Common"),
  number_of_DEGs = c(
    length(deseq2_deg_genes),
    length(edger_deg_genes),
    length(common_degs)
  )
)

comparison_summary

write_tsv(
  comparison_summary,
  file.path(out_dir, "DESeq2_edgeR_DEG_comparison_summary.tsv")
)

write_tsv(
  data.frame(gene_id = common_degs),
  file.path(out_dir, "common_DEGs_DESeq2_edgeR.tsv")
)
```

```r
# ============================================================
# Plot DEG count comparison
# ============================================================

p_deg_count <- ggplot(comparison_summary, aes(x = method, y = number_of_DEGs, fill = method)) +
  geom_col() +
  theme_minimal(base_size = 12) +
  labs(
    title = "Number of DEGs detected by DESeq2 and edgeR",
    x = "Method",
    y = "Number of DEGs"
  ) +
  theme(legend.position = "none")

p_deg_count

ggsave(
  file.path(assets_dir, "bulkRNAseq_DESeq2_edgeR_DEG_count_comparison.png"),
  p_deg_count,
  width = 6,
  height = 5,
  dpi = 300
)
```

<p align="center">
  <img src="assets/bulkRNAseq_DESeq2_edgeR_DEG_count_comparison.png" alt="DEG count comparison" width="650">
</p>

---

# 38. Compare Log2 Fold Changes

```r
# ============================================================
# Compare DESeq2 and edgeR log2 fold changes
# ============================================================

lfc_comparison <- res_annot %>%
  select(
    gene_id,
    deseq2_log2FC = log2FoldChange,
    deseq2_padj = padj,
    external_gene_name
  ) %>%
  inner_join(
    edger_results %>%
      select(
        gene_id,
        edger_log2FC = logFC,
        edger_FDR = FDR
      ),
    by = "gene_id"
  )

p_lfc_compare <- ggplot(
  lfc_comparison,
  aes(x = deseq2_log2FC, y = edger_log2FC)
) +
  geom_point(alpha = 0.5) +
  geom_smooth(method = "lm", se = FALSE) +
  theme_minimal(base_size = 12) +
  labs(
    title = "Comparison of log2 fold changes: DESeq2 vs edgeR",
    x = "DESeq2 log2FC",
    y = "edgeR log2FC"
  )

p_lfc_compare

ggsave(
  file.path(assets_dir, "bulkRNAseq_DESeq2_edgeR_logFC_comparison.png"),
  p_lfc_compare,
  width = 6,
  height = 5,
  dpi = 300
)
```

<p align="center">
  <img src="assets/bulkRNAseq_DESeq2_edgeR_logFC_comparison.png" alt="DESeq2 edgeR logFC comparison" width="650">
</p>

---

# 39. Prepare Gene Lists for Enrichment Analysis

```r
# ============================================================
# Prepare gene lists
# ============================================================

deg_entrez <- DEGs %>%
  filter(!is.na(ENTREZID)) %>%
  pull(ENTREZID) %>%
  unique()

deg_symbols <- DEGs %>%
  filter(!is.na(external_gene_name), external_gene_name != "") %>%
  pull(external_gene_name) %>%
  unique()

background_entrez <- res_annot %>%
  filter(!is.na(ENTREZID)) %>%
  pull(ENTREZID) %>%
  unique()

length(deg_entrez)
length(deg_symbols)
length(background_entrez)
```

---

# 40. GO Enrichment with clusterProfiler

```r
# ============================================================
# GO Biological Process enrichment
# ============================================================

ego_bp <- enrichGO(
  gene = deg_entrez,
  universe = background_entrez,
  OrgDb = org.Hs.eg.db,
  keyType = "ENTREZID",
  ont = "BP",
  pAdjustMethod = "BH",
  pvalueCutoff = 0.05,
  qvalueCutoff = 0.2,
  readable = TRUE
)

ego_bp_df <- as.data.frame(ego_bp)

write_tsv(
  ego_bp_df,
  file.path(out_dir, "clusterProfiler_GO_BP_enrichment.tsv")
)

head(ego_bp_df)
```

```r
# ============================================================
# GO dotplot
# ============================================================

p_go_dot <- dotplot(ego_bp, showCategory = 15) +
  ggtitle("GO Biological Process enrichment")

p_go_dot

ggsave(
  file.path(assets_dir, "bulkRNAseq_clusterProfiler_GO_BP_dotplot.png"),
  p_go_dot,
  width = 8,
  height = 6,
  dpi = 300
)
```

<p align="center">
  <img src="assets/bulkRNAseq_clusterProfiler_GO_BP_dotplot.png" alt="GO enrichment dotplot" width="750">
</p>

---

# 41. KEGG Enrichment with clusterProfiler

```r
# ============================================================
# KEGG pathway enrichment
# ============================================================

ekegg <- enrichKEGG(
  gene = deg_entrez,
  universe = background_entrez,
  organism = "hsa",
  pAdjustMethod = "BH",
  pvalueCutoff = 0.05,
  qvalueCutoff = 0.2
)

ekegg_df <- as.data.frame(ekegg)

write_tsv(
  ekegg_df,
  file.path(out_dir, "clusterProfiler_KEGG_enrichment.tsv")
)

head(ekegg_df)
```

```r
# ============================================================
# KEGG dotplot
# ============================================================

p_kegg_dot <- dotplot(ekegg, showCategory = 15) +
  ggtitle("KEGG pathway enrichment")

p_kegg_dot

ggsave(
  file.path(assets_dir, "bulkRNAseq_clusterProfiler_KEGG_dotplot.png"),
  p_kegg_dot,
  width = 8,
  height = 6,
  dpi = 300
)
```

<p align="center">
  <img src="assets/bulkRNAseq_clusterProfiler_KEGG_dotplot.png" alt="KEGG enrichment dotplot" width="750">
</p>

---

# 42. Enrichr Pathway Enrichment

Enrichr is useful for quick enrichment analysis using many gene-set libraries.

It works well with gene symbols.

```r
# ============================================================
# Enrichr analysis
# ============================================================

available_dbs <- enrichR::listEnrichrDbs()

head(available_dbs)

enrichr_databases <- c(
  "GO_Biological_Process_2023",
  "KEGG_2021_Human",
  "Reactome_2022",
  "WikiPathway_2023_Human"
)

enrichr_results <- enrichR::enrichr(
  genes = deg_symbols,
  databases = enrichr_databases
)

names(enrichr_results)
```

```r
# ============================================================
# Save Enrichr results
# ============================================================

for (db in names(enrichr_results)) {
  output_file <- paste0(
    "Enrichr_",
    gsub("[^A-Za-z0-9]", "_", db),
    ".tsv"
  )
  
  write_tsv(
    enrichr_results[[db]],
    file.path(out_dir, output_file)
  )
}
```

```r
# ============================================================
# Plot top Enrichr KEGG pathways
# ============================================================

enrichr_kegg <- enrichr_results[["KEGG_2021_Human"]]

top_enrichr_kegg <- enrichr_kegg %>%
  arrange(Adjusted.P.value) %>%
  slice_head(n = 15) %>%
  mutate(Term = factor(Term, levels = rev(Term)))

p_enrichr_kegg <- ggplot(
  top_enrichr_kegg,
  aes(x = Term, y = -log10(Adjusted.P.value))
) +
  geom_col() +
  coord_flip() +
  theme_minimal(base_size = 12) +
  labs(
    title = "Top Enrichr KEGG pathways",
    x = "Pathway",
    y = "-log10 adjusted p-value"
  )

p_enrichr_kegg

ggsave(
  file.path(assets_dir, "bulkRNAseq_Enrichr_KEGG_barplot.png"),
  p_enrichr_kegg,
  width = 8,
  height = 6,
  dpi = 300
)
```

<p align="center">
  <img src="assets/bulkRNAseq_Enrichr_KEGG_barplot.png" alt="Enrichr KEGG barplot" width="750">
</p>

---

# 43. Gene Set Enrichment Analysis

After identifying differentially expressed genes, we want to understand which biological pathways are systematically upregulated or downregulated.

**Gene Set Enrichment Analysis**, or **GSEA**, evaluates genome-wide ranked statistics to determine whether predefined gene sets are enriched at the top or bottom of a ranked gene list.

Unlike over-representation analysis, GSEA does not require a DEG cutoff.  
It uses all genes, making it sensitive and biologically interpretable.

---

# 44. Prepare Ranked Gene List for GSEA

```r
# ============================================================
# Prepare ranked gene list
# ============================================================

score <- if (!all(is.na(res_annot$stat))) {
  res_annot$stat
} else {
  res_annot$log2FoldChange
}

rank_df <- tibble(
  ENTREZID = res_annot$ENTREZID,
  score = score
) %>%
  filter(
    !is.na(ENTREZID),
    is.finite(score)
  ) %>%
  group_by(ENTREZID) %>%
  summarise(
    score = score[which.max(abs(score))],
    .groups = "drop"
  )

ranks <- sort(
  setNames(rank_df$score, rank_df$ENTREZID),
  decreasing = TRUE
)

head(ranks)
tail(ranks)
```

---

# 45. GSEA with Hallmark Pathways

```r
# ============================================================
# GSEA using Hallmark gene sets
# ============================================================

m_h <- msigdbr(
  species = "Homo sapiens",
  category = "H"
) %>%
  select(gs_name, entrez_gene)

set.seed(42)

gseaH <- GSEA(
  geneList = ranks,
  TERM2GENE = m_h,
  pAdjustMethod = "BH",
  minGSSize = 10,
  maxGSSize = 500,
  verbose = FALSE
)

gseaH_df <- as.data.frame(gseaH)

write_tsv(
  gseaH_df,
  file.path(out_dir, "GSEA_Hallmark_results.tsv")
)

head(gseaH_df)
```

```r
# ============================================================
# GSEA top pathway plot
# ============================================================

png(file.path(assets_dir, "GSEA.png"), width = 900, height = 700)

print(
  enrichplot::gseaplot2(
    gseaH,
    geneSetID = 1:4,
    title = "Top Hallmark pathways"
  )
)

dev.off()
```

<p align="center">
  <img src="assets/GSEA.png" alt="GSEA hallmark pathway plot" width="700">
</p>

---

# 46. GSEA Dotplot

```r
# ============================================================
# GSEA Hallmark dotplot
# ============================================================

p_gsea_dot <- dotplot(gseaH, showCategory = 15) +
  ggtitle("Hallmark GSEA")

p_gsea_dot

ggsave(
  file.path(assets_dir, "bulkRNAseq_GSEA_Hallmark_dotplot.png"),
  p_gsea_dot,
  width = 8,
  height = 6,
  dpi = 300
)
```

<p align="center">
  <img src="assets/bulkRNAseq_GSEA_Hallmark_dotplot.png" alt="Hallmark GSEA dotplot" width="750">
</p>

---

# 47. KEGG GSEA

```r
# ============================================================
# KEGG GSEA
# ============================================================

set.seed(42)

gsea_kegg <- gseKEGG(
  geneList = ranks,
  organism = "hsa",
  minGSSize = 10,
  maxGSSize = 500,
  pvalueCutoff = 0.05,
  pAdjustMethod = "BH",
  verbose = FALSE
)

gsea_kegg_df <- as.data.frame(gsea_kegg)

write_tsv(
  gsea_kegg_df,
  file.path(out_dir, "GSEA_KEGG_results.tsv")
)

head(gsea_kegg_df)
```

```r
# ============================================================
# KEGG GSEA dotplot
# ============================================================

p_gsea_kegg_dot <- dotplot(gsea_kegg, showCategory = 15) +
  ggtitle("KEGG GSEA")

p_gsea_kegg_dot

ggsave(
  file.path(assets_dir, "bulkRNAseq_GSEA_KEGG_dotplot.png"),
  p_gsea_kegg_dot,
  width = 8,
  height = 6,
  dpi = 300
)
```

<p align="center">
  <img src="assets/bulkRNAseq_GSEA_KEGG_dotplot.png" alt="KEGG GSEA dotplot" width="750">
</p>

---

# 48. Enrichment Map

```r
# ============================================================
# Enrichment map
# ============================================================

if (nrow(as.data.frame(gseaH)) > 1) {
  
  gseaH_sim <- pairwise_termsim(gseaH)
  
  p_emap <- emapplot(gseaH_sim, showCategory = 20) +
    ggtitle("Enrichment map: Hallmark GSEA")
  
  p_emap
  
  ggsave(
    file.path(assets_dir, "bulkRNAseq_GSEA_Hallmark_enrichment_map.png"),
    p_emap,
    width = 9,
    height = 7,
    dpi = 300
  )
}
```

<p align="center">
  <img src="assets/bulkRNAseq_GSEA_Hallmark_enrichment_map.png" alt="GSEA enrichment map" width="750">
</p>

---

# 49. KEGG Signaling Pathway Visualization with pathview

The `pathview` package maps gene expression changes directly onto KEGG pathway diagrams.

This is useful for visualizing signaling pathways such as:

- NF-kappa B signaling pathway
- TNF signaling pathway
- Toll-like receptor signaling pathway
- JAK-STAT signaling pathway
- PI3K-Akt signaling pathway
- MAPK signaling pathway

---

## 49.1 Prepare log2FC Vector

```r
# ============================================================
# Prepare log2FC vector for pathview
# ============================================================

pathview_df <- res_annot %>%
  filter(!is.na(ENTREZID), !is.na(log2FoldChange)) %>%
  group_by(ENTREZID) %>%
  summarise(
    log2FC = log2FoldChange[which.max(abs(log2FoldChange))],
    .groups = "drop"
  )

gene_log2fc <- pathview_df$log2FC

names(gene_log2fc) <- pathview_df$ENTREZID

head(gene_log2fc)
```

---

## 49.2 Visualize Selected KEGG Signaling Pathways

| Pathway | KEGG ID |
|---|---|
| NF-kappa B signaling pathway | hsa04064 |
| TNF signaling pathway | hsa04668 |
| Toll-like receptor signaling pathway | hsa04620 |
| JAK-STAT signaling pathway | hsa04630 |
| PI3K-Akt signaling pathway | hsa04151 |
| MAPK signaling pathway | hsa04010 |

```r
# ============================================================
# Run pathview for selected KEGG pathways
# ============================================================

kegg_pathways <- c(
  "hsa04064",
  "hsa04668",
  "hsa04620",
  "hsa04630",
  "hsa04151",
  "hsa04010"
)

old_wd <- getwd()

setwd(assets_dir)

for (pid in kegg_pathways) {
  pathview(
    gene.data = gene_log2fc,
    pathway.id = pid,
    species = "hsa",
    gene.idtype = "entrez",
    out.suffix = "MS_vs_Control_DESeq2",
    kegg.native = TRUE,
    same.layer = FALSE
  )
}

setwd(old_wd)
```

Expected output examples:

```text
assets/hsa04064.MS_vs_Control_DESeq2.png
assets/hsa04668.MS_vs_Control_DESeq2.png
assets/hsa04620.MS_vs_Control_DESeq2.png
assets/hsa04630.MS_vs_Control_DESeq2.png
assets/hsa04151.MS_vs_Control_DESeq2.png
assets/hsa04010.MS_vs_Control_DESeq2.png
```

<p align="center">
  <img src="assets/hsa04064.MS_vs_Control_DESeq2.png" alt="NF-kappa B KEGG pathway" width="850">
</p>

<p align="center">
  <em>NF-kappa B signaling pathway with DESeq2 log2 fold changes mapped onto KEGG.</em>
</p>

<p align="center">
  <img src="assets/hsa04668.MS_vs_Control_DESeq2.png" alt="TNF KEGG pathway" width="850">
</p>

<p align="center">
  <em>TNF signaling pathway with DESeq2 log2 fold changes mapped onto KEGG.</em>
</p>

<p align="center">
  <img src="assets/hsa04620.MS_vs_Control_DESeq2.png" alt="Toll-like receptor KEGG pathway" width="850">
</p>

<p align="center">
  <em>Toll-like receptor signaling pathway with DESeq2 log2 fold changes mapped onto KEGG.</em>
</p>

<p align="center">
  <img src="assets/hsa04630.MS_vs_Control_DESeq2.png" alt="JAK-STAT KEGG pathway" width="850">
</p>

<p align="center">
  <em>JAK-STAT signaling pathway with DESeq2 log2 fold changes mapped onto KEGG.</em>
</p>

<p align="center">
  <img src="assets/hsa04151.MS_vs_Control_DESeq2.png" alt="PI3K-Akt KEGG pathway" width="850">
</p>

<p align="center">
  <em>PI3K-Akt signaling pathway with DESeq2 log2 fold changes mapped onto KEGG.</em>
</p>

<p align="center">
  <img src="assets/hsa04010.MS_vs_Control_DESeq2.png" alt="MAPK KEGG pathway" width="850">
</p>

<p align="center">
  <em>MAPK signaling pathway with DESeq2 log2 fold changes mapped onto KEGG.</em>
</p>

---

# 50. Automatically Visualize the Top Enriched KEGG Pathway

```r
# ============================================================
# Visualize top KEGG enrichment result
# ============================================================

if (exists("ekegg") && nrow(as.data.frame(ekegg)) > 0) {
  
  top_kegg_id <- as.data.frame(ekegg)$ID[1]
  
  old_wd <- getwd()
  setwd(assets_dir)
  
  pathview(
    gene.data = gene_log2fc,
    pathway.id = top_kegg_id,
    species = "hsa",
    gene.idtype = "entrez",
    out.suffix = "top_enriched_DESeq2",
    kegg.native = TRUE,
    same.layer = FALSE
  )
  
  setwd(old_wd)
  
  top_kegg_id
}
```

---

# 51. Save Normalized Counts

```r
# ============================================================
# Save normalized counts
# ============================================================

normalized_counts <- counts(dds, normalized = TRUE)

normalized_counts_df <- as.data.frame(normalized_counts)

normalized_counts_df$gene_id <- rownames(normalized_counts_df)

normalized_counts_df <- normalized_counts_df %>%
  relocate(gene_id)

write_tsv(
  normalized_counts_df,
  file.path(out_dir, "DESeq2_normalized_counts.tsv")
)
```

---

# 52. Save VST Counts

```r
# ============================================================
# Save VST counts
# ============================================================

vst_counts_df <- as.data.frame(vst_matrix)

vst_counts_df$gene_id <- rownames(vst_counts_df)

vst_counts_df <- vst_counts_df %>%
  relocate(gene_id)

write_tsv(
  vst_counts_df,
  file.path(out_dir, "DESeq2_VST_counts.tsv")
)
```

---

# 53. Save Session Information

```r
# ============================================================
# Save session information
# ============================================================

sink(file.path(out_dir, "sessionInfo.txt"))

sessionInfo()

sink()
```

---

# 54. Output Files

| Output | Description |
|---|---|
| `deseq2_results_annotated.tsv` | Full annotated DESeq2 results |
| `DEGs_MS_vs_Control_padj0.05_LFC0.50.tsv` | Significant DESeq2 DEGs |
| `edgeR_MS_vs_Control_all_results.tsv` | Full edgeR results |
| `edgeR_MS_vs_Control_DEGs.tsv` | Significant edgeR DEGs |
| `common_DEGs_DESeq2_edgeR.tsv` | Genes significant in both methods |
| `clusterProfiler_GO_BP_enrichment.tsv` | GO enrichment results |
| `clusterProfiler_KEGG_enrichment.tsv` | KEGG enrichment results |
| `GSEA_Hallmark_results.tsv` | Hallmark GSEA results |
| `GSEA_KEGG_results.tsv` | KEGG GSEA results |
| `Enrichr_*.tsv` | Enrichr pathway enrichment results |
| `DESeq2_normalized_counts.tsv` | DESeq2 normalized counts |
| `DESeq2_VST_counts.tsv` | VST-transformed expression matrix |
| `sessionInfo.txt` | Reproducibility information |

---

# 55. Summary

In this tutorial, you performed a complete bulk RNA-seq workflow using local R/RStudio:

1. Downloaded and imported a featureCounts matrix
2. Prepared metadata
3. Explained CPM, TMM, and median-of-ratios normalization
4. Checked library sizes
5. Ran DESeq2
6. Visualized dispersion, MA plot, PCA plot, and volcano plot
7. Identified DEGs
8. Created DEG and variable-gene heatmaps
9. Ran edgeR
10. Compared edgeR and DESeq2
11. Performed GO and KEGG enrichment with clusterProfiler
12. Performed Enrichr analysis
13. Performed GSEA
14. Visualized KEGG signaling pathways using pathview