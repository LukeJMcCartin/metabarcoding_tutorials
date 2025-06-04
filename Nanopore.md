# Analyzing DNA Metabarcoding Data Generated using Oxford Nanopore Sequencing with Natrix2

Luke McCartin - Postdoc - Lehigh University in the Herrera Lab

***Text in bold italics indicates things that you should do on your computer.***

# Background
## DNA Metabarcoding

DNA metabarcoding involves amplifying a variable and taxonomically diagnostic region of a gene (a barcode) with polymerase chain reaction (PCR) in DNA isolated from a sample. Typically, the samples that would be interesting to analyze with DNA metabarcoding include mixed communities of organisms. Two examples of samples commonly analyzed with metabarcoding include the microbiomes of plant and animal tissues and environmental samples.

During DNA metabarcoding library preparation, PCR primers can be used that amplify sequences from a taxonomically diverse suite of organisms (e.g. many eukaryotic groups including protists and invertebrate animals with 18S-v9 primers) or specific taxonomic groups (e.g. fish environmental DNA with 12S rRNA “MiFish” primers) (Amaral-Zettler, 2009; Miya et al., 2015). 

In a ‘two-step’ metabarcoding library preparation, first the barcode of interest is amplified with PCR using the primers that complement sequences that flank the DNA barcode. The PCR primers also include an overhanging sequence that I'll refer to as a linking adapter in this tutorial. In the second step, PCR primers that contain the linking adapter sequence, a unique index for each sample (sometimes called a tag), and the sequencing adapters are incorporated into the first PCR product using another round of PCR (Bohmann et al., 2022). These second PCR products undergo size selection to remove unwanted sequences that are shorter or longer than the barcode of interest. Finally, the PCR products from all samples are pooled together (multiplexed) and sequenced, most commonly on an Illumina platform but also on PacBio and Oxford Nanopore platforms. After sequencing, the sequencing reads are reassigned to the samples they originated from based on their unique combination of indices in a process referred to as demultiplexing. This step is typically conducted by the sequencing facility using software dedicated to the sequencing instrument.

Once you receive your metabarcoding data, bioinformatic processing typically first involves trimming primer and adapter sequences from the reads. In the case of Nanopore data, you’ll also need to reorient your sequencing reads so that they are in the 5’ to 3’ direction in the forward direction (i.e. with the sequence complementary to your forward primer at the beginning of the read). After this, the data are filtered for quality and then clustered and/or error-corrected infer ‘real’ biological sequences from the data that do not contain sequencing errors. The typical outputs of these processes are a file with the biologically accurate barcode sequences, a table with the taxonomic identity of these barcodes, and a table of their frequencies in each sample. Once you have generated these outputs, you can start to ask questions about the composition of organisms in the samples based on their distributions across the samples. Typically, metabarcoding is used to compare the composition of organisms across sampling sites, species, and environmental factors. There are a number of statistical tests developed specifically for metabarcoding data that complement these analyses.

## The Data

In this tutorial, we are focused on analyzing data generated from coral samples. DNA was isolated from these corals at the Smithsonian National Museum of Natural History. At Lehigh, we amplified a portion of 18S with using the UNonMet-PCR primer set, which is an “easy, inexpensive, and near-universal method for the study of animal-associated microeukaryotes” (Campo et al. 2019). The primer sequences are 574F: TTTCTGTTGGTGCTGATATTGCCGGTAAYTCCAGCTCYV and Reverse: ACTTGCCTGTCGCTCTATCTTCCTTTAARTTTCASYCTTGCG. The sequences are written in the 5’ to 3’ directions. The adapter sequences that are included for compatibility with library prep for Oxford Nanopore Sequencing are underlined. At Lehigh, the first PCR amplification and bead cleanup of the PCR products were performed.

Clean PCR products were shipped to the University of Connecticut Microbial Analysis, Resources and Services (MARS) facility for further library preparation following the PCR barcoding with native barcodes, ligation sequencing protocol (SQK-LSK114). Libraries were sequenced on a Minion using a FLO-MIN114 flow cell that has R10 chemistry. Basecalling was conducted using MinKNOW (v. 25.03.7), Bream (v. 8.4.4), Configuration (6.4.10), Dorado (7.8.3), and MinKNOW Core (6.4.8) and with super-accurate basecalling at 400 bps. From the facility, we received two demultiplexed .fastq files for each library. One file contains data from an initial run to “improve the balance for the "real" run”. Since some of the samples produced multiple thousands of reads, these data are useful too. The second file contains the rest of the data.

# Processing Nanopore Metabarcoding Data with Natrix2

To analyze the data we received from the sequencing facility, we are going to use a pipeline called Natrix2 [Deep et al., 2023](https://mbmg.pensoft.net/article/109389/). Natrix2 will perform all of the typical steps for analyzing metabarcoding data, and can be specifically configured for Nanopore sequencing data. It relies on a number of other tools that are developed for metabarcoding analysis and the analysis of Nanopore sequencing data. Natrix2 is hosted on a [GitHub Page](https://github.com/dbeisser/Natrix2). This should serve as your main resource for using and troubleshooting the pipeline on your computer.

## Installing Docker, git, and Conda

To use Natrix2, we'll need to first install a few programs on your computer: 1) Docker Desktop, 2) GitHub Desktop , and 3) Conda. 

Docker is a program that allows programmers to share "containers" that include necessary software and code to run a program. It is especially useful because it allows programs to be run across platforms, for example between Linux and Mac OS computers.

The easiest way to run Natrix2 using Docker is to install Docker Desktop onto your computer.

***Install Docker Desktop for your computer from the following [website](https://www.docker.com/products/docker-desktop/)***

Now that we have Docker installed and running, we'll need to download the scripts to run Natrix2 from its GitHub page. "Cloning" a github repository onto your own computer is easy, but first you need to also install git on your computer. The easiest way to do this is to install GitHub Desktop. 

GitHub Desktop is an especially useful program because it allows you to work directly with your own GitHub repositories directly on your computer via integration with VSCode. I highly recommend you use it and make a GitHub account!

***Install GitHub Desktop on your computer from the following [website](https://docs.github.com/en/desktop/installing-and-authenticating-to-github-desktop/installing-github-desktop)***

Once you have GitHub Desktop installed, it should prompt you to make a GitHub account if you open it. I recommend that you make a GitHub account using your personal email (one that outlasts your time at Lehigh). If you plan to work on coding projects in the future, and especially if you plan to share your code with others, GitHub is a fantastic resource.

Now that we have git installed on our computer, we can clone the Natrix2 GitHub repository that contains the code we need to run the program. First, we'll need to open a terminal in VSCode, then we'll navigate to a directory where we want to place it, and clone the GitHub repository.

***Open a terminal in VSCode, navigate to the directory where you want to copy Natrix2, and clone the GitHub repository***

Here, I'll just clone Natrix2 to my desktop as an example. Wherever you place the Natrix2 repository on your computer, that is where you'll work on your analysis of the Nanopore sequencing data.

```zsh

cd ~/Desktop

git clone https://github.com/dbeisser/Natrix2

```

Lastly, we need to install "conda" on our computer. Conda is a software management tool that allows you to build multiple software "environments" on your computer. Software for bioinformatics programs, including those used in the Natrix2 pipeline, tend to have a lot of "dependencies". Dependencies are simply other software (like Python modules for example) that a software relies on. Conda ensures that the programs you install to complete a task are compatible with one another in terms of their versions. In this way, conda is an indispensible tool for bioinformatics.

To install conda, we'll install "miniconda", which is a sized down version of conda that will do everything we need to do.

***Install miniconda using the instructions for macOS on its [website](https://www.anaconda.com/docs/getting-started/miniconda/install).***

## Running Natrix 2

Great. Now we are set up to run Natrix2 on our computer. The first step is to create a conda environment for Natrix2 to run in. The developers have included a .yaml configuration file in the Natrix2 repository that makes this easy to accomplish with a simple line of code.

***Navigate to the Natrix directory and create the natrix conda environment.***

```zsh

cd Natrix2

conda env create --file=natrix.yaml

```

We're now ready to download the Docker container that is set up to run Natrix2. We'll start Docker on your computer to get the Docker "engine" running. You can also make a Docker account if you'd like, though you can skip this step and still use the program.

***Open Docker Desktop by double clicking to start the Docker engine.***

Opening Docker Desktop will start the Docker engine. In the lower left hand corner of the Docker window, you should see that it will say "engine running" in green. Once the engine is running, we can pull the Natrix2 container as described in the Natrix2 tutorial on GitHub in the section Running Natrix with docker or docker-compose: https://github.com/dbeisser/Natrix2?tab=readme-ov-file#running-natrix-with-docker-or-docker-compose.

***Pull the Natrix2 container to create it in Docker.***

```zsh

docker pull dbeisser/natrix2:latest

```

We are close to being able to run Natrix2 using Docker on our computer. Running Natrix2 using docker requires a very specific set of directories that include your data and configuration files for the run as described in the GitHub tutorial.  To run Natrix2, we'll replicate this directory structure in the main Natrix2 folder exactly as it is described in the GitHub tutorial.

***Navigate to the Natrix folder and create the necessary directories that will be populated with our data.***

```zsh

mkdir input
mkdir input/samples

mkdir output
mkdir output/results

mkdir database

```

Next, we'll populate these folders with our sequencing data and configuration files for our Natrix2 analysis.

To run Natrix2, our sequencing data has to be formatted in a certain manner. Specifically, each file must be renamed SAMPLEID_A_R1/R2.fastq.gz. I renamed the data, combined data from different files that corresponded to the same samples, "gzipped" the data to compress them. The data are uploaded to our Google drive and are linked [here](https://drive.google.com/drive/folders/14Sl1tFhT_nV_g9Ta9UyA7HyJek7f0CbB?usp=drive_link) in the 'combined_renamed' folder.

***Download the combined and renamed data files (.fastq.gz) from Google Drive and move it to your /input/samples directory.***

The other data that we need to run Natrix2 is a simple comma-separated values file that contains our primer sequences used to generate the libraries. If you inspect the "Nanopore.csv" file in the /primer_table/ directory, you can see how this file should be formatted.

***Make a primer table for our 18S primers by first copying this table as a template into the "/input" directory.***

```zsh 

cp ./primer_table/Nanopore.csv ./input/18S_nanopore.csv # my suggested name for this file (the configuration file for the run will inherit this name)

```

The GitHub tutorial describes how this primer table file is created. 

The "Probe" column is just a name for your primer set. In our case, we're only using one primer set. 

We do not have poly_N's incorporated into our primers, so these columns can be left blank.

For the barcode sequences, we can leave these blank as well, because our samples are already demultiplexed.

So, we are left with filling in the specific_forward_primer and specific_reverse_primer. For these values, we're going to include the adapter sequences that we used for library prep. 

Our primer table should look like this:

Probe,poly_N,Barcode_forward,specific_forward_primer,poly_N_rev,Barcode_reverse,specific_reverse_primer
UNonMet_574F,,,TTTCTGTTGGTGCTGATATTGCCGGTAAYTCCAGCTCYV,,,ACTTGCCTGTCGCTCTATCTTCCTTTAARTTTCASYCTTGCG

***Edit your primer table in VSCode and save it.***

Every Natrix2 run requires a configuration file that is customized to our dataset and the individual data analysis parameters that you'd like to implement. Most of the work involved with using Natrix2 (beyond installing the software...) involves modifying this configuration file to suit your needs. The developers of Natrix2 provide us with several configuration files in the Natrix2 directory for data from various sequencing platforms that we can use as a starting point. The file that we'll use as a tempplate is named "nanopore_vsearch.yaml".

To run our analysis, we'll make a copy of this file and edit it for our project.

***Copy the nanopore_vsearch.yaml file and rename it with the analysis and date.***

```zsh

cp ./nanopore_vsearch.yaml ./nanopore_18S_test_20250604.yaml # something like this would suffice

```

There are several lines in the nanopore_vsearch.yaml file that we need to edit for our dataset and to suit your individual data analysis.

***Open your configuration file in VSCode to edit it.***

The first section we'll take a look at are in the "general:" section. Lines 3-8 pertain to the input and output paths for the analysis and the computing power that can be allocated to Natrix2. These have to be carefully edited for our directory structure and to match the capabilities of your computer.

If you've named your primer table in the same manner as I have, the first three lines should look the same. They point Natrix2 to the appropriate directories for your data, primers, and output for results.

*The cores and memory values are especially important and MUST be adjusted for your computer! If you let Natrix2 run loose without restricting the resources it can use, it could crash your computer.*

```yaml

general:
        filename: input/samples # The path / filename of the project folder, primertable (.csv) and configfile (.yaml). If the raw data folder is not in the root directory of Natrix, please add the path relative to the root directory (e.g. input/example_data)
        output_dir: output/results # Path to custom output directory / relative to the root directory of natrix. Do not use a dash in the folder name.
        primertable: input/18S_nanopore.csv # Path to the primertable. If the primertable is not in the root directory of Natrix, please add the path relative to the root directory (e.g. input/example_data.yaml)
        units: units.tsv # Path to the sequencing unit sheet. (name will be concatenated with output_dir)
        cores: 6 # Amount of cores available for the workflow.
        memory: 8000 # Available RAM in Mb.
        multiqc: TRUE # Initial quality check (fastqc & multiqc), currently only works for not yet assembled reads.
        demultiplexing: FALSE # Boolean, run demultiplexing for reads if they were not demultiplexed by the sequencing company (only Illumina support & slow).
        read_sorting: FALSE # Boolean, run read sorting for paired end reads if they were not sorted by the sequencing company (only Illumina support & slow).
        already_assembled: FALSE # Boolean, skip the quality control and read assembly steps for data if it is already assembled (only Illumina support).
        seq_rep: OTU # Type of sequence representative, possible values are: "ASV", amplicon sequence variants, created with DADA2 or "OTU", operational taxonomic units, created with SWARM or VSEARCH.

```

There are some other changes to the configuration file that we should make or investigate.

Take a look at the "nanopore" section. These parameters can be adjusted to quality filter/trim your data. the most important of these are minlen and maxlen, which remove sequences outside of our expected length range.

***Set these parameters in "nanopore" carefully and think about any adjustments you'd like to make to suit your dataset! Pay specific attention to quality_file, min_length, and max_length parameters.***

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

Two of the biggest decisions to make when using this pipeline are how to "cluster" similar sequences to represent species and how to determine the identity of those clustered sequences. With Nanopore data, we'll use the clustering algorith vsearch. The "vsearch_id" percent identity is a very important parameter that you may want to edit. It would be worth doing some research to determine what percent identities have been used in other Nanopore studies that use Natrix2.

***Adjust lines 65 and 66 to reflect your clustering parameters.***

```yaml

clustering: "vsearch" # Allows you to specify OTU clustering method to use. Your options are: swarm and vsearch. Nanopore only supports vsearch option.
vsearch_id: 0.97 #Percent identity for vsearch OTU clustering (1 = 100%).

```

To classify the taxonomy of the sequences, we're going to simply use BLAST to compare your clustered sequences to the SILVA database. We'll use SILVA, rather than the NCBI database, because SILVA is a curated database specifically for ribosomal DNA sequences like 18S. BLAST is a straightforward and intuitive way to identify sequences, so we'll use that for our first analysis of Nanopore data.

***Adjust taxonomic classification settings accordingly.***

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

The configuration file should be set!

***Once you've created this file, move it to the "input" folder***

Now, our data, configuration file, and primer file should be ready to go in the appropriate folders. We are ready to run Natrix2 using Docker.

This first command gets the Docker container for Natrix2 set up for our run. You'll have to change the file paths in this command to match those in your computer that point to the input, output, and database directories. You need to specify the *absolute* paths to each of these directories, *not* the relative paths.

```zsh

docker run -it --label natrix2_container -v /Users/callogorgia/Desktop/Natrix2/input:/app/input -v /Users/callogorgia/Desktop/Natrix2/output:/app/output -v /Users/callogorgia/Desktop/Natrix2/database:/app/database dbeisser/natrix2:latest bash

```

Now that you've set up the container, we can run the Natrix2 pipeline. The following command runs Natrix2 using the test configuration file. You'll adjust the name to reflect the name of your own configuration file, without the file extension.

```zsh

./docker_pipeline.sh test_docker # adjust to the name of your configuration file.

```

Once the Natrix2 pipeline is running, it will show up in Docker Desktop. You can check in on its progress there to make sure that it runs to completion. It will take a while to complete!
