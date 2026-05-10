# Comparing scRNA-seq Protocols with Bioconductor simpleSingleCell
wokflow


# 🔬 Comparing scRNA-seq Protocols with Bioconductor

This repository contains a single-cell RNA-seq analysis and presentation
comparing **three scRNA‑seq protocols and datasets** using Bioconductor:

- **Paul et al.** – Mouse haematopoietic stem and progenitor cells
  (HSCs) profiled with **MARS‑seq** (plate-based UMI). \[file:265\]  
- **Bach et al.** – Mouse mammary epithelial and stromal cells profiled
  with **10X Genomics** (droplet-based 3′ UMI). \[file:265\]  
- **Messmer et al.** – Human embryonic stem cells (hESCs) profiled with
  **Smart‑seq2** (plate-based full-length). \[file:265\]

Each dataset represents a different experimental protocol, but all are
analyzed with a **common Bioconductor workflow** for QC, normalization,
HVG selection, dimensionality reduction, clustering, and marker
detection. \[file:265\]

------------------------------------------------------------------------

## 📁 Repository structure

Typical layout (adjust if needed):

- `analysis.qmd` Full analysis document that produces `analysis.html`

- `analysis_files/`  
  PNG figures saved from the analysis (QC plots, mean–variance plots,
  t‑SNE/UMAP, etc.).

- `index.qmd` \|Full analysis document that produces `index.html`

- `index_files/`

  - revealjs presentation and the plots generated from the `index.qmd`
    are saved here

- `README.qmd` / `README.md`  
  This project description.

------------------------------------------------------------------------

## 🧬 Background: from reads to count matrices

Single-cell RNA-seq begins with sequencing reads that must be summarized
into a **gene × cell expression matrix**.

- Each **row** corresponds to a gene, each **column** to a cell; entries
  are read or UMI counts per gene per cell.  
- **UMIs** (unique molecular identifiers) are interpreted like read
  counts but are less affected by PCR amplification artifacts. Modern
  scRNA‑seq protocols fall into three broad categories:

1.  **Droplet-based protocols** – 10X Genomics, inDrop, Drop‑seq (10X
    used in the Bach dataset).  
2.  **Plate-based protocols with UMIs** – CEL‑seq2, MARS‑seq (MARS‑seq
    used in the Paul dataset).  
3.  **Plate-based full‑length protocols** – Smart‑seq2, sciRNA‑seq
    (Smart‑seq2 used in the Messmer dataset).

Each protocol requires its own mapping, demultiplexing, and
UMI-de-duplication pipeline to generate a count matrix, but once counts
are obtained, downstream analysis can be unified in R/Bioconductor.
\[file:265\]

------------------------------------------------------------------------

## 📊 Datasets & protocols

All datasets are accessed via Bioconductor experiment packages:

- **Paul HSC (MARS‑seq)**
  - Mouse haematopoietic stem and progenitor cells.  
  - Plate-based MARS‑seq with UMIs.  
  - Represents a **plate + UMI** protocol.
- **Bach mammary (10X Genomics)**
  - Mouse mammary epithelial and stromal cells.  
  - Droplet-based 3′ UMI (10X Genomics).  
  - Represents a **droplet UMI** protocol.
- **Messmer hESC (Smart‑seq2)**
  - Human embryonic stem cells (naive, primed, early differentiation).  
  - Plate-based full-length Smart‑seq2 (non‑UMI).  
  - Represents a **plate full‑length reads** protocol.

------------------------------------------------------------------------

## 🧰 Methods: what was done and how

Across all three datasets, the analysis follows the same core steps:

1.  **Data loading and basic inspection**
    - Load counts into `SingleCellExperiment` objects using the
      appropriate Bioconductor data packages (e.g. `scRNAseq`).
    - Inspect dimensions, sample metadata, and basic distributions.
2.  **Per-cell quality control (QC)**
    - Compute per-cell metrics: library size, number of detected genes,
      mitochondrial percentage, and (where available) ERCC spike-in
      percentage.  
    - Use thresholds to remove cells with low counts, few detected
      genes, high mitochondrial content, or high spike-in fraction,
      tailored to each dataset.
3.  **Normalization**
    - Apply scran-style normalization (deconvolution size factors) and
      compute log‑normalized expression values.  
    - For Smart‑seq2, normalization is adapted to non‑UMI counts while
      still using size factors to account for sequencing depth and
      composition.
4.  **Variance modelling and HVG selection**
    - Fit mean–variance trends to distinguish technical noise from
      biological variation.  
    - Select highly variable genes (HVGs) that show more variation than
      expected at a given mean expression level, using either
      Poisson-based models or spike-in‑aware models.
5.  **Dimensionality reduction**
    - Perform PCA on HVGs to capture major sources of variation.  
    - Use t‑SNE or UMAP on the PCA space to visualize cell populations
      and trajectories in two dimensions.
6.  **Graph-based clustering**
    - Construct a shared nearest neighbour (SNN) graph from the PCA
      representation.  
    - Apply community detection (Louvain/Walktrap) to identify clusters
      of similar cells.
7.  **Marker detection and biological interpretation**
    - Compare clusters to find marker genes that are up-regulated in
      each group.  
    - Use literature markers (e.g. *Gata1*, *Krt5*, *POU5F1/OCT4*,
      *NANOG*) to interpret clusters as specific cell types or states.

------------------------------------------------------------------------

## 🔍 Protocol-specific highlights

### Paul HSC – MARS‑seq (plate + UMI)

- Focuses on mouse haematopoietic stem and progenitor cells measured
  with MARS‑seq.  
- QC is adapted to UMI-based plates and experimental conditions.  
- Dimensionality reduction and clustering reveal a **continuous
  differentiation trajectory** rather than sharply separated clusters,
  consistent with myeloid/erythroid lineage commitment.

### Bach mammary – 10X Genomics (droplet UMI)

- Focuses on mammary epithelial and stromal/immune cells profiled with
  10X.  
- QC emphasizes library size, number of features, and mitochondrial
  percentage, typical for droplet-based data.  
- Clustering identifies discrete cell populations (e.g. basal cells,
  luminal progenitors, stromal cells), well separated in t‑SNE/UMAP
  space.

### Messmer hESC – Smart‑seq2 (plate full-length)

- Focuses on human ESCs in naive and primed states plus early
  differentiation.  
- QC combines library size, detected genes, mitochondrial percentage,
  and ERCC spike-ins, with batch and phenotype information.  
- Normalization and variance modelling account for non‑UMI, full-length
  reads and multiple experiment batches.  
- Dimensionality reduction and clustering distinguish pluripotent states
  and early lineage trajectories; marker genes include pluripotency
  factors (*POU5F1/OCT4*, *NANOG*, *SOX2*).

------------------------------------------------------------------------

## 🧪 Results summary

- All three protocols can be analyzed with the same conceptual pipeline,
  but **QC and variance modelling must be tuned** to the protocol
  (droplet vs plate, UMI vs read).  
- MARS‑seq (Paul) emphasizes **continuous trajectories**, 10X (Bach)
  emphasizes **discrete cell types**, and Smart‑seq2 (Messmer)
  emphasizes **high-resolution state differences** within fewer, deeply
  sequenced cells.
- Graph-based clustering and marker detection provide a consistent way
  to define and interpret cell populations across protocols.

------------------------------------------------------------------------

## ▶️ How to run the analysis

1.  **Install Quarto** (if you haven’t already) and ensure R and
    Bioconductor are installed.

2.  **Install required R packages** in an R session (outside Quarto).
    For example:

    ``` r
    if (!requireNamespace("BiocManager", quietly = TRUE))
      install.packages("BiocManager")

    BiocManager::install(c(
      "SingleCellExperiment",
      "scRNAseq",
      "scater",
      "scran",
      "scuttle",
      "batchelor",
      "BiocSingular",
      "AnnotationHub",
      "igraph"
    ), ask = FALSE, update = TRUE)
    ```

3.  **Render the analysis document**:

    ``` bash
    quarto render analysis.qmd
    ```

    This produces `analysis.html` (already present in your repo).

4.  **Render the revealjs slides** (using the `.qmd` provided below or
    your existing one):

    ``` bash
    quarto render index.qmd
    ```

------------------------------------------------------------------------

## 📌 Take-home messages

- A **unified Bioconductor workflow** (SingleCellExperiment + scater +
  scran + graph clustering) can be applied across different scRNA‑seq
  protocols with relatively small adjustments.
- Protocol choice (MARS‑seq, 10X, Smart‑seq2) determines trade‑offs
  between cell numbers, read depth, and the kind of structure
  (continuous vs discrete) that is easiest to detect.
- Using Quarto allows you to combine **code, figures, and narrative** in
  a reproducible document (`analysis.html`) and a polished presentation
  (revealjs slides).

------------------------------------------------------------------------

## 📖 Attribution

- Paul HSC MARS‑seq dataset, Bach mammary 10X dataset, and Messmer hESC
  Smart‑seq2 dataset as described in the original publications and
  Bioconductor documentation.
- Methods and analysis structure inspired by Bioconductor’s
  simpleSingleCell and OSCA workflows, with appropriate credit in the
  analysis document.
