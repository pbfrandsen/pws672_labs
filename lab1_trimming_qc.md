#Lab 1: Trimming and QC of resequencing data

In today's lab, we will be trimming raw Illumina reads from each individual and running some QC on them to evaluate how the sequencing went.

###Trimming

Often, Illumina sequences come from the sequencing center with Illumina adapters still attached. Some sequencing centers remove those before delivering the data. Regardless, it doesn't hurt to run a trimming software just in case to remove the adapter sequence. There are many trimming utilities out there, but during this exercise, we will be using [`fastp`](https://github.com/OpenGene/fastp). `fastp` is an all-in-one trimming software. Take a look at their [GitHub page](https://github.com/OpenGene/fastp) to see all of the functions. It also outputs a `.json` file with a lot of information that can be useful to visualize later.

####1. Install `fastp`
We're going to use `conda` to install `fastp`. First, you'll want to create a new `conda` environment to house it (note, you can also use `mamba` for this process. Just substitute `conda` with `mamba` in the below commands. You're going to gain a lot of experience installing things in `conda` during this class and should get a hang of creating virtual environments and installing things. You can create a new `conda` environment with:

```$ conda create -n fastp```

You should press `y` to install the environment. Now you can activate the environment with:

```$ conda activate fastp```

Note that you have done is created a fresh environment and then activated it. You haven't yet installed anything into it. Luckily, `fastp` has a [`bioconda` installation](https://bioconda.github.io/recipes/fastp/README.html) and you can install it with:

```$ conda install -c bioconda fastp```

Give it a try. If it works, you are ready for the next step. Note that if you want to both create an environment *and* install a package, you can do that with a handy one-liner:

```$ conda create -n fastp -c bioconda -c conda-forge fastp```

You don't need to do that again because you have already installed it, but we will use the shortened version in the future


####2. Run `fastp` on your sequence

Now, generate a fresh job script from the [BYU job script generator](https://rc.byu.edu/documentation/slurm/script-generator). Choose a single node, a single processor, and 4 GB per processor. A wall time of 2 hours should be plenty. You might want to add your email and alerts when the job begins, aborts, or finishes.

For this experiment, you can access the raw Illumina reads in ` ~/groups/fslg_pws670/nobackup/archive/l_tetonica_raw/pop_data/01.RawData`. Go ahead and `cd` into that directory. There are ten subdirectories there. In each directory, there are three (or maybe five) files. The first is an `MD5.txt` file that has the checksum for your sequencing files. The other two are `FASTQ` files containing the paired raw reads. Each pair is in a separate file. In some cases, there are multiple `FASTQ` files, which is because the sequencing was performed in multiple runs.

Now, work with each other to split up which student does what. There are 24 different sets of reads so each of you can trim 4 sets. You should run all analyses in your `~/nobackup/autodelete` directory. You might want to change directories into `~/nobackup/autodelete` then make a new directory called `raw_data/raw_reads`, e.g., `mkdir -p raw_data/raw_reads`. Also make a new directory in the `raw_data` directory called `trimmed_reads`, e.g., `mkdir raw_data/trimmed_reads`. You can change directories into the `raw_data/raw_reads` directory. Go ahead and copy the raw reads that you are assigned to your `raw_reads` directory.

First, check the checksum for each file to ensure that the file was transferred correctly. You can do this with the program `md5sum`. To check the sums that are in the file, you can use `md5sum -c MD5.txt`.

Create a new file with `nano` or `vim` (probably `nano` if you are new to Unix) called `fastp_code.job` where you replace `code` with the code of the sample you are trimming. Now copy and paste the code from the Job Script Generator. It should look something like this:

```
#!/bin/bash

#SBATCH --time=8:00:00   # walltime
#SBATCH --ntasks=1   # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=4096M   # memory per CPU core
#SBATCH --mail-user=<your_email@address.com>   # email address
#SBATCH --mail-type=BEGIN
#SBATCH --mail-type=END
#SBATCH --mail-type=FAIL


# Set the max number of threads to use for programs using OpenMP. Should be <= ppn. Does nothing if the program doesn't use OpenMP.
export OMP_NUM_THREADS=$SLURM_CPUS_ON_NODE

# LOAD MODULES, INSERT CODE, AND RUN YOUR PROGRAMS HERE
```

Now, you'll want to navigate to the bottom of the file and add the command to activate your `conda` environment. This is:

```
source ~/.bashrc
conda activate fastp
```

The first line activates `conda`, which doesn't happen automatically when the job is on a compute node and the second line activates the `fastp` environment within `conda`, allowing you to use the `fastp` software. Next, you'll want to add the command for `fastp` to the file. Here is a command for one of the resequencing runs. When you add this, let's discuss each option in class. You'll want to be justified in your use of each command in the final paper so you should know what they are doing. You can find the documentation at the [GitHub repo](https://github.com/OpenGene/fastp). Note that whenever I use the `\` it is to indicate a line break. You can enter the command without them on a single line in the job file if you would like. But don't miss them because this is a common cause of headaches when people copy and paste commands. Notice that I am grabbing the raw reads from the `raw_reads/raw_data` directory and outputting the trimmed reads into the `trimmed_reads` directory.

***Hint: You have to be in the right directory (raw_reads) for this command to work. Make sure that you are there!***

```
fastp -i <name>_L1_1.fastq.gz \
-I <name>_L1_2.fastq.gz \
-o ../trimmed_reads/<name>_R1_trimmed.fastq.gz \
-O ../trimmed_reads/<name>_R2_trimmed.fastq.gz \
--unpaired1 ../trimmed_reads/<name>_R1_unpaired.fastq.gz \
--unpaired2 ../trimmed_reads/<name>_R2_unpaired.fastq.gz \
-R '../trimmed_reads/<name>_fastp_report' -j '../trimmed_jsons/<name>_fastp.json' \
-h '../trimmed_reads/<name>_fastp.html' -Q -g
```

Once you have the command pasted in and modified for particular sample, you can submit the job to the cluster with:

`$ sbatch fastp_code.job`

If you want to check on its progress, you can check it with:

`$ squeue -u <username>`

You can also check on the size of the trimmed files. Once they approach the size of the raw files, you know the job is close to being complete. You can do this with a long list command:

`$ ls -lhtr`

Once your job is complete. Check out the size of the resulting files. You should observe that the `trimmed.fastq.gz` files are much larger than the `unpaired.fastq.gz` files. If they aren't, that's your first clue that something went wrong. Why would that be? 

####3. Install `MultiQC`

Now we are going to run a program called `MultiQC` on the output files from the trimming jobs. [`MultiQC`](https://multiqc.info/) is a powerful piece of software that allows you to visualize the quality of multiple samples in one place.

Luckily, `MultiQC` also has a [`bioconda` image](https://bioconda.github.io/recipes/multiqc/README.html). Let's exit the `fastp` environment if we're still in it:

`$ conda deactivate`

Now, you can create the `MultiQC` environment and install it all in one command:

`$ conda create -n multiqc -c bioconda -c conda-forge multiqc`

Once it is installed, you'll need to activate it with:

`$ conda activate multiqc`

####4. Copy the `json` files into your own directory and run `MultiQC`

Once you are finished trimming, copy your output `json` files to the group `trimmed_jsons` directory (`~/groups/fslg_pws670/l_tetonica_raw/pop_data/trimmed_jsons`). `MultiQC` only needs to be run once, but I would like to get you all practice so I'm going to have you copy the contents of that directory (wait until everyone's trimming job is complete!) to your own `autodelete` directory. You can do this with a recursive copy.

`$ cp -r trimmed_jsons ~/nobackup/autodelete/raw_data`

This command will copy the directory `trimming_jsons` and all of its contents into your `~/nobackup/autodelete/raw_data` directory. Note that the `~` is a shortcut for your home directory. If there is any part of that command that you don't understand, take some time to make sure you understand it (as you should for each command we enter).

Once that copy is complete, you can `cd` into your `~/nobackup/autodelete/raw_data` directory.

`$ cd ~/nobackup/autodelete/raw_data`

Now if you list the contents of that directory, there should be a new folder called `trimmed_jsons`. Go ahead and change into the `trimmed_jsons` directory (I'm not giving you the command this time because you should know how to do it by now!).

Once there, ensure that the `multiqc` conda environment is activated. You can do this by making sure that there is a `(multiqc)` at the beginning of your terminal prompt. If it says `(base)`, you might need to activate it.

Once it is activated, run `MultiQC` to generate your report.

`$ multiqc .`

This command will run `MulitQC` on your current directory (indicated with a `.`). The `.` is very important. Make sure it is there. When `MultiQC` is complete, it will generate a report called `multiqc_report.html`. Download it and check it out. What do you see? What do you think? You will turn this report in to get credit for this exercise.