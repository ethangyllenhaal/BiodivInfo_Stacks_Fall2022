# Biodiversity Informatics Stacks Overview

Here are the main commands we'll be running for the Stacks class in Biodiversity Informatics.

The first step is to set up a conda environment for Stacks. This is pretty straightforward, and can be done to install most software you'd want to use on CARC! First, you have to load the miniconda module. Because it is used so much, you don't even need to sue its full name.

    module load miniconda3

Then, you can create a conda environment! Here it is named "stacks-example-env", but you can name it whatever you want. The --channel options tell it where to download files from. It will first load all the files you need, and you need to approve it by typing "yes" and hitting enter.

    conda create --name stacks-example-env --channel bioconda --channel conda-forge stacks

Then, we'll set up some directories:

    mkdir reads
    mkdir stacks_output
    mkdir input_files
  
