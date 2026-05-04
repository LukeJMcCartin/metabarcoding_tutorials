Luke McCartin - Postdoc - Smithsonian National Museum of Natural History

***Text in bold italics indicates things that you should do on your computer.***

# Background
## DNA Metabarcoding

DNA metabarcoding involves amplifying a variable and taxonomically diagnostic region of a gene (a barcode) with polymerase chain reaction (PCR) in DNA isolated from a sample. Typically, the samples that would be interesting to analyze with DNA metabarcoding include mixed communities of organisms. Two examples of samples commonly analyzed with metabarcoding include the microbiomes of plant and animal tissues and environmental samples. The analysis of DNA from environmental samples is referred to as environmental DNA, or eDNA, and these samples can include water, sediment, and even more recently, air (Taberlet, 2018; Lynggaard et al., 2022). 

During DNA metabarcoding library preparation, PCR primers can be used that amplify sequences from a taxonomically diverse suite of organisms (e.g. many eukaryotic groups including protists and invertebrate animals with 18S-v9 primers) or specific taxonomic groups (e.g. fish environmental DNA with 12S rRNA “MiFish” primers) (Amaral-Zettler, 2009; Miya et al., 2015). 

In a ‘two-step’ metabarcoding library preparation, first the barcode of interest is amplified with PCR using the primers that complement sequences that flank the DNA barcode. The PCR primers also include an overhanging sequence that is referred to as a linking adapter in this tutorial. In the second step, PCR primers that contain the linking adapter sequence, a unique index for each sample (sometimes called a tag), and the Illumina sequence adapters are incorporated into the first PCR product using another round of PCR (Bohmann et al., 2022). These second PCR products undergo size selection to remove unwanted sequences that are shorter or longer than the barcode of interest. Finally, the PCR products from all samples are pooled together (multiplexed) and sequenced, most commonly on an Illumina platform. After sequencing, the sequencing reads are reassigned to the samples they originated from based on their unique combination of indices in a process referred to as demultiplexing.

Bioinformatic analysis of the resulting sequence data files involves 1) trimming primer and adapter sequences, 2) quality filtering the data, and 3) denoising the trimmed and filtered data to infer the diversity and distribution of ‘real’ barcodes that do not contain sequencing errors. Downstream data analysis involves assigning the barcodes a taxonomic identity (i.e. which species the barcodes originated from), and then studying the ecology of the sampling locations and biological communities based on the distribution of barcodes across samples. However, the sky is the limit and metabarcoding data can be analyzed in many ways to answer interesting questions!

## The Data

In this tutorial, we are focused on analyzing data generated from ARMS and eDNA samples that were amplified using the 12S MiFish Universal primers (Miya, et al. 2015). The primer sequences are Forward: GTCGGTAAAACTCGTGCCAGC and Reverse: CATAGTGGGGTATCTAATCCCAGTTTG. The sequences are written in the 5’ to 3’ directions.

The data from each sample included in the sequencing run were already demultiplexed into separate .fastq files on the Illumina instrument. The exact sequencing instrument used was an Illumina MiSeq, and the MiSeq reagent kit V3 was used for sequencing. Pairing this instrument and kit enabled paired-end sequencing with 300 base pair sequence lengths in both the forward and reverse direction. Thus, in this project we have two ‘gzipped’ (a type of file compression) .fastq files for each sample, one corresponding to the forward direction (filenames end in ‘R1_001.fastq.gz’) and another corresponding to the reverse direction (filenames end in ‘R2_001.fastq.gz’). 

## Installing R, RStudio, and Conda

To use this tutorial, we'll need to first install R, RStudio, and conda on your computer. 

Conda is a software management tool that allows you to build multiple software "environments" on your computer. Software for bioinformatics programs, including those used in the Natrix2 pipeline, tend to have a lot of "dependencies". Dependencies are simply other software (like Python modules for example) that a software relies on. Conda ensures that the programs you install to complete a task are compatible with one another in terms of their versions. In this way, conda is an indispensible tool for bioinformatics.

To install conda, we'll install "miniconda", which is a sized down version of conda that will do everything we need to do.

***Install miniconda using the instructions for macOS on its [website](https://www.anaconda.com/docs/getting-started/miniconda/install).***

## Trimming primers/adapters and quality filtering the data

We'll use cutadapt to trim the primers and adapter sequences from the data. Afterwards, we’ll conduct quality trimming and filtering in R.

First, we need to install cutadapt into our conda environment. Follow the tutorial on the cutadapt [website](https://cutadapt.readthedocs.io/en/stable/installation.html).

```bash

conda create -n cutadapt cutadapt

```

Next, we'll activate the environment and run cutadapt. cutadapt commands are structured like this:

```bash

cutadapt -a fprimersequence -A rprimersequence --discard-untrimmed --pair-filter=any -o forward_output_file.fastq.gz -p reverse_output_file.fastq.gz forward_read_file.fastq.gz reverse_read_file.fastq.gz

```

The -a flag specifies the sequence(s) to be trimmed from the forward reads, the -A flag specifies the sequence(s) to be trimmed from the reverse reads, --discard-untrimmed tells cutadapt to drop any reads that are untrimmed,  --pair-filter=any tells cutadapt to drop read pairs if either the forward or reverse read aren’t trimmed, -o specifies the path to the forward read output, -p specifies the path to the reverse read output, and the last two arguments specified the paths to the forward and reverse reads you want to trim/filter.

In today’s tutorial, we’ll specify linked adapters. The syntax looks like this: SEQUENCE1...SEQUENCE2. The first sequence specifies the primer/adapter found towards the 5’ end of the read and the second sequence specifies the primer/adapter found towards the 3’ end.

We’ll use this syntax with a couple of modifications to trim the primer sequences that should always be found at the 5’ ends of the reads and the reverse complements of the primer sequences that might be found at the 3’ ends of the reads.

Here’s what this will look like with the modifications that we need to make:

```bash

-a "^FPRIMER;required...RPRIMER_RC;optional" 
-A "^RPRIMER;required...FPRIMER_RC;optional"

```

The quotes surround the linked adapter so that the 5’ sequence and 3’ sequences can be specified as required and optional, respectively, using ;required and ;optional after each sequence. Remember, we’ll always trim the 5’ primer sequence, but not always the 3’ primers/adapters, since typically the amplicon is longer than the read length. If we trim the reverse complement of the 3’ primers, we’ll also trim the adapter sequences downstream that are present in the data.

^ specifies that the 5’ primer sequence must be found at the very beginning of the read. 

Any read pair where the primer at the 5’ end is not matched and trimmed in either the forward or reverse read will be excluded from the output.

We'll use cutadapt to trim the sequences from the 12S data, identifying amplicons where the primers are found. ***Fill in the appropriate file paths, file names, and sequences to run cutadapt and generate trimmed data.***

```bash

cd ./YOUR_WORKING_DIRECTORY_WITH_SEQUENCES

mkdir ./trimmed_seqs

conda activate cutadapt

for sample in ./YOUR_WORKING_DIRECTORY_WITH_SEQUENCES/*_S*_R1_001.fastq.gz; do
  SAMPLE=$(echo ${sample} | sed "s/.\/YOUR_WORKING_DIRECTORY_WITH_SEQUENCES\///" | sed "s/_R1_\001\.fastq.gz//")
  echo $SAMPLE
  cutadapt \
  -a "^FPRIMER;required...RPRIMER_RC;optional" \
  -A "^RPRIMER;required...FPRIMER_RC;optional" \
  --discard-untrimmed --pair-filter=any \
  -o ./trimmed_seqs/primers-trimmed-${SAMPLE}_R1_001.fastq.gz \
  -p ./trimmed_seqs/primers-trimmed-${SAMPLE}_R2_001.fastq.gz \
  ./YOUR_WORKING_DIRECTORY_WITH_SEQUENCES/${SAMPLE}_R1_001.fastq.gz \
  ./YOUR_WORKING_DIRECTORY_WITH_SEQUENCES/${SAMPLE}_R2_001.fastq.gz
done

conda deactivate

```

## Quality filtering, denoising, and merging read pairs and generating a list and table of amplicon sequence variants (ASVs).

The read pairs that survived cutadapt filtering are not completely free of errors and thus may not represent the actual, biological DNA barcodes that we’re after. To infer accurate DNA barcodes from the organisms in our original sample, we will use a process called denoising through the R package DADA2. Denoising involves removing sequencing error from the dataset by inferring amplicon sequence variants, or ASVs, that represent the true sequences of the organisms in our samples. 

ASVs are inferred by comparing the similarity and abundances of sequences in our quality-filtered data, and by analyzing their quality scores. In an overly simplistic explanation, sequences that occur often in the data and have high quality scores are likely to represent the true barcodes or ASVs from organisms in the sample. Sequences that are similar to abundant sequences but have lower quality scores and lower abundances are likely the result of errors of the more abundant, closely related sequences. These sequences with lower abundance and worse quality scores on average are merged with more abundant sequences that have higher quality scores on average.

Open RStudio on your personal computer.

Start a new R “project” in an existing working directory by clicking “File” > “New Project…” > “Existing Directory” and choose the directory that you created on your personal computer where your trimmed sequence files are.

Using an R project sets your working directory, so that you don’t have to manually change it, and allows you to save your work while switching between projects.

***Create a new RScript by clicking “File” > “New File” > “R Script”.***

***Use the template code below to create a DADA2 script and process your data to denoise amplicon sequence variants.*** 

First, we’ll do some error filtering and quality control using the DADA2 function filterAndTrim.

```r

library(dada2)

fnFs <- list.files("./trimmed_seqs", pattern="_R1_001.fastq.gz", full.names=TRUE) # listing and defining the fastq.gz files with trimmed, forward reads

fnRs <- list.files("./trimmed_seqs", pattern="_R2_001.fastq.gz", full.names=TRUE) # listing and defining the fastq.gz files with trimmed, reverse reads

plotQualityProfile(fnFs[c(1)]) # plotting quality of forward reads

plotQualityProfile(fnRs[c(1)]) # plotting quality of reverse reads

```

Inspect the quality profiles of the data to consider how we should quality filter them. 

First, in your R code chunk specify the paths to write the filtered sequencing files to.

```r

filtFs <- file.path("./filtered", basename(fnFs)) #creating a directory for the filtered forward reads

filtRs <- file.path("./filtered", basename(fnRs)) #creating a directory for the filtered reverse reads

```

There are a few ways we can filter the data using the filterAndTrim function in DADA2. Today, we’ll consider using truncQ, truncLen, maxEE, and minQ.  truncQ trims the reads after a certain, low quality score is encountered; trunLen cuts the ends of the reads to a certain length, and you can specify the truncation lengths of the forward and reverse reads separately using c() like this c(FWD,REV); maxEE filters out reads that contain a number of expected errors, calculated based on the quality scores, that exceed the specified value; and minQ filters out reads with a quality score less than the expected value.

Decide whether you’d like to use truncQ, truncLen, maxEE, and/or minQ to trim and filter your data. Choose your trimming lengths and/or quality scores and run the command in your R script.

Important note: Be careful with truncQ, maxEE, and minQ. Specifying too high of a quality score or too few expected errors can result in the loss of a lot of data. Check the defaults for the help page to see where you should start

```r

out <- filterAndTrim(fnFs, filtFs, fnRs, filtRs, truncLen = c(NNN, NNN), truncQ = N, maxEE = N, minQ = N, verbose = TRUE)

```

Next, we’ll generate 1) a list of amplicon sequence variants (ASVs) and 2) a table with the number of sequencing reads assigned to each ASV across the samples in the dataset.

```r

#learning the error models for these data
errF <- learnErrors(filtFs, nbases=1e6, multi=TRUE, verbose = TRUE)
errR <- learnErrors(filtRs, nbases=1e6, multi=TRUE, verbose = TRUE)

#plotting the error model profiles for the forward and reverse
plotErrors(errF)
plotErrors(errR)

#denoising the forward and reverse reads
ddF <- dada(filtFs, errF)
ddR <- dada(filtRs, errR)

#merging denoised read pairs
merged <- mergePairs(ddF, filtFs, ddR, filtRs, verbose=TRUE)

#constructing the ASV table
table.chimeras <- makeSequenceTable(merged)

#removing chimeras from the ASV table
table.no.chimeras <- removeBimeraDenovo(table.chimeras, multi=TRUE, verbose=TRUE)

#checking how many reads were retained after each step in the pipeline
getN <- function(x) sum(getUniques(x))
track <- cbind(sapply(ddF, getN), sapply(ddR, getN), sapply(merged, getN), rowSums(table.no.chimeras))
colnames(track) <- c("denoisedF", "denoisedR", "merged", "nonchim")
rownames(track) <- basename(fnFs)
head(track)
write.csv(track, "./DADA2_pipeline_stats.csv")

#exporting the ASV table and the ASV sequences
write.table(t(table.no.chimeras), file = 'asv_table.tsv', sep = "\t", row.names = TRUE, col.names=NA, quote=FALSE)
uniquesToFasta(table.no.chimeras, fout='rep-seqs.fasta', ids = paste("ASV",1:ncol(table.no.chimeras),sep="_"))

```

Your ASV table is now saved as "asv_table.txt" and your representative sequences for each ASV is now saved as "rep-seqs.fasta". Downstream processing will involve idenitifying these sequences using BLAST and analyzing their diversity and distribution across the samples.
