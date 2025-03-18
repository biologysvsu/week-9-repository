# YOU NEED AN ACTIVE CONDA ENVIRONMENT TO RUN QIIME!
# GO TO WORKING DIRECTORY
```
my ocean
cd week-7-repository
```
- `ls` just in case to confirm you are in the right working directory

# ACTIVATE CONDA ENVIRONMENT TO RUN QIIME2



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
