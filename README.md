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

**Adapter trimming** *(optional)*

We want to make sure that as many reads as possible map or align accurately to the genome. To ensure accuracy, only a small number of mismatches between the read sequence and the genome sequence are allowed, and any read with more than a few mismatches will be marked as being unaligned.

Therefore, to make sure that all the reads in the dataset have a chance to map/align to the genome, unwanted information can be trimmed off from every read, one read at a time. The types of unwanted information can include one or more of the following:

- leftover adapter sequences
- known contaminants (strings of As/Ts, other sequences)
- poor quality bases at read ends

You can get a sense of whether this step is needed by looking through the FastQC report, paying special attention to the *Overrepresented Sequences* section. 

I did not perform adapter trimming for this analysis because the quality of reads were good. Should you decide to perform this step, you can read more [here](https://hbctraining.github.io/Intro-to-rnaseq-hpc-O2/lessons/02_assessing_quality.html).  

## 2. STAR-RSEM Pipeline 

### Aligning reads to transcriptome using STAR
```
STAR --genomeDir [path to STAR Index] --readFilesCommand zcat --readFilesIn [read1] [read2] --runThreadN [threads] --sjdbGTFfile [path to GTF file] --twopassMode Basic --outWigType bedGraph --quantMode TranscriptomeSAM --outSAMtype BAM SortedByCoordinate --outFileNamePrefix [path/output filename]

STAR --genomeDir /home/shared/genomes/hg38/StarIndex/ --readFilesCommand zcat --readFilesIn ERR2496065_1.fastq.gz ERR2496065_2.fastq.gz --runThreadN 10 --sjdbGTFfile /home/sara/genomes/Homo_sapiens.GRCh38.100.chr.gtf --twopassMode Basic --outWigType bedGraph --quantMode TranscriptomeSAM --outSAMtype BAM SortedByCoordinate --outFileNamePrefix /home/sara/KRAS/STAR.transcriptome/ERR2496065
```

**Note** that the STAR ```--quantMode TranscriptomeSAM``` option will output alignments translated into transcript coordinates in the ```Aligned.toTranscriptome.out.bam``` file (in addition to alignments in genomic coordinates in ```Aligned.*.sam/bam``` files). These transcriptomic alignments can be used with
various transcript quantification software that require reads to be mapped to transcriptome, such as RSEM. 

For more parameters, check out the STAR [manual](https://physiology.med.cornell.edu/faculty/skrabanek/lab/angsd/lecture_notes/STARmanual.pdf).

### Quantifying reads using RSEM

RSEM is a software package for estimating gene and isoform expression levels from single-end or paired-end RNA-Seq data. If an annotated reference genome is available, RSEM can use a gtf file representation of those annotations to extract the transcript sequences for which quantification will be performed, and build the relevant genome and transcriptome indices.

#### Prepare reference 

Before calculating expression, prepare a reference for RSEM with a GTF file. 

```
$ rsem-prepare-reference --gtf [path to gft file] [path to FASTA file] [path/reference filename] 
$ rsem-prepare-reference --gtf /home/sara/genomes/Homo_sapiens.GRCh38.100.chr.gtf /home/shared/genomes/hg38/hg38.fa /home/sara/genomes/ref/hg38
```
For more parameters, refer to prepare-reference [manual](https://deweylab.github.io/RSEM/rsem-prepare-reference.html)

#### Calculate expression 

```
$ rsem-calculate-expression --paired-end --bam --strandedness reverse --alignments [path/BAM filename] [path/reference filename] [path/output filename] --no-bam-output -p [threads]
$ /usr/local/bin/rsem-calculate-expression --paired-end --bam --strandedness reverse --alignments ERR2496065Aligned.toTranscriptome.out.bam /home/sara/genomes/ref/hg38 /home/sara/KRAS/rsem.output/ERR2496065Aligned.results --no-bam-output -p 10
```
For more parameters, refer to rsem-calculate-expression [manual](https://deweylab.github.io/RSEM/rsem-calculate-expression.html#ARGUMENTS).

## 3. Reading DE into R

Making the data information matrix: I got the information on the individual RNA-seq files from [here](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=ERP107752&o=acc_s%3Aa).

## Alternative Option (?): STAR-featureCounts Pipeline 

Today, we will be using the featureCounts tool to get the gene counts. It counts reads that map to a single location (uniquely mapping) and follows the scheme in the figure below for **assigning reads to a gene/exon**." Follow this [training](https://github.com/hbctraining/Intro-to-rnaseq-hpc-O2/blob/master/lessons/05_counting_reads.md).

"When assigning reads to genes or exons, most reads can be successfully assigned without
ambiguity. However if reads are to be assigned to transcripts, due to the high overlap between
transcripts from the same gene, many reads will be found to overlap more than one transcript
and therefore cannot be uniquely assigned. Specialized transcript-level quantification tools
are recommended for counting reads to transcripts. Such tools use model-based approaches
to deconvolve reads overlapping with multiple transcripts." - Subread [manual](http://subread.sourceforge.net/SubreadUsersGuide.pdf)

```
samtools sort [-@ threads] [in.bam]
samtools sort -@ 10 ERR2496065.sortedByCoord.out.bam > ERR2496065.sorted.bam
```
For more parameters, check out the samtools sort [manual](http://www.htslib.org/doc/samtools-sort.html)

```
featureCounts -p -B -t exon -g gene_id -a [GTF filename] -T [threads] -s 2 -o [output filename] [input filename]
featureCounts -p -B -t exon -g gene_id -a /home/sara/genomes/Homo_sapiens.GRCh38.100.chr.gtf -T 10 -s 2 -o /home/sara/KRAS/counts/ERR2496065_counts.txt ERR2496065.sorted.bam
```

Subread [manual](http://subread.sourceforge.net/SubreadUsersGuide.pdf)


featureCounts: after obtaining the .txt files and downloading them on my machine, I don't know how I'm supposed to read them into R and convert them into read counts.
