
Grab the raw reads from an online repository
```
cp -r /pl/active/courses/2024_summer/maw_2024/raw_reads .
```

Launch an interactive session, purge and activate qiime2
```
#launch an interactive session: 
ainteractive --ntasks=6 --time=02:00:00


#insert your code here to activate qiime. Hint: there should be 2 things you add here
module purge
module load qiime2/2024.10_amplicon
```

Import the raw sequence data that are currently fasta.q files into .qza format readable in qiime
```
qiime tools import \--type EMPPairedEndSequences \--input-path raw_reads \--output-path oxycow_reads.qza
```
``
Debug the barcodes file
**Sample IDs in oxy_barcode.txt contain "/" and QIIME doesnt like it**
```
# checking to see if the SampleIDs contain "/"

head oxy_barcodes.txt

# code to make a cleaned copy
awk 'BEGIN{FS=OFS="\t"} 
NR==1 {print; next} 
{
  gsub(/\//, "_", $1)
  gsub(/:/, "_", $1)
  gsub(/ /, "", $1)
  print
}' oxycow_barcodes0.txt > oxycow_barcodes.txt


# check the copy

  

cut -f1 oxycow_barcodes.txt | head

cut -f1 oxycow_barcodes.txt | grep '/'

cut -f1 oxycow_barcodes.txt | grep ':'

cut -f1 oxycow_barcodes.txt | grep ' '

  

# Those last three should return nothing.
```

debug

```
head oxycow_metadata0.txt

awk 'BEGIN{FS=OFS="\t"}
NR==1 {print; next}
{
  split($1,a,"_")
  $1 = a[1]"_"a[2]"_"a[3]"_2025_"a[4]"_00"a[5]
  print
}' oxycow_metadata0.txt > oxycow_metadata.txt


head oxycow_metadata.txt



sed -E 's/([0-9]+_[0-9]+_[0-9]+_2025_)([0-9]+)(AM|PM)_00/\1\2_00\3/' oxycow_metadata1.txt > oxycow_metadata.txt

cut -f1 oxycow_metadata.txt | head -20
```
Demultiplex the qza formatted raw reads. Now you need that barcode.txt file in addition to the raw reads qza file and you get a demux qza file, which you then convert to a visualizable qzv format. The following code is the content of the slurm file created to run this demultiplexing and conversion.
```
#!/bin/bash
#SBATCH --job-name=demux
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=amara.onwunzo@colostate.edu

module purge
module load qiime2/2024.10_amplicon

#change the following line if your file path looks different
cd /scratch/alpine/$USER/oxycow/demux

#Below is the command you will run to demultiplex the samples.

qiime demux emp-paired \--m-barcodes-file ../metadata/oxycow_barcodes.txt \--m-barcodes-column Barcode \--p-rev-comp-mapping-barcodes \--p-rev-comp-barcodes \--i-seqs ../oxycow_reads.qza \--o-per-sample-sequences demux_oxycow.qza \--o-error-correction-details oxycow_demux_error.qza

#visualize the read quality
qiime demux summarize \--i-data demux_oxycow.qza \--o-visualization demux_oxycow.qzv
```

Now submit the slurm file.
```
 dos2unix demux.sh
 sbatch demux.sh
```

25541440 - failed
25541913 - Failed again 
25541967 - changed the barcode column name from barcode to Barcode in the slurm file because that's how it appears in the barcode metadata file.

26557223

Visualize the qzv file generated from the previous code and determine the trimming and truncation parameters you need for the following code. Here we are trying to remove reads of unreasonable length, typically > 250bp
```
cd /scratch/alpine/$USER/oxycow/dada2

qiime dada2 denoise-paired \--i-demultiplexed-seqs ../demux/demux_oxycow.qza \--p-trim-left-f 0 \--p-trim-left-r 0 \--p-trunc-len-f 250 \--p-trunc-len-r 250 \--o-representative-sequences oxycow_seqs_dada2.qza \--o-denoising-stats oxycow_dada2_stats.qza \--o-table oxycow_table_dada2.qza

#Visualize the denoising results:
qiime metadata tabulate \--m-input-file oxycow_dada2_stats.qza \--o-visualization oxycow_dada2_stats.qzv

qiime feature-table summarize \--i-table oxycow_table_dada2.qza \--m-sample-metadata-file ../metadata/oxycow_metadata.txt \--o-visualization oxycow_table_dada2.qzv

qiime feature-table tabulate-seqs \--i-data oxycow_seqs_dada2.qza \--o-visualization oxycow_seqs_dada2.qzv
```

To make it a job
```
#!/bin/bash
#SBATCH --job-name=dada2
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=amara.onwunzo@colostate.edu

module purge
module load qiime2/2024.10_amplicon

#change the following line if your file path looks different
cd /scratch/alpine/$USER/oxycow/dada2

#Below is the command you will run to denoise the samples.

qiime dada2 denoise-paired \--i-demultiplexed-seqs ../demux/demux_oxycow.qza \--p-trim-left-f 0 \--p-trim-left-r 0 \--p-trunc-len-f 250 \--p-trunc-len-r 250 \--o-representative-sequences oxycow_seqs_dada2.qza \--o-denoising-stats oxycow_dada2_stats.qza \--o-table oxycow_table_dada2.qza


#visualize the read quality
qiime metadata tabulate \--m-input-file oxycow_dada2_stats.qza \--o-visualization oxycow_dada2_stats.qzv
```

To submit
```
dos2unix dada2.sh
sbatch dada2.sh
```
26560818



### Remove long (300+ base pair) amplicons from the representative sequences file and the feature table
```
# filter out any large amplicons from the seqs and table (because they are contaminates)

cd /scratch/alpine/$USER/oxycow/dada2

qiime feature-table filter-seqs \--i-data oxycow_seqs_dada2.qza \--m-metadata-file oxycow_seqs_dada2.qza \--p-where 'length(sequence) < 300' \--o-filtered-data oxycow_seqs_dada2_filtered300.qza

qiime feature-table tabulate-seqs \--i-data oxycow_seqs_dada2_filtered300.qza \--o-visualization oxycow_seqs_dada2_filtered300.qzv

qiime feature-table filter-features \--i-table oxycow_table_dada2.qza \--m-metadata-file oxycow_seqs_dada2_filtered300.qza \--o-filtered-table oxycow_table_dada2_filtered300.qza

qiime feature-table summarize \--i-table oxycow_table_dada2_filtered300.qza \--m-sample-metadata-file ../metadata/oxycow_metadata.txt \--o-visualization oxycow_table_dada2_filtered300.qzv
```
```


Then taxonomy plots


```
```
cd /scratch/alpine/$USER/oxycow/taxonomy
```

```
wget --no-check-certificate https://ftp.microbio.me/greengenes_release/2024.09/2024.09.backbone.v4.nb.qza
```

Use greengenes to classify our data
```
qiime feature-classifier classify-sklearn \--i-reads ../dada2/oxycow_seqs_dada2_filtered300.qza \--i-classifier 2024.09.backbone.v4.nb.qza \--o-classification taxonomy_gg2_filtered.qza
```
visualize the output
```
qiime metadata tabulate \--m-input-file taxonomy_gg2_filtered.qza \--o-visualization taxonomy_gg2_filtered.qzv
```

Filter mitochondria and the rest
```
qiime taxa filter-table \--i-table ../dada2/oxycow_table_dada2_filtered300.qza \--i-taxonomy taxonomy_gg2_filtered.qza \--p-exclude mitochondria,chloroplast,sp004296775 \--p-include c__ \--o-filtered-table ../dada2/table_nomitochloro_gg2_filtered300.qza
```

visualize it
```
qiime taxa barplot \--i-table ../dada2/table_nomitochloro_gg2_filtered300.qza \--i-taxonomy taxonomy_gg2_filtered.qza \--m-metadata-file ../metadata/oxycow_metadata.txt \--o-visualization ../taxaplots/taxa_barplot_nomitochloro_gg2_filtered300.qzv
```

Get the controls

```
qiime feature-table filter-samples \--i-table ../dada2/oxycow_table_dada2.qza \--m-metadata-file ../metadata/oxycow_metadata.txt \--p-where "[Treatment]='ext_control'" \--o-filtered-table ../dada2/table_controls.qza
```

Controls taxa baxplot

```
qiime taxa barplot \--i-table ../dada2/table_controls.qza \--i-taxonomy ../taxonomy/taxonomy_gg2_filtered.qza \--m-metadata-file ../metadata/oxycow_metadata.txt \--o-visualization taxa_barplot_controls.qzv
```

Tree

```
#!/bin/bash
#SBATCH --job-name=tree
#SBATCH --nodes=1
#SBATCH --ntasks=8
#SBATCH --partition=amilan
#SBATCH --time=04:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=amara.onwunzo@colostate.edu
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal

#Activate qiime

module purge  
module load qiime2/2024.10_amplicon


#Get reference
wget --no-check-certificate -P ../tree https://ftp.microbio.me/greengenes_release/2022.10/2022.10.backbone.sepp-reference.qza


#Command
qiime fragment-insertion sepp \--i-representative-sequences ../dada2/oxycow_seqs_dada2_filtered300.qza \--i-reference-database ../tree/2022.10.backbone.sepp-reference.qza \--o-tree ../tree/tree_gg2.qza \--o-placements ../tree/tree_placements_gg2.qza
```

make plots with the full data
```
qiime taxa filter-table \--i-table ../dada2/oxycow_table_dada2.qza \--i-taxonomy ../taxonomy/taxonomy_gg2_filtered.qza \--p-exclude mitochondria,chloroplast,sp004296775 \--p-include c__ \--o-filtered-table ../dada2/table_nomitochloro.qza
```
Not correct, to do this, you need taxonomy_gg2, not filtered. And this means to make a fresh classification file using the unfiltered sequence file 

```
cd taxonomy

qiime feature-classifier classify-sklearn \--i-reads ../dada2/oxycow_seqs_dada2.qza \--i-classifier 2024.09.backbone.v4.nb.qza \--o-classification taxonomy_gg2.qza
```
Then use it to make the plot 
```
qiime taxa filter-table \--i-table ../dada2/oxycow_table_dada2.qza \--i-taxonomy ../taxonomy/taxonomy_gg2.qza \--p-exclude mitochondria,chloroplast,sp004296775 \--p-include c__ \--o-filtered-table ../dada2/table_nomitochloro.qza
```

visualize the plot
```
qiime taxa barplot \--i-table ../dada2/table_nomitochloro.qza \--i-taxonomy ../taxonomy/taxonomy_gg2.qza \--m-metadata-file ../metadata/oxycow_metadata.txt \--o-visualization taxa_barplot_all_samples.qzv

```

Filter out the controls
```
cd ..

qiime feature-table filter-samples \--i-table dada2/table_nomitochloro.qza \--m-metadata-file metadata/oxycow_metadata.txt \--p-where "NOT [Treatment] IN ('ext_control') " \--o-filtered-table dada2/table_nomitochloro_nocontrol.qza
```

Now, run alpha-rarefaction

```
cd alpha_rarefaction  
  
qiime diversity alpha-rarefaction \--i-table ../dada2/table_nomitochloro_nocontrol.qza \--m-metadata-file ../metadata/oxycow_metadata.txt \--p-max-depth 10000 \--o-visualization alpha_rarefaction_curves.qzv
```
run core metrics
```
qiime diversity core-metrics-phylogenetic \--i-table dada2/table_nomitochloro_nocontrol.qza \--i-phylogeny tree/tree_gg2.qza \--m-metadata-file metadata/oxycow_metadata.txt \--p-sampling-depth 1500 \--output-dir core-metrics-results
```
observed features
```
qiime diversity alpha-group-significance \--i-alpha-diversity core-metrics-results/observed_features_vector.qza \--m-metadata-file metadata/oxycow_metadata.txt \--o-visualization core-metrics-results/observed_features_statistics.qzv
```

shannon statistics
```
qiime diversity alpha-group-significance \--i-alpha-diversity core-metrics-results/shannon_vector.qza \--m-metadata-file metadata/oxycow_metadata.txt \--o-visualization core-metrics-results/shannon_statistics.qzv
```

faiths richness
```
qiime diversity alpha-group-significance \--i-alpha-diversity core-metrics-results/faith_pd_vector.qza \--m-metadata-file metadata/oxycow_metadata.txt \--o-visualization core-metrics-results/faiths_pd_statistics.qzv
```

alpha-corelation

```
qiime diversity alpha-correlation \--i-alpha-diversity core-metrics-results/faith_pd_vector.qza \--m-metadata-file metadata/oxycow_metadata.txt \--o-visualization core-metrics-results/faith_pd_correlation_statistics.qzv
```