# Lab 2: Mapping resequencing data

In this lab we will be mapping resequencing reads to the reference genome. For this exercise, each student will use the same assignments from the last lab to map reads to the reference genome (each student should map four samples).

There are many short-read mappers to choose from and there are trade-offs to each. We will be using `minimap2`, one of the most commonly used all-purpose mappers.

Before we start the lab, please copy all of your trimmed reads files from the last lab to `~/groups/fslg_pws670/nobackup/autodelete/2_resequencing_mapping/trimmed_reads`. You will be mapping these back to the reference genome.

### 1. Install `minimap2` with `mamba`
There is a `minimap2` package in [`bioconda`](https://anaconda.org/bioconda/minimap2). To install it, make sure you are in your `base` environment (your cursor should be prepended by `(base)`). If you don't see anything in `()` before your cursor, try:

`$ source ~/.bashrc`

Create a new `mamba` environment called `minimap2` and install `minimap2` in the same command. You can do this with:

`$ mamba create -n minimap2 -c conda-forge -c bioconda minimap2`

### 2. Index your genome
Before you can map the reads to your genome, you need to build an index, which allows the mapping to occur quickly. We can do this with `minimap2`. You only need to do this once per genome per data type (then you can map multiple sets of reads to the indexed genome, however, if you switch data type, you have to re-index). Do do this, navigate to a new directory that I created at `~/groups/fslg_pws670/nobackup/autodelete/2_resequencing_mapping/reference_genome`. The genome `FASTA` file will be in that directory. Work with your classmates to decide who will index the genome. One of you create a new job file called `minimap_index.job` and ask for 10GB of RAM and a single thread.

```
source ~/.bashrc
mamba activate minimap2
```

Add that to the bottom of your job file. To index the genome (and only one person per group needs to do this), you use the command:

```
minimap2 -x sr -d Ltet.mmi Ltet_assembly.fasta
```

Make sure that the name of the `FASTA` file matches that within the folder.

You can submit the job with:

`$ sbatch minimap_index.job`

When it is complete, check out the new files that are listed in the directory where your genome sits. You should have a new file ending in `.mmi` (as specified in your job command).

### 2. Map the reads

For each individual sequencing run, you will need to map the reads separately.

Work with your team to choose who maps which set of reads to the genome. You only need to map each set of reads to each genome once.

Change directories into `~/groups/fslg_pws670/nobackup/autodelete/2_resequencing_mapping/job_files`

To map the reads, create a new job file. You can use the [job script generator](https://rc.byu.edu/documentation/slurm/script-generator) and reserve a single node, 12 GB of RAM and 24 hour wall-time. You also might want to enter your email address so that you receive alerts about your job begin/end/abort.

Paste the embedded `SLURM` directives into a new job file. Make sure you load both your new `minimap2` environment and the `samtools` module.

```
source ~/.bashrc
conda activate minimap2
module load samtools
```

For each job, we will need to specify the path to the reference genome, the paths to the trimmed reads, and a unique name for the output bam file. This is where I'm going to let you work together to ensure that you don't overlap. Here is a sample job command if you were mapping the `CV_01` sample to the `Ltet_assembly.fasta` assembly and running the job from the `job_files` directory:

```
minimap2 -ax sr ../reference_genome/Ltet.mmi \
../trimmed_reads/CV_01_R1_trimmed.fastq.gz \
../trimmed_reads/CV_01_R2_trimmed.fastq.gz \
| samtools sort -o ../bam_files/CV_01.bam -
```

Note the `..` notation in the path. That just means "up" one directory.

The `-a` specifies that you want to output a `sam` file, the `-x sr` specifies that you are using short-read data. The `|` takes the output sam file and "pipes" the stream directly into `samtools` that sorts and outputs a `bam` file.

Go ahead and do this for each resequencing run. Make sure you work together with your lab mates to ensure that you don't duplicate effort/name things the same way.

One of you should also map the reference PacBio HiFi reads back to the reference **paul will do this**. To do this you can use the HiFi reads in the `FASTQ` files  found in the `pacbio_reads` subdirectory in `2_resequencing_mapping`. The person who maps the PacBio HiFi reads should modify the first part of the command to `minimap2 -ax map-hifi` followed by the paths to the reference genome and the raw PacBio HiFi reads.

Ok, now sit back, relax, and hopefully everything maps! We will start the next lab by examining the results of this lab.


