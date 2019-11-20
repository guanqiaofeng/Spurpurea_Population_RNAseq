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
submit job with qsub2.sh
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
