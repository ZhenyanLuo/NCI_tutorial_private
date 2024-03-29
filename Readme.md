### What is NCI
Read this:

https://nci.org.au/

### {} steps for starting to use NCI

#### Step 1 Signup

You should register for a new NCI account 

https://my.nci.org.au/mancini/signup/0

You should recieve a username after providing your information, the username should be consisted of two lower case characters and four numbers.

#### Step 2 Login

Login by using mobaxterm or wsl on window11 (VPN free) , Terminal for Mac user (Applications>Utilities>Terminal), termminal for Unix/Linux 

```
ssh {username}@gadi.nci.org.au
```

#### Step 3 Set up your home 

### (Note: since we are using shared conda environment now, this step has been changed)

type 
```
nano .bashrc 
```
replace the end of the file like this (replace {project_id} with your project id)
:
```
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/g/data/{project_id}/miniconda/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/g/data/{project_id}/miniconda/etc/profile.d/conda.sh" ]; then
        . "/g/data/{project_id}/miniconda/etc/profile.d/conda.sh"
    else
        export PATH="/g/data/{project_id}/miniconda/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
unset SUDO_UID SUDO_GID SUDO_USER
```
And each time submit script which need to use conda, add these commands after PBS setting (add the project id between /data/ and /miniconda/)
```
set -xue
source /g/data/ /miniconda/etc/profile.d/conda.sh
conda activate {your env}
```


Make your own folder in gdata

```
mkdir -p /g/data/{your project id}/{your user name}
```

You can check your home space by using:

```
quota -s -f /home
```

##### About storage

Gadi has two places to store files, gdata and scratch, available space of /gdata is 3TB, /scratch is 1TB. In principle, files stored in /scratch is not permanent.


You can check space already used by your project:

```
nci_account -P {your project id}
```

Check how much space does each user in your project used:

scratch :

```
nci-files-report -f scratch --group {your project id}
```

gdata:

```
nci-files-report -f gdata --group {your project id}
```

You can also use this:

```
lquota
```

You may notice when you use 'nci_account -P' to check space already used, you can also get a usage report.

I personally regard KSU as a type of currency, the more you used, the more you will be charged.

For example:

----------------------------------------------------------------------
Usage Report: Project={} Period=2021.q4 <-----q4 means the 4th quarter

    Grant:    75.00 KSU     <----- total ksu in each quarter

    Used:    40.34 KSU     <----- ksu you have used in this quarter
    
    Reserved:     0.00 SU
    
    Avail:    34.66 KSU     <----- ksu you can use in this quarter


You can check how they charge compute jobs: 
https://opus.nci.org.au/display/Help/2.+Compute+Grant+and+Job+Debiting

How different types of job queue cost:

https://opus.nci.org.au/display/Help/2.2+Job+Cost+Examples

Use normal queue in most situations. 

#### Step 4 Write a script for submitting PBS job

NCI already has installed some basic modules, you can check available modules at https://opus.nci.org.au/display/Help/Gadi+Latest+Software+Update

If the modules you hope to use already be included in this list, you won't need to install them in your conda environment. 

Assume you hope to use a module which already install, write a script like this:
```
#!/bin/bash
#PBS -N Example_Job
#PBS -q {normal/hugemem}
#PBS -l mem={}GB
#PBS -l walltime={}:00:00
#PBS -l jobfs={}GB
#PBS -l ncpus={}
#PBS -l wd
#PBS -l storage=scratch/{project_id}+gdata/{project_id}
#PBS -M {your_email_address}                    <-------if you provide your email address, you will receive a mail if your job exceed memory you requested

set -xue

module load {module}/{module version}

{put your command here}

module unload {module}/{module version}

```

If you installed your modules in conda enviornment, you can use following codes to replcae 'modele load' and its relevant codes:

```
set -xue
source /g/data/ /miniconda/etc/profile.d/conda.sh
conda activate {your env}
```

#### Step 5 Submit your job


After you write this script, you can submit it with:

```
qsub {your script}

```
After submission, you will get a job id

Then you can trace your job progress with the job id provided:

```
qstat -fx {job_id}

```

If you forget your job id

```
qstat -u {your user id} -sw

```

If you want to delete job:

```
qdel {job_id}
```

A quick look at PBS commands : https://www.cqu.edu.au/eresearch/high-performance-computing/hpc-user-guides-and-faqs/pbs-commands

Different status of the job status:

Q: waiting

R: running

H: holding, possibly your project doesn't have enough KSU or the job memory you requested exceed

More potential reasons cause your job not running can be found here: https://opus.nci.org.au/display/Help/FAQ+1%3A+Why+My+Jobs+are+NOT+Running

If your job use memory exceed requested, you might receive email like this:

PBS Job Id: 28860049.gadi-pbs
Job Name:  {}
Job to be deleted at request of root@gadi-cpu-clx-1429.gadi.nci.org.au

Then you should request more memeory and submit this job again


#### Step 6 Submit a interactive job 

Sometime you might need to run interactive job, submit an interactive job will allow you to work on a compute node directly and use NCI like using command line. Only single CPU core, 4 GB memory and 30 minutes running time is allowed on the login node, so submit an interactive job if you want to use node like using command line. 

```
qsub -I -P {project_id} -q {normal/express/hugemem} -l walltime={}:00:00,ncpus={},mem={}GB,jobfs={}GB,storage=gdata/{project_id}+scratch/{project_id},wd

```

Once the interactive job is ready, it will give you a job id and you can see @gadi-login-0x ~ change to @cpu~xxxxx (not quite remember) 

After you finish your work, you can simply type 'quit' to end your interactive job


Remember: interactive jobs will stop once you log out from the login node, keep the window open before finishing your job   


------------------------------------------------------------------------------
#### Some tips 




#### What should I do if my home is full but I still need to create a new conda environment

You can remove some environments you no longer need. You can run this to save these environment settings and create new environments with these config files when you need them.


```
conda env export >{name you like}.yml

```

Create the same environment with the configure file you made:

```
conda env create --name {name you like} -f {your configure file name} 

```



Or

you can create conda environment in another directory (not quite recommend)

For example

```
conda create --prefix {to/the/folder/you/like}

conda activate {to/the/folder/you/like}

```

You can find packages you install in this folder. 



#### Why my job not running (both submitting job and interactive job)
First, run the following code to have a look at not running reason with following code, the comments under the JobID gives reason:

```
qstat -u $USER -Esw
```

The common reason is that your job is waiting in the queue for available resources such as cups, mem, Sus and licenses. The solution is waiting for a while or ask scheme manager for more Sus. You can check the licenses waiting queue on <https://usersupport.nci.org.au/license-status.html> .The comments for reasons can be like:

```
Not Running: Insufficient amount of resource: ncpus
Not Running: Insufficient amount of resource: mem
Not Running: Job would conflict with reservation or top job
Not Running: PBS Error: Could not reserve allocation from project “<prj>” to run job.
Not Running: PBS Error: Waiting for software licenses
```

If the comment shows that the usage of gdata is over the limit. You may use the following command line to remove all the useless packages and files to spare space. Or move the unused files in gdata to other places. It is a good habit to clean the gdata regularly.

```
conda clean --all
```

If the job limit wall time get into a scheduled downtime, you need to reduce the wall time or run the job after the downtime, the comment will be like:

```
Not Running: Job would cross dedicated time boundary
```

If the node that your job runs on got issues the comment will be like as following, if the job does not run in the next 10 minutes, you need to contact NCI for solution.

```
Not Running: PBS Error: Execution server rejected request
```

If the jobs has too many failed attempts to start, the comment will appear as following. It can be something wrong in the submitted script or the job was sent to failed node. You need to check your script or contact NCI for help.

```
job held, too many failed attempts to run
```

#### How to transfer files between remote servers or local computer

It is recommanded to use rsync rather than scp because rsync allows to start again from the last broken point. 

rsync:
```
# copy local files to remote server, your terminal should be in local computer.
rsync -avzh /local/file/path  {user}@{remote_host}:/remote/file/path 
# for example:
rsync -avzh /Users/XXXXX/Desktop/research/project/script XX1234@gadi-dm.nci.org.au:/scratch/XX1/XX1234

# copy remote files to local computer, your terminal should be in local computer.
rsync -avzh {user}@{remote_host}:/remote/file/path /local/file/path
# for example:
rsync -avzh XX1234@gadi-dm.nci.org.au:/scratch/XX1/XX1234/code /Users/XXXXX/Desktop/research/project


```

scp:
```
# copy local files to remote server, your terminal should be in local computer.
scp -r /local/file/path  {user}@{remote_host}:/remote/file/path 
# for example:
scp -r /Users/XXXXX/Desktop/research/project/ XX1234@gadi-dm.nci.org.au:/scratch/XX1/XX1234

# copy remote files to local computer, your terminal should be in local computer.
scp -r {user}@{remote_host}:/remote/file/path /local/file/path
# for example:
scp -r XX1234@gadi-dm.nci.org.au:/scratch/XX1/XX1234/project /Users/XXXXX/Desktop/research/
```
If you want to transfer files between two servers, you may login the destination server first, for example, transferring files from Gadi to lab computer:

```
# login lab computer 
ssh XXXX@130.56.33.39

# transfer files from Gadi to lab computer, '.' represent the current directory.
rsync -avzh XX1234@gadi-dm.nci.org.au:/scratch/XX1/XX1234/project .
```


#### Why I can't transfer large files to NCI

Details about transfering files to or from NCI: 

No-interactive job will be killed after 30 minutes, to transfer large files, you can submit a PBS job for transfering.   



#### About Singularity 

You might want to read this first: https://sylabs.io/guides/3.0/user-guide/build_a_container.html

First, you should load the singularity module on login node or submit a interactive job
```
module load singularity
```
NCI not allow user to make writable singularity container by using fake-root or sandbox, you can't use mysql.

Singularity image can be big, some files will automatelly save in ./singularity (your home path), which may make you get a error like:

WARNING: mkdir {xxxx}/.singularity: disk quota exceeded

To avoid this error, you can create a folder in scratch then link this folder to the .singularity folder in your home path

```
mkdir -p /scratch/{your project id}/{your username}/.singularity

ln -s /scratch/{your project id}/{your username}/.singularity ${HOME}/.singularity
```

Then you should be able to download singularity image by using 'singularity pull', you can download the image from dockerhub, container library...

For example, download a docker image in a directory (specify in --dir option)

```
singularity pull --dir {your directory} docker://{something}
```

After it finish download, you will find a .sif file in your folder, this is the singularity image. 

Normally, if you have root permission, you can use

```
sudo singularity build --sandbox {sandbox name you like} {sif file of singularity image}
```
This should allow you to build a writable singularity container.

Or you can use the fake-root option:
```
singularity build --fakeroot {sandbox name you like} {sif file of singularity image}
```

However, NCI not allow to build writable container.

To solve this problem, we can copy the file from the image then bind these folders to the folder in the singularity image.



```
singularity  exec  {image sif file} cp -r {folder you wants to copy} {absolute path you want to copy the file in}

```
Then bind this writable folder to the folder in container

For the data you wants to use as input file, you can also place these files in a folder and bind to the container (notice: the folder made by you is not neccessary to be already existed in the container)

You have two different methods can bind folders to container

Method 1:

Bind the folder (data) on host to /mnt folder in the container

```
singularity exec --bind /data:/mnt {image sif file} ls /mnt
```
To bind multiple folder

```
singularity shell --bind /opt,/data:/mnt {image sif file}
```
Method 2:

Set environment variable

```
export SINGULARITY_BIND="$HOME, {your folder on host}:{folder in container}"
```

After setting all the directories you want to blind in one of these methods, you can run the singularity image with
```
singularity shell {your sif file}
```

### Use jupyter lab in Gadi

First, configure Jupiter account in login node of gadi:

```
module load pangeo/2021.01
jupyter server --generate-config
jupyter server password
```

Then, writer and submit a Pango job:

create a script:
```
nano run_ipynb_job.sh
```

Here is the script of job (run_ipynb_job.sh):
```
#!/bin/bash
#PBS -N pangeo_test
#PBS -P <project code>
#PBS -q normal
#PBS -l walltime=5:00:00
#PBS -l ncpus=16
#PBS -l mem=50GB
#PBS -l jobfs=100GB
#PBS -l storage=scratch/<project code>+gdata/<project code>
module load pangeo/2021.01
pangeo.ini.all.sh
sleep infinity

```
Then submit the job:
```
qsub run_ipynb_job.sh
```

There will be a file called 'client_cmd' in the current dirctory. Open the file and find ssh cmd like following:
```
# open the file
cat client_cmd

# There are two ssh cmd like following:
ssh -N -L 8321:gadi-cpu-clx-0526.gadi.nci.org.au:8321 XX1234@gadi.nci.org.au  
ssh -N -L 8757:gadi-cpu-clx-0526.gadi.nci.org.au:8757 XX1234@gadi.nci.org.au 
```
Copy one of the ssh cmd into your local terminal and type your gadi login password. The port number can be found in ssh cmd, for example, the port number is 8321 in the first ssh cmd shonwn above. Then type 'localhost:{port number}' into your local browser and type Jupyter password. The jupyter lab will be open
