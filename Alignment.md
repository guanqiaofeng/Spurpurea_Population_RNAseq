# Find the best aligner

Based on paper "Simulation-based comprehensive benchmarking of RNA-seq aligners", to me, Novoalign, STAR and GSNAP are the best options. 

**Novoalign**
http://www.novocraft.com/documentation/novoalign-2/novoalign-user-guide/getting-started/

**STAR**
https://github.com/alexdobin/STAR

**STAR-fusion**
https://github.com/STAR-Fusion/STAR-Fusion/wiki

**GSNAP**
https://github.com/juliangehring/GMAP-GSNAP

1. Base-level precision and recall. These three programs are among the best performers with different genome complexity.
2. Junction-level precision and recall. In low and middle genome complexity, STAR is the best one, GSNAP has better precision and Novoaglin has better recall. In high genome complexity, Novoaglin perform best, GSNAP and STAR has similar performance. As junction performance is important for splicing analysis, STAR would be the better option.
3. Effect of parameters. Novoalgin default parameters could yield the best performance already. GSNAP parameter tuning would slightly increase performance and STAR parameter tuning could increase maybe ~10% performance for the high complexity genome. 
4. Runtime. From short to long, STAR, GSNAP, Novoalign. Roughly, GSNAP is ~2 times of STAR and Novoalign is ~8 times of STAR.

To summarize, **STAR** would be the best option for me, as it has good base-level and junction-level calling and short runtime. And I will need to tune parameters to achieve the best performance. 

# Getting started with STAR (Spliced Transcripts Alignment to a Reference)

Dobin A, Davis CA, Schlesinger F, Drenkow J, Zaleski C, Jha S, Batut P, Chaisson M, Gingeras TR (2013) STAR: ultrafast universal RNA-seq aligner. Bioinformatics 29: 15-21

Dobin A & Gingeras TR (2016). Optimizing RNA-Seq mapping with STAR. In Data Mining Techniques for the Life Sciences (pp. 245-262). Humana Press, New York, NY.

STAR can detect annotated and novel splice junctions, as well as chimeric and circular RNA. STAR-Fusion can detect fusion transcript. STAR take fastq files as input and output standard SAM/BAM files, as well as various other data files useful for downstream analyses such as transcript/gene expression quantification, differential gene expression, novel isoform reconstruction, signal visualization, etc. 

## 1. Materials
### 1.1 Hardware
System: 64-bit Unix, Linux, or Mac OS X
Maximun RAM: ~10 x GenomeSize bytes. For S. purpurea, ~5GB
Disk space: > 100GB recommended
Multithreaded: Yes --runThreadN <number-of-threads>. Typically, this number should be equal to the number of processor cores

### 1.2 Software
Download: https://github.com/alexdobin/STAR/releases
Discussion: https://groups.google.com/forum/#!forum/rna-star
Manual: https://github.com/alexdobin/ STAR /raw/master/doc/STARmanual.pdf

### 1.3 Input
RNA-seq: FASTQ/FASTA
Reference genome: FASTA
Annotation: GTF/GFF3

## Guidance for multi-sample 2-pass mapping
https://groups.google.com/forum/#!topic/rna-star/Cpsf-_rLK9I

## Methods
### Step 1. Genrating Genome indices
```sh
#!/bin/sh
#$ -V
#$ -cwd
#$ -S /bin/bash
#$ -N star_genome
#$ -q omni
#$ -pe sm 6
#$ -P quanah

STAR --runMode genomeGenerate \
     --genomeDir /lustre/scratch/gufeng/genome/Spur/187Zspecific/ \
     --genomeFastaFiles /lustre/scratch/gufeng/genome/Spur/187Zspecific/Spurpurea_519_v5.0_187Zspecific.fa \
     --sjdbGTFfile /lustre/scratch/gufeng/genome/Spur/187Zspecific/Spurpurea_519_v5.1.gene_exons_187Zspecific.gff3 \
     --sjdbGTFtagExonParentTranscript Parent \
     --sjdbOverhang 149 \
     --genomeSAindexNbases 13 \
     --runThreadN 6
```
\* Make sure genome sequence and reference sequence has the same name for chromosomes

\* For --sjdbGTFfile, STAR only process lines which has "exon" in the third column

\* Size of N-mers by default is 14

### Step 2. Mapping Reads to Genome (1st pass)
Multi-sample 2-Pass Mapping. The 1st pass serves to detect novel junctions, and in the 2nd pass, the detected junctions are added to the annotated junctions and all reads are re-mapped to finalize the alignment. It substantially increases the number of reads crossing the novel junctions, by allowing novel splices with shorter overhang. It adds sensitivity to unannotated splices. 

Run 1st mapping pass for all samples with for loop.
```sh
#!/bin/bash

for i in {1..92..1}
do
    qsub star1_M$i.sh
done
```
example code
```sh
#!/bin/sh
#$ -V
#$ -cwd
#$ -S /bin/bash
#$ -N F1
#$ -q omni
#$ -pe sm 8
#$ -P quanah

STAR --genomeDir /lustre/scratch/gufeng/genome/Spur/187Zspecific/ \
     --sjdbGTFfile /lustre/scratch/gufeng/genome/Spur/187Zspecific/Spurpurea_519_v5.1.gene_exons_187Zspecific.gff3 \
     --readFilesIn /lustre/scratch/gufeng/S_purpurea_RNA_seq/QC_Filtered_Raw_Data_process_split/10262.1.154566.ATTACTC-ATAGAGG_1.anqrpht.fastq /lustre/scratch/gufeng/S_purpurea_RNA_seq/QC_Filtered_Raw_Data_process_split/10262.1.154566.ATTACTC-ATAGAGG_2.anqrpht.fastq \
     --outFileNamePrefix F1. \
     --outFilterMismatchNoverReadLmax 0.05 \
     --outFilterMultimapNmax 5 \
     --alignIntronMin 19 \
     --alignIntronMax 20000 \
     --runThreadN 8

# minimum intron length in S. purpurea annotation is 20, maximum intron length in S. purpurea annotation is 16653
```
 Collapse the junctions (cat), remove non-canonical junctions ($5>0), remove junctions exists in less than two samples ($7>2), remove annotated junctions ($6==0). remove the duplicate ones after filtering (uniq)
```sh
cat *.tab | awk '($5 > 0 && $7 > 2 && $6==0)' | cut -f1-6 | sort | uniq > SJ.filtered.tab
```
### Step 3. Genrating Genome indices with the new junctions
```sh
#!/bin/sh
#$ -V
#$ -cwd
#$ -S /bin/bash
#$ -N star_genome
#$ -q omni
#$ -pe sm 6
#$ -P quanah


STAR --runMode genomeGenerate \
     --genomeDir /lustre/scratch/gufeng/genome/Spur/187Zspecific/SJ_Index/ \
     --genomeFastaFiles /lustre/scratch/gufeng/genome/Spur/187Zspecific/SJ_Index/Spurpurea_519_v5.0_187Zspecific.fa \
     --sjdbGTFfile /lustre/scratch/gufeng/genome/Spur/187Zspecific/SJ_Index/Spurpurea_519_v5.1.gene_exons_187Zspecific.gff3 \
     --sjdbGTFtagExonParentTranscript Parent \
     --sjdbOverhang 149 \
     --genomeSAindexNbases 13 \
     --sjdbFileChrStartEnd SJ.filtered.tab \
     --runThreadN 6
```

### Step 4. Mapping Reads to Genome (2nd pass)
Run 2nd mapping pass for all samples with for loop.
```sh
#!/bin/bash

for i in {1..90..1}
do
    qsub star1_F$i.sh
done
```
example code
```sh
#!/bin/sh
#$ -V
#$ -cwd
#$ -S /bin/bash
#$ -N M92
#$ -q omni
#$ -pe sm 8
#$ -P quanah

STAR --genomeDir /lustre/scratch/gufeng/genome/Spur/187Zspecific/SJ_Index/ \
     --sjdbGTFfile /lustre/scratch/gufeng/genome/Spur/187Zspecific/SJ_Index/Spurpurea_519_v5.1.gene_exons_187Zspecific.gff3 \
     --readFilesIn /lustre/scratch/gufeng/S_purpurea_RNA_seq/QC_Filtered_Raw_Data_process_split/11801.1.220153.ATAGCGG-ACCGCTA_1.anqrpht.fastq,/lustre/scratch/gufeng/S_purpurea_RNA_seq/QC_Filtered_Raw_Data_process_split/11801.1.220153.CCTCAGT-AACTGAG_1.anqrpht.fastq,/lustre/scratch/gufeng/S_purpurea_RNA_seq/QC_Filtered_Raw_Data_process_split/11801.3.220165.GAGCTCA-TTGAGCT_1.anqrpht.fastq /lustre/scratch/gufeng/S_purpurea_RNA_seq/QC_Filtered_Raw_Data_process_split/11801.1.220153.ATAGCGG-A
CCGCTA_2.anqrpht.fastq,/lustre/scratch/gufeng/S_purpurea_RNA_seq/QC_Filtered_Raw_Data_process_split/11801.1.220153.CCTCAGT-AACTGAG_2.anqrpht.fastq,/lustre/scratch/gufeng/S_purpurea_RNA_seq/QC_Filtered_Raw_Data_process_split/11801.3.220165.GAGCTCA-TTGAGCT_2.anqrpht.fastq \
     --outFileNamePrefix M92. \
     --outFilterMismatchNoverReadLmax 0.05 \
     --outFilterMultimapNmax 5 \
     --alignIntronMin 19 \
     --alignIntronMax 20000 \
     --outSAMtype BAM Unsorted \
     --runThreadN 8

```
