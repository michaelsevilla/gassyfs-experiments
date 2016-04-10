---
title: "Popper: Making Systems Performance Evaluation Practical"
author:
- name: "Ivo Jimenez, Michael Sevilla, Noah Watkins, Carlos Maltzahn"
  affiliation: "_UC Santa Cruz_"
  email: "`{ivo,msevilla,jayhawk,carlosm}@cs.ucsc.edu`"
number-of-authors: 4
abstract: |
  Independent validation of experimental results in the field of 
  parallel and distributed systems research is a challenging task, 
  mainly due to changes and differences in software and hardware in 
  computational environments. Recreating an environment 
  that resembles the original is difficult and time-consuming. In this 
  paper we introduce the _Popper Convention_, a set of principles for 
  producing computational research that is easy to validate. 
  Concretely, we make the case for treating an article as an open 
  source software (OSS) project and for applying software engineering 
  best-practices to manage its associated artifacts and maintain the 
  reproducibility of its findings. We propose 
  leverage existing cloud-computing infrastructure and modern OSS 
  development tools to produce academic articles that are 
  easy to validate. We present a use case in the area of distributed 
  storage systems to illustrate the usefulness of this approach. We 
  show how, by following Popper, re-executing experiments becomes a 
  less daunting task and a reviewer can quickly get to the point of 
  getting results without relying on the author's invention.
documentclass: ieeetran
classoption: conference
ieeetran: true
numbersections: true
substitute-hyperref: true
csl: "ieee.csl"
bibliography: "citations.bib"
usedefaultspacing: true
fontfamily: times
conferencename: SC2016
copyrightyear: 2016
keywords:
 - reproducibility
 - computer-systems
 - systems-validation
 - systems-evaluation
linkcolor: black
---

# Introduction

A key component of the scientific method is the ability to revisit and 
replicate previous experiments. Managing information about an 
experiment allows scientists to interpret and understand results, as 
well as verify that the experiment was performed according to 
acceptable procedures. Additionally, reproducibility plays a major 
role in education since the amount of information that a student has 
to digest increases as the pace of scientific discovery accelerates. 
By having the ability to repeat experiments, a student learns by 
looking at provenance information about the experiment, which allows 
them to re-evaluate the questions that the original experiment 
addressed. Instead of wasting time managing package conflicts
and learning the paper author's ad-hoc experimental setups, the 
student can immediately run the original experiments and build on the 
results in the paper, thus allowing them to "stand on the shoulder of 
giants".

Independently validating experimental results in the field of computer 
systems research is a challenging task. Recreating an environment that 
resembles the one where an experiment was originally executed is a 
challenging endeavour. Version-control systems give authors,
reviewers and readers access to the same code base 
[@brown_how_2014] but the availability of source code does not 
guarantee reproducibility [@collberg_repeatability_2015]; code may not
compile, and even it does, the results may 
differ. In this case, validating the outcome is a subjective task that 
requires domain-specific expertise in order to determine the 
differences between original and recreated environments that might be 
the root cause of any discrepancies in the results 
[@jimenez_tackling_2015-1 ; @freire_computational_2012 ; 
@donoho_reproducible_2009]. Additionally, reproducing experimental 
results when the underlying hardware environment changes is 
challenging mainly due to the inability to predict the effects of such 
changes in the outcome of an experiment [saavedra-barrera_cpu_1992 ; 
woo_splash2_1995]. A Virtual Machine (VM) can be used to partially 
address this issue but the overheads in terms of performance (the 
hypervisor "tax") and management (creating, storing and transferring) 
can be high and, in some fields of computer science such as systems 
research, cannot be accounted for easily [@clark_xen_2004 ; 
@klimeck_nanohuborg_2008]. OS-level virtualization can help in 
mitigating the performance penalties associated with VMs 
[@jimenez_woc_2015].

One central issue in reproducibility is how to easily organize an 
article's experiments so that readers or students can easily repeat 
them. The current practice is to make the code available in a public 
repository and leave readers with the daunting task of recompiling, 
reconfiguring, deploying and re-executing an experiment. In this work, 
we revisit the idea of an executable paper 
[@strijkers_executable_2011], which poses the integration of 
executables and data with scholarly articles to help facilitate its 
reproducibility, but look at implementing it in today's 
cloud-computing world by treating an article as an open source 
software (OSS) project. We outline high-level guidelines for 
organizing an article's artifacts and make all these publicly 
available with the goal of easing the re-execution of experiments and 
validation of results. There are two main goals for our convention:

 1. It should apply to as many research projects as possible, 
    regardless of their domain. While the use case shown in _Section 
    IV_ pertains to the area of distributed storage systems, our goal 
    is to embody any project with a computational component in it.
 2. It should be applicable, regardless of the underlying 
    technologies. In general, Popper relies on software-engineering 
    practices like continuous integration (CI) which are implemented 
    in multiple existing tools. Applying this convention should work, 
    for example, regardless of what CI tool is being used.

By using version-control systems, lightweight OS-level virtualization, 
automated multi-node orchestration, continuous integration and 
web-based data visualization, re-executing and validating an 
experiment becomes practical. In particular, we make the case for 
using Git, Docker, Ansible and Jupyter notebooks, and use Github, 
Cloudlab, Binder and Travis as our proof-of-concept infrastructure.

The rest of the paper is organized as follows. _Section II_ gives an 
overview of the high-level workflow that a researcher goes through 
when writing an article following the Popper convention. _Section III_ 
describes _Popper_ in greater detail. In _Section IV_ we present a use 
case of a project following Popper. We discuss some of the central 
issues in _Section V_, review related work on _Section VI_ and 
conclude.

# An article as an OSS project


## Version Control

The idea of keeping an article's source in a version-controlled 
repository is not new and in fact many people follow this practice. 
What we propose is not only to have the source of an article in 
version control but to treat all the artifacts in a repository, 
treating it as a monolithic ball. This has several advantages. Take a 
look to a project and break it down into its components w.r.t. the 
development parts (code, tests, artifacts, 3rd party libs).

  * code: the latex file
  * artifacts: figures, input/output data
  * 3rd party libraries: code from us that we've developed for the 
    article or stuff we make use of
  * tests:
      * unit tests: check that PDF/figures are generated correctly
      * integration tests: experiments are runnable, e.g. all the 
        dependencies
      * regression tests: we ensure that the claims made in the paper 
        are valid, e.g. after a change on the associated code base or 
        by adding a new "supported platform"

A CI tool needs to be available, e.g. if we have Travis or Jenkins 
then we can make sure that we don't "break" the paper. For example in 
CloudLab, we might have multiple experiments that might be broken 
after a new site gets upgraded.

### GitHub

A web-based Git repository hosting service. It offers all of the 
distributed revision control and source code management (SCM) 
functionality of Git as well as adding its own features. It gives 
new users the ability to look at the entire history of the authors' 
scientific method.

## Configuration Management and Multi-node Orchestration

### Ansible

Ansible is a configuration management utility for configuring and 
managing computers, as well as deploying and orchestrating multi-node 
applications.

## Continuous Integration

### Travis CI

An open-source, hosted, distributed continuous integration service 
used to build and test software projects hosted at GitHub.

## Package Management and Application Deployment

Docker is usually viewed as an alternative to virtualization but can 
also be viewed as an enhancement on package management, where an app 
is ready to run without having to worry about packages and specific OS 
versions.

Docker automates the deployment of applications inside software 
containers by providing an additional layer of abstraction and 
automation of operating-system-level virtualization on Linux. 
Alternatives to docker are modern package managers such as Nix or 
Spack, or virtualmachines.

## Data Visualization

### Jupyter

Jupyter notebooks run on a web-based application. It facilitates the 
sharing of documents containing live code (in Julia, Python or R), 
equations, visualizations and explanatory text.

### Binder.org

Binder is an online service that allows one to turn a GitHub repository into 
a collection of interactive Jupyter notebooks so that readers don't 
need to deploy web servers themselves.

While we use all these, as stated in goal 2, any of these should be 
swappable for other tools, for example: VMs instead of Docker; Puppet 
instead of Ansible; Jenkins instead of Travis CI; and so on and so 
forth.
Yeah

## Bare-metal as a Service

### Cloudlab, Chameleon and PRObE

NSF-sponsored infrastructures for research on cloud computing that 
allows users to easily provision bare-metal machines to execute 
multi-node experiments.

## Automated Performance Regression Testing

There are not much out there but we have a tool for this called aver.

# The Popper Convention

_Popper_ is a convention for treating an article as an OSS project. 
Our approach can be summarized as follows:

  * Github repository stores all details for the paper. It stores the 
    metadata necessary to build the paper and re-run experiments. 
  * Docker images capture the experimental environment, packages and tunables.
  * Ansible playbook deploy and execute the experiments.
  * Travis tests the integrity of all experiments.
  * Jupyter notebooks analyze and visualize experimental data produced 
  * by the authors.
  * Every experiment involving performance metrics can be launched in 
    CloudLab, Chameleon or PRObE.
  * Every image in an article has a link in its caption that takes the 
    reader to a Jupyter notebook that visualizes the experimental 
    results.

Figure 1 shows the end-to-end workflow for reviewers and authors. 
Given all the elements listed above, readers of a paper can look at a 
figure and click the associated link that takes them to a notebook. 
Then, if desired, they instantiate a Binder and can analyze the data 
further. After this, they might be interested in re-executing an 
experiment, which they can do by cloning the github repository and, if 
they have resources available to them (i.e. one or more docker hosts), 
they just point Ansible to them and re-execute the experiment locally 
in their environment. If resources are not available, an alternative 
is to launch a Cloudlab, Chameleon or PRObE instance (one or more 
docker nodes hosted in Cloudlab) and point Ansible to the assigned IP 
addresses. An open question is how do we deal with datasets that are 
too big to fit in Git. An alternative is to use `git-lfs` to version 
and store large datasets.

![End-to-end workflow for an article that follows the Popper 
convention.](figures/wflow.png)

We now describe in greater detail our convention. As mentioned before, 
the main idea is to treat a paper like an OSS project.

## Organizing Files

The structure of a "paper repo" is shown in Figure 3. 

![Structure of a folder for a project following the Popper convention. 
The red markers correpond to dependencies for the generation of the 
PDF, while the blue ones mark files used for the 
experiment.](figures/experiment-metadata.png)

We note the following:

  * A paper is written in any desired format. Here we use markdown as 
    an example (`main.md`).

  * There is a `build.sh` command that generates the output format 
    (e.g. `PDF`).

  * Every experiment in the paper has a corresponding folder in the 
    repo. For example, `exp1` referred in a paper, there is a 
    `experiments/exp1/` folder in the repo.

  * Every figure in the paper has a `[source]` link in its caption 
    that points to the URL of the corresponding experiment folder in 
    the web interface of the VCS (e.g. github).

  * `notebook.ipynb` contains the notebook that, at the very least, 
    displays the figures for the experiment. It can serve as an 
    "extended" version of what figures in the paper display, including 
    other figures that contain analysis that show similar results. If 
    the repo is checked out locally into another person's machine, 
    it's a nice way of having readers play with the result's data 
    (although they need to know how to instantiate a local notebook 
    server).

  * If desired, the experiment can be re-executed. The high-level data 
    flow is the following:

    ```
      edit(inventory) -> invoke(run.sh) ->
        ansible(pull_docker_images) ->
        ansible(run_docker_images) ->
        fetch(output, facts, etc) ->
        postprocess ->
        genarate_image ->
        aver_assertions
    ```

    Thus, the absolutely necessary files are `run.sh` which bootstraps 
    the experiment (by invoking a containerized ansible); `inventory`, 
    `playbook.yml` and `vars.yml` which are given to ansible.

    The execution of the experiment will produce output that is either 
    consumed by a postprocessing script, or directly by the notebook. 
    The output can be in any format (CSVs, HDF, NetCDF, etc.).

  * `output.csv` is the ultimate output of the experiment and what it 
    gets displayed in the notebook.

  * `playbook.yml`, `inventory`, `vars.yml`. Files for `ansible`. An 
    important component of the playbook is that it should `assert` the 
    environment and corroborate as much assumptions as possible (e.g. 
    via the `assert` task). `vars.yml` contains the parametrization of 
    the experiment.

  * `assertions.aver`. An optional file that contains assertions on 
    the output data in the _aver_ language.

## Organizing Dependencies

### Executables

For every execution element in the high-level script, there is a repo 
that has the source code of the executables, and an artifact repo that 
holds the output of the "pointed-to" version. In our example, we use 
git and docker. So, let's say the execution that resulted in `fig1` 
refers to code of a `foo` codebase. Then:

  * there's a git repo for `foo` and there's a tag/sha1 that we refer 
    to in the paper repo. This can optionally be also 
    referenced/tracked via git submodules (e.g. placed in the 
    `vendor/` folder).

  * for the version that we are pointing to, there is a docker image 
    in the docker hub. E.g. if foo#tag1 is what we refer to, then 
    there's a docker image <repo>/foo:tag1. We can optionally track 
    the image's source (dockerfile) with submodules.

### Datasets

Input/output files should be also versioned. For small datasets, we 
can can put them in the git repo (as in the example). For large 
datasets we can use `git-lfs`.

## Obtaining and Reporting Baseline Raw Performance

We have a toolkit that is composed of multiple docker images that 
measure CPU, memory, I/O and network raw performance of a deployment. 
We make this part of the results since this is our fingerprint of our 
systems. This also gives us an idea of the proportionality of the 
multiple subsystems (e.g. 10:1 of network to IO for example)

# GassyFS: a model project for Popper

GassyFS [@watkins_gassyfs_2016] is a new prototype filesystem system 
that stores files in distributed remote memory and provides support 
for checkpointing file system state. The architecture of GassyFS is 
illustrated in Figure 2. The core of the file system is a user-space 
library that implements a POSIX file interface. File system metadata 
is managed locally in memory, and file data is distributed across a 
pool of network-attached RAM managed by worker nodes and accessible 
over RDMA or Ethernet. Applications access GassyFS through a standard 
FUSE mount, or may link directly to the library to avoid any overhead 
that FUSE may introduce.

![GassyFS has facilities for explicitly managing  persistence to 
different storage targets. A checkpointing infrastructure gives 
GassyFS flexible policies for persisting namespaces and federating 
data.](figures/arch.pdf)

By default all data in GassyFS is non-persistent. That is, all 
metadata and file data is kept in memory, and any node failure will 
result in data loss. In this mode GassyFS can be thought of as a 
high-volume tmpfs that can be instantiated and destroyed as needed, or 
kept mounted and used by applications with multiple stages of 
execution. The differences between GassyFS and tmpfs become apparent 
when we consider how users deal with durability concerns.

At the bottom of Figure 2 are shown a set of storage targets that can 
be used for managing persistent checkpoints of GassyFS. Given the 
volatility of memory, durability and consistency are handled 
explicitly by selectively copying data across file system boundaries. 
Finally, GassyFS supports a form of file system federation that allows 
checkpoint content to be accessed remotely to enable efficient data 
sharing between users over a wide-area network. 

In subsequent sections we describe several experiments run on GassyFS 
and detail how we obtained baselines.

## Experiment 1: GassyFS vs. TempFS

The goal of this experiment is to compare the performance of GassyFS 
with respect to that of TempFS on a single node. As mentioned before, 
the idea of GassyFS is to serve as a distributed version of TmpFS.
Figure 3 shows the results of this test. We can see that GassyFS, due 
to the FUSE overhead, performs within 90% of TmpFS's performance.

The corresponding experiment folder in the paper repository contains 
the necessary ansible files to re-execute this experiment with minimum 
effort. The only assumption is docker +1.10 and root access on the 
remote machine where this runs. The validation statements for this 
experiments are the following:

```
when
  workload=*, size=*
expect
  time(fs=gassyfs) > 0.8 * time(fs=tmpfs)

when
  workload=1
```

![GassyFS vs tmpfs variability.](figures/gassyfs-variability.png)

## Experiment 2: Scalability

f

## Experiment 3: Analytics on GassyFS

One of the use cases of tmpfs is in the analysis of data. When data is 
too big to fit in memory, alternatives resort to either scale-out or 
do


The corresponding experiment folder in the paper repository contains 
the necessary ansible files to re-execute this experiment with minimum 
effort. The only assumption is docker +1.10 and root access on the 
remote machine where this runs. The validation statements for this 
experiments are the following:

```
for
  workload=*
expect
  time(fs=gassyfs) > 0.8 * time(fs=tmpfs)
```

![Dask workload on GassyFS.](figures/dask.png)

# Lessons Learned

## Numerical vs. Performance Reproducibility

In many areas of computer systems research, the main subject of study 
is performance, a property of a system that is highly dependant on 
changes and differences in software and hardware in computational 
environments. Performance reproducibility can be contrasted with 
numerical reproducibility. Numerical reproducibility deals with 
obtaining the same numerical values from every run, with the same code 
and input, on distinct platforms. For example, the result of the same 
simulation on two distinct CPU architectures should yield the same 
numerical values. Performance reproducibility deals with the issue of 
obtaining the same performance (run time, throughput, latency, etc.) 
across executions. We set up an experiment on a particular machine and 
compare two algorithms or systems.

We can compare two systems with either controlled or statistical 
methods. In controlled experiments, the computational environment is 
controlled in such a way that the executions are deterministic, and 
all the factors that influence performance can be quantified. The 
statistical approach starts by first executing both systems on a 
number of distinct environments (distinct computers, OS, networks, 
etc.). Then, after taking a significant number of samples, the claims 
of the behavior of each system are formed in statistical terms, e.g. 
with 95\% confidence one system is 10x better than the other. The 
statistical reproducibility method is gaining popularity, e.g. 
[@hoefler_scientific_2015].

Current practices in the Systems Research community don't include 
either controlled or statistical reproducibility experiments. Instead, 
people run several executions (usually 10) on the same machine and 
report averages. Our research focuses in looking at the challenges of 
providing controlled environments by leveraging OS-level 
virtualization. [@jimenez_characterizing_2016] reports some 
preliminary work.

Our convention can be used to either of these two approaches.

# Related Work

The challenging task of evaluating experimental results in applied 
computer science has been long recognized [@ignizio_establishment_1971 
; @ignizio_validating_1973 ; @crowder_reporting_1979]. This issue has 
recently received a significant amount of attention from the 
computational research community [@freire_computational_2012 ; 
@neylon_changing_2012 ; @leveqije_reproducible_2012 ; 
@stodden_implementing_2014], where the focus is more on numerical 
reproducibility rather than performance evaluation. Similarly, efforts 
such as _The Recomputation Manifesto_ [@gent_recomputation_2013] and 
the _Software Sustainability Institute_ [@crouch_software_2013] have 
reproducibility as a central part of their endeavour but leave runtime 
performance as a secondary problem. In systems research, runtime 
performance _is_ the subject of study, thus we need to look at it as a 
primary issue. By obtaining profiles of executions and making them 
part of the results, we allow researchers to validate experiments with 
performance in mind.

Recent efforts have looked at creating open science portals or 
repositories [@bhardwaj_datahub_2014 ; @king_introduction_2007 ; 
@stodden_researchcompendiaorg_2015 ; @centerforopenscience_open_2014] 
that hold all (or a subset of) the artifacts associated to an article. 
In our case, by treating an article as an OSS project, we benefit from 
existing tools and web services such as git-lfs without having to 
implement domain-specific tools. In the same realm, some services 
provide researchers with the option of generating and absorving the 
cost of a digital object identifier (DOI). Github projects can have a 
DOI associated with it [@smith_improving_2014], which is one of the 
main reasons why we use it as the VCS in _Popper_

A related issue is the publication model. In 
[@chirigati_collaborative_2016] the authors propose to incentivize the 
reproduction of published results by adding reviewers as co-authors of 
a subsequent publication. We see the Popper convention as a 
complementary effort that can be used to make the facilitate the work 
of the reviewers.

The issue of structuring an articles associated files has been 
discussed in [@dolfi_model_2014], where the authors introduce a "paper 
model" of reproducible research which consists of an MPI application 
used to illustrate how to organize a project. In [@brown_how_2014], 
the authors propose a similar approach based on the use of `make`, 
with the purpose of automating the generation of a PDF file. We extend 
these ideas by having our convention be centered around OSS 
development practices and include the notion of instant replicability 
by using docker and ansible.

In [@collberg_measuring_2014] the authors took 613 articles published 
in 13 top-tier systems research conferences and found that 25% of the 
articles are reproducible (under their reproducibility criteria). The 
authors did not analyze performance. In our case, we are interested 
not only in being able to rebuild binaries and run them but also in 
evaluating the performance characteristics of the results.

Containers, and specifically docker, have been the subject of recent 
efforts that try to alleviate some of the reproducibility problems in 
data science [@boettiger_introduction_2014]. Existing tools such as 
Reprozip [@chirigati_reprozip_2013] package an experiment in a 
container without having to initially implement it in one (i.e. 
automates the creation of a container from an "non-containerized" 
environment). This tool can be useful for researchers that aren't 
familiar with tools

# Conclusion

In the words of Karl Popper: "_the criterion of the scientific status 
of a theory is its falsifiability, or refutability, or testability_". 
By providing a way to specify the high-level components of an 
experiment and validation clauses for observed metrics we effectively 
incorporate falsifiability to the field of experimental storage 
systems. We are in the process of studying the viability of the ESF on 
experiments from other areas of systems research. While we envision 
our findings to be applicable in the area of systems research, we plan 
to evaluate its suitability on other areas of computer science.

**Acknowledgements:** We would like to thank all the good people.

# Bibliography

<!-- hanged biblio -->

\noindent
\vspace{-2em}
\setlength{\parindent}{-0.22in}
\setlength{\leftskip}{0.2in}
\setlength{\parskip}{8pt}
