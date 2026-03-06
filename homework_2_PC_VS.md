**ANEQ Class Assignment #2: Practice Qiime2 Workflow**

Due: March 5th 2026 at midnight

**For complete credit for this assignment, you must answer all questions and include all commands in your obsidian upload.**

------------------------------------------------------------------
**Learning Objectives**
1. Practice independently running a partial Qiime2 workflow on new data using both the command line and slurm to submit jobs. 
2. Practice recording commands and editing code to match your analysis.
3. Perform taxonomic assignments using greengenes2, filtering out unwanted taxonomy and large amplicons, run another job script for generating a SEPP tree.
--------------------------------------------------

**Cow Site Data Workflow**, part 2

Load qiime2 in a terminal session after you go into the taxonomy folder 

```
# Insert the two commands to activate qiime2
module purge
module load qiime2/2024.10_amplicon


```

### Remove long (300+ base pair) amplicons from the representative sequences file and the feature table
```
# filter out any large amplicons from the seqs and table (because they are contaminates)

cd /scratch/alpine/$USER/cow/dada2

qiime feature-table filter-seqs \--i-data cow_seqs_dada2.qza \--m-metadata-file cow_seqs_dada2.qza \--p-where 'length(sequence) < 300' \--o-filtered-data cow_seqs_dada2_filtered300.qza

qiime feature-table tabulate-seqs \--i-data cow_seqs_dada2_filtered300.qza \--o-visualization cow_seqs_dada2_filtered300.qzv

qiime feature-table filter-features \--i-table cow_table_dada2.qza \--m-metadata-file cow_seqs_dada2_filtered300.qza \--o-filtered-table cow_table_dada2_filtered300.qza

qiime feature-table summarize \--i-table cow_table_dada2_filtered300.qza \--m-sample-metadata-file ../metadata/cow_metadata.txt \--o-visualization cow_table_dada2_filtered300.qzv
```


### Classify taxonomy using GreenGenes2

First get the Greengenes2 database:

```
cd /scratch/alpine/$USER/cow/taxonomy
```

```
wget --no-check-certificate https://ftp.microbio.me/greengenes_release/2024.09/2024.09.backbone.v4.nb.qza
```

Classify taxonomy using GreenGenes2 classify the ASVs (takes about 5 mins). ~={red}(1point)=~
```
qiime feature-classifier classify-sklearn \--i-reads ../dada2/cow_seqs_dada2_filtered300.qza \--i-classifier 2024.09.backbone.v4.nb.qza \--o-classification taxonomy_gg2_filtered.qza
```

Visualize the taxonomy of your ASVs: (~={red}1point)=~
```
qiime metadata tabulate \--m-input-file taxonomy_gg2_filtered.qza \--o-visualization taxonomy_gg2_filtered.qzv
```

- Filter mitochondria and chloroplast out to generate a filtered feature table, keep only ASVs with a class or lower taxonomy. fill in the blank (--p-exclude) to exclude these DNA. Fill in the blank to include only class level or below classifications. ~={red}(1point)=~
```
qiime taxa filter-table \--i-table ../dada2/cow_table_dada2_filtered300.qza \--i-taxonomy taxonomy_gg2.qza \--p-exclude mitochondria,chloroplast,sp004296775 \--p-include c__ \--o-filtered-table ../dada2/table_nomitochloro_gg2_filtered300.qza
```

- Visualize the taxa bar plot
```
qiime taxa barplot \--i-table ../dada2/table_nomitochloro_gg2_filtered300.qza \--i-taxonomy taxonomy_gg2_filtered.qza \--m-metadata-file ../metadata/cow_metadata.txt \--o-visualization ../taxaplots/taxa_barplot_nomitochloro_gg2_filtered300.qzv
```

## Filtered Taxa Bar Plot Questions ~={red}(10 points)=~

**Question 1**: Attach a picture of your taxa bar plot, organized by cow sampling location (body_site) at the level 7 taxonomic level. What general trends do you notice? 

![[Pasted image 20260227150352.png]]

**_Question 2**: What are the top 2 most abundant bacterial **classes** in the fecal samples? 
Bacteroidia and Clostridia

**_Question 3**: What highly abundant ASV is shared between both the udder and skin samples?
d__Bacteria;p__Bacillota_A_368345;c__Clostridia_258483
Faecousia sp000434635
d__Bacteria;p__Bacillota_A_368345;c__Clostridia_258483;o__Oscillospirales;f__Oscillospiraceae_88309;g__Faecousia;s__Faecousia sp000434635

**_Question 4**: Which samples (still sorted by body_site) have higher alpha diversity in terms of observed features? Skin and fecal samples. Followed by udder

**Question 5**: do all samples contain archaea as well? No, they are present in skin and udder samples but not in everything

**Question 6**: why do we filter out sp004296775? Because we are uncertain of its taxonomic status.

**Question 7**: what is the difference between these two flags? 
--p-exclude mitochondria,chloroplast,sp004296775 \
--p-include c__ \
The first one (-exclude) filters out records that have the strings of text following the exclude flag (mitochondria,chloroplast,sp004296775) 
The second one (--p-include c__ \ ) says to include only the records that have taxa up to class level.

**Question 8**: do the positive controls look the same as each other? Yes or No? Yes

**Question 9**: Do the negative/extraction controls (Samples labeled as EC), look like the positive controls? Yes or no? No

**Question 10**: do the negative/extraction controls (Samples labeled as EC), look like the real samples? Yes or no? No

## Phylogenetic tree ~={red}(1 point)=~
Create a job script to run the phylogenetic tree building. Remember you must start a new terminal session, navigate to your slurm directory, and then submit the job. You do NOT need to start any other interactive sessions. This job will take about an hour. 

Go to OnDemand and create a new text file for your job script
```
nano phylogen.sh
```

```
#!/bin/bash
#SBATCH --job-name=tree
#SBATCH --nodes=1
#SBATCH --ntasks=8
#SBATCH --partition=amilan
#SBATCH --time=20:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=amara.onwunzo@colostate.edu
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal

#Activate qiime

module purge  
module load qiime2/2024.10_amplicon
#Insert the two commands you need to load qiime2


#Get reference
wget --no-check-certificate -P ../tree https://ftp.microbio.me/greengenes_release/2022.10/2022.10.backbone.sepp-reference.qza


#Command
qiime fragment-insertion sepp \--i-representative-sequences ../dada2/cow_seqs_dada2_filtered300.qza \--i-reference-database ../tree/2022.10.backbone.sepp-reference.qza \--o-tree ../tree/tree_gg2.qza \--o-placements ../tree/tree_placements_gg2.qza
```

- submit the job from the terminal
```
#submit the job
dos2unix phylogen.sh
sbatch phylogen.sh
```
We will use this file in the next homework!
24454144
### Once this job finishes, copy and paste what the slurm email says here ~={red}(1 point)=~: 

#### for example: 
ob ID: 24454144
Cluster: alpine
User/Group: c837102365@colostate.edu/c837102365pgrp@colostate.edu
State: COMPLETED (exit code 0)
Nodes: 1
Cores per node: 8

-------- CPU Metrics --------
CPU Utilized: 03:15:14
CPU Efficiency: 11.97% of 1-03:10:24 core-walltime
Job Wall-clock time: 03:23:48
Memory Utilized: 8.56 GiB
Memory Efficiency: 28.52% of 30.00 GiB (3.75 GiB/core)
