# LeafCutter pipeline

## Step1. Converting bam files to junc files
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

## Step2. Identify intron clustering
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
