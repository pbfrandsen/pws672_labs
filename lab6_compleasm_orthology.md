# Lab 6: Orthology prediction with compleasm

During this lab, we will be using [compleasm](https://github.com/huangnengCSU/compleasm?tab=readme-ov-file) to predict single copy orthologs across published whole genome sequences related to _Lednia tetonica_. `compleasm` is a reimplementation of `busco` that has been [shown to be faster and more accurate](https://doi.org/10.1093/bioinformatics/btad595). Each of you will be assessing a genome that you downloaded last week. The assignments are [here](https://docs.google.com/spreadsheets/d/1JyHKOnVlw0TYR7mXHR-6ysyHzA9T5o_CtK0IvhFcrqQ/edit?usp=sharing) under the "phylogenomics_assignments" tab.

#### Create a new `mamba` environment for `compleasm`

```
mamba create -n compleasm -c bioconda compleasm
```

#### Create new directory and copy genome assembly and BUSCO gene models

We'll be working in a new directory. Perhaps create it in your `~/nobackup/autodelete` folder.

```
cd ~/nobackup/autodelete
mkdir lab6
cd lab6
```

Now copy the genome(s) that you were assigned to from the directory where you downloaded the assembly last week into your `~/compute` directory.

Now copy the `mb_downloads` folder from `~/groups/fslg_pws670/nobackup/archive/pws672_w2026/6_compleasm` into your same directory. This is important because it contains all of the gene models for the single-copy orthologs that we are searching for.

```
cp -r ~/groups/fslg_pws670/nobackup/archive/pws672_w2026/6_compleasm/mb_downloads .
```

#### Create and submit your job

Visit the [job script generator](https://rc.byu.edu/documentation/slurm/script-generator) and create a new job reserving 4 CPUs and 6 GB of RAM per CPU. Set a wall time of 24 hours.

Copy the output into a new job file called, e.g. `compleasm.job`.

In the job file also add the commands to activate your `compleasm` environment

```
source ~/.bashrc
mamba activate compleasm
```

Now, add the command for `compleasm`. Here is an example commmand, you can modify the input and output to your particular genome.

```
compleasm run -a <genome.fasta> -o <genome_name>_compleasm -l insecta -t $SLURM_NPROCS
```

Now that you have that in place, go ahead and submit your job.

```
batch compleasm.job
```

Once your `compleasm` run is complete, change directories to your `<species_name>_compleasm/insecta_odb12` directory. In there, you should see a file called `gene_marker.fasta`. Go ahead and copy that file into our class directory (while also renaming it with your species name--this is important, because we will want to use the species name in a downstream step).

For example, if I was copying over the file for _Lednia tetonica_, I would use the following command. Just remember to change the species name to whatever species you are working with:

```
cp gene_marker.fasta ~/groups/fslg_pws670/nobackup/archive/pws672_w2026/7_alignment_models/lednia_tetonica.fasta
```


In the next lab, we will look at the output of the `compleasm` results and then use them to start generating a phylogeny. Before that, take a look at the output files in our output file. How complete do these genomes look in terms of single copy orthologs? Add your results to the [spreadsheet](https://docs.google.com/spreadsheets/d/1JyHKOnVlw0TYR7mXHR-6ysyHzA9T5o_CtK0IvhFcrqQ/edit?usp=sharing). Given these results, is there anything we should consider changing before moving forward with our phylogenomics work?
