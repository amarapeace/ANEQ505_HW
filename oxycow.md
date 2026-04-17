
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
qiime tools import \--type EMPPairedEndSequences \--input-path raw_reads \--output-path cow_reads.qza
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

Visualize the qzv file generated from the previous code and determine the trimming and truncation parameters you need for the following code. Here we are trying to remove reads of unreasonable length, typically > 250bp
```
cd /scratch/alpine/$USER/oxycow/dada2

qiime dada2 denoise-paired \--i-demultiplexed-seqs ../demux/demux_oxycow.qza \--p-trim-left-f 0 \--p-trim-left-r 0 \--p-trunc-len-f 250 \--p-trunc-len-r 250 \--o-representative-sequences oxycow_seqs_dada2.qza \--o-denoising-stats oxycow_dada2_stats.qza \--o-table oxycow_table_dada2.qza

#Visualize the denoising results:
qiime metadata tabulate \--m-input-file oxycow_dada2_stats.qza \--o-visualization oxycow_dada2_stats.qzv

qiime feature-table summarize \--i-table oxycow_table_dada2.qza \--m-sample-metadata-file ../metadata/oxycow_metadata.txt \--o-visualization oxycow_table_dada2.qzv

qiime feature-table tabulate-seqs \--i-data oxycow_seqs_dada2.qza \--o-visualization oxycow_seqs_dada2.qzv
```