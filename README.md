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
$ fastqc -o /home/sara/KRAS/fastqc_output/ *.fastq.gz
```
You can view other options for fastqc by typing fastqc --help in the command line. 

Once the output has been generated, use WinSCP (Windows) or ```rsync``` (Mac) to transfer the output files into your local machine. 

```
$ rsync [options] [source] [destination]
$ rsync -azvP sara@cbio.yale-nus.edu.sg:/home/sara/KRAS/fastqc_output/* ~/Desktop/KRAS/[folder_name]
```
The * indicates all files in the specified directory. 

**Note for Mac users**: You have to exit ```ssh``` to use ```rsync```. Also, depending on your machine, you may or or may not get an error message that says ```/etc/profile.d/lang.sh: line 19: warning: setlocale: LC_CTYPE: cannot change locale (UTF-8): No such file or directory``` but files will have been transferred - check local machine output folder to verify! 

[This](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) and [this](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/) are good resources to learn more about FastQC and the analysis modules found in the HTML report. 
