[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)

# Data, Code and Workflows Guideline

Assembly of high-quality genomes is critical for the characterization of structural variations (SV), for the development of a high-resolution map of polymorphisms and to serve as the foundation for gene annotation. In recent years，the advent of high-quality, long-read sequencing has enabled an affordable generation of highly contiguous de novo assemblies, as evidenced by the release of many reference genomes, including from species with large and complex genomes. The long-read sequencing technology is instrumental in accurately profiling the highly abundant repetitive sequences which otherwise challenge sequence alignment and assembly programs in eukaryotic genomes. In this protocol, we describe a step-by-step pipeline to assemble a maize genome with PacBio long reads using Canu and polish the genome using Arrow and ntEdit. Our protocol provides an optional procedure for genome assembly and could be adapted for other plant species.

To guide eBook authors having a better sense of the workflow layout, here we briefly introduce the specific purposes of the dir system. 


1. __cache__: Here, it stores intermediate datasets or results that are generated during the preprocessing steps.
2. __graphs__: The graphs/figures produced during the analysis.
3. __input__: Here, we store the raw input data. 
4. __lib__: The source code, functions, or algorithms used within the workflow.
5. __output__: The final output results of the workflow.
6. __workflow__: Step by step pipeline. 
7. __README__: This document.

## Overview of the workflow: Pre-processing of the PacBio raw reads, genome assembly and polishing

This is the workflow to show a step-by-step pipeline to assemble a maize genome with PacBio long reads.

![](graphs/Diagram.png)

## Installation

- __Running environment__: 
    - The workflow was constructed based on the __Red Hat Enterprise Linux Server release 7.7 (Maipo)__.

- __Required software and versions__: 
    - [SMRT Tools v10.1.0](https://www.pacb.com/support/software-downloads/)
    - [SequelTools v1.1.0](https://github.com/ISUgenomics/SequelTools)
    - [Canu v1.8](https://canu.readthedocs.io/en/latest/)
    - [BUSCO v3.0.2](https://busco.ezlab.org/)
    - [Pbalign v0.3.2](https://github.com/PacificBiosciences/pbalign)
    - [Sambamba v0.6.9](https://github.com/biod/sambamba)
    - [Samtools v1.12](https://github.com/samtools/samtools)
    - [Arrow v2.3.3](https://github.com/PacificBiosciences/GenomicConsensus/)
    - [ntHits v1.2.1](https://github.com/bcgsc/ntHits)
    - [ntEdit v1.2.1](https://github.com/bcgsc/ntEdit)

## Input Data

The raw data of each SMRT-cell include files named *.subreads.bam, *.subreads.pbi and *.subreadset.xml. One subreads.bam file contains multiple copies of subreads generated from the single SMRTBell from high-quality regions. It is analysis-ready and will be used directly for the following analysis. Subreads containing unaligned base calls outside of high-quality regions or excised adapter and barcode sequences are retained in a scraps.bam file. 

The example data used here is the subreads.bam file generated by using PacBio Sequel System.  

- subreads.bam file: `input/reads1.fastq`  
- subreads.bam file: `input/reads2.fastq`  

Each entry in a FASTQ files consists of 4 lines:  

1. A sequence identifier with information about the sequencing run and the cluster. The exact contents of this line vary by based on the BCL to FASTQ conversion software used.  
2. The sequence (the base calls; A, C, T, G and N).  
3. A separator, which is simply a plus (+) sign.  
4. The base call quality scores. These are Phred +33 encoded, using ASCII characters to represent the numerical quality scores.  

The first entry of the input data:
```
@HWI-ST361_127_1000138:2:1101:1195:2141/1
CGTTNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNGGAGGGGTTNNNNNNNNNNNNNNN
+
[[[_BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
```


## Major steps

#### Step 1: A.	Pre-processing of the PacBio raw reads
- Covert the *.subreads.bam files to fastq or fasta files with the PacBio tool bam2fastq.
- Use -c 9 to get all the subreads and then let the assembler decide which reads are good for genome assembly. The command bam2fastq will generate a fastq file (raw_PacBio.fastq) 

```
bam2fastq -c 9 raw_PacBio.subreads.bam
```

- Use SequelTools to perform quality control of PacBio Sequel raw sequencing data from multiple SMRTcells.
- First, generate a file with a list of locations of *.subreads.bam files and *.scraps.bam files.

```
 find $PWD/*.subreads.bam > subFiles.txt
 find $PWD/*.scraps.bam > scrFiles.txt
```

- Then, run the QC tool of SequelTools with *.scraps.bam files.
- Use -t Q to use the QC tool specifically. The argument -u is mandatory to identify a file listing the locations of the subread BAM files. The argument -c is optional to identify a file listing the locations of the scraps BAM files.

```
  ./SequelTools.sh -t Q -u subFiles.txt -c scrFiles.txt
```

#### Step 2: Genome assembly
- Correct the raw reads
- In this phase, Canu will do multiple rounds of overlapping and correction. In order to run the correction phase specifically, the users need to use -pacbio-raw option to provide raw PacBio reads as input data and use -correct option to let Canu only correct the raw reads. If the users have more than 4,096 input files, they must consolidate them into fewer files. The output of the correction phase will be one compressed fasta file with all corrected reads (maize.correctedReads.fasta.gz in our example).
- The -p <string> option is mandatory to set the file name prefix of intermediate and output files. The -d <assembly directory> is optional. If it is not provided, Canu will run in the current directory. The genomeSize parameter is required by Canu which will be used to determine coverage in the input reads. The users can provide the estimated genome size in bases or with common SI prefixes.

```
canu -correct \
     -p maize -d maize \
     genomeSize=2.3g \
     -pacbio-raw raw_PacBio.fastq

```

#### Step 3: view the results

- Results can be visualized by clicking `output/multiqc_report.html`.
- Alternatively, you can plot the results yourself using the below R code.

```
3_visualize_results.Rmd
```

## Expected results

![](graphs/figure1.png)

## License
It is a free and open source software, licensed under []() (choose a license from the suggested list:  [GPLv3](https://github.com/github/choosealicense.com/blob/gh-pages/_licenses/gpl-3.0.txt), [MIT](https://github.com/github/choosealicense.com/blob/gh-pages/LICENSE.md), or [CC BY 4.0](https://github.com/github/choosealicense.com/blob/gh-pages/_licenses/cc-by-4.0.txt)).
