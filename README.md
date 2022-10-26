# Biodiversity Informatics Stacks Overview

## Logging in and setting up directories

Here are the main commands we'll be running for the Stacks class in Biodiversity Informatics.

Before we get into stacks, we'll have to log on to CARC and make our working directory. Fortunately, everything here should take up a maximum of 20 Gb, which is within the range of your alloted home directory storage (100 Gb, up to 200 Gb for a week). First, we'll log into our machine. I'm assuming we'll be running this on hopper, as it tends to have the fastest queue times:

    ssh yourname@hopper.alliance.unm.edu

Then, we will make a directory for our project (mkdir) and move into it (cd). If you run the "pwd" command, you can see that we're in a broader file system called "users", which will be important later on.

    mkdir stacks_example
    cd stacks_example
    pwd

Then, we'll set up some directories:

    mkdir reads
    mkdir stacks_output
    mkdir input_files
    mkdir alignment_files
    mkdir snp_files
  
## Making a slurm script and transferring reads

After this, we'll make ourfirst script, which will copy over your reads! This is called a batch script (specifically a slurm script). Pretty much, it has some options you set at the top that make sure you get the resources you need, then it runs every command after that in order. There are many text editors, but nano is a good option for beginners, as it lists commands at the bottom (with ^ corresponding to the control key). The most important ones to know are Exit (Ctrl + X) and Write Out (AKA save, Ctrl + O). We'll start editing our script like:

    nano rsync_reads.slurm

Then, instead of having you write the full script, just copy this and paste it into your terminal once nano is open (right click for Windows, Ctrl+V for Mac). Before you save it, change "Genus_species" to your species's name:

    #!/bin/bash

    #SBATCH --ntasks=1
    #SBATCH --time=1:00:00
    #SBATCH --job-name=rsync
    #SBATCH --output=_rsync_out_%j
    #SBATCH --error=_rsync_error_%j
    #SBATCH --partition=general
    #SBATCH --mail-type=FAIL,END
    #SBATCH --mail-user=username@unm.edu

    # two commands to show you the differences between error and output files
    echo "Hello world!!"
    cat "Hello world!!"
    cd $SLURM_SUBMIT_DIR

    # copies reads into your reads directory
    # takes advantage of the fact the reads are also in the shared "users" directory
    rsync --progress /users/lnbarrow/RawReadsBiodivInf/Genus_species* reads/

You will then submit your script with sbatch:

    sbatch rsync_reads.slurm

There are a few ways to look at files without editing them. Once you save your script, you can use "cat" to spit it out into the output. This can also be used to direct the contents of files into other programs.

    cat rsync_reads.slurm

A cleaner way of viewing files is to use less, which lets you scroll through files without putting them in your terminal. Once you open it, you can exit by just hitting "q". There are lots of shortcuts for navigating and searching the file you're viewing, too! You can also automatically fill files names by hitting tab once if there are no conflicts, twice if there are multiple options. Try typing this out but hitting tab after "rsy".

    less rsync_reads.slurm

We'll then look at the "out" and "error" files. Some programs will write errors to out files and output to error files, but most follow the rule that neutral things go in the output and bad thing go in the error. Try using less to see what's in your output file first. Then look at your error file. Each one has a job number associated with it, so tab completing is very helpful! Try to figure out what caused the error in the error file.

    less _rsync_out_#####
    less _rsync_error_#####

We can also look at our read files, but they are compressed ("gzipped"), so we need to use species versions of cat and less that process gzipped files. First, we need to see the file names associated with the reads using the list command (ls). The ll command is just a shorthand version of "ls -l", and gives more information:

    ll reads

Then, let's look at one of them with zless:

    zless reads/Name_of_read_file

Then, we'll use zcat to spit the contents of the file into the word count (wc) command, and use the -l flag to count the line number. This number divided by four is the number of reads in the file.

    zcat reads/Name_of_read_file | wc -l

## Running Stacks

### Making a conda environment

Now that we have our reads, we need to set up a conda environment for Stacks. This is pretty straightforward, and can be done to install most software you'd want to use on CARC! First, you have to load the miniconda module. Because it is used so much, you don't even need to sue its full name.

    module load miniconda3

Then, you can create a conda environment! Here it is named "stacks-example-env", but you can name it whatever you want. The --channel options tell it where to download files from. It will first load all the files you need, and you need to approve it by typing "yes" and hitting enter.

    conda create --name stacks-example-env --channel bioconda --channel conda-forge stacks

### Making two population maps

In order to run Stacks, we need a population map (popmap). We will make two different kinds, one that lumps populations and one that has each individual separate (the latter is only relevant for making specific output files). The easiest way to make these is in excel. The first column has to be the sample name (matching the read files from before), and second is the population (real or artificial), with the two separated by a tab. It's easiest to make these in excel, but you can make them in nano as well, you just have to make sure the sample name is right. You can then copy them in to a popmap with nano. We'll make the first popmap like this:

    nano popmap_genus_species

Then copy over a two-column populatiom map, something like this

    Hyla_femoralis_LNB00715 Pop1
    Hyla_femoralis_LNB00716 Pop1
    Hyla_femoralis_LNB00975 Pop2
    Hyla_femoralis_LNB00976 Pop2
    Hyla_femoralis_LNB01285 Pop3
    Hyla_femoralis_LNB01451 Pop3
    
Now we need an individual-specific popmap, which we'll make like:

    nano popmap_phylo_genus_species

And enter a popmap like this. Note that the "population" is a shortened version of the sample name to a two letter abbreviation of the species, the first letter of the collector name, and the collector's number. This is to make sure our file matches the "strict phylip" format:

    Hyla_femoralis_LNB00715 HfL00715
    Hyla_femoralis_LNB00716 HfL00716
    Hyla_femoralis_LNB00975 HfL00975
    Hyla_femoralis_LNB00976 HfL00976
    Hyla_femoralis_LNB01285 HfL01285
    Hyla_femoralis_LNB01451 HfL01451


### Running Stacks's denovo assembly

With the environment set up, it's time to set up the denovo assembly pipeline in Stacks. 

    nano run_stacks.slurm

Fortunately the script is pretty simple, particularly because these parameters seem to work fairly well! 

    #!/bin/bash

    #SBATCH --ntasks=1
    #SBATCH --cpus-per-task=8
    #SBATCH --time=10:00:00
    #SBATCH --job-name=example
    #SBATCH --output=_example_out_%j
    #SBATCH --error=_example_error_%j
    #SBATCH --partition=general
    #SBATCH --mail-type=FAIL,END
    #SBATCH --mail-user=username@unm.edu

    module load miniconda3
    source activate stacks-example-env

    cd $SLURM_SUBMIT_DIR

    denovo_map.pl -T 8 -o stacks_output/ \
                  -m 10 -M 5 -n 5 \
                  --popmap popmap_example \
                  --samples reads/

Then submit the script like because. It should take ~15 minutes to run for small and ~30 for large datasets, so if it finishes early there was probably an issue.

    sbatch run_stacks.slurm

POPULATIONS NEEDS VCF, STRUCTURE, AND PHYLIP

Sampling site info -> summ stats, vcf, structure

No sampling site info -> phylip

TODO:

Add looking at/counting fastqs to markdown and ppt

Copying to device:

