~={red}(1point)=~ for Alpha Rarefaction Plot
Run Core Metrics ~={red}(1 point; .25pts per line)=~
Make alpha diversity plots ~={red}(3points)=~
~={red}10 points=~ for the questions 

~={red}15 points total=~
------------------------------------------------------------------

Due: 

**For complete credit for this assignment, you must answer all questions and include all commands in your obsidian upload.**

------------------------------------------------------------------
**Learning Objectives**
1. Practice recording commands and editing code to match your analysis.
2. Perform alpha rarefaction and determine an appropriate sequencing depth.
3. Run core metrics, generate plots for alpha and beta diversity
--------------------------------------------------

**Cow Site Data Workflow**, part 3

Load qiime2 in a terminal session after you go into the **cow** folder

```
# Insert the two commands to activate qiime2
module purge
module load qiime2/2024.10_amplicon


```

### Alpha Rarefaction Plot ~={red}(1 point)=~
- Chose the input sequencings depths (min and max) for generating the alpha rarefaction plot: 

```
#go to the cow directory

qiime diversity alpha-rarefaction \--i-table dada2/cow_table_dada2_filtered300.qza \--m-metadata-file metadata/cow_metadata.txt \--o-visualization alpha_rarefaction_curves_16S.qzv \--p-min-depth ADD MIN RAREFACTION DEPTH \--p-max-depth ADD MAX RAREFACTION DEPTH
```


### Run Core Metrics ~={red}(1 point)=~

```
qiime diversity core-metrics-phylogenetic \--i-table INSERT FILTERED TABLE HERE \--i-phylogeny INSERT FILE HERE \--m-metadata-file INSERT FILE HERE \--p-sampling-depth INSERT SEQ DEPTH HERE \--output-dir core_metrics_results
```


### Visualize alpha diversity plots
- generate a plot to visualize the observed features ~={red}(1 point)=~
```
qiime diversity alpha-group-significance \--i-alpha-diversity core_metrics_results/FILENAME.qza \--m-metadata-file metadata/cow_metadata.txt \--o-visualization core_metrics_results/OUTPUT-FILENAME.qzv
```

- generate a plot to visualize faith's PD ~={red}(2 points)=~
```
## insert the entire code chunk for generating this visualization 
qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results/shannon_vector.qza \
--m-metadata-file metadata/cow_metadata.txt \
--o-visualization core_metrics_results/shannon_statistics.qzv  
  
qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results/faith_pd_vector.qza \
--m-metadata-file metadata/cow_metadata.txt \
--o-visualization core_metrics_results/faiths_pd_statistics.qzv

```



## Homework questions ~={red}(10 points)=~

1. what is the name of the file you needed to use to figure out what min and max depths to use to generate the alpha rarefaction plot? (Hint: which file contains the sequencing depths for each sample) 
2cow_table_dada2_filtered300.qzv

2. what did you choose for the rarefaction depth (the input for core metrics -p-sampling-depth flag)? why? 

1500 reads. Because this depth retains most of the samples. Looking at the rarefaction curves too, the graph plateaus around that depth.

3. Which cow body location had more observed features? Which has the lowest? 

Fecal samples have the highest. Nasal samples have the lowest with exception of the controls.

4. What is the main difference between Faiths PD and Shannons alpha diversity metrics? 
 
Faiths PD measures diversity using phylogenetic relationships between organisms while Shannons do not use phylogenetic relationship.  

5. Which diversity metrics produced by the core-metrics pipeline require phylogenetic information? 

Faith’s Phylogenetic Diversity (Faith’s PD), Unweighted UniFrac, and Weighted UniFrac

7. Which two body sites have the highest Faiths PD alpha diversity?  Are the groups significantly different? 

Fecal and skin. They are significantly different based on Kruskal-Wallis pairwise test with p-value 0.000182.

7. Does it seem like there are any groupings in the beta diversity? What are the groupings? 

Yes, there are groupings observed in the beta diversity plots. There are clusters based on body site. Nasal and oral samples cluster together while skin and udder samples cluster together. Fecal samples are distinct on their own.

8. Why do you think these samples are grouping together? 
 The samples clustering together come from body sites that are close together and share similar microbial communities. For instance, nasal and oral samples are close together and are associated with upper respiratory functions. Also skin and udder are external body surfaces.

9. What test can you run to determine if the groups are significantly different?PERMANOVA

10. What command would you use to run that test?

```
#insert command for running the test you suggest from question 7

#permanova

qiime diversity beta-group-significance \
--i-distance-matrix core_metrics_results/unweighted_unifrac_distance_matrix.qza \
--m-metadata-file metadata/cow_metadata.txt \
--m-metadata-column body_site \
--p-method permanova \
--o-visualization core_metrics_results/unweighted_unifrac_body_site_metric.qzv

#braycurtis
qiime diversity beta-group-significance \
--i-distance-matrix core_metrics_results/bray_curtis_distance_matrix.qza \
--m-metadata-file metadata/cow_metadata.txt \
--m-metadata-column body_site \
--p-method permanova \
--o-visualization core_metrics_results/bray_curtis_body_site_metric.qzv
```