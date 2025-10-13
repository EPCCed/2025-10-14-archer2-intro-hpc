---
title: "Accessing software via Modules"
teaching: 30
exercises: 15
---



::: questions
 - How do we load and unload software packages?
:::

::: objectives
 - Understand how to load and use a software package.
:::

On a HPC system, it is seldom the case that the software
we want to use is available when we log in. It may be installed, but we will need
to "load" it before it can run.

Before we start using individual software packages, however, we should
understand the reasoning behind this approach. The three biggest factors are:

### Software incompatibilities
Software incompatibility is a major headache for programmers. Sometimes the
presence (or absence) of a software package will break others that depend on
it. Two of the most famous examples are Python 2 vs 3 and C compiler versions: 
- Python 3 famously provides a `python` command that conflicts with that provided
  by Python 2. 
- Software compiled against a newer version of the C libraries and
  then used when they are not present will result in a nasty `'GLIBCXX_3.4.20' not found` error.

### Versioning
Software versioning is another common issue. A team might depend on a certain
package version for their research project - if the software version was to
change (for instance, if a package was updated), it might affect their results.
Having access to multiple software versions on the same system helps to
prevent software versioning issues from affecting their results.

### Dependencies
Dependencies are where a particular software package (or even a particular
version) depends on having access to another software package (or even a
particular version of another software package). For example, the VASP
materials science software may depend on having a particular version of the
FFTW (Fastest Fourier Transform in the West) software library available for it
to work. Therefore, it is useful to have multiple versions available for multiple use cases. 

## Environment Modules

Environment modules are the solution to these problems. A *module* is a
self-contained description of a software package - it contains the
settings required to run a software package and, usually, encodes required
dependencies on other software packages.

There are a number of different environment module implementations commonly
used on HPC systems: the two most common are *TCL modules* and *Lmod*. Both of
these use similar syntax and the concepts are the same so learning to use one
will allow you to use whichever is installed on the system you are using. In
both implementations the `module` command is used to interact with environment
modules. An additional subcommand is usually added to the command to specify
what you want to do. For a list of subcommands you can use `module -h` or
`module help`. As for all commands, you can access the full help on the *man*
pages with `man module`.

On login you may start out with a default set of modules loaded or you may
start out with an empty environment; this depends on the setup of the system
you are using.

### Listing Available Modules

To see available software modules, use `module avail`:

```bash
userid@ln03:/work/ta215/ta215/userid> module avail
```

```output
-------------------------------------------------------------- /opt/cray/pe/lmod/modulefiles/core ---------------------------------------------------------------
   PrgEnv-amd/8.3.3      (D)      aocc/4.0.0        (D)      cray-cti/2.16.0                    cray-pals/1.2.5      (D)      gdb4hpc/4.14.6         (D)
   PrgEnv-amd/8.4.0               atp/3.14.16       (D)      cray-cti/2.17.1           (D)      cray-pals/1.2.12              gdb4hpc/4.15.1
   PrgEnv-aocc/8.3.3     (D)      atp/3.15.1                 cray-cti/2.18.1                    cray-pmi/6.1.8       (D)      iobuf/2.0.10
   PrgEnv-aocc/8.4.0              cce/15.0.0        (L,D)    cray-dsmml/0.2.2          (L)      cray-pmi/6.1.12               papi/6.0.0.17          (D)
   PrgEnv-cray-amd/8.3.3          cce/16.0.1                 cray-dyninst/12.1.1       (D)      cray-python/3.9.13.1 (D)      papi/7.0.1.1
   PrgEnv-cray-amd/8.4.0 (D)      cpe-cuda/22.12    (D)      cray-dyninst/12.3.0                cray-python/3.10.10           perftools-base/22.12.0 (L,D)
   PrgEnv-cray/8.3.3     (L,D)    cpe-cuda/23.09             cray-libpals/1.2.5        (D)      cray-stat/4.11.13    (D)      perftools-base/23.09.0
   PrgEnv-cray/8.4.0              cpe/22.12         (D)      cray-libpals/1.2.12                cray-stat/4.12.1              rocm/5.2.3
   PrgEnv-gnu-amd/8.3.3           cpe/23.09                  cray-libsci/22.12.1.1     (L,D)    craype/2.7.19        (L,D)    sanitizers4hpc/1.0.4   (D)
   PrgEnv-gnu-amd/8.4.0  (D)      cray-R/4.2.1.1    (D)      cray-libsci/23.09.1.1              craype/2.7.23                 sanitizers4hpc/1.1.1
   PrgEnv-gnu/8.3.3      (D)      cray-R/4.2.1.2             cray-libsci_acc/22.12.1.1 (D)      craypkg-gen/1.3.28   (D)      valgrind4hpc/2.12.10   (D)
   PrgEnv-gnu/8.4.0               cray-ccdb/4.12.13 (D)      cray-libsci_acc/23.09.1.1          craypkg-gen/1.3.30            valgrind4hpc/2.13.1
   amd/5.2.3                      cray-ccdb/5.0.1            cray-mrnet/5.0.4          (D)      gcc/10.3.0
   aocc/3.2.0                     cray-cti/2.15.14           cray-mrnet/5.1.1                   gcc/11.2.0           (D)

----------------------------------------------------- /opt/cray/pe/lmod/modulefiles/craype-targets/default ------------------------------------------------------
   craype-accel-amd-gfx908    craype-arm-grace        craype-hugepages2G      craype-hugepages64M        craype-x86-genoa          craype-x86-spr
   craype-accel-amd-gfx90a    craype-hugepages128M    craype-hugepages2M      craype-hugepages8M         craype-x86-milan-x        craype-x86-trento
   craype-accel-host          craype-hugepages16M     craype-hugepages32M     craype-network-none        craype-x86-milan
   craype-accel-nvidia70      craype-hugepages1G      craype-hugepages4M      craype-network-ofi  (L)    craype-x86-rome    (L)
   craype-accel-nvidia80      craype-hugepages256M    craype-hugepages512M    craype-network-ucx         craype-x86-spr-hbm

...

Many more 

... 

```

### Listing Currently Loaded Modules

You can use the `module list` command to see which modules you currently have
loaded in your environment. If you have no modules loaded, you will see a
message telling you so

```bash
userid@ln03:/work/ta215/ta215/userid> module list
```
```output
Currently Loaded Modules:
  1) craype-x86-rome                         6) cce/15.0.0             11) PrgEnv-cray/8.3.3
  2) libfabric/1.12.1.2.2.0.0                7) craype/2.7.19          12) bolt/0.8
  3) craype-network-ofi                      8) cray-dsmml/0.2.2       13) epcc-setup-env
  4) perftools-base/22.12.0                  9) cray-mpich/8.1.23      14) load-epcc-module
  5) xpmem/2.5.2-2.4_3.30__gd0f7936.shasta  10) cray-libsci/22.12.1.1
```

## Loading and Unloading Software

To load a software module, use `module load`. Let's say we would like
to use the HDF5 utility `h5dump`. 

On login, `h5dump` is not available. We can test this by using the `which`
command. `which` looks for programs the same way that Bash does,
so we can use it to tell us where a particular piece of software is stored.

```bash
 which h5dump
```
```output
which: no h5dump in (/work/y07/shared/utils/core/bolt/0.8/bin:/mnt/lustre/a2fs-work4/work/y07/shared/utils/core/bin:/opt/cray/pe/mpich/8.1.23/ofi/crayclang/10.0/bin:/opt/cray/pe/mpich/8.1.23/bin:/opt/cray/pe/craype/2.7.19/bin:/opt/cray/pe/cce/15.0.0/binutils/x86_64/x86_64-pc-linux-gnu/bin:/opt/cray/pe/cce/15.0.0/binutils/cross/x86_64-aarch64/aarch64-linux-gnu/../bin:/opt/cray/pe/cce/15.0.0/utils/x86_64/bin:/opt/cray/pe/cce/15.0.0/bin:/opt/cray/pe/cce/15.0.0/cce-clang/x86_64/bin:/opt/cray/pe/perftools/22.12.0/bin:/opt/cray/pe/papi/6.0.0.17/bin:/opt/cray/libfabric/1.12.1.2.2.0.0/bin:/usr/local/bin:/usr/bin:/bin:/usr/lib/mit/bin:/opt/cray/pe/bin)
```

We can find the `h5dump` command by using `module load`:

```bash
 module load cray-hdf5
 which h5dump
```
```output
/opt/cray/pe/hdf5/1.12.2.1/bin/h5dump
```

So, what just happened?

To understand the output, first we need to understand the nature of the
`$PATH` environment variable. `$PATH` is a special environment variable
that controls where a UNIX system looks for software. Specifically,
`$PATH` is a list of directories (separated by `:`) that the OS searches
through for a command before giving up and telling us it can't find it.
As with all environment variables we can print it out using `echo`.

```bash
 echo $PATH
```
```output
/opt/cray/pe/hdf5/1.12.2.1/bin:/work/y07/shared/utils/core/bolt/0.8/bin:/mnt/lustre/a2fs-work4/work/y07/shared/utils/core/bin:/opt/cray/pe/mpich/8.1.23/ofi/crayclang/10.0/bin:/opt/cray/pe/mpich/8.1.23/bin:/opt/cray/pe/craype/2.7.19/bin:/opt/cray/pe/cce/15.0.0/binutils/x86_64/x86_64-pc-linux-gnu/bin:/opt/cray/pe/cce/15.0.0/binutils/cross/x86_64-aarch64/aarch64-linux-gnu/../bin:/opt/cray/pe/cce/15.0.0/utils/x86_64/bin:/opt/cray/pe/cce/15.0.0/bin:/opt/cray/pe/cce/15.0.0/cce-clang/x86_64/bin:/opt/cray/pe/perftools/22.12.0/bin:/opt/cray/pe/papi/6.0.0.17/bin:/opt/cray/libfabric/1.12.1.2.2.0.0/bin:/usr/local/bin:/usr/bin:/bin:/usr/lib/mit/bin:/opt/cray/pe/bin
```

You'll notice a similarity to the output of the `which` command. In this case,
there's only one difference: the different directory at the beginning. When we
ran the `module load` command, it added a directory to the beginning of our
`$PATH`. Let's examine what's there:

```bash
 ls /opt/cray/pe/hdf5/1.12.2.1/bin/
```
```output
gif2h5	h5c++  h5clear	h5debug  h5dump  h5format_convert  h5jam  h5mkgrp	 h5redeploy  h5repart  h5unjam
h52gif	h5cc   h5copy	h5diff	 h5fc	 h5import	   h5ls   h5perf_serial  h5repack    h5stat    h5watch
```

In summary, `module load` will add software to your `$PATH`.
`module load` may also load additional modules with software dependencies.

To unload a module, use `module unload` with the relevant module name.

::: challenge
## Unload!
Confirm you can unload the `cray-hdf5` module and check what happens to the `PATH` environment variable.
:::

## Software versioning

So far, we've learned how to load and unload software packages. This is very useful, however we
have not yet addressed the issue of software versioning. At some point or other, you will run into
issues where only one particular version of some software will be suitable. Perhaps a key bugfix
only happened in a certain version, or version X broke compatibility with a file format you use. In
either of these example cases, it helps to be very specific about what software is loaded.

Let's examine the output of `module avail` more closely.

```bash
 module avail cray-hdf5
```
```output
---------------- /opt/cray/pe/lmod/modulefiles/mpi/crayclang/14.0/ofi/1.0/cray-mpich/8.0 ----------------
   cray-hdf5-parallel/1.12.2.1 (D)    cray-hdf5-parallel/1.12.2.7

---------------- /opt/cray/pe/lmod/modulefiles/compiler/crayclang/14.0 ----------------------------------
   cray-hdf5/1.12.2.1 (L,D)    cray-hdf5/1.12.2.7

  Where:
   L:  Module is loaded
   D:  Default Module

```

Note that we have two different versions of `cray-hdf5`.

::: challenge
## Using `module swap`
Load module `cray-hdf5` as before. Note that if we do not specifify
a particular version, we load a default version.
If we wish to change versions, we can use
`module swap <old-module> <new-module>`. Try this to obtain
`cray-hdf5/1.12.2.7`. Check what has happened to the location of
the `h5dump` utility.
:::

::: challenge
## Using Software Modules in Scripts
Create a job that is able to run `h5dump --version`. 
Remember, submitting a job is just like logging in to a new remote system. What modules would you
expect to be there? 


::: solution

```output
#!/bin/bash
#SBATCH --partition=standard
#SBATCH --qos=short
module load cray-hdf5
h5dump --version
```

:::
:::

::: keypoints
- HPC systems use modules to help deal with software incompatibilities, versioning and dependencies.
- We can see what modules we currently have loaded with `module list`.
- We can see what modules are available with `module avail`.
- We can load a module with `module load softwareName`.
- We can unload a module with `module unload softwareName`.
- We can swap modules for different versions with `module swap old-softwareName new-softwareName`.
:::
