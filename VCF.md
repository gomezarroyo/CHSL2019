# Variant calling analysis

Heterozygote == heterozygote alleles because one haplotype has a mutation but the other does not
Mendelian violation == de novo mutation that does not follow mendelian genetics

NONE OF THESE WILL WORK ON STRUCTURAL VARIANTS BECAUSE **GATK SOFTWARE** (THAT MAKES VCF FILES) WILL NOT FIND IT


## Exercise 
https://samtools.github.io/bcftools/howtos/index.html)

````
zcat platinum-exome.vcf.gz | grep -v “^#” | wc -l

Q1. 177,738

Q2. How many (biallelic) SNPs are in the file? (Hint: awk's "length" option or bcftools)

total snps
bcftools +counts platinum-exome.vcf.gz 
Number of SNPs:    162, 280


bcftools stats platinum-exome.vcf.gz | grep ^SN

SN	0	number of samples:	4
SN	0	number of records:	177675
SN	0	number of no-ALTs:	0
SN	0	number of SNPs:	162280
SN	0	number of MNPs:	4220
SN	0	number of indels:	12782
SN	0	number of others:	727
SN	0	number of multiallelic sites:	3371
SN	0	number of multiallelic SNP sites:	297

total biallelic snps

bcftools view -m2 -M2 -v snps platinum-exome.vcf.gz | wc -l
Number of balletic SNPS: 159,941

Q3. 
zcat platinum-exome.vcf.gz |  grep -v “^#” | awk '$4=="C" && $5=="T"' |wc -l

**do this again for all the types. then add them. 

22,955

Q4. 
How many transversions?
zcat platinum-exome.vcf.gz | awk '$4=="A" && $5=="C"' | wc -l

16,130

**do this again for all the types. then add them. 

```
you can also use:

```
bcftools stats platinum-exome.vcf.gz | grep ^TSTV
```

There is LOTS of information on the STATs command. 

```
bcftools stats platinum-exome.vcf.gz | grep -v “^#”

```

Q5. How many high confidence (QUAL >30) variants are there? 
How many high confidence (QUAL >30) variants are there (Hint: bcftools filter)?

```
zcat platinum-exome.vcf.gz | bcftools filter -i '%QUAL> 30' platinum-exome.vcf.gz | grep -v "^#" | wc -l

122046

q6. How many variants are present as a heterozygous genotype in 1 individual? (Hint: the AC attribute in VCF)
##INFO=<ID=AC,Number=A,Type=Integer,Description="Total number of alternate alleles in called genotypes">

zcat platinum-exome.vcf.gz | bcftools filter -i 'AC==1' platinum-exome.vcf.gz | grep -v "^#" | wc -l

40351

q7. Which chromosome has the most SNPs?

zcat platinum-exome.vcf.gz | bcftools query -i 'TYPE="snp"' | grep -v "^#" | cut -f1 | sort | uniq -c 

 17700 1
   7189 10
  10236 11
   9015 12
   3312 13
   6062 14
   6125 15
   8090 16
  10522 17
   2768 18
  13982 19
  11752 2
   4508 20
   2553 21
   4961 22
   8674 3
   6591 4
   7284 5
   9525 6
   8968 7
   6158 8
   7785 9
   3803 X
    112 Y

q8. Which chromosome has the highest density of SNPs?

zcat platinum-exome.vcf.gz | grep -v "^#" | cut -f1 | uniq -c | sort

bcftools query -i 'TYPE="snp"' -f'%CHROM\n' platinum-exome.vcf.gz \
  | grep -v "^#"  \
  | sort \
  | uniq -c \
  | awk '{print $2"\t"$1}' \
  > chrom.snp.counts
 
 #this makes a file with the chromosome counts. 
  
 #downlaod the chromsome info from UCSC
curl http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/hg19.chrom.sizes > hg19.chrom.sizes

sed -e 's/^chr//' hg19.chrom.sizes | sort > grch37.chrom.sizes

#sed - is the string editor. this will find (s) anytnig that starts with chr and replace with BLANK space. then pipe it to make file grch37.chrom.sizes

join -t $'\t' chrom.snp.counts grch37.chrom.sizes > chrom.snp.counts.and.chrom.sizes.txt


R...
 
q9. What do you guess for the sex of the four individuals? How would you go about this?
NA12878, NA12891, and NA12892 are a family. Who do you think are is the kid and who are the parents?

You can use IDENTITY BY STATE. 

A/A  vs A/A == IBS 2
A/G vs  A/A == IBS 1
A/A vs G/G == IBS 0 -- this you should never see unless there was sample swaps or not parental relationship. 



q10. How would you go about finding de novo mutations in the kid?


````
# NOMAD
You can always find out how rare a mutation cmpared to the general population is by looking into NOMAD database. 

# Using Gemini for variant analysis

```
gemini query -q "SELECT * FROM fakevariants" --header learnSQL2.db

gemini query -q "SELECT chrom,start,end FROM fakevariants --header WHERE in_dbsnp == 1" learnSQL2.db

gemini query -q "SELECT chrom,start,end FROM fakevariants --header WHERE chrom == chr1" learnSQL2.db

gemini.readthedocs.io 
  FIND: DATABASE SCHEMA
  
gemini db_info DATABASEname.db
gemini db_info trio.trim.vep.denovo.db

gemini query -q "SELECT chrom, start, end, ref, alt, is_coding, codon_change, gene, impact  FROM variants WHERE is_lof=1" --header trio.trim.vep.denovo.db | wc -l #this last part will count


gemini query -q "SELECT chrom, start, end, ref, alt, is_coding, codon_change, gene, impact, polyphen_pred  FROM variants WHERE is_lof=1" --header trio.trim.vep.denovo.db | column -t

gemini query -q "SELECT chrom, start, end, ref, alt, is_coding, codon_change, gene, impact, polyphen_pred  FROM variants WHERE is_lof=1 and qual>=100 and max_aaf_all< 0.0002941176471" --header trio.trim.vep.denovo.db | wc -l

#max_aaf_all: max Raw allele frequency (population independent) of the variant based on ExAC exomes (AF). calculated by incidence 1/3400 (example is CF patient)

##select commmand will select the columns you might need
###This will find variants with known LoF and pipe to wc to count the lines using wc -l 

gemini query -q "SELECT chrom, start, end, ref, alt, is_coding, codon_change, gene, impact, polyphen_pred  FROM variants WHERE is_lof=1 and qual>=100 and max_aaf_all< 0.0002941176471" --header trio.trim.vep.denovo.db | column - t | head

gemini query -q "SELECT chrom, start, end, ref, alt, is_coding, codon_change, gene, impact, polyphen_pred, in_exac FROM variants WHERE is_lof=1 and qual>=100 and max_aaf_all< 0.0002941176471" --header trio.trim.vep.denovo.db | column -t | head

```
# de_novo GEMINI tutorial

```
gemini load --cores 2 \
#             -v  trio.trim.vep.vcf.gz \
#             -t VEP \
#             --tempdir . \
#             --skip-gene-tables --skip-cadd --skip-gerp-bp \
#             -p denovo.ped \
#        trio.trim.vep.denovo.db

gemini de_novo --columns "chrom, start, end, ref, alt, filter, qual, gene, impact" trio.trim.vep.denovo.db | head | column -t | less

#find variations with at least 6 reads of coverage, quality >20
gemini de_novo --columns "chrom, start, end, ref, alt, filter, qual, gene, impact" \
-d 6 --min-gq 20 \
trio.trim.vep.denovo.db | wc -l

A: 618 variants

##remove the variants where GATK was NOT passed 

gemini de_novo --columns "chrom, start, end, ref, alt, filter, qual, gene, impact" \
-d 6 --min-gq 20 --filter "filter is NULL" \
trio.trim.vep.denovo.db | wc -l

A: 53

###HOWEVER if you are working with WES data then you need to remove the filters that were applied for STRAND bias given that they will be flagged but it does not mean that its an error

gemini de_novo --columns "chrom, start, end, ref, alt, filter, qual, gene, impact" \
-d 6 --min-gq 20 \
--filter "(filter is NULL or filter=='SBFilter')" \
trio.trim.vep.denovo.db | wc -l

A: 54

##add extra filter to find SEVERITY and allele frequency (here only .005 but in clinical labs 0.00001)

gemini de_novo --columns "chrom, start, end, ref, alt, filter, qual, gene, impact" \
-d 6 --min-gq 20 \
--filter "(filter is NULL or filter=='SBFilter') and impact_severity !='LOW' and max_aaf_all <= 0.005" \
trio.trim.vep.denovo.db | wc -l

A: 6 genes!

gemini de_novo --columns "chrom, start, end, ref, alt, filter, qual, gene, impact" \
-d 6 --min-gq 20 \
--filter "(filter is NULL or filter=='SBFilter') and impact_severity !='LOW' and max_aaf_all <= 0.005" \
trio.trim.vep.denovo.db








