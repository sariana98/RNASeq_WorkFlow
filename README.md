# RNASeq_WorkFlow

Where I obtained the RNA-seq files: 

[Original paper](https://www.nature.com/articles/s41591-019-0368-8#change-history); 
[Supplemental data used](https://www.ebi.ac.uk/ena/browser/view/PRJEB25797?show=reads)

```
$ wget [file link address]
$ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR249/000/ERR2496060/ERR2496060_1.fastq.gz
```
To download all files without having to type ```wget``` indvidually for each one, write a bash script with a recursive function such as the one in ```download_data.sh```.

## 1. FastQC 

FastQC aims to provide a simple way to do some quality control checks on raw sequence data coming from high throughput sequencing pipelines such as RNA or ChIP sequencing data. It provides a quick overview of problematic areas of which you should be aware before doing any further analysis. This is done through an html report containing summary graphs and tables produced for each raw data file. 

To run fastqc on a dataset, first add FastQC to your path: 

On the Harmston server, this can be done by: 
```
$ export PATH=$PATH:/home/shared/software/FastQC 
```
Run fastqc from the directory containing the .fastq.gz files. FastQC will accept multiple file names as input, so we can use the ```*.fq``` wildcard:
```
$ fastqc [-o output dir] seqfile1 .. seqfileN
$ fastqc -o /home/sara/KRAS/fastqc_output/ *.fastq.gz
```

As this can take some time, create a screen session before running fastqc by typing ```screen -S [name of screen session]``` in the command line. Detach by typing ```ctrl+A+D```. fastqc will run in the background even when you're logged out from the server. To re-attach, type ```screen -r [name of screen session]```.

To view other options for fastqc, type ```fastqc --help``` in the command line. 

Once the output has been generated, use WinSCP (Windows) or ```rsync``` (Mac) to transfer the output files into your local machine. 

```
$ rsync [options] [source] [destination]
$ rsync -azvP sara@cbio.yale-nus.edu.sg:/home/sara/KRAS/fastqc_output/* ~/Desktop/KRAS/[folder_name]
```
The * indicates all files in the specified directory. 

**Note for Mac users**: You have to exit ```ssh``` to use ```rsync```. Also, depending on your machine, you may or or may not get an error message that says ```/etc/profile.d/lang.sh: line 19: warning: setlocale: LC_CTYPE: cannot change locale (UTF-8): No such file or directory``` but files will have been transferred - check local machine output folder to verify! 

[This](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) and [this](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/) are good resources to learn more about FastQC and the analysis modules found in the HTML report. 


## 2. Adapter trimming *(optional)*

We want to make sure that as many reads as possible map or align accurately to the genome. To ensure accuracy, only a small number of mismatches between the read sequence and the genome sequence are allowed, and any read with more than a few mismatches will be marked as being unaligned.

Therefore, to make sure that all the reads in the dataset have a chance to map/align to the genome, unwanted information can be trimmed off from every read, one read at a time. The types of unwanted information can include one or more of the following:

- leftover adapter sequences
- known contaminants (strings of As/Ts, other sequences)
- poor quality bases at read ends

You can get a sense of whether this step is needed by looking through the FastQC report, paying special attention to the *Overrepresented Sequences* section. 

I did not perform adapter trimming for this analysis because the quality of reads were good. Should you decide to perform this step, you can read more [here](https://hbctraining.github.io/Intro-to-rnaseq-hpc-O2/lessons/02_assessing_quality.html).  

## 3. STAR Workflow 

1. STAR with featureCounts

samtools sort

featureCounts: after obtaining the .txt files and downloading them on my machine, I don't know how I'm supposed to read them into R and convert them into read counts. 

2. STAR with RSEM

- I first prepared reference for RSEM using rsem-prepare-reference: I'm not sure whether I need to do this with or without the gtf file 
[prepare-reference](https://deweylab.github.io/RSEM/rsem-prepare-reference.html)

- rsem-calculate-expression: I get different errors depending on which reference I performed this with
[calculate-reference](https://deweylab.github.io/RSEM/rsem-calculate-expression.html#ARGUMENTS)

Making the data information matrix: I got the information from https://www.ncbi.nlm.nih.gov/Traces/study/?acc=ERP107752&o=acc_s%3Aa
