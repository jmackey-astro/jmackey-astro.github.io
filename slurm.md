Jonathan Mackey > Miscellaneous > Slurm Tutorial 









![JM photo](images/jmackey.jpg)

**Jonathan Mackey > Micsellaneous > Slurm**
-------------------------------------------

> Quick overview of using Slurm for job management on supercomputers.

[![Logo of the Dublin Institute for Advanced Studies (DIAS)](images/DIAS_logo.png)](https://www.dias.ie "https://www.dias.ie")

[Follow @jm\_astro](https://twitter.com/jm_astro "https://twitter.com/jm_astro")

[Home](index.html "index.html")
[Research Group](research.html "research.html")

PION code 

[Introduction to PION](pion.html "pion.html")
[Old documentation](jmac/index.html "jmac/index.html")
[PION homepage](https://www.pion.ie/ "https://www.pion.ie/")

[Publications](publications.html "publications.html")
[Public Engagement](public.html "public.html")


Miscellaneous 

[Miscellaneous > Home](misc.html "misc.html")
[mercurial.html](mercurial.html "mercurial.html")
[SSH tips](misc.html#ssh "misc.html#ssh")
[Shell scripts](misc.html#scripts "misc.html#scripts")
[Imagemagick](misc.html#convert "misc.html#convert")
[Linux/UNIX commands](misc.html#unix "misc.html#unix")
[LaTeX hints](misc.html#latex "misc.html#latex")
[Gnuplot](misc.html#gnuplot "misc.html#gnuplot")
[Mac OS X stuff](misc.html#mac "misc.html#mac")
[VisIt Visualisation tool](misc.html#VisIt "misc.html#VisIt")

[EEStars 2017](https://www.dias.ie/eestars/ "https://www.dias.ie/eestars/")





> Slurm
> -----
>
> [Slurm](https://slurm.schedmd.com/ "https://slurm.schedmd.com/") is a batch job management system for running lots of compute jobs on a computer.
> I am getting to grips with it because the new Irish supercomputer [kay](https://www.ichec.ie/about/infrastructure/kay "https://www.ichec.ie/about/infrastructure/kay"), run by [ICHEC](https://www.ichec.ie "https://www.ichec.ie") uses it.
>
> Here are some key commands:
>
> ```
> sbatch script.sh
> ```
>
> This submits a batch job that is described in the file script.sh.
>
> ```
> scancel 12345
> ```
>
> This will cancel the running job with id 12345.
> As far as I know it will fail silently if you don't own this job or if the job-id is invalid.
>
> ```
> squeue --account=account-name
>              JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
>              12346     ProdQ case15km  jmackey PD       0:00      4 (Dependency)
>              12345     ProdQ case15km  jmackey  R       6:14      4 n[46,49-50,52]
> ```
>
> This displays a list of all jobs (queued or running) that are associated with a given user account.
>
> ```
> squeue -t PENDING
> ```
>
> Prints a list of jobs that are queued, and the reason that they are not running.
>
> ```
> sinfo
> ```
>
> Prints a list of all queues including information on the number of idle and active nodes.
>
> ### More advanced options
>
> * Run a second job only after the first job completes:
>
>   ```
>   $ sbatch script1.sh
>   12345
>   $ sbatch --dependency=afterok:12345 script2.sh
>   ```
>
>   This will run script 1 first, and then the job associated with script 2 will only start after script 1 is finished.
> * Get more detailed info on the status of a job:
>
>   ```
>   $ scontrol show job 1002572
>   ```






Made with [w3.css](https://www.w3schools.com/w3css/default.asp "https://www.w3schools.com/w3css/default.asp")

© Jonathan Mackey 2019-2022.
