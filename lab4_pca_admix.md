#Lab 4: Population structure: PCA, ADMIXTURE

###Before we get started into the population structure analyses, let's look at the results from last week.

First, install `R`  and activate it using `conda`:

```
$ source ~/.bashrc
$ mamba create -n r r
$ conda activate r
$ conda install r-tidyverse
```

Now open `R`:

```
$ R
```

Once within `R` install the `stringi` package:

```
> install.packages("stringi")
```

It will prompt you to choose a `CRAN` mirror. You can choose the one closest. For me that was option `68: USA (OR)`. Type in the number of the `CRAN` mirror and press `enter`.

Since this will take a little while, it might make sense to open an additional `ssh` session and start on the next steps.

This will allow you to run the R scripts on the supcomputer. Alternatively, you can simply download the R scripts and the output data files and run it within, e.g. [Rstudio](https://www.rstudio.com/).

###Combine Tajima's D estimates from your two populations into single file

You can check out the format of the theta estimate files with, e.g. (use whatever population code you used for your samples):
```
$ head cv.thetas.pestPG
```

In my output, the Tajima's D estimate is in the 9th column so you can cut out those values from the `pestPG` files and redirect them to a new file with: 

```
$ cut -f9 cv.thetas.pestPG > tajimas_d_cv.txt
$ cut -f9 wc.thetas.pestPG > tajimas_d_wc.txt
```

Then you can combine the two files, while maintaining different columns with `paste`:

```
$ paste tajimas_d_cv.txt tajimas_d_wc.txt > tajimas_d_both.txt
```

Go ahead and open them with your preferred text editor and replace the headers with the taxon names. If you used the example above, the CV population would be in the first column and WC population would be in the second. Go ahead and changes the headers to `CV` and `WC`, separated by a tab. This will ensure that you can tell the samples apart when we plot them. If you look at the first few lines of `tajimas_d_both.txt` with `head`, it should look something like this:

```
$ head tajimas_d_both.txt 
CV      WC
-0.321417       -0.082358
-0.462809       -0.241159
-0.041191       0.371440
0.102866        0.373471
0.138438        0.286188
0.180380        0.358081
0.094888        0.294139
```

###Create a new file with the `Fst` values

You can also check out the results of your sliding window Fst analysis with:

```
$ head window.fst.txt
```

In this case, we'd like the 5th column in a new file for plotting in R, so we can make a fresh file containing only `Fst` values with:

```
$ cut -f5 window.fst.txt > fst_cv_wc.txt
```

Now open up the file with your favored text editor (`nano` or `vim`) and add `Fst` to the first line. If you look at the first few lines of your file with `head`, it should look something like this:

```
$ head fst_cv_wc.txt 
Fst
0.172951
0.175078
0.204180
0.221292
0.208176
0.199710
0.194982
0.151947
0.135191
```

###Create plots for Tajima's D and `Fst`

Copy `tajimas_d_plot.r` and `fst_plots.r` from the `~/groups/fslg_pws670/nobackup/archive/pws672_w2026/3_angsd_pop_stats` to your current directory.

Make sure that your `R` conda environment is active (you should see `(r)` in front of your cursor if it is). If it isn't, activate it. Another alternative is to download the scripts to your computer and to run them on the input files using Rstudio. If you do this, you may have to modify the scripts so that they include full paths to the input files and the output files.

To run `tajimas_d_plot.r` on the supercomputer, first ensure that you are running it in the same directory as the `tajimas_d_both.txt` file that you created above. To run it, simply enter the following command:

```
$ Rscript tajimas_d_plot.r
```

You will see some text printed to the screen and the result should be a new PDF file called `tajimas_d.pdf`. Go ahead and download that file and view it on your computer.

Now, run `fst_plots.r`, again ensuring that `fst_cv_wc.txt` is in the same directory. If your comparison was for a different set of populations, then you will need to change the argument in the command below so that it refers to your particular set of populations.

```
$ Rscript fst_plots.r fst_cv_wc.txt
```

After you run this script, you should have two new files, `Fst_window.pdf` and `Fst_density.pdf`. Go ahead and download those and take a look.

What do you notice about Tajima's D? How about the `Fst` values? Turn these in and interpret them for your computational lab assignment this week on Learning Suite. Use full sentences in your interpretation and ensure it is accurate. If you want to add some conjecture, make sure you qualify it as such.

###1. Install more `ANGSD` dependencies

```
$ mamba activate angsd
$ mamba install cython scipy pandas gxx
$ mamba deactivate
```

###2. Generate genotype likelihoods with `ANGSD`

Create a new directory in your `~/nobackup/autodelete` directory called `lab4_angsd_pca`. Navigate into that directory.

Generate a list of the paths to all of the resequencing data bam files. An easy way to do this is, e.g.:

```
$ ls ~/groups/fslg_pws670/nobackup/autodelete/2_resequencing_mapping/bam_files/*bam > bamlist.txt
```

Make sure your `bamlist.txt` file has 24 lines:

```
$ wc -l bamlist.txt
```

If it doesn't, then something went wrong with the generation of that file.

We will use that list as input to `ANGSD` to generate the genotype likelihoods.

Make a new job file called, e.g. `angsd_gl.job`, and specify 12 threads with 4 GB of RAM per thread and request 24 hours (you can use the [BYU job script generator][1]. Add the following to run `ANGSD` to generate genotype likelihoods.

```
source ~/.bashrc
mamba activate angsd
angsd -GL 1 -out PCA -nThreads $SLURM_NPROCS \
   -doGlf 2 -doMajorMinor 1 -SNP_pval 1e-6 -doMaf 1 \
   -bam bamlist.txt
```

Go ahead and submit that job. It may take a couple of hours to complete. In the meantime, you can begin the next steps. For any step below that relies on the `PCA_beagle.gz` file, you'll need to wait until the job you just submitted is complete.

###3. Generate a covariance matrix using `PCAngsd`

First install `PCAngsd`:

```
source ~/.bashrc
mamba activate angsd
wget https://github.com/Rosemeis/pcangsd/archive/refs/tags/v.0.99.tar.gz
tar -xvzf v.0.99.tar.gz
cd pcangsd-v.0.99/
python setup.py build_ext --inplace
cd ..
```
Once `PCAngsd` is installed and your job from step 2 is complete, you can set up a new job file called, e.g., `angsd_pca.job`. Make sure that you request 10 threads and 4 GB of RAM per thread. This one should be pretty fast so you can probably get by with requesting 2 hours of wall time. Once you have created the job file, add the following to generate the covariance matrix from the genotype likelihoods.

```
source ~/.bashrc
mamba activate angsd
python pcangsd-v.0.99/pcangsd.py -beagle PCA.beagle.gz -o lednia_PCA \
    -threads $SLURM_NPROCS
```

Go ahead and submit that job.

###4. Run Admixture analysis with `NGSAdmix`

`NGSAdmix` is now part of the `ANGSD` package and can perform admixture analysis using the beagle file of genotype likelihoods that you generated in step 2. Create a job file called, e.g., `angsd_admix.job` with the following commands. The parameter that you need to think about here is `K`, which as you learned in the lecture is the number of ancestral populations. We have four different localities, but aren't sure how many different ancestries (nearby populations may have substantial ongoing gene flow. We will start by setting `K` to 2.

```
source ~/.bashrc
mamba activate angsd
NGSadmix -likes PCA.beagle.gz -K 2 -o lednia_admix_k2 -P $SLURM_NPROCS
```

When the job is finished, you will have a couple of new files in the `angsd` directory. `lednia_admix_k2.fopt.gz` is a gzipped file that contains the estimated allele frequencies for each population for each locus. `lednia_admix_k2.qopt` contains the estimated admixture proportions for each individual.

Now modify the job file (including the `-K` argument and the name of the output, `-o`, for a `K` of 3 and 4. This will allow you to observe the effect of `K` on the admixture analysis and may aid in figuring out the number of distinct ancestries.

[1]:	https://rc.byu.edu/documentation/slurm/script-generator