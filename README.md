# RNASeq_WorkFlow

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


## 2. Adapter trimming 




## 3. STAR Workflow 

STAR 
