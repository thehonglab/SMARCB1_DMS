# SMARCB1 Deep Mutational Scanning
Deep mutational scanning of the SMARCB1 coding sequence to evaluate mutations which disrupt its antiproliferative function

Authors: Garrett Cooper, Benjamin Lee, [Andrew Hong](https://www.thehonglab.org/), and co authors.

📄 [Preprint](https://pubmed.ncbi.nlm.nih.gov/40196006/) | 🧬 [Interactive Heatmap](https://thehonglab.github.io/SMARCB1_DMS/) | 📦 Raw Data: dbGaP accession [phs003896.v1.p1](https://www.ncbi.nlm.nih.gov/projects/gap/cgi-bin/study.cgi?study_id=phs003896.v1.p1)

## Repository Contents

1. Raw Data: 
All raw data used to generate the analyses presented in the manuscript. Files such as ```.bam``` and ```.fastq``` and ```.bw``` can be obtained from dbGAP (phs003896.v1.p1).

  - For Hong Lab members, data files can be accessed via the lab's secure s3 storage system. Contact the administrator for details.

2. Analysis Code:
Annotated scripts required to reproduce the analysis.

3. AWS Processing Instructions:
Step-by-step guidance for setting up and running data processing on an AWS instance.

## Running the Analysis

### Computing Environment
Analysis are split between two environments

- Cloud (AWS): Portions of the pipeline run on AWS; corresponding code is labeled as bash within .Rmd files.

- Local (R/Python): All other analyses can be run locally after cloning this repository. (Of note: MD simulations utilized a NVIDIA GeForce RTX 5090 GPU, which may not be available on all local machines.)


### Cloud Setup (AWS)

We primarily used a t2.2xlarge instance (8 CPU, 32 GiB RAM, 1 TB storage)

1. Install minicoda

```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```
Close and reconnect to your instance, then configure:

```
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
conda install python=3.10
```



2. Install required packages
```
conda install -c bioconda samtools
conda install -c bioconda deeptools
conda install -c bioconda macs3
conda install -c bioconda bedtools
```

3. Install HOMER
```
wget http://homer.ucsd.edu/homer/configureHomer.pl
perl configureHomer.pl -install
echo 'export PATH=/path/to/homer/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

## Genome Alignment Using Illumina Dragen

Launch a DRAGEN instance via the AWS marketplace.

| Pipeline | DRAGEN Version | Notes |
|----------|---------------|-------|
| ATAC-seq | v4.2.4 | Uses v9 hash tables |
| RNA-seq | v3.7.5 | Uses v8 hash tables |

> **Note:** Different DRAGEN versions require different reference genome hash tables. Build instructions are provided below.

ATAC-seq (v4.2.4)

1. Transfer raw FASTQ files to the /ephemeral directory.

2. Build v9 hash tables for the GrCh38p13:

```
dragen --build-hash-table true --ht-reference hashref/GRCh38.p13.genome.fa --ht-build-rna-hashtable true --output-directory ref/  --enable-cnv true
```

3. Align, trim, and remove duplicates (run per sample):
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
1. Build v8 hash tables for the GrCh38p13:

```
dragen --build-hash-table true --ht-reference hashref/GRCh38.p13.genome.fa --ht-build-rna-hashtable true --output-directory hashref/ --enable-cnv true --enable-rna true
```

2. Align and quantify reads (run per sample):

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


## Configuration

- Analysis parameters are defined in ```config.yaml``` .
- Input files referenced in ```config.yaml``` are located in ```./data/```.
- To customize the analysis, edit ```config.yaml``` as needed.


## Running Notebooks

Jupyter Notebooks (*.ipynb) and R Markdown scripts (*.Rmd) are located at the top level of the repository. Follow the instructions in each file's comments and documentation for execution details.





