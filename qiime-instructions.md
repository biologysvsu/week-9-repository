# IMPORTANT: This tutorial assumes you have already installed multiqc and qiime, if not, follow the installation steps in `week-7-repository` -> `microbiome-instructions.md`

## Go to the right working directory
1. Go to your ocean folder using your alias. If you dont have an alias, cd the long way `cd /ocean/projects/agr250001p/your-username`
```
myocean
```
2. cd into the week-7-repository you forked and cloned last week
```
cd week-7-repository
```
3. List directory using `ls`, you should have the following files and folders:
```
Blueberry_metadata_reduced.tsv  fastqc_mock_out      microbiome-instructions.md  multiqc_mock_data
MultiQC                         fastqc_raw_data_out  mock-data                   raw_data
README.md                       filt_stats.qza       multiqc_mock.html           reads_qza
```
- If you do, you may start at the **Denoising reads** section, step 4.
- If you do NOT, you may start with **Quality control**

# Quality Control (QC)
1. Create a symlink to the data like this, yup, you not always have to copy or move files around!
```
ln -s /ocean/projects/agr250001p/shared/week-7-data/raw_data .
```
2. Create a directory for the fastqc output:
```
mkdir fastqc_raw_data_out
```
3. Run fastqc
```
module load FastQC
fastqc -t 4 raw_data/*.fastq.gz -o fastqc_raw_data_out
```
4. Run multiqc
```
multiqc --dirs fastqc_raw_data_out --filename multiqc_raw_data.html
```

  - If multiqc gives you a python error, run the following lines:
  - ```
    /jet/packages/spack/opt/spack/linux-centos8-zen2/gcc-10.2.0/python-3.8.6-            jaihmn5fofhkpkdsskfz25ez6s2camcf/bin/python3 -m ensurepip --default-pip

    /jet/packages/spack/opt/spack/linux-centos8-zen2/gcc-10.2.0/python-3.8.6-jaihmn5fofhkpkdsskfz25ez6s2camcf/bin/python3 -m pip install --upgrade pip

    /jet/packages/spack/opt/spack/linux-centos8-zen2/gcc-10.2.0/python-3.8.6-jaihmn5fofhkpkdsskfz25ez6s2camcf/bin/python3 -m pip install --user multiqc
```

## Let's run Quimme to study the microbiome
1. Create a Symlink for the metadata file from the share folder
```
ln -s /ocean/projects/agr250001p/shared/week-7-data/Blueberry_metadata_reduced.tsv .
```
2. Inspect the metadata file
```         
head Blueberry_metadata_reduced.tsv
```
- Answer questions 11 and 12

## A few formatting steps required by quimme
1. Let's create the following directory
```         
mkdir reads_qza
```
2. You had installed the latest version of QIIME2, so you can either activate that environment with the command below.

```         
conda activate qiime2-amplicon-2024.2
```
- When you are finished this tutorial you can deactivate the environment using:

```         
conda deactivate
```
3. Feed qiime the raw reads
```         
qiime tools import \
  --type SampleData[PairedEndSequencesWithQuality] \
  --input-path raw_data/ \
  --output-path reads_qza/reads.qza \
  --input-format CasavaOneEightSingleLanePerSampleDirFmt
```
- cassava format, files follow a pattern `SampleID_L001_R1_001.fastq.gz`

4. remove primer sequences from reads
```
qiime cutadapt trim-paired \
  --i-demultiplexed-sequences reads_qza/reads.qza \
  --p-cores 4 \
  --p-front-f ACGCGHNRAACCTTACC \
  --p-front-r ACGGGCRGTGWGTRCAA \
  --p-discard-untrimmed \
  --p-no-indels \
  --o-trimmed-sequences reads_qza/reads_trimmed.qza
```
5 Visualize your data now
```         
qiime demux summarize \
  --i-data reads_qza/reads_trimmed.qza \
  --o-visualization reads_qza/reads_trimmed_summary.qzv
```
- Git add, commit and push `reads_trimmed_summary.qzv` Download and open in https://view.qiime2.org/

### Denoising reads
1. Join pair-end reds
```         
qiime vsearch merge-pairs \
  --i-demultiplexed-seqs reads_qza/reads_trimmed.qza \
  --output-dir reads_qza/reads_joined
```
2. Filter out low-quality reads
```         
qiime quality-filter q-score \
  --i-demux reads_qza/reads_joined/merged_sequences.qza \
  --o-filter-stats filt_stats.qza \
  --o-filtered-sequences reads_qza/reads_trimmed_joined_filt.qza
```
3. Summarize results
```         
qiime demux summarize \
  --i-data reads_qza/reads_trimmed_joined_filt.qza \
  --o-visualization reads_qza/reads_trimmed_joined_filt_summary.qzv
```
- Git add, commit and push `reads_trimmed_joined_filt.qzv` Download and open in https://view.qiime2.org/
- Answer question 13

4. Actual denoising with deblur

```         
qiime deblur denoise-16S \
  --i-demultiplexed-seqs reads_qza/reads_trimmed_joined_filt.qza \
  --p-trim-length 390 \
  --p-sample-stats \
  --p-jobs-to-start 4 \
  --p-min-reads 1 \
  --output-dir deblur_output
```
- Note: this command may take up to 10 minutes or so to run.

5. Summarize dublur output
```         
qiime feature-table summarize \
  --i-table deblur_output/table.qza \
  --o-visualization deblur_output/deblur_table_summary.qzv
```
- Git add, commit and push `deblur_table_summary.qzv` Download and open in https://view.qiime2.org/

###  Run taxonomic classification
1. Create symlink to taxonomic data
```
ln -s /ocean/projects/agr250001p/shared/week-7-data/taxa .
```
2. Run taxonomic classification (Homework)
```         
qiime feature-classifier classify-sklearn \
  --i-reads deblur_output/representative_sequences.qza \
  --i-classifier /ocean/projects/agr250001p/shared/week-7-data/silva-138-99-nb-classifier.qza \
  --p-n-jobs 4 \
  --output-dir taxa
```
### Filtering resultant table
1. Filter out rare ASVs
```         
qiime feature-table filter-features \
  --i-table deblur_output/table.qza \
  --p-min-frequency 10 \
  --p-min-samples 1 \
  --o-filtered-table deblur_output/deblur_table_filt.qza
```
- replace X with desired frequency
2. Filter out contaminant and unclassified ASVs
```         
qiime taxa filter-table \
  --i-table deblur_output/deblur_table_filt.qza \
  --i-taxonomy taxa/classification.qza \
  --p-include p__ \
  --p-exclude mitochondria,chloroplast \
  --o-filtered-table deblur_output/deblur_table_filt_contam.qza
```
3. Subset and summarize filtered table
```         
qiime feature-table summarize \
  --i-table deblur_output/deblur_table_filt_contam.qza \
  --o-visualization deblur_output/deblur_table_filt_contam_summary.qzv
```
- Git add, commit and push `deblur_table_filt_contam_summary.qzv` Download and open in https://view.qiime2.org/
- Answer question 16 What is the minimum and maximum sequencing depth across all samples?

4. Copy final table to current directory
```         
cp deblur_output/deblur_table_filt_contam.qza deblur_output/deblur_table_final.qza
```
5. Produce final table summary
```         
qiime feature-table filter-seqs \
  --i-data deblur_output/representative_sequences.qza \
  --i-table deblur_output/deblur_table_final.qza  \
  --o-filtered-data deblur_output/rep_seqs_final.qza
```
```         
qiime feature-table summarize \
  --i-table deblur_output/deblur_table_final.qza \
  --o-visualization deblur_output/deblur_table_final_summary.qzv
```

- Git add, commit and push `deblur_table_final_summary.qzv` Download and open in https://view.qiime2.org/

# VISUALIZATION
1. This is a phylogenetic tree I pre-generated, please create a symlink
```         
ln -s /ocean/projects/agr250001p/shared/week-7-data/asvs-tree.qza .
ln -s /ocean/projects/agr250001p/shared/week-7-data/insertion-placements.qza .
```
2. Generate rarefaction curves
```         
qiime diversity alpha-rarefaction \
  --i-table deblur_output/deblur_table_final.qza \
  --p-max-depth X \
  --p-steps 20 \
  --i-phylogeny asvs-tree.qza \
  --o-visualization rarefaction_curves.qzv
```
- Answer question 17

2. Calculating diversity metrics
```         
qiime diversity core-metrics-phylogenetic \
  --i-table deblur_output/deblur_table_final.qza \
  --i-phylogeny asvs-tree.qza \
  --p-sampling-depth X  \
  --m-metadata-file Blueberry_metadata_reduced.tsv \
  --p-n-jobs-or-threads 4 \
  --output-dir diversity
```
3. Produce boxplot for shannon
```         
qiime diversity alpha-group-significance \
  --i-alpha-diversity diversity/shannon_vector.qza \
  --m-metadata-file Blueberry_metadata_reduced.tsv \
  --o-visualization diversity/shannon_compare_groups.qzv
```
4. Generate stacked bar chart of taxa relative abundances
```         
qiime taxa barplot \
  --i-table deblur_output/deblur_table_final.qza \
  --i-taxonomy taxa/classification.qza \
  --m-metadata-file Blueberry_metadata_reduced.tsv \
  --o-visualization taxa/taxa_barplot.qzv
```
