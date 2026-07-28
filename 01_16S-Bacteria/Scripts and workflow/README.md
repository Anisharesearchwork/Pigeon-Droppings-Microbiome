#Scripts and Workflow

## 1. Computational Environment
Window system (LINUX-WSL) environment with Ubuntu platform
QIIME 2 - for amplicon sequencing analysis 
R and R studio- for data visualization 
Galaxy - for quality check and selected downstream analyses and visualizations
Analysis performed using [aired-end illumina sequencing data in FASTQ (.fastq.gz) format. A sample metadata and a manifest file were prepared for importing sequencing data into QIIME2

# Update Conda
conda update conda

# Create Python environment
conda create -n test_env python=3.10

# Activate environment
conda activate test_env

# Verify environments
conda env list

# Install QIIME 2
wget https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.10-py310-linux-conda.yml

# creating conda environment
conda env create \
-n qiime2-amplicon-2024.10 \
--file qiime2-amplicon-2024.10-py310-linux-conda.yml

# activating environment
conda activate qiime2-amplicon-2024.10

qiime --help

# Organize Data
mkdir -p ~/qiime_project/fastq_files

cd ~/qiime_project/fastq_files

cp *.fastq.gz ~/qiime_project/fastq_files/

# Prepare Manifest file
sample-id
forward-absolute-filepath
reverse-absolute-filepath

## 2. Data Import
Raw data imported into QIIME 2 using manifest file, sample meatdata provided in TSV format. The imported sequences wee stored in QIIME 2 artifact as - Demultiplexed sequence artifact (.qza)

# Import into QIIME 2
qiime tools import \
--type 'SampleData[PairedEndSequencesWithQuality]' \
--input-path manifest.tsv \
--output-path demux.qza \
--input-format PairedEndFastqManifestPhred33V2

# Output
demux.qza


## 3. Quality Assessment

## 4. Denoising and ASV generation 

## 5. Phylogenetic Analysis

## 6. Taxonomic Classification

## 7. Diversity Analysis

## 8. Data Visualization

## 9. Functional prediction 

