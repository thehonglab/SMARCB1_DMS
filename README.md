# SMARCB1_DMS
Deep mutational scanning of the SMARCB1 coding sequence to evaluate mutations which disrupt its tumor suppressor function

Study and analyis by Garrett Cooper, Benjamin Lee, Andrew Hong, and co authors.

For the full paper, see X

## Running the Analysis

This repository contains:

1. Raw data used to generate analysis in manuscript

2. Computer code to run the analysis.

Much of the computational analysis was performed using cloud computing servers on Amazon Web Services (AWS). For portions of analysis which utilize AWS cloud computing, code is denoted in .Rmd files as "bash".

The rest of the analysis was completed locally using either R or Python. The repository is set up in such a way that all local analysis done in R or Python can be run by those who download this repository on their local computer. The only exception is differential accessibility analysis, which requires bam files, which are too large to store on Git LFS. To download these bam files, please refer to dbGAP (X).

To set up cloud computing environment:

1. Set up your Linux AWS instance
  - We used a t2.2xlarge instance with 32 GiB of memory and 8 vCPU with 1Tb of storage.
  
2. Install software necessary to run analysis

a)Install miniconda on your instance
```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```
(Close terminal and reconnect to your instance)

b) Configure your miniconda installation
```
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```

c) Update python version
```
conda install python=3.10
```

d) Install required packages
```
conda install -c bioconda samtools
conda install -c bioconda deeptools
conda install -c bioconda macs3
conda install -c bioconda bedtools
```

e) Install HOMER
```
wget http://homer.ucsd.edu/homer/configureHomer.pl
perl configureHomer.pl -install
echo 'export PATH=/path/to/homer/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```


To perform alignment and read trimming using Illumina Dragen, launch an in instance using AWS Marketplace.
We utilized version v4.2.4 for ATACseq processing and v3.7.5 for RNAseq alignment and transcript quantification. Of note these different version use different hash tables for their reference genome. Instructions on making of this hash table are provided below.

### ATACSeq Analysis (Dragen v4.2.4)
Raw FASTQ files were transferred into the preloaded /ephemeral directory on the instance (computational resources only work in this directory).

We then generated v9 hash tables for the reference GrCh38p13 using:

```
dragen --build-hash-table true --ht-reference hashref/GRCh38.p13.genome.fa --ht-build-rna-hashtable true --output-directory ref/  --enable-cnv true
```

For ATACseq analysis, alignment, adaptor trimming, and duplicate removal were completed using the following command for each sample. An example script for one of the samples is shown below.

```
dragen -r GRCh38p13v9 \
-1 DNA/I315I-R1_S1_R1_001.fastq.gz \
-2 DNA/I315I-R1_S1_R2_001.fastq.gz \
--trim-adapter-read1 /opt/edico/config/adapter_sequences.fasta \
--trim-adapter-read2 /opt/edico/config/adapter_sequences.fasta \
--read-trimmers adapter \
--soft-read-trimmers none \
--remove-duplicates true \
--output-directory alignments \
--output-file-prefix I315I-R1 \
--RGID I315I-R1_01 \
--RGSM I315I-R1_01_S1;
```

### RNAseq Analysis (Dragen v3.7.5)
Raw FASTQ files were transferred into the preloaded /ephemeral directory on the instance (computational resources only work in this directory).

We then generated v8 hash tables for the reference GrCh38p13 using:

dragen --build-hash-table true --ht-reference hashref/GRCh38.p13.genome.fa --ht-build-rna-hashtable true --output-directory hashref/ --enable-cnv true --enable-rna true

For RNAseq analysis, alignment and read quantification was performed using this following command:

```
dragen -f -r GrCh38p13 \
-1 RNA/G401_P16_STT21_MRT00_SEQ61_I315I_NOV5_1.fq.gz \
-2 RNA/G401_P16_STT21_MRT00_SEQ61_I315I_NOV5_2.fq.gz \
-a GrCh38p13/gencode.v36.annotation.gtf \
--enable-map-align true \
--enable-rna true \
--enable-rna-quantification true \
--enable-map-align-output true \
--output-directory RNAOut \
--output-file-prefix G401_P16_STT21_MRT00_SEQ61_I315I_NOV5 \
--RGID G401_P16_STT21_MRT00_SEQ61_I315I_NOV5_01 \
--RGSM G401_P16_STT21_MRT00_SEQ61_I315I_NOV5_01_S1;
```


## Reading input files

The analysis configuration is defined in the config.yaml file, which specifies key variables for the analysis. The file is designed to be straightforward and easy to understand. To customize the analysis, simply update the values in this configuration file.

The input files referenced in config.yaml are located in the ./data/ subdirectory.

## Utilizing Code in this Repository

All Jupyter notebooks and R markdown scripts are contained within the top level of this repository with the extension ```*.ipynb``` or ```*.Rmd```.





