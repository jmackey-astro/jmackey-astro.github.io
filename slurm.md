Jonathan Mackey > Miscellaneous > Slurm Tutorial 


Slurm
-----

[Slurm](https://slurm.schedmd.com/ "https://slurm.schedmd.com/") is a batch job management system for running lots of compute jobs on a computer.
Many HPC systems use it, including those of [ICHEC](https://www.ichec.ie "https://www.ichec.ie"), [LuxProvide](https://www.luxprovide.lu/meluxina/) and [JSC](https://www.fz-juelich.de/en/jsc/systems/supercomputers/juwels).

Here are some useful commands:

```
sbatch script.sh
```

This submits a batch job that is described in the file script.sh.

```
scancel 12345
```

This will cancel the running job with id 12345.
As far as I know it will fail silently if you don't own this job or if the job-id is invalid.

```
squeue --account=account-name
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
             12346     ProdQ case15km  jmackey PD       0:00      4 (Dependency)
             12345     ProdQ case15km  jmackey  R       6:14      4 n[46,49-50,52]
```

This displays a list of all jobs (queued or running) that are associated with a given user account.

```
squeue -t PENDING
```

Prints a list of jobs that are queued, and the reason that they are not running.

```
sinfo
```

Prints a list of all queues including information on the number of idle and active nodes.

### More advanced options

Run a second job only after the first job completes:

  ```
  $ sbatch script1.sh
  12345
  $ sbatch --dependency=afterok:12345 script2.sh
  ```

  This will run script 1 first, and then the job associated with script 2 will only start after script 1 is finished.

Get more detailed info on the status of a job:

  ```
  $ scontrol show job 1002572
  ```


