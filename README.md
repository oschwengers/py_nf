# py_nf library

This is a draft!

Python wrapper around Nextflow to execute job lists in parallel.

For those who writes computational pipelines in Python and needs Nextflow only
to execute parallel steps -- on cluster or CPU.
The idea is the following: user just imports the py_nf library, sets some parameters such as
executor, amount of memory and so on and runs the jobs.
So there are just 3 commands from user's perspective.
The library takes care about creating a Nextflow script and configuration file. 

This library is convenient if you create job lists for cluster in Python
and don't like to learn another scripting language or the Nextflow language is not
powerful enough for you needs.
In this case the only thing you need from Nextflow is pushing your jobs to scheduler,
this is exactly what this library is written for.

## Installation

First of all install Nextflow and make sure you can call it.
Please see [nextflow.io](https://nextflow.io) for details.

Most likely one of these commands will help:

```shell script
curl -fsSL https://get.nextflow.io | bash
# OR
conda install -c bioconda nextflow
```

For now this is the only way to install this library, will be added
to PyPi index later:

```shell script
python3 setup.py bdist_wheel
pip3 install dist/py_nf-0.1.0-py3-none-any.whl
```

## Usage

This is a basic usage scenario:

```python
# import py_nf library:
from py_nf.py_nf import Nextflow

# initiate nextflow handler:
nf = Nextflow(executor="local", project_name="project", cpus=4)

# create joblist:
joblist = ["script.py in/1.txt out/1.txt",
           "script.py in/2.txt out/2.txt"
           "script.py in/3.txt out/3.txt"
           "script.py in/4.txt out/4.txt"
           "script.py in/5.txt out/5.txt"]

# execute jobs:
status = nf.execute(joblist)

if status == 0:
    # enjoy your results
    pass
else:
    # pipeline failed, do something!
    # do_some_cleanup()
    exit(1)
```

Important:

please use absolute pathways in your commands!

You can use paths_to_abspaths_in_joblist function:

```python
from py_nf.py_nf import paths_to_abspaths_in_joblist

joblist = ["script.py in/1.txt out/1.txt -p",
           "script.py in/2.txt out/2.txt -p"
           "script.py in/3.txt out/3.txt -p"
           "script.py in/4.txt out/4.txt -p"
           "script.py in/5.txt out/5.txt -p"]

abs_path_joblist = paths_to_abspaths_in_joblist(joblist)
```

This function will search for file or directory paths in your job list and
then replace them with absolute paths.

For example, this command:

```text
script.py in/1.txt out/1.txt -p
```

Will be replaced with something like this:

```text
/home/user/proj/script.py /home/user/proj/in/1.txt /home/user/proj/out/1.txt -p
```

### Read more about nextflow executors

Nextflow supports a wide range of cluster schedulers, please read about them and
acceptable parameters [here](https://www.nextflow.io/docs/latest/executor.html).

### Nextflow class parameters

You can initiate Nextflow() class with the parameters listed here.
Most of these options reproduce Nextflow process parameters, you can read
about them in the [documentation](https://www.nextflow.io/docs/latest/process.html).

1) *nextflow_executable*, "nextflow" is default.
Define the path to nextflow executable you like to use:
nf = Nextflow(nextflow_executable="/home/user/nf_v20/nextflow")
2) *executor*, "local" is default.
Please read more about Nextflow executor in the corresponding section.
To use "slurm" executor do the following:
nf = Nextflow(executor="slurm")
3) *error_strategy*, "retry" is default.
You can define errorStrategy nextflow parameter.
Please look for available errorStrategy options
[here](https://www.nextflow.io/docs/latest/process.html#errorstrategy).
Usage example:
nf = Nextflow(error_strategy="ignore")
4) *max_retries*, default 3.
Controls Nextflow [maxRetries parameter](https://www.nextflow.io/docs/latest/process.html#maxretries).
nf = Nextflow(max_retries=5).
5) *queue*, default "batch".
Controls Nextflow process queue parameter.
To set "long" queue do:
nf = Nextflow(queue="long").
A list of available queues depends on your scheduler.
6) *memory*, default "10G".
Amount of memory each process is allowed to use.
Please find [here](https://www.nextflow.io/docs/latest/process.html#memory)
how to format the memory amount.
To set "memory" parameter to 100Gb do the following:
nf = Nextflow(memory="100G").
7) *time*, default "1h".
This parameter controls how long a process is allowed to run.
Please read formatting rules [here](https://www.nextflow.io/docs/latest/process.html#time).
To set the time limit to 1 day:
nf = Nextflow(time="1d").
8) *cpus*, default 1.
Controls the number of CPUs required by each job.
A usage example:
nf = Nextflow(cpus=8).
9) *queue_size*, default 100.
Controls nextflow process "queue_size" parameter.
The number of tasks the executor will handle in a parallel manner.
10) *remove_logs*, default False.
If set to True and Nextflow executes jobs successfully, all intermediate and log files
will be removed.
This might be important because Nextflow produces a whole bunch of files which might
be not welcome at some file systems.
To set this parameter do the following:
nf = Nextflow(remove_logs=True).
11) *force_remove_logs*, default False.
The only difference with "remove_logs" parameter is that Nextflow logs and intermediate 
files will be removed in any case.
nf = Nextflow(force_remove_logs=True).
12) *wd*, cwd is default (directory you call the script from).
This is the directory where the library creates the project directory.
Then the library saves nextflow script, configuration and all intermediate files to
the created project directory.
Usage example:
nf = Nextflow(wd="/tmp/project/").
Might be useful if the filesystem where you run your pipeline doesn't support file
locks and you have to run the nextflow pipeline outside the filesystem.
Please see Troubleshooting section case 1 for details.
13) *project_name*, default: "nextflow_project_at_{timestamp}".
Basically a project directory name where the library keeps all nextflow-related data and files.
nf = Nextflow(project_name="test_project").
If not set, the library will automatically generate the project name using timestamp.
14) *no_nf_check*, default False.
Normally the library checks whether there is an accessible nextflow executable and
raises an error if that's not the case.
If you set it to True:
nf = Nextflow(no_nf_check=True)
then library will not check for that on the stage of Nextflow class initiation.
15) *switch_to_local*, default False.
Normally the library checks, whether the executor which the user set is accessible.
For instance, if user set the "executor" parameter to "slurm", but there is not "sbatch"
executable accessible, then program raises an error.
However, if you set switch_to_local parameter:
nf = Nextflow(switch_to_local=True),
the library will just replace "slurm" executor to "local".

### execute function parameters

Input: list/tuple or other iterable of strings.
Each string is a separate shell script command, such as:

```shell script
python3 script.py in_dir/file_1.txt out_dir/file_1.txt --some_option 1 --other_option 2
```

Output: 0 or 1
- If 0: nextflow pipeline executed successfully.
- Otherwise, if 1: nextflow pipeline crashed.
Please have a look at logs.

Please note that if pipeline crashed, py_nf does not raise an error!
User should decide what to do in this case, for instance do some cleanup before or so.

The proper usage would be:

```python
status = nf.execute(job_list)
if status == 1:
    # pipeline failed, need to do some cleanup
    do_some_cleanup()
    sys.exit(1)
```

#### config_file option

If you already have a robust configuration file then you can use it with py_nf library.
To do so:

```python
status = nf.execute(job_list, config_file="/path/to/your/config/file")
if status == 1:
    # pipeline failed, need to do some cleanup
    do_some_cleanup()
    sys.exit(1)
```


## Troubleshooting

Case 1, you see an error message like this:

```txt
Can't open cache DB: /lustre/projects/project-xxx/.nextflow/cache/a80d212d-5a68-42b0-a8a5-d92665bdc492/db

Nextflow needs to be executed in a shared file system that supports file locks.
Alternatively you can run it in a local directory and specify the shared work
directory by using by `-w` command line option.
```

That means your filesystem doesn't file locks: maybe it's lustre and your system
administrator disabled the locks.
The simplest way to override this is to pick some directory outside lustre filesystem and
call nextflow from there.
You can use "wd" parameter to do so:

```python
from py_nf.py_nf import Nextflow

some_dir = "/home/user/nextflow_stuff"
nf = Nextflow(wd=some_dir)
```
