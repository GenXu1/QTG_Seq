# QTG_Seq

The open source code of pipeline for the QTG-Seq, which is dedicated to accelerating QTL fine-mapping through QTL partitioning and whole genome sequencing on bulked segregant samples

Source code of the QTG-Seq

1, Copyright (c) 2018

Huazhong Agricultural University, All Rights Reserved. Authors: Lin Li, Guoying Wang, Hongwei Zhang, Xi Wang

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

o Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

o Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

o Neither the name of the Huazhong Agricultural University nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL LIN LI, GUOYING WANG, HONGWEI ZHANG, XI WANG (OR HUAZHONG AGRICULTURAL UNIVERSITY) BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

DESCRIPTION: QTG-Seq enables rapid QTL fine-mapping using native 2nd Next Generation Sequence Data derived from the bulked high and low segregant pools. This script essentially uses the results from external alignment programs and performs a series of statistic analyses via a set of specified parameters.

CITATION: QTG-Seq can be cited as: Zhang HW, Wang X, Pan QC, Li P, Li J, Han LQ, Liu YJ, Wang PX, Li DD, Liu Y, Zhang YM, Wang GY, Li L: QTG-seq accelerates QTL fine-mapping through QTL partitioning and whole genome sequencing on bulked segregating samples, 2018.

2, Prerequisite of external softwares

Need to install Perl Statistics module, bwa in your local environment correctly and set the running path of external programs at the begining of the pipeline

3, Usage

usage:./QTG_Seq.sh [VCF] [GFF] [Cov Threshold] [Win Size] \<EuclideanDist\>

QTG-Seq provides several different statistics for analysis as follows:

	EuclideanDist (default) - Euclidean Distance
	
	SNPindex - Delta SNPindex
	
	Pvalue - Delta P(Chi-Sqrt)
	
	ED4 

4, Input

The main input file is the VCF file, which contains genomic variants for both low pool and high pool. For the genomic variant calling, we'd love to recommendate using GATK using the guided bioinformatic pipeline as follows:

<B>#mapping </B>

<I>
bwa mem -t 8 -M -P Referencegenome.fa High_Forward.fastq High_Reverse.fastq >bsa_H.sam

bwa mem -t 8 -M -P Referencegenome.fa Low_Forward.fastq Low_Reverse.fastq >bsa_L.sam
</I>

<B>#pretreatment for GATK SNP calling for hight pool</B>

<I>
java -jar ${EBROOTPICARD}/picard.jar CleanSam INPUT=bsa_H.sam OUTPUT=bsa_H_cleaned.sam

java -jar ${EBROOTPICARD}/picard.jar FixMateInformation INPUT=bsa_H_cleaned.sam OUTPUT=bsa_H_cleaned_fixed.sam SO=coordinate

java -jar ${EBROOTPICARD}/picard.jar AddOrReplaceReadGroups INPUT=bsa_H_cleaned_fixed.sam OUTPUT=bsa_H_cleaned_fixed_group.bam LB=bsa_H SO=coordinate RGPL=illumina PU=barcode SM=bsa_H

samtools index bsa_H_cleaned_fixed_group.bam

java -jar ${EBROOTPICARD}/picard.jar MarkDuplicatesWithMateCigar INPUT=bsa_H_cleaned_fixed_group.bam OUTPUT=bsa_H_cleaned_fixed_group_DEDUP.bam M=bsa_H_cleaned_fixed_group_DEDUP.mx AS=true REMOVE_DUPLICATES=true MINIMUM_DISTANCE=500

samtools index bsa_H_cleaned_fixed_group_DEDUP.bam
</I>

<B>#pretreatment for GATK SNP calling for low pool</B>

<I>
Similar to high pool
</I>


<B>#genomic variant calling</B>

<i>java -Xmx64g -jar $EBROOTGATK/GenomeAnalysisTK.jar -T HaplotypeCaller -R Referencegenome.fa -nct 8 -I bsa_H_cleaned_fixed_group_DEDUP.bam -I bsa_L_cleaned_fixed_group_DEDUP.bam -o bsa_H_L_snps_indels.vcf</i>



5, Output

Several output files will be generated by QTG_Seq. Notably, QTG_Seq_plots.pdf illustrates the genome-wide scan of QTG, while QTG_regions.txt details the QTG regions. QTG_Seq_R_summary_*.csv contains all the statistic information across the whole genome, which could be re-plotted in a more sophisticated manner at the user's will.
