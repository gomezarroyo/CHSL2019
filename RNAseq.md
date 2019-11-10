# RNA seq

Things to study:
1) Docker
2) Python for iteration
3) Data design (see link in slides)
4) Download ATOM
5) Download or look for bpipe software for paralallelyzing

Notes:
1) Always remove duplicates for DNA seq data. You do not need to remove/mark duplicates in RNAseq data. 
2) How much library depth is needed?
  --Differential expression may be OK with lower reads per sample thus allowing more samples at the time. 
  --However if you are going for discovery experiments, then you may need deeper coverage. find other papers with similar goals. 
3) What mapping? Depends on lenght
  - if < 50 bp reads then OK to use BWA and a genome with a **junction database**
  - if >50 bp reads you will need spliced aligner such as bowtie/tophat, STAR, HISAT, etc. 

##Set up environment

```
mkdir /$$$path/rnaseq

export RNA_HOME=~/$$$path/rnaseq
#this will make a shortcut called RNA_home

echo $RNA_HOME
```

# Add other environmental variables needed
This includes the software you need. 

```
export RNA_DATA_DIR=$RNA_HOME/data
export RNA_DATA_TRIM_DIR=$RNA_DATA_DIR/trimmed
export RNA_REFS_DIR=$RNA_HOME/refs

export RNA_REF_INDEX=$RNA_REFS_DIR/###IndexSAMPLEfile
export RNA_REF_FASTA=$RNA_REF_INDEX.fa
export RNA_REF_GTF=$RNA_REF_INDEX.gtf

export RNA_ALIGN_DIR=$RNA_HOME/alignments/hisat2
```
# Start up JAVA for the FASTQC software

```
export _JAVA_OPTIONS=-Djavax.accessibility.assistive_technologies=

```
# ACCESS YOUR PATH
## ON MODIFYING YOUR PATH
You can add different software to your PATH

```
nano ~/.bashrc

#alternatively

vi ~/.bashrc

```
## ACCESS YOUR BASHRC FILE
Once in nano you can edit your **bashrc** file

```
export LD_LIBRARY_PATH=$HOME/bin/htslib-1.9:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=~/bin/flexbar-3.4.0-linux:$LD_LIBRARY_PATH
export PATH=/home/ubuntu/bin/samblaster:$PATH
export PATH=/home/ubuntu/bin/sratoolkit.2.9.2-ubuntu64/bin:$PATH
export PATH=/home/ubuntu/bin/edirect:$PATH
export PATH=/home/ubuntu/bin/stringtie-1.3.4d.Linux_x86_64:$PATH
export PATH=/home/ubuntu/bin/bam-readcount/bin:$PATH
export PATH=/home/ubuntu/bin/hisat2-2.1.0:$PATH
export PATH=/home/ubuntu/bin/gffcompare-0.10.6.Linux_x86_64:$PATH
export PATH=/home/ubuntu/bin/htseq-release_0.11.0:$PATH
export PATH=/home/ubuntu/bin/tophat-2.1.1.Linux_x86_64:$PATH
export PATH=/home/ubuntu/bin/kallisto_linux-v0.44.0:$PATH
export PATH=/home/ubuntu/bin/FastQC:$PATH
export PATH=/home/ubuntu/bin/flexbar-3.4.0-linux:$PATH
export PATH=/home/ubuntu/bin/regtools/build:$PATH
export PATH=/home/ubuntu/.local/bin/:$PATH
export PATH=/home/ubuntu/bin/bedops_linux_x86_64-v2.4.35/bin:$PATH
export PATH=/home/ubuntu/bin/gtfToGenePred:$PATH
export PATH=/home/ubuntu/bin/MUMmer3.23:$PATH
export PATH=/home/ubuntu/bin/Sniffles-master/bin/sniffles-core-1.0.11:$PATH
export PATH=/home/ubuntu/bin/ngmlr-0.2.6:$PATH
export PATH=/home/ubuntu/bin/bwa:$PATH
export PATH=/home/ubuntu/bin/SURVIVOR/Debug:$PATH
export PATH=/home/ubuntu/bin/salmon-0.11.3-linux_x86_64/bin:$PATH
export PATH=/home/ubuntu/bin/bcftools-1.9:$PATH
export PATH=/home/ubuntu/anaconda/bin/:$PATH
export PATH=/home/ubuntu/bin/samtools-1.9:$PATH
export PICARD=/home/ubuntu/bin/picard.jar
export PATH=/home/ubuntu/bin/CellRanger/cellranger-3.1.0:$PATH
export PATH=/home/ubuntu/bin/STAR-2.7.3a/bin/Linux_x86_64_static:$PATH
```

# Add the new PATHS for your WORKING directories
These can change between experiments. **ALL THESE NEED TO BE CREATED STILL**

```
export RNA_HOME=~/workspace/rnaseq

export RNA_DATA_DIR=$RNA_HOME/data

#variable RNA_DATA_DIR means it will go to RNA_HOME/data folder even if it does not exist. you can later use it to make it with mkdir -p $RNA_DATA_DIR 

export RNA_DATA_TRIM_DIR=$RNA_DATA_DIR/trimmed

export RNA_REFS_DIR=$RNA_HOME/refs

export RNA_REF_INDEX=$RNA_REFS_DIR/chr22_with_ERCC92

export RNA_REF_FASTA=$RNA_REF_INDEX.fa

export RNA_REF_GTF=$RNA_REF_INDEX.gtf

export RNA_ALIGN_DIR=$RNA_HOME/alignments/hisat2

export _JAVA_OPTIONS=-Djavax.accessibility.assistive_technologies=

#to check if your bashrc file was edited you can look using 

env | grep "WHATEVER_WORD_YOU_WANT" ~/.bashrc

#this will access your environment first then pipe to use grep 

```

# FIXING your PATH
NOTE: If you are worried your .bashrc is messed up you can redownload as follows:

```
cd ~
wget -N https://raw.githubusercontent.com/griffithlab/rnabio.org/master/assets/setup/.bashrc
source ~/.bashrc
```

## TOOL INSTALLATION
(here we will only show SAMTOOLS but you need all of them!)

**REMEMBER SOME SOFTWARE WILL NEED COMPILING AND SOME WILL NEED YOU TO CHANGE PATH DIRECTORIES, SOME WILL NEED PYTHON OR OTHER INSTALLATION COMMANDS BESIDES WGET**

```
uname -m

cd $RNA_HOME
mkdir student_tools
cd student_tools

```
#SAMtools

```
cd $RNA_HOME/student_tools/

wget https://github.com/samtools/samtools/releases/download/1.9/samtools-1.9.tar.bz2
bunzip2 samtools-1.9.tar.bz2
tar -xvf samtools-1.9.tar

#check the extension for the unzipping if needed. in above file you need to BUNZIP then TAR

then you need to compile the source code into software here

cd /samtools-1.9

make 
#this command will start the compilation
```

Access the SAM tools! 

```
cd $RNA_HOME/student_tools/samtools-1.9

./samtools

#you can always find which tool your using by using the which command


```

# Add locally installed tools to your PATH [OPTIONAL]
To use the locally installed version of each tool without having to specify complete paths, you could add the install directory of each tool to your ‘$PATH’ variable

```
PATH=$RNA_HOME/student_tools/genePredToBed:$RNA_HOME/student_tools/gtfToGenePred:$RNA_HOME/student_tools/bedops_linux_x86_64-v2.4.35/bin:$RNA_HOME/student_tools/samtools-1.9:$RNA_HOME/student_tools/bam-readcount/bin:$RNA_HOME/student_tools/hisat2-2.1.0:$RNA_HOME/student_tools/stringtie-1.3.4d.Linux_x86_64:$RNA_HOME/student_tools/gffcompare-0.10.6.Linux_x86_64:$RNA_HOME/student_tools/htseq-release_0.11.0/scripts:$RNA_HOME/student_tools/tophat-2.1.1.Linux_x86_64:$RNA_HOME/student_tools/kallisto_linux-v0.44.0:$RNA_HOME/student_tools/FastQC:$RNA_HOME/student_tools/flexbar-3.4.0-linux:$RNA_HOME/student_tools/regtools/build:/home/ubuntu/bin/bedtools2/bin:$PATH

export LD_LIBRARY_PATH=$RNA_HOME/student_tools/flexbar-3.4.0-linux:$LD_LIBRARY_PATH

```

# DOCKER TOOLS: NEED TO REVIEW HOW TO INSTALL

Running docker tool example

```
docker pull biocontainers/samtools:v1.9-4-deb_cv1
docker run -t biocontainers/samtools:v1.9-4-deb_cv1 samtools --help
```

# Introduction to INPUTS

Always check the quality values read i.e sanger format vs old solexa or old illumina <1.8  version

1) Obtain a reference genome
example using GRCh38 version of the genome from Ensembl. limited to chr22

2) Obtain the annotations file. Access a specific gene
```
grep ENST00000342247 $RNA_REF_GTF | less -p "exon\s" -S
```
3) Index your refence genome. This is usually part of the aligner sofware. 

# YOU WILL NEED A TON OF RAM SO BE MINDFUL. 
You could also download/find indexed files already! 

You can extract features from the GTF file to be used to index the FASTA file. In this example we use python scripts

```
hisat2_extract_splice_sites.py $RNA_REF_GTF > $RNA_REFS_DIR/splicesites.tsv
hisat2_extract_exons.py $RNA_REF_GTF > $RNA_REFS_DIR/exons.tsv
```

Then you can go ahead an INDEX your reference genome

```
hisat2-build -p 8 --ss $RNA_REFS_DIR/splicesites.tsv --exon $RNA_REFS_DIR/exons.tsv $RNA_REF_FASTA $RNA_REF_INDEX
```

4) Download your datasets

```
echo $RNA_DATA_DIR
mkdir -p $RNA_DATA_DIR
cd $RNA_DATA_DIR
wget http://genomedata.org/rnaseq-tutorial/HBR_UHR_ERCC_ds_5pc.tar

#untar it
tar -xvf HBR_UHR_ERCC_ds_5pc.tar

then access it, you will need to use zcat command

zcat UHR_Rep1_ERCC-Mix1_Build37-ErccTranscripts-chr22.read1.fastq.gz | head -n 8

to count it
zcat UHR_Rep1_ERCC-Mix1_Build37-ErccTranscripts-chr22.read1.fastq.gz | grep -P "^\@HWI" | wc -l

```

5) Verify the quality of your FASTQ files

Try to run FastQC on your fastq files:
```
cd $RNA_HOME/data
fastqc *.fastq.gz
```

6) Trimm the things you don't need after QC file showed you what it is. 
This time we will use Flexbar

FIRST

Download necessary Illumina adapter sequence files.

`
echo $RNA_REFS_DIR
mkdir -p $RNA_REFS_DIR
cd $RNA_REFS_DIR
wget http://genomedata.org/rnaseq-tutorial/illumina_multiplex.fa
`

**7) ALIGNMENT (FINALLY!)**

```
find *.bam
#find me all the files that look like this
find *.bam -exec echo samtools index {} \;
find *.bam -exec echo samtools index {} \; | sh

docker run -v /home
```

#EXERCISE

#make your directory 
```
mkdir -p ~/workspace/rnaseq/team_exercise/references
```

#download your data
```
#download adapter sequences for trimming
wget -c http://genomedata.org/seq-tec-workshop/references/RNA/illumina_multiplex.fa

#reference FASTA corresponding to your team's assigned chromosome (e.g. chr11)
wget -c http://genomedata.org/seq-tec-workshop/references/RNA/chr11.fa

#download annotated reference for teams chromosome
wget -c http://genomedata.org/seq-tec-workshop/references/RNA/chr11_Homo_sapiens.GRCh38.95.gtf
```

q1. What are the different types of features contained in the gtf file (e.g. transcript, gene)? What are the frequencies of the different types of features? (This is referring to the third field/column of the data).

In order to get this answer, there are a series of commands that we can pipe together: 

```
cat chr11_Homo_sapiens.GRCh38.95.gtf | grep gene_name | cut -d$'\t' -f3 | sort | uniq -c | sort -r

75294 exon
  45290 CDS
  12774 transcript
  10313 five_prime_utr
   9268 three_prime_utr
   5902 start_codon
   5140 stop_codon
   3285 gene
      3 Selenocysteine

```

q2: Which genes have the highest number of transcripts (either gene id or gene name)? How many?
```

cat chr11_Homo_sapiens.GRCh38.95.gtf | grep -w transcript | cut -d$'\t' -f9 |  cut -d$';' -f1 | sort | uniq -c | sort -r | head -n 5

#cut first to cut out column 9
#then cut out 'between columns'
 then sort -- count uniq sequences -- then sort
 ```
 
 Output shoud look like 
  
```
     84 gene_id "ENSG00000007372"
     61 gene_id "ENSG00000166444"
     55 gene_id "ENSG00000006071"
     50 gene_id "ENSG00000110436"
     39 gene_id "ENSG00000255248"
```

Adaptor trimming 
**Flexbar basic usage**

```
#flexbar -r reads [-t target] [-b barcodes] [-a adapters] [options]

flexbar --adapter-min-overlap 7 --adapter-trim-end RIGHT --adapters ~/workspace/rnaseq/team_exercise/references/illumina_multiplex.fa --pre-trim-left 13 --max-uncalled 300 --min-read-length 25 --threads 8 --zip-output GZ --reads ~/workspace/rnaseq/team_exercise/data/SRR10045016_1.fastq.gz --reads2 ~/workspace/rnaseq/team_exercise/data/SRR10045016_2.fastq.gz --target ~/workspace/rnaseq/team_exercise/data/trimmed/SRR10045016

flexbar --adapter-min-overlap 7 --adapter-trim-end RIGHT --adapters ~/workspace/rnaseq/team_exercise/references/illumina_multiplex.fa --pre-trim-left 13 --max-uncalled 300 --min-read-length 25 --threads 8 --zip-output GZ --reads ~/workspace/rnaseq/team_exercise/data/SRR10045017_1.fastq.gz --reads2 ~/workspace/rnaseq/team_exercise/data/SRR10045017_2.fastq.gz --target ~/workspace/rnaseq/team_exercise/data/trimmed/SRR10045017

flexbar --adapter-min-overlap 7 --adapter-trim-end RIGHT --adapters ~/workspace/rnaseq/team_exercise/references/illumina_multiplex.fa --pre-trim-left 13 --max-uncalled 300 --min-read-length 25 --threads 8 --zip-output GZ --reads ~/workspace/rnaseq/team_exercise/data/SRR10045018_1.fastq.gz --reads2 ~/workspace/rnaseq/team_exercise/data/SRR10045018_2.fastq.gz --target ~/workspace/rnaseq/team_exercise/data/trimmed/SRR10045018

flexbar --adapter-min-overlap 7 --adapter-trim-end RIGHT --adapters ~/workspace/rnaseq/team_exercise/references/illumina_multiplex.fa --pre-trim-left 13 --max-uncalled 300 --min-read-length 25 --threads 8 --zip-output GZ --reads ~/workspace/rnaseq/team_exercise/data/SRR10045019_1.fastq.gz --reads2 ~/workspace/rnaseq/team_exercise/data/SRR10045019_2.fastq.gz --target ~/workspace/rnaseq/team_exercise/data/trimmed/SRR10045019

flexbar --adapter-min-overlap 7 --adapter-trim-end RIGHT --adapters ~/workspace/rnaseq/team_exercise/references/illumina_multiplex.fa --pre-trim-left 13 --max-uncalled 300 --min-read-length 25 --threads 8 --zip-output GZ --reads ~/workspace/rnaseq/team_exercise/data/SRR10045020_1.fastq.gz --reads2 ~/workspace/rnaseq/team_exercise/data/SRR10045020_2.fastq.gz --target ~/workspace/rnaseq/team_exercise/data/trimmed/SRR10045020

flexbar --adapter-min-overlap 7 --adapter-trim-end RIGHT --adapters ~/workspace/rnaseq/team_exercise/references/illumina_multiplex.fa --pre-trim-left 13 --max-uncalled 300 --min-read-length 25 --threads 8 --zip-output GZ --reads ~/workspace/rnaseq/team_exercise/data/SRR10045021_1.fastq.gz --reads2 ~/workspace/rnaseq/team_exercise/data/SRR10045021_2.fastq.gz --target ~/workspace/rnaseq/team_exercise/data/trimmed/SRR10045021

```
Perform a quality check before and after cleaning up your data
```
cd ~/workspace/rnaseq/team_exercise/data/trimmed/
fastqc *.fastq.gz

#make sure your fastqc is installed
```

# Aligment
## Index your REFERENCE genome

```
mkdir ~/workspace/rnaseq/team_exercise/alignments/hisat2
cd ~/workspace/rnaseq/team_exercise/alignments/hisat2

~/workspace/rnaseq/team_exercise/data/
~/workspace/rnaseq/team_exercise/references/

cd ~/workspace/rnaseq/team_exercise/references/

hisat2_extract_splice_sites.py ~/workspace/rnaseq/team_exercise/references/chr11_Homo_sapiens.GRCh38.95.gtf > ~/workspace/rnaseq/team_exercise/references/splicesites.tsv

hisat2_extract_exons.py ~/workspace/rnaseq/team_exercise/references/chr11_Homo_sapiens.GRCh38.95.gtf > ~/workspace/rnaseq/team_exercise/references/exons.tsv

hisat2-build -p 8 --ss ~/workspace/rnaseq/team_exercise/references/splicesites.tsv --exon ~/workspace/rnaseq/team_exercise/references/exons.tsv ~/workspace/rnaseq/team_exercise/references/chr11.fa ~/workspace/rnaseq/team_exercise/references/chr11

#the last part $RNA_REF_INDEX (in the rnabio.org exercise) means that every output file will be $RNA_REF_INDEX which in our case is (~/workspace/rnaseq/team_exercise/references/)chr11

```
#Alignment! (HISAT2)
```
mkdir ~/workspace/rnaseq/team_exercise/alignments/hisat2

make your path directories

export HISAT=~/workspace/rnaseq/team_exercise/alignments/hisat2
export REFS=/home/ubuntu/workspace/rnaseq/team_exercise/references

hisat2 -p 8 --rg-id=KO_S1 --rg SM:KO --rg LB:lib1 --rg PL:ILLUMINA --rg PU:CXX1234-ACTGAC.1 -x /home/ubuntu/workspace/rnaseq/team_exercise/references/chr11 --dta --rna-strandness RF -1 /home/ubuntu/workspace/rnaseq/team_exercise/data/trimmed/SRR10045016_1.fastq.gz -2 /home/ubuntu/workspace/rnaseq/team_exercise/data/trimmed/SRR10045016_2.fastq.gz -S $HISAT/SRR10045016.sam

hisat2 -p 8 --rg-id=KO_S2 --rg SM:KO --rg LB:lib1 --rg PL:ILLUMINA --rg PU:CXX1234-ACTGAC.1 -x /home/ubuntu/workspace/rnaseq/team_exercise/references/chr11 --dta --rna-strandness RF -1 /home/ubuntu/workspace/rnaseq/team_exercise/data/trimmed/SRR10045017_1.fastq.gz -2 /home/ubuntu/workspace/rnaseq/team_exercise/data/trimmed/SRR10045017_2.fastq.gz -S $HISAT/SRR10045017.sam

hisat2 -p 8 --rg-id=KO_S3 --rg SM:KO --rg LB:lib1 --rg PL:ILLUMINA --rg PU:CXX1234-ACTGAC.1 -x /home/ubuntu/workspace/rnaseq/team_exercise/references/chr11 --dta --rna-strandness RF -1 /home/ubuntu/workspace/rnaseq/team_exercise/data/trimmed/SRR10045018_1.fastq.gz -2 /home/ubuntu/workspace/rnaseq/team_exercise/data/trimmed/SRR10045018_2.fastq.gz -S $HISAT/SRR10045018.sam

hisat2 -p 8 --rg-id=RQ_S1 --rg SM:RQ --rg LB:lib1 --rg PL:ILLUMINA --rg PU:CXX1234-ACTGAC.1 -x /home/ubuntu/workspace/rnaseq/team_exercise/references/chr11 --dta --rna-strandness RF -1 /home/ubuntu/workspace/rnaseq/team_exercise/data/trimmed/SRR10045019_1.fastq.gz -2 /home/ubuntu/workspace/rnaseq/team_exercise/data/trimmed/SRR10045019_2.fastq.gz -S $HISAT/SRR10045019.sam

hisat2 -p 8 --rg-id=RQ_S2 --rg SM:RQ --rg LB:lib1 --rg PL:ILLUMINA --rg PU:CXX1234-ACTGAC.1 -x /home/ubuntu/workspace/rnaseq/team_exercise/references/chr11 --dta --rna-strandness RF -1 /home/ubuntu/workspace/rnaseq/team_exercise/data/trimmed/SRR10045020_1.fastq.gz -2 /home/ubuntu/workspace/rnaseq/team_exercise/data/trimmed/SRR10045020_2.fastq.gz -S $HISAT/SRR10045020.sam

hisat2 -p 8 --rg-id=RQ_S3 --rg SM:RQ --rg LB:lib1 --rg PL:ILLUMINA --rg PU:CXX1234-ACTGAC.1 -x /home/ubuntu/workspace/rnaseq/team_exercise/references/chr11 --dta --rna-strandness RF -1 /home/ubuntu/workspace/rnaseq/team_exercise/data/trimmed/SRR10045021_1.fastq.gz -2 /home/ubuntu/workspace/rnaseq/team_exercise/data/trimmed/SRR10045021_2.fastq.gz -S $HISAT/SRR10045021.sam

```
#Convert from BAM to SAM

```
samtools sort -@ 8 -o SRR10045016.bam SRR10045016.sam
samtools sort -@ 8 -o SRR10045017.bam SRR10045017.sam
samtools sort -@ 8 -o SRR10045018.bam SRR10045018.sam
samtools sort -@ 8 -o SRR10045019.bam SRR10045019.sam
samtools sort -@ 8 -o SRR10045020.bam SRR10045020.sam
samtools sort -@ 8 -o SRR10045021.bam SRR10045021.sam
```

#Merge HISAT2 BAM files
use picard tools

```
cd $HISAT
/home/ubuntu/workspace/rnaseq/team_exercise/alignments/hisat2

java -Xmx2g -jar $RNA_HOME/student_tools/picard.jar MergeSamFiles OUTPUT=KO.bam INPUT=SRR10045016.bam INPUT=SRR10045017.bam INPUT=SRR10045018.bam

java -Xmx2g -jar $RNA_HOME/student_tools/picard.jar MergeSamFiles OUTPUT=RQ.bam INPUT=SRR10045019.bam INPUT=SRR10045020.bam INPUT=SRR10045021.bam

#Here I kept $RNA_HOME cos it has all the student tools we installed earlier

```
Figure out how much they aligned

```
$RNA_HOME/student_tools/samtools-1.9/samtools flagstat FILENAME.bam

#Here I kept $RNA_HOME cos it has all the student tools we installed earlier

You could have done it as a loop too

find *.bam -exec echo samtools flagstat {} \; | sh

```
***INDEX YOUR BAM FILES***

```

cd $HISAT
~/workspace/rnaseq/team_exercise/alignments/hisat2

samtools index RQ.bam

or loop it

find *.bam -exec echo samtools index {} \; | sh

```
you can also try a docker tool!!!

```
docker run -v /workspace/rnaseq/team_exercise/alignments/hisat2:/workspace biocontainers/samtools:v1.9-4-deb_cv1 samtools index /workspace/KO.bam

1) set your workspace equivalent (/workspace/rnaseq/team_exercise/alignments/hisat2) 
2) find your docker hub in this case its from biocontainers , thus biocontainers/samtools 
3) SOMETIMES you need find your tag in this case samtools needs one and it was (:v1.9-4-deb_cv1) which is the version the tools in the docker container
4) type your tool name and the command (samtools index) and your sample name (KO.bam)

```
Visualize in IGV

Load your IP address on browser
Then find your BAM file and copy link address
Go to IGV >> file >> load from URL




