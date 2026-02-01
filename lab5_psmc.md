# Lab 5: Demographic histories with `psmc`

### First, let's plot the results of the PCA and admixture analyses from Lab 4

First, navigate to the directory where you performed the PCA and admixture analyses. Then, copy `led_pop.txt`, `led_pop2.txt`, `led_pop_color.txt`, `PCA_plot.r`, and `admix_plot.r` from the `~/groups/fslg_pws670/nobackup/archive/pws672_w2026/4_angsd_pca_admix` directory into the same directory as your PCA/admixture results.

Activate the `r` environment that you installed during the last lab, i.e.:

```
$ source ~/.bashrc
$ mamba activate r
```

Now, examine the order of the samples in `led_pop.txt` and `led_pop2.txt`. You need to make sure that these are in the same order as the samples in the `bamlist.txt` that you used during lab 4. If they are not, reorder them so that they are in the same order.

#### PCA plot
To generate the PCA plot, you will need the `led_pop.txt` file and also ensure that there is a `.cov` file. If you followed the Lab 4 naming, this would be called `lednia_PCA.cov`. Make sure both of those files are in the same directory as where you are running the `PCA_plot.r` script. If they are all there, simply run the command:

```
$ Rscript PCA_plot.r
```

#### Admixture plot
For the admixture plots, you'll need to modify some of the commands in the `admix_plot.r` script. The script that you copied over will expect that you have the files `pen_pop.txt` and `penstemon_admix_k2.qopt` in your directory. It will write a new `.png` image file called `pen_admix_k2.png`. This will work well for the admixture analysis that you ran with a `K` of 2. However, you will need to modify both the name of the `qopt` file and the image filename for other values of `k`. You can simply change these in the script in your terminal window with your favorite text editor. You can then run the script with:

```
$ Rscript admix_plot.r
```

Make sure you make the changes between each value of `K` (2-4).

#### Computational assignment

Turn in your PCA plot and each of the admixture plots (representing `K` from 2-6). Along with these plots, interpret them. What do the axes mean? What do you learn about these the distance between the resequencing lines? Do you notice anything in the admixture plots that weren't clear from the PCA plots?

## Demographic analysis with `PSMC`
During this lab, we will use our bam files (generated during lab 2) to estimate demographic histories using the pairwise sequential Markovian coalescent (PSMC).

The first thing you'll want to do is create a `lab5` directory in your `~/nobackup/autodelete` directory. Change directories into your newly created `lab5` directory. Then copy both the reference genome and your particular `bam` files to your directory (only copy the bam files that you created during lab 2, if you need a reminder of your assignments, check out the [Google Sheet](https://docs.google.com/spreadsheets/d/1JyHKOnVlw0TYR7mXHR-6ysyHzA9T5o_CtK0IvhFcrqQ/edit?usp=sharing)). The reference genome can be found at `~/groups/fslg_pws670/nobackup/autodelete/2_resequencing_mapping/reference_genome/Ltet_assembly.fasta`. The various `bam` files are in `~/groups/fslg_pws670/nobackup/autodelete/2_resequencing_mapping/bam_files`.

### 1. Identify heterozygous sites using mpileup and bcftools, then export to a diploid `FASTQ` that can be converted into a special `PSMC` `FASTA` file.

#### First install a new `mamba` environment with the necessary version of `samtools`

```
mamba create -n samtools -c bioconda samtools=1.9 htslib=1.9 bcftools=1.9
```

* Create a new directory in `lab5` called `psmc`
* Change directories into that directory.
* This process is outlined in the commands below. The `-d` parameter is for minimum depth, which should be about 1/3 of the average depth of coverage and `-D` is maximum depth, which should be about twice the depth of coverage. This is a bit complicated for our resequencing data. You can estimate these from the [coverage stats](https://docs.google.com/spreadsheets/d/1JyHKOnVlw0TYR7mXHR-6ysyHzA9T5o_CtK0IvhFcrqQ/edit?usp=sharing) that you computed earlier. For example, sample `CV_04` had an average coverage of 28x. So `-d` would be ~`9` and `-D` would be `56`.


Modify the following commands for your particular samples (again, these are the same samples you were assigned for the read mapping in lab 2--each of you should have four) and submit the job from your `psmc` directory from a job file called `diploid_fastq.job` with a single thread and 18 GB of RAM with a 72 hour wall time. This step can take several hours. Make sure you substitute the appropriate path to the bam file and the appropriate values for `-d` and `-D` in the example below. Where ever there is a term within `<>` it means that you should modify that portion of the filename to match your individual codes.

+ **command**: 

```
source ~/.bashrc
mamba activate samtools
samtools mpileup -C50 -uf ../Ltet_assembly.fasta ../<bam_file> | bcftools call -c - | vcfutils.pl vcf2fq -d 9 -D 56 | gzip > <sample>_diploid.fq.gz
```

### 2. `PSMC`
* In this step, we will create the `PSMC` `FASTA` file. Do this by submitting this job using a job file in your `jobs` directory, perhaps called `psmc.job`. Remember to replace the `<sample>` with whatever sample code that you are using. These will be much faster than Step 1.

	+ **module to load**: ```module load psmc```
	+ **command**: ```fq2psmcfa -q20 <sample>_diploid.fq.gz > <sample>.psmcfa```

* Then, run `PSMC` using the `psmcfa` file generated in the previous step.
	+ **module to load**: `module load psmc`
	+ **command**: ```psmc -N25 -t15 -r5 -p "4+25*2+4+6" -o <sample>.psmc <sample>.psmcfa```

* Finally, we can create the `PSMC` plot without bootstrapping. This doesn't take a long time so go ahead and run it directly into the `lab5/psmc` directory. First, to create the PDF, you'll need to install `texlive-core` with `mamba`. You can do this with the following. First, make sure `mamba` is loaded.

```
source ~/.bashrc
```

Next, install `texlive-core`:

```
mamba install texlive-core
```

You'll only need to run those commands once, but make sure that `mamba` is active when you run the plotting script (you should have a `(base)` in front of your username).

To generate the plot, we'll use a mutation rate of `2.8e-09`. This is the published mutation rate for _Drosophila melanogaster_. Since I couldn't find any published mutation rates for stoneflies, we can start there. However, bumblebees have a higher mutation rate of `3.9e-09`. We could try both of those and see how it impacts the PSMC plot. We will use a generation length of 1 year. Substitute <sample> with your sample number.

   + **module to load**: `module load gnuplot`
   + **module to load**: `module load psmc`
   + **command**: ```psmc_plot.pl -p -u 2.8e-09 -g 1 <sample>_psmc <sample>.psmc```

The result should be a PDF file called <sample>_psmc.pdf.

### 3. Copy your `PSMC` to the lab directory and run `pcmc_plot.pl` on your other samples.
* Now copy your `.psmc` file (clearly named with your sample number) to `~/groups/fslg_pws670/nobackup/archive/pws672_w2026/5_psmc`. Once everyone has copied their files over, re-run the plot incoporating all of the different lines. If your plot looks weird, think about adjusting the axis with the `-Y` parameter. Often if there is noise, you will have a large increase in Ne, which is just due to noise (or due to misspecification of parameters).

See appendix II in the [psmc documentation](https://github.com/lh3/psmc) for more information. 


### Key takeaways for the lab write-up

* What do you notice about your `psmc` plots from the various resequencing runs? How are they similar? How do they differ? Why might this be?

* What do these, collectively, tell us about the demographic history of this species/the populations?