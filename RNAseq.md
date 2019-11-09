# RNA seq

Things to study:
1) Docker
2) Python for iteration
3) Data design (see link in slides)

Notes:
1) Always remove duplicates for DNA seq data. You do not need to remove/mark duplicates in RNAseq data. 
2) How much library depth is needed?
  --Differential expression may be OK with lower reads per sample thus allowing more samples at the time. 
  --However if you are going for discovery experiments, then you may need deeper coverage. find other papers with similar goals. 
3) What mapping? Depends on lenght
  < 50 bp reads then OK to use BWA and a genome with a **junction database**
  >50 bp reads you will need spliced aligner such as bowtie/tophat, STAR, HISAT, etc. 
