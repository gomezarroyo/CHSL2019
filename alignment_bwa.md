# SEQUENCE ALIGNMENT

Download/Upload your FASTQ files and reference sequence

Index your ref sequence. We will use BWA
You need to make BWA accessible to your $PATH

```
bwa index refsequence.fa
#we are using bwa aligner tool

```

## Setup an environment variable to point to the Picard executable file. 

```
export PICARD=/home/ubuntu/bin/picard.jar
```

## Alignment

```
cd /workspace/dna_alignment_lab/alignment_results

bwa mem -t 8 -o /workspace/dna_alignment_lab/alignment_results/NAME_of_yourSAMFILE.sam /workspace/dna_alignment_lab/reference_sequences/refseq.fa /workspace/dna_alignment_lab/fastq_files/
fastafiles_R1.fastq.gz /workspace/dna_alignment_lab/fastq_files/astafiles_R1.fastq.gz

# -t 8 is the number of threads
```
## convert SAM to BAM. you will need to install samtools

```
samtools view -@ 8 -h -b -o HCC1395_Exome_chr21.bam HCC1395_Exome_chr21.sam
```
## Query name sort bam files. this is needed for duplicate making. 
**you need to name sort to better identify duplicate sequences

```
java -Xmx60g -jar /home/ubuntu/bin/picard.jar SortSam I=HCC1395_Exome_chr21.bam O=HCC1395_Exome_chr21_namesorted_picard.bam SO=queryname

java #find java
-Xmx60g #asign memory
-jar /home/ubuntu/bin/picard.jar #find the directory

```
## Sort out the **duplicated sequences** from the previously processed file

```
#we will use the MarkDuplicates picard tool

java -Xmx60g -jar /home/ubuntu/bin/picard.jar #this start java picard

MarkDuplicates I=NAME_namesorted_picard.bam  O=NAME_namesorted_picard_mrkdup.bam 
ASSUME_SORT_ORDER=queryname 
METRICS_FILE= NAME_mrk_dup_metrics.txt 
QUIET=true 
COMPRESSION_LEVEL=0 
VALIDATION_STRINGENCY=LENIENT

#all options need to be in a single line! 

#print data to check on duplication metrics

```

## "Position" sort the BAM file. This will be needed for indexing of our BAM files

```
java -Xmx60g -jar /home/ubuntu/bin/picard.jar SortSam I=HCC1395_Exome_chr21_namesorted_picard_mrkdup.bam O=HCC1395_Exome_chr21_pos_sorted_mrkdup_picard.bam SO=coordinate
```

## Index your BAM file.
We will be using the BuildBamIndex tool from Picard tools

```
java -Xmx60g -jar /home/ubuntu/bin/picard.jar BuildBamIndex I=HCC1395_Exome_chr21_pos_sorted_mrkdup_picard.bam
```

## Assess quality of your alignment. Fast summary is to use flagstat
***You need to install samtools

```
samtools flagstat HCC1395_Exome_chr21_pos_sorted_mrkdup_picard.bam > HCC1395_Exome_chr21_pos_sorted_mrkdup_picard_flagstat.flagstat

#to print your flagstat head -n 20 *.flagstat

1787552 + 0 properly paired (97.96% : N/A) -- this will be the number you want. 

```

## FINAL WORKING MATERIAL
### Clean up the un-needed BAM/SAM files 
(optional)

***At the end YOU ONLY NEED: 

mrk_dup_metrics.txt
pos_sorted_mrkdup_picard.bai #position sorted, index BAM file
pos_sorted_mrkdup_picard.bam #position sorted, NOT-index BAM files
pos_sorted_mrkdup_picard_flagstat.flagstat #flag-based quality stats for your aligntment. 
