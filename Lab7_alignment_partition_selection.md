# Lab 7: Ortholog alignment and partition selection


### 1. Move single copy orthologs from `compleasm` output
The output from the `compleasm` lab are here:

```
~/groups/fslg_pws670/nobackup/archive/pws672_w2026/7_alignment_models
```

### 2. Create new unaligned `FASTA` files for each single copy ortholog

**Note**: I will be doing this step with you during class. Please do not do this step yourselves as it will overwrite what I am doing. However, I will outline it here so you know how to do it for future datasets.

Make sure that you are in the `~/groups/fslg_pws670/nobackup/archive/pws672_w2026/7_alignment_models` directory. The `awk` command that we enter here will generate unaligned ortholog files for each ortholog for each species. We want to save these into the `unaligned_aa` directory so we will first `cd` into that directory. We will also run a `sed` command to fix the headers. Note I put the command on multiple lines so that you can see it, but running it on a single line (without the back slashes) would be the best idea to avoid any issues.

```
cd unaligned_aa
awk 'BEGIN{RS=">"; FS="\n"} {m=(split($1,b,"_")); fnme=b[1]".fas";\
 n=split(FILENAME,a,"/"); print ">" a[2] "\n" $2 >> fnme; close(fnme);}'\
  ../*fasta
sed -i -e 's/.fasta//g' *fas
```

Now look at a couple of alignments and make sure they look like you expect. They should have a `FASTA` header that is simply a species name, followed by amino acid sequences.

### 3. Copy unaligned alignments into your own directory
Now, you will all be working independently. First you should make a new directory in your `nobackup/autodelete` directory. Perhaps call it `lab7/unaligned_aa`

```
mkdir -p ~/nobackup/autodelete/lab7/unaligned_aa
```

Copy the unaligned `FASTA` files into your new directory. Before you enter this command, make sure you are in the `~/groups/fslg_pws670/nobackup/archive/pws672_w2026/7_alignment_models/unaligned_aa` directory.

```
$ cp *fas ~/nobackup/autodelete/lab7/unaligned_aa
```

### 4. Align the single copy orthologs

First, you'll need to create a new `mamba` environment that has `mafft` installed.

```
source ~/.bashrc
mamba create -n mafft -c bioconda mafft
```

To process more than one file at a time, we are going to use an `sbatch` array. First, we need to have a list of the files that we want to analyze. To do this, we'll simply list the `FASTA` files in the directory and re-direct them to a new text file.

```
ls *fas > locus_names.txt
```

Now check how many lines there are in the `locus_names.txt` file. That is how many tasks we'll need to feed our array.

```
$ wc -l locus_names.txt
```

Here is some example text that you'll want to put in your array file (you could call it `mafft_array.job`. Make sure you substitute your email (it will only email you if it fails) and the appropriate number of loci in the `#SBATCH --array=1-35` line. This job file only has 35, you're going to have more than 2,000 (the number from the `wc -l` command above is the one that you should substitute).

```
#!/bin/bash

#SBATCH --job-name=mafft_array
#SBATCH -o %A_%a.mafft.out
#SBATCH -e %A_%a.mafft.err
#SBATCH --mail-type=FAIL
#SBATCH --mail-user=<your_email@email.com>
#SBATCH --mem-per-cpu=4gb
#SBATCH --time=24:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --array=1-35

source ~/.bashrc
conda activate mafft

#parses locus_names.txt for array 
locus=$(head -n $SLURM_ARRAY_TASK_ID locus_names.txt | tail -n1)

mafft-linsi ${locus} > `basename ${locus} .fas`_aligned.fas
```

Then you can submit the job with

```
sbatch mafft_array.job
```

When the job is complete, you should have many new files that end in `_aligned.fas`. Make sure that you don't go to the next step until all of the alignments are complete (you can check the job with `squeue -u <username>`. Once they are complete, look at a few of the files. How are the `_aligned.fas` files different than their unaligned counterparts? What do you notice?

###5. Run `Aliscore` and `ALICUT` on your aligned sequences

Make a new directory in your `lab7` directory called `aligned_aa`. If you are still in your `unaligned_aa` directory, you would do this with:

```
mkdir ../aligned_aa
cd ../aligned_aa
```

Now move all of the aligned files into your new directory.

```
mv ../unaligned_aa/*aligned.fas .
```

Copy the `Aliscore` and `ALICUT` scripts to your new directory

```
cp ~/groups/fslg_pws670/nobackup/archive/pws672_w2026/7_alignment_models/ALICUT_V2.31.pl .
cp ~/groups/fslg_pws670/nobackup/archive/pws672_w2026/7_alignment_models/Aliscore* .
```

Now, we're going to make another list called `aligned_loci.txt`

```
ls *aligned.fas > aligned_loci.txt
```

Then we'll make another array job called `aliscore_array.job`. Here is what that might look like. Remember to modify the `#SBATCH --array` parameter and the email.

```
#!/bin/bash

#SBATCH --job-name=aliscore_array
#SBATCH --mail-type=FAIL
#SBATCH --mail-user=<your_email@email.com>
#SBATCH --mem-per-cpu=4gb
#SBATCH --time=24:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --array=1-35

#parses aligned_loci.txt for array 
locus=$(head -n $SLURM_ARRAY_TASK_ID aligned_loci.txt | tail -n1)

perl Aliscore.02.2.pl -i ${locus} -N
```
After you fix those parameters, you can submit this job with, e.g:

```
sbatch aliscore_array.job
```

Once your `Aliscore` jobs are complete, (double check with `squeue -u <username>`), you are ready to run `ALICUT`. First, maybe download a couple of the svg files. What do you notice? Are there any regions of your alignments that are indiscernable from noise? `ALICUT` will cut those portions out. It runs very quickly so you can run it interactively. Simply enter the command:

```
perl ALICUT_V2.31.pl
```

It will request some information. We are using amino acid data so we don't have to worry about the nucleotide specific parameters. Instead, you can simply press `s` and it will start the process and automatically cut out the regions of the alignment that are indistinguishable from random noise. The newly cut alignments will start with the prefix `ALICUT`.

### 6. Concatenate your alignment and create partition definition file

Make a new directory in your `lab7` directory called `alicut_aa`

```
cd ..
mkdir alicut_aa
cd alicut_aa
```

Now, copy all of the `ALICUT` alignments to that directory.

```
mv ../aligned_aa/ALICUT*fas .
```

We're going to concatenate the alignments with a untility called `FASconCAT`. Copy it from `~/groups/fslg_pws670/nobackup/archive/pws672_w2026/7_alignment_models/FASconCAT_v1.11.pl`.

Now run `FASconCAT` with:

```
perl FASconCAT_v1.11.pl
```

You will encounter a screen that allows you to choose a couple of options. press `i` and `enter` to make sure you output an "info" file. Then press `s` and `enter` to start the concatenation.

When this is complete, you will have a couple of new files in your directory, `FcC_smatrix.fas`, which is the concatenated `FASTA` file, and `FcC_info.xls`, which is the file that contains information about what was concatenated. Now we'll convert that file into a partition definition file that can be read by `IQ-TREE`, which is what we'll use to select our models. I wrote a little `python` script that you can use to do that. You can copy it over from the `~/groups/fslg_pws670/nobackup/archive/pws672_w2026/7_alignment_models` directory. It is called `extract_partition_file_from_fcc.py`.

Run it with:

```
python extract_partition_file_from_fcc.py FcC_info.xls
```

You will now have a new file in that directory called `partition_def.txt`.

### 7. Start partition search using `IQ-TREE`

Create another new directory in your `lab7` directory called `iqtree` and navigate into it.

```
mkdir ../iqtree
cd ../iqtree
```

Copy concatenated alignment and partition definition.

```
cp ../alicut_aa/FcC_smatrix.fas .
cp ../alicut_aa/partition_def.txt .
```

Download the `IQ-TREE` executable.

```
wget https://github.com/iqtree/iqtree2/releases/download/v2.1.3/iqtree-2.1.3-Linux.tar.gz
```

Unpack and unzip the `tar.gz` file.

```
tar -xvzf iqtree-2.1.3-Linux.tar.gz
```

Now make a new job file using the [research computing job script generator](https://rc.byu.edu/documentation/slurm/script-generator). Select 4 GB of memory per processor 18 processor cores and 72 hours of wall time. Enter your email and request notifications for the job beginning, end, and abort.

Copy that into a new job file called `iqtree_model_selection.job`. Then at the bottom, you'll want to paste the following command, which will do the partition selection.

```
iqtree-2.1.3-Linux/bin/iqtree2 -s FcC_smatrix.fas \
-spp partition_def.txt -nt $SLURM_NPROCS -safe \
-pre penstemon_partition -m TESTMERGEONLY -mset LG+G
```

This will select an appropriate partition scheme. In the next lab, we will start tree searches which will be combined with further model selection on the newly chosen partitioning scheme.
