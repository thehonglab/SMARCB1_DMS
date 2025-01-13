# SMARCB1 Deep Mutational Scanning
Deep mutational scanning of the SMARCB1 coding sequence to evaluate mutations which disrupt its tumor suppressor function

Study and analyis by Garrett Cooper, Benjamin Lee, [Andrew Hong](https://www.thehonglab.org/), and co authors.

For the full paper, see X

## Repository Contents

1. Raw Data 
All raw data used to generate the analyses presented in the manuscript. Files such as ```.bam``` and ```.fastq`` can be obtained from dbGAP (X).

2. Analysis Code
Computer scripts and tools required to reproduce the analysis.

3. AWS Processing Instructions
Step-by-step guidance for setting up and running data processing on an AWS instance.

## Running the Analysis

### Overview
- Cloud Computing (AWS): Portions of the analysis utilize AWS cloud servers, with corresponding code labeled as "bash" within .Rmd files.

- Local Computing: Other analyses are performed locally using R or Python. The repository is organized to allow users to run these analyses after downloading the repository. **Exception:** Differential accessibility analysis requires BAM files, which are too large to store in this repository. Refer to dbGAP ([X]) for access.



### Setting up cloud computing
To set up cloud computing environment:

1. Set up your Linux AWS instance
  - We used a t2.2xlarge instance with 32 GiB of memory and 8 vCPU with 1Tb of storage.
  
2. Install software necessary to run analysis

a) Install miniconda on your instance
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

## Genome Alignment Using Illumina Dragen

To perform genome alignment using Illumina Dragen, launch an in instance using AWS Marketplace.
We utilized version v4.2.4 for ATACseq processing and v3.7.5 for RNAseq alignment and transcript quantification. Of note these different version use different hash tables for their reference genome. Instructions on making of this hash table are provided below.

### ATACSeq Analysis (v4.2.4)
1. Transfer raw FASTQ files into the /ephemeral directory on the instance.
**NOTE** Computational resources are limited to this directory. 

2. Build v9 hash tables for the GrCh38p13 reference genome:

```
dragen --build-hash-table true --ht-reference hashref/GRCh38.p13.genome.fa --ht-build-rna-hashtable true --output-directory ref/  --enable-cnv true
```

3. Align, trim, and remove duplicates:

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

### RNAseq Analysis (v3.7.5)
1. Build v8 hash tables for the GrCh38p13 reference genome:

```
dragen --build-hash-table true --ht-reference hashref/GRCh38.p13.genome.fa --ht-build-rna-hashtable true --output-directory hashref/ --enable-cnv true --enable-rna true
```

2. Align and quantify reads:

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


## Configurations and Input Files

- The analysis configuration is defined in the ```config.yaml``` file.
- Input files referenced in ```config.yaml``` are located in the ```./data/``` subdirectory.
To customize the analysis, edit the ```config.yaml``` file as needed.


## Utilizing Code in this Repository

- Jupyter Notebooks (```*.ipynb```) and R Markdown scripts (```*.Rmd```) are located at the top level of this repository.
- Follow the instructions provided in the code comments and documentation for execution.




