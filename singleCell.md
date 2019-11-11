# On using cellRanger

# Tools (you could find in www.scrna-tools.org >> repository

1) cell ranger vs kallisto bustools vs star solo
2) loop browser (this will not help with cell death removal)
3) seurat vs monocle vs LIGER
4) RNAvelocity

cellRanger count will align and count. 
  it will use a reference genome and transcript annotations file
  software will index everything. 
  software will start associating the UMIs to read counts
  
   multi-core node with 12-16 CPU cores and 128GB memory if you use cell ranger
   
   .9% per 1000 cells will likely be doublets when sequencing. 
   
   you put in 10k cells to try to get 5-6k cells out to use for cell ranger. 
    this should give you enough data to work with but better coverage
    mean reads per cell should aim towards 50,000s per cell but you need to balance coverage vs money spent (you can resequence multiple times)
   
   analysis from html file
   what is the sequencing saturation?  
   
   always look at the fraction of reads per cell
   
# Analysis Pipeline (once you get your counts table)
 
 code words
 
 features = genes
 counts = UMIS
 barcodes = cells
 
# Seurat Workflow

1) load your data

```
scrna.counts <- Read10X(data.dir = "###/yourpath/outs/filtered_feature_bc_matrix")

scrna <- CreateSeuratObject(counts = scrna.counts, min.cells = 10, min.features
= 100, project = Project1)

#also figure out if you need to do batch effect removal
# you can multiplex data loading -- see presenter slides


#analyse ribosomal and mitochondrial genes

# Calculate percentage of mitochondrial genes

mito.genes <- grep(pattern = "^MT-", x = rownames(x = scrna), value = TRUE);
percent.mito <- Matrix::colSums(x = GetAssayData(object = scrna, slot = 'counts')[mito.genes, ]) /
Matrix::colSums(x = GetAssayData(object = scrna, slot = 'counts'));
scrna[['percent.mito']] <- percent.mito; # assign it to the meta data

# ribosomal genes

ribo.genes <- grep(pattern = "^RP[SL][[:digit:]]", x = rownames(x = scrna), value = TRUE);
percent.ribo <- Matrix::colSums(x = GetAssayData(object = scrna, slot = 'counts')[ribo.genes, ]) /
Matrix::colSums(x = GetAssayData(object = scrna, slot = 'counts'));
scrna[['percent.ribo']] <- percent.ribo; # assign it to the meta data

vln <- VlnPlot(object = scrna, features = c("percent.mito", "percent.ribo"), pt.size=0, ncol = 2,
group.by="Batch"); # make a violin plot, and color by batch or sample

vln <- VlnPlot(object = scrna, features = "nCount_RNA", pt.size=0, group.by="Batch", y.max=25000)
vln <- VlnPlot(object = scrna, features = "nFeature_RNA", pt.size=0, group.by="Batch")

```
## Filter your Data
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

2) create your seurat object, this will store your data to work with
scrna <- CreateSeuratObject(counts = scrna.counts)

```

3) Find variable genes, scale, and normalize
```
scrna <- SCTransform(scrna) 

scrna <- NormalizeData(object = scrna) 
scrna <- FindVariableFeatures(object = scrna) 
scrna <- ScaleData(object = scrna) 
```
4) Dimensionality reduction -- Principal Component Analysis
```
scrna <- RunPCA(object = scrna) 
scrna <- FindNeighbors(object = scrna)
scrna <- FindClusters(object = scrna)
```
5) Plots
```
scrna <- RunTSNE(object = scrna)
scrna <- RunUMAP(object = scrna)
DimPlot(object = scrna, reduction = "tsne")
UMAPPlot(object = scrna)
```
  
   
