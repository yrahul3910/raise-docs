# RAISE Lab Onboarding

Welcome to the RAISE (Research on AI for SE) lab! This document is meant to help you pick stuff up quickly so you don't have to relearn stuff that the PhD students before you did. In general, you'll find a lot of the lab's collective wisdom over here.

## ARC

### Access

The CSC department has a high-performance computing cluster called ARC. See [the website](https://arcb.csc.ncsu.edu/~mueller/cluster/arc/) (under Obtaining an Account) for instructions on how to set up your account. You can generate the required RSA key using

```sh
ssh-keygen -t rsa -b 4096
```

If you're outside the NCSU network, you will not be able to ssh into the ARC cluster directly; you will need to use the Cisco VPN first. Alternatively, you could ssh into the NCSU EOS machines:

```sh
ssh remote.eos.ncsu.edu
```

And ssh into ARC from there. Note that in this case, you will need to generate your SSH keypair on the EOS machine, NOT your local machine.

### Usage

ARC uses the slurm manager to schedule jobs. There are a few commands you should know:

* `salloc` gives you an interactive shell. If you want to get a specific machine like `c30`, use `salloc -w c30`.
* `sinfo` gives you information about the nodes that are free, and what kind of system they are.
* `squeue` shows all the queued jobs. You want to use `squeue | grep <unity>` to check your jobs.
* `scancel` cancels a job that you started.

Do not run code on the login node! Either use `salloc` to get a node and run stuff there, or use `sbatch`, or one of the scripts below.

We have some [convenience scripts](https://github.com/yrahul3910/convenience-scripts) that you can use that wraps the above. To use these, first modify `run-template.sh` so that the last line reflects the actual command you want to run. Then, rename this file to `run.sh`. Once you're done, you can use `mk-jobs.sh` to create multiple copies of the same job to run multiple repeats of your experiments in parallel. Please be courteous and use your best judgement: if most of the compute nodes are used, don't take up all the remaining nodes. The usage is:

```sh
./mk-jobs.sh <repeats> ./run.sh
```

This will give you a list of the slurm job IDs that you started. Keep these in mind. Inevitably, you will sometimes realize your code has an error, or some other reason to stop all the jobs you just started. `rm-jobs` makes this easier for you, but note that it also deletes the log files:

```sh
./rm-jobs <first_job_id> <last_job_id>
```

Finally, the `status.sh` is a little script that helps you see the progress being made in each of your runs. You will need to modify this script so that the `grep` expression matches the format of the logs that your code actually prints out.

## The RAISE package

We have a package called `raise_utils` that provides wrappers around some common functionality. You can use `pip` or `poetry` to install it. The [repository](https://github.com/yrahul3910/raise) has some examples and docs that you can use to get started. Some of the features that are implemented are:

* Scott-Knott and Kruskal-Wallis tests
* Classification and distribution metrics
* Implementations of DODGE, GHOST, TPE, BOHB, and random HPO methods
* Convenience classes to load tabular datasets

## LaTeX Help

### Submitting to arXiv

Preprints are put up on arXiv before peer review. Although arXiv compiles LaTeX (and requires you to submit the LaTeX source), it has a few quirks. The main ones you will run into are:

* arXiv does not like directory structures. You'll have to flatten out your file structure before uploading.
* arXiv does not handle `.bib` files directly. You'll have to compile, and make sure you also add the `.bbl` file generated in your zip file. Overleaf has an option (under Submit), but it can be iffy. You might benefit from just having a local texlive installation and using it.

### Code in LaTeX

If you need to add code to your LaTeX, the `minted` package is your friend. However, arXiv historically does not like when you use minted. [This reply](https://github.com/gpoore/minted/issues/113#issuecomment-888045507) shows you how to get your code with `minted` up on arXiv:

1. Compile with `\usepackage[finalizecache,cachedir=.]{minted}`
2. Go to the output logs (in Overleaf, this is logs and output files -> other logs and files), and download everything with `pyg`.
3. Change `finalizecache` to `frozencache`, and upload all your files along with your `pyg` files.

### Bar and quartile charts

Occasionally, you will want to have a quartile chart or a bar showing a summary of your algorithm's wins/ties/losses against prior work. We have some commands that do this for you, depending on how you want to do things:

**Grayscale bars:**

![Grayscale bar plot](./img/grayscale-bar.png)

```tex
% Preamble
\newcommand{\sbar}[1]{{\color{darkgray}\rule{\dimexpr 1cm * #1 / 100}{5pt}\color{lightgray}\rule{\dimexpr 1cm * (100 - #1) / 100}{5pt}}}

% Usage
\sbar{percentage}{value}
```

For example, if your method wins 7/10 times, you would use

```tex
\sbar{70}{7}
```

**Colored bars:**

![Colored bar plot](./img/color-bar.png) 

```tex
\usepackage[table]{xcolor}

\xdefinecolor{g}{rgb}{0.063, 0.725, 0.506}
\xdefinecolor{y}{rgb}{0.98, 0.765, 0.114}
\xdefinecolor{r}{rgb}{0.937, 0.267, 0.267}
\newcommand{\sbar}[3]{{\color{g}\rule{\dimexpr 2cm * #1 / (#1 + #2 + #3)}{5pt}\color{y}\rule{\dimexpr 2cm * #2 / (#1 + #2 + #3)}{5pt}\color{r}\rule{\dimexpr 2cm * (#3) / (#1 + #2 + #3)}{5pt}}}
```

Usage:

```tex
\sbar{wins}{ties}{losses}
```

**Quartile plots:**

![Quartile plot](./img/iqr.png) 

For quartile charts, here's the command to add in the preamble:

```tex
\newcommand{\quart}[3]{\begin{adjustbox}{max width=.2\linewidth}\begin{picture}(100,5)%1
    {\color{black}\put(#3,2){\circle*{7}}\put(#1,2){\line(1,0){#2}}}\end{picture}\end{adjustbox}}
```

You would use this as follows:

```tex
\quart{25%ile}{IQR}{median}
```

### Norms and absolute values

Writing out `\left\lvert` and `\right\rvert` each time is tedious. Use the commands below instead:

```tex
\usepackage{mathtools}

\DeclarePairedDelimiter\abs{\lvert}{\rvert}%
\DeclarePairedDelimiter\norm{\lVert}{\rVert}%
% Swap the definition of \abs* and \norm*, so that \abs
% and \norm resizes the size of the brackets, and the 
% starred version does not.
\makeatletter
\let\oldabs\abs
\def\abs{\@ifstar{\oldabs}{\oldabs*}}
%
\let\oldnorm\norm
\def\norm{\@ifstar{\oldnorm}{\oldnorm*}}
\makeatother
```

