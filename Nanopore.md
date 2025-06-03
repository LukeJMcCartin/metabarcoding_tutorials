# Analyzing DNA Metabarcoding Data Generated using Oxford Nanopore Sequencing
May 2025
Luke McCartin - Postdoc - Lehigh University in the Herrera Lab

**Text in bold indicates things that you should do on your computer.**

# Background
## DNA Metabarcoding

DNA metabarcoding involves amplifying a variable and taxonomically diagnostic region of a gene (a barcode) with polymerase chain reaction (PCR) in DNA isolated from a sample. Typically, the samples that would be interesting to analyze with DNA metabarcoding include mixed communities of organisms. Two examples of samples commonly analyzed with metabarcoding include the microbiomes of plant and animal tissues and environmental samples. The analysis of DNA from environmental samples is referred to as environmental DNA, or eDNA, and these samples can include water, sediment, and even more recently, air (Taberlet, 2018; Lynggaard et al., 2022). 

During DNA metabarcoding library preparation, PCR primers can be used that amplify sequences from a taxonomically diverse suite of organisms (e.g. many eukaryotic groups including protists and invertebrate animals with 18S-v9 primers) or specific taxonomic groups (e.g. fish environmental DNA with 12S rRNA “MiFish” primers) (Amaral-Zettler, 2009; Miya et al., 2015). 

In a ‘two-step’ metabarcoding library preparation, first the barcode of interest is amplified with PCR using the primers that complement sequences that flank the DNA barcode. The PCR primers also include an overhanging sequence that is referred to as a linking adapter in this tutorial. In the second step, PCR primers that contain the linking adapter sequence, a unique index for each sample (sometimes called a tag), and the sequencing adapters are incorporated into the first PCR product using another round of PCR (Bohmann et al., 2022). These second PCR products undergo size selection to remove unwanted sequences that are shorter or longer than the barcode of interest. Finally, the PCR products from all samples are pooled together (multiplexed) and sequenced, most commonly on an Illumina platform but also on PacBio and Oxford Nanopore platforms. After sequencing, the sequencing reads are reassigned to the samples they originated from based on their unique combination of indices in a process referred to as demultiplexing. This step is typically conducted by the sequencing facility using software dedicated to the sequencing instrument.

Once you receive your metabarcoding data, bioinformatic processing typically first involves trimming primer and adapter sequences from the reads. In the case of Nanopore data, you’ll also need to reorient your sequencing reads so that they are in the 5’ to 3’ direction in the forward direction (i.e. with the sequence complementary to your forward primer at the beginning of the read). After this, the data are filtered for quality and then clustered and/or error-corrected infer ‘real’ biological sequences from the data that do not contain sequencing errors. The typical outputs of these processes are a file with the biologically accurate barcode sequences, a table with the taxonomic identity of these barcodes, and a table of their frequencies in each sample. Once you have generated these outputs, you can start to ask questions about the composition of organisms in the samples based on their distributions across the samples. Typically, metabarcoding is used to compare the composition of organisms across sampling sites, species, and environmental factors. There are a number of statistical tests developed specifically for metabarcoding data that complement these analyses.

## The Data

In this tutorial, we are focused on analyzing data generated from coral samples. DNA was isolated from these corals at the Smithsonian National Museum of Natural History. At Lehigh, you amplified a portion of 18S with using the UNonMet-PCR primer set, which is an “easy, inexpensive, and near-universal method for the study of animal-associated microeukaryotes” (Campo et al. 2019). The primer sequences are 574F: TTTCTGTTGGTGCTGATATTGCCGGTAAYTCCAGCTCYV and Reverse: ACTTGCCTGTCGCTCTATCTTCCTTTAARTTTCASYCTTGCG. The sequences are written in the 5’ to 3’ directions. The adapter sequences that are included for compatibility with library prep for Oxford Nanopore Sequencing are underlined. At Lehigh, the first PCR amplification and bead cleanup of the PCR products were performed.

Clean PCR products were shipped to the University of Connecticut Microbial Analysis, Resources and Services (MARS) facility for further library preparation following the PCR barcoding with native barcodes, ligation sequencing protocol (SQK-LSK114). Libraries were sequenced on a Minion using a FLO-MIN114 flow cell that has R10 chemistry. Basecalling was conducted using MinKNOW (v. 25.03.7), Bream (v. 8.4.4), Configuration (6.4.10), Dorado (7.8.3), and MinKNOW Core (6.4.8) and with super-accurate basecalling at 400 bps. From the facility, we received two demultiplexed .fastq files for each library. One file contains data from an initial run to “improve the balance for the "real" run”. Since some of the samples produced multiple thousands of reads, these data are useful too. The second file contains the rest of the data.

## WITH DOCKER

To run Natrix2 we'll have to clone the repository from GitHub and also install the program Docker Desktop. Docker is a program that allows us to install dependencies for certain packages on our computer to run them in "containers".

**Install Docker Desktop for your computer from the following (website)[https://www.docker.com/products/docker-desktop/]**

**Once you have Docker Desktop installed on your computer, open it to start the Docker engine**

To run Natrix2, we'll use the scripts and files that can be cloned from the Natrix2 GitHub page. Cloning a github repository is easy, but first you need to also install git on your computer. The easiest way to do this is to install GitHub Desktop.

**Install GitHub Desktop on your computer from the following (website)[https://docs.github.com/en/desktop/installing-and-authenticating-to-github-desktop/installing-github-desktop]**

Now that we have GitHub Desktop installed, we can clone the GitHub repository onto our computer. First, we'll need to open a terminal in VSCode, navigate to a directory where we want to work, and clone the GitHub repository.

**Open a terminal in VSCode, navigate to the directory where you want to copy Natrix2, and clone the GitHub repository**

Here, I'll just clone Natrix2 to my desktop as an example.

```zsh

cd ~/Desktop

git clone https://github.com/dbeisser/Natrix2

```

This should download a folder named Natrix2 into the directory that you've navigated to! 

Next, to run Natrix2 we'll need to create a Docker container for it.

**First, open the Docker Desktop application**

Opening Docker Desktop will start the Docker engine. Once the engine is running, we can pull the Natrix2 container as described in the Natrix2 tutorial on GitHub in the section (Running Natrix with docker or docker-compose)[https://github.com/dbeisser/Natrix2?tab=readme-ov-file#running-natrix-with-docker-or-docker-compose].

**Pull the Natrix2 container***



**To check that the Docker engine is running, in your terminal type ```docker --version```.**

Docker should report back with the version





## WITH SNAKEMAKE

# Connecting to the remote computer

Natrix2 is installed on our lab's "Thelio" computer, which is a powerful desktop computer that we can connect to remotely. To connect to the computer, we'll use the `ssh` command.

**Open a terminal and connect to the remote computer using ssh** 

```zsh

ssh your_lehigh_ID@128.180.49.174 # as an example, I log in with ljm419@128.180.49.174

```

Your terminal should prompt you for your Lehigh ID password. When you connect, you will be directed to your home directory. After you connect once, you should be able to log in by using the remote explorer function in VSCode.

# Setting up Natrix2 in your home directory

To run Natrix2, we'll use the scripts and files that are located on the remote computer in the directory /media/other/software/Natrix2/. For your analysis, you should make a copy of this directory into your home directory. This is simple to do using the ```cp``` command. The first argument of the ```cp``` command is the origin of the files, and the second argument is the destination you want to copy to. The -r or "recursive" argument tells the copy command to copy all of the files in the original directory.

**Copy the Natrix2 directory that was cloned from github into your home directory**

```zsh

cp /media/other/software/Natrix2 ./ -r 

```

**Make sure that Natrix2 has successfully copied into your home directory either by using remote explorer in VS code or ```ls```**

To run Natrix2, our sequencing data has to be formatted in a certain manner. Specifically, each file must be renamed SAMPLEID_A_R1/R2.fastq.gz. I renamed the data,  combined data from different files that corresponded to the same samples, "gzipped" the data to compress them. The data are uploaded to the remote computer in the directory /media/data3/metabar/microbiome/genohub5010689/combined_renamed.

To work with the data, we need to copy it to the "input_data" folder in the Natrix2 folder within our home directory. Again, this is quite straightforward to do using the ```cp``` command.

**First, make a folder within "input_data" for the data files.** 

Feel free to name the file what you want to, but I will continue to refer to the folder as "18S_nanopore_data".

```zsh

mkdir ./Natrix2/input_data/18S_nanopore_data

```

**Next, copy the combined and renamed fastq.gz files into this folder, and make sure that it has copied using ```ls```.**

```zsh

cp /media/data3/metabar/microbiome/genohub5010689/combined_renamed/*fastq.gz ./Natrix2/input_data/18S_nanopore_data -r # the *fastq.gz copies all of the gzipped sequence data files by using a wildcard for the rest of the file name

ls ./Natrix2/input_data/18S_nanopore_data

```

Great, now the data are copied into the directory, and we can configure a Natrix2 run.

A [conda environment](https://www.anaconda.com/docs/tools/working-with-conda/environments) has been created on the remote computer that allows you to load the Natrix2 software and any other software it depends on. 

**To get ready to run Natrix2, activate this environment with ```mamba```.**

```zsh

mamba activate natrix

```

To run Natrix2, we'll follow the tutorial on [the Natrix2 GitHub page](https://www.anaconda.com/docs/tools/working-with-conda/environments) under the "Running Natrix with the pipeline.sh script" option.

As described in the GitHub tutorial, running Natrix2 in this manner involves creating a configuration file for Natrix2 that is customized to our Nanopore data and the data analysis parameters that we want to implement.

There are already several configuration files in the Natrix2 directory that we can use as a starting point. The file that we'll use is named "Nanopore.yaml".

**Open the Nanopore.yaml file in VSCode and take a look at it**

There are several lines in the Nanopore.yaml file that we need to edit for our dataset. Lines 3-8 pertain to the input and output paths for the analysis and the computing power that can be allocated to Natrix2.

To run our analysis, we'll make a copy of this file and edit it for our project.

**Copy nanopore.yaml file and rename it with the analysis and date**

```zsh

cp ./Natrix2/Nanopore.yaml ./Natrix2/nanopore_18S_test_20250327.yaml

```

I'll edit this new configuration file to run a test analysis on two samples from the entire dataset. 

In general, I'll make the following changes and save them.

```yaml

general:
        filename: input_data/18S_nanopore_test # The path / filename of the project folder, primertable (.csv) and configfile (.yaml). If the raw data folder is not in the root directory of Natrix, please add the path relative to the root directory (e.g. input/example_data)
        output_dir: 18S_nanopore_test_results # Path to custom output directory / relative to the root directory of natrix. Do not use a dash in the folder name.
        primertable: primer_table/18S_nanopore.csv # Path to the primertable. If the primertable is not in the root directory of Natrix, please add the path relative to the root directory (e.g. input/example_data.yaml)
        units: units.tsv # Path to the sequencing unit sheet. (name will be concatenated with output_dir)
        cores: 20 # Amount of cores available for the workflow.
        memory: 10000 # Available RAM in Mb.
        multiqc: TRUE # Initial quality check (fastqc & multiqc), currently only works for not yet assembled reads.
        demultiplexing: FALSE # Boolean, run demultiplexing for reads if they were not demultiplexed by the sequencing company (only Illumina support & slow).
        read_sorting: FALSE # Boolean, run read sorting for paired end reads if they were not sorted by the sequencing company (only Illumina support & slow).
        already_assembled: FALSE # Boolean, skip the quality control and read assembly steps for data if it is already assembled (only Illumina support).
        seq_rep: OTU # Type of sequence representative, possible values are: "ASV", amplicon sequence variants, created with DADA2 or "OTU", operational taxonomic units, created with SWARM or VSEARCH.

```

The one requied file that we've yet to discuss is the primertable file. This is a comma separated values file that contains the primer sequences used to prepare the metabarcoding libraries. If you inspect the "Nanopore.csv" file in the /primer_table/ directory, you can see how it is formatted.

**Make a primer table for our 18S primers by first copying this table as a template.**

```zsh 

cp ./Natrix2/primer_table/Nanopore.csv ./Natrix2/primer_table/18S_nanopore.csv

```

The GitHub tutorial describes how this primer table file is created. 

The "Probe" column is just a name for your primer set. In our case, we're only using one primer set. 

We do not have poly_N's incorporated into our primers, so these columns can be left blank.

For the barcode sequences, we can leave these blank as well, because our samples are already demultiplexed.

So, we are left with filling in the specific_forward_primer and specific_reverse primer. For these values, we're going to include the adapter sequences that we used for library prep. 

My primer table looks like this:


Probe,poly_N,Barcode_forward,specific_forward_primer,poly_N_rev,Barcode_reverse,specific_reverse_primer
UNonMet_574F,,,TTTCTGTTGGTGCTGATATTGCCGGTAAYTCCAGCTCYV,,,ACTTGCCTGTCGCTCTATCTTCCTTTAARTTTCASYCTTGCG

**Edit your primer table accordingly and save it.**

Now, our file paths to the input data and primer file should be ready to go. There are some other changes to the configuration file that we should make or investigate.

Take a look at the "nanopore" section. These parameters can be adjusted to quality filter/trim your data. the most important of these are minlen and maxlen, which remove sequences outside of our expected length range.

**Set these parameters carefully and think about any adjustments you'd like to make to the values below!**

```yaml

nanopore:
        quality_filt: 15 # Minimum Phred quality score.
        min_length: 100 # Minimum length of reads.
        max_length: 1000 # Maximum length of reads.
        head_trim: 0 # Trim N nucleotides from the start of reads.
        tail_trim: 0 # Trim N nucleotides from the end of reads.
        pychopper: TRUE # Boolean that indicates if pychopper should be used for reorientation, trimming and quality check of reads, if not done before.
        pychopqual: 7 #Minimum mean Q-score base quality for pychopper (default 7).
        racon: 4 #Iterations of racon for read correction. Possible values are 1, 2, 3, 4 or 5. The higher the quality of reads, the less iterations are required.

```

In clustering, we will use vsearch. 

**Inspect lines 65 and 66.**

```yaml

clustering: "vsearch" # Allows you to specify OTU clustering method to use. Your options are: swarm and vsearch. Nanopore only supports vsearch option.
vsearch_id: 0.97 #Percent identity for vsearch OTU clustering (1 = 100%).

```

The "vsearch_id" percent identity is a very important parameter that you may want to edit.


Lastly, we're going to classify the taxonomy of the sequences using BLAST, not mothur. In addition, we'll use the SILVA database rather than the NCBI database, since SILVA is a curated database specifically for ribosomal DNA sequences.

**ADdjust the Mothur and BLAST settings accordingly.

```yaml

# Mothur parameter
classify:
        mothur: FALSE # Boolean for the use of mothur
        search: kmer # Allows you to specify the method to find most similar template. Your options are: suffix, kmer, blast, align and distance. The default is kmer
        method: wang # Allows you to specify classification method to use. Your options are: wang, knn and zap. The default is wang.
        database: pr2 # Database against which MOTHUR should be carried out, at the moment "pr2" , "unite" and "silva" are supported
database_version:
        pr2: 4.14.0
        silva: 138.1
database_path:
        silva_tax: database/silva_db.138.1.tax # Path for Silva taxonomy database
        silva_ref: database/silva_db.138.1.fasta # Path for Silva reference database
        pr2_ref: database/pr2db.4.14.0.fasta # Path for PR2 reference database
        pr2_tax: database/pr2db.4.14.0.tax # Path for PR2 taxonomy database
        unite_ref: database/unite_v10.fasta # Path for UNITE reference database
        unite_tax: database/unite_v10.tax # Path for UNITE taxonomy database

# BLAST
blast:
        blast: TRUE # Boolean to indicate the use of the BLAST search algorithm to assign taxonomic information to the OTUs.
        database: SILVA # Database against which the BLAST should be carried out, at the moment "NCBI" and "SILVA" are supported.
        drop_tax_classes: '' # Given a comma-separated list, drops undesired classes either by id, by name or using regex
        db_path: database/silva/silva.db # Path to the database file against which the BLAST should be carried out, at the moment only the SILVA (database/silva/silva.db) and NCBI (database/ncbi/nt) databases will be automatically downloaded.
        max_target_seqs: 10 # Number of NCBI blast hits that are saved per sequence / OTU.
        ident: 90.0 # Minimal identity overlap between target and query sequence. Set to lower threshold to be able to filter later by hand-
        evalue: 1e-20 # Highest accepted evalue. Set to higher threshold (e.g. 1e-5) to be able to filter later by hand.

```

OK! Now that our congifuration file is set, we can run Natrix2.

**Run Natrix2 by navigating to the Natrix2 root directory and following the "Running Natrix with Docker or docker-compose"**

Docker is installed in the Natrix conda environment, so as long 

```zsh

cd ./Natrix2

python3 create_dataframe.py nanopore_18S_test_20250327.yaml

nohup snakemake --use-conda --configfile nanopore_18S_test_20250327.yaml --cores 20 >> ./nanopore_18S_test_20250328.txt &

```