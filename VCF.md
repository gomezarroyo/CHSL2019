#Variant calling analysis

Heterozygote == heterozygote alleles because one haplotype has a mutation but the other does not
Mendelian violation == de novo mutation that does not follow mendelian genetics


# Exercise 

````
zcat platinum-exome.vcf.gz | grep -v “^#” | wc -l

Q1. 177738

Q2. How many (biallelic) SNPs are in the file? (Hint: awk's "length" option or bcftools)

total snps
bcftools +counts platinum-exome.vcf.gz 
Number of SNPs:    162, 280

total biallelic snps

bcftools view -m2 -M2 -v snps platinum-exome.vcf.gz | wc -l

Number of balletic SNPS: 159,941

Q3. 
zcat platinum-exome.vcf.gz | awk '$4=="C" && $5=="T"' | wc -l

22,955

Q4. 
How many transversions?
zcat platinum-exome.vcf.gz | awk '$4=="A" && $5=="C"' | wc -l

16,130

Q5. How many high confidence (QUAL >30) variants are there? 

