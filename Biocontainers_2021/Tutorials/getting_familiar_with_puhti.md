---
topic: Batch jobs
title: Tutorial - Getting familiar with Puhti
---
# Getting familiar with Puhti

The main purpose of this tutorial is to get familar wit working in CSC
Puhti supercomputer. 

Some exercises are done in interactive mode using the [sinteractive](https://docs.csc.fi/computing/running/interactive-usage/) 
command and some as batch jobs.

## Interactive use: Retrieving data from bio repositories

> These exercises cover retrieving data from various commonly used bio data repositories.

- We will do these exercises using the [sinteractive](https://docs.csc.fi/computing/running/interactive-usage/) command (substitute xxxx with correct project number):

  ```text
  sinteractive --account project_xxxx
  ```

- Move to `/scratch` directory area of the course project (unless there already):

  ```text
  cd /scratch/project_xxxx  # substitute xxxx with correct project number
  ```
- Create a directory for yourself and move to it (unless there already):

  ```text
  mkdir $USER
  cd $USER
  ```

  💭 Everyone in the project shares the same `/scratch` directory, so
  it is a good idea to use subdirectories for each user and task, so 
  you won't accidentally delete or overwrite each others files.

  🗯 In normal usage it may be a good idea to even use `chmod` command 
  to alter file access rights so that only you have write access to
  your own subfolder, but please do not do this in the course project, 
  as it would make clean-up after course harder.

- You can find more information about this in [Disk areas](https://docs.csc.fi/computing/disk/)
page in the Docs.

### 1. Downloading data with curl

- `curl` and `wget` are general tools to download data from an URL.

- Download a dataset from internet using `curl` and uncompress it. The dataset contains some *Pythium* genomes with  related BWA indexes.
    ```text
    curl https://a3s.fi/course_12.11.2019/pythium.tgz > pythium.tgz
    ls -ltr
    tar -zxvf pythium.tgz  
    ls -ltr
    ```

### 2. Downloading data with NCBI edirect

- In this exercise we are using some specialized tools to download
data. To use them we must first load the `biokit` module. 

    ```text
    module load biokit
    ```
- Create directory `cellulose_synthase` and move to this new directory:

   ```text
   mkdir cellulose_synthase
   cd cellulose_synthase
   ```

- Next we use [NCBI edirect tool](https://docs.csc.fi/apps/edirect/) to retrieve some data.

- Check how many proteins are found the NCBI protein databanks for *Pythium* species (`count` row in the results):

    ```text
    esearch -db protein -query "Pythium [ORGN]" 
    ```

- Then check the number of proteins: **cellulose synthase 1**, **cellulose synthase 2** and **cellulose synthase 3** that are found for *Pythium* species.

- For cellulose synthase 1 this can be done with:

    ```text
    esearch -db protein -query "Pythium [ORGN] AND cellulose synthase 1 [PROT]"
    ```

- Do the same for the other proteins.

- Retrive the cellulose synthase 3 sequenses in Fasta format

    ```text
    esearch -db protein -query "Pythium [ORGN] AND cellulose synthase 3 [PROT]" | efetch -format fasta > cesy3.fasta
    ```

- Then run `esearch` command that tells how many **cellulose synthase 3** sequences there are in total in the NCBI protein database?


## Batch jobs: Align the cellulose synthase 3 set with mafft

1. Make a file called `mafft.sh`:
    ```bash
    module load nano   # The compute nodes do not have nano by default
    nano mafft.sh
    ```
2. Copy the following contents into the file and change "project_xxxx" to the correct project name:
    ```bash
    #!/bin/bash
    #SBATCH --job-name=test           # Name of the job visible in the queue.
    #SBATCH --account=project_xxxx    # Choose the billing project. Has to be defined!
    #SBATCH --partition=test          # Job queues: test, interactive, small, large, longrun, hugemem, hugemem_longrun
    #SBATCH --time=00:10:00           # Maximum duration of the job. Max: depends of the partition. 
    #SBATCH --mem=8G                  # How much RAM is reserved for job per node.
    #SBATCH --ntasks=1                # Number of tasks. Max: depends on partition.
    #SBATCH --cpus-per-task=1         # How many processors work on one task. Max: Number of CPUs per node.

    # 
    module load biokit
    mafft cesy3.fasta > cesy3_aln.fasta
   
    ```

    💬 In nano you can use `ctrl + o` to save and `ctrl + x` to exit.

3. Submit the job to the queue with:

    ```bash
   sbatch test.sh
    ```
`
4. And study the results:

- What files were created?

- Study the alignment in more detail:
    ```text
    infoalign cesy3_aln.fasta
    showalign cesy3_aln.fasta
    ```

## Extra exercise for the fast ones: Downloading with enaDataGet

This exercise should ne run using `sinteractive`. Module `biokit` needs to be loaded.
- Check the options of enaDataGet with command:

    ```text
    enaDataGet -h
    ```

- Download a file (Pythium iwayamai  genome assembly)

    ```text
    enaDataGet AKYA02000000 -f fasta
    gunzip AKYA02.fasta.gz 
    ls -ltr
    ```

- study the downloaded file:

    ```text
    head -20 AKYA02.fasta
    tail AKYA02.fasta
    infoseq_summary  AKYA02.fasta
    ```

- Then compare the cellulose synthase 3 sequences against the genome using BLAST

    ```text
    pb tblastn -query cesy3.fasta -dbnuc AKYA02.fasta -out blast_result.txt
    ```
  💬 Command `pb` (Parallel BLAST) is designed for situations, where the query file includes 
  large amount of sequences. It splits the query task into several subjobs, that can be run 
  simultaneously using the resources of the server very effectively. Because `pb` will create and
  run a batch job, the `pb` command itself can be run in the login node. For more information, check
  the Docs CSC page for [BLAST](https://docs.csc.fi/apps/blast/)