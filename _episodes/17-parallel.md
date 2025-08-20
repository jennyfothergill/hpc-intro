---
title: "Using resources effectively"
teaching: 30
exercises: 60
questions:
- "What are the different types of parallelism?"
- "How do we execute a task in parallel?"
- "What benefits arise from parallel execution?"
- "What are the limits of gains from execution in parallel?"
objectives:
- "Prepare a job submission script for the parallel executable."
- "Launch jobs with parallel execution."
- "Record and summarize the timing and accuracy of jobs."
- "Describe the relationship between job parallelism and performance."
keypoints:
- "Parallel programming allows applications to take advantage of
  parallel hardware."
- "The queuing system facilitates executing parallel tasks."
- "Some performance improvements from parallel execution do not scale linearly."
---

We now have the full toolset we need to run a job, and we're going to learn how
to scale up our job perfromance using parallelism. This is a very
important aspect of HPC systems, as parallelism is one of the primary tools
we have to improve the performance of computational tasks.

In this lesson, we will learn about three ways to parallelize a problem:
embarassingly parallel, shared memory parallelism, and distributed parallelism.

If you disconnected, log back in to the cluster.

```
{{ site.local.prompt }} ssh {{ site.remote.user }}@{{ site.remote.login }}
```
{: .language-bash}

## Infinite Monkey Theorem

The [infinite monkey theorem][monkeys] states that a monkey hitting keys
independently and at random on a typewriter keyboard for an infinite amount of
time will almost surely type any given text, including the complete works of
William Shakespeare.

{% include figure.html max-width="75%" caption=""
   file="/fig/Chimpanzee_seated_at_typewriter.jpg"
   alt="Chimpanzee seated at typewriter, credit New York Zoological Society" %}

We don't have infinite time or resources, but we can simulate this problem with
A LOT of monkeys.

Create the following script to simulate a "monkey". Let's name this monkey.py:
```
#!/usr/bin/env python3

import random
import string

nwords = 100
minlen = 2
maxlen = 10
dictionaryfile = "wordlist.txt"

# Generate random string of character of certain length
def randomchars(length):
    return "".join([
        random.choice(string.ascii_lowercase) for _ in range(length)
        ])


# Create a list of random character strings with range of lengths randomly
# distributed between minlen and maxlen
wordlist = [
    randomchars(random.choice(range(minlen,maxlen))) for i in range(nwords)
]

# Read the words from our downloaded dictionary
with open(dictionaryfile, "r") as f:
    englishwords = [ line.strip() for line in f ]

# Print all the randomly generated "words" that are also in the dictionary
for word in wordlist:
    if word in englishwords:
        print(word)
```
{: .language-python}

## Let your monkey loose on a Compute Node

Create a submission file, requesting one task on a single node, then launch it.

```
{{ site.remote.prompt }} nano monkey-job.sh
{{ site.remote.prompt }} cat monkey-job.sh
```
{: .language-bash}

{% include {{ site.snippets }}/parallel/one-task-jobscript.snip %}

```
{{ site.remote.prompt }} {{ site.sched.submit.name }} monkey-job.sh
```
{: .language-bash}

As before, use the {{ site.sched.name }} status commands to check whether your
job is running and when it ends:

```
{{ site.remote.prompt }} {{ site.sched.status }} {{ site.sched.flag.user }}
```
{: .language-bash}

Use `ls` to locate the output file. The `-t` flag sorts in
reverse-chronological order: newest first. What was the output?

> ## Read the Job Output
>
> The cluster output should be written to a file in the folder you launched the
> job from. For example,
>
> ```
> {{ site.remote.prompt }} ls -t
> ```
> {: .language-bash}
> ```
> slurm-2114623.out  monkey-job.sh   monkey.py  wordlist.txt
> ```
> {: .output}
> ```
> {{ site.remote.prompt }} cat slurm-2114623.out
> ```
> {: .language-bash}
> ```
> wow
> ld
> ax
> tk
> bl
> kg
> ```
> {: .output}
> Your monkey no doubt came up with a different list of words.
{: .solution}

## Running multiple jobs at once

This is an example of an [embarassingly parallel][embarrassingly-parallel]
problem: Each monkey doesn't need to know anything about what the other monkeys
are doing. So if our goal is for the monkeys to generate the most words, more
monkeys is the solution.

By making the following modification to our submission script (`monkey-job.sh`),
we can use a [job array][jobarray] to put multiple monkeys to work!

{% include {{ site.snippets }}/parallel/array-jobscript.snip %}

After modifying your submission script, resubmit your job:
```
{{ site.remote.prompt }} {{ site.sched.submit.name }} monkey-job.sh
```
{: .language-bash}

You can check on your job while it's running using:
```
{{ site.remote.prompt }} squeue --me
```
{: .language-bash}
Or see the time, hostname, and exitcode of finished jobs using:
```
{{ site.remote.prompt }} sacct -X
```
{: .language-bash}

Once your job is finished, we can use the word count program, `wc`, to see how
many words were generated by our array of monkeys:
```
{{ site.remote.prompt }} wc -l  slurm-2114626_*
  8 slurm-2114626_0.out
  2 slurm-2114626_10.out
  6 slurm-2114626_1.out
  4 slurm-2114626_2.out
  7 slurm-2114626_3.out
 11 slurm-2114626_4.out
  8 slurm-2114626_5.out
  4 slurm-2114626_6.out
  4 slurm-2114626_7.out
  5 slurm-2114626_8.out
 10 slurm-2114626_9.out
 69 total
```
{: .output}

## Other kinds of parallelism

Shared memory parallel

> ## Hands on activity
>
> Distributed vs shared memory parallelism.
> Hands on activity about parallelization.
{: .challenge}


{% include figure.html max-width="75%" caption=""
   file="/fig/MachinesSharedDist.gif"
   alt="Diagram of two nodes showing memory and processors, credit Cornell
   Virtual Workshop" %}


--ntasks (-n) vs. --cpus-per-task (-c)

    More resources doesn't equal better performance.

> ## What is MPI?
>
> The Message Passing Interface is a set of tools which allow multiple tasks
> running simultaneously to communicate with each other.
> Typically, a single executable is run multiple times, possibly on different
> machines, and the MPI tools are used to inform each instance of the
> executable about its sibling processes, and which instance it is.
> MPI also provides tools to allow communication between instances to
> coordinate work, exchange information about elements of the task, or to
> transfer data.
> An MPI instance typically has its own copy of all the local variables.
{: .callout}

While MPI-aware executables can generally be run as stand-alone programs, in
order for them to run in parallel they must use an MPI _run-time environment_,
which is a specific implementation of the MPI _standard_.
To activate the MPI environment, the program should be started via a command
such as `mpiexec` (or `mpirun`, or `srun`, etc. depending on the MPI run-time
you need to use), which will ensure that the appropriate run-time support for
parallelism is included.

> ## MPI Runtime Arguments
>
> On their own, commands such as `mpiexec` can take many arguments specifying
> how many machines will participate in the execution,
> and you might need these if you would like to run an MPI program on your
> own (for example, on your laptop).
> In the context of a queuing system, however, it is frequently the case that
> MPI run-time will obtain the necessary parameters from the queuing system,
> by examining the environment variables set when the job is launched.
{: .callout}

In an HPC environment, we try to reduce the execution time for all types of
jobs, and MPI is an extremely common way to combine dozens, hundreds, or
thousands of CPUs into solving a single problem. To learn more about
parallelization, see the [parallel novice lesson][parallel-novice] lesson and
the [Cornell Virtual Workshop on Parallel Programming Concepts and High
Performance Computing][cornell].

{% include links.md %}

[monkeys]: https://en.wikipedia.org/wiki/Infinite_monkey_theorem
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[cornell]: https://cvw.cac.cornell.edu/parallel/intro/index
[parallel-novice]: http://www.hpc-carpentry.org/hpc-parallel-novice/
[jobarray]: https://slurm.schedmd.com/job_array.html

{% include new-window-fix.html %}
