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
qiime --help
```
# VISUALIZATION (Homework)
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
