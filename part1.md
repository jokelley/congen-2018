RNAseq analysis methods
Prerequisites:
* Python 2.7
* R
* EdgeR (installation described below within R)
* HISAT2 v2.1.0 https://ccb.jhu.edu/software/hisat2/index.shtml
* StringTie https://ccb.jhu.edu/software/stringtie/index.shtml#install
* prepDE.py https://ccb.jhu.edu/software/stringtie/dl/prepDE.py
* Samtools http://www.htslib.org/download/

Worksheet: 
1.	We have RNAseq data from chr22 of the human genome. The reads have already been de-multiplexed, adapter trimmed, and quality trimmed. The necessary reference files (gtf and fasta) have already been downloaded and are available in the data folder. The files were downloaded from: 

Fasta file for chr22 is found on this page: http://hgdownload.cse.ucsc.edu/goldenPath/hg38/chromosomes/

Gtf file for whole genome is found here:
ftp://ftp.ensembl.org/pub/release-81/gtf/homo_sapiens

We want to limit to the genes found on chromosome 22, so I used this command to pull out those genes:
```
grep ^22 humangenome.gtf > chr22.gtf	
```
A bit more about gtf format:
http://www.ensembl.org/info/website/upload/gff.html

2.	Index the reference using hisat2 build (Note: hisat2 executables must be in your PATH or specify the entire path to the executable)
```
hisat2-build chr22.fa chr22
```
3.	Align the reads to the reference fasta using hisat2. Note: hisat2 must be in your PATH. 
First it is necessary to make the directory that all data will be written to, note: this directory only needs to be created once.
```
mkdir alignments 

hisat2 -f -x /Users/jlkelley/Dropbox/ConGen2018/ref/chr22 -1 data/sample_01_1.fasta -2 data/sample_01_2.fasta -S sample01
```
4.	Now that you have a bam file, you will want to check the mapping quality. Note: The output of samtools idxstats is tab-delimited with each line consisting of reference sequence name, sequence length, # mapped reads and # unmapped reads.

Use samtools view to convert the SAM file into a BAM file
```
cd alignments 
samtools view -bSh -o sample01.bam sample01.sam 
```
Use samtools sort to convert the BAM file to a sorted BAM file. The following command requires samtools version 1.2 or higher.
```
samtools sort -o sample01.sort.bam sample01.bam
samtools index sample01.sort.bam
samtools idxstats sample01.sort.bam
```
5.	Next use stringtie to generate a gtf for each sample for each gene in the reference annotation set on a per individual basis, this was an unstranded library preparation (it’s important to know how your data was generated), -e limits matches to the specified reference annotation file
```
stringtie alignments/sample01.sort.bam -G ref/chr22_genes.gtf -e > stringtie/sample01.gtf
```
6.	Repeat for all 20 samples. Let’s write a script that will do this for us! The shell script that works on my mac:
```
for i in 0{1..9} {10..20}
do
echo ${i}
hisat2 -f -x ref/chr22 -1 data/sample_${i}_1.fasta -2 data/sample_${i}_2.fasta -S alignments/sample${i}.sam
samtools view -bSh -o alignments/sample${i}.bam alignments/sample${i}.sam
samtools sort -o alignments/sample${i}.sort.bam alignments/sample${i}.bam
samtools index alignments/sample${i}.sort.bam
samtools idxstats alignments/sample${i}.sort.bam > alignments/sample${i}.stats
stringtie alignments/sample${i}.sort.bam -G ref/chr22_genes.gtf -e -B -o ballgown/${i}/sample${i}.gtf
done
```
At this point, you will have 20 bam files, one for each individual 
7.	We want to generate counts data for each individual. There are many ways to do this, we will use prepDE.py, which is a python script provided with stringtie.  
```
prepDE.py
```
8.	We’ll now move to R for the remainder of our analyses. (will be provided as part2)
