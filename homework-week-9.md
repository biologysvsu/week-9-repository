# YOU NEED AN ACTIVE CONDA ENVIRONMENT TO RUN QIIME!
# GO TO WORKING DIRECTORY
```
myocean
#If you do not have an alias, you should go to your ocean folder using:
#cd /ocean/projects/agr250001p/your-username/week-7-repository
cd week-7-repository
```
- `ls` just in case to confirm you are in the right working directory. You should get something like this:
```
Blueberry_metadata_reduced.tsv  fastqc_mock_out      microbiome-instructions.md  multiqc_mock_data
MultiQC                         fastqc_raw_data_out  mock-data                   raw_data
README.md                       filt_stats.qza       multiqc_mock.html           reads_qza
```

# ACTIVATE CONDA ENVIRONMENT TO RUN QIIME2
```
module load anaconda3
conda activate qiime2-amplicon-2024.2
```
# Generate random sample you will work with
```
shuf -n 1 Blueberry_metadata_reduced.tsv
```
- This command will randomly select one sample from the dataset.
- Make sure you get a screenshot to include in your homework. Your output should look something like this:
```
BB207   Rhizosphere     Forest
```
Your sample ID is `BB207` (the first column). You will use this sample ID in your homework analysis.

# VISUALIZATION (Homework)
1. Generate rarefaction curves
```         
qiime diversity alpha-rarefaction \
  --i-table deblur_output/deblur_table_final.qza\
  --p-max-depth 7000 \
  --p-steps 20\
  --o-visualization deblur_output/rarefaction_curves_no_tree.qzv
```
- p-max-depth or sample depth is how many sequences (reads) from each sample will be used for comparison. We want to set this value to 7000 since the sample with fewest reads has 7704.
- p-steps 20 means that QIIME 2 will test 20 different sequencing depths when generating a rarefaction curve. QIIME 2 will analyze your data at 20 different depths from a low number (e.g., 250) up to 7000, to see how diversity changes.

- You need to "add, commit and push `deblur_output/rarefaction_curves_no_tree.qzv`" to your forked week-7-repository, download the file, and visualize it in qiime view.

2. Calculating diversity metrics
```         
qiime diversity core-metrics \
  --i-table deblur_output/deblur_table_final.qza \
  --p-sampling-depth 7000 \
  --m-metadata-file Blueberry_metadata_reduced.tsv \
  --p-n-jobs 4 \
  --output-dir diversity
```
3. Generate boxplot for shannon
```         
qiime diversity alpha-group-significance \
  --i-alpha-diversity diversity/shannon_vector.qza \
  --m-metadata-file Blueberry_metadata_reduced.tsv \
  --o-visualization diversity/shannon_compare_groups.qzv
```
- You need to "add, commit and push `diversity/shannon_compare_groups.qzv`" to your forked week-7-repository, download the file, and visualize it in qiime view.

4. Generate stacked bar chart of taxa relative abundances
```         
qiime taxa barplot \
  --i-table deblur_output/deblur_table_final.qza \
  --i-taxonomy taxa/classification.qza \
  --m-metadata-file Blueberry_metadata_reduced.tsv \
  --o-visualization taxa_barplot.qzv
```
- You need to "add, commit and push `taxa_barplot.qzv`" to your forked week-7-repository, download the file, and visualize it in qiime view.
