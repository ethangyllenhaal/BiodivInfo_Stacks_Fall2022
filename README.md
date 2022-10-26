# Biodiversity Informatics Stacks Overview

Here are the main commands we'll be running for the Stacks class in Biodiversity Informatics.

Before we get into stacks, we'll have to log on to CARC and make our working directory. Fortunately, everything here should take up a maximum of 20 Gb, which is within the range of your alloted home directory storage (100 Gb, up to 200 Gb for a week). First, we'll log into our machine. I'm assuming we'll be running this on hopper, as it tends to have the fastest queue times:

    ssh yourname@hopper.alliance.unm.edu

Then, we will make a directory for our project (mkdir) and move into it (cd). If you run the "pwd" command, you can see that we're in a broader file system called "users", which will be important later on.

    mkdir stacks_example
    cd stacks_example
    pwd

Once we have our working directory, we need to set up a conda environment for Stacks. This is pretty straightforward, and can be done to install most software you'd want to use on CARC! First, you have to load the miniconda module. Because it is used so much, you don't even need to sue its full name.

    module load miniconda3

Then, you can create a conda environment! Here it is named "stacks-example-env", but you can name it whatever you want. The --channel options tell it where to download files from. It will first load all the files you need, and you need to approve it by typing "yes" and hitting enter.

    conda create --name stacks-example-env --channel bioconda --channel conda-forge stacks

Then, we'll set up some directories:

    mkdir reads
    mkdir stacks_output
    mkdir input_files
  
After this, we'll make our first script, which will copy over your reads! This is called a batch script (specifically a slurm script). Pretty much, it has some options you set at the top that make sure you get the resources you need, then it runs every command after that in order. There are many text editors, but nano is a good option for beginners, as it lists commands at the bottom (with ^ corresponding to the control key). The most important ones to know are Exit (Ctrl + X) and Write Out (AKA save, Ctrl + O). We'll start editing our script like:

    nano rsync_reads.slurm

Then, instead of having you write the full script, just copy this into your command line:

    #!/bin/bash

    #SBATCH --ntasks=1
    #SBATCH --time=1:00:00
    #SBATCH --job-name=rsync
    #SBATCH --output=rsync_out_%j
    #SBATCH --error=rsync_error_%j
    #SBATCH --partition=general
    #SBATCH --mail-type=FAIL,END
    #SBATCH --mail-user=egyllenhaal@unm.edu

    # two commands to show you the differences between error and output files
    echo "Hello world!!"
    cat "Hello world!!"

    rsync --progress /path/to/Genus_species* reads/

Once your reads are copied over, we need to run Stacks. Fortunately this is pretty simple, particularly because these parameters seem to work fairly well!

    #!/bin/bash

    #SBATCH --ntasks=1
    #SBATCH --cpus-per-task=8
    #SBATCH --time=10:00:00
    #SBATCH --job-name=example
    #SBATCH --output=example_out_%j
    #SBATCH --error=example_error_%j
    #SBATCH --partition=general
    #SBATCH --mail-type=FAIL,END
    #SBATCH --mail-user=egyllenhaal@unm.edu

    module load miniconda3
    source activate stacks-example-env

    cd $SLURM_SUBMIT_DIR

    denovo_map.pl -T 8 -o stacks_output/ \
                  -m 10 -M 5 -n 5 \
                  --popmap popmap_example \
                  --samples reads/

