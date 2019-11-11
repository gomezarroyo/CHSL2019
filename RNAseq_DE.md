
# DIFFERENTIAL GENE EXPRESSION
We will be using R

First,

create a file that lists our 6 expression files, then view that file, then start an R session where we will examine these results:

```
cd $RNA_HOME/de/ballgown/ref_only/
/home/ubuntu/workspace/rnaseq/de/ballgown/ref_only

printf "\"ids\",\"type\",\"path\"\n\"UHR_Rep1\",\"UHR\",\"$RNA_HOME/expression/stringtie/ref_only/UHR_Rep1\"\n\"UHR_Rep2\",\"UHR\",\"$RNA_HOME/expression/stringtie/ref_only/UHR_Rep2\"\n\"UHR_Rep3\",\"UHR\",\"$RNA_HOME/expression/stringtie/ref_only/UHR_Rep3\"\n\"HBR_Rep1\",\"HBR\",\"$RNA_HOME/expression/stringtie/ref_only/HBR_Rep1\"\n\"HBR_Rep2\",\"HBR\",\"$RNA_HOME/expression/stringtie/ref_only/HBR_Rep2\"\n\"HBR_Rep3\",\"HBR\",\"$RNA_HOME/expression/stringtie/ref_only/HBR_Rep3\"\n" > UHR_vs_HBR.csv
cat UHR_vs_HBR.csv

this makes the UHR_vs_HBR.csv

```
## Start R and load your packages

```
R

library(ballgown)
library(genefilter)
library(dplyr)
library(devtools)

```
# Load phenotype data from a file we saved in the current working directory

```
pheno_data = read.csv("UHR_vs_HBR.csv")
```

# Load ballgown data structure and save it to a variable "bg"

```
bg = ballgown(samples=as.vector(pheno_data$path), pData=pheno_data)

#this makes a table for the covariates ids, type, path

```

# Load all attributes including gene name

```
bg_table = texpr(bg, 'all')
bg_gene_names = unique(bg_table[, 9:10])

#takes data from columns 9:10 then collapse towards only having uniques and then store in bg_gene_names

```
# Save the ballgown object to a file for later use

```
save(bg, file='bg.rda')
```
# Perform differential expression (DE) analysis with no filtering

```
results_transcripts = stattest(bg, feature="transcript", covariate="type", getFC=TRUE, meas="FPKM")
results_genes = stattest(bg, feature="gene", covariate="type", getFC=TRUE, meas="FPKM")
results_genes = merge(results_genes, bg_gene_names, by.x=c("id"), by.y=c("gene_id"))
```
# Save a tab delimited file for both the transcript and gene results
```
write.table(results_transcripts, "UHR_vs_HBR_transcript_results.tsv", sep="\t", quote=FALSE, row.names = FALSE)
write.table(results_genes, "UHR_vs_HBR_gene_results.tsv", sep="\t", quote=FALSE, row.names = FALSE)
```
**CRITICAL**
# Filter low-abundance genes. Here we remove all transcripts with a variance across the samples of less than one

```
bg_filt = subset (bg,"rowVars(texpr(bg)) > 1", genomesubset=TRUE)

```
# Load all attributes including gene name
bg_filt_table = texpr(bg_filt , 'all')
bg_filt_gene_names = unique(bg_filt_table[, 9:10])

# Perform DE analysis now using the filtered data
results_transcripts = stattest(bg_filt, feature="transcript", covariate="type", getFC=TRUE, meas="FPKM")
results_genes = stattest(bg_filt, feature="gene", covariate="type", getFC=TRUE, meas="FPKM")
results_genes = merge(results_genes, bg_filt_gene_names, by.x=c("id"), by.y=c("gene_id"))

# Output the filtered list of genes and transcripts and save to tab delimited files
write.table(results_transcripts, "UHR_vs_HBR_transcript_results_filtered.tsv", sep="\t", quote=FALSE, row.names = FALSE)
write.table(results_genes, "UHR_vs_HBR_gene_results_filtered.tsv", sep="\t", quote=FALSE, row.names = FALSE)

# Identify the significant genes with p-value < 0.05
sig_transcripts = subset(results_transcripts, results_transcripts$pval<0.05)
sig_genes = subset(results_genes, results_genes$pval<0.05)

# Output the signifant gene results to a pair of tab delimited files
write.table(sig_transcripts, "UHR_vs_HBR_transcript_results_sig.tsv", sep="\t", quote=FALSE, row.names = FALSE)
write.table(sig_genes, "UHR_vs_HBR_gene_results_sig.tsv", sep="\t", quote=FALSE, row.names = FALSE)

# Exit the R session
quit(save="no")

# LETs go back to Ubuntu termina

```
head UHR_vs_HBR_gene_results.tsv
What does the raw output from Ballgown look like?

head UHR_vs_HBR_gene_results.tsv
How many genes are there on this chromosome?

grep -v feature UHR_vs_HBR_gene_results.tsv | wc -l
How many passed filter in UHR or HBR?

grep -v feature UHR_vs_HBR_gene_results_filtered.tsv | wc -l

```
How many differentially expressed genes were found on this chromosome (p-value < 0.05)?
```

sort UHR_vs_HBR_gene_results_sig.tsv | uniq | wc -l

grep -v feature UHR_vs_HBR_gene_results_sig.tsv | wc -l


grep -v feature UHR_vs_HBR_gene_results_sig.tsv | sort -rnk 3 | head -n 20 | column -t
grep -v feature UHR_vs_HBR_gene_results_sig.tsv | sort -nk 3 | head -n 20 | column -t

```
Save all genes with P<0.05 to a new file.
```

grep -v feature UHR_vs_HBR_gene_results_sig.tsv | cut -f 6 | sed 's/\"//g' > DE_genes.txt






