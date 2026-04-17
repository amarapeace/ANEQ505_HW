```
cp -r /pl/active/courses/2024_summer/maw_2024/raw_reads .
```

```
#launch an interactive session: 
ainteractive --ntasks=6 --time=02:00:00


#insert your code here to activate qiime. Hint: there should be 2 things you add here
module purge
module load qiime2/2024.10_amplicon
```


```
qiime tools import \--type EMPPairedEndSequences \--input-path raw_reads \--output-path cow_reads.qza
```

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

qiime demux emp-paired \--m-barcodes-file ../metadata/oxycow_barcodes.txt \--m-barcodes-column barcode \--p-rev-comp-mapping-barcodes \--p-rev-comp-barcodes \--i-seqs ../oxycow_reads.qza \--o-per-sample-sequences demux_oxycow.qza \--o-error-correction-details oxycow_demux_error.qza

#visualize the read quality
qiime demux summarize \--i-data demux_oxycow.qza \--o-visualization demux_oxycow.qzv
```

25541440

```
 dos2unix demux.sh
 sbatch demux.sh
```


```

```