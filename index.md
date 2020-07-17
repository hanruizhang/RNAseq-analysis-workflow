---
title: "RNAseq-analysis-workflow"
output:
    html_document:
        toc: false
---

[Zhang Lab@Columbia](https://hanruizhang.github.io/zhanglab/) by [Hanrui Zhang](https://github.com/hanruizhang) [2019-06-10]

## 1. Introduction to RNA-seq
This [video](https://www.youtube.com/watch?v=08h3-05Y9JY) provides a thorough introduction about RNA-seq.

## 2. Install R and RStudio
We will use data sets in R. Please install the latest version of R or RStudio.

If you need a refresher, or have never used R before, please step through these tutorials.

* [Download R](https://cran.r-project.org/): R is the free software programming language we will use. Choose the correct version for your laptop: Mac/Windows 
* [Download R studio](https://www.rstudio.com/products/rstudio/download/): R studio is free software that will help us develop programs in R. Choose the correct version for your laptop: Mac/Windows
* [How to Download R and R studio](https://www.youtube.com/watch?v=cX532N_XLIs): A tutorial on how to download R and R Studio
* [How to Install a Package in R studio](https://www.youtube.com/watch?v=u1r5XTqrCTQ): Steps to install a package in r studio
* **Additional useful learning resources (for now and the future):**
	* [Unix cheat sheet](https://files.fosswire.com/2007/08/fwunixref.pdf)
	* [Unix tutorial](http://www.ee.surrey.ac.uk/Teaching/Unix/)
	* [Introduction to R](https://www.edx.org/course/introduction-to-r-for-data-science-2): A free edX class on R fundamentals using Datacamp platform
	* [HarvardX Biomedical Data Science Open Online Training](https://rafalab.github.io/pages/harvardx.html)
	* [R for data science](https://r4ds.had.co.nz/index.html)

## 3. Install Miniconda
* Go [here](https://docs.conda.io/en/latest/miniconda.html) to download Miniconda
* Follow the instruction for installation on [macOS](https://conda.io/projects/conda/en/latest/user-guide/install/macos.html) or [Windows](https://conda.io/projects/conda/en/latest/user-guide/install/windows.html)

## 4. Salmon: Transcript quantification from RNA-seq data

### 4.1 Install Salmon

* Follow the [instruction](https://combine-lab.github.io/salmon/getting_started/) to install salmon via bioconda.  
	`$ conda config --add channels conda-forge`  
	`$ conda config --add channels bioconda`  
	`$ conda create -n salmon salmon`  
	
* The environment can then be activated via:   
	`$ conda activate salmon`
	
* To deactivate  
	`$ conda deactivate`
	
* Use the following command line to get the help file for all the arguments.  
	`	$ salmon quant –h   `

### 4.2 Download Gencode annotation

* Go [here](https://www.gencodegenes.org/) to downlead the necessary files 
* Download the release you need (we use [human v30](https://www.gencodegenes.org/human/release_30.html)
 in this workflow)
* Download both the GTF and the Fasta file   


### 4.3 Obtain fastq file from SRA
* The first step is to install [SRA Toolkit](https://ncbi.github.io/sra-tools/install_config.html)
* To test whether the installation is successful, Open a terminal or command prompt and "cd" into the directory containing the toolkit executables
(e.g., [download_location]/sratoolkit[version]/bin/).  
	* For Linux/Mac OSX: ./fastq-dump -X 5 -Z SRR390728.  
	* For Windows:fastq-dump.exe -X 5 -Z SRR390728.   
	_If successful, the test should connect to NCBI, download a small amount of data from SRR390728 and the reference sequence needed to extract the data, and stream the first 5 spots of the file ("-X 5" option) to the screen ("-Z" option)._
* If the installation was unsuccessful, the toolkit may need to be reconfigured and adjusted to the default settings. The guide can also be found [above](https://ncbi.github.io/sra-tools/install_config.html).
*  Now we are ready to download the fastq files we will analyze
* Here are our RNA-seq data in GEO with accession number [GSE55536](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE55536).    

<div> 
  <img src="{{ '/images/HMDM_IPSDM.png' | absolute_url }}" alt="HMDM_IPSDM" width="600">
</div>   

* There are totally 33 samples. Let's try 6 samples.   

	| Sample ID     | SRR        | Description          |  
	|:------------- |:----------:| :-------------------:|  
	| HMDM MAC Rep1 | SRR1182374 | M0-HMDM, Replicate 1 |  
	| HMDM MAC Rep2 | SRR1182375 | M0-HMDM, Replicate 2 |  
	| HMDM MAC Rep3 | SRR2910663 | M0-HMDM, Replicate 3 |  
	| HMDM M1 Rep1  | SRR1182376 | M1-HMDM, Replicate 1 |  
	| HMDM M1 Rep2  | SRR1182377 | M1-HMDM, Replicate 2 |  
	| HMDM M1 Rep3  | SRR2910664 | M1-HMDM, Replicate 3 |   
	      
	**HMDM**: human monocyte-derived macrophages;  
	**M0**: Resting HMDM without stimulation;   
	**M1**: HMDM treated with endotoxin and interferon-gamma for 18-20 hours to induce inflammatoryresponse.   

* Use the command line below in terminal to download the fastq file (for now let's do it one by one). The code means "to only download the first 1M reads from SRR, and split the pair-end reads".   
	
	**Make sure you "cd" into /bin first.**  
		
			$ ./fastq-dump -X 1000000 --split-files SRR1182374  
			$ ./fastq-dump -X 1000000 --split-files SRR1182375  
			$ ./fastq-dump -X 1000000 --split-files SRR2910663  
			$ ./fastq-dump -X 1000000 --split-files SRR1182376  
			$ ./fastq-dump -X 1000000 --split-files SRR1182377  
			$ ./fastq-dump -X 1000000 --split-files SRR2910664   

* Alternatively, can use "prefetch" to download the fastq file.  
	                `$ ./sratoolkit.2.9.1-1-mac64/bin/prefetch SRR1182374`         
			
	  
	
### 4.4 Build salmon index
* First "cd" into the directory with the gencode GTF and Fasta files.
* The "--" is to trim the extra symbols in GENCODE for convenience to handle the data later. 
* Make sure to use `$ salmon --version` to check the Salmon version and change the index name in the code accordingly.
* This step will take a few minutes. 

        $ source activate salmon
        $ salmon index -t gencode.v30.transcripts.fa.gz -i gencode.v30_salmon_1.2.1 --gencode

### 4.5 Perform quantification using Salmon

  * Make sure the Salmon index folder and all of your fastq files are in the same directory and you are in the directory.   


		$ salmon quant -i gencode.v30_salmon_1.2.1 -p 8 --libType A --validateMappings --gcBias --biasSpeedSamp 5 -1 SRR1182374_1.fastq  -2 SRR1182374_2.fastq   -o M0_HMDM_1
		$ salmon quant -i gencode.v30_salmon_1.2.1 -p 8 --libType A --validateMappings --gcBias --biasSpeedSamp 5 -1 SRR1182375_1.fastq  -2 SRR1182375_2.fastq   -o M0_HMDM_2
		$ salmon quant -i gencode.v30_salmon_1.2.1 -p 8 --libType A --validateMappings --gcBias --biasSpeedSamp 5 -1 SRR2910663_1.fastq  -2 SRR2910663_2.fastq   -o M0_HMDM_3
		$ salmon quant -i gencode.v30_salmon_1.2.1 -p 8 --libType A --validateMappings --gcBias --biasSpeedSamp 5 -1 SRR1182376_1.fastq  -2 SRR1182376_2.fastq   -o M1_HMDM_1 
		$ salmon quant -i gencode.v30_salmon_1.2.1 -p 8 --libType A --validateMappings --gcBias --biasSpeedSamp 5 -1 SRR1182377_1.fastq  -2 SRR1182377_2.fastq   -o M1_HMDM_2
		$ salmon quant -i gencode.v30_salmon_1.2.1 -p 8 --libType A --validateMappings --gcBias --biasSpeedSamp 5 -1 SRR2910664_1.fastq  -2 SRR2910664_2.fastq   -o M1_HMDM_3
			
		
		
	**This way works, but in the end we will need to use loops.**



## 5. QC of the RNA-seq data using MultiQC
* Install [MultiQC](https://multiqc.info/)   

         $ conda install -c bioconda -c conda-forge multiqc
         
* Run multiqc

	     $ multiqc .    
	     
	**Multiqc will search in the current directory, so make sure you are in the directory with Salmon Quant folders.**
* You may read the documents to understand how to interpret the QC data.  
	  
## 6.  tximport: Importing salmon’s transcript-level quantifications and aggregate them to the gene level for gene-level differential expression analysis 
* From now on, everything is done in RStudio. And here is the [link](RNAseq_Bootcamp_GSE55536.html) and the [Rmd file](RNAseq_Bootcamp_GSE55536.Rmd).
* The Rmd file is modified from the Workflows below, which have more detailed explaination.  

	[https://f1000research.com/articles/7-952/v1](https://f1000research.com/articles/7-952/v1)

	[https://bioconductor.org/packages/devel/bioc/vignettes/tximport/inst/doc/tximport.html](https://bioconductor.org/packages/devel/bioc/vignettes/tximport/inst/doc/tximport.html)



## 7. Exploratory analysis and visualization

* Continue using the Rmd file above, which is modified from this [workflow](https://www.bioconductor.org/packages/devel/workflows/vignettes/rnaseqGene/inst/doc/rnaseqGene.html).  

## 8. Additional resources

* [https://seandavi.github.io/ITR/](https://seandavi.github.io/ITR/)   
		  

**Materials here are licensed as [CC BY-NC-SA 4.0 Creative Commons License](https://creativecommons.org/licenses/by-nc-sa/4.0/).**
	

	
