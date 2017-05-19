# Part 4: Getting to know Hipergator (and Stampede)
Hipergator and Stampede, or really any modern research-grade supercomputer, are very powerful machines that work a little differently from your laptop. Below are a few things you should know about how these computers are set up and how to use them.

For simplicity, I'll only talk about Hipergator here, but for the most part Stampede and Hipergator work in a similar way.

---------------
## Infrastructure
It might be helpful to think of Hipergator as being 3 different partitions of nodes (aka processors) on a single computer. The nodes in the first partition are the **login nodes**, the second are the **scratch storage nodes**, and the third are the **compute nodes**. Each are used for different purposes, and you will need to understand all three to use the computer correctly.

**--- Login nodes ---**

As you might have guessed, these are the nodes to which you log in when you connect via ``ssh`` to hipergator. Your ``/home/your_username`` directory is the most important thing located here. Everything you store here is automatically backed up and pretty safe, but you have a limited amount of storage so you don't want to store all your research data here. The main thing you will keep here are software packages that you install. Most users prefer to have a ``software/`` folder under their home folder so they can keep everything tidy. If you've been following the previous objectives, you should have a folder called ``miniconda3`` under your home folder.

**--- Scratch storage nodes ---**

Okay, since you don't want to store your research data (which is often several hundred GB) under your home folder, where should you put it? That's what the scratch nodes are for. These guys are not backed up but you have a much larger allotment of storage here. Actually, the group shares a quota of a few TB, so it's still not nice to hog all of that up, but you can keep your research data here with no problem.

That means that you should switch to these nodes whenever you're running calculations that generate data. These nodes are located under ``/ufrc/`` (**UF** **R**esearch **C**omputing) on Hipergator. On other machines like Stampede, they're under ``/scratch/``. So to switch to them, simply ``cd /ufrc/hennig/your_username``.

Eventually, you will create directories here to store your calculation results, etc. You want to submit your calculations from these nodes.

**--- Compute nodes ---**

When you submit a job from the scratch nodes, you're sending it to one of the compute nodes to do the actual work. You almost never work directly on these nodes, unless you want to [run a developmental session](https://wiki.rc.ufl.edu/doc/Development_and_Testing) to test something quickly. These perform the calculation as specified by the instructions in your submission script and return the results to the scratch node directory where you launched the calculation from. An example submission script for Hipergator is detailed in the next section.

## Usage

Below is an example submission script, which you would prepare somewhere on your scratch filesystem and then submit to the compute nodes to run.

```
#!/bin/bash
#SBATCH --job-name=TEST              # The name of the job
#SBATCH -o out_%j.log                # The stdout file
#SBATCH -e err_%j.log                # The stderr file
#SBATCH --qos=hennig-b               # Your group queue name
#SBATCH --nodes=1                    # Number of copmute nodes you want working on your job
#SBATCH --ntasks=1                   # Number of processors you want working on your job
#SBATCH --mem-per-cpu=100mb          # Amount of virtual memory to reserve for running your job
#SBATCH -t 00:01:00                  # Amount of time you think your job will take (if exceeded, job will die!)
cd $SLURM_SUBMIT_DIR                 # Tell the compute node to go into your working directory on scratch
echo 'Your job worked!' > test.txt   # Perform the actual command you want
```

When you have a submission script ready and located on your scratch filesystem, submit it to the compute nodes via ``sbatch whatever_you_named_your_submission_script``.

Hipergator uses a tool called SLURM to schedule all of its jobs, since hundreds of users are submitting thousands of jobs to their compute nodes every day. Our group has several hundred cores reserved for only our use, but we are often using all of them. This means that you might have to wait a little while (or even a few days in some cases) for your job to get its turn and run. You can check the status of all the jobs you currently have submitted via ``squeue -u your_username``. This will give an output like this:
```
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
7198142 hpg2-comp mp-57109  mashton PD       0:00      1 (Priority)
```
The main things to read from this output are the JOBID, ST (STatus) and TIME. ST is the job status, and "PD" means pending. The only other status you are likely to see is "R" for running. If you decide you want to cancel a pending or running job, type ``scancel JOBID``, where you should replace JOBID with the actual JOBID corresponding to the job. Sadly, if you notice that the TIME is getting close to the total walltime you requested for the job, there is very little you can do and your job will probably be killed by the SLURM scheduler. Just ask for more time the next time you submit.