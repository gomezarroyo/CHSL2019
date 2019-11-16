# install packages and data using conda and ggd
this will also install ggd tool to be able to install/download all the good stuff

```
conda install minimap2 sniffles samtools gsort
ggd install grch37-reference-genome-ensembl-v1
ggd install grch37-hg002-pb-chr22-giab-v1
ggd install grch37-hg003-pb-chr22-giab-v1
ggd install grch37-hg004-pb-chr22-giab-v1
```

# create working directories

```
mkdir bams
mkdir vcfs
```

Use GGD, align using minimap pipe to samtools to convert to bam then pipe it to a folder
then use samtools to index the bam files 
then use sniffles for variant calling

```
#hg002
kid_fq="$(ggd get-files grch37-hg002-pb-chr22-giab-v1 -p "*fastq.gz")"
reference="$(ggd get-files grch37-reference-genome-ensembl-v1 -s 'Homo_sapiens' -g 'GRCh37' -p 'grch37-reference-genome-ensembl-v1.fa')"
minimap2 -t 4 --MD -ax map-pb $reference $kid_fq | samtools view -hSb | samtools sort > bams/chr22_HG002.grch37.bam
samtools index bams/chr22_HG002.grch37.bam
sniffles --genotype -t 4 -m bams/chr22_HG002.grch37.bam -v vcfs/chr22_HG002_unsort.grch37.vcf

```

make a script to repeat everything without having to copy paste each line

vim commands.sh
[type insert to add text, copy paste the text]

#hg003
dad_fq="$(ggd get-files grch37-hg003-pb-chr22-giab-v1 -p "*fastq.gz")"
reference="$(ggd get-files grch37-reference-genome-ensembl-v1 -s 'Homo_sapiens' -g 'GRCh37' -p 'grch37-reference-genome-ensembl-v1.fa')"
minimap2 -t 4 --MD -ax map-pb $reference $dad_fq | samtools view -hSb | samtools sort > bams/chr22_HG003.grch37.bam
samtools index bams/chr22_HG003.grch37.bam
sniffles --genotype -t 4 -m bams/chr22_HG003.grch37.bam -v vcfs/chr22_HG003_unsort.grch37.vcf

#hg004
mom_fq="$(ggd get-files grch37-hg004-pb-chr22-giab-v1 -p "*fastq.gz")"
reference="$(ggd get-files grch37-reference-genome-ensembl-v1 -s 'Homo_sapiens' -g 'GRCh37' -p 'grch37-reference-genome-ensembl-v1.fa')"
minimap2 -t 4 --MD -ax map-pb $reference $mom_fq | samtools view -hSb | samtools sort > bams/chr22_HG004.grch37.bam
samtools index bams/chr22_HG004.grch37.bam
sniffles --genotype -t 4 -m bams/chr22_HG004.grch37.bam -v vcfs/chr22_HG004_unsort.grch37.vcf

#execute the commands.sh
bash commands.sh


```
# VCF

```
#sanity check 

#access the vcf file

cat vcfs/chr22_HG003_unsort.grch37.vcf | grep -v "^#" | wc -l

#sort+bgzip, index, and merge
genome=https://raw.githubusercontent.com/jbelyeu/ggd-recipes/master/genomes/Homo_sapiens/GRCh37/GRCh37.genome
gsort vcfs/chr22_HG002_unsort.vcf $genome | bgzip -c > vcfs/chr22_HG002.grch37.vcf.gz
gsort vcfs/chr22_HG003_unsort.vcf $genome | bgzip -c > vcfs/chr22_HG003.grch37.vcf.gz
gsort vcfs/chr22_HG004_unsort.vcf $genome | bgzip -c > vcfs/chr22_HG004.grch37.vcf.gz

tabix vcfs/chr22_HG002.grch37.vcf.gz
tabix vcfs/chr22_HG003.grch37.vcf.gz
tabix vcfs/chr22_HG004.grch37.vcf.gz

#visualization example
samplot plot -c 22 -s 18057764 -e 18060464 -b bams/chr22_HG002.grch37.bam -t DEL -o del_hg002_22_18057764_18060464.png
@gomezarroyo
 
