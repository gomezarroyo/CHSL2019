(this code will not work, its only (mostly formatted notes)

# BASIC PROGRAMMING TIPS

```
ls /  #takes you the highest level directory
ls -lh #function
touch #command to make a file	

mv filename Directoryname/ . #to either move a file or rename
cp #copy file
cp -R #if you wanna copy directory
```
**BE CAREFUL about removing
```
rm -i #remove all 
rm -ir #R is to keep going within the directory

#Accessing txt file

nano
cat 
less 

echo 
#will help you add text to text file 
echo “add text” > filename. 

#use double dash to APPEND rather than OVERWRITE
```

COUNTING CHARACTERS 

```
Wc 
Wc -l #will show you size of the file in bytes
```

MATCHING LINES/FINDING IN FILES

```grep```

**Other basic grep functions

ignore case when matching (```-i```)
only match whole words (```-w```) if NOT it will do pattern recognition
show lines that don’t match a pattern (```-v```)
Use wildcard characters and other patterns to allow for alternatives (*, ., and [])

**Other Useful Commands

```scp``` for transferring files over SSH
```wget``` for downloading files from URL
```watch``` for monitoring
```xargs``` for complex inputs
```ps``` for process status
```top``` for running processes
```kill``` for stopping processes

**Other cool tips

Cut out the 3rd column of a tab-delimited text file and sort it to only show unique lines (i.e. remove duplicates):
 ```cut -f 3 file.txt | sort -u```

Count how many lines in a file contain the words ‘cat’ or ‘bat’ (-c option of grep counts lines):
 ```grep -c '[bc]at' file.txt```

Turn lower-case text into upper-case (using tr command to ‘transliterate’):
 ```cat file.txt | tr 'a-z' 'A-Z'```

# AWS

## SETTING UP YOUR INSTANCE

Need 8 cores, 32 G RAM, 250G space and a root with 50Gb
ALWAYS make sure you check protection for accidental protection. 

Configure a security group. SSH and HTTP. Protocol TCP. 

Once your create a PEM file you cannot download again. 

## CONNECTING TO YOUR INSTANCE

Download your keyfile (PEM)

```
chmod 400 cshl_2019_student.pem
ssh -i cshl_2019_student.pem ubuntu@IPaddress_yourinstance
#ssh is the command to start a shell
#if your connected then your prompt will change color. 

exit #to disconnect but will not terminate your instance! 

```
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



