# gDNA SEQUENCE ALIGNMENT

## First steps

First you need to download your FASTQ files and reference sequence to the appropriate directories

```
mkdir ref #download your reference file here
mkdir fqfiles#downlaod your FASTQ files here

```

## Index your ref sequence 
Use BWA aligner tool
**You need to make BWA accessible to your $PATH**

```
cd ref
bwa index $$$REFSEQFILE.fa

#we are using bwa aligner tool

```

## Setup an environment variable to point to the Picard executable file. 

```
export PICARD=/home/ubuntu/bin/picard.jar
```

## Alignment of your sample FASTQ files. 
The OUTPUT of this command will be a SAM file (don't worry you will transform to BAM later)

```
#you need to be in the results folder

cd /workspace/results_folder

#REMEMBER everything goes into one line. it is only split to easy copy/paste

bwa mem -t 8 -o /workspace/results_folder/$$$SAMPLE_NAME.sam 
/$$$path/REFERENCE_FOLDER/REFSEQFILE.fa 
/$$$path/FASTQFILE_FOLDER sampleFASTQ_R1.fastq.gz 
/$$$path/FASTQFILE_FOLDER sampleFASTQ_R2.fastq.gz

# -t 8 is the number of threads

```

## Convert SAM to BAM. 
**You will need to install samtools**

```
cd /workspace/results_folder

samtools sort -@ 8 -h -b -o $$$SAMPLE_NAME.bam $$$SAMPLE_NAME.sam

#note that you select your BAM file first then SAM file

```
## "Name sort" sample BAM files. 
this is needed for duplicate selection. 
**you need to name sort to better identify duplicate sequences**not 

```
java -Xmx60g -jar /home/ubuntu/bin/picard.jar SortSam I=$$$SAMPLE_NAME.bam O=$$$SAMPLE_NAME_namesorted_picard.bam SO=queryname

#notice that you add _namesorted_picard to the name of file. this is not necessary but it is useful to identify your files later on. similarly you can opt to remove the picard part! 

```
**NOTE:** The command above java -Xmx60g -jar /home/ubuntu/bin/picard.jar means:
java #call java
-Xmx60g #asign memory to the program
-jar /home/ubuntu/bin/picard.jar #find the directory were PICARD tools are

## Mark **duplicated sequences** from the previously name_sorted BAM file
we will use the MarkDuplicates tool

```

java -Xmx60g -jar /home/ubuntu/bin/picard.jar MarkDuplicates I=$$$SAMPLE_NAME_namesorted_picard.bam  O=$$$SAMPLE_NAME_namesorted_picard_MRKDUP.bam ASSUME_SORT_ORDER=queryname METRICS_FILE=$$$SAMPLE_NAME_MRKDUP_metrics.txt QUIET=true COMPRESSION_LEVEL=0 VALIDATION_STRINGENCY=LENIENT

#note that you add _mrkdup to the name meaning this file contains marked duplicates
#DONT FORGET TO CHANGE THE NAME TO THE METRICS_FILE

```
You can print data to check on duplication metrics

```
head -n 20 $$$SAMPLE_NAME_MRKDUP_metrics.txt
```
## "Position" sort the sample BAM file. 
This will be needed for indexing of our sample BAM files. You will 'sort by position' the previously 'sorted by name' BAM files

```
java -Xmx60g -jar /home/ubuntu/bin/picard.jar SortSam I=$$$SAMPLE_NAME_namesorted_picard_MRKDUP.bam O=$$$SAMPLE_NAME_pos_sorted_picard_MRKDUP.bam SO=coordinate
```

## Index your sample BAM file.
We will be using the BuildBamIndex tool from Picard tools

```
java -Xmx60g -jar /home/ubuntu/bin/picard.jar BuildBamIndex I=$$$SAMPLE_NAME_pos_sorted_picard_MRKDUP.bam
```

#GREAT! YOU DID IT! (I hope) 🤞 

## Assess quality of your alignment. 
This fast summary will use flagstat. you will use the last file you created (position sorted, duplicates marked up INDEXED sample BAM file)
**You need to install samtools**

```
samtools flagstat $$$SAMPLE_NAME_pos_sorted_picard_MRKDUP.bam > $$$SAMPLE_NAME_pos_sorted_picard_MRKDUP_flagstat.flagstat

#NOTE THAT YOU WILL USE YOUR BAM FILE NOT BAI
#this will make a file with *.flagstat extension

#to print your flagstat

head -n 20 *.flagstat

#this will spit something that will include something like: 1787552 + 0 properly paired (97.96% : N/A) 
-- this will be the number you want. 

```

## FINAL WORKING MATERIAL
### Clean up the un-needed BAM/SAM files 

**At the end YOU ONLY NEED:** 

1) mrk_dup_metrics.txt
2) $$$SAMPLE_NAME_pos_sorted_picard_MRKDUP.bai position sorted **which is your positioned sorted, marked duplicates index BAM file**
3) $$$SAMPLE_NAME_pos_sorted_picard_MRKDUP.bam **which is your positioned sorted, marked duplicates NOT-index BAM file**
$) $$$SAMPLE_NAME_pos_sorted_picard_MRKDUP_flagstat.flagstat **which is the FLAG-based quality stats file for your aligntment.** 

#GRACIAS! GOOD luck! 
