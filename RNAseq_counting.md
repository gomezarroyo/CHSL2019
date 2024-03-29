# COUNTING AND DIFFERENTIAL GENE EXPRESSION

This uses data from HBR and UHR exercise from rnabio.org

NOTES
Choose a tool for counting. In this case we will use Stringtie because it can help you count isoforms as well (as opposed to Deseq or HTseq). 

1) StringTie will help you count but taking into account isoforms instead of just globally adding all transcript levels together. 
2)Some Basic stringie stuff. 
  - Merge all gene structures from all samples 
    - this makes a new reference file that will include transcripts with assembled and potentially novel transcripts
  - Then re-run against your de novo transcriptome from all samples -- this will help understand count levels of isoforms that may have not been found from sample to sample. 
  3) Then you will get total transcript levels plus isoform transcript levels. 
  
```
cd $RNA_HOME/
~/workspace/rnaseq

mkdir -p expression/stringtie/ref_only/

cd expression/stringtie/ref_only/
#you need to be in directory above!

stringtie -p 8 -G $RNA_REF_GTF -e -B -o HBR_Rep1/transcripts.gtf -A HBR_Rep1/gene_abundances.tsv $RNA_ALIGN_DIR/HBR_Rep1.bam

# -G use the reference GTF. if you don't use it then it will go ALL de novo
# -e explore only known isoforms. if you remove it it will try to find new from within the GTF
# -B ouput for BallGown
# -o output Path
# -a gene level abundance estimates

```
Access the file

```
less -S UHR_Rep1/transcripts.gtf column -l
#here you can find the TPM and FPKM

cut -f 3 $RNA_REF_GTF | sort | uniq -c

OUTPUT
 14888 CDS
     12 Selenocysteine
  26155 exon
   3109 five_prime_utr
   1318 gene
   1856 start_codon
   1613 stop_codon
   2872 three_prime_utr
   4472 transcript

now look for the amount o counts per transcript levels from the total file per sample). 

cut -f 3 */transcripts.gtf | sort | uniq -c

#View transcript records. grep the word transcript

grep -v "^#" UHR_Rep1/transcripts.gtf | grep -w "transcript" | column -t | less -S

#Limit the view to transcript records and their expression values (FPKM and TPM values)

awk '{if ($3=="transcript") print}' UHR_Rep1/transcripts.gtf | cut -f 1,4,9 | less 


```
Gene and transcript level expression values can also be viewed in these two files:
you need to access each sample folder

```
Look at UHR_Rep1
cd ~/workspace/rnaseq/expression/stringtie/ref_only/UHR_Rep1

column -t t_data.ctab | less -S
less -S -x20 gene_abundances.tsv

```
## Expression matrix!
Create a tidy expression matrix files for the StringTie results. This will be done at both the gene and transcript level and also will take into account the various expression measures produced: coverage, FPKM, and TPM.
This will use a script made by griffithlab/rnabio.org
```

cd $RNA_HOME/expression/stringtie/ref_only/
wget https://raw.githubusercontent.com/griffithlab/rnabio.org/master/assets/scripts/stringtie_expression_matrix.pl
chmod +x stringtie_expression_matrix.pl

#for TPM
./stringtie_expression_matrix.pl --expression_metric=TPM --result_dirs='HBR_Rep1,HBR_Rep2,HBR_Rep3,UHR_Rep1,UHR_Rep2,UHR_Rep3' --transcript_matrix_file=transcript_tpm_all_samples.tsv --gene_matrix_file=gene_tpm_all_samples.tsv

#for FPKM
./stringtie_expression_matrix.pl --expression_metric=FPKM --result_dirs='HBR_Rep1,HBR_Rep2,HBR_Rep3,UHR_Rep1,UHR_Rep2,UHR_Rep3' --transcript_matrix_file=transcript_fpkm_all_samples.tsv --gene_matrix_file=gene_fpkm_all_samples.tsv

#for gene coverage
./stringtie_expression_matrix.pl --expression_metric=Coverage --result_dirs='HBR_Rep1,HBR_Rep2,HBR_Rep3,UHR_Rep1,UHR_Rep2,UHR_Rep3' --transcript_matrix_file=transcript_coverage_all_samples.tsv --gene_matrix_file=gene_coverage_all_samples.tsv
```
access your pretty tables it!

```
column -t transcript_tpm_all_samples.tsv | less -S

column -t gene_tpm_all_samples.tsv | less -S

```

# EXERCISE

```
cd $RNA_HOME/practice/
pwd/home/ubuntu/workspace/rnaseq/practice

mkdir -p expression/stringtie/ref_only/
cd expression/stringtie/ref_only/

stringtie -p 8 -G $RNA_REF_GTF -e -B -o HCC1395_tumor_rep1/transcripts.gtf $RNA_HOME/practice/alignments/hisat2/HCC1395_tumor_rep1.bam

stringtie -p 8 -G $RNA_REF_GTF -e -B -o HCC1395_tumor_rep2/transcripts.gtf -A HCC1395_tumor_rep2/gene_abundances.tsv $RNA_HOME/practice/alignments/hisat2/HCC1395_tumor_rep2.bam

Do it on the merged files

stringtie -p 8 -G $RNA_REF_GTF -e -B -o HCC1395_normal/transcripts.gtf -A HCC1395_normal/gene_abundances.tsv $RNA_HOME/practice/alignments/hisat2/HCC1395_normal.bam

stringtie -p 8 -G $RNA_REF_GTF -e -B -o HCC1395_tumor/transcripts.gtf -A HCC1395_tumor/gene_abundances.tsv $RNA_HOME/practice/alignments/hisat2/HCC1395_tumor.bam

#this one has option A to spit out gene_abundances

```
# HTSeq Count: The RAW counts method

**NOT RECOMMENDED FOR TRANSCRIPT LEVEL BUT RATHER GENE LEVEL EXPRESSION ANALYSIS**

```

cd $RNA_HOME/
mkdir -p expression/htseq_counts
cd expression/htseq_counts

YOU MUST BE INSIDE HTSEQ COUNTS FOLDER

htseq-count --format bam --order pos --mode intersection-strict --stranded reverse --minaqual 1 --type exon --idattr gene_id $RNA_ALIGN_DIR/UHR_Rep1.bam $RNA_REF_GTF > UHR_Rep1_gene.tsv

#OUTPUT

REF_GTF > UHR_Rep1_gene.tsv
56295 GFF lines processed.
100000 SAM alignment record pairs processed.
200000 SAM alignment record pairs processed.
Warning: Mate records missing for 672 records; first such record: <SAM_Alignment object: Paired-end read 'HWI-ST718_146963544:5:1208:19054:7891' aligned to 22:[15798693,15798793)/+>.
227728 SAM alignment pairs processed.
```
check out the output. 
its is basically gene name and raw counts

```
head UHR_Rep1_gene.tsv
```
#OPTIONS to use

’–format’ specify the input file format one of BAM or SAM. Since we have BAM format files, select ‘bam’ for this option.
’–order’ provide the expected sort order of the input file. Previously we generated position sorted BAM files so use ‘pos’.
’–mode’ determines how to deal with reads that overlap more than one feature. We believe the ‘intersection-strict’ mode is best.
’–stranded’ specifies whether data is stranded or not. The TruSeq strand-specific RNA libraries suggest the ‘reverse’ option for this parameter.
’–minaqual’ will skip all reads with alignment quality lower than the given minimum value
’–type’ specifies the feature type (3rd column in GFF file) to be used. (default, suitable for RNA-Seq and Ensembl GTF files: exon)
’–idattr’ The feature ID used to identify the counts in the output table. The default, suitable for RNA-SEq and Ensembl GTF files, is gene_id.
```
RUN the rest!

htseq-count --format bam --order pos --mode intersection-strict --stranded reverse --minaqual 1 --type exon --idattr gene_id $RNA_ALIGN_DIR/UHR_Rep2.bam $RNA_REF_GTF > UHR_Rep2_gene.tsv
htseq-count --format bam --order pos --mode intersection-strict --stranded reverse --minaqual 1 --type exon --idattr gene_id $RNA_ALIGN_DIR/UHR_Rep3.bam $RNA_REF_GTF > UHR_Rep3_gene.tsv
htseq-count --format bam --order pos --mode intersection-strict --stranded reverse --minaqual 1 --type exon --idattr gene_id $RNA_ALIGN_DIR/HBR_Rep1.bam $RNA_REF_GTF > HBR_Rep1_gene.tsv
htseq-count --format bam --order pos --mode intersection-strict --stranded reverse --minaqual 1 --type exon --idattr gene_id $RNA_ALIGN_DIR/HBR_Rep2.bam $RNA_REF_GTF > HBR_Rep2_gene.tsv
htseq-count --format bam --order pos --mode intersection-strict --stranded reverse --minaqual 1 --type exon --idattr gene_id $RNA_ALIGN_DIR/HBR_Rep3.bam $RNA_REF_GTF > HBR_Rep3_gene.tsv
```
## Merge results files into a single matrix for use in edgeR. 
The following joins the results for each replicate together, adds a header, reformats the result as a tab delimited file, and shows you the first 10 lines of the resulting file

```
cd $RNA_HOME/expression/htseq_counts/
join UHR_Rep1_gene.tsv UHR_Rep2_gene.tsv | join - UHR_Rep3_gene.tsv | join - HBR_Rep1_gene.tsv | join - HBR_Rep2_gene.tsv | join - HBR_Rep3_gene.tsv > gene_read_counts_table_all.tsv
echo "GeneID UHR_Rep1 UHR_Rep2 UHR_Rep3 HBR_Rep1 HBR_Rep2 HBR_Rep3" > header.txt
cat header.txt gene_read_counts_table_all.tsv | grep -v "__" | awk -v OFS="\t" '$1=$1' > gene_read_counts_table_all_final.tsv
rm -f gene_read_counts_table_all.tsv header.txt
head gene_read_counts_table_all_final.tsv | column -t

#OPTIONS

-grep -v "__" is being used to filter out the summary lines at the end of the files that ht-seq count gives to summarize reads that had no feature, were ambiguous, did not align at all, did not align due to poor alignment quality, or the alignment was not unique.

-awk -v OFS="\t" '$1=$1' is using awk to replace the single space characters that were in the concatenated version of our header.txt and gene_read_counts_table_all.tsv with a tab character. -v is used to reset the variable OFS, which stands for Output Field Separator. By default, this is a single space. By specifying OFS="\t", we are telling awk to replace the single space with a tab. The '$1=$1' tells awk to reevaluate the input using the new output variable.
```

## Expression analysis TEST

ERCC expression analysis
Based on the above read counts, plot the linearity of the ERCC spike-in read counts versus the known concentration of the ERCC spike-in Mix. In this step we will first download a file describing the expected concentrations and fold-change differences for the ERCC spike-in reagent. Next we will use a Perl script to organize the ERCC expected values and our observed counts for each ERCC sequence. Finally, we will use an R script to produce an x-y scatter plot that compares the expected and observed values.

# TEAM EXERCISE

```

mkdir -p expression/stringtie/ref_only/
/home/ubuntu/workspace/rnaseq/team_exercise/expression/stringtie/ref_only

#THIS MUST HAPPEN IN THE ABOVE DIRECTORY

stringtie -p 8 -G $REFS/chr11_Homo_sapiens.GRCh38.95.gtf -e -B -o SRR10045016/transcripts.gtf -A SRR10045016/gene_abundances.tsv /home/ubuntu/workspace/rnaseq/team_exercise/alignments/hisat2/SRR10045016.bam
stringtie -p 8 -G $REFS/chr11_Homo_sapiens.GRCh38.95.gtf -e -B -o SRR10045017/transcripts.gtf -A SRR10045017/gene_abundances.tsv /home/ubuntu/workspace/rnaseq/team_exercise/alignments/hisat2/SRR10045017.bam
stringtie -p 8 -G $REFS/chr11_Homo_sapiens.GRCh38.95.gtf -e -B -o SRR10045018/transcripts.gtf -A SRR10045018/gene_abundances.tsv /home/ubuntu/workspace/rnaseq/team_exercise/alignments/hisat2/SRR10045018.bam
stringtie -p 8 -G $REFS/chr11_Homo_sapiens.GRCh38.95.gtf -e -B -o SRR10045019/transcripts.gtf -A SRR10045019/gene_abundances.tsv /home/ubuntu/workspace/rnaseq/team_exercise/alignments/hisat2/SRR10045019.bam
stringtie -p 8 -G $REFS/chr11_Homo_sapiens.GRCh38.95.gtf -e -B -o SRR10045020/transcripts.gtf -A SRR10045020/gene_abundances.tsv /home/ubuntu/workspace/rnaseq/team_exercise/alignments/hisat2/SRR10045020.bam
stringtie -p 8 -G $REFS/chr11_Homo_sapiens.GRCh38.95.gtf -e -B -o SRR10045021/transcripts.gtf -A SRR10045021/gene_abundances.tsv /home/ubuntu/workspace/rnaseq/team_exercise/alignments/hisat2/SRR10045021.bam

cd ~/workspace/rnaseq/team_exercise/expression/stringtie/ref_only

wget https://raw.githubusercontent.com/griffithlab/rnabio.org/master/assets/scripts/stringtie_expression_matrix.pl
chmod +x stringtie_expression_matrix.pl

#this part is only for this exercise

SRR10045016,SRR10045017,SRR10045018,SRR10045019,SRR10045020,SRR10045021

./stringtie_expression_matrix.pl --expression_metric=TPM --result_dirs='SRR10045016,SRR10045017,SRR10045018,SRR10045019,SRR10045020,SRR10045021' --transcript_matrix_file=transcript_tpm_all_samples.tsv --gene_matrix_file=gene_tpm_all_samples.tsv
./stringtie_expression_matrix.pl --expression_metric=FPKM --result_dirs='SRR10045016,SRR10045017,SRR10045018,SRR10045019,SRR10045020,SRR10045021' --transcript_matrix_file=transcript_fpkm_all_samples.tsv --gene_matrix_file=gene_fpkm_all_samples.tsv
./stringtie_expression_matrix.pl --expression_metric=Coverage --result_dirs='SRR10045016,SRR10045017,SRR10045018,SRR10045019,SRR10045020,SRR10045021' --transcript_matrix_file=transcript_coverage_all_samples.tsv --gene_matrix_file=gene_coverage_all_samples.tsv

```

Q1. Based on your stringtie results, what are the top 5 genes with highest average expression levels across all knockout samples? What about in your rescue samples? How large is the overlap between the two sets of genes? (Hint: You can use R for this analysis)

```
mkdir /home/ubuntu/workspace/rnaseq/team_exercise/de/ballgown/ref_only
cd /home/ubuntu/workspace/rnaseq/team_exercise/de/ballgown/ref_only 

#this will also be your working directory

###YOU MUST BE IN BALLGOWN DIR###

printf "\"ids\",\"type\",\"path\"\n\"SRR10045016\",\"KO\",\"~/workspace/rnaseq/team_exercise/expression/stringtie/ref_only/SRR10045016\"\n\"SRR10045017\",\"KO\",\"~/workspace/rnaseq/team_exercise/expression/stringtie/ref_only/SRR10045017\"\n\"SRR10045018\",\"KO\",\"~/workspace/rnaseq/team_exercise/expression/stringtie/ref_only/SRR10045018\"\n\"SRR10045019\",\"RQ\",\"~/workspace/rnaseq/team_exercise/expression/stringtie/ref_only/SRR10045019\"\n\"SRR10045020\",\"RQ\",\"~/workspace/rnaseq/team_exercise/expression/stringtie/ref_only/SRR10045020\"\n\"SRR10045021\",\"RQ\",\"~/workspace/rnaseq/team_exercise/expression/stringtie/ref_only/SRR10045021\"\n" > KO_vs_RQ.csv

cat KO_vs_RQ.csv

#this last file is necessary for Ballgown to work

R

bg
ballgown instance with 12774 transcripts and 6 samples


# Load all attributes including gene name
bg_table = texpr(bg, 'all')
bg_gene_names = unique(bg_table[, 9:10])

# Save the ballgown object to a file for later use
save(bg, file='bg.rda')

# Perform differential expression (DE) analysis with no filtering
results_transcripts = stattest(bg, feature="transcript", covariate="type", getFC=TRUE, meas="FPKM")
results_genes = stattest(bg, feature="gene", covariate="type", getFC=TRUE, meas="FPKM")
results_genes = merge(results_genes, bg_gene_names, by.x=c("id"), by.y=c("gene_id"))

              id feature        fc       pval      qval gene_name
1 ENSG00000002330    gene 0.7047020 0.03624227 0.3858446       BAD
2 ENSG00000005801    gene 1.9619699 0.18471402 0.6270660    ZNF195
3 ENSG00000006071    gene 0.9768149 0.40473517 0.8197058     ABCC8
4 ENSG00000006118    gene 0.8657977 0.76053495 0.9320573  TMEM132A
5 ENSG00000006534    gene 1.0367493 0.87831075 0.9827270   ALDH3B1
6 ENSG00000006611    gene 0.6284114 0.01009314 0.2899279     USH1C

size of results genes without filtering: 3285
size of results genes with filtering: 960 (after removing files with  low-abundance genes)

#this will export table to the folder in the instance
write.table(results_transcripts, "KO_vs_RQ_transcript_results_filtered.tsv", sep="\t", quote=FALSE, row.names = FALSE)
write.table(results_genes, "KO_vs_RQ_gene_results_filtered.tsv", sep="\t", quote=FALSE, row.names = FALSE)

Q:3 
514 transcripts that are significantly different
146 genes that are significantly different

write.table(sig_transcripts, "KO_vs_RQ_transcript_results_sig.tsv", sep="\t", quote=FALSE, row.names = FALSE)
write.table(sig_genes, "KO_vs_RQ_gene_results_sig.tsv", sep="\t", quote=FALSE, row.names = FALSE)

Q4. By referring back to the supplementary tutorial in the DE Visualization Module, can you construct a heatmap showcasing the significantly de genes?

 pdf(file="Team_Exercise_Chr11_KO_vs_RQ_Figures.pdf")
 
 178 differentially expressed genes (after sorting by pvalue <0.05 first 
 
 
 heatmap.2(y, hclustfun=myclust, distfun=mydist, na.rm = TRUE, scale="none", dendrogram="both", margins=c(10,4), Rowv=TRUE, Colv=TRUE, symbreaks=FALSE, key=TRUE, symkey=FALSE, density.info="none", trace="none", main=main_title, cexRow=0.3, cexCol=1, labRow=sig_gene_names_de,col=rev(heat.colors(75)))
 
 
 ```



