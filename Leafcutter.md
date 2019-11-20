# LeafCutter pipeline

## Step1. Converting bam files to junc files (working directory /Junc)
code used (privided by LeafCutter): scripts/bam2junc.sh
```
#!/bin/bash

leafCutterDir='/home/gufeng/leafcutter' ## use only if you don't have scripts folder in your path
bamfile=$1
bedfile=$1.bed
juncfile=$2


if [ ! -z "$leafCutterDir" ]
then
	samtools view $bamfile | python $leafCutterDir/scripts/filter_cs.py | $leafCutterDir/scripts/sam2bed.pl --use-RNA-strand - $bed
file
	$leafCutterDir/scripts/bed2junc.pl $bedfile $juncfile
else
	if ! which filter_cs.py>/dev/null
	then 
		echo "ERROR:"
		echo "Add 'scripts' forlder to your path or set leafCutterDir variable in $0"
		exit 1
	fi
	samtools view $bamfile | filter_cs.py | sam2bed.pl --use-RNA-strand - $bedfile
	bed2junc.pl $bedfile $juncfile
fi
rm $bedfile
```

To run this step:
1. prepare job_submit.sh
```
for bamfile in `ls /lustre/scratch/gufeng/S_purpurea_RNA_seq/STAR2/*.bam`
do
    echo Converting $bamfile to $bamfile.junc
    sh /home/gufeng/leafcutter/scripts/bam2junc.sh $bamfile $bamfile.junc
    echo $bamfile.junc >> test_juncfiles.txt
done
```
2. submit job_submit.sh with qsub.sh
```
#!/bin/sh
#$ -V
#$ -cwd
#$ -S /bin/bash
#$ -N F1
#$ -q omni
#$ -pe sm 8
#$ -P quanah

sh job_submit.sh
```

Two output file types:
1. F/M*.Aligned.out.bam.junc (It includes the junctions identified in each bam file)
```
$ head F1.Aligned.out.bam.junc
Chr04	15444928	15445161	.	99	+
Chr10	4353258	4353365	.	110	-
Chr03	3732023	3733534	.	16	-
Chr19	13926926	13927016	.	1	-
Chr03	12926146	12926761	.	23	-
Chr01	5097398	5097573	.	279	-
Chr16	17597293	17597621	.	41	+
Chr05	1440339	1441375	.	6	+
Chr02	4692501	4692639	.	13	-
Chr02	258962	259041	.	12	+
```
2. test_juncfiles.txt (It includes the path to all the .junc files)
```
$ head test_juncfiles.txt
/lustre/scratch/gufeng/S_purpurea_RNA_seq/STAR2/F10.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/STAR2/F11.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/STAR2/F12.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/STAR2/F13.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/STAR2/F14.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/STAR2/F15.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/STAR2/F16.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/STAR2/F17.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/STAR2/F18.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/STAR2/F19.Aligned.out.bam.junc
```
The above two types of files are all in directory /STAR2, will be moved into /Junc, and update test_juncfiles.txt to test_juncfiles00.txt
```
$ head test_juncfiles00.txt
/lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/F10.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/F11.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/F12.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/F13.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/F14.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/F15.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/F16.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/F17.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/F18.Aligned.out.bam.junc
/lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/F19.Aligned.out.bam.junc
```

## Step2. Identify intron clustering (working directory /Junc)
code used (provided by LeafCutter): clustering/leafcutter_cluster.py

1. submit job with qsub2.sh
```
#!/bin/sh
#$ -V
#$ -cwd
#$ -S /bin/bash
#$ -N intron_clustering
#$ -q omni
#$ -pe sm 8
#$ -P quanah

python /home/gufeng/leafcutter/clustering/leafcutter_cluster.py -j test_juncfiles00.txt -m 50 -o SP -l 20000
# -m 50: each cluster need to be supported by >= 50 junction reads
# -l 20000: maximum intron 20000
# -o SP: output prefix "SP"
```

Output file: SP_perind_numers.counts.gz
```
$ zcat SP_perind_numers.counts.gz | more
M71.Aligned.out.bam F64.Aligned.out.bam M5.Aligned.out.bam F31.Aligned.out.bam F84.Aligned.out.bam M39.Aligned.out.bam M56.Aligned.out.bam M59.Aligned.out.bam F23.Aligned.out.bam F6.Aligned.out.bam F70.Aligned.out.bam M14.Aligned.out.bam F87.Aligned.out.bam F85.Aligned.out.bam F19.Aligned.out.bam M19.Aligned.out.bam F40.Aligned.out.bam M43.Aligned.out.bam M57.Aligned.out.bam M34.Aligned.out.bam F26.Aligned.out.bam M11.Aligned.out.bam F46.Aligned.out.bam F41.Aligned.out.bam F66.Aligned.out.bam M7.Aligned.out.bam M38.Aligned.out.bam M15.Aligned.out.bam M49.Aligned.out.bam F25.Aligned.out.bam M91.Aligned.out.bam F83.Aligned.out.bam F60.Aligned.out.bam M1.Aligned.out.bam M21.Aligned.out.bam F49.Aligned.out.bam F68.Aligned.out.bam M9.Aligned.out.bam M84.Aligned.out.bam M85.Aligned.out.bam M65.Aligned.out.bam F57.Aligned.out.bam M70.Aligned.out.bam F8.Aligned.out.bam F38.Aligned.out.bam M48.Aligned.out.bam M27.Aligned.out.bam F47.Aligned.out.bam M18.Aligned.out.bam F51.Aligned.out.bam F56.Aligned.out.bam F75.Aligned.out.bam F45.Aligned.out.bam M77.Aligned.out.bam F80.Aligned.out.bam F4.Aligned.out.bam F15.Aligned.out.bam F12.Aligned.out.bam F39.Aligned.out.bam F73.Aligned.out.bam M36.Aligned.out.bam M6.Aligned.out.bam F67.Aligned.out.bam F58.Aligned.out.bam F5.Aligned.out.bam M52.Aligned.out.bam M33.Aligned.out.bam M58.Aligned.out.bam F50.Aligned.out.bam M24.Aligned.out.bam M13.Aligned.out.bam F59.Aligned.out.bam F29.Aligned.out.bam F86.Aligned.out.bam F48.Aligned.out.bam M40.Aligned.out.bam M28.Aligned.out.bam F27.Aligned.out.bam F16.Aligned.out.bam M16.Aligned.out.bam F63.Aligned.out.bam M69.Aligned.out.bam M35.Aligned.out.bam F81.Aligned.out.bam F79.Aligned.out.bam F53.Aligned.out.bam F65.Aligned.out.bam M20.Aligned.out.bam M47.Aligned.out.bam M51.Aligned.out.bam F13.Aligned.out.bam F90.Aligned.out.bam F78.Aligned.out.bam M41.Aligned.out.bam M75.Aligned.out.bam F10.Aligned.out.bam F33.Aligned.out.bam F34.Aligned.out.bam M32.Aligned.out.bam F72.Aligned.out.bam M50.Aligned.out.bam F55.Aligned.out.bam M10.Aligned.out.bam M60.Aligned.out.bam M26.Aligned.out.bam M45.Aligned.out.bam F76.Aligned.out.bam M92.Aligned.out.bam M83.Aligned.out.bam M78.Aligned.out.bam M55.Aligned.out.bam M72.Aligned.out.bam M12.Aligned.out.bam F77.Aligned.out.bam F62.Aligned.out.bam F43.Aligned.out.bam F14.Aligned.out.bam M74.Aligned.out.bam M66.Aligned.out.bam M89.Aligned.out.bam M42.Aligned.out.bam M17.Aligned.out.bam M46.Aligned.out.bam M82.Aligned.out.bam F30.Aligned.out.bam M44.Aligned.out.bam M2.Aligned.out.bam M53.Aligned.out.bam F11.Aligned.out.bam M76.Aligned.out.bam M37.Aligned.out.bam M61.Aligned.out.bam F3.Aligned.out.bam F35.Aligned.out.bam F17.Aligned.out.bam M88.Aligned.out.bam F89.Aligned.out.bam F71.Aligned.out.bam F54.Aligned.out.bam M87.Aligned.out.bam F21.Aligned.out.bam M67.Aligned.out.bam F28.Aligned.out.bam M30.Aligned.out.bam F36.Aligned.out.bam M54.Aligned.out.bam F9.Aligned.out.bam F2.Aligned.out.bam M73.Aligned.out.bam F82.Aligned.out.bam F88.Aligned.out.bam M63.Aligned.out.bam F44.Aligned.out.bam F24.Aligned.out.bam M29.Aligned.out.bam M31.Aligned.out.bam F7.Aligned.out.bam M81.Aligned.out.bam M68.Aligned.out.bam M90.Aligned.out.bam M86.Aligned.out.bam F22.Aligned.out.bam M8.Aligned.out.bam F69.Aligned.out.bam F32.Aligned.out.bam F52.Aligned.out.bam M64.Aligned.out.bam M80.Aligned.out.bam M3.Aligned.out.bam M23.Aligned.out.bam M4.Aligned.out.bam F20.Aligned.out.bam F61.Aligned.out.bam M25.Aligned.out.bam M62.Aligned.out.bam F42.Aligned.out.bam F74.Aligned.out.bam F1.Aligned.out.bam M79.Aligned.out.bam F18.Aligned.out.bam F37.Aligned.out.bam M22.Aligned.out.bam
scaffold_633:10783:20704:clu_1_NA 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 4 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 4 0 0 0
scaffold_633:13573:20704:clu_1_NA 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 2 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
scaffold_633:14363:20704:clu_1_NA 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 4 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0
scaffold_633:16795:20704:clu_1_NA 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 4 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 4 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0
scaffold_633:19125:20704:clu_1_NA 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 4 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
scaffold_633:20760:21117:clu_2_NA 4 0 2 6 4 8 4 6 4 8 0 8 0 4 2 4 2 0 2 8 6 2 8 8 12 14 10 6 2 0 0 4 0 0 0 2 0 18 10 0 0 8 2 22 2 2 6 4 0 2 6 6 4 0 6 6 8 18 12 12 4 2 10 8 4 0 4 2 16 8 4 0 0 2 4 6 6 2 2 20 10 2 2 0 0 2 4 2 6 0 14 2 4 8 2 8 6 2 2 0 6 0 4 10 2 10 10 16 6 4 2 0 4 2 4 2 12 0 22 0 6 2 10 0 8 8 4 0 6 8 2 2 14 0 2 8 2 0 4 8 4 8 6 22 6 2 6 2 2 10 0 0 6 4 10 2 10 2 6 2 4 6 4 0 4 4 4 0 14 0 14 0 0 0 0 0 0 0 4 8 4 0
```
Each column corresponds to a different sample (original bam file) and each row to an intron, which are identified as chromosome:intron_start:intron_end:cluster_id.

## Step3. Differential splicing (working directory /Diff_splicing)
code used (provided by LeafCutter): scripts/leafcutter_ds.R

1. prepare groups_file.txt
```
$ head groups_file.txt
F1.Aligned.out.bam	Female
F2.Aligned.out.bam	Female
F3.Aligned.out.bam	Female
F4.Aligned.out.bam	Female
F5.Aligned.out.bam	Female
F6.Aligned.out.bam	Female
F7.Aligned.out.bam	Female
F8.Aligned.out.bam	Female
F9.Aligned.out.bam	Female
F10.Aligned.out.bam	Female

$ tail group_file.txt
M83.Aligned.out.bam	Male
M84.Aligned.out.bam	Male
M85.Aligned.out.bam	Male
M86.Aligned.out.bam	Male
M87.Aligned.out.bam	Male
M88.Aligned.out.bam	Male
M89.Aligned.out.bam	Male
M90.Aligned.out.bam	Male
M91.Aligned.out.bam	Male
M92.Aligned.out.bam	Male
```

2. prepare Spurpurea_exons.txt.gz
```
$ zcat Spurpurea_exons.txt.gz | more
chr	start	end	strand	gene_name
Chr01	8313	8511	+	Sapur.001G000100
Chr01	8705	8771	+	Sapur.001G000100
Chr01	9974	10069	+	Sapur.001G000100
Chr01	10164	10236	+	Sapur.001G000100
Chr01	12184	12196	+	Sapur.001G000100
Chr01	8313	8607	+	Sapur.001G000100
Chr01	8705	8771	+	Sapur.001G000100
```

3. submit job with qsub3.sh
```
#!/bin/sh
#$ -V
#$ -cwd
#$ -S /bin/bash
#$ -N diff_splicing
#$ -q omni
#$ -pe sm 8
#$ -P quanah

/home/gufeng/leafcutter/scripts/leafcutter_ds.R --num_threads 8 --exon_file=/lustre/scratch/gufeng/genome/Spur/187Zspecific/MAX_intron/Spurpurea_exons.txt.gz /lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/SP_perind_numers.counts.gz groups_file.txt
```

Two output files
1. leafcutter_ds_cluster_significance.txt
```
$ head leafcutter_ds_cluster_significance.txt
cluster	status	loglr	df	p	p.adjust	genes
chrChr01:clu_17249_NA	Success	4.70554115364803	2	0.00904501814869177	0.0383996917784009	Sapur.001G000100
chrChr01:clu_17250_NA	Success	164.590788533304	3	4.7984386469265e-71	2.68692978764019e-68	Sapur.001G000100
chrChr01:clu_17251_NA	Success	30.4196752707103	1	6.19276022448288e-15	2.77642083397649e-13	Sapur.001G000100
chrChr01:clu_17252_NA	<=1 sample with coverage>min_coverage	NA	NA	NA	NA	NA
chrChr01:clu_17253_NA	<=1 sample with coverage>min_coverage	NA	NA	NA	NA	NA
chrChr01:clu_17254_NA	<=1 sample with coverage>min_coverage	NA	NA	NA	NA	NA
chrChr01:clu_17255_NA	<=1 sample with coverage>min_coverage	NA	NA	NA	NA	NA
chrChr01:clu_17256_NA	<=1 sample with coverage>min_coverage	NA	NA	NA	NA	NA
chrChr01:clu_17257_NA	<=1 sample with coverage>min_coverage	NA	NA	NA	NA	NA
```
2. leafcutter_ds_effect_sizes.txt
```
$ head leafcutter_ds_effect_sizes.txt 
intron	logef	Female	Male	deltapsi
chrChr01:8511:8705:clu_17249_NA	0.461795406743337	0.594235390465909	0.745387851507405	0.151152461041496
chrChr01:8511:8711:clu_17249_NA	-0.23093612791807	0.0332298781097303	0.0208498561193559	-0.0123800219903745
chrChr01:8607:8705:clu_17249_NA	-0.230859278825267	0.372534731424361	0.23376229237324	-0.138772439051121
chrChr01:8771:9232:clu_17250_NA	-0.427855143001642	0.0103741875942272	0.00191800471225914	-0.00845618288196806
chrChr01:8771:9974:clu_17250_NA	-0.428141510400785	0.00970196637600011	0.00179320924913552	-0.00790875712686459
chrChr01:9501:9974:clu_17250_NA	-0.42815089181476	0.00882142729948899	0.00163044438727408	-0.00719098291221491
chrChr01:9759:9974:clu_17250_NA	1.28414754521719	0.971102418730284	0.994658341651331	0.0235559229210477
chrChr01:10000:10164:clu_17251_NA	-0.352107466392657	0.0155575692870026	0.00775414740625983	-0.00780342188074281
chrChr01:10069:10164:clu_17251_NA	0.352107466392657	0.984442430712997	0.99224585259374	0.00780342188074279
```

## Step4. Ploting significant differential splicing (working directory /Plotting)
code used (provided by LeafCutter): scripts/ds_plots.R

1. submit job with qsub4.sh
```
#!/bin/sh
#$ -V
#$ -cwd
#$ -S /bin/bash
#$ -N plotting
#$ -q omni
#$ -pe sm 8
#$ -P quanah

/home/gufeng/leafcutter/scripts/ds_plots.R -e /lustre/scratch/gufeng/genome/Spur/187Zspecific/MAX_intron/Spurpurea_exons.txt.gz /lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/SP_perind_numers.counts.gz groups_file.txt /lustre/scratch/gufeng/S_purpurea_RNA_seq/Diff_splicing/leafcutter_ds_cluster_significance.txt -f 0.05
# -f 0.05: FDR of 5%
```

## Step5. Visualization

1. prepare annotation files (working directory /lustre/scratch/gufeng/genome/Spur/187Zspecific/leafcutter_bed/new/)

(1) convert gff3 to gtf with gff3_to_gtf.pl
```
#!/usr/bin/perl

open FH, "<Spurpurea_519_v5.1.gene_exons_187Zspecific.gff3";
open OUT, ">Spurpurea_187Zspecific.gtf";

$n = 0;

while (<FH>)
{
    $seq = $_;
    chomp $seq;
    $n = $n + 1; 
    if ($n == 1)
    {
	print OUT "##gtf\n";
    }
    elsif ($n < 4)
    {
	print OUT "$seq\n";
    }
    elsif (/^(\S+)\s+(\S+)\s+(\S+)\s+(\S+)\s+(\S+)\s+(\S+)\s+(\S+)\s+(\S+)\s+ID=(Sapur\.[^\.]+)(\.*\d*)\.v/)
    {
	$one = $1;
	$two = $2;
	$three = $3;
	$four = $4;
	$five = $5;
	$six = $6;
	$seven = $7;
	$eight = $8;
	$nine = $9;
	$ten = $10;
	if ($three eq "gene")
	{
	    print OUT "$one\t$two\t$three\t$four\t$five\t$six\t$seven\t$eight\tgene_id \"$nine\";\n";
	}
	elsif ($three eq "mRNA")
	{
	    print OUT "$one\t$two\ttranscript\t$four\t$five\t$six\t$seven\t$eight\tgene_id \"$nine\"; transcript_id \"$nine$ten\";\n";
	}
	else
	{
	    print OUT "$one\t$two\t$three\t$four\t$five\t$six\t$seven\t$eight\tgene_id \"$nine\"; transcript_id \"$nine$ten\";\n";
	}
    }
}
close FH;
close OUT;
```

``` 
$ perl gff3_to_gtf.pl
```

(2) convert gtf to annotation files with leafviz/gtf2leafcutter.pl (provided by LeafCutter)
```
/home/gufeng/leafcutter/leafviz/gtf2leafcutter.pl -o Spurpurea_187Zspecific Spurpurea_187Zspecific.gtf 
```

Output files:
``` 
ls -lrth *.gz
-rw-r--r-- 1 gufeng bio 3.2M Sep 26 17:12 Spurpurea_187Zspecific_all_exons.txt.gz
-rw-r--r-- 1 gufeng bio 2.6M Sep 26 17:12 Spurpurea_187Zspecific_all_introns.bed.gz
-rw-r--r-- 1 gufeng bio 2.4M Sep 26 17:13 Spurpurea_187Zspecific_fiveprime.bed.gz
-rw-r--r-- 1 gufeng bio 2.4M Sep 26 17:13 Spurpurea_187Zspecific_threeprime.bed.gz
```

2. Generate visualization results (working directory /Visualization)
code used (provided by LeafCutter): leafviz/prepare_results.R

(1) submit job with qsub5.sh
```
#!/bin/sh
#$ -V
#$ -cwd
#$ -S /bin/bash
#$ -N Visualization
#$ -q omni
#$ -pe sm 2
#$ -P quanah

/home/gufeng/leafcutter/leafviz/prepare_results.R --meta_data_file=groups_file.txt /lustre/scratch/gufeng/S_purpurea_RNA_seq/Junc/SP_perind_numers.counts.gz /lustre/scratch/gufeng/S_purpurea_RNA_seq/Diff_splicing/leafcutter_ds_cluster_significance.txt /lustre/scratch/gufeng/S_purpurea_RNA_seq/Diff_splicing/leafcutter_ds_effect_sizes.txt /lustre/scratch/gufeng/genome/Spur/187Zspecific/leafcutter_bed/new/Spurpurea_187Zspecific
```

Output file leafviz.RData

3. Visualization (working directory local /Users/gfeng/Documents/project/Cornell/leafcutter/visualization)
copy "leafviz.RData" into this local directory
copy dependency files in leafviz/* into local directory
code used (provided by LeafCutter): leafviz/run_leafviz.R

```
./run_leafviz.R leafviz.Rdata
```

data will be shown in browser

DONE
