# compute-canada

steps to set up your compute canada resources and important guidelines.

##### Important Links

- [Status of CC](https://status.alliancecan.ca/)
- [GPU information](https://docs.alliancecan.ca/wiki/Using_GPUs_with_Slurm)

---

##### Table of Contents

- Tips for Waterloo (Jimmy Lin)
- Login to CC
- Using modules
- Creating and using a virtual environment
- Build everything from scratch using anaconda
- Installing Python modules
- Transfer data
- Interactive jobs
- Job scheduling
- sample shell script (non-queued)

---

## Tips for Waterloo (Jimmy Lin)

- rrg-jimmylin will get a preference over def-jimmylin.
- In Narval, we only have def-jimmylin.
- Cedar generally slower than Narval and less probability of getting resources.

```
[name@server ~]$ export SLURM_ACCOUNT=def-jimmylin
[name@server ~]$ export SBATCH_ACCOUNT=$SLURM_ACCOUNT
[name@server ~]$ export SALLOC_ACCOUNT=$SLURM_ACCOUNT
```

---

## Login to CC

when using Linux or MacOs, you will need to open a terminal and use the command shown below:

```
[name@server ~]$ ssh -Y userid@cluster_name
```

The cluster name will be something like cedar.computecanada.ca or niagara.computecanada.ca.

More information about this can be found [here](https://docs.computecanada.ca/wiki/SSH)

---

## Using modules

### sub-command spider

The spider sub-command searches the complete tree of all modules and displays it.

```
[name@server ~]$ module spider
```

If you specify the name of an application, for example with

```
[name@server ~]$ module spider python
```

this will show you a list of all available versions of the application. The outcome will be something like

```
Versions:
  python/2.7.14
  python/3.5.4
  python/3.6.3
  python/3.7.0
  python/3.7.4
  python/3.8.0
```

If you specify the name of the application along with a version number, for example with

```
[name@server ~]$ module spider python/3.6.3
```

this will display a list of the modules you must load in order to access this version. However, there is no prerequisite to load Python.

### sub-command list

The sub-command list lists the modules that are currently loaded in your environment.

```
[name@server ~]$ module list
```

### sub-command load

The sub-command load lets you load a given module. For example,

```
[name@server ~]$ module load python/3.6.3
```

could let you access the Python application, version 3.6.3. It is recommended to use this version.

Anytime you can find the version of the ongoing Python using the command `python3 --version`. Besides, if there is more than one Python installed on your personal computer, the command `which python3` returns the address where the ongoing Python is stored on your personal computer. It is highly recommended not to use Anaconda distribution of Python, instead install and add the required packages to Python3 using the package manager pip3. pip3 is the Python3 package manager and we later use this tool to install software packages written in Python3. Also the command `pip3 --version` returns the version of the Python3 for which the software packages are going to be installed.

More information about this can be found [here](https://docs.computecanada.ca/wiki/Utiliser_des_modules/en)

### SciPy stack

In addition to the base Python module, the SciPy package is also available as an environment module. The scipy-stack module includes:

- NumPy
- SciPy
- Matplotlib
- IPython
- pandas
- Sympy
- nose

If you want to use any of these Python modules, load a Python version, then use `module load scipy-stack`. However, as an alternative way, you can install these modules with the python package manager as explained later.

More information about this can be found [here](https://docs.computecanada.ca/wiki/Python)

---

## Creating and using a virtual environment using virtualenv

With each version of Python, Compute Canada provides the tool virtualenv. This tool allows users to create virtual environments within which you can easily install Python modules. This tool allows one to install many versions of the same module, for example, or to customize a Python installation according to the needs of a specific project. It also helps to resolve version conflicts while installing new modules by simply removing virtualenv folder and reinstalling the modules. This will take no time and keep the Python tidy and prevents chaos. Therefore, it is highly recommended to first set up a virtual environment in both your personal computer and Compute Canada account before installing Python modules.

To create a virtual environment, ensure you have selected a Python version with `module load python` as shown above. Then enter the following command, where `ENV` is the name of the empty directory containing your environment. It is recommended to place `ENV` in the home directory on the Compute Canada cluster:

```
[name@server ~]$ virtualenv --no-download ~/ENV
```

Once the virtual environment has been created, it must be activated:

```
[name@server ~]$ source ~/ENV/bin/activate
```

You should also upgrade pip in the environment:

```
[name@server ~]$ pip install --no-index --upgrade pip
```

To exit the virtual environment, simply enter the command deactivate:

```
(ENV) [name@server ~] deactivate
```

The procedure for creating a virtual environment on your personal computer is different but simple.

More information about this can be found [here](https://docs.computecanada.ca/wiki/Python#Creating_and_using_a_virtual_environment)

## Creating and using a virtual environment using anaconda

You can create a virtual environment using Anaconda. Conda is provided within Compute Canada.

```
[name@server ~]$ conda create --prefix /home/n3thakur/scratch/envs/sparse-retrieval python=3.7
```

You can now load the environment once after it has been created:

```
[name@server ~]$ conda activate /home/n3thakur/scratch/test-env
```

---

## Build everything from scratch using anaconda

This section will introduce how to set enviroment from scratch without depending on **ComputeCanada(CC)**:

While CC encourages users to load pre-built modules. To achieve full independence from CC, we should follow this golden rule: verify every installation. Use method like `pip show` to comfirm all the denpendencies are from your virtual conda env instead of from **CC**. This ensures a clean setup, preventing unexpected conflicts in your environment.

### Step 1: unload all modules

Unload all the module first, using `module --force purge` and use `module list` to check whether you unload all modules!

### Step 2: install packages

For every package, you can use `pip install --no-cache-dir package_name` and use `pip show package_name` to check every package should be without any dependence on CC.

### Step 3: install your own Cuda

You need to install your own Cuda instead of loading from CC.
Determine the correct Cuda version first `$ navidai-smi`.

After that, you can click [here](https://developer.nvidia.com/cuda-12-4-0-download-archive?target_os=Linux) or other versions of Cuda to download Cuda.

You can use `$ cat /etc/os-release` to know your system details

```
wget https://developer.download.nvidia.com/compute/cuda/12.4.0/local_installers/cuda_12.4.0_550.54.14_linux.run -O ~/cuda_12.4.0.run
```

After that, you should install Cuda. You can't use the `sudo` command in the official website. You can use the command below to install Cuda.

```
sh ~/cuda_12.4.0.run --silent --toolkit --override --installpath=$HOME/cuda-12.4
```

### Step 4: set up environment variables

To ensure your system correctly detects tha installed Cuda version, update the environment variables:

```
$ export CUDA_HOME=$HOME/cuda-12.4
$ export PATH=$CUDA_HOME/bin:$PATH
$ export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH
$ echo 'export CUDA_HOME=$HOME/cuda-12.4' >> ~/.bashrc
$ echo 'export PATH=$CUDA_HOME/bin:$PATH' >> ~/.bashrc
$ echo 'export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
$ source ~/.bashrc
```

### Step 5: verify Cuda Installation

Check if Pytorch detects CUDA:

`python -c "import torch; print(torch.cuda.is_available())"`

Expected Output `True`

Check the detected GPU

`python -c "import torch; print(torch.cuda.get_device_name(0))"
`

---

## Installing Python modules

### Pytorch (GPU and CPU)

**NOTE**: Before you install python packages on compute canada, you must add these two commands:

```
[name@server ~]$ export PYTHONPATH=""
[name@server ~]$ export PIP_CONFIG_FILE=""
```

The preferred option is to install it using the python package manager pip3 as follows:

1. Load a Python module, as shown above
2. Create and start a virtual environment.
3. Install PyTorch in the virtual environment with pip3 install.

```
(ENV) [name@server ~] pip3 install --no-index torch torchvision
```

More information about this can be found [here](https://docs.computecanada.ca/wiki/PyTorch)

### TensorFlow

The preferred option is to install it using the Python wheel as follows:

1. Load a Python module, as shown above
2. Create and start a virtual environment.
3. Install TensorFlow in the virtual environment with pip3 install.

#### CPU-only

```
(ENV) [name@server ~] pip3 install --no-index tensorflow_cpu
```

#### GPU

```
(ENV) [name@server ~]$ pip3 install --no-index tensorflow_gpu
```

More information about this can be found [here](https://docs.computecanada.ca/wiki/TensorFlow)

### tensorboardX

tensorboardX is the indespensible module installed along with deep learning libraries to visualize the results such as training or validation carves.

Install tensorboardX in the virtual environment with pip3 install:

```
(ENV) [name@server ~] pip3 install tensorboardX
```

tensorboardX should be installed both on your personal computer and Compute Canada account. After running the Python script, you need to first download the log files created by the Python script from Compute Canada to your personal computer and then visualize the results using the command `tensorboard --logdir logs`. logs is the folder address where you store the log files.

More information about this can be found [here](https://github.com/lanpa/tensorboardX)

### Matplotlib

Matplotlib is a plotting library for the Python programming language. Matplotlib can either be loaded as part of SciPy stack as shown above or installed with pip3 install in the virtual environment. Because Compute Canada doest not let the Python script get access to the display server you need to save the figures created by matplotlib as picture files, then download them to your personal computer to see the results.

### IPython

IPython (Interactive Python) is a command shell for interactive computing, originally developed for the Python programming language. IPython can either be loaded as part of SciPy stack as shown above or installed with pip3 install in the virtual environment.

---

## Transfer data

It is recommended to Use data transfer nodes, also called data mover nodes, instead of login nodes whenever you are transferring data to and from Compute Canada clusters. If a data transfer node is available, its URL will be given near the top of the main page for each cluster: Béluga, Cedar, Graham, Niagara.

As Compute Canada restricts the volume of data you can upload to the root directory, it is highly recommended to store your datasets and scripts in the `/home/userid/scratch` folder, and use the root directory merely to set up your virtual environment as mentioned above.

The commands `scp` can be used in a command-line environment on Linux or Mac computers to transfer data to and from Compute Canada:

```
scp -r source_address destination_address
```

For example, if your destination folder is located on the graham cluster, the destination address would be either `userid@graham.computecanada.ca:/home/userid/scratch/destination_folder` or `userid@gra-dtn1.computecanada.ca:/home/userid/scratch/destination_folder`. The later is recommended because it uses the transfer node instead of login node.

More information about this can be found [here](https://docs.computecanada.ca/wiki/Transferring_data/en)

More conveniently, you can transfer data to and from Compute Canada with FileZilla software. Install the software with `sudo apt-get install filezilla` and then set up a connection with the desired cluster as follows:

- file menu --> site manager
- my site --> new site: choose a name, e.g., grahamCC
- general tab:
  - protocol: sftp
  - host: garahm.computecanada.ca or gra-dtn1.computecanada.ca (recommended)
  - logon type: normal
  - user and pass
- ok
- connect

---

## Interactive jobs

Though batch submission is the most common and most efficient way to take advantage of the clusters, interactive jobs are also supported. These can be useful for things like:

- Data exploration at the command line
- Interactive "console tools" like R and iPython
- Significant software development, debugging, or compiling

You can request resources from Compute Canada and start an interactive session on compute nodes with salloc:

```
salloc --time=1:0:0 --gres=gpu:1 --cpus-per-task=16 --mem=63500M --ntasks=1 --account=def-wkui
```

In this example, we requested the resources for 1 hour. You should not occupy the resources in interactive mode persistently for a long time (e.g, one day), otherwise you are likely to receive a warning from Compute Canada.

After getting the required resources, load a Python module as shown above, activate already created virtual environment as shown above, then run the Python script with the command `pthon3 script_name.py`. Because we do not have access to IDE, you need to use the Python Debugger module to debug the script. To this end, use the command `import pdb: pdb.set_trace()` within the script wherever you want to pause the program. This command plays the role of breakpoint in IDEs. More information about how to step in and step out with `pdb` can be found [here](https://docs.python.org/3/library/pdb.html). For the training purpose and working with gpu units, the commands `ipython` or `pyhton3` in the virtual environment provides you with an interactive tool to write and run your code interactively.

More information about this can be found [here](https://docs.computecanada.ca/wiki/Running_jobs#Interactive_jobs)

---

## Job scheduling

On Compute Canada clusters, the job scheduler is the Slurm Workload Manager. This section provides guidance on submitting jobs to Compute Canada clusters. The command to submit a job is **sbatch**:

```
sbatch job_name.sh
```

It returns a job ID that you can later use to monitor or cancel the job. It is noteworthy that you do not have to load Python or activate the virtual environment from terminal while submitting a job.

For each Python script/project to be scheduled for running, a Slurm job script is created. The job script allows to start, execute, and monitor the Python script (typically on parallel) on compute nodes. A Slurm job script looks like this:

```
#!/bin/bash
# ---------------------------------------------------------------------
# SLURM script for a multi-step job on a Compute Canada cluster.
# ---------------------------------------------------------------------
#SBATCH --account=def-wkui
#SBATCH --gres=gpu:1        # Request GPU "generic resources"
#SBATCH --cpus-per-task=16  # Cores proportional to GPUs: 6 on Cedar, 16 on Graham.
#SBATCH --mem=63500M        # Memory proportional to GPUs: 31500 Cedar, 63500 Graham.
#SBATCH --time=03:00:00
#SBATCH --job-name=some_name
#SBATCH --output=/scratch/destination_folder/some_name-%j.out
# ---------------------------------------------------------------------
echo "Current working directory: `pwd`"
echo "Starting run at: `date`"
# ---------------------------------------------------------------------
echo ""
echo "Job Array ID / Job ID: $SLURM_ARRAY_JOB_ID / $SLURM_JOB_ID"
echo "This is job $SLURM_ARRAY_TASK_ID out of $SLURM_ARRAY_TASK_COUNT jobs."
echo ""
# ---------------------------------------------------------------------
# Run your simulation step here...

module load python/3.6.3
source ~/ENV/bin/activate
python ~/scratch/destination_folder/python_script_name.py
# ---------------------------------------------------------------------
echo "Job finished with exit code $? at: `date`"
```

It is mainly made up of two parts. Within the first part, we request the desired resources, and the second part consists of commands that we want the compute node to execute. After running the Python script, besides log files created by tensorboardX and picture files created by matplotlib, a text file known as the output file will be created that reports the running process and records the results of I/O commands such as print().

**It is noteworthy that you rarely need more than one gpu as it is less likely to become saturated during the training process. In case you want to further decrease the running time of your script, request enough memory and increase the batch_size for each iteration. Otherwise, you need to make some changes to your script to fully carry out the task parallelism and also you need to wait further in the queue to get the resources.**

More information about this can be found [here](https://docs.computecanada.ca/wiki/Running_jobs#Use_sbatch_to_submit_jobs)

### Time limits

Niagara accepts jobs of up to 24 hours run-time, Béluga up to 7 days, and Cedar and Graham up to 28 days.

On the three general-purpose clusters, longer jobs are restricted to use only a fraction of the cluster. There are partitions for jobs of

- 3 hours or less,
- 12 hours or less,
- 24 hours (1 day) or less,
- 72 hours (3 days) or less,
- 7 days or less, and
- 28 days or less

Because any job of 3 hours is also less than 12 hours, 24 hours, and so on, shorter jobs can always run in partitions with longer time-limits. A shorter job will have more scheduling opportunities than an otherwise-identical longer job.

More information about this can be found [here](https://docs.computecanada.ca/wiki/Job_scheduling_policies)

For the program that is estimated to take too long to run, it is highly recommended to save and resume the state parameters and training index on a regular basis. This lets you resubmit the job several times with shorter running intervals to avoid waiting too long in the queue. To this end, this [repository](https://github.com/vcg-uvic/compute-canada-goodies) provides a great asset.

### Node characteristics

Each cluster of Compute Canada has its own node characteristics such as the amount of memory or cpu cores available per gpu unit. This information can be found on cluster's page on Compute Canada Wiki. Fortunately, they are concisely summarized in this [python script](https://github.com/vcg-uvic/compute-canada-goodies/blob/master/python/queue_cc.py).

### Cancelling jobs

Use scancel with the job ID, that you have already received when submitted the job, to cancel a job:

```
$ scancel <jobid>
```

You can also use it to cancel all your jobs, or all your pending jobs:

```
$ scancel -u $USER
$ scancel -t PENDING -u $USER
```

### Monitoring jobs

By default squeue will show all the jobs the scheduler is managing at the moment. It will run much faster if you ask only about your own jobs with

```
$ squeue -u $USER
```

You can also use the utility sq to do the same thing with less typing.

You can show only running jobs, or only pending jobs:

```
$ squeue -u <username> -t RUNNING
$ squeue -u <username> -t PENDING
```

---
