---
title: "Working with the scheduler"
teaching: 50
exercises: 30
---



::: questions
 - How do I launch a program to run on any one node in the cluster?
 - How does an HPC system decide which jobs run, when, and where? 
 - What is a scheduler and how do I use it? 
:::

::: objectives
 - Run a simple program on the cluster.
 - Submit a simple script to the cluster.
 - Monitor the execution of your job.
 - Inspect the output and error files of your jobs.
:::

## Job Scheduler

An HPC system might have thousands of nodes and thousands of users. How do we
decide who gets what and when? How do we ensure that a task is run with the
resources it needs? This job is handled by a special piece of software called
the scheduler. On an HPC system, the scheduler manages which jobs run where and
when.

The following illustration compares the tasks of a job scheduler to a waiter
in a restaurant. Have you had to wait for a while in a queue to get in to a popular restaurant? 
Scheduling a job can be thought of in the same way! 

![The waiter scheduler](fig/restaurant_queue_manager.svg){caption="" alt="Compare a job scheduler to a waiter in a restaurant"}

The scheduler used in this lesson is Slurm. Although
Slurm is not used everywhere, running jobs is quite similar
regardless of what software is being used. The exact syntax might change, but
the concepts remain the same.

## Running a Batch Job

The most basic use of the scheduler is to run a command non-interactively. Any
command (or series of commands) that you want to run on the cluster is called a
*job*, and the process of using a scheduler to run the job is called *batch job
submission*.

In this case, the job we want to run is just a shell script. Let's create a
demo shell script to run as a test. The landing pad will have a number of
terminal-based text editors installed. Use whichever you prefer. Unsure? `vim` or `nano`
are pretty good, basic choices.

```bash
userid@ln03:~> vim example-job.sh
userid@ln03:~> chmod +x example-job.sh
userid@ln03:~> cat example-job.sh
```

```output
#!/bin/bash

echo -n "This script is running on "
hostname
echo "This script has finished successfully."
```

::: challenge
## Creating Our Test Job

Run the script. Does it execute on the cluster or just our login node?

```bash
userid@ln03:~> ./example-job.sh
```

::: solution

```output
This script is running on ln03 
This script has finished successfully.
```
This job runs on the login node.
:::
:::

If you completed the previous challenge successfully, you probably realise that
there is a distinction between running the job through the scheduler and just
"running it". 

We need to submit the job to the scheduler, so we use the `sbatch` command.

```bash
userid@ln03:~> sbatch --partition=standard --qos=short example-job.sh
```

```output
sbatch: Your job has no time specification (--time=). The maximum time for the short QoS of 20 minutes has been applied.
sbatch: Warning: It appears your working directory may not be on the work filesystem. It is /home1/home/ta215/ta215/ta215broa1. The home filesystem and RDFaaS are not available from the compute nodes - please check that this is what you intended. You can cancel your job with 'scancel <JOBID>' if you wish to resubmit.
Submitted batch job 11102066
```

Ah! What went wrong here? Slurm is telling us that the file system we are currently on, `/home`, is not available
on the compute nodes and that we are getting the default, short runtime. We will deal with the runtime 
later, but we need to move to a different file system to submit the job and have it visible to the 
compute nodes. 

On ARCHER2, this is the `/work` file system. The path is similar to home but with 
`/work` at the start. Lets move there now, copy our job script across and resubmit:

```bash
userid@ln03:~> cd /work/ta215/ta215/userid
userid@ln03:/work/ta215/ta215/userid> cp ~/example-job.sh .
userid@ln03:/work/ta215/ta215/userid> sbatch --partition=standard --qos=short example-job.sh
```

```output
sbatch: Your job has no time specification (--time=). The maximum time for the short QoS of 20 minutes has been applied
Submitted batch job 36855
```

That's better! And that's all we need to do to submit a job. Our work is done --- now the
scheduler takes over and tries to schedule the job to run on the compute nodes. While the job is waiting
to run, it goes into a list of jobs called the *queue*. To check on our job's
status, we can check the queue using the command
`squeue -u userid`.

```bash
userid@ln03:/work/ta215/ta215/userid> squeue -u userid
```
Or, we can use: 
```bash
userid@ln03:/work/ta215/ta215/userid> squeue --me 
```

You have to be quick! If you are, you should see an output that looks like this: 


```output
        JOBID PARTITION     NAME             USER       ST       TIME  NODES NODELIST(REASON)
    11102103  standard      example-job.sh   ta215bro   R       0:01      1 nid001810
```

We can see all the details of our job, most importantly that it is in the `R`
or `RUNNING` state. Sometimes our jobs might need to wait in a queue and show the `PD` or `PENDING` state.

::: discussion
## Where's the Output?
On the login node, this script printed output to the terminal but now there's nothing. Where'd it go?
HPC job output is typically redirected to a file in the directory you
launched it from. Use `ls` to find and read the file.
:::


The best way to check our job's status is with `squeue`. Of
course, running `squeue` repeatedly to check on things can be
a little tiresome. To see a real-time view of our jobs, we can use the `watch`
command. `watch` reruns a given command at 2-second intervals. This is too
frequent for a large machine like ARCHER2 and **will upset your system administrator.** 
You can change the interval to a more reasonable value with the `-n seconds`
parameter. ARCHER2 system administration recommmend this to be set for 60 seconds or longer. 

```bash
userid@ln03:/work/ta215/ta215/userid> sbatch --partition=standard --qos=short example-job.sh
userid@ln03:/work/ta215/ta215/userid> watch -n 60 squeue -u userid
```

You should see an auto-updating display of your job's status. When it finishes,
it will disappear from the queue. Press `Ctrl-c` when you want to stop the
`watch` command.


## Customising a Job

The job we just ran used some of the scheduler's default options. In a
real-world scenario, that's probably not what we want. The default options
represent a reasonable minimum. Chances are, we will need more cores, more
memory, more time, among other special considerations. To get access to these
resources we must customize our job script.

Comments in UNIX shell scripts (denoted by `#`) are typically ignored, but
there are exceptions. For instance the special `#!` comment at the beginning of
scripts specifies what program should be used to run it (you'll typically see
`#!/bin/bash`). Schedulers like Slurm also
have a special comment used to denote special scheduler-specific options.
Though these comments differ from scheduler to scheduler,
Slurm's special comment is `#SBATCH`. Anything
following the `#SBATCH` comment is interpreted as an
instruction to the scheduler.

Let's illustrate this by example. By default, a job's name is the name of the
script, but the `--job-name` option can be used to change the
name of a job. Add an option to the `example-job.sh` script:

```output
#!/bin/bash
#SBATCH --job-name=new_name

echo -n "This script is running on "
hostname
echo "This script has finished successfully."
```

Submit the job and monitor its status:

```bash
userid@ln03:/work/ta215/ta215/userid> sbatch --partition=standard --qos=short example-job.sh
userid@ln03:/work/ta215/ta215/userid> squeue -u userid
```


```output
        JOBID PARTITION     NAME       USER       ST       TIME  NODES NODELIST(REASON)
    11102119  standard      new_name   ta215bro   CG       0:02      1 nid004855
```

Fantastic, we've successfully changed the name of our job!

We can see also see a new job state. The state is reporting as `CG` which stands for `COMPLETING`. 

### Resource Requests

But what about more important changes, such as the number of cores and memory
for our jobs? One thing that is absolutely critical when working on an HPC
system is specifying the resources required to run a job. This allows the
scheduler to find the right time and place to schedule our job. If you do not
specify requirements (such as the amount of time you need), you will likely be
stuck with your site's default resources or face an error when submitting, 
which is probably not what you want.

The following are several key resource requests:


* `--nodes=<nodes>` - Number of nodes to use
* `--ntasks-per-node=<tasks-per-node>` - Number of parallel processes per node
* `--cpus-per-task=<cpus-per-task>` - Number of cores to assign to each parallel process
* `--time=<days-hours:minutes:seconds>` - Maximum real-world time (walltime)
your job will be allowed to run. The `<days>` part can be omitted.

Note that just *requesting* these resources does not make your job run faster,
nor does it necessarily mean that you will consume all of these resources. It
only means that these are made available to you. Your job may end up using less time 
or fewer tasks or nodes, than you have requested, and it will still run.

It's best if your requests accurately reflect your job's requirements. We'll
talk more about how to make sure that you're using resources effectively in a
later episode of this lesson.

::: callout
## Command line options or job script options?
All of the options we specify can be supplied on the command line (as we
do here for `--partition=standard`) or in the job script (as we have done
for the job name above). These are interchangeable. It is often more convenient
to put the options in the job script as it avoids lots of typing at the command
line.
:::

::: challenge
## Submitting Resource Requests
Modify our `hostname` script so that it runs for a minute, then submit a job
for it on the cluster. You should also move all the options we have been specifying
on the command line (e.g. `--partition`) into the script at this point.

::: solution

```bash
userid@ln03:/work/ta215/ta215/userid> cat example-job.sh
```

```output
#!/bin/bash
#SBATCH --time 00:01:15
#SBATCH --partition=standard
#SBATCH --qos=short
echo -n "This script is running on "
sleep 60 # time in seconds
hostname
echo "This script has finished successfully."
```

```bash
userid@ln03:~> sbatch example-job.sh
```

Why do we not make the Slurm time and `sleep` time identical?
:::
:::

Resource requests are typically binding. If you exceed them, your job will be
killed. Let's use the walltime as an example. We will request 30 seconds of
walltime, and attempt to run a job for two minutes.

```bash
userid@ln03:/work/ta215/ta215/userid> cat example-job.sh
```

```output
#!/bin/bash
#SBATCH --job-name long_job
#SBATCH --time 00:00:30
#SBATCH --partition=standard
#SBATCH --qos=short

echo "This script is running on ... "
sleep 120 # time in seconds
hostname
echo "This script has finished successfully."
```

Submit the job and wait for it to finish. Once it is has finished, check the
log file.

```bash
userid@ln03:/work/ta215/ta215/userid> sbatch example-job.sh
userid@ln03:/work/ta215/ta215/userid> squeue -u userid
```


```bash
userid@ln03:/work/ta215/ta215/userid> cat slurm-11102156.out
```

```output
This script is running on slurmstepd: error: *** JOB 11102156 ON nid001142 CANCELLED AT 2025-10-08T10:39:46 DUE TO TIME LIMIT ***
```

Our job was killed for exceeding the amount of resources it requested. Although
this appears harsh, this is actually a feature. Strict adherence to resource
requests allows the scheduler to find the best possible place for your jobs.
Even more importantly, it ensures that another user cannot use more resources
than they've been given. If another user messes up and accidentally attempts to
use all of the cores or memory on a node, Slurm will either
restrain their job to the requested resources or kill the job outright. Other
jobs on the node will be unaffected. This means that one user cannot mess up
the experience of others, the only jobs affected by a mistake in scheduling
will be their own.

## What else affects a job's resources? 

As well as resources that you specifically request in your job script, ARCHER2's 
resource limits are also definied by two arguments that must be passed to `sbatch`: 
`--partition` and `--qos`. 

The **partition** specifies the nodes that are eligible to run your job, and
the **Quality of Service (QoS)** specifies the job limits that apply.

The reason for the partition is so that users can ask for more memory, run in 
serial or run with GPUs. Helps to accommodate different types of workloads, all running 
on the same machine. 

The reason for the QoS is so that user's are restricted to certain time limits or certain numbers of jobs, 
to help ensure fair and shared use of the machine. This is a difficult balance to make as we want to ensure that 
reearchers can use the full computational power of the machine, but also want to ensure that many 
researchers can do science all at the same time. 

### Partitions 

| Partition | Description     | Max nodes available |
| --------- | --------------- | ------------------- |
| standard  | CPU nodes 256/512 GB memory | 5860    |
| highmem   | CPU nodes 512 GB memory | 584     |
| serial    | CPU nodes 512 GB memory | 2       |
| gpu  | GPU nodes 512 GB memory | 4    |

### QoS 

| QoS        | Max Nodes Per Job | Max Walltime | Jobs Queued | Jobs Running | Partition(s) | Notes |
| ---------- | ----------------- | ------------ | ----------- | ------------ | ------------ | ------|
| standard   | 1024               | 24 hrs       | 64          | 16           | standard     | Maximum of 1024 nodes in use by any one user at any time |
| highmem   | 256               | 24 hrs       | 16          | 16           | highmem     | Maximum of 256 nodes in use by any one user at any time |
| taskfarm   | 16               | 24 hrs       | 128          | 32           | standard     | Maximum of 256 nodes in use by any one user at any time |
| short      | 32                 | 20 mins      | 16           | 4            | standard     | |
| long       | 64                | 96 hrs       | 16          | 16           | standard     | Minimum walltime of 24 hrs, maximum 512 nodes in use by any one user at any time, maximum of 2048 nodes in use by QoS |
| largescale | 5860               | 12 hrs        | 8           | 1            | standard     | Minimum job size of 1025 nodes |
| lowpriority | 2048               | 24 hrs        | 16           | 16            | standard     | Jobs not charged but requires at least 1 CU in budget to use. |
| serial | 32 cores and/or 128 GB memory   | 24 hrs        | 32           | 4            | serial    | Jobs not charged but requires at least 1 CU in budget to use. Maximum of 32 cores and/or 128 GB in use by any one user at any time. |
| reservation | Size of reservation  | Length of reservation       | No limit           | no limit           | standard   |  |
| capabilityday | At least 4096 nodes  | 3 hrs        | 8           | 2            | standard     | Minimum job size of 512 nodes. Jobs only run during [Capability Days](#capability-days) |
| gpu-shd    | 1               | 12 hrs      | 2          | 1           | gpu    | GPU nodes potentially shared with other users |
| gpu-exc    | 2               | 12 hrs      | 2          | 1           | gpu    | GPU node exclusive node access |




::: callout
## But how much does it cost?
Although your job will be killed if it exceeds the selected runtime,
a job that completes within the time limit is only charged for the
time it actually used. However, you should always try and specify a
wallclock limit that is close to (but greater than!) the expected
runtime as this will enable your job to be scheduled more
quickly.

If you say your job will run for an hour, the scheduler has
to wait until a full hour becomes free on the machine. If it only ever
runs for 5 minutes, you could have set a limit of 10 minutes and it
might have been run earlier in the gaps between other users' jobs.
:::

## Cancelling a Job

Sometimes we'll make a mistake and need to cancel a job. This can be done with
the `scancel` command. Let's submit a job and then cancel it using
its job number (remember to change the walltime so that it runs long enough for
you to cancel it before it is killed!).

```bash
userid@ln03:/work/ta215/ta215/userid> sbatch example-job.sh
userid@ln03:/work/ta215/ta215/userid> squeue -u userid
```


```output
Submitted batch job 11102156

             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          11102156  standard example- ta215bro  R       0:18      1 nid001142
```

Now cancel the job with its job number (printed in your terminal). Absence of any
job info indicates that the job has been successfully cancelled.

```bash
userid@ln03:/work/ta215/ta215/userid> scancel 11102156
userid@ln03:/work/ta215/ta215/userid> squeue -u userid
```


```output
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
```


::: challenge
## Cancelling multiple jobs
We can also cancel all of our jobs at once using the `-u` option. This will delete all jobs for a
specific user (in this case us). Note that you can only delete your own jobs.
Try submitting multiple jobs and then cancelling them all with `scancel -u yourUsername`.
:::

## Other Types of Jobs

Up to this point, we've focused on running jobs in batch mode.
Slurm also provides the ability to start an interactive session.

There are very frequently tasks that need to be done interactively. For example, when we want to debug something 
that went wrong with a previous job. The amount of resources needed is too much for a login node, but writing 
an entire job script is overkill. 

In this instance, we can run these types of tasks as a one-off with `srun`.

`srun` runs a single command in the queue system and then exits.
Let's demonstrate this by running the `hostname` command with `srun`. (We can cancel an 
`srun` job with `Ctrl-c`.)

```bash
 srun --partition=standard --qos=short --time=00:01:00 hostname
```

```output
srun: job 11102176 queued and waiting for resources
srun: job 11102176 has been allocated resources
nid001810
```

`srun` accepts all of the same options as `sbatch`. However, instead of specifying these in a
script, these options are specified on the command-line when starting a job.

Typically, the resulting shell environment will be the same as that for
`sbatch`.


## Running parallel jobs using MPI

The power of HPC systems comes from *parallelism*. That is, connecting many processors, disks, and 
other components to work together, rather than relying on having more powerful components than your
laptop or workstation. 

Often, when running research programs on HPC you will need to run a program that has been built 
to use the MPI (Message Passing Interface) parallel library for parallel computing. MPI enables programs to 
take advantage of multiple processing cores in parallel, allowing researchers to run large simulations or 
models more quickly. 

The details of how MPI works, or even using MPI-based programs, is not important for this course. 
However, it's important to know that MPI programs are launched differently from serial programs, 
and you'll need to submit them correctly in your job submission scrips. Specifically, launching
parallel MPI programs typically requires four things:

  - A special parallel launch program such as `mpirun`, `mpiexec`, `srun` or `aprun`.
  - A specification of how many processes to use in parallel. For example, our parallel program
    may use 256 processes in parallel.
  - A specification of how many parallel processes to use per compute node. For example, if our
    compute nodes each have 32 cores we can specify 32 parallel processes per node.
  - The command and arguments for our parallel program.


::: prereq

## Required Files

The program used in this example can be retrieved using wget or a browser and copied to the remote.

**Using wget**: 
```bash
userid@ln03:~> wget https://epcced.github.io/2025-10-14-archer2-intro-hpc/files/pi-mpi.py
```

**Download via web browser**:

[https://epcced.github.io/2025-10-14-archer2-intro-hpc/files/pi-mpi.py](https://epcced.github.io/2025-10-14-archer2-intro-hpc/files/pi-mpi.py)

:::

To illustrate this process, we will use a simple MPI parallel program that estimates the value of Pi - 
_We will meet this example program in more detail in a later episode of this course._ 

Here is a job submission script that runs the program on 1 compute node, using 16 parallel tasks (or cores) 
on the cluster. Create a job submission script (e.g. called: `run-pi-mpi.slurm`) with the following contents: 


```bash
#!/bin/bash

#SBATCH --partition=standard
#SBATCH --qos=short
#SBATCH --time=00:05:00

#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16

module load cray-python

srun python pi-mpi.py 10000000
```

The parallel launch line for our program can be seen towards the bottom of the script:


```bash
srun python pi-mpi.py 10000000
```

And this corresponds to the four required items we described above:

  1. Parallel launch program: in this case the parallel launch program is
     called `srun`; the additional argument controls which cores are used.
  2. Number of parallel processes per node: in this case this is 16,
     and is specified by the option `--ntasks-per-node=16` option.
  2. Total number of parallel processes: in this case this is also 16, because
     we specified 1 node and 16 parallel processes per node.
  4. Our program and arguments: in this case this is `python pi-mpi.py 10000000`.

We can now launch using the `sbatch` command.

```bash
userid@ln03:/work/ta215/ta215/userid> sbatch run-pi-mpi.slurm
```


::: challenge
## Running parallel jobs
Modify the pi-mpi-run script that you used above to use all 128 cores on
one node.  Check the output to confirm that it used the correct number
of cores in parallel for the calculation.

::: solution
Here is a modified script

```bash
#!/bin/bash

#SBATCH --partition=standard
#SBATCH --qos=short
#SBATCH --time=00:00:30

#SBATCH --nodes=1
#SBATCH --ntasks-per-node=128

module load cray-python
srun python pi-mpi.py 10000000
```
:::
:::


::: challenge
## Configuring parallel jobs
You will see in the job output that information is displayed about
where each MPI process is running, in particular which node it is
on.

Modify the pi-mpi-run script that you run a total of 2 nodes and 16 processes;
but to use only 8 tasks on each of two nodes.
Check the output file to ensure that you understand the job
distribution.

::: solution
```bash
#!/bin/bash

#SBATCH --partition=standard
#SBATCH --qos=short
#SBATCH --time=00:00:30

#SBATCH --nodes=2
#SBATCH --ntasks-per-node=8

module load cray-python
srun python pi-mpi.py 10000000
```
:::
:::

::: keypoints
 - Schedulers manage fairness and efficiency on HPC systems, deciding which user jobs run and when.
 - A job is any command or script submitted for execution.
 - The scheduler handles how compute resources are shared between users.
 - Jobs should not run on login nodes — they must be submitted to the scheduler.
 - MPI jobs require special launch commands (srun, mpirun, etc.) and explicit process 
   counts to utilize multiple cores or nodes effectively.
:::
