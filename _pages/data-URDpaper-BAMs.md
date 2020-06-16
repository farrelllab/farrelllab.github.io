---
layout: content
title: BAM files from Farrell, Wang, *et al.* 2018.
display-title: BAM files from Farrell, Wang, <i>et al.</i> 2018
permalink: /data/2018FarrellWang-BAMs.html
---

## Why we posted the BAM files instead of FASTQ files

During the synthesis of Drop-seq beads, there are often synthesis errors where beads miss out on one of the cell barcode synthesis cycles. This results in a cell barcode that is one base too short, with the first base of the UMI getting read as the last base of the cell barcode. So, essentially, that cell gets split into four cells with a different base in the last position of the cell barcode. The Drop-seq tools include a tool (`DetectBeadSynthesisErrors`) to detect this and correct the cell barcodes, adding an N to the cell barcode and moving the final base back into the UMI tag. However, it is run as the last step in the pipeline, and its detection is dependent on some parameters of the alignment. Thus, to ensure that the cell barcodes are consistent across studies that use this data (especially if users process it with a different pipeline), we decided it was better to provide users with the BAM files from the terminal step of our processing, which would include those corrected cell barcodes.

## How to download the BAM files

SRA has hosted our mapped BAM files. If you download them with `sratools` or other similar tools, unfortunately the relevant cell barcode and UMI tagged are generally stripped off. However, the BAMs can be downloaded from the SRA Run Selector website.

1. Visit SRA Run Selector, Study GSE106474: [https://www.ncbi.nlm.nih.gov/Traces/study/?acc=GSE106474](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=GSE106474)
2. Click on the 'SRRnnnnn' Run access numbers
3. Click the 'Data Access' tab at the top
4. Find the link to the 'Original Format' BAM at the bottom of the page.

## How to re-align from the BAM files

The following steps depend on Picard tools and are essentially re-running several steps of the Drop-seq pipeline.

1. Use Picard tools `SamToFastq` to extract a FASTQ from the BAM that can be passed to an aligner.

*Example:*
`java -Xmx4g -jar /path/to/picard/picard.jar SamToFastq INPUT=JeffsBam.bam FASTQ=ToAligner.fastq`

2. Align using whatever you want! To whatever you want! Go **wild**! The file you want is `ToAligner.fastq`

3. (Can be done in parallel with step 2.) Use Picard tools `RevertSam` to remove the alignment flags from my BAM.

*Example:*
`java -Xmx4g -jar /path/to/picard/picard.jar RevertSam INPUT=JeffsBam.bam OUTPUT=Reverted.bam`

4. Use Picard tools `SortSam` to ensure that the output from the aligner and the output from `RevertSam` are both sorted in the same (**queryname**) order. 

*Example:*
`java -Xmx4g -jar /path/to/picard/picard.jar SortSam I=FromAligner.sam O=aligned.sort.bam SO=queryname`
`java -Xmx4g -jar /path/to/picard/picard.jar SortSam I=Reverted.bam O=reverted.sort.bam SO=queryname`

5. Use Picard tools `MergeBamAlignment` to merge my BAM and your newly aligned BAM. This will combine the tags from my BAM, like the cell (**XC:**) and molecular barcodes (**XM:**) and the alignment information and tags produced by the aligner. The REFERENCE_SEQUENCE argument refers to the fasta file that was used to generate the reference you aligned against (i.e. probably the genome).

*Example:*
`java -Xmx4g -jar /path/to/picard/picard.jar MergeBamAlignment REFERENCE_SEQUENCE=my_fasta.fasta UNMAPPED_BAM=reverted.sort.bam ALIGNED_BAM=aligned.sort.bam`

6. Continue however you like with your new and amazing BAM files! You could use programs from the Drop-seq pipeline (`TagReadWithGeneExon` and `DigitalExpression`) to generate a Cell x Gene expression table. Or do something else! You should **not** run the Drop-seq tool `DetectBeadSynthesisErrors` on the BAM, as that has already been run in the past.

