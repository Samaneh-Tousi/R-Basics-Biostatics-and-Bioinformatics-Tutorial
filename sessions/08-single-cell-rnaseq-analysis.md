# Single-cell RNA-seq Analysis with Seurat and extra

This tutorial demonstrates a complete single-cell RNA-seq workflow using **Seurat** and related R packages.

The workflow includes:

- loading embedded example data from Seurat
- gene and cell filtering
- mitochondrial and ribosomal QC
- different mitochondrial thresholds for scRNA-seq and snRNA-seq
- outlier detection using detected genes and UMI counts
- doublet detection using `scDblFinder`
- normalization
- variable feature detection
- scaling
- PCA
- PC selection using elbow plot and explained variance
- UMAP and t-SNE
- graph-based clustering
- cluster marker detection
- cluster annotation using Azimuth
- extraction and subclustering of a specific cell type
- pseudobulk aggregation by cell type, sample, and condition
- differential expression analysis on pseudobulk values
- trajectory analysis using Slingshot
- cellular heterogeneity analysis
- integration method benchmarking

---

# 1. Install and load packages

Install packages only once. After installation, you only need to load them with `library()`.

Repeatedly reinstalling packages that already exist can sometimes cause version conflicts or installation issues.

```{r install-packages, eval=FALSE}
install.packages(c(
  "Seurat",
  "SeuratObject",
  "tidyverse",
  "patchwork",
  "Matrix",
  "ggplot2",
  "pheatmap",
  "remotes"
))

if (!requireNamespace("BiocManager", quietly = TRUE)) {
  install.packages("BiocManager")
}

BiocManager::install(c(
  "SingleCellExperiment",
  "scDblFinder",
  "scran",
  "scater",
  "DESeq2",
  "edgeR",
  "slingshot",
  "tradeSeq",
  "BiocParallel"
))

remotes::install_github("satijalab/azimuth")
```

```{r load-packages}
library(Seurat)
library(SeuratObject)
library(SingleCellExperiment)
library(scDblFinder)
library(scran)
library(scater)
library(DESeq2)
library(edgeR)
library(slingshot)
library(tradeSeq)
library(tidyverse)
library(Matrix)
library(patchwork)
library(ggplot2)
library(pheatmap)
```

---

# 2. Load embedded Seurat example data

Seurat includes a small example object called `pbmc_small`.

This dataset is useful for demonstrating the workflow, but it is too small for a real biological analysis.

For real projects, replace this section with your own 10X Genomics or count matrix input.

```{r load-data}
data("pbmc_small")

seurat_obj <- pbmc_small

seurat_obj
```

---

# 3. Inspect the object

```{r inspect-object}
seurat_obj

dim(seurat_obj)

head(colnames(seurat_obj))
head(rownames(seurat_obj))

seurat_obj@meta.data %>%
  head()
```

---

# 4. Add example sample and condition metadata

The embedded `pbmc_small` object does not contain real experimental conditions.

For demonstration, we create artificial metadata.

For your real data, replace this section with your true metadata table.

```{r add-example-metadata}
set.seed(123)

seurat_obj$sample_id <- sample(
  c("Sample_1", "Sample_2", "Sample_3", "Sample_4"),
  size = ncol(seurat_obj),
  replace = TRUE
)

seurat_obj$condition <- ifelse(
  seurat_obj$sample_id %in% c("Sample_1", "Sample_2"),
  "Control",
  "Treatment"
)

table(seurat_obj$sample_id)
table(seurat_obj$condition)
```

---

# 5. Gene filtering

Genes detected in very few cells or nuclei are usually uninformative and can add noise.

A common rule is to keep genes expressed in at least **3 to 10 cells**.

For small demonstration data, we use `min_cells <- 3`.

For real data, you can increase this to 5 or 10.

```{r gene-filtering}
min_cells <- 3

counts <- GetAssayData(seurat_obj, assay = "RNA", layer = "counts")

genes_to_keep <- Matrix::rowSums(counts > 0) >= min_cells

table(genes_to_keep)

seurat_obj <- subset(
  seurat_obj,
  features = rownames(seurat_obj)[genes_to_keep]
)

seurat_obj
```

---

# 6. Calculate mitochondrial and ribosomal percentages

Mitochondrial genes are commonly detected using gene names beginning with:

- `MT-` in human
- `mt-` in mouse

Ribosomal genes are often detected using:

- `RPS`
- `RPL`
- `Rps`
- `Rpl`

The embedded `pbmc_small` object has limited gene names, so these percentages may be zero or very small.

```{r qc-percentages}
seurat_obj[["percent.mt"]] <- PercentageFeatureSet(
  seurat_obj,
  pattern = "^MT-|^mt-"
)

seurat_obj[["percent.ribo"]] <- PercentageFeatureSet(
  seurat_obj,
  pattern = "^RPS|^RPL|^Rps|^Rpl"
)

head(seurat_obj@meta.data)
```

---

# 7. Initial QC visualization

Important QC metrics include:

- `nFeature_RNA`: number of detected genes per cell
- `nCount_RNA`: total UMI counts per cell
- `percent.mt`: mitochondrial percentage
- `percent.ribo`: ribosomal percentage

```{r qc-violin-plots}
VlnPlot(
  seurat_obj,
  features = c("nFeature_RNA", "nCount_RNA", "percent.mt", "percent.ribo"),
  ncol = 4
)
```

```{r qc-scatter-plots}
plot1 <- FeatureScatter(
  seurat_obj,
  feature1 = "nCount_RNA",
  feature2 = "percent.mt"
)

plot2 <- FeatureScatter(
  seurat_obj,
  feature1 = "nCount_RNA",
  feature2 = "nFeature_RNA"
)

plot1 + plot2
```

---

# 8. Recommended mitochondrial thresholds for scRNA-seq and snRNA-seq

Mitochondrial thresholds should not be blindly copied between experiments.

They depend on:

- tissue type
- organism
- sample quality
- dissociation stress
- sequencing method
- whether the assay is single-cell RNA-seq or single-nucleus RNA-seq

Typical starting values:

| Data type | Common starting mitochondrial threshold |
|---|---|
| scRNA-seq | 5% to 20% |
| snRNA-seq | 1% to 5%, often lower than scRNA-seq |

In scRNA-seq, high mitochondrial RNA can indicate stressed or dying cells.

In snRNA-seq, nuclei usually contain less cytoplasmic and mitochondrial RNA, so mitochondrial thresholds are often lower.

However, the best threshold should be selected after inspecting the QC distributions.

```{r set-qc-thresholds}
data_type <- "scRNAseq"

if (data_type == "scRNAseq") {
  mt_threshold <- 15
} else if (data_type == "snRNAseq") {
  mt_threshold <- 5
}

mt_threshold
```

---

# 9. Detect outlier cells using MAD-based thresholds

Instead of using fixed thresholds only, we can detect outliers based on the median absolute deviation, or MAD.

This is useful because each dataset has different count and gene-number distributions.

```{r mad-outlier-function}
is_outlier <- function(x, nmads = 3, type = c("both", "lower", "higher")) {
  type <- match.arg(type)
  
  med <- median(x, na.rm = TRUE)
  mad_value <- mad(x, na.rm = TRUE)
  
  lower <- med - nmads * mad_value
  higher <- med + nmads * mad_value
  
  if (type == "both") {
    return(x < lower | x > higher)
  }
  
  if (type == "lower") {
    return(x < lower)
  }
  
  if (type == "higher") {
    return(x > higher)
  }
}
```

```{r detect-outliers}
seurat_obj$low_gene_outlier <- is_outlier(
  seurat_obj$nFeature_RNA,
  nmads = 3,
  type = "lower"
)

seurat_obj$high_gene_outlier <- is_outlier(
  seurat_obj$nFeature_RNA,
  nmads = 3,
  type = "higher"
)

seurat_obj$low_count_outlier <- is_outlier(
  seurat_obj$nCount_RNA,
  nmads = 3,
  type = "lower"
)

seurat_obj$high_count_outlier <- is_outlier(
  seurat_obj$nCount_RNA,
  nmads = 3,
  type = "higher"
)

seurat_obj$high_mt_outlier <- seurat_obj$percent.mt > mt_threshold

table(seurat_obj$low_gene_outlier)
table(seurat_obj$high_gene_outlier)
table(seurat_obj$low_count_outlier)
table(seurat_obj$high_count_outlier)
table(seurat_obj$high_mt_outlier)
```

---

# 10. Remove poor-quality cells

For real datasets, high detected genes and high UMI counts may indicate doublets.

However, doublet detection will also be performed using `scDblFinder`.

```{r remove-qc-outliers}
seurat_obj <- subset(
  seurat_obj,
  subset =
    low_gene_outlier == FALSE &
    low_count_outlier == FALSE &
    high_mt_outlier == FALSE
)

seurat_obj
```

---

# 11. Doublet detection with scDblFinder

Doublets occur when two or more cells are captured together and incorrectly treated as one cell.

`scDblFinder` works on a `SingleCellExperiment` object, so we convert the Seurat object, run doublet detection, and then transfer results back to Seurat.

```{r scdblfinder}
sce <- as.SingleCellExperiment(seurat_obj)

set.seed(123)

sce <- scDblFinder(
  sce,
  samples = sce$sample_id
)

table(sce$scDblFinder.class)

seurat_obj$scDblFinder.score <- sce$scDblFinder.score
seurat_obj$scDblFinder.class <- sce$scDblFinder.class
```

```{r remove-doublets}
seurat_singlets <- subset(
  seurat_obj,
  subset = scDblFinder.class == "singlet"
)

seurat_singlets
```

---

# 12. Normalize data

Seurat’s standard log-normalization normalizes gene expression for each cell by total expression, multiplies by a scale factor, and log-transforms the result.

```{r normalize-data}
seurat_singlets <- NormalizeData(
  seurat_singlets,
  normalization.method = "LogNormalize",
  scale.factor = 10000
)
```

---

# 13. Find highly variable genes

Highly variable genes are genes that show strong biological variation across cells.

These genes are used for PCA and downstream clustering.

```{r variable-features}
seurat_singlets <- FindVariableFeatures(
  seurat_singlets,
  selection.method = "vst",
  nfeatures = 2000
)

top10_hvg <- head(VariableFeatures(seurat_singlets), 10)

top10_hvg
```

```{r variable-feature-plot}
plot1 <- VariableFeaturePlot(seurat_singlets)
plot2 <- LabelPoints(
  plot = plot1,
  points = top10_hvg,
  repel = TRUE
)

plot1 + plot2
```

---

# 14. Scale data

Scaling centers and scales gene expression values.

This step is needed before PCA.

You can also regress out unwanted technical effects such as mitochondrial percentage.

Use regression carefully because over-regression can remove real biological signal.

```{r scale-data}
all_genes <- rownames(seurat_singlets)

seurat_singlets <- ScaleData(
  seurat_singlets,
  features = all_genes,
  vars.to.regress = c("percent.mt")
)
```

---

# 15. PCA

PCA reduces high-dimensional gene expression data into principal components.

```{r run-pca}
seurat_singlets <- RunPCA(
  seurat_singlets,
  features = VariableFeatures(seurat_singlets)
)

print(seurat_singlets[["pca"]], dims = 1:5, nfeatures = 5)
```

```{r pca-visualization}
VizDimLoadings(
  seurat_singlets,
  dims = 1:2,
  reduction = "pca"
)

DimPlot(
  seurat_singlets,
  reduction = "pca",
  group.by = "condition"
)

DimHeatmap(
  seurat_singlets,
  dims = 1:6,
  cells = 50,
  balanced = TRUE
)
```

---

# 16. Select significant PCs before UMAP

Before running `RunUMAP()`, determine how many PCs to use.

Two common approaches are:

1. elbow plot
2. explained variance threshold

```{r elbow-plot}
ElbowPlot(seurat_singlets, ndims = 30)
```

```{r explained-variance}
pca_stdev <- seurat_singlets[["pca"]]@stdev

pca_variance <- pca_stdev^2
pca_variance_percent <- pca_variance / sum(pca_variance) * 100
cumulative_variance <- cumsum(pca_variance_percent)

pc_table <- data.frame(
  PC = seq_along(pca_variance_percent),
  VariancePercent = pca_variance_percent,
  CumulativeVariance = cumulative_variance
)

head(pc_table, 20)

ggplot(pc_table, aes(x = PC, y = VariancePercent)) +
  geom_point() +
  geom_line() +
  theme_classic() +
  labs(
    title = "Variance explained by each PC",
    x = "Principal component",
    y = "Variance explained (%)"
  )

ggplot(pc_table, aes(x = PC, y = CumulativeVariance)) +
  geom_point() +
  geom_line() +
  theme_classic() +
  labs(
    title = "Cumulative variance explained",
    x = "Principal component",
    y = "Cumulative variance (%)"
  )
```

```{r choose-pcs}
pcs_to_use <- which(cumulative_variance <= 80)

if (length(pcs_to_use) < 5) {
  pcs_to_use <- 1:min(10, length(pca_stdev))
}

pcs_to_use
```

---

# 17. Find neighbors

Seurat clustering is usually performed on a nearest-neighbor graph built from selected PCs.

```{r find-neighbors}
seurat_singlets <- FindNeighbors(
  seurat_singlets,
  dims = pcs_to_use
)
```

---

# 18. Clustering

Seurat supports graph-based clustering.

The resolution parameter controls cluster granularity.

Lower resolution gives fewer clusters.

Higher resolution gives more clusters.

```{r clustering}
seurat_singlets <- FindClusters(
  seurat_singlets,
  resolution = 0.5,
  algorithm = 1
)

table(Idents(seurat_singlets))
```

Common Seurat clustering algorithms:

| Algorithm value | Method |
|---|---|
| 1 | Louvain |
| 2 | Louvain with multilevel refinement |
| 3 | SLM |
| 4 | Leiden |

Leiden requires the `leidenbase` package.

```{r leiden-clustering, eval=FALSE}
install.packages("leidenbase")

seurat_singlets <- FindClusters(
  seurat_singlets,
  resolution = 0.5,
  algorithm = 4
)
```

---

# 19. UMAP

UMAP is commonly used to visualize local and global cell-state structure in two dimensions.

UMAP is not used to calculate clusters here.

Clusters are calculated from the PCA-based neighbor graph.

```{r run-umap}
seurat_singlets <- RunUMAP(
  seurat_singlets,
  dims = pcs_to_use
)

DimPlot(
  seurat_singlets,
  reduction = "umap",
  label = TRUE
) +
  ggtitle("UMAP by Seurat clusters")
```

```{r umap-condition}
DimPlot(
  seurat_singlets,
  reduction = "umap",
  group.by = "condition"
) +
  ggtitle("UMAP by condition")
```

---

# 20. t-SNE

t-SNE is another nonlinear dimensionality reduction method.

Main differences:

| Method | Main use |
|---|---|
| UMAP | Often preserves more global structure and is widely used for scRNA-seq visualization |
| t-SNE | Often emphasizes local neighborhoods but can separate groups strongly |
| PCA | Linear reduction used for clustering input |
| UMAP/t-SNE | Mainly visualization tools |

```{r run-tsne}
seurat_singlets <- RunTSNE(
  seurat_singlets,
  dims = pcs_to_use
)

DimPlot(
  seurat_singlets,
  reduction = "tsne",
  label = TRUE
) +
  ggtitle("t-SNE by Seurat clusters")
```

```{r compare-umap-tsne}
p1 <- DimPlot(
  seurat_singlets,
  reduction = "umap",
  label = TRUE
) +
  ggtitle("UMAP")

p2 <- DimPlot(
  seurat_singlets,
  reduction = "tsne",
  label = TRUE
) +
  ggtitle("t-SNE")

p1 + p2
```

---

# 21. Cluster markers

Cluster marker genes help identify the biological identity of each cluster.

```{r cluster-markers}
cluster_markers <- FindAllMarkers(
  seurat_singlets,
  only.pos = TRUE,
  min.pct = 0.25,
  logfc.threshold = 0.25
)

head(cluster_markers)

write.csv(
  cluster_markers,
  "cluster_markers.csv",
  row.names = FALSE
)
```

```{r top-markers}
top_markers <- cluster_markers %>%
  group_by(cluster) %>%
  slice_max(order_by = avg_log2FC, n = 5)

top_markers
```

```{r marker-heatmap}
DoHeatmap(
  seurat_singlets,
  features = unique(top_markers$gene)
) +
  NoLegend()
```

---

# 22. Example marker visualization

For real PBMC data, common markers include:

| Cell type | Example markers |
|---|---|
| T cells | CD3D, CD3E, IL7R |
| B cells | MS4A1, CD79A |
| NK cells | NKG7, GNLY |
| Monocytes | LST1, S100A8, S100A9 |
| Dendritic cells | FCER1A, CST3 |
| Platelets | PPBP |

The embedded `pbmc_small` object may not contain all these genes.

```{r marker-featureplots}
marker_genes <- c(
  "CD3D", "CD3E", "IL7R",
  "MS4A1", "CD79A",
  "NKG7", "GNLY",
  "LST1", "S100A8", "S100A9",
  "FCER1A", "CST3",
  "PPBP"
)

marker_genes_present <- marker_genes[marker_genes %in% rownames(seurat_singlets)]

marker_genes_present

if (length(marker_genes_present) > 0) {
  FeaturePlot(
    seurat_singlets,
    features = marker_genes_present,
    reduction = "umap"
  )
}
```

---

# 23. Cluster annotation with Azimuth

Azimuth performs reference-based annotation.

For PBMC data, you can use a PBMC reference.

This section may require downloading the reference and having compatible package versions.

```{r azimuth-annotation, eval=FALSE}
library(Azimuth)

seurat_azimuth <- RunAzimuth(
  query = seurat_singlets,
  reference = "pbmcref"
)

head(seurat_azimuth@meta.data)

DimPlot(
  seurat_azimuth,
  group.by = "predicted.celltype.l2",
  label = TRUE,
  repel = TRUE
) +
  ggtitle("Azimuth cell type annotation")
```

Because the embedded `pbmc_small` dataset is very small, Azimuth annotation may not be ideal.

For real PBMC datasets, Azimuth is more useful.

For this tutorial, we create a simple marker-based example annotation.

```{r simple-manual-annotation}
seurat_singlets$manual_celltype <- paste0(
  "Cluster_",
  Idents(seurat_singlets)
)

table(seurat_singlets$manual_celltype)

DimPlot(
  seurat_singlets,
  reduction = "umap",
  group.by = "manual_celltype",
  label = TRUE
)
```

---

# 24. Extract one cell type and perform subclustering

Subclustering is useful when you want to study one major cell type in more detail.

For example:

- T cell subtypes
- monocyte subsets
- rare immune populations
- malignant subclones
- stem/progenitor states

Here we extract one available cell type from the tutorial object.

```{r select-celltype-for-subclustering}
selected_celltype <- unique(seurat_singlets$celltype)[1]

selected_celltype
```

```{r subset-celltype}
celltype_subset <- subset(
  seurat_singlets,
  subset = celltype == selected_celltype
)

celltype_subset
```

```{r recluster-subset}
celltype_subset <- NormalizeData(celltype_subset)

celltype_subset <- FindVariableFeatures(
  celltype_subset,
  selection.method = "vst",
  nfeatures = 1000
)

celltype_subset <- ScaleData(celltype_subset)

celltype_subset <- RunPCA(
  celltype_subset,
  features = VariableFeatures(celltype_subset)
)

ElbowPlot(celltype_subset, ndims = 20)

subset_pcs <- 1:min(10, ncol(Embeddings(celltype_subset, "pca")))

celltype_subset <- FindNeighbors(
  celltype_subset,
  dims = subset_pcs
)

celltype_subset <- FindClusters(
  celltype_subset,
  resolution = 0.3
)

celltype_subset <- RunUMAP(
  celltype_subset,
  dims = subset_pcs
)

DimPlot(
  celltype_subset,
  reduction = "umap",
  label = TRUE
) +
  ggtitle(paste("Subclusters of", selected_celltype))
```

---

# 25. Detect markers of expected and rare subclusters

Rare subclusters may represent:

- rare cell states
- activated cells
- cycling cells
- stressed cells
- doublet contamination
- disease-associated populations
- technical artifacts

Always validate rare subclusters carefully.

```{r subcluster-markers}
subcluster_markers <- FindAllMarkers(
  celltype_subset,
  only.pos = TRUE,
  min.pct = 0.20,
  logfc.threshold = 0.25
)

head(subcluster_markers)

write.csv(
  subcluster_markers,
  paste0("subcluster_markers_", selected_celltype, ".csv"),
  row.names = FALSE
)
```

```{r rare-subclusters}
subcluster_sizes <- table(Idents(celltype_subset))

subcluster_sizes

rare_subclusters <- names(subcluster_sizes[subcluster_sizes < quantile(subcluster_sizes, 0.25)])

rare_subclusters
```
---

# 26. Pseudobulk aggregation

Single-cell data contain many cells from the same biological sample.

However, cells from the same sample are not fully independent biological replicates.

For condition-level differential expression, it is often better to aggregate counts by:

- sample
- condition
- cell type

This creates pseudobulk count matrices.

Then differential expression can be performed using bulk RNA-seq tools such as DESeq2 or edgeR.

This reduces false confidence caused by treating thousands of cells as independent samples.

```{r prepare-pseudobulk-metadata}
seurat_singlets$celltype <- seurat_singlets$manual_celltype

meta <- seurat_singlets@meta.data %>%
  rownames_to_column("cell")

head(meta)
```

```{r pseudobulk-function}
create_pseudobulk_counts <- function(seurat_object,
                                     group_vars = c("celltype", "sample_id", "condition"),
                                     assay = "RNA") {
  
  counts <- GetAssayData(
    seurat_object,
    assay = assay,
    layer = "counts"
  )
  
  metadata <- seurat_object@meta.data %>%
    rownames_to_column("cell")
  
  metadata$pseudobulk_group <- metadata %>%
    select(all_of(group_vars)) %>%
    unite("group", everything(), sep = "__", remove = FALSE) %>%
    pull(group)
  
  group_factor <- factor(metadata$pseudobulk_group)
  
  design_matrix <- Matrix::sparse.model.matrix(~ 0 + group_factor)
  colnames(design_matrix) <- levels(group_factor)
  
  pseudobulk_counts <- counts %*% design_matrix
  
  pseudobulk_metadata <- metadata %>%
    distinct(pseudobulk_group, across(all_of(group_vars))) %>%
    arrange(pseudobulk_group)
  
  pseudobulk_metadata <- pseudobulk_metadata %>%
    column_to_rownames("pseudobulk_group")
  
  pseudobulk_counts <- pseudobulk_counts[, rownames(pseudobulk_metadata)]
  
  list(
    counts = pseudobulk_counts,
    metadata = pseudobulk_metadata
  )
}
```

```{r create-pseudobulk}
pb <- create_pseudobulk_counts(
  seurat_singlets,
  group_vars = c("celltype", "sample_id", "condition")
)

pseudobulk_counts <- pb$counts
pseudobulk_metadata <- pb$metadata

dim(pseudobulk_counts)
head(pseudobulk_metadata)
```

---

# 27. Pseudobulk DEA with DESeq2

Differential expression is usually performed separately for each cell type.

The example data are too small for a real DEA, so this code is written as a reusable template.

```{r pseudobulk-deseq2-function}
run_pseudobulk_deseq2 <- function(pseudobulk_counts,
                                  pseudobulk_metadata,
                                  selected_celltype,
                                  condition_col = "condition",
                                  reference_condition = "Control",
                                  comparison_condition = "Treatment") {
  
  selected_samples <- rownames(pseudobulk_metadata)[
    pseudobulk_metadata$celltype == selected_celltype
  ]
  
  counts_subset <- pseudobulk_counts[, selected_samples, drop = FALSE]
  metadata_subset <- pseudobulk_metadata[selected_samples, , drop = FALSE]
  
  keep_genes <- rowSums(counts_subset >= 10) >= 2
  counts_subset <- counts_subset[keep_genes, , drop = FALSE]
  
  metadata_subset[[condition_col]] <- factor(
    metadata_subset[[condition_col]],
    levels = c(reference_condition, comparison_condition)
  )
  
  if (ncol(counts_subset) < 4) {
    message("Not enough pseudobulk samples for reliable DESeq2 analysis.")
    return(NULL)
  }
  
  dds <- DESeqDataSetFromMatrix(
    countData = round(as.matrix(counts_subset)),
    colData = metadata_subset,
    design = as.formula(paste0("~ ", condition_col))
  )
  
  dds <- DESeq(dds)
  
  res <- results(
    dds,
    contrast = c(condition_col, comparison_condition, reference_condition)
  )
  
  res_df <- as.data.frame(res) %>%
    rownames_to_column("gene") %>%
    arrange(padj)
  
  return(res_df)
}
```

```{r run-example-pseudobulk-dea}
available_celltypes <- unique(pseudobulk_metadata$celltype)

available_celltypes

dea_results <- run_pseudobulk_deseq2(
  pseudobulk_counts = pseudobulk_counts,
  pseudobulk_metadata = pseudobulk_metadata,
  selected_celltype = available_celltypes[1],
  condition_col = "condition",
  reference_condition = "Control",
  comparison_condition = "Treatment"
)

if (!is.null(dea_results)) {
  head(dea_results)
  
  write.csv(
    dea_results,
    paste0("DEA_pseudobulk_", available_celltypes[1], ".csv"),
    row.names = FALSE
  )
}
```

---

# 28. Pseudobulk volcano plot

```{r pseudobulk-volcano}
plot_volcano <- function(dea_df,
                         padj_cutoff = 0.05,
                         log2fc_cutoff = 1) {
  
  if (is.null(dea_df)) {
    message("DEA results are NULL.")
    return(NULL)
  }
  
  dea_df <- dea_df %>%
    mutate(
      significant = ifelse(
        padj < padj_cutoff & abs(log2FoldChange) > log2fc_cutoff,
        "Significant",
        "Not significant"
      )
    )
  
  ggplot(dea_df, aes(x = log2FoldChange, y = -log10(padj))) +
    geom_point(aes(shape = significant), alpha = 0.7) +
    theme_classic() +
    labs(
      title = "Pseudobulk differential expression",
      x = "log2 fold change",
      y = "-log10 adjusted p-value"
    )
}

plot_volcano(dea_results)
```

---

# 29. Trajectory analysis with Slingshot

Trajectory analysis is useful when cells are expected to follow a continuous biological process, such as:

- differentiation
- activation
- development
- disease progression

Slingshot uses reduced dimensions and cluster labels to infer lineages and pseudotime.

Here we run Slingshot on UMAP coordinates for demonstration.

For real trajectory analysis, PCA or diffusion map coordinates may sometimes be preferable.

```{r prepare-slingshot}
sce_sling <- as.SingleCellExperiment(seurat_singlets)

reducedDims(sce_sling)$UMAP <- Embeddings(
  seurat_singlets,
  reduction = "umap"
)

sce_sling$cluster <- as.character(Idents(seurat_singlets))
sce_sling$condition <- seurat_singlets$condition
```

```{r run-slingshot}
sce_sling <- slingshot(
  sce_sling,
  clusterLabels = "cluster",
  reducedDim = "UMAP"
)

summary(slingPseudotime(sce_sling))
```

```{r plot-slingshot}
umap_df <- as.data.frame(reducedDims(sce_sling)$UMAP)
colnames(umap_df) <- c("UMAP_1", "UMAP_2")

umap_df$cluster <- sce_sling$cluster
umap_df$condition <- sce_sling$condition

plot(
  reducedDims(sce_sling)$UMAP,
  col = as.numeric(factor(sce_sling$cluster)),
  pch = 16,
  asp = 1,
  xlab = "UMAP 1",
  ylab = "UMAP 2",
  main = "Slingshot trajectory"
)

lines(SlingshotDataSet(sce_sling), lwd = 2)
```

---

# 30. Compare pseudotime by condition

```{r pseudotime-condition}
pseudotime_df <- as.data.frame(slingPseudotime(sce_sling)) %>%
  rownames_to_column("cell")

pseudotime_df$condition <- sce_sling$condition

head(pseudotime_df)

pseudotime_long <- pseudotime_df %>%
  pivot_longer(
    cols = starts_with("Lineage"),
    names_to = "lineage",
    values_to = "pseudotime"
  )

ggplot(
  pseudotime_long,
  aes(x = condition, y = pseudotime)
) +
  geom_boxplot(outlier.shape = NA) +
  geom_jitter(width = 0.2, alpha = 0.5) +
  theme_classic() +
  labs(
    title = "Pseudotime by condition",
    x = "Condition",
    y = "Pseudotime"
  )
```

---

# 31. Cellular heterogeneity across cell types

Cellular heterogeneity can be measured in several ways.

Here we calculate:

- cell type proportions per sample
- Shannon diversity index per sample
- Simpson diversity index per sample

Higher Shannon diversity usually indicates more diverse cell type composition.

```{r heterogeneity-functions}
calculate_shannon <- function(proportions) {
  proportions <- proportions[proportions > 0]
  -sum(proportions * log(proportions))
}

calculate_simpson <- function(proportions) {
  proportions <- proportions[proportions > 0]
  1 - sum(proportions^2)
}
```

```{r celltype-proportions}
celltype_counts <- seurat_singlets@meta.data %>%
  count(sample_id, condition, celltype)

celltype_proportions <- celltype_counts %>%
  group_by(sample_id) %>%
  mutate(proportion = n / sum(n)) %>%
  ungroup()

celltype_proportions
```

```{r plot-celltype-proportions}
ggplot(
  celltype_proportions,
  aes(x = sample_id, y = proportion, fill = celltype)
) +
  geom_col() +
  theme_classic() +
  labs(
    title = "Cell type proportions per sample",
    x = "Sample",
    y = "Proportion"
  )
```

```{r diversity-indices}
heterogeneity_df <- celltype_proportions %>%
  group_by(sample_id, condition) %>%
  summarise(
    Shannon = calculate_shannon(proportion),
    Simpson = calculate_simpson(proportion),
    .groups = "drop"
  )

heterogeneity_df
```

```{r plot-heterogeneity}
ggplot(
  heterogeneity_df,
  aes(x = condition, y = Shannon)
) +
  geom_boxplot(outlier.shape = NA) +
  geom_jitter(width = 0.15) +
  theme_classic() +
  labs(
    title = "Cellular heterogeneity by condition",
    x = "Condition",
    y = "Shannon diversity"
  )

ggplot(
  heterogeneity_df,
  aes(x = condition, y = Simpson)
) +
  geom_boxplot(outlier.shape = NA) +
  geom_jitter(width = 0.15) +
  theme_classic() +
  labs(
    title = "Cellular heterogeneity by condition",
    x = "Condition",
    y = "Simpson diversity"
  )
```

---

# 32. Save processed Seurat objects

```{r save-objects}
saveRDS(
  seurat_singlets,
  "seurat_singlets_processed.rds"
)

saveRDS(
  celltype_subset,
  paste0("subclustered_", selected_celltype, ".rds")
)
```

---

# 33. Integration tools benchmark

Integration is used when combining datasets from different:

- batches
- donors
- conditions
- technologies
- sequencing runs

The goal is to reduce technical differences while preserving biological differences.

Common integration tools include:

- Seurat CCA integration
- Seurat RPCA integration
- Harmony
- fastMNN
- scVI

Benchmarking integration should evaluate both:

1. batch mixing
2. biological conservation

Useful benchmark metrics include:

- silhouette score by cell type
- silhouette score by batch
- kBET
- graph connectivity
- LISI
- marker conservation
- differential expression preservation

The embedded example dataset is too small for meaningful benchmarking, so the code below is a template for real multi-sample data.

---

## 33.1 Split object by sample

```{r split-by-sample}
object_list <- SplitObject(
  seurat_singlets,
  split.by = "sample_id"
)

length(object_list)
names(object_list)
```

---

## 33.2 Seurat CCA integration

```{r seurat-cca-integration, eval=FALSE}
object_list_cca <- lapply(
  object_list,
  function(x) {
    x <- NormalizeData(x)
    x <- FindVariableFeatures(x, selection.method = "vst", nfeatures = 2000)
    return(x)
  }
)

features_cca <- SelectIntegrationFeatures(
  object.list = object_list_cca,
  nfeatures = 2000
)

anchors_cca <- FindIntegrationAnchors(
  object.list = object_list_cca,
  anchor.features = features_cca,
  reduction = "cca"
)

integrated_cca <- IntegrateData(
  anchorset = anchors_cca
)

DefaultAssay(integrated_cca) <- "integrated"

integrated_cca <- ScaleData(integrated_cca)
integrated_cca <- RunPCA(integrated_cca)
integrated_cca <- RunUMAP(integrated_cca, dims = 1:20)
integrated_cca <- FindNeighbors(integrated_cca, dims = 1:20)
integrated_cca <- FindClusters(integrated_cca, resolution = 0.5)

DimPlot(integrated_cca, group.by = "sample_id") +
  ggtitle("Seurat CCA integration by sample")

DimPlot(integrated_cca, group.by = "celltype", label = TRUE) +
  ggtitle("Seurat CCA integration by cell type")
```

---

## 33.3 Seurat RPCA integration

```{r seurat-rpca-integration, eval=FALSE}
object_list_rpca <- lapply(
  object_list,
  function(x) {
    x <- NormalizeData(x)
    x <- FindVariableFeatures(x, selection.method = "vst", nfeatures = 2000)
    x <- ScaleData(x)
    x <- RunPCA(x)
    return(x)
  }
)

features_rpca <- SelectIntegrationFeatures(
  object.list = object_list_rpca,
  nfeatures = 2000
)

object_list_rpca <- PrepSCTIntegration(
  object.list = object_list_rpca,
  anchor.features = features_rpca
)

anchors_rpca <- FindIntegrationAnchors(
  object.list = object_list_rpca,
  anchor.features = features_rpca,
  reduction = "rpca"
)

integrated_rpca <- IntegrateData(
  anchorset = anchors_rpca
)

DefaultAssay(integrated_rpca) <- "integrated"

integrated_rpca <- ScaleData(integrated_rpca)
integrated_rpca <- RunPCA(integrated_rpca)
integrated_rpca <- RunUMAP(integrated_rpca, dims = 1:20)
integrated_rpca <- FindNeighbors(integrated_rpca, dims = 1:20)
integrated_rpca <- FindClusters(integrated_rpca, resolution = 0.5)

DimPlot(integrated_rpca, group.by = "sample_id") +
  ggtitle("Seurat RPCA integration by sample")

DimPlot(integrated_rpca, group.by = "celltype", label = TRUE) +
  ggtitle("Seurat RPCA integration by cell type")
```

---

## 33.4 Harmony integration

```{r harmony-integration, eval=FALSE}
install.packages("harmony")
library(harmony)

harmony_obj <- seurat_singlets

harmony_obj <- NormalizeData(harmony_obj)
harmony_obj <- FindVariableFeatures(harmony_obj)
harmony_obj <- ScaleData(harmony_obj)
harmony_obj <- RunPCA(harmony_obj)

harmony_obj <- RunHarmony(
  object = harmony_obj,
  group.by.vars = "sample_id"
)

harmony_obj <- RunUMAP(
  harmony_obj,
  reduction = "harmony",
  dims = 1:20
)

harmony_obj <- FindNeighbors(
  harmony_obj,
  reduction = "harmony",
  dims = 1:20
)

harmony_obj <- FindClusters(
  harmony_obj,
  resolution = 0.5
)

DimPlot(harmony_obj, group.by = "sample_id") +
  ggtitle("Harmony integration by sample")

DimPlot(harmony_obj, group.by = "celltype", label = TRUE) +
  ggtitle("Harmony integration by cell type")
```

---

## 33.5 fastMNN integration

```{r fastmnn-integration, eval=FALSE}
BiocManager::install("batchelor")
library(batchelor)

sce_list <- lapply(
  object_list,
  as.SingleCellExperiment
)

mnn_out <- fastMNN(
  sce_list,
  batch = names(sce_list)
)

reducedDim(mnn_out, "MNN")[1:5, 1:5]
```

---

# 34. Simple benchmark metrics

This section calculates simple visual and quantitative metrics.

For real benchmarking, use dedicated packages such as `scIB`, `scIB-metrics`, or other integration benchmark frameworks.

```{r silhouette-function, eval=FALSE}
install.packages("cluster")
library(cluster)

calculate_silhouette <- function(seurat_object,
                                 reduction = "pca",
                                 group_col = "celltype",
                                 dims = 1:20) {
  
  embeddings <- Embeddings(seurat_object, reduction = reduction)[, dims]
  groups <- seurat_object@meta.data[[group_col]]
  
  distance_matrix <- dist(embeddings)
  
  sil <- silhouette(
    as.numeric(factor(groups)),
    distance_matrix
  )
  
  mean(sil[, 3])
}
```

```{r benchmark-example, eval=FALSE}
benchmark_results <- data.frame(
  method = c("Unintegrated", "CCA", "RPCA", "Harmony"),
  celltype_silhouette = c(
    calculate_silhouette(seurat_singlets, reduction = "pca", group_col = "celltype"),
    calculate_silhouette(integrated_cca, reduction = "pca", group_col = "celltype"),
    calculate_silhouette(integrated_rpca, reduction = "pca", group_col = "celltype"),
    calculate_silhouette(harmony_obj, reduction = "harmony", group_col = "celltype")
  ),
  batch_silhouette = c(
    calculate_silhouette(seurat_singlets, reduction = "pca", group_col = "sample_id"),
    calculate_silhouette(integrated_cca, reduction = "pca", group_col = "sample_id"),
    calculate_silhouette(integrated_rpca, reduction = "pca", group_col = "sample_id"),
    calculate_silhouette(harmony_obj, reduction = "harmony", group_col = "sample_id")
  )
)

benchmark_results

ggplot(
  benchmark_results,
  aes(x = method, y = celltype_silhouette)
) +
  geom_col() +
  theme_classic() +
  labs(
    title = "Biological conservation benchmark",
    x = "Integration method",
    y = "Cell type silhouette"
  )

ggplot(
  benchmark_results,
  aes(x = method, y = batch_silhouette)
) +
  geom_col() +
  theme_classic() +
  labs(
    title = "Batch mixing benchmark",
    x = "Integration method",
    y = "Batch silhouette"
  )
```

---

# 35. Export final tables

```{r export-final-tables}
write.csv(
  seurat_singlets@meta.data,
  "final_cell_metadata.csv"
)

write.csv(
  cluster_markers,
  "final_cluster_markers.csv",
  row.names = FALSE
)

write.csv(
  celltype_proportions,
  "celltype_proportions.csv",
  row.names = FALSE
)

write.csv(
  heterogeneity_df,
  "cellular_heterogeneity_indices.csv",
  row.names = FALSE
)
```

---

# 36. Session information

Always report session information for reproducibility.

```{r session-info}
sessionInfo()
```