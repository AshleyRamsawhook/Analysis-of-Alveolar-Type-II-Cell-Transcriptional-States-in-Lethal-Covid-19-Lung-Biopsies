Analysis of Alveolar Type II Cell Transcriptional States in Lethal COVID-19 Lung Biopsies

By Ashley Ramsawhook, PhD.


Introduction

This project is an independent analysis of the alveolar type II (AT2) cell transcriptional state in lethal COVID-19 biopsies based on the work by Melms et al, 2021, in their landmark paper “A Molecular Single-Cell Lung Atlas of Lethal COVID-19” (https://www.nature.com/articles/s41586-021-03569-1) While this notebook reproduces and modifies the pre-processing steps on the raw data performed by the authors, the exploration, analysis, deductions and conclusions are independent and self-directed. The agentic Large Language Model Claude Sonnet 5.0 (Anthropic) was utilised for code debugging, statistical methodology discussions and literature dataset navigation. This work was performed in Python 3.13.4. All code and markdown entries were written by myself. 

The Dataset

The Melms et al, 2021 dataset consisted of lung biopsies extracted from live 7 healthy donors (not infected with COVID-19) and 19 deceased COVID-19 patients harvested post-mortem. Sequencing libraries were prepared from single nuclei suspensions. Only one biopsy was harvested per donor, making the terms “donor” and “sample” synonymous for this dataset. Cell Bender was implemented by the authors to remove contaminating ambient RNA and empty cells prior to the publication of the dataset.

Methodology 

Manual Pre-processing

The raw counts dataset (GSE171524) was downloaded from Gene Expression Omnibus (GEO) using the link provided in the supplementary materials of the publication. For exploratory data analysis, raw counts for control and COVID-19 donor samples were loaded into Scanpy as an Anndata object and transposed to enable navigation of the data in a cell x genes format. Prior to doublet detection via SCVI tools “SOLO” model simulating R programming language’s “Seurat v3” functionality, nuclei were filtered to exclude genes with minimum coverage of 10 cells and subsequently subsetted to include only the top 2000 highly variable genes (HVGs). These steps minimised the probability of the deep-learning generative “SOLO” model from training on empty, burst or damaged cells and reduced the dataset dimensions and computational processing required for training. Following doublet identification, labelling and exclusion, mitochondrial and ribosomal genes were annotated. Genes with a coverage less than 3 nuclei were excluded. Scanpy quality control metrics (QC Metrics) were calculated to identify outliers in violin plots of n_genes_by_count, total_counts, percentage mitochondrial and ribosomal counts (pct_counts_MT and pct_counts_ribo respectively). Nuclei with n_genes_by_count above the 98th percentile, with mitochondrial gene count proportions above 20% and ribosomal gene count proportions above 5% were excluded as outliers as these nuclei were likely to contain stressed, burst or dying cells.    

Automated Pre-processing
The manual pre-processing steps implemented during the exploratory data analysis of single samples were compiled into a function and applied to all 26 samples in an automated workflow for concatenation into a single Anndata object as a sparse matrix to prevent tedious conversion post-normalisation. Raw read counts data was preserved in the “counts” layer of the Anndata object prior to normalisation to 1000 counts and log transformation was performed to reconstitute the count integrity. Gene filters were applied to reduce the dataset dimensions and exclude genes with less than 100 nuclei coverage and the top 3000 highly variable genes were saved without subsetting to include non-highly variable genes in downstream differential gene expression analysis. HGVs were saved in the Anndata “counts” layer. Sample integration to remove technical artefacts and batch effects between donors was conducted using SCVI tools with “sample” used as the batch key and the percentage ribosomal and mitochondrial counts as well as total counts being included as covariates.

Clustering
Latent representation, normalised expression and connectivity distances were extracted from the SCVI integration model and utilised for cell type clustering with the uniform manifold approximation and projection (UMAP) algorithm. Cell cluster number label assignment to was performed using the Leiden algorithm. Cluster cell type marker assignment involved interrogation of significant (log2 fold change > 0.5 & p-value < 0.05) highly differentially expressed cluster genes, cross-referencing against Human Cell Atlas and PanglaoDB databases and validation against Cell Typist “Human Lung Atlas” cell type identification. Manually curated cluster label assignment was only updated against Cell Typist results if a complete mismatch in cell type and count existed. 25 Clusters were identified with multiple cell types distributed in more than one cluster

Single Cell Differential Expression
SCVI was employed to generate single cell differential expression using normalised counts from the previously trained model. Differential expression specific to alveolar type II (AT2) cells between healthy and COVID-19 conditions was extracted from the model, feature engineered to create a log2 fold change metric and filtered to select normalised mean counts with Bayes factor greater than 3 and log2foldchange greater than 0.5. 

Pseudobulk Analysis 
Pseudobulk profiles of differential gene expression were generated by building arrays of cell counts for each donor, integrating donor metadata, applying a minimum cell count threshold for donor and subsequently parsing this curated data into the Pydeseq2 pipeline for fitting onto the negative binomial distribution for statistical inference. Statistics were extracted from Pydeseq2, sorted by adjusted p-value (applying Benjamini-Hochberg Correction) and filtered for significance by screening for results with adjusted p-values less than 0.05 and log2 fold change greater than 0.5. Second layer Benjamini-Hochberg correction was performed when conducting pseudobulk interrogation across multiple cell types simultaneously to prevent exponential compounding error incursion.    

Gene Set Enrichment Analysis (GSEA)
GSEA was performed via GSEAPY and interrogated unfiltered pseudobulk results against MSigDB Hallmark 2020 gene ontology vocabularies in pre-rank assessments, utilised significantly differentially expressed gene lists in enrichr over-representation analysis (ORA) against KEGG 2021 Human vocabularies and Gene Ontology Biological Process 2021. Significant gene lists were also selected for reactome analysis using the Reactome 2022 vocabulary. Rankings were performed based on the Wald statistic generated by Pydeseq2, thus accounting for statistical power and effect magnitude.  

Partition-based Graphical Abstraction (PAGA)
PAGA graph maps were constructed using Scanpy and build using Leiden algorithm partitions and K-nearest neighbour connectivities generated previously for UMAP clustering. Threshold was set to 0.1 to display moderately strong connections at minimum. 
