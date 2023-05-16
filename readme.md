# Notes on High Performance Computing

*More information in the [VSC documentation](https://docs.vscentrum.be/en/latest/index.html)!*

## Your account

- Create a VSC account on the [website](https://account.vscentrum.be/).
- Generate a public/private key pair (must be a 4096-bit key):

```
ssh-keygen -t rsa -b 4096
```

- Upload public key to give your computer access to your VSC account

```
cat /Users/christof/.ssh/id_rsa.pub
```

## Hardware/Software stack

- The VSC exists of several clusters: 
    - Tier-1 hardware -> only for approved projects
    - Tier-2 hardware -> several clusters per institution (KU Leuven user can access Genius, wICE, and Superdome).
    - Every cluster consists of nodes (Genius/wICE):
        - Login nodes (4/0)
        - Thin nodes (230/172) with 2 CPU each (18/36 cores) and  192/256 GB RAM
        - Big memory nodes  (10/5) with 2 CPU each (18/36 cores) and  786GB/2 TB RAM
        - GPU nodes (22/4)
        - AMD nodes (4/0)
        - Interactive nodes (0/5)
- This is only a limited introductory presentation: 
    - I use a simple data transferring protocol (scp/sftp or git)
    - The presentation focusses on running jobs on MATLAB and Julia
    - I use the wICE cluster, which is the KU Leuven/U Hasselt’s most recent Tier-2 hardware, and the Genius cluster, which is the previous cluster (but still contains the login nodes)
    - There is some big memory node information
    - Nothing about GPU! (I have some additional information for the fans...)
- Every job costs credits:
    - Credits are very cheap: 1M credits = €3.5
    - Within the lp_largescalemacaulay group, I try to manage the available credits and buy enough credits...
    - See the [credit website](https://icts.kuleuven.be/sc/onderzoeksgegevens/HPC_credits#hoeveel).

## Getting access

- Logging in: 

```
ssh vsc31826@login.hpc.kuleuven.be
```

- You are now working on one of the four Genius login nodes.
- Do not run jobs here! They really do not like it!
- You can check the balance of your projects

```
sam-balance
```

- An easier approach to logging in is via a `.ssh/config` file.

## About modules (= the software stack)

- Get a list of all modules: 

```
module av
```

- Search for a specific module via 

```
module spider <name>
```
    
```
module spider julia
module spider matlab/R2019a
```

- Load a module: 
    
```
module load matlab/R2020a
```

- List with loaded modules: 

```
module list
```

- Other commands: 

```
module unload <name> 
module help <name> 
module purge
```

These are the only prerequisites! **Let’s start running jobs now!**

## Simple job

Note that VSC changed from Torque to Slurm, which uses different commands. Some parts of the VSC documentation are deprecated. More information about Slurm can be found in the documentation page about [slurm](https://docs.vscentrum.be/en/latest/antwerp/SLURM_UAntwerp.html#antwerp-slurm).

We start with three basic commands: 

```
sbatch <options> <jobname.sh>
squeue
scancel <jobid>
```

- A job is a batch script

```
#!/bin/bash
```

- Running a batch can be done via 

```
sbatch <options> <jobname.sh>
sbatch -A lp_largescalemacaulay hello_world.sh
```

- The output is gathered in the file

```
slurm-<jobid>.out
```

- You can check your queued and running jobs via

```
squeue
```

- Cancel a job via 

```
scancel <jobid>
```

Jobs are scheduled to run depending on your requested resources and available nodes. You can check the available nodes via 

```
slurmtop [optional --cluster=wice]
```

To change the requested resources, you can change the job options: 

- Name of the project (the only essential option!):

```
-A <nameoftheproject>
```

```
-A lp_largescalemacaulay
```

- Cluster that you want to use (use Genius right now!):

```
--cluster=<nameofthecluster> 
-M <nameofthecluster>
```

```
--cluster=genius (default)
--cluster=wice
```

- Amount of time that you want to request (if you succeed the requested time, the your job gets killed - allowed formats =  mm, mm:ss, hh:mm:ss, d-hh, d-hh:mm, or d-hh:mm:ss):

```
--time=<time> 
-t <time>
```

```
--time=01:00:00 (default)
```

- Number of node that you need:

```
--ntasks=<numberofnodes> 
-n <numberofnodes>
```

```
--ntasks=1 (default)
```

- Number of cpus per node that you need:

```
--cpus-per-task=<numberofcores> 
-c <numberofcores>
```

```
--cpus-per-task=1 (default)
```

- Required memory per cpu (allowed formats = 10mb or 10gb):

```
--mem-per-cpu=<requiredmemory> 
```

```
--mem-per-cpu=8gb (default)
```

- Type of partition that you want to use:

```
--partition=<partitionname> 
-p <partitionname>
```

```
--partition=bigmem
```

- Repeat your job via an array (you can even use the iteration count to do different things in every iteration, see the documentation)

```
--array=0-<numberofrepeatsminus1>
```

```    
--array=0-3 [-> this runs 4 times]
```

There are also some other options that help you with processing the output, handling the errors, using a different job name, or sending e-mails:

```
--output=<filename>
--error=<filename>
--job-name=<filename>
--mail-user=<emailaddress>
```

You can also add these parameters in your bash script (= your job). You simply need to add #SBATCH in front of the parameter value. For example: 

```
#SBATCH -A lp_largescalemacaulay

#SBATCH --cluster=wice
#SBATCH --time=01:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=8gb
```

## Data transfer

I typically use scp to transfer data between my computer and the VSC nodes.

```
scp <localpath> <id>@login.hpc.kuleuven.be:<serverpath>
```

```
scp ~/Documents/foo.txt vsc31826@login.hpc.kuleuven.be:FolderOnServer/serverfoo.txt
```

## More advanced job in MATLAB

Let us now push a job in MATLAB via MacaulayLab!

- The first step is to clone MacaulayLab on your server:

```
git clone git@gitlab.esat.kuleuven.be:Christof.Vermeersch/macaulaylab-public.git
mv macaulaylab-public MacaulayLab
```

- Now, we can create a MATLAB bash script or run the job directly (-> avoid this, because you are working on the login node!)

```
module purge 
module load matlab/R2019a

cd ~/MacaulayLab

matlab -nojvm -nodisplay -r "<matlabfunction>; exit;"
```

- Finally, you can submit the job!

```
sbatch matlabscript.sh
```

## Some tools 

I use some bash scripts to simplify working with the VSC:

```
vsc-cat
vsc-status
vsc-update
vsc-cancel <jobid>
vsc-download <localpath>
vsc-submit <options> <jobserverpath>
```

Add them to a folder (e.g., ~/Documents/Scripts) and add that folder to your path

```
export PATH=~/Document/Scripts/:$PATH
```

Do not forget to make the bash scripts executable!

```
chmod 755 <namescript>
```

```
chmod 755 vsc-cat
```
