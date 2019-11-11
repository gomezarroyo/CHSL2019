# Single cell sequencing

# Introduction
## Useful things to study before you start:
- Concepts of negative binomial distribution
- Concepts of data dimensionality reduction (PCA, tSNE vs UMAP)

## Tools (you could find in www.scrna-tools.org >> repository

1) cell ranger vs kallisto bustools vs star solo
2) loop browser (this will not help with cell death removal)
3) seurat vs monocle vs LIGER
4) RNAvelocity

## Some tips

cellRanger count will align and count. 
  it will use a reference genome and transcript annotations file
  software will index everything. 
  software will start associating the UMIs to read counts
  
you may need multi-core node with 12-16 CPU cores and 128GB memory if you use cell ranger
   
   .9% per 1000 cells will likely be doublets when sequencing. 
   
   you put in 10k cells to try to get 5-6k cells out to use for cell ranger. 
    this should give you enough data to work with but better coverage
    mean reads per cell should aim towards 50,000s per cell but you need to balance coverage vs money spent (you can resequence multiple times)
   
  always look at the fraction of reads per cell
   
 # Analysis Pipeline (once you get your counts table)
 
 **Useful code words**
 
 features = genes
 counts = UMIS
 barcodes = cells
 principal compnents = embeddings (seurat wording)
 
# Seurat Workflow

FIRST PART OF PIPELINE

1. Create Seurat object
2. QC by filtering out cells
3. SCT normalize each dataset
4. Integrate datasets

## Load your data

```
scrna.counts <- Read10X(data.dir = "###/yourpath/outs/filtered_feature_bc_matrix")

scrna <- CreateSeuratObject(counts = scrna.counts, min.cells = 10, min.features
= 100, project = Project1)

#also figure out if you need to do batch effect removal
# you can multiplex data loading -- see presenter slides
```

## Create your seurat object, this will store your data to work with
```
scrna <- CreateSeuratObject(counts = scrna.counts)

```
## Analyse ribosomal and mitochondrial genes

### Fisrt calculate percentage of mitochondrial genes
```
mito.genes <- grep(pattern = "^MT-", x = rownames(x = scrna), value = TRUE);
percent.mito <- Matrix::colSums(x = GetAssayData(object = scrna, slot = 'counts')[mito.genes, ]) /
Matrix::colSums(x = GetAssayData(object = scrna, slot = 'counts'));
scrna[['percent.mito']] <- percent.mito; # assign it to the meta data
```
### Then calculate percentage ribosomal genes
```
ribo.genes <- grep(pattern = "^RP[SL][[:digit:]]", x = rownames(x = scrna), value = TRUE);
percent.ribo <- Matrix::colSums(x = GetAssayData(object = scrna, slot = 'counts')[ribo.genes, ]) /
Matrix::colSums(x = GetAssayData(object = scrna, slot = 'counts'));
scrna[['percent.ribo']] <- percent.ribo; # assign it to the meta data
```
Plot your results

```
vln <- VlnPlot(object = scrna, features = c("percent.mito", "percent.ribo"), pt.size=0, ncol = 2,
group.by="Batch"); # make a violin plot, and color by batch or sample

vln <- VlnPlot(object = scrna, features = "nCount_RNA", pt.size=0, group.by="Batch", y.max=25000)
vln <- VlnPlot(object = scrna, features = "nFeature_RNA", pt.size=0, group.by="Batch")

```
## Filter your Data!
• Purpose: Remove cells with high mitochondrial content, too many genes, too few genes, etc
• Use the plots on the previous slides to choose appropriate thresholds for your data
• Some example thresholds:
• nFeature_RNA between 200 and 4000, or between 200 and 95th percentile (for
example)
• nFeature > x, nUMI < 93rd percentile
• Mitochondrial content below 5% or 10%
• Ribosomal content below 50%

```
scrna <- subset(x = scrna, subset = nFeature_RNA > 200 & nFeature_RNA < Feature95 & percent.mito <
mc.hi)

```
## Calculate cell cycle phase of each cell
Purpose: Cell cycle signature can dominate tSNE/UMAP plots and may need to be
removed
• Seurat has a built-in function that calculates relative expression level of G1/S and G2/M genes defined in Tirosh et al 2016

```
cell.cycle.tirosh <- read.table("CellCycleTirosh.txt", sep='\t', header=FALSE);
s.genes = cell.cycle.tirosh$V2[which(cell.cycle.tirosh$V1 == "G1/S")];
g2m.genes = cell.cycle.tirosh$V2[which(cell.cycle.tirosh$V1 == "G2/M")];

scrna <- CellCycleScoring(object=scrna, s.features=s.genes, g2m.features=g2m.gen
es, set.ident=FALSE)
```

## Normalize your data: Find variable genes, scale, and normalize: Goal is to remove technical effects while preserving biological variation

### Old version

```
scrna <- SCTransform(scrna) 

#Older versions of Seurat will use: 

scrna <- NormalizeData(object = scrna) 
scrna <- FindVariableFeatures(object = scrna) 
#FindVariableFeatures: use a variance stabilizing transformation

scrna <- ScaleData(object = scrna) 
#ScaleData: subtract mean, divide by standard deviation, remove unwanted signal
using multiple regression

```
### NEW version for NORMALIZATION: SCTransform function 
However! Now Seurat uses a different method for normalization called SCTransform function
Use negative binomial regression (with parameters derived from groups of genes
with similar expression) to remove impact of sequencing depth. 
```
# SCTransform (with removal of cell cycle signal):

scrna <- SCTransform(scrna, vars.to.regress = c("S.Score", "G2M.Score"), verbose=FALSE)
# Note that this replaces the three separate steps implemented in older workflows:

scrna <- NormalizeData(object = scrna, normalization.method = "LogNormalize", scale.factor = 1e4) # fea
ture counts divided by total, multiplied by scale factor, add 1, ln-transformed

scrna <- FindVariableFeatures(object = scrna, selection.method = 'vst', mean.cutoff = c(0.1,8), dispers
ion.cutoff = c(1, Inf)) # designed to find ~2000 variable genes

scrna <- ScaleData(object = scrna, features = rownames(x = scrna), vars.to.regress = c("S.Score","G2M.S
core"), display.progress=FALSE) # center and regress out unwanted variation

```
## Dimensionality reduction -- Principal Component Analysis

How to select your principal component

Calculate statistical significance of each PC
Randomly permute data
Recalculate PCs of randomized data
Compare “real” PCs to “random” PCs to derive
significance

Looking at the Elbow plot is useful. 

Ball park selection 

Number of PCs:
• Single samples: 5-10
• Multiple samples: 20-50
• Cellranger default: 10
• Partek default: 50

**Remember that "Minor components contain signal that is not represented in tSNE/UMAP**

```
# NB: jackstraw analysis is slow
scrna <- JackStraw(object = scrna, num.replicate = 100, dims=N) # specify N:
30, 50, etc
scrna <- ScoreJackStraw(object = scrna, dims = 1:50)
js <- JackStrawPlot(object = scrna, dims = 1:20) # make plot shown above
pc.pval <- scrna@reductions$pca@jackstraw@overall.p.values # get overall
pvalues for each PC

scrna <- RunPCA(object = scrna) 
scrna <- FindNeighbors(object = scrna)
scrna <- FindClusters(object = scrna)

```

5) Plots

```

# choose number of dimensions - experiment-specific!
n = 10; 

scrna <- RunUMAP(object = scrna, reduction = "pca", dims = 1:n) # calculate
UMAP

scrna <- RunTSNE(object = scrna, reduction = "pca", dims = 1:n) # calculate
tSNE
# make some plots:

DimPlot(object = scrna, reduction = "tsne", group.by = ”Batch", pt.size=0.1) #
color by Batch

FeaturePlot(object = scrna, features = c("nCount_RNA"), reduction=“tsne”) #
plot one or more genes or variable on t-SNE

scrna <- RunTSNE(object = scrna)
scrna <- RunUMAP(object = scrna)

DimPlot(object = scrna, reduction = "tsne")

UMAPPlot(object = scrna)

```
  

## Clustering

Common methods:
• Graph-based: Unsupervised, but “adjustable” using ”resolution” parameter
• Calculates k-nearest neighbors for each cell using Euclidian distance in PCA-space;
constructs shared nearest-neighbor (SNN) graph; optimize graph modularity (Waltman
and van Eck algorithm)
• K-means: Supervised: specify number of clusters. (available in **cellranger**, not Seurat)

```
nPC = 20; # specify number of PCs to use
cluster.res = 0.7; # specify resolution of clustering.

scrna <- FindNeighbors(object=scrna, dims=1:nPC);

scrna <- FindClusters(object=scrna, resolution=cluster.res);

scrna <- StashIdent(object = scrna, save.name = sprintf("ClusterNames_%.1f_%dPC",
cluster.res, nPC)) 

# save cluster names in a new identity (ClusterNames_0.7_20 in this
example) if desired
```

## Characterizing clusters using marker genes: Color cells by graph-based cluster

## Characterizing clusters using differential gene expression

Data has zero-inflated negative binomial distribution (lots of zeros, overdispersed) so can’t use bulk methods
Default in Seurat: Wilcoxon rank-sum test
Nonparametric version of t-test
For two clusters (A and B), and one gene, rank each cell in each cluster according to expression
Determine whether sum-of-ranks for cluster A is significantly different than sum-of-ranks for cluster B
Clear explanation of Wilcoxon rank-sum test:
http://statweb.stanford.edu/~susan/courses/s141/hononpara.pdf
Numerous other tests in Seurat and other packages

```
DEGs <- FindAllMarkers(object=scrna); # Compare each clusters to all
other cells. output is a matrix.
de.markers <- FindMarkers(scrna, ident.1 = ”1", ident.2 = ”2") #
compare identity 1 to identity 2
# NB: May first need to set default identities, e.g.:
Idents(object = scrna) <- ”seurat_clusters”; # sets the default
identity to Seurat_clusters
# Do the DEGs make sense? Plot them
FeaturePlot(object = scrna, features = ‘CD34’)
# Prettier version:
FeaturePlot(object = scrna, features = genesToPlot, cols =
c("gray","red"), ncol=2, reduction = "umap") +
theme(axis.title.x=element_blank(),axis.title.y=element_blank(),axis.
text.x=element_blank(),axis.text.y=element_blank(),axis.ticks.x=eleme
nt_blank(),axis.ticks.y=element_blank())
```

# Better approach? Cluster by PC then isolate genes with highly-weighted genes then calculate the average expression of each gene in each cluster or hierarchical clustering to cluster the clusters based on these genes

# Other questions to consider when comparing two samples/clusters
• Pairwise differential gene expression
• What fraction of cells in each sample express a given gene?
• Of the cells in each sample that express a given gene, does the mean expression in those
cells differ?
• Does the distribution of cell types differ between samples?
• Do the samples exhibit cell-type-specific differential gene expression?




```




   
