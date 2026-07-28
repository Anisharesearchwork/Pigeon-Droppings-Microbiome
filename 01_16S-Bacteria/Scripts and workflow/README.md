#Scripts and Workflow

## 1. Computational Environment
Window system (LINUX-WSL) environment with Ubuntu platform
QIIME 2 - for amplicon sequencing analysis 
R and R studio- for data visualization 
Galaxy - for quality check and selected downstream analyses and visualizations
Analysis performed using Paired-end illumina sequencing data in FASTQ (.fastq.gz) format. A sample metadata and a manifest file were prepared for importing sequencing data into QIIME2

# Updating Conda
conda update conda

# Create Python environment
conda create -n test_env python=3.10

# Activate and verifying the environment
conda activate test_env

conda env list

# Installing the QIIME 2
wget https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.10-py310-linux-conda.yml

# Creating conda environment
conda env create \
-n qiime2-amplicon-2024.10 \
--file qiime2-amplicon-2024.10-py310-linux-conda.yml

# Activating QIIME2 environment 
conda activate qiime2-amplicon-2024.10

qiime --help

# Organize Data
mkdir -p ~/qiime_project/fastq_files

cd ~/qiime_project/fastq_files

cp *.fastq.gz ~/qiime_project/fastq_files/

# Prepare Manifest file
includes- sample-id, forward-absolute-filepath and reverse-absolute-filepath

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
visualiozation of demux file produced using the following command - 

qiime demux summarize \
--i-data demux.qza \
--o-visualization demux.qzv
Output
demux.qzv

# Additional quality reports used in the study

Using the Galaxy Europe platform - fastqc, falco and multiqc reports of the raw data files were also generated to look into the quality of the data. 

Steps involved include -

for FastQC - 
1. Logged into the Galaxy Europe server.
2. Uploaded paired-end FASTQ (.fastq.gz) files.
3. Searched for *FastQC* in the Galaxy tools panel.
4. Selected the forward and reverse FASTQ files as input.
5. Run the FastQC tool using the default parameters.
6. Downloaded and reviewed the HTML quality reports.

For Falco- 
1. Selected the uploaded FASTQ files.
2. Searched for *Falco* in the Galaxy tools panel.
3. Run Falco using the default parameters.
4. Examined the generated quality assessment report.

For MultiQC analysis- 
1. Completed FastQC analysis for all samples.
2. Searched for *MultiQC* in the Galaxy tools panel.
3. Selected all FastQC output files as input.
4. Executed MultiQC using the default parameters.
5. Downloaded the combined quality control report.

## 4. Denoising and ASV generation 
DADA2 was used to perform quality filter, trim , denoise , merging ad chimera removal. It generated ASVs , representative sequences and denoising statistics.

qiime dada2 denoise-paired \
--i-demultiplexed-seqs demux.qza \
--p-trim-left-f 0 \
--p-trim-left-r 0 \
--p-trunc-len-f 240 \
--p-trunc-len-r 240 \
--o-table table.qza \
--o-representative-sequences rep-seqs.qza \
--o-denoising-stats denoising-stats.qza

# Denoising statistics visualization file creation

qiime metadata tabulate \
--m-input-file denoising-stats.qza \
--o-visualization stats.qzv

# Summarizing feature table
qiime feature-table summarize \
--i-table table.qza \
--o-visualization table.qzv \
--m-sample-metadata-file Metadata_16s.tsv

# Visualization of representative sequences
qiime feature-table tabulate-seqs \
--i-data rep-seqs.qza \
--o-visualization rep-seqs.qzv

# Output
Feature table (.qza)
Representativesequences (.qza)
Denoising statistics (.qza)

## 5. Phylogenetic Analysis
Representative sequences were aligned using MAFFT and phylogenetic tree was created using FastTree. the rooted phylogentic tree wsa used for the downstream work.

qiime phylogeny align-to-tree-mafft-fasttree \
--i-sequences rep-seqs.qza \
--o-alignment aligned-rep-seqs.qza \
--o-masked-alignment masked-aligned-rep-seqs.qza \
--o-tree unrooted-tree.qza \
--o-rooted-tree rooted-tree.qza

# Output
Multiple sequence alignmnet (.qza)
Masked alignmnet (.qza)
Unrooted phylogenetic tree (.qza)
Rooted phylogenetic tree (.qza)

## 6. Taxonomic Classification
Representative sequences were taxonomically classified using the Greengeens2 reference database and SILVA 138-99. Taxonomic assignments were summarized to find the bacterial community composition.

# Greengenes classifier 

qiime feature-classifier classify-sklearn \
--i-classifier gg-13-8-99-515-806-nb-classifier.qza \
--i-reads rep-seqs.qza \
--o-classification taxonomy_gg.qza

qiime metadata tabulate \
--m-input-file taxonomy_gg.qza \
--o-visualization taxonomy_gg.qzv

qiime taxa barplot \
--i-table table.qza \
--i-taxonomy taxonomy_gg.qza \
--m-metadata-file Metadata_16s.tsv \
--o-visualization taxa-barplot-gg.qzv

## SILVA Classifier

qiime feature-classifier classify-sklearn \
--i-classifier silva-138-99-nb-classifier.qza \
--i-reads rep-seqs.qza \
--o-classification taxonomy_silva.qza

qiime metadata tabulate \
--m-input-file taxonomy_silva.qza \
--o-visualization taxonomy_silva.qzv

qiime taxa barplot \
--i-table table.qza \
--i-taxonomy taxonomy_silva.qza \
--m-metadata-file Metadata_16s.tsv \
--o-visualization taxa-barplot-silva.qzv

# Output
for both Greengeen and SILVA - 
Taxonomy artifact (.qza)
Taxonomy visualization (.qzv)
Taxonomic abundance tables
Taxa bar plots

## 7. Diversity Analysis
Alpha and Beta diversity analyses were performed using the rooted phylogenetic tree and feature table. 
Alpha diversity metrics used to estimate sample diversity within  and beta diversity metrics to compare microbial community compositionn between samples.

qiime diversity core-metrics-phylogenetic \
--i-phylogeny rooted-tree.qza \
--i-table table.qza \
--p-sampling-depth 10000 \
--m-metadata-file Metadata_16s.tsv \
--output-dir core-metrics-results

# Output
Alpha diversity metrics
Beta diversity metrics 
Principal Coordinate Analysis (PCOA) plots

## Export
Export of files to various formats using the commands for downstream analysis of the ddata produced via QIIME2

# Exported taxonomy

qiime tools export \
--input-path taxonomy.qza \
--output-path exported-taxonomy

# Exported feature table

qiime tools export \
--input-path table.qza \
--output-path exported-feature-table

# Exported representative sequencs

qiime tools export \
--input-path rep-seqs.qza \
--output-path exported-rep-seqs

# BIOM files to TSV 

biom convert \
-i exported-feature-table/feature-table.biom \
-o feature-table.tsv \
--to-tsv

# Output
Taxonomy tables
Feature table
BIOM files
TSV tables

## 9. Data Visualization
Data visualization and diversity results were visualized using QIIME2 and R analysis using R studio.
Stacked bar plot, heatmaps, krona charts , relative abundance plots , phylogenetic tress and other graphical representations were generated  for interpretation of bacterial community structure in the sample.

# R studio - Set working directory
setwd("C:/Users/USER/OneDrive/Desktop/Microbiome_R/Greengenes")

# Load libraries

library(ggplot2)
library(dplyr)
library(tidyr)
library(readr)
library(reshape2)
library(pheatmap)
library(RColorBrewer)
library(vegan)
library(stringr)

# Import data
imported these data - feature-table.tsv, taxonomy.tsv and metadata_16s.tsvfor R visualization.

feature_table <- read.delim("Data/feature-table.tsv",
                            skip = 1,
                            check.names = FALSE)

taxonomy <- read.delim("Data/taxonomy.tsv",
                       check.names = FALSE)

feature if=d renamed -

colnames(feature_table)[1] <- "Feature ID"
colnames(taxonomy)[1] <- "Feature ID"

# Merge feature table and taxonomy
merged feature_table, taxonomy with Feature ID- 

merged_table <- merge(feature_table,
                      taxonomy,
                      by = "Feature ID")
                      
# Extract taxonomy levels
Kingdom, Phylum, Class, Order, Family, Genus and Species using stringr- level 1 to 7 indicate the differnt taxonomic levels.

taxonomy_split <- str_split_fixed(merged_table$Taxon, ";", 7)

merged_table$Kingdom <- str_trim(taxonomy_split[,1])
merged_table$Phylum  <- str_trim(taxonomy_split[,2])
merged_table$Class   <- str_trim(taxonomy_split[,3])
merged_table$Order   <- str_trim(taxonomy_split[,4])
merged_table$Family  <- str_trim(taxonomy_split[,5])
merged_table$Genus   <- str_trim(taxonomy_split[,6])
merged_table$Species <- str_trim(taxonomy_split[,7])

# Save merged table
To save each table 

write.csv(merged_table,
          "Results/Merged_Table_SILVA.csv",
          row.names = FALSE)

# Create abundance tables
Generated abundance tables for Kingdom, Phylum, Class, Order, Family, Genus and Species using group_by() summarise() mutate() arrange()
Save each as CSV.

# Calculate relative abundance
Create Relative_Phylum.csv, Relative_Class.csv, Relative_Order.csv, Relative_Family.csv, Relative_Genus.csv and Relative_Species.csv

# Stacked bar plots
Generated stacked bar plots for Phylum, Class, Order, Family, Genus and Species
Save as PNG.

# Heat maps
Generated heat maps for Complete genus and Top 50 genus using pheatmap()
Blue–white–red colour scale.
Save PNG.

# Pie charts
Generated pie charts for Phylum, Class, Order, Family, Genus and Species using ggplot(), geom_col() and coord_polar()
Save PNG.

# Taxonomic abundance bar plots
Generated abundance plots for Phylum, Class, Order, Family, Genus, Species using geom_col()and coord_flip()
Save PNG.

# Output
Merged table
Abundance tables
Relative abundance tables
Figures
Stacked bar plots
Heat maps
Pie charts
Abundance bar plots

This is the complete R console workflow followed for the Greengenes and SILVA analysed visualizations making. 

## 10. Functional prediction 
Functional profiling of the bacterial communities was performed using the PICRUSt2. Predicted functional pathways and gene family abundaces were obtained from the ASV and representative sequences. PICRUSt2 was done in Galaxy Europe platform.

Steps involved include- 

1. Uploaded the ASV feature table and representative sequences exported from QIIME 2.
2. Searched for *PICRUSt2* in the Galaxy tools panel.
3. Selected the feature table and representative sequences as input.
4. Executed the PICRUSt2 workflow using the recommended default parameters.
5. Downloaded the predicted functional pathway and gene family output files for downstream analysis.


