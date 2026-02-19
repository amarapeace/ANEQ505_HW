**ANEQ Homework #1: Practice Importing Data, Demultiplexing Reads, and Denoising**

**Due Feb 12th at midnight**

**Name:** Amara Onwunzo

**For complete credit for this assignment, you must answer all questions and include all commands in your obsidian upload.**

------------------------------------------------------------------------
**Learning Objectives**

1. Practice independently running the first few steps of a Qiime2 workflow We will focus on importing data, demultiplexing, and denoising for homework 1.

2. Practice recording commands and editing code to match your analysis

3. Push to GitHub for credit

------------------------------------------------------------------------


**Cow Site Dataset**

20 adult dairy cattle were sampled (via swabbing) to determine the microbial community composition between 5 different body sites (nasal/skin/udder/fecal/oral).  We want you to determine if and how the microbial composition changes between sites over the course of multiple homework assignments.

We will first begin by copying raw sequencing data from a public folder on Alpine into a new directory (that you create) on your Alpine account.

**Steps:**

1. Log into Alpine using OnDemand and create a new directory for this new analysis in your _scratch_ directory. Hint: it should look something like: /scratch/alpine/$USER/cow/
2. Move into your new directory using OnDemand
3. Create the following sub-directories using OnDemand: 
	1. slurm
	2. taxonomy
	3. tree
	4. taxaplots
	5. dada2
	6. demux
	7. metadata
	8. core_metrics

4. Download the cow_barcodes.txt and cow_metadata.txt files from Canvas and upload them both to your metadata folder within your new cow directory. So, your filepath should look something like this: /scratch/alpine/$USER/cow/metadata

5. On OnDemand, go to your cow directory and open a new terminal

6. Copy the raw sequencing files from this public folder to **your new folder** using the terminal. Do **not** change the names of these files and folders. Hint: make sure you are in your new cow folder before you run this code (this will copy over the whole folder):  

```
cp -r /pl/active/courses/2024_summer/maw_2024/raw_reads .
```


5.    Launch an interactive session and load qiime2 within your cow directory. 

```
#launch an interactive session: 
ainteractive --ntasks=6 --time=02:00:00


#insert your code here to activate qiime. Hint: there should be 2 things you add here
module purge
module load qiime2/2024.10_amplicon

```


6.    Import the sequences/reads into a Qiime2-readable format (.qza). Note this might take 10-20 mins

```
qiime tools import \--type EMPPairedEndSequences \--input-path raw_reads \--output-path cow_reads.qza
```


7.    Demultiplex the reads by submitting a job. Note this may take ~30 mins

a.    Go into your slurm directory using OnDemand. Create a new file named **demux.sh** so you can submit a job that will demultiplex your sequences quicker. Fill in the lines that need editing (denoted by capital letters or hashes) to this demultiplexing command and add that to your new script. 


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
cd /scratch/alpine/$USER/cow/demux

#Below is the command you will run to demultiplex the samples.

qiime demux emp-paired \--m-barcodes-file ../metadata/cow_barcodes.txt \--m-barcodes-column barcode \--p-rev-comp-mapping-barcodes \--p-rev-comp-barcodes \--i-seqs ../cow_reads.qza \--o-per-sample-sequences demux_cow.qza \--o-error-correction-details cow_demux_error.qza

#visualize the read quality
qiime demux summarize \--i-data demux_cow.qza \--o-visualization demux_cow.qzv
```


 Run the script in your slurm directory as a job using: 
 ```
 dos2unix demux.sh
 sbatch demux.sh

 ```

8.    Denoise

Fill in the blank to denoise your samples based on what you think should be trimmed (from the front of the reads) or truncated (from the ends of the reads) based on the demux_cow.qzv file. You can run this in the terminal or as a job.

```
cd /scratch/alpine/$USER/cow/dada2

qiime dada2 denoise-paired \--i-demultiplexed-seqs ../demux/demux_cow.qza \--p-trim-left-f 0 \--p-trim-left-r 0 \--p-trunc-len-f 250 \--p-trunc-len-r 250 \--o-representative-sequences cow_seqs_dada2.qza \--o-denoising-stats cow_dada2_stats.qza \--o-table cow_table_dada2.qza

#Visualize the denoising results:
qiime metadata tabulate \--m-input-file cow_dada2_stats.qza \--o-visualization cow_dada2_stats.qzv

qiime feature-table summarize \--i-table cow_table_dada2.qza \--m-sample-metadata-file ../metadata/cow_metadata.txt \--o-visualization cow_table_dada2.qzv

qiime feature-table tabulate-seqs \--i-data cow_seqs_dada2.qza \--o-visualization cow_seqs_dada2.qzv

```

	
Briefly **describe** the key information from each denoising output file:
1. Representative Sequences: This file presents the DNA sequences of the microbes found in the samples taken. After denoising, it shows 4653 unique sequences with a mean length of 250 base pairs and maximum of 427 base pairs.
2. Denoising Stats: It shows the performance of the samples during the denoising step. Statistics like, how many reads passed through filter, how many merged, how many removed as chimeras. It tells you the quality of your data. 
3. Denoised Table: It is a table that shows the abundance of each sequence in the samples. From this table, there are a total of 4653 features, with 

**Answer the following questions:**  
1. Where does the median Q-score begin to dip below Q30 for the forward reads and the reverse reads? For the forward reads, it stayed above Q30 throughout. While for reverse reads, it dipped after 250 to 13.
2. What is the mean reads per sample? 15163.394558
3. How long are the reads? 251 bp
4. What is the maximum length of all your sequences? 427
5. Which sample (not including extraction controls starting with EC) lost the highest % of reads? 2019.3.14.cow.oral.20. It lost 84.47% of its reads, so only 15.53% of reads passed the filter.
6. Why did you chose to trim or truncate where you did? I didn't trim because all the reads at the beginning for both forward and reverse reads, have Q-score above 30. I truncated at 250 for both forward and reverse reads because up to that point, the median Q-score stayed above Q30 for both. After 250, the reverse reads Q-score for the 251st dropped to 13 which is below the accepted benchmark.

**To submit your homework from this document:**
write all of your commands here, then use command+P (for mac) or control+P (for windows) and search Git: commit. click it. then search for Git: Push and click it. go to your github online to check that it pushed correctly. we will check your github for homework credit. 