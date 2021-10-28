# RNASeq_WorkFlow

## 1. FastQC 

FastQC aims to provide a simple way to do some quality control checks on raw sequence data coming from high throughput sequencing pipelines such as RNA or ChIP sequencing data. It provides a quick overview to tell you in which areas there may be problems of which you should be aware before doing any further analysis. This is done through an html report containing summary graphs and tables produced for each raw data file. 

To run fastqc on a dataset, you might need to first add FastQC to your path: 

On the Harmston server, this can be done by: 
```
$ export PATH=$PATH:/home/shared/software/FastQC 
```
You can then run fastqc from the directory containing the .fastq.gz files:
```
$ fastqc [-o output dir] seqfile1 .. seqfileN
$ fastqc -o /home/sara/output/ *.fastq.gz
```

