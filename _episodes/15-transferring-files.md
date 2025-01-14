---
title: "Transferring files with remote computers"
teaching: 15
exercises: 15
questions:
- "How do I transfer files to (and from) the cluster?"
objectives:
- "Transfer files to and from a computing cluster."
keypoints:
- "`wget` and `curl -O` download a file from the internet."
- "`scp` and `rsync` transfer files to and from your computer."
- "You can use an SFTP client like FileZilla to transfer files through a GUI."
---

Performing work on a remote computer is not very useful if we cannot get files
to or from the cluster. There are several options for transferring data between
computing resources using CLI and GUI utilities, a few of which we will cover.

## Download Files From the Internet

One of the most straightforward ways to download files is to use either `curl`
or `wget`. Any file that can be downloaded in your web browser through a direct 
link can be downloaded using `curl -O` or `wget`. This is a quick way to 
download datasets or source code.

The syntax for these commands is: `curl -O https://some/link/to/a/file`
and `wget https://some/link/to/a/file`. Try it out by downloading
some material we'll use later on, from a terminal on your local machine.

```
{{ site.local.prompt }} curl -O {{ site.url }}{{ site.baseurl }}/files/hpc-intro-data.tar.gz
```
{: .language-bash}
or
```
{{ site.local.prompt }} wget {{ site.url }}{{ site.baseurl }}/files/hpc-intro-data.tar.gz
```
{: .language-bash}


> ## `tar.gz`?
>
> This is an archive file format, just like `.zip`, commonly used and supported
> by default on Linux, which is the operating system the majority of HPC
> cluster machines run. You may also see the extension `.tgz`, which is exactly
> the same. We'll talk more about "tarballs," since "tar-dot-g-z" is a
> mouthful, later on.
{: .discussion}

## Transferring Single Files and Folders With `scp`

To copy a single file to or from the cluster, we can use `scp` ("secure copy").
The syntax can be a little complex for new users, but we'll break it down.
The `scp` command is a relative of the `ssh` command we used to
access the system, and can use the same public-key authentication
mechanism.

To _upload to_ another computer:

```
{{ site.local.prompt }} scp path/to/local/file.txt {{ site.remote.user }}@{{ site.remote.login }}:/path/on/{{ site.remote.name }}
```
{: .language-bash}

To _download from_ another computer:

```
{{ site.local.prompt }} scp {{ site.remote.user }}@{{ site.remote.login }}:/path/on/{{ site.remote.name }}/file.txt path/to/local/
```
{: .language-bash}

Note that everything after the `:` is relative to our home directory on the
remote computer. We can leave it at that if we don't care where the file goes.

```
{{ site.local.prompt }} scp local-file.txt {{ site.remote.user }}@{{ site.remote.login }}:
```
{: .language-bash}

> ## Upload a File
>
> Copy the file you just downloaded from the Internet to your home directory on
> {{ site.remote.name }}.
>
> > ## Solution
> >
> > ```
> > {{ site.local.prompt }} scp hpc-intro-data.tar.gz {{ site.remote.user }}@{{ site.remote.login }}:~/
> > ```
> > {: .language-bash}
> {: .solution}
{: .challenge}

To copy a whole directory, we add the `-r` flag, for "**r**ecursive": copy the
item specified, and every item below it, and every item below those... until it
reaches the bottom of the directory tree rooted at the folder name you
provided.

```
{{ site.local.prompt }} scp -r some-local-folder {{ site.remote.user }}@{{ site.remote.login }}:target-directory/
```
{: .language-bash}

> ## Caution
>
> For a large directory -- either in size or number of files --
> copying with `-r` can take a long time to complete.
{: .callout}

## What's in a `/`?

When using `scp`, you may have noticed that a `:` _always_ follows the remote
computer name; sometimes a `/` follows that, and sometimes not, and sometimes
there's a final `/`. On Linux computers, `/` is the ___root___ directory, the
location where the entire filesystem (and others attached to it) is anchored. A
path starting with a `/` is called _absolute_, since there can be nothing above
the root `/`. A path that does not start with `/` is called _relative_, since
it is not anchored to the root.

If you want to upload a file to a location inside your home directory --
which is often the case -- then you don't need a leading `/`. After the
`:`, start writing the sequence of folders that lead to the final storage
location for the file or, as mentioned above, provide nothing if your home
directory _is_ the destination.

A trailing slash on the target directory is optional, and has no effect for
`scp -r`, but is important in other commands, like `rsync`.

> ## A Note on `rsync`
>
> As you gain experience with transferring files, you may find the `scp`
> command limiting. The [rsync][rsync] utility provides
> advanced features for file transfer and is typically faster compared to both
> `scp` and `sftp` (see below). It is especially useful for transferring large
> and/or many files and creating synced backup folders.
>
> The syntax is similar to `scp`. To transfer _to_ another computer with
> commonly used options:
>
> ```
> {{ site.local.prompt }} rsync -avzP path/to/local/file.txt {{ site.remote.user }}@{{ site.remote.login }}:directory/path/on/{{ site.remote.name }}/
> ```
> {: .language-bash}
>
> The options are:
> * `a` (archive) to preserve file timestamps and permissions among other things
> * `v` (verbose) to get verbose output to help monitor the transfer
> * `z` (compression) to compress the file during transit to reduce size and
>   transfer time
> * `P` (partial/progress) to preserve partially transferred files in case
>   of an interruption and also displays the progress of the transfer.
>
> To recursively copy a directory, we can use the same options:
>
> ```
> {{ site.local.prompt }} rsync -avzP path/to/local/dir {{ site.remote.user }}@{{ site.remote.login }}:directory/path/on/{{ site.remote.name }}/
> ```
> {: .language-bash}
>
> As written, this will place the local directory and its contents under the
> specified directory on the remote system. If the trailing slash is omitted on
> the destination, a new directory corresponding to the transferred directory
> ('dir' in the example) will not be created, and the contents of the source
> directory will be copied directly into the destination directory.
>
> The `a` (archive) option implies recursion.
>
> To download a file, we simply change the source and destination:
>
> ```
> {{ site.local.prompt }} rsync -avzP {{ site.remote.user }}@{{ site.remote.login }}:path/on/{{ site.remote.name }}/file.txt path/to/local/
> ```
> {: .language-bash}
{: .callout}

## Transferring Files Interactively with FileZilla

FileZilla is a cross-platform client for downloading and uploading files to and
from a remote computer. It is absolutely fool-proof and always works quite
well. It uses the `sftp` protocol. You can read more about using the `sftp`
protocol in the command line in the
[lesson discussion]({{ site.baseurl }}{% link _extras/discuss.md %}).

Download and install the FileZilla client from <https://filezilla-project.org>.
After installing and opening the program, you should end up with a window with
a file browser of your local system on the left hand side of the screen. When
you connect to the cluster, your cluster files will appear on the right hand
side.

To connect to the cluster, we'll just need to enter our credentials at the top
of the screen:

* Host: `sftp://{{ site.remote.login }}`
* User: Your cluster username
* Password: Your cluster password
* Port: (leave blank to use the default port)

Hit "Quickconnect" to connect. You should see your remote files appear on the
right hand side of the screen. You can drag-and-drop files between the left
(local) and right (remote) sides of the screen to transfer files.

Finally, if you need to move large files (typically larger than a gigabyte)
from one remote computer to another remote computer, SSH in to the computer
hosting the files and use `scp` or `rsync` to transfer over to the other. This
will be more efficient than using FileZilla (or related applications) that
would copy from the source to your local machine, then to the destination
machine.

## Archiving Files

One of the biggest challenges we often face when transferring data between
remote HPC systems is that of large numbers of files. There is an overhead to
transferring each individual file and when we are transferring large numbers of
files these overheads combine to slow down our transfers to a large degree.

The solution to this problem is to _archive_ multiple files into smaller
numbers of larger files before we transfer the data to improve our transfer
efficiency. Sometimes we will combine archiving with _compression_ to reduce
the amount of data we have to transfer and so speed up the transfer.

The most common archiving command you will use on a (Linux) HPC cluster is
`tar`. `tar` can be used to combine files into a single archive file and,
optionally, compress it.

Let's start with the file we downloaded from the lesson site,
`hpc-lesson-data.tar.gz`. The "gz" part stands for _gzip_, which is a
compression library. Reading this file name, it appears somebody took a folder
named "hpc-lesson-data," wrapped up all its contents in a single file with
`tar`, then compressed that archive with `gzip` to save space. Let's check
using `tar` with the `-t` flag, which prints the "**t**able of contents"
without unpacking the file, specified by `-f <filename>`, on the remote
computer. Note that you can concatenate the two flags, instead of writing
`-t -f` separately.

```
{{ site.local.prompt }} ssh {{ site.remote.user }}@{{ site.remote.login }}
{{ site.remote.prompt }} tar -tf hpc-lesson-data.tar.gz
hpc-intro-data/
hpc-intro-data/north-pacific-gyre/
hpc-intro-data/north-pacific-gyre/NENE01971Z.txt
hpc-intro-data/north-pacific-gyre/goostats
hpc-intro-data/north-pacific-gyre/goodiff
hpc-intro-data/north-pacific-gyre/NENE02040B.txt
hpc-intro-data/north-pacific-gyre/NENE01978B.txt
hpc-intro-data/north-pacific-gyre/NENE02043B.txt
hpc-intro-data/north-pacific-gyre/NENE02018B.txt
hpc-intro-data/north-pacific-gyre/NENE01843A.txt
hpc-intro-data/north-pacific-gyre/NENE01978A.txt
hpc-intro-data/north-pacific-gyre/NENE01751B.txt
hpc-intro-data/north-pacific-gyre/NENE01736A.txt
hpc-intro-data/north-pacific-gyre/NENE01812A.txt
hpc-intro-data/north-pacific-gyre/NENE02043A.txt
hpc-intro-data/north-pacific-gyre/NENE01729B.txt
hpc-intro-data/north-pacific-gyre/NENE02040A.txt
hpc-intro-data/north-pacific-gyre/NENE01843B.txt
hpc-intro-data/north-pacific-gyre/NENE01751A.txt
hpc-intro-data/north-pacific-gyre/NENE01729A.txt
hpc-intro-data/north-pacific-gyre/NENE02040Z.txt
```
{: .language-bash}

This shows a folder containing another folder, which contains a bunch of files.
If you've taken The Carpentries' Shell lesson recently, these might look
familiar. Let's see about that compression, using `du` for "**d**isk
**u**sage".

```
{{ site.remote.prompt }} du -sh hpc-lesson-data.tar.gz
36K     hpc-intro-data.tar.gz
```
{: .language-bash}

> ## Files Occupy at Least One "Block"
>
> If the filesystem block size is larger than 36 KB, you'll see a larger
> number: files cannot be smaller than one block.
{: .callout}

Now let's unpack the archive. We'll run `tar` with a few common flags:

* `-x` to e**x**tract the archive
* `-v` for **v**erbose output
* `-z` for g**z**ip compression
* `-f` for the file to be unpacked

When it's done, check the directory size with `du` and compare.

> ## Extract the Archive
>
> Using the four flags above, unpack the lesson data using `tar`.
> Then, check the size of the whole unpacked directory using `du`.
>
> Hint: `tar` lets you concatenate flags.
>
> > ## Commands
> >
> > ```
> > {{ site.remote.prompt }} tar -xvzf hpc-lesson-data.tar.gz
> > ```
> > {: .language-bash}
> >
> > ```
> > hpc-intro-data/
> > hpc-intro-data/north-pacific-gyre/
> > hpc-intro-data/north-pacific-gyre/NENE01971Z.txt
> > hpc-intro-data/north-pacific-gyre/goostats
> > hpc-intro-data/north-pacific-gyre/goodiff
> > hpc-intro-data/north-pacific-gyre/NENE02040B.txt
> > hpc-intro-data/north-pacific-gyre/NENE01978B.txt
> > hpc-intro-data/north-pacific-gyre/NENE02043B.txt
> > hpc-intro-data/north-pacific-gyre/NENE02018B.txt
> > hpc-intro-data/north-pacific-gyre/NENE01843A.txt
> > hpc-intro-data/north-pacific-gyre/NENE01978A.txt
> > hpc-intro-data/north-pacific-gyre/NENE01751B.txt
> > hpc-intro-data/north-pacific-gyre/NENE01736A.txt
> > hpc-intro-data/north-pacific-gyre/NENE01812A.txt
> > hpc-intro-data/north-pacific-gyre/NENE02043A.txt
> > hpc-intro-data/north-pacific-gyre/NENE01729B.txt
> > hpc-intro-data/north-pacific-gyre/NENE02040A.txt
> > hpc-intro-data/north-pacific-gyre/NENE01843B.txt
> > hpc-intro-data/north-pacific-gyre/NENE01751A.txt
> > hpc-intro-data/north-pacific-gyre/NENE01729A.txt
> > hpc-intro-data/north-pacific-gyre/NENE02040Z.txt
> > ```
> > {: .output}
> >
> > Note that we did not type out `-x -v -z -f`, thanks to the flag
> > concatenation, though the command works identically either way.
> >
> > ```
> > {{ site.remote.prompt }} du -sh hpc-lesson-data
> > 144K    hpc-intro-data
> > ```
> > {: .language-bash}
> {: .solution}
>
> > ## Was the Data Compressed?
> >
> > Text files compress nicely: the "tarball" is one-quarter the total size of
> > the raw data!
> {: .discussion}
{: .challenge}

If you want to reverse the process -- compressing raw data instead of
extracting it -- set a `c` flag instead of `x`, set the archive filename,
then provide a directory to compress:

```
{{ site.local.prompt }} tar -cvzf compressed_data.tar.gz hpc-intro-data
```
{: .language-bash}

> ## Working with Windows
>
> When you transfer text files to from a Windows system to a Unix system (Mac,
> Linux, BSD, Solaris, etc.) this can cause problems. Windows encodes its files
> slightly different than Unix, and adds an extra character to every line.
>
> On a Unix system, every line in a file ends with a `\n` (newline). On
> Windows, every line in a file ends with a `\r\n` (carriage return + newline).
> This causes problems sometimes.
>
> Though most modern programming languages and software handles this correctly,
> in some rare instances, you may run into an issue. The solution is to convert
> a file from Windows to Unix encoding with the `dos2unix` command.
>
> You can identify if a file has Windows line endings with `cat -A filename`. A
> file with Windows line endings will have `^M$` at the end of every line. A
> file with Unix line endings will have `$` at the end of a line.
>
> To convert the file, just run `dos2unix filename`. (Conversely, to convert
> back to Windows format, you can run `unix2dos filename`.)
{: .callout}

{% include links.md %}

[rsync]: https://rsync.samba.org/
