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

## Methods
### Step 1. Genrating Genome indices
```sh
STAR --runMode genomeGenerate --genomeDir /path/to/genome/directory/ --genomeFastaFiles /path/to/genome.fa --sjdbGTFfile path/to/annotation.gtf --runThreadN <number-of-threads/12>
```
\* Make sure genome sequence and reference sequence has the same name for chromosomes

\* For --sjdbGTFfile, STAR only process lines which has "exon" in the third column

\* Size of N-mers by default is 14

### Step 2. Mapping Reads to Genome
```sh
STAR --genomeDir /path/to/genome/directory/ --sjdbGTFfile /path/to/annotations.gtf --readFileIn path/to/read1.fastq path/to/read2.fastq --outSAMtype BAM SortedByCoordinate --outSAMattributes ALL --runThreadN <number-of-threads/12>
```
