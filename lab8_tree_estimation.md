# Lab 8 tree estimation

In this lab, we're going to generate both a tree from the concatenated supermatrix and a species tree from multiple gene trees. 

We're going to estimate a maximum likelihood tree using the partitioning scheme that we estimated in Lab 7. We're also going to select the best substitution model for each subset in the partitioning scheme using `ModelFinder`, which is a piece of software implemented into `IQ-Tree`. 

#### 1. Estimate tree from concatenated supermatrix
First, you're going to navigate back to your `iqtree` directory from lab 7. If you used the command given in `lab7`, you should have a new file there called `lednia_partition.best_scheme.nex`. This contains the information for your partitioning scheme. We are now going to select protein models for each of the new subsets selected during partition finding using `ModelFinder` and then estimate a tree using that fixed model. We will also generate 1000 ultrafast bootstrap replicates to estimate node support. Generate a new job file with the [research computing job script generator](https://rc.byu.edu/documentation/slurm/script-generator). Select 4 GB of RAM per processor, 18 processor cores, and 72 hours of wall time. Make a new job file called `iqtree_supermatrix.job`. Paste in the information from the job script generator and then add the following command. Make sure that the `iqtree-2.1.3-Linux` folder is still in your current directory.

```
iqtree-2.1.3-Linux/bin/iqtree2 -s FcC_smatrix.fas \
-spp lednia_partition.best_scheme.nex -nt $SLURM_NPROCS -safe \
-pre lednia_concatenated -m MFP -bb 1000 -bnni
```

Go ahead and submit the job. When it is complete, you will have a new treefile called `lednia_concatenated.treefile`. This is the result of your supermatrix analysis. We will compare this tree to the `ASTRAL` species tree at the end of the lab.

#### 2. Estimate individual locus trees

Navigate to your `alicut_aa` directory. In this directory, you will have all of your cut alignments. We're going to use a job array to generate an individual gene tree for each locus.

Make a list of your alignments. You can do this with:

```
ls *aligned.fas > locus_alignments.txt
```

Now check how many alignments you have with:

```
wc -l locus_alignments.txt
```

This will be the number of jobs you feed your array. Copy the iqtree executable directory (that we downloaded from GitHub) from lab 7 into this directory. Assuming iqtree is in the same directory as `alicut_aa`, the command would be:

```
cp -r ../iqtree/iqtree-2.1.3-Linux .
```

Now generate a new job, `iqtree_array.job`. In it, you will put something like this, except make sure that you modify the `#SBATCH --array` directive so that it matches the number of loci in `locus_alignments.txt`.

```
#!/bin/bash

#SBATCH --job-name=iqtree_array
#SBATCH --mail-type=FAIL
#SBATCH --mail-user=<your_email@email.com>
#SBATCH --mem-per-cpu=4gb
#SBATCH --time=1:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --array=1-3096

#parses locus_alignments.txt for array 
align=$(head -n $SLURM_ARRAY_TASK_ID locus_alignments.txt | tail -n1)

iqtree-2.1.3-Linux/bin/iqtree2 -s $align -m MFP -bb 1000 \
-pre `basename $align _aligned.fas`
```

#### 3. Estimate species tree using `ASTRAL`.

First, we need to install a new `mamba` environment for the software we are using. `ASTRAL` is now included in the [`ASTER`](https://github.com/chaoszhang/ASTER?tab=readme-ov-file) suite. Go ahead and create a new environment with ASTER installed:

```
mamba create -n aster -c bioconda aster
```

Navigate to the directory that holds your gene trees.

Now, concatenate all of the gene trees into a single input file.

```
cat *treefile > all.tre
```

You can double check the number of trees in that file with:

```
wc -l all.tre
```


Now run `ASTRAL` on your gene trees:

```
mamba activate aster
astral4 -i all.tre -o astral.tre
```

This should pretty fast with so few taxa (don't need to make a supercomputer job). The species tree will be in `astral.tre`. Go ahead and download it and visualize it with [FigTree](https://github.com/rambaut/figtree/releases).

FigTree will prompt you to name the new values on the branch lengths. These are local posterior probabilities (you can just say `lpp`). 

Take a look. Click the arrow next the "Node Labels" in teh lefthand side of FigTree. Go ahead and select the `lpp`. This will then display the local posterior probabilities.
