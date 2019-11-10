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
